---
title: HD Day 2
date: 2017-02-21
categories:
  - 生活
tags:
  - HD
---

实训第二天，尹老师讲 Spring 框架。

好久没写 Java 了，先搞一下 IDE。

> IDE: IntelliJ IDEA Ultimate 2016.3

之前用的 Community 被我换成了 Ultimate，它支持 JavaEE，这正是这次实训所需的。Ultimate 本来是收费的只有 30 天的 trial，但是我们学生可以申请 Education Pack 呀，所有 JetBrains 产品都免费使用，这可真是太棒了。

> JetBrains 是迄今为止最好的 IDE 产商。

利用 IDEA 导入 JAR (Java ARchive)是一件很方便的事。

首先你需要在 IDE 级别安装插件(plugin)，例如 Spring, Struts, Hibernate, JUnit 等等。IDEA 插件的功能强大，它**至少**可以帮助我们快速导入 JAR。

在Project名字上右键就有一个选项 **Add Framework Support...** 它可以快速添加一个框架所有所需的包，如果本地没有，还可以自动下载，十分方便。

然后就是 Spring 的一些笔记：

关于 Spring 依赖注入的基本用法，实际上就是通过 XML 配置对象，产生一个上下文，然后在其中按名存取对象。

Bean 实际上跟 java.lang.Object 是一个概念，就是一个对象。

所以我们得到的 Bean 形式上并没有什么可用的性质，但它是所有类的基类，可以也必须通过**强制类型转换**来访问目标派生类对象的成员。

按名存取使得编译器很难提前得知 XML 中的内容（因为它可能是运行时才产生的），再加上强制类型转换，这带来了一个运行时崩溃的风险。

Spring 更多地使用软的**约定**而不是硬的**接口**来规范开发者，这可能带来一些风险，并一定程度上增加了理解的难度。通过反射应该是可以避免运行时崩溃这种尴尬的事情的发生，所以这个 Spring 的水还是很深的。

我对 Java 还真是一无所知啊。