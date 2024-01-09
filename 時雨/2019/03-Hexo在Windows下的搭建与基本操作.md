---
title: Hexo 在 Windows 下的搭建与基本操作
date: 2019-05-13 21:19:00
category: 技术向
tags:
  - Hexo
---

记录一下第一次使用静态生成式的博客框架。

## 前言

我们之所以选择 `Hexo` 来写博客是因为他的简洁高效，正如官方所说的一样：

> Hexo 是一个快速、简洁且高效的博客框架。`Hexo` 使用 `Markdown`（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页

但是安装过程不是想象中这么美好，按照目前官方提供的教程去安装，多半安装过程中会出问题

所以我写这篇文章的主要目的是为了让我在部署 `Hexo` 的时候少走弯路

## 准备工作

准备好 `Hexo` 运行所需的环境

- [Nodejs](https://nodejs.org/en/)
- [Git](https://git-scm.com/)

### 配置 Nodejs

进入官网下最新版还是稳定版都可以的，然后安装的时候保持默认的设置一路 `Next` 点下去就好

安装完成后我们应该验证一下各组件是否正常

`WIN+R` 打开运行，输入 `CMD` 回车（我感觉一般都会嘿嘿）接下来输入：

```bash
node -v

# 结果应类似：
v12.2.0
npm -v

# 结果应类似：
6.9.0
```

如果结果类似上面那么应该就已经成功安装好 `Nodejs` 了

顺便提一下，不用手打，你直接把代码复制然后按 `Shift+INSERT`

这样就可以把命令快捷输入到 `CMD` 里面

### 配置 Git

和安装 `Nodejs` 的时候一样，默认设置就可以了，自己可以微调

安装完成之后也检查一下：

```bash
git --version

# 结果应类似：
git version 2.21.0.windows.1
```

## 配置 Hexo

### 安装 Hexo

选一个你喜欢的地方作为你安装 `Hexo` 的文件夹

按住 `Shift+鼠标右键`，选择 **在此处打开 `PowerShell` 窗口**

下文中这个操作就简称为打开命令行

然后在命令行里面输入：

```bash
npm install hexo-cli -g

#输出结果应类似：
C:\Users\Hexo\AppData\Roaming\npm\hexo -> C:\Users\Hexo\AppData\Roaming\npm\node_modules\hexo-cli\bin\hexo
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\hexo-cli\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo-cli@1.1.0
added 225 packages from 434 contributors in 71.891s
# 可能你会和我一样看到WARN，但是这不影响
```

接着输入：

```bash
npm install hexo --save

# 输出结果应类似：
npm WARN deprecated core-js@1.2.7: core-js@<2.6.5 is no longer maintained. Please, upgrade to core-js@3 or at least to actual version of core-js@2.
npm WARN saveError ENOENT: no such file or directory, open 'C:\Users\Hexo\Desktop\package.json'
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN enoent ENOENT: no such file or directory, open 'C:\Users\Hexo\Desktop\package.json'
npm WARN Desktop No description
npm WARN Desktop No repository field.
npm WARN Desktop No README data
npm WARN Desktop No license field.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ hexo@3.8.0
added 360 packages from 475 contributors and audited 4536 packages in 75.733s
found 2 low severity vulnerabilities
  run `npm audit fix` to fix them, or `npm audit` for details
# 只要没ERROR就行
```

这个时候 `Hexo` 应该安装好了  
在命令行里面输入：

```bash
hexo -v

# 输出结果应类似：
hexo-cli: 1.1.0
os: Windows_NT 10.0.18362 win32 x64
node: 12.2.0
v8: 7.4.288.21-node.17
uv: 1.28.0
zlib: 1.2.11
brotli: 1.0.7
ares: 1.15.0
modules: 72
nghttp2: 1.38.0
napi: 4
llhttp: 1.1.3
http_parser: 2.8.0
openssl: 1.1.1b
cldr: 35.1
icu: 64.2
tz: 2019a
unicode: 12.1
```

如果你的结果和我的大致一样，那么恭喜你已经安装成功了
### 初始化 Hexo

接着刚才的命令行，输入：

```bash
hexo init

# 输出应类似于：
INFO  Start blogging with Hexo!
```

这个时候 `Hexo` 基本部署好了，但我们还需要补全一下组件，输入：

```bash
npm install

# 输出应类似于：
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

audited 4698 packages in 4.145s
found 3 vulnerabilities (2 low, 1 moderate)
  run `npm audit fix` to fix them, or `npm audit` for details
```

至此，`Hexo` 已经初始化完成
## 使用 Hexo

`Hexo` 是解析文章然后生成静态的网页，一般的流程是：  
1. 新建文章  
2. 生成页面  
3. 本地预览  
4. 发布生成的页面到服务器
### 基本命令

以下所有命令均可用小字母缩写来代替，本篇文章在此之后皆以此为准

```bash
# 基本命令如下：

clean
# 清除生成的文件
config
# 配置Hexo的设置
deploy
# 发布到服务器
generate
# 生成页面
help
# Hexo的帮助
init
# 新建一个新的Hexo博客
list
# 列出博客基本信息
migrate
# 从其他博客迁移内容过来
new
# 新建一篇文章
publish
# 发表草稿
render
# 渲染文件
server
# 启动本地调试服务器
version
# 显示版本信息
```

### 自己的第一篇文章：Hello Hexo

命令行输入：

```bash
# 新建文章
hexo n "Hello Hexo"
# 生成
hexo g
# 运行服务
hexo s
```

当服务运行之后，打开浏览器输入且回车：  

[http://localhost:4000](http://localhost:4000)  

至此，我们已经基本掌握了 `Hexo` 的流程了