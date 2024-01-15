---
title: 通过预置KernelSU到Android构建中来获取Root权限
date: 2023-03-11 15:51:00
category: 折腾笔记
tags:
  - Android
  - 编程
---

通过内置 KernelSU 来获取对设备的 Root 权限，特别是在 user 版本默认没有 root 的情况下，或者有客户有特殊的需求的时候。

## 简介

KernelSU 是 Android GKI 设备的 root 解决方案，它工作在内核模式，并直接在内核空间中为用户空间应用程序授予 root 权限。

## 如何构建

因为国内对 github 连通性很差，建议直接下载下来

下载KernelSu：https://github.com/tiann/KernelSU

下载好的文件解压缩到对应的kernel目录当中

同时修改 `kernel/setup.sh` 文件，去掉 `git` 相关指令（因为我们已经下载下来了，不需要clone）

```diff
- test -d "$GKI_ROOT/KernelSU" || git clone https://github.com/tiann/KernelSU
+ test -d "$GKI_ROOT/KernelSU"

- git stash && git pull
```

之后，执行脚本自动集成，记得先 `chmod +x` 给权限

`./kernel-4.19/KernelSU/kernel/setup.sh`

然后正常编译即可。

## 启用

KernelSu 默认是不启用的，需要安装 KernelSu 的管理 APP 来对 Root 权限进行管理和授权

APP 下载地址：https://github.com/tiann/KernelSU/releases

当安装完 APP 之后，显示 Working 就说明已经编进去了，在 superuser 界面就可以进行授权操作了

## 安装其他模块

推荐 Zygisk On KernelSu ：https://github.com/Dr-TSNG/ZygiskNext

其他的例如存储空间隔离，核心破解等模块也是实用方便的

## 给应用授权

以MT管理器为例，再对其进行授权之后，在软件自带的终端中执行 su 即可提权为 root

执行需要 root 权限的 sentest

打印出了对应的 Sensor 信息，验证OK