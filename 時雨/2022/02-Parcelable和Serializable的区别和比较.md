---
title: Parcelable和Serializable的区别和比较
date: 2022-11-15 17:36:56
category: 学习笔记
tags:
  - Android
  - 编程
---

先前学习了 Fragment ，多个 Fragment 传递数据的话，如果有很复杂的对象的时候就不方便用 `Intent` 的 `putExtra` 了，这个时候用 `Parcelable` 将对象进行序列化就方便很多。

## Parcelable 概要

`Parcelable` 和 `Serializable` 都是实现序列化并且都可以用于 `Intent` 间传递数据，

`Serializable` 是 Java 的实现方式，可能会频繁的IO操作，所以消耗比较大，但是实现方式简单。

`Parcelable` 是 `Android` 提供的方式，效率比较高，但是实现起来复杂一些 ， 

二者的选取规则是：

内存序列化上选择 `Parcelable` ，

存储到设备或者网络传输上选择 `Serializable`（当然`Parcelable` 也可以但是稍显复杂）

选择序列化方法的原则

1、 在使用内存的时候，`Parcelable` 比 `Serializable` 性能高，所以推荐使用 `Parcelable`。

2、`Serializable` 在序列化的时候会产生大量的临时变量，从而引起频繁的GC。

3、`Parcelable` 不能使用在要将数据存储在磁盘上的情况，因为 `Parcelable` 不能很好的保证数据的持续性在外界有变化的情况下。尽管 `Serializable` 效率低点，但此时还是建议使用`Serializable`。

下面介绍下 `Parcelable` 和 `Serializable` 的作用、效率、区别及选择。

## 作用及区别

`Serializable` 的作用是为了保存对象的属性到本地文件、数据库、网络流以方便数据传输，

当然这种传输可以是程序内的也可以是两个程序间的。

而Android的 `Parcelable` 的设计初衷是因为 `Serializable` 效率过慢，

为了在程序内不同组件间以及不同 Android 程序间 (AIDL) 高效的传输数据而设计，

这些数据仅在内存中存在，`Parcelable`是通过 `IBinder` 通信的消息的载体。

从上面的设计上我们就可以看出优劣了。

## 效率及选择

`Parcelable` 的性能比 `Serializable` 好，在内存开销方面较小，所以在内存间数据传输时推荐使用 `Parcelable`，

如 `activity` 间传输数据，而 `Serializable` 可将数据持久化方便保存，所以在需要保存或网络传输数据时选择 `Serializable`，

因为 Android 不同版本 `Parcelae` 可能不同，所以不推荐使用 `Parcelable` 进行数据持久化。

## 编程实现上

对于 `Serializable`，类只需要实现 `Serializable` 接口，

并提供一个序列化版本`id(serialVersionUID)` 即可。

`Parcelable` 则需要实现 `writeToParcel`、`describeContents` 函数以及静态的 `CREATOR` 变量，

实际上就是将如何打包和解包的工作自己来定义，而序列化的这些操作完全由底层实现。

## 高级功能上

`Serializable`序列化不保存静态变量，

可以使用`Transient`关键字对部分字段不进行序列化，

也可以覆盖`writeObject`、`readObject`方法以实现序列化过程自定义。