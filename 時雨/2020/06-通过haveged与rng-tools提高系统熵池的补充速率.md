---
title: 通过haveged与rng-tools提高系统熵池的补充速率
date: 2020-07-12 20:00:00
category: 折腾笔记
tags:
  - CentOS
  - 服务器
---

本文将简单介绍Linux系统的熵池系统，以及如何通过`haveged`与`rng-tools`提高系统熵池补充速度，从而避免系统阻塞。

## 前言

许多使用过廉价入门主机的朋友可能遇到过，系统花了很长时间才能连到 VPS 主机，许多新手并不知道为什么主机的启动速度如此缓慢。

所以现在来写篇文章记录一下。

## Linux 的熵

> Entropy（熵，[shāng]）在信息论中表示数据的混乱程度或者不确定性，混乱程度越高则数据更难被预测。  
> 在信息论里，熵是对不确定性的度量，熵值越高则能传递多的信息，熵值越低能传递的信息越少。

对于使用者不需要考虑这么复杂的问题，我们需要知道的是Linux系统有两套随机数生成机制。

## Linux 系统的随机数生成机制

`/dev/random` 在 Linux 系统里被认为是真随机数发生器，会持续收集系统的环境噪音，比如硬件设备的活动（键盘输入、磁盘读写、内存错误等），这些活动被认为无法被预测，因此系统可以依靠这些随机信号生成高随机性的公钥或一次性密码本。如果 `/dev/random` 空了，系统会等到收集到足够的熵才会继续进行加密操作。

`/dev/urandom` 在 Linux 系统里则被认为是伪随机数发生器，与前者不同，即使空掉，Linux 也会持续生成伪随机数，所以不会被堵塞。

## 系统缓慢原因

由于 `openssl` 、 `tomcat` 等高加密要求的软件需要 `/dev/random` 所生成的真随机数来确保公匙不易于被猜测，与此同时由于 Linux 从 2.4 至 2.6 版本砍掉了大量的外来噪音来源，以及 VPS 由于是虚拟主机环境过于安静而缺乏各种噪音，因此会出现随机数生成速度过慢所产生的阻塞现象。

## 操作

本文建议大家在主机上安装 `haveged` 和 `rng-tools` 从而大量生成熵值避免阻塞。

### 查看系统熵值

通过以下命令，我们可以查看系统熵值：

```bash
watch -n 1 cat /proc/sys/kernel/random/entropy_avail

782
```

可以看出这一台主机的熵池非常少，因此在启动过程中可以明显感觉到堵塞。我至少等待了 20s 来完成连接主机。

### 安装增熵服务

我们先通过以下命令来安装增熵服务：

```bash
# CentOS 7
yum install rng-tools haveged -y
# Debian 9
apt install rng-tools haveged -y
```

Centos 可能无法安装 `rng-tools`，需要根据以下命令添加 `enpl` 源：

```bash
yum -y install epel-release
```

接着我们启动服务并确保他们自启

```bash
# haveged

systemctl enable haveged
systemctl restart haveged 
systemctl status haveged

# rng-tools

systemctl enable rng-tools 
systemctl status rng-tools
```

Centos 首次可能无法启动。可以通过以下命令启动 `rng-tools`：

```bash
echo 'EXTRAOPTIONS="--rng-device /dev/urandom"' >/etc/sysconfig/rngd 
service rngd restart 
chkconfig rngd on
```

## 测试系统熵值

接着我们再来测试熵值：

```bash
watch -n 1 cat /proc/sys/kernel/random/entropy_avail

3132
```

## 查看结果

我们可以看出，系统的熵值被迅速提高，从782迅速提升到了3132。阻塞的情况有了显著缓解。
