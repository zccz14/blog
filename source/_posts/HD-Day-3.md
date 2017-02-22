---
title: HD Day 3
date: 2017-02-22
categories:
  - 生活
tags:
  - HD
---

实训第三天，学习 Hibernate 框架。

<!--more-->

Hibernate 是一个对象关系映射（ORM: Object-Relation Mapping）框架，用于**对象持久化**。对象是指 Java 对象，而关系是指关系型数据库中的关系。数据存入数据库就意味着**数据持久化**，非易失。

> ORM 不支持 NoSQL 数据库，因为 NoSQL 中根本就没有关系，谈何 ORM。在基于文档(Document)的 NoSQL 中，通常使用 ODM (Object-Document Mapping)。

如果你曾经用 node.js 配合 MongoDB 开发过应用，应该听说过著名的 mongoose 库，一个 ODM 库。

ORM 也好，ODM 也好，都是从内存空间中的对象映射到数据库中的对象，这就是一个适配器模式的应用。在适配器模式下，我们可以转换接口，因而就能抽象出一个**数据持久层**，与具体的数据库层和业务逻辑层解耦。

Hibernate 支持很多关系型数据库，包括 MySQL, SQL Server 等。不同的数据库自然是有不一样的连接器(connector)，或者叫驱动(Driver)，使用哪个数据库就要导入哪个数据库的驱动。

Hibernate 中有几个命题成立：

+ Hibernate 中可以配置若干个 SessionFactory。

+ SessionFactory 用于获取 Session，可以获取若干个 Session。

+ 一个 SessionFactory 对应一个数据库连接配置。

  包括 数据库类型、URL、授权等。

+ 一个 Session 对应一组单线程数据库查询 (Query)。

  Session 实例是线程不安全的，因此不应有多个线程使用同一个 Session 实例。

+ Hibernate 提供了一个跨数据库通用的普通话（HQL）来使得查询跨平台。

  但我认为这功能十分鸡肋，直接使用 SQL 基本规范即可。

+ Hibernate 提供了一个缓存机制，并建立了一个 *Transient - Persistent - Detached* 三态的缓存协议，一定程度上降低了数据库 I/O 压力，提高了性能。

  这个套路实际上与 CPU Cache 的套路很相似，设计思想是一致的。

Hibernate 的配置分为三个部分：

+ Hibernate 自身属性 `hibernate.properties`

  主要包括 Hibernate 的各种运行配置，有一些开关选项。

+ Hibernate 配置 `hibernate.cfg.xml`

  主要包括 SessionFactory 的配置，连接数据库的配置。

+ Hibernate 映射配置 `*.hbm.xml`

  与一个 Java 类对应，即 ORM 的 Mapping 定义文件。定义了具体是如何从一个 Java 类对应到一个数据库关系的。

