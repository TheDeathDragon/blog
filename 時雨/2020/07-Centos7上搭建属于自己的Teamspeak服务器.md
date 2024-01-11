---
title: Centos7上搭建属于自己的Teamspeak服务器
date: 2020-07-16 20:00:00
category: 折腾笔记
tags:
  - CentOS
  - 服务器
---

我们平时玩游戏的时候，一般都会和朋友开语音，常用的QQ语音和YY都是有些地方不太方便的，比如说，声音大小不能单独调，降噪效果差等，而 teamspeak 就是相对比较完美的一款语音聊天软件。

## 前言

首先看一下百度百科对 Teamspeak 的介绍：

>Teamspeak（简称TS）是一款团队语音通讯工具，但比一般的通讯工具具有更多的功能而且使用方便。它由服务器端程序和客户端程序两部分组成，如果不是想自己架设TS服务器，只需下载客户端程序即可。teamspeak依靠先进的体系结构，方便灵活的应用功能，特别是领先的多媒体技术，为用户提供了一款强大的网络通讯工具。

Teamspeak 的官网 [https://www.teamspeak.com/en/downloads/](https://www.teamspeak.com/en/downloads/)

这篇文章简单讲述一下，在 Centos7 中如何搭建属于自己的 Teamspeak 服务器。

## 准备工作

首先，我们先做好前期的准备工作。你要准备的有：

- 心灵手巧有耐心的自己；
- 一台服务器，拥有 ~~公网IP~~ 开放端口，线路比较好的那种，也不能离你太远
- 去官网下载好teamspeak服务端文件：[https://www.teamspeak.com/en/downloads/](https://www.teamspeak.com/en/downloads/)；
- 在你自己电脑上下一个teamspeak客户端 (~~要不然搭来干嘛~~)；
- 能够远程链接你的服务器以及向服务器上传文件的工具，例如`putty`，`Xshell`，`winscp`或`MobaXterm`等；

## 国际惯例

按照国际惯例，装什么东西之前都应该：

```bash
yum update
```

## 处理防火墙

接下来开启防火墙端口：

```bash
systemctl start firewalld
firewall-cmd --zone=public --add-port=9987/udp --permanent
firewall-cmd --zone=public --add-port=10011/tcp --permanent
firewall-cmd --zone=public --add-port=30033/tcp --permanent
firewall-cmd --reload
```

Teamspeak 服务的运行是不能以 `root` 用户进行的，所以我们得先给它添加一个用户：

```bash
useradd teamspeak
passwd teamspeak
```

## 上传服务端文件

接下来，将之前准备好的服务端文件上传至服务器并解压，

因为我安装了宝塔面板，所以我喜欢把一些服务放在，

`/www/wwwroot/`

的下面。

## 新建 Teamspeak 用户

接下来，我们使用 `teamspeak` 用户来运行 `teamspeak` 的服务，因此我们还要给它这个 `teamspeak`用户设置权限：

```bash
chown -R teamspeak:teamspeak /www/wwwroot/teamspeak/
```

## 运行服务

接下来就是运行服务端文件了，首先切换到我们刚才新建的用户：

```bash
su - teamspeak
```

接下来进入服务端文件所在的目录：

```bash
cd /www/wwwroot/teamspeak
```

接下来我们得先同意teamspeak的用户协议：

```bash
touch .ts3server_license_accepted
```

在这之后，我们就可以运行teamspeak服务端了：

```bash
./ts3server_startscript.sh start
```

运行之后，你应该可以看到这样类似于这样的一串信息(密码改成#号了233)：

```bash
[teamspeak@imoe teamspeak]$ ./ts3server_startscript.sh start
Starting the TeamSpeak 3 server
TeamSpeak 3 server started, for details please view the log file
[teamspeak@imoe teamspeak]$ 
------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
               Server Query Admin Account created                 
         loginname= "serveradmin", password= "#######"
------------------------------------------------------------------


------------------------------------------------------------------
                      I M P O R T A N T                           
------------------------------------------------------------------
      ServerAdmin privilege key created, please use it to gain 
      serveradmin rights for your virtualserver. please
      also check the doc/privilegekey_guide.txt for details.

       token=######################################
```

这里你得把这一段中的 `token` 保存下来，然后按 `Ctrl+C` 先停止一下服务，

然后切换回 `root` 用户(要输密码！)：

```bash
su -
```

## 设置服务

然后我们需要让它在后台一直运行并且开机启动。

所以我们来建立一个服务去控制它。

我们先来新建一个服务文件，比如说：`ts3.service`，

一般来说都是用 `vim` 编辑，

```bash
vim /lib/systemd/system/ts3.service
```

因为我有宝塔我就直接新建一个文件来改了，当然你用 `nano` 编辑也随你。

配置文件的内容：

```yaml
[Unit]
Description=Teamspeak server
After=network.target
[Service]
WorkingDirectory=/home/teamspeak/teamspeak3
User=teamspeak
Group=teamspeak
Type=forking
ExecStart=/home/teamspeak/teamspeak3/ts3server_startscript.sh start inifile=ts3server.ini
ExecStop=/home/teamspeak/teamspeak3/ts3server_startscript.sh stop
PIDFile=/home/teamspeak/teamspeak3/ts3server.pid
RestartSec=15
Restart=always
[Install]
WantedBy=multi-user.target
```

需要注意的是，这里的 `WorkingDirectory` ， `ExecStart` ， `ExecStop` ，  `PIDFile` 这四个参数是你服务端文件的绝对路径，记得修改成你自己的路径。

之后保存退出即可

## 开机自启服务

在服务文件编辑完毕之后，我们就可以使用 `systemctl` 指令来启动 `teamspeak` 的服务并且让它开机自启了：

```bash
# 启动服务端
systemctl start ts3
# 关闭服务端
systemctl stop ts3
# 开机自启
systemctl enable ts3
# 查看服务端运行信息
systemctl status ts3
```

至此，服务端配置完毕，开始运行。

## 连接服务器

装完服务之后肯定是要来用呀，找出刚才我们保存的 `token`，然后打开客户端连接服务器，连上了它默认会问你要 `token`，输入了你就是管理员了。

管理员可以通过客户端对服务器进行设置，具体自己看着改吧。

## 解析域名到服务器

因为服务器的 IP 地址不好记啊，所以我们可以用域名来连接。

直接解析一个 A 记录到服务器就行了，之后连接的时候填域名多方便。