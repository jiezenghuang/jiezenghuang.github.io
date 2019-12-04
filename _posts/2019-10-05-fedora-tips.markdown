---
layout: post
title:  "Fedora Tips"
date:   2019-10-05 12:02:13
categories: Fedora
#permalink: pretty
---

## 更新系统

``` console
$ sudo dnf update #更新系统
$ sudo dnf clean all #删除更新临时文件
```

## 删除旧内核

如果是刚刚更新系统，建议重启后再删除旧内核

``` console
$ sudo reboot now #重启系统
$ sudo dnf list installed | grep kernel-core #查询已安装内核版本
kernel-core.x86_64                     5.2.16-200.fc30                 @updates
kernel-core.x86_64                     5.2.17-200.fc30                 @updates
$ sudo dnf remove kernel-core-5.2.16 #删除旧版本5.2.16内核
```

## 安装驱动

编译好后，发现rviz2运行时会闪一下，然后会崩溃，在网上搜了一些文章，分析应该显卡驱动问题。由于笔者的笔记本是Intel集显，所以先从Intel显卡驱动入手（首次安在建议先看第三步）：

* 安装Intel和mesa驱动，安装完重启，rviz2仍然崩溃。

``` console
$ sudo dnf install igt-gpu-tools libva-intel-driver libva-utils libva mesa-libOSMesa cairo-gobject cairo mesa-dri-drivers mesa-filesystem mesa-libEGL mesa-libGL mesa-libGLES mesa-libgbm mesa-libglapi mesa-libwayland-egl mesa-libxatracker
```

* 安装在某个帖子里看到的一些不知道是什么的驱动，安装完重启，rviz2仍然崩溃。
``` console
$ sudo dnf install libXt-devel boost-devel freetype-devel freeimage-devel ois-devel cppunit-devel doxygen libXaw-devel libXrandr-devel zziplib-devel Cg
```

* 安装nvidia kmod，这是在orge官网论坛看到的，重启后rviz2可以正常启动了。不知道前面两步的驱动不安装会不会有什么问题，首次安装可以先试试先安装kmod。
``` console
$ sudo dnf install kmod-nvidia
``` 