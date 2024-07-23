---
title: Repo同步代码注意事项
date: 2023-04-23 16:23:00
category: 折腾笔记
tags:
  - Repo
  - 代码管理
---

手上有个闲置的一加7，想自己编译第三方 Rom 进行刷机，所以尝试自己拉代码自己编译，但是Repo拉代码确实有点曲折，所以记录一下过程。

## 初始化

需要设置 python 为 python3 ，而且版本不能低于 3.6.4

```bash
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

## 拉代码

repo 仓库都有一个对应的 manifest 需要在 repo 初始化的时候进行下载，然后 sync

```bash
repo init -u https://github.com/Evolution-X/manifest -b udc
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

## 基本指令

```bash
# 同步代码 
repo sync

# 相当于给每个仓库 git status
repo status

# 给所有仓库执行 git log --oneline -1 
# 其中 p 是打印仓库的路径
repo forall -cp "git log --oneline -1"

```

## 同步遇到的问题

`fatal: this operation must be run in a work tree`

这种情况一般是网络问题导致sync中断出现的

找到对应的仓库目录

比如说 `packages/apps/DocumentsUI` 文件夹，和 `.repo/project/packages/apps/DocumentsUI`

以及 `.repo/project-objects/packages/apps/DocumentsUI`

删掉之后重新 sync 即可