---
title: adb-pull-输出信息为什么跑到了stderr
date: 2025-04-21 11:35:00
category: 学习笔记
tags:
  - Android
  - adb
  - CSharp
  - 编程小坑
---

今天用 C# 调 adb 执行 `adb pull`，导出功能正常，文件也拉下来了，但是走的是报错流程。

然后开 Debug 打断点看了，

发现问题出在输出：全打在 **stderr**，不是 **stdout**。

查阅相关资料，有人给谷歌提了 issue ，谷歌回复：不是 bug，是设计。

[提交记录在这](https://android-review.googlesource.com/c/platform/packages/modules/adb/+/3019947)：

> LinePrinter 改为输出到 stderr，因为这些信息是“给用户看的”。

也就是说，进度条、提示、非错误信息——全算“用户消息”，所以扔 stderr。

Google 觉得 stdout 应该只交付“程序该处理的结果”，其余的，全让人类去看。

符合他们一贯的风格：你不需要知道为什么，只需要知道它现在就这么做了。
