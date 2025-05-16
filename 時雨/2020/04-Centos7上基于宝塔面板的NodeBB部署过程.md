---
title: Centos7上基于宝塔面板的NodeBB部署过程
date: 2020-06-02 20:00:00
category: 折腾笔记
tags:
  - CentOS
---

偶然想着以后可能会需要一个论坛和小伙伴们交流，尝试了 NodeBB 和 Flarum， Flarum 我半个小时就搭好了，这 NodeBB 的部署坑挺多的，所以就写个文章记一下坑，怕自己忘了。

## NodeBB 简介

Github：[https://github.com/NodeBB/NodeBB](https://github.com/NodeBB/NodeBB)

> NodeBB 是 Design Create Play 开发的一款使用 Node.js 构建的论坛系统，使用 redis 或 mongoDB 数据库，采用 web socket 技术实现。支持响应式布局，兼容IE8，将论坛的体验带向一个新的高度。

国外的论坛系统都相当纯粹，并不会像 `Discuz` 、`phpwind` 那样提供各种建站所需模块。

## 开始折腾

直接上SSH

```bash
yum -y update

yum -y install epel-release

yum -y groupinstall "Development Tools"
#这里不用装Redis和npm因为可以直接在宝塔上装
yum -y install git ImageMagick

nvm list-remote
#直接装最新的，v12也行
nvm install v13.7.0
#使用cnpm代替npm
npm install -g cnpm
#下载
cd /path/to/nodebb/install/location
git clone https://github.com/NodeBB/NodeBB.git
#安装依赖
cd NodeBB
cnpm install
#初始化
./nodebb setup
```

这个时候可以进行简单的设置了，他会提示你：

```bash
Welcome to NodeBB v1.13.1!

This looks like a new installation, so you'll have to answer a few questions about your environment before we can proceed.
Press enter to accept the default setting (shown in brackets).
URL used to access this NodeBB (http://localhost:4567) 
Please enter a NodeBB secret (f560ba0d-1665-40a2-8703-43036d6f21fe) 
Would you like to submit anonymous plugin usage to nbbpm? (yes) 
Which database to use (mongo) redis
```

到这一步的时候，看你用 `Mongodb` 还是 `Redis` ，我这里是 `Redis`
接下来看个人设置了，一般回车就行，然后设置完了他就会开始构建前端这些文件

等到出现

```bash
NodeBB Setup Completed. Run "./nodebb start" to manually start your NodeBB server.

Starting NodeBB
  "./nodebb stop" to stop the NodeBB server
  "./nodebb log" to view server output
  "./nodebb help" for more commands
```

就说明OK了

但是我们肯定不能用 `4567` 这个端口对不对，所以得用 `Nginx` 反代它

假设你已经在宝塔上解析好域名和配置好 `SSL`

接下来在宝塔的网站设置里面，添加反向代理，目标Url就填你的域名加端口 `4567`

点保存，然后关掉，点反代的配置文件，里面全部删掉，填这个：

```conf
location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;

        proxy_pass http://你的域名:4567;
        proxy_redirect off;

        # Socket.IO Support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```

别忘了改config.json里面的URL参数，一样的 [你的域名](http://你的域名:4567)

到这里差不多就大功告成了

```bash
./nodebb start
```

设置服务的话参考官方文档，建一个服务就好

[https://docs.nodebb.org/configuring/running/](https://docs.nodebb.org/configuring/running/)
