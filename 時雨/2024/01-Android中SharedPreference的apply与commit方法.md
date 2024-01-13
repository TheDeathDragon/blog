---
title: Android中SharedPreference的apply与commit方法
date: 2024-01-13 10:47:00
category: 学习笔记
tags:
  - Android
  - 编程
---

最近项目的需求是有一个设置界面切换一些东西，但是测试的时候发现居然没有马上生效，检查代码发现自己用的是 `sharedPreferences` 的 `apply()` 方法，改成 `commit()` 就好了，故记录一下。

## 官方说明

### SharedPreferences.commit()

> Commit your preferences changes back from this Editor to the SharedPreferences object it is editing. 
> This atomically performs the requested modifications, replacing whatever is currently in the SharedPreferences.
> Note that when two editors are modifying preferences at the same time, the last one to call commit wins.
> If you don’t care about the return value and you’re using this from your application’s main thread, consider using apply() instead.

### SharedPreferences.apply()

> Commit your preferences changes back from this Editor to the SharedPreferences object it is editing. 
> This atomically performs the requested modifications, replacing whatever is currently in the SharedPreferences.
> Note that when two editors are modifying preferences at the same time, the last one to call apply wins.
> Unlike commit(), which writes its preferences out to persistent storage synchronously, apply() commits its changes to the in-memory SharedPreferences immediately but starts an asynchronous commit to disk and you won’t be notified of any failures. 
> If another editor on this SharedPreferences does a regular commit() while a apply() is still outstanding, the commit() will block until all async commits are completed as well as the commit itself.
> As SharedPreferences instances are singletons within a process, it’s safe to replace any instance of commit() with apply() if you were already ignoring the return value.
> You don’t need to worry about Android component lifecycles and their interaction with apply() writing to disk. 
> The framework makes sure in-flight disk writes from apply() complete before switching states.

需要注意的是 `commit()` 方法是 Added in API level 1 的，也就是 sdk1 就已经存在了。而 `apply()` 方法是 Added in API level 9 的。

`commit()` 有返回值，成功返回 `true` ，失败返回 `false` 。而 `apply()` 没有返回值。

`commit()` 方法是同步提交到硬件磁盘，

因此，在多个并发的提交 commit 的时候，

他们会等待正在处理的 commit 保存到磁盘后在操作，从而降低了效率。

`apply()` 是将修改的数据提交到内存，而后异步真正的提交到硬件磁盘。

## 为什么建议使用 `apply()` 替代 `commit()` ?

因为 Android 的设计人员发现，开发人员对 `commit()` 的返回值不感兴趣，而且在数据并发处理时使用 `commit()` 要比 `apply()` 效率低，所以推荐使用 `apply()`。

## 什么时候用 `commit()` ?

当需要立即保存条目的时候，应该用 `commit()`，如果使用 `apply()`，由于 `apply()` 是先提交到内存，然后异步保存到文件，如果程序出现异常或者被划掉后台，可能就没有生效，所以应该用 `commit()`。