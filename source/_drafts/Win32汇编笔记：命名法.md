---
title: Win32汇编笔记：变量命名法
date: 2017-01-05 20:58:08
categories:
  - 笔记
tags:
  - Win32汇编
---

记录一下 Win32 汇编中惯用的变量命名法。

变量使用**匈牙利命名法**。

+ 通过变量的名称约定/了解变量的类型
+ 区分不同类型的相同名称的变量/标识符

例如 `szHello` 中的 `sz` 表示这是一个以 0 结尾的字符串(**S**tring with **Z**ero-terminated)。

> 现代编辑器具有强大的 Intellisense 工具可以匹配类型，因此使用匈牙利命名法的人越来越少。
>
> 但是对于汇编这种难以确定类型的语言来说，用这种命名法还是颇具好处的。

<!--more-->

表示内容：

+ `sz` ：**S**tring with **Z**ero-terminated
+ `st`：**St**ruct
+ `fn`：**F**unctio**n**

表示指针：

+ `p`：**P**ointer
+ `h`：**H**andle (Similar to Pointer)

表示变量大小：

+ `w`：Word (16 bits)
+ `l`：Long (32 bits)

特殊的记号：

+ `@`：Local Variable Annotation

不同类型的记号可以组合使用，例如`lpst` 表示指向结构体的长指针。
