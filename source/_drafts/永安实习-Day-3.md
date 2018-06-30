---
title: 永安实习 Day 3
date: 2017-01-13
categories:
  - 生活
---

实习第三天，直接拿着门禁卡就进去了，感觉独立了许多。

我觉得 C# 这门语言真是好玩。

```c#
; lazy query max in a single statement.
double maxPrice = BarList.AsEnumerable().Reverse().Take(N).Select(v => v.ClosePrice).Max();
```

大致上与 JavaScript 一样用 Lambda 表达式，但 C# 原生支持 Lazy 操作，针对只读数组的遍历访问比 JavaScript 更好了一些。

不过这个语言特性我觉得 ECMAScript 早晚会抄过去。

现在有 immutable.js, lazy.js 等库能实现类似的 lazy 操作。

但原生的 JavaScript 目前像 C# 那样不拷贝的方法还没有。

啊下班回家以后才发现今天是星期五呢。