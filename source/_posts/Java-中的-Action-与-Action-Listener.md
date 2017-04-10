---
title: Java 中的 Action 与 Action Listener
date: 2017-04-08
categories:
  - 系统分析
tags:
  - Java
---

>  一般地，面向对象分析与设计中存在三种事情处理的机制，除了普通的方法调用外，常常用到回调函数，而 J2EE 中还提供了一种基于监听方式的事件处理机制，请查阅资料，对 `Action` 以及 `ActionListener` 的机制进行分析，完成一个分析实例。

`Action` 与 `ActionListener` 都是 `javax.swing` 包中的**接口**。`javax.swing` 这个包的主要功能是关于 GUI 的。自然而然地，`Action` 与 `ActionListener` 都是与 GUI 相关的东西。

要理解 `Action` 与 `ActionListener` 的原理要从 `ActionListener` 开始，这很奇怪，但是事实——`Action` 继承了 `ActionListener` 。这意味着一个 `Action` 也是一个 `ActionListener` ？怎么理解？

![Action & ActionListener Hierarchy](https://zccz14.com/images/2017/04/11/1.svg)

<!--more-->

 `ActionListener` 接口中只声明了一个方法 `void actionPerformed(ActionEvent e)`，看下图中 `ActionListener` 与 `EventHandler` 都继承自一个平凡接口 `EventListener` ，并且都只声明了参数仅有一个 `EventObject` 的方法。

> `EventHandler` 是一个泛型事件处理器，T 是类型参数，有 `T extends Event (extends EventObject)` 的约束。

而且遍览所有继承 `EventListener` 的接口，都只声明了仅有一个以 EventObject 为下界类型的参数的方法。但可能会同时监听多种事件。

listener 与 handler 是相似的概念，而 actionPerformed 与 handle 也是相似的概念。

![All in one](https://zccz14.com/images/2017/04/11/2.svg)

这个 `ActionListener` 接口中声明了当 `ActionEvent` 出现后的处理函数。一切实现了 `ActionListener` 接口的类都要重写这个方法 `actionPerformed`。

## 发布自定义事件

根据上文描述，如果你想要发布一种事件，你需要：

1. 定义一个 `java.util.EventObject` 的派生类，假设叫 `SampleEventObject`；

2. 定义一个 `java.util.EventListener` 的派生接口，假设叫 `SampleEventListener`。

   1. `SampleEventListener` 接口要声明处理 `SampleEventObject` 的方法，例如 `void onSampleEvent(SampleEventObject e);`

   2. 直接从继承 `EventListener` 的接口派生新的接口也是可以的。

      这解释了文章开头的 Action 与 ActionListener 的关系—— **Action 是功能更强的 ActionListener，它包含了启动/禁用状态以及一个值的枚举状态**。

举个例子：

先定义一个 `SampleEventObject` 类。

```java
// SampleEventObject.java
import java.util.EventObject;

public class SampleEventObject extends EventObject {
    /**
     * Constructs a prototypical Event.
     *
     * @param source The object on which the Event initially occurred.
     * @throws IllegalArgumentException if source is null.
     */
    public SampleEventObject(Object source) {
        super(source);
    }
}
```

这个类是用于描述一类事件的，这个类的实例对应这类事件的实例，即“运行时发生的一个事件”。如果这类事件在不同情况下参数不同，可以在这个类中添加字段与方法或者属性来增强这个事件类的表达能力。

然后定义一个对应的监听器如下：

```java
// SampleEventListener.java
import java.util.EventListener;

public interface SampleEventListener extends EventListener {
  onSampleEvent(SampleEventObject e);
}
```

这样一来，我们就成功地发布了一类事件。

## 订阅事件

订阅一类事件分为两个阶段：**实现对应的监听器**、**绑定监听器的实例到产生事件的实例上**。

首先来实现一个监听器 `SampleEventLogger`，它监听 `SampleEvent` 并打印时间。

```java
// SampleEventLogger.java
import java.time.LocalDateTime;

public class SampleEventLogger implements SampleEventListener {
    @Override
    public void onSampleEvent(SampleEventObject e) {
        System.out.println(LocalDateTime.now() + ": Sample Event");
    }
}

```

万事俱备，只欠东风！我们上哪儿找我们要监听的事件？对于 `Action` 来说，从 `JComponent` 里找这个 `Action` 的生产容器，例如 `JButton` ——我们只要实例化一个 `JButton`，就能调用实例方法 `addActionListener` 然后将我们实例化后的 `ActionListener` 绑定到 `JButton` 的实例上。

但我们现在没有 `JButton` ，如何产生事件并将其传给 Listener？

```java
SampleEventLogger l = new SampleEventLogger();
// ...
// the event producer is an Object
SampleEventObject e = new SampleEventObject(new Object());
l.onSampleEvent(e);
```

啊哈！当然是手动调用了！当然，对于一个可拓展的事件发布者来说，一般要求监听器可以被动态加入/删除。

```java
// SampleEventPublisher.java
import java.util.HashSet;
import java.util.Set;

public class SampleEventPublisher {
    private Set<SampleEventListener> listeners = new HashSet<>();
    public void publish() {
        SampleEventObject e = new SampleEventObject();
        // Lambda for Java 8+
        listeners.forEach(l -> l.onSampleEvent(e));
    }
    public void addListener(SampleEventListener listener) {
        listeners.add(listener);
    }
    public void removeListener(SampleEventListener listener) {
        listeners.remove(listener);
    }
}


```

老哥，稳！现在已经有了事件的发布器，最后再给一个使用的例子来组合这一切。

```java
public class SampleEventTest {
    public static void main(String[] args) {
        // 1 publisher & 1 subscriber
        System.out.println("1 vs 1");
        SampleEventPublisher p = new SampleEventPublisher();
        SampleEventLogger l = new SampleEventLogger();
        p.addListener(l);
        p.publish(); // (1 line)
        // 1 publisher & multiple subscriber
        System.out.println("1 vs n");
        SampleEventListener l2 = new SampleEventLogger();
        p.addListener(l2);
        p.publish(); // (2 lines)
        // multiple publisher & multiple subscriber
        System.out.println("n vs m");
        SampleEventPublisher p2 = new SampleEventPublisher();
        p2.addListener(l);
        p2.addListener(l2);
        p.publish(); // (2 lines)
        p2.publish(); // (2 lines)
    }
}

```



## Listener 的副作用

Listener 接口所定义的方法（函数）一定不是一个纯函数，它带来了副作用(side-effect)，会对外部变量做修改，或者打印输出一些内容……

如果 Listener 访问了外部的变量作为状态使用，**并且这些状态只对单个 Listener 实例有效**，那么说明 Listener 是一个状态机。

为了提高内聚性，我们可以将这些状态封装到 Listener 类的内部，作为成员存在。这也是为什么这类 Listener 不能使用**单例模式**的原因。





## 参考资料

+ [How to Write an Action Listener - The Java Tutorials](https://docs.oracle.com/javase/tutorial/uiswing/events/actionlistener.html)


