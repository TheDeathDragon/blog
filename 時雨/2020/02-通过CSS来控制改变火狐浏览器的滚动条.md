---
title: 通过CSS的控制来改变火狐的滚动条
date: 2020-04-16 20:00:00
category: 学习笔记
tags:
  - CSS
  - 浏览器
---

之前只看到过别人改过 Chrome 的滚动条，但是突然有一天看到别人博客，火狐浏览器也改了，然后就查了一下，在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/) 找到了我想要的答案。

## 定义

先来看看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/) 是怎么说的：

> 这是一个实验中的功能  
> 此功能某些浏览器尚在开发中，请参考浏览器兼容性表格以得到在不同浏览器中适合使用的前缀。由于该功能对应的标准文档可能被重新修订，所以在未来版本的浏览器中该功能的语法和行为可能随之改变。

**scrollbar-color** 该 CSS 属性设置滚动条轨道和拇指的颜色

**track**（轨道）是指滚动条，其一般是固定的而不管滚动位置的背景。

**thumb**（拇指）是指滚动条通常漂浮在轨道的顶部上的移动部分。

## 语法

```css
/* Keyword values */
scrollbar-color: auto;
scrollbar-color: dark;
scrollbar-color: light;

/*  values */
scrollbar-color: rebeccapurple green;   /* Two valid colors. 
The first applies to the thumb of the scrollbar, the second to the track. */

/* Global values */
scrollbar-color: inherit;
scrollbar-color: initial;
scrollbar-color: unset;
```

## 例子

```css
.scroller {
  width: 300px;
  height: 100px;
  overflow-y: scroll;
  scrollbar-color: rebeccapurple green;
}
```

## 浏览器兼容性

![火狐滚动条支持情况](/IMAGES/通过CSS来控制改变火狐浏览器的滚动条/火狐滚动条支持情况.webp)

## 参考链接

[https://developer.mozilla.org/zh-CN/docs/Web/CSS/scrollbar-color](https://developer.mozilla.org/zh-CN/docs/Web/CSS/scrollbar-color)
