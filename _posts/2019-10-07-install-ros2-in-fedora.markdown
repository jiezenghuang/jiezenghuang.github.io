---
layout: post
title:  "在Fedora 30中安装ROS 2.0"
date:   2019-10-07 19:04:28
categories: Fedora ROS2
#permalink: pretty
---

## 系统信息

笔者使用的是Windows 10自带的Hyper-V虚拟机来安装Fedora，安装的是Fedora Server 30，不带图像界面。

``` console
$ cat /etc/system-release
Fedora release 30 (Thirty)
$ uname -a
Linux ros2-vm 5.2.17-200.fc30.x86_64 #1 SMP Mon Sep 23 13:42:32 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## 安装系统依赖

安装之前建议先[更新系统](archivers/fedora#%E6%9B%B4%E6%96%B0%E7%B3%BB%E7%BB%9F)。

>可参考[官网安装指引](https://index.ros.org/doc/ros2/Installation/Dashing/Fedora-Development-Setup/)。

``` console
$ sudo dnf install \
  asio-devel \
  cmake \
  cppcheck \
  eigen3-devel \
  gcc-c++ \
  liblsan \
  libXaw-devel \
  libyaml-devel \
  make \
  opencv-devel \
  patch \
  python3-argcomplete \
  python3-colcon-common-extensions \
  python3-coverage \
  python3-devel \
  python3-empy \
  python3-lark-parser \
  python3-lxml \
  python3-mock \
  python3-nose \
  python3-pep8 \
  python3-pip \
  python3-pydocstyle \
  python3-pyflakes \
  python3-pyparsing \
  python3-pytest \
  python3-pytest-cov \
  python3-pytest-runner \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  python3-yaml \
  poco-devel \
  poco-foundation \
  python3-flake8 \
  python3-flake8-import-order \
  redhat-rpm-config \
  tinyxml-devel \
  tinyxml2-devel \
  uncrustify \
  wget
```

>后续安装流程也可以参考[官网安装指南](https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/#dashing-linux-dev-get-ros2-code)。

## 获取ROS 2源码

>官网安装指南建议安装dashing发布版本，但笔者之前编译dashing版本发现有一处头文件缺失的bug，会导致编译出错，不知道是否已经修复。为保险起见，本文使用master版本进行安装。

创建工作目录ros2_ws，并获取所有ros 2源码。基于网络情况，可能需要等待比较长的时间。

``` console
$ mkdir -p ~/ros2_ws/src
$ cd ~/ros2_ws
$ wget https://raw.githubusercontent.com/ros2/ros2/master/ros2.repos
--2019-10-07 19:04:28--  https://raw.githubusercontent.com/ros2/ros2/master/ros2.repos
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.108.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10347 (10K) [text/plain]
Saving to: ‘ros2.repos’

ros2.repos                           100%[=====================================================================>]  10.10K  --.-KB/s    in 0.001s

2019-10-07 19:04:32 (7.19 MB/s) - ‘ros2.repos’ saved [10347/10347]
$ vcs import src < ros2.repos
Cloning into '.'...
Already on 'master'
Your branch is up to date with 'origin/master'.
```

>后续编译过程中还有一些组件需要访问raw.githubusercontent.com下载依赖。如果出现raw.githubusercontent.com访问失败，建议更换一下网络再重新尝试。

## 使用rosdep安装依赖

``` console
$ sudo rosdep init
Wrote /etc/ros/rosdep/sources.list.d/20-default.list
Recommended: please run

        rosdep update
$ rosdep update
reading in sources list data from /etc/ros/rosdep/sources.list.d
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/releases/fuerte.yaml
Query rosdistro index https://raw.githubusercontent.com/ros/rosdistro/master/index-v4.yaml
Skip end-of-life distro "ardent"
Skip end-of-life distro "bouncy"
Add distro "crystal"
Add distro "dashing"
Add distro "eloquent"
Skip end-of-life distro "groovy"
Skip end-of-life distro "hydro"
Skip end-of-life distro "indigo"
Skip end-of-life distro "jade"
Add distro "kinetic"
Skip end-of-life distro "lunar"
Add distro "melodic"
Add distro "noetic"
updated cache in /home/matthew/.ros/rosdep/sources.cache
$ rosdep install --from-paths src --ignore-src --rosdistro dashing -y --skip-keys "console_bridge fastcdr fastrtps libopensplice67 libopensplice69 rti-connext-dds-5.3.1 urdfdom_headers"
```

## 安装其他DDS实现（可选）

前面步骤下载的ROS 2源码已经包含了eProsima’s Fast RTPS，如果需要安装其他DDS实现，请参考[官网安装指南](https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/#install-more-dds-implementations-optional)。

## 编译

编译的过程比较耗时，中途还需要联网下载组件源码和依赖，需要耐心等待。

``` console
$ cd ~/ros2_ws/
$ colcon build --symlink-install
Summary: 282 packages finished [1h 31min 29s]
  10 packages had stderr output: ros1_bridge rviz2 rviz_common rviz_default_plugins rviz_ogre_vendor rviz_rendering rviz_rendering_tests rviz_visual_testing_framework tf2_eigen tf2_kdl
```

有些组件（比如：rviz_assimp_vendor, rviz_ogre_vendor）下载比较耗时，如果下载失败容易出现编译失败。出现编译失败后，可以使用第三方下载工具先下载好编译源，再重新编译。以rviz_ogre_vendor为例：

* 进入日志目录

``` console
$ cd ~/ros2_ws/log/build_<最近编译时间目录>/rviz_ogre_vendor
```

* 在stdout_stderr.log文件中找到下载源”src=“及目标路径“dst=”

``` console
$ cat stdout_stderr.log
[ 12%] Creating directories for 'ogre-v1.12.1'
[ 25%] Performing download step (download, verify and extract) for 'ogre-v1.12.1'
-- Downloading...
   dst='/home/matthew/ros2_ws/build/rviz_ogre_vendor/ogre-v1.12.1-prefix/src/v1.12.1.zip'
   timeout='1200 seconds'
-- Using src='https://github.com/OGRECave/ogre/archive/v1.12.1.zip'
```

* 使用第三方下载工具下载https://github.com/OGRECave/ogre/archive/v1.12.1.zip，并将其放到~/ros2_ws/build/rviz_ogre_vendor/ogre-v1.12.1-prefix/src/目录下

* 重新编译。

## 检验是否安装成功

编译完成后，可以运行ROS 2源码自带的Demo程序检验ROS 2是否安装成功。
打开一个终端执行：

``` console
$ . ~/ros2_ws/install/local_setup.bash #导入环境变量
$ ros2 run demo_nodes_cpp talker # 运行cpp demo talker
[INFO] [talker]: Publishing: 'Hello World: 52'
[INFO] [talker]: Publishing: 'Hello World: 53'
[INFO] [talker]: Publishing: 'Hello World: 54'
[INFO] [talker]: Publishing: 'Hello World: 55'
[INFO] [talker]: Publishing: 'Hello World: 56'
```

打开另一个终端执行：

``` console
$ . ~/ros2_ws/install/local_setup.bash
$ ros2 run demo_nodes_py listener # 运行python demo talker
[INFO] [listener]: I heard: [Hello World: 52]
[INFO] [listener]: I heard: [Hello World: 53]
[INFO] [listener]: I heard: [Hello World: 54]
[INFO] [listener]: I heard: [Hello World: 55]
[INFO] [listener]: I heard: [Hello World: 56]
```

看到上述信息，表示ROS 2安装成功。
