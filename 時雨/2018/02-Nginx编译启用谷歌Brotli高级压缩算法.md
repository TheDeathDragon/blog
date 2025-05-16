---
title: Nginx 编译启用谷歌 Brotli 高级压缩算法
date: 2018-10-06 21:37:00
category: 折腾笔记
tags:
  - Nginx
  - Linux
---

Google 认为互联网用户的时间是宝贵的，他们的时间不应该消耗在漫长的网页加载中，因此在 2015 年 9 月 Google 推出了无损压缩算法 Brotli。

## 简介

Brotli 通过变种的 LZ77 算法、Huffman 编码以及二阶文本建模等方式进行数据压缩，与其他压缩算法相比，它有着更高的压缩效率。

根据 Google 发布的研究报告，Brotli 压缩算法具有多个特点，最典型的是以下 3 个：

- 针对常见的 Web 资源内容，Brotli 的性能相比 Gzip 提高了 17-25%；
- 当 Brotli 压缩级别为 1 时，压缩率比 Gzip 压缩等级为 9（最高）时还要高；
- 在处理不同 HTML 文档时，Brotli 依然能够提供非常高的压缩率；

Brotli 如此高的压缩比率，得益于其使用一个预定义的字典，该字典包含超过 13000 个来自文本和 HTML 文档的大型语料库的常用字符串，预定义的算法可以提升较小文件的压缩密度，而压缩与解压缩速度则大致不变。

Brotli 凭借它优异的压缩性能迅速占领了市场。

从下图可以看到，除了 IE 和 Opera Mini 之外，几乎所有的主流浏览器都已支持 Brotli 算法，因此处于资源占用的考虑，比如说流量，建议启用：

![[IMAGES/Nginx-编译启用谷歌-Brotli-高级压缩算法/浏览器Brotli支持情况.webp]]

以下操作基于宝塔平台，同样适用于LNMP 等平台，区别于操作目录而已。

## 开始部署

```bash
# 进入宝塔的环境目录
cd /www/server
git clone https://github.com/bagder/libbrotli
cd libbrotli
./autogen.sh
./configure
make && make install

cd /www/server
# 下载brotli
git clone https://github.com/google/ngx_brotli.git
cd ngx_brotli
# 更新brotli
git submodule update --init

# 在nginx.sh的编译参数里面加上这个，大概是200多行的样子
# 路径/www/server/panel/install/nginx.sh
--add-module=/www/server/ngx_brotli

# 执行编译
sh /www/server/panel/install/nginx.sh install 1.17

```

## 启用配置

- 如果只在某个路径启用，那么在 `location` 段中添加；
- 如果某个站点启用，那么在 `server` 段中添加；
- 如果要全局启用，则在 `http` 段中添加；

```bash
brotli on;
brotli_comp_level 5;
brotli_buffers 16 8k;
brotli_min_length 51200;
brotli_types text/plain application/javascript application/x-javascript text/javascript text/css application/xml;
brotli_static always;
brotli_window 512k;
```

PS：后面发现宝塔更新了，可以直接在安装 Nginx 的时候编译模块。

![[IMAGES/Nginx-编译启用谷歌-Brotli-高级压缩算法/宝塔编译Nginx-Brotli.webp]]

## 验证

假设当我们已经全局启用了 Brotli 算法，

如果看到响应头包含了 `Content-Encoding: br` 字样，

br 是 Brotli 流压缩的内容编码类型，

就表示我们已经成功的启用了 Brotli 算法。

## 参考文章

- [《给Nginx添加谷歌Brotli压缩算法支持》](https://www.imydl.tech/lnmp/510.html)
- [《启用 Brotli 压缩算法，减少流量》](https://www.qtxh.net/2016/08/25/qi-yong-brotli-ya-suo-suan-fa-ti-gao-xing-neng/)
- [《Results of experimenting with Brotli for dynamic web content》](https://blog.cloudflare.com/results-experimenting-brotli/)
- [《GitHub/ngx_brotli readme》](https://github.com/google/ngx_brotli/blob/master/README.md)
- [《Wiki/Brotli》](https://zh.wikipedia.org/wiki/Brotli)
