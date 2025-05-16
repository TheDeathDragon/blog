---
title: Linux 下 DD 安装 Windows
date: 2018-09-17 08:00:00
category: 折腾笔记
tags:
  - CentOS
---

当你有台闲置的服务器的时候，你想拿来挂机器人或者挂卡什么的，但是服务器又没有提供 Windows 的系统，所以想找个办法在 Linux 上装 Windows。

## 所需配置

1H 1G以上。

本文以最常见的 Centos7 系统来测试。

## 准备工作

### 准备安装所需的环境

1、安装 gawk sed grep

```bash
#Debian/Ubuntu:
apt-get install -y gawk sed grep
#RedHat/CentOS:
yum install -y gawk sed grep
```

如果报错，请先检查更新再重试：

```bash
#Debian/Ubuntu:
apt-get update
#RedHat/CentOS:
yum update
```

2、无DHCP时额外需求：iconv

```bash
#Debian/Ubuntu
## 一般自带
#RedHat/CentOS
yum install glibc-common
```

### 准备一个DD包的直链

收集了一些现成的DD包，当然喜欢折腾也可以自己打包制作。

1、Lolico

```bash
# Windows server 2008 R2 Standard 中文版
# 2.9G（解压后8G） KVM 未激活 全新安装 已经过测试可用
http://nico-ni.co/dd/Win2008R2Standard-x64.gz
# Windows 10 LTSB
# 3.18G（解压后10G） KVM 未激活 全新安装 已经过测试可用
谷歌盘文件ID: 1o75Pawa30_1W1-VUlqk7BhyHSdQFzZWR
# Username: Administrator
# Password: lolico.moe123
```

2、萌咖

```bash
# Windows 7 Embedded Standard x86 (Thin PC)
# 1.19G KVM XEN 已激活
https://moeclub.org/get-win7embx86-auto
# Windows 8.1 Embedded Industry Pro x64
# 2.87G KVM XEN Hyper-V 未激活
https://moeclub.org/get-win8embx64-auto
# Username: Administrator
# Password: Vicer
```

3、Joodle

```bash
# Windows server 2008 R2
http://down.80host.com/iso/dd/WS2008R2Enterprise-Joodle-Template.gz
# Windows server 2012 R2
http://down.80host.com/iso/dd/Windows2012R2-Joodle-Template.gz
# Windows 7
http://down.80host.com/iso/dd/Windows7-Joodle-Template.gz
# Windows 8.1
http://down.80host.com/iso/dd/Windows8.1-Joodle-Template.gz
# Username: Administrator
# Password: Password147
```

4、WhatUpTime

```bash
# Windows 7 Enterprise x64
http://mirror.whatuptime.com/besw26/7.ENT.EVAL.64.VIRTIO-SCSI.gz
http://down.80host.com/iso/dd/7.ENT.EVAL.64.VIRTIO-SCSI.gz
# Username: WhatUpTime.com
# Password: P@ssword64
```

5、80Host

补充：有次续费80Host的机器，刚好碰上活动，80Host还寄了麻辣兔头给我，抽真空的包装。不过我是真的接收不了，所以给别人吃了。

```bash
# Windows 7
# 支持OVH VPS的scsi磁盘驱动，其他viostor的DD包在上面会蓝屏
http://down.80host.com/iso/dd/win7_cn_5gb_virtio_scsi.gz
http://down.80host.com/iso/dd/win7_cn_5gb_virtio_scsi_faster.gz
# Username: administrator
# Password: www.80host.com

# Leaseweb / Linode 专用
http://down.80host.com/iso/dd/cn2003-virtio-pass-Linode.gz
# Username: Administrator
# Password: Linode
```

6、yxz

```bash
# Windows server 2003 for Kimsufi
http://down.80host.com/iso/dd/Kimsufi2003.gz
# Username: Administrator
# Password: password!yxz.me
```

7、Laiboke

```bash
# Windows server 2012 R2 中文版
http://down.80host.com/iso/dd/Win2012R2ZW.gz
# Username: Administrator
# Password: Laiboke.com
```

## 开始搭建

使用的命令：

```bash
wget --no-check-certificate -qO DebianNET.sh 'https://moeclub.org/attachment/LinuxShell/DebianNET.sh' && bash DebianNET.sh -dd '[Windows dd包直连地址]'
```

本文所使用的：

```bash
wget --no-check-certificate -qO DebianNET.sh 'https://moeclub.org/attachment/LinuxShell/DebianNET.sh' && bash DebianNET.sh -dd 'https://moeclub.org/get-win7embx86-auto'
```

## 注意事项

建议在安装前先把各组件更新到最新的版本，避免出现异常情况。

```bash
# Debian/Ubuntu:
apt-get update
# RedHat/CentOS:
yum update
```

此外，请阅读你服务器的TOS守则，避免因使用此脚本而违规。
