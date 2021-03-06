---
title: "分布式生成书"
path: "/distributed-spanning-book"
date: "2017-06-22"
tags: ['Design']
---

Distributed Spanning Book (DSB)，不行，看到这个缩略词我好想笑。

搞事情：把分布式的知识网络转化为树结构（生成树形图），进而就可以组织成一个书(手册)的形式。

## 动机 Motivation

之前搞了一下计算机网络的课程知识点的一个提纲，想用格式，又苦于没有很好很开放很方便的标准。

+ 用 Word 写：不是一个很开放的格式。
+ 用 HTML 写：太麻烦了，通常要配合模板降低单个资料的复杂度
+ 用 XML 写：真的挺好的，就是解析起来有点麻烦。
+ 用 JSON 写：对前端挺有亲和力，在 JSON 格式大行其道的时代……

所以还是用了 JSON 作为一个载体语言。

由于一个人写这样一个东西实在太累了，我就想，这个东西能不能合作呢？

合作了的情况，究竟是让一个人来整合(Centralized)呢，还是通过直接拉取资源(Decentralized / Distributed)呢？

思考了一下自己设计的知识节点(Node)的结构：

```json
{
    "name": "Sample Node",
    "desc": [
        "It's a sample node",
        "It has a list of description"
    ],
    "keys": [
        "example"
    ],
    "children": [
        {
            "name": "Sample Child 1",
            "desc": [
                "It also has a list of description",
                "%!@%@#^!@#$$"
            ]
        },
        {
            "name": "Sample Child 2",
            "desc": "It's a simplified description"
        },
        "Sample Child 3 (Simplified Node)"
    ]
}
```

每个 `{}` 包裹的部分是独立的呀！如果可以的话，是不是可以把这个东西“外包”出去，形成一个合作的形式呢？

## 挑战 Challenge

只要在 `{}` 中新增一个 `resolve` 字段，填入 URL，就可以通过网络获取一个 JSON 描述的 Node，然后挂载到这个节点上。

这样这个树形 JSON 就变成了一个网络图。

但这带来了一些问题

+ 循环引用
  
  如果 Node 间存在相互引用，不加控制的情况下，可能会引起无穷递归而崩溃。
  
+ 引用深度
  
  如果引用的 Node 本身是一个很大很宽的概念，可能为了充分解释这个概念所用的篇幅过长，所以要“点到为止”。
  
+ 多重引用
  
  一个 Node 可能被多个 Node 同时引用，会在最终的结果中产生冗余。

+ 部分引用
  
  如果只想引用一个 Node 中的一个部分，而不是全盘接受这个 Node，应该怎么说明？

另外还要考虑的是**知识产权**的问题：要在引用的节点上添加作者的信息与版权信息。