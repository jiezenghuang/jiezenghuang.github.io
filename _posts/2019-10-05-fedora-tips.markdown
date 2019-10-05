---
layout: post
title:  "Fedora Tips"
date:   2019-10-05 12:02:13
categories: Fedora
permalink: /archivers/fedora
---

## 更新系统

``` console
$sudo dnf update #更新系统
$sudo dnf clean all #删除更新临时文件
```

## 删除旧内核

如果是刚刚更新系统，建议重启后再删除旧内核

``` console
$sudo reboot now #重启系统
$sudo dnf list installed | grep kernel-core #查询已安装内核版本
kernel-core.x86_64                     5.2.16-200.fc30                 @updates
kernel-core.x86_64                     5.2.17-200.fc30                 @updates
$sudo dnf remove kernel-core-5.2.16 #删除旧版本5.2.16内核
```
