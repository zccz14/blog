---
title: Win32汇编笔记：结构体
date: 2017-01-05 19:55:51
categories:
  - 笔记
tags:
  - Win32汇编
---

记录一下 Win32 汇编中结构体的用法。

<!--more-->

# 结构体声明

先声明后定义，先看声明的语法：

```assembly
STNAME struct
	prop1 type init
	prop2 type init
	; ...
STNAME ends
```

> 实际上结构体的声明是把其属性的定义有 `struct... ends` 结构包裹了一下。

例如一个典型的二维点结构：

```assembly
Point struct
	x dword 0
	y dword 0
Point ends
```

>  可以看到 `x dword 0` 就是一个典型的变量定义。

这个时候 `Point` 就是一个合法的 `type` 标识了，因而可以嵌套声明：

```assembly
Vector struct
	first Point <>
	second Point <>
Vector ends
```

# 结构体定义

定义结构体实例：

```assembly
P1 Point <>; equivalent to <0, 0>
P2 Point <1>; equivalent to <1, 0>
V1 Vector <>; equivalent to <<0, 0>, <0, 0>>
V2 Vector <<1, 1>>; equivalent to <<1, 1>, <0, 0>>
V2 Vector <<1, 1>, <2, 3>>
```

`type <...>` 结构有些类似 OOP 中的构造函数，参数会按照声明时的顺序填入对应的位置。

如果有参数并没有实际填入，那么就会用声明时定义的初始值填入。

# 结构体访问

定义结构体：

```assembly
stVec1 Vector <<1, 2>, <3, 4>>
```

访问：

```assembly
; 直接按名访问
mov eax, stVec1.first.x; eax = 1
mov stVec1.second.y, eax; <<1, 2>, <3, 1>>
; 无类型指针 + 偏移访问
lea esi, stVec1
mov eax, [esi + Vector.second.x]; Vector.second.x == 8, eax = 3
; 有类型指针可以直接按名访问
assume esi: ptr Vector
mov eax, [esi].first.y; eax = 2
assume esi: nothing
```

