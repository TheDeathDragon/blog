---
title: 配置GPG签名来进行Git操作
date: 2024-07-23 13:30:55
category: 学习笔记
tags:
  - 代码管理
---

`GPG Key` 可以让我们的 `commit` 安全性更上一层，并且在 `Github` 中会显示 `Verified` 标签，所以今天也来配置一下并使用。

## 准备工作

安装 `GPG` 工具 : <https://www.gpg4win.org>

下载的时候会问要不要赞助，这边选择 0 美元就可以直接点击下载了，

安装的时候选默认选项即可。

## 创建密钥

安装好 `GPG` 后，启动 `Kleopatra` 点击文件，新建 `OpenPGP` 密钥对，用户名和邮件最好还是按照 Github 的来填写。

## 导出 GPG 公钥

找到我们刚才创建的密钥，右键导出，把导出的文件里面的内容复制，这个就是公钥，

到这一步之后，我们要导出并添加 `GPG` 公钥到 `GitHub`，

打开 `Github Settings`  网页 : <https://github.com/settings/keys>

找到 `GPG keys` ，新建一个粘贴进去即可。

## 配置 Git 使用 GPG 签名

打开终端，执行指令让 `Git` 和 `GPG` 关联起来，

`YOUR_KEY_ID` 可以在 `Github` 网页上看见，复制即可。

```bash
git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"

gpg --list-secret-keys --keyid-format LONG

git config --global user.signingkey YOUR_KEY_ID

git config --global commit.gpgSign true
```
