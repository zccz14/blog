---
title: 永安实习 Day 6
date: 2017-01-18
categories:
  - 生活
tags:
  - C#
---

抽离技术指标类…更有趣的是发现了 Linq 中的一个坑方法——`Reverse`。

<!--more-->

原型如下：

```c#
public static IEnumerable<TSource> Reverse<TSource> (this IEnumerable<TSource> source);
```

之前觉得 IEnumerable 接口方法应该都是 lazy 操作吧……所以一直把它当常数方法使，类似这样：

```c#
List<double> a; // n = a.Count
double maxA = a.AsEnumerable().Reverse().Take(10).Max();
```

本以为这个**取最后10个求最大值**的操作是 O(1) 的，没想到用了这个 Reverse 以后给我搞成了 O(n)——它将 a 中的元素逐个推入栈中，然后取出来 `yield return` 之。

这导致我在 n 比较大的时候就开始爆炸，内存刷刷地上 GB，运行得很慢，Timeline 上遍布 GC 事件。

Reverse 的实现显然是没有为随机访问容器优化的，至少当前版本 .net framework 4.5.2 尚未优化，所以取最后 k 个数需要自己拓展实现一个 lazy 方法。

```c#
static class IListExtends {
  public static IEnumerable<T> Last<T> (this IReadOnlyList<T> list, int n) {
    int i = list.Count - 1;
    while (n > 0 && i >= 0) {
      yield return list[i];
      i--;
      n--;
    }
  }
}
```

> 逆序地取最后 n 个元素

然后就可以对 `List<T>` 调用这个方法了：

```c#
double maxA = a.AsReadOnly().Last(10).Max();
```

已测试，GC 没了，并且快了很多，有了明确的时间保证，很稳健。