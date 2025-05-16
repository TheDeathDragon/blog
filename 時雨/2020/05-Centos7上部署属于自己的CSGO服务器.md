---
title: Centos7上部署属于自己的CSGO服务器
date: 2020-06-29 20:00:00
category: 折腾笔记
tags:
  - CentOS
  - CSGO
  - 游戏
---

CSGO 应该是我平时玩的最多的一个游戏了，所以自己摸索开了个服，于是有了这篇文章。

## 吐槽

开一个 CSGO 服说难也不难，主要是不好找服务商，国内服务商质量参差不齐，小商家不稳，大厂太贵，CSGO 又是一个吃 CPU 单核性能的怪兽

好了，吐槽完了该说正事了

## 开搞

首先你得有一台服务器对吧，`NAT` 也可以开，最好可以问一下商家有没有 `27015` 端口，让他直通给你。

假设我们已经准备好了一台服务器，那么接下来我们来部署它。

Centos7 下我个人习惯是用宝塔面板来管理服务器

所以我们先装个宝塔面板先

官网 ：[宝塔面板](https://www.bt.cn/)

然后安装完了面板，进去先改面板设置，怎么方便怎么来就是了，不过不推荐用默认的 `8888` 端口

然后如果改了面板端口记得去 `SSH` 把这个端口放行

这里我习惯直接关闭 Centos7 的防火墙

```bash
systemctl stop firewalld.service

systemctl disable firewalld.service
```

完事了去宝塔面板的安全里面把你 CSGO 服务器的端口先放行了，一般都是 `27015`

然后进入到软件商店页面，这里我们可以选择性的装一个 `MySQL` 数据库，之后可以用到，然后把 `PureFTPD` 也装了，方便传文件，其他就暂时不用了

之后你得有一个绑定了手机的 Steam 小号，用于生成 `GSLT` ，相当于开服的 `Key` ，为什么要用小号是因为以后加改皮肤模型的插件可能会被封

网址：[Steam服务器管理](https://steamcommunity.com/dev/managegameservers)

> GSLT creates a persistent token for a game server. This allows any users who added your server to favorites to join, even if you change your ip address. This is very useful if you change your server/hosting provider.

生成一个 `GSLT` 就行了，记一下等下要用

接下来的话，该来安装 CSGO 服务器了

一般我们都是用 `LinuxGSM` 来管理众多游戏服务器， CSGO 是其中一个

网址：[lgsm](https://linuxgsm.com/lgsm/csgoserver/)

直接上命令：

```bash
# 安装依赖

yum install epel-release

yum install mailx postfix curl wget tar bzip2 gzip unzip python binutils bc jq tmux glibc.i686 libstdc++ libstdc++.i686

yum install python3

yum install zlib.i686
```

这里 python3 先装了是因为后面脚本装不上不知道为什么，继续

```bash
# 创建csgoserver用户
adduser csgoserver
# 修改密码
passwd csgoserver
# 切换到csgoserver用户
su - csgoserver

# 下载安装脚本
wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh csgoserver
# 开始安装服务器
./csgoserver install
```

这个地方有个坑， GitHub 的仓库被墙了，得改一手 `Host`  
打开宝塔面板，`hosts` 在 `/etc` 目录下，在这最后加一行就行了

`199.232.4.133 raw.githubusercontent.com`

之后就等它慢慢下载了，下完了它会问你是否成功安装

然后要你输入 `GSTL` ，接下来还问你愿不愿意分享匿名数据，这个就看你自己了

等它装好了，先别急着开，我们先简单的配制一下服务器启动参数

我们打开宝塔进到 `/home/csgoserver/lgsm/config-lgsm/csgoserver` 目录下

有三个文件，我们只改 `csgoserver.cfg` 就行了

我的是这样配置的

```yml
gslt="你的GSLT"
defaultmap="de_mirage"
maxplayers="12"
tickrate="128"
ip="0.0.0.0"
port="27015"
clientport="27005"
sourcetvport="27020"

gametype="0"
gamemode="1"
mapgroup="mg_active"

fn_parms(){
parms="-game csgo -usercon -strictportbind -ip ${ip} -port ${port} +clientport ${clientport} +tv_port ${sourcetvport} +sv_setsteamaccount ${gslt} -tickrate ${tickrate} +map ${defaultmap} -maxplayers_override ${maxplayers} +mapgroup ${mapgroup} +game_type ${gametype} +game_mode ${gamemode} -nobots"
}
```

这个 `-nobots` 是不需要 `Bot` 的意思，看情况吧，另外双引号好像有没有都一样，我有点强迫症就都加算了

- 休闲模式 `+game_type 0 +game_mode 0`
- 竞技模式 `+game_type 0 +game_mode 1`
- 军备竞赛 `+game_type 1 +game_mode 0`
- 爆破模式 `+game_type 1 +game_mode 1`
- 死亡竞赛 `+game_type 1 +game_mode 2`

好，接下来我们该来改一下主配置文件了  

`/home/csgoserver/serverfiles/csgo/cfg/csgoserver.cfg`  

可以照着默认改，之后就自由发挥了

然后服务器插件肯定是少不了的，得去

[sourcemod](https://www.sourcemod.net)

以及

[metamodsource](https://www.metamodsource.net/)

安装前置

下载的时候选 `Stable Builds` 就是稳定版的意思就行了

之后再去

[sourcemod](https://www.sourcemod.net)

里面找插件，这个就得自己折腾了，坑太多不想写

另外如果你用宝塔面板上传文件的话，你得注意一下权限的问题，宝塔默认用户是 `www`

CSGO 服务器它要求是 `csgoserver` 用户，得注意一下

修改要用到的指令，自己看着改：

```bash
chown -R csgoserver:csgoserver /home/csgoserver/serverfiles/你的文件名.cfg
chmod 775 /home/csgoserver/serverfiles/你的文件名.cfg
```

接下来没什么好说的，开服！

```bash
./csgoserver
```

顺便说一下 LinuxGSM 的基础指令，记得要用 `csgoserver` 用户登录 SSH：

```bash
# 开服
./csgoserver start
# 关服
./csgoserver stop
# 重启
./csgoserver restart
# 查看控制台
./csgoserver console
# 更新服务器
./csgoserver update
# 强制更新
./csgoserver force-update
# 验证服务器完整性
./csgoserver validate
# 查看服务器详细信息
./csgoserver details
# Debug
./csgoserver debug
# 打包备份服务器
./csgoserver backup
```

## 后记

就写到这吧。

要是真想搞慢慢折腾吧，一时半会搞不好的，多看点论坛，花点心思。

最好是去社区群里多问一下。
