---
title: 量化系统的技术指标
date: "2017-02-03"
path: "/technical-indicators-in-quant-system"
tags: ['Design']
---

> 技术指标泛指一切通过数学公式计算得出的源于**市场数据**的**数据集合**。

这表明技术指标(Technical Indicator)最终都来源于市场数据；技术指标所谓的技术(Technique)就是指数学公式；技术指标的表现方式是数据的集合。

市场数据实际上基本可以表述为市场价格的**集合**。

从单笔交易这个微观的层面上看就是成交的价格与成交的量，这是一个瞬时值；

从一段时间上又可以将复数的交易信息合并起来，形成开、高、低、收四种价格以及总成交量。

整合这些信息，通过数学公式可以得到一个新的数据集合，称为**技术指标**。

<!--more-->

# 设计问题

最初，技术指标的设计实现都是相当简单的，通常是写一个函数便可。

例如森破移动平均线(Simple Moving Average)，通常只实现一个函数，然后将市场数据与时间周期参数传入：

```c#
public class SimpleMA {
  public static double calc(IReadOnlyList<double> data, int period) {
    return data.AsEnumerable().Reverse().Take(period).Average();
  }
}
```

> 以上实现有严重的性能缺陷，并不建议使用。
>
> 原因在于 IEnumerable 的 Reverse 方法对于 List 的随机访问没有做优化。

这样去实现一个指标会有若干问题：

+ 每一次引用指标都要重新做计算，计算性能不足。

  分析：可以用**缓存指标值**的方法来解决。因为技术指标是通过数学公式得出的，因此具有纯函数的性质，对于相同的输入，其输出一定相同——这就特别适合缓存了。

+ 缓存指标值由谁管理？

  分析：有如下设计方案

  1. 保持技术指标的纯度，在技术指标的外部保存缓存，而保持技术指标内部无状态。

     优点是正确性可以通过数学分析来保证。

     缺点是内部状态缺失会导致对外的状态依赖增多。（引用指标将变得很繁琐）

  2. 在指标内部维护状态，暴露必要的缓存的只读引用。

     优点是引用指标简洁明快，指标计算效率高。

     缺点是不同的指标性质不一，导致指标接口高度抽象。

  总体上来说，**在指标内部维护状态**是一个好的设计，隐藏指标的实现会降低系统的耦合度，提高指标的内聚性。

+ 指标需要维护哪些状态？

  分析：首先考虑指标的在线计算，要在正确的前提下，尽量减少指标的计算量。其次考虑到外部对指标历史值的引用，要暴露一个对历史数据的只读引用。

  + 指标内部需要维护图形吗？

    分析：不需要。因为技术指标是数据集合，与其表现方式无关。尽管每种指标都有其特定的惯用图形表示，但实际上图形与图形容器是强关联。

+ 如何编写指标能使之适应不同周期的数据？

  分析：订阅不同周期的外部数据事件即可，总体上是一个**观察者模式**。若外部环境没有提供合适的数据，可以利用**适配器模式**来构造合适的数据。

# 解决方案

利用泛型接口来定义技术指标类：

```c#
public interface ITechnicalIndicator<I, O> {
  void OnInput (object sender, I data);
  event EventHandler<O> OnOutput;
  IReadOnlyList<O> History { get; }
}
```

现在让我们来造一个简单平均线指标：

```c#
using System;
using System.Collections.Generic;
using System.Linq;

namespace TechnicalIndicators {
    public class SimpleMovingAverage : ITechnicalIndicator<double, double> {
        public SimpleMovingAverage(int period) {
            if (period < 1) {
                throw new Exception("均线周期不能小于 1");
            }
            ArgPeriod = period;
        }
        public IReadOnlyList<double> History {
            get {
                return bufHistory;
            }
        }

        public event EventHandler<double> OnOutput;

        public void OnInput (object sender, double data) {
            bufData.Add(data);
            double sum = 0;
            int total = Math.Min(bufData.Count, ArgPeriod);
            for (int i = 0; i < total; i++) {
                sum += bufData[bufData.Count - i - 1];
            }
            bufHistory.Add(sum / total);
            OnOutput?.Invoke(this, bufHistory.Last());
        }
        private List<double> bufData = new List<double>();
        private List<double> bufHistory = new List<double>();
        private int ArgPeriod;
    }
}
```

可以看到这个指标并不依赖于其他第三方库。

在其他地方如何引用这个指标呢？只需要简单的按参数构造指标实例、订阅数据源即可。发布指标数据是一个可选事件（即便不发布，指标值也会通过`History`属性暴露出来），通常可以用来绘图或者进行下一步的操作。

```c#
using TechnicalIndicators;
// 构造 20 均线
SimpleMovingAverage iSMA = new SimpleMovingAverage(20);
// 订阅数据源 (object sender, double data)
SomeEvent += iSMA.OnInput;
// 发布指标数据
iSMA.OnOutput += (sender, data) => {
  // data is double
}
// 引用历史 20 均线数据
iSMA.History[0]; // 第一个均线数据
```

>  如果你发现没有合适的订阅数据源，可以运用适配器模式来创建一个数据源适配器。

你的均线基于什么数据完全取决于它订阅了什么，可以是不同周期的市场价格，也可以是其他技术指标。

## 优化的具体思路

上面关于简单均线指标的实现的效率并不很高。

**如果你想提升计算速度**，就运用迭代法实现指标。既然保存了数据源的历史数据备份，就可以再维护一份前缀和列表，然后直接在常数时间完成一次指标迭代。这样的做法需要比上述实现多 50 % 的内存消耗。

**如果你想降低内存消耗**，就思考哪些数据是没有必要维护的。我们发现超出均线周期的数据是没有用的，因此我们可以构造固定长度的滑窗(Slide Window)来维护数据源，总的来说内存消耗量比上述实现要低 50 %。

两种优化思路是不冲突的，只是都会增加一些代码量。

但无论如何，这对外部引用指标的系统是透明的！你无须更改引用指标的代码便可优化指标的性能。
