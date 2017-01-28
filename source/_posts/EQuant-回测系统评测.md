---
title: EQuant 回测系统评测
date: 2017-01-24 09:42:17
categories:
  - 设计模式
---

东方宽客(EQuant) 是永安期货开发的一款基于 C# 的量化交易系统。

EQuant 里回测策略的编写分为两个部分：策略定义、策略执行。

<!--more-->

## 策略定义

向 EQuant 声明策略的存在，提供策略的元信息

+ 策略基本信息 Basic Info.

  策略名、标题、描述、公司、分组等

+ 时间周期 Timeframe

  设定本策略可以接受的时间周期。

+ 图表相关 Graph

  + Graph 图表区 `GraphDefine`
  + Axis 坐标轴 `AxisDefine`
  + Series 序列 `SeriesDefine`
  + Graphics 图形 `GenericGraphics`
  + Graph Object 图形对象 `GraphObject`

  从层次与数量对应关系上看：Graph >= Axis >= Series = Graphics >= Graph Object

  + 一个策略仅**拥有**一个独立的图表区(Graph)；

  + 一个图表区**包含**若干坐标轴(Axis)；

    这些坐标轴的纵坐标（数值）是独立的，但横坐标（时间）是共享的。

  + 一个坐标轴**对应**一个子图区域(Sub Graph)；

  + 一个坐标轴**包含**若干序列(Series)，这些序列是**有序**的；

    先添加到轴里的序列先绘制，图形重叠的位置会被后添加的序列覆盖。

  + 一个序列**对应**一个图形(Graphics)，此处可以跨类**按名访问**；

    序列在**策略定义类**中定义，而图形在**策略执行器类**中定义。

  + 一个图形**包含**若干图形对象(Graph Object)，这些图形对象是**有序**的；

    同上，先添加到图形的对象会被后添加的对象覆盖。

  > 出于隔离实现的原因，不同的轴不可以引用同一序列。

+ 策略参数 Parameter

  提供了一个简单的利用参数来定制策略的方式，可用于 EQuant 的自动参数计算与 GUI 上的参数配置操作。

  有一个抽象基类`ParameterDefine`定义了参数，目前定义了 5 个派生类来标注参数类型：`BoolParameterDefine, DoubleParameterDefine, EnumParameterDefine, IntParameterDefine, StringParameterDefine`，支持 `bool, double, enum, int, string` 类型的参数。

  参数实例具有一个名字`Name`，同图形中的序列一样，可以在执行器中**按名访问**。

  参数实例具有一个标题`Title`，在 GUI 策略配置的时候会显示这个值。

  构造完参数实例以后，要将它添加到一个集合中：

  ```c#
  // 在 StrategyDefine 派生类的构造器中
  // 已构造好 ParameterDefine p;
  ParameterDefines.Add(p);
  ```

  这个时候会检查参数的名字，确保没有重复。

## 策略执行

策略定义中 `CreateExecuter` 方法直接返回一个策略执行器实例，至此开始策略的执行。

执行器继承抽象类 `EQuant.STG.StrategyExecuter` 以获得 EQuant 提供的**公有**上下文，并部分重写事件的处理器来“订阅”由EQuant中发生的相应事件：

> 说是“订阅”是因为它们本质上并不是发送者-订阅者的事件模型。而是通过反射然后重写事件处理函数的一个机制。

+ `OnInit();`
+ `OnSetConfig(StrategyConfig Config);`
+ `OnProgressStarted();`
  + `OnDrawBegan();`
  + `OnBeginSegment();`
    + `OnDayBegining();`
    + `OnDayBegan();`
    + `OnDayEnding();`
    + `OnDayEnded();`
  + `OnEndSegment();`
  + `OnDrawEnded();`
+ `OnProgressEnded();`
+ `OnDispose();`

实际上在执行器的整个生命周期都不会有图形界面的显现。

在执行器的构造函数中甚至没有任何初始化数据，都是 null，什么都拿不到，直到 `OnInit` 被调前，所有的**公共上下文**才被 EQuant **注入**。

奇怪的是，EQuant 的事件系统既包括这种反射的机制，又有发送-订阅机制。我想这应该是迭代的过程中发现发送-订阅模型更具有拓展性。

`ContractExecuter` 中有 3 个 event ：`OnBar, OnBarTick, OnTick`，这些都是数据的来源。其中`OnBarTick` 在回测模式中不触发，因此不讨论。

以上所有的数据来源都已说明完毕。

