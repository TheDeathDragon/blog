---
title: 如何开启 Google BBR 加速
date: 2019-04-11 08:46:00
category: 折腾笔记
tags:
  - Nginx
  - CentOS
---

既然 Brotli 比 Gzip 效率高，那咱也来折腾一番，编译一个试试。

## 简介

> BBR 是 Google 提出的一种新型拥塞控制算法，可以使 Linux 服务器显著地提高吞吐量和减少 TCP 连接的延迟

BBR 解决了两个问题：

- 再有一定丢包率的网络链路上充分利用带宽，非常适合高延迟，高带宽的网络链路
- 降低网络链路上的buffer占用率，从而降低延迟，非常适合慢速接入网络的用户

项目地址 : [https://github.com/google/bbr](https://github.com/google/bbr)

![Google-BBR](/IMAGES/如何开启-Google-BBR-加速/Google-BBR.webp)

## 升级内核

开启 BBR 要求 4.10 以上版本 Linux 内核，可使用如下命令查看当前内核版本：

```bash
uname -r
#输出应类似
3.10.0-957.el7.x86_64
```

如果当前内核版本低于 4.10，可使用 `ELRepo` 源更新：

```bash
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
sudo yum --enablerepo=elrepo-kernel install kernel-ml -y
```

安装完成后，查看已安装的内核：

```bash
rpm -qa | grep kernel
#输出应类似：
kernel-ml-5.1.1-1.el7.elrepo.x86_64
#当前内核是5.1
kernel-tools-3.10.0-957.el7.x86_64
kernel-headers-3.10.0-957.12.1.el7.x86_64
kernel-tools-libs-3.10.0-957.el7.x86_64
kernel-3.10.0-957.el7.x86_64
```

## 修改 grub2 引导

执行：

```bash
sudo egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
```

结果应类似：

```bash
CentOS Linux (5.1.1-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-95abd3179adc4a7dbe7deb570bded11f) 7 (Core)
```

编号从0开始，所以设置默认启动项为1并重启系统：

```bash
sudo grub2-set-default 1
reboot
```

重启完成后，重新登录并重新运行 `uname` 命令来确认你是否使用了正确的内核：

```bash
uname -r
#输出应类似：
5.1.1-1.el7.elrepo.x86_64
```

## 开启 BBR

执行：

```bash
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

完成后，分别执行如下命令来检查 BBR 是否开启成功：

```bash
sudo sysctl net.ipv4.tcp_available_congestion_control
#输出应类似:
net.ipv4.tcp_available_congestion_control = reno cubic bbr

sudo sysctl -n net.ipv4.tcp_congestion_control
# 输出应类似：
bbr

lsmod | grep bbr
# 输出应类似：
tcp_bbr  16384  28
```

## 速度测试

```bash
#需先在 firewalld 中开启 http 服务
sudo dd if=/dev/imoe of=500mb.zip bs=1024k count=500
```
