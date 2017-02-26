---
title: 使用 Maven 管理项目依赖
date: 2017-02-26
categories:
  - 技术
tags:
  - Maven
  - Java
  - HD
---

Maven 是一个 Java 项目中依赖管理与自动构建工具。

说白了，就是可以一键导入 jar 包，除此之外还能够一键构建你的项目的可执行文件的一个工具。

Maven 解决的问题是：

+ 人工管理项目所依赖的 jar 包实在是太麻烦了。

  你需要经常性地拷贝 jar 包，除此之外还有可能因为疏忽造成版本冲突。

+ 一个项目在本机配置运行正常后无法在其他机器上自动配置运行。

  > 我机器上是可以跑起来的，为什么拷贝项目到你的机器上就跑不起来呢？

  这说明配置者不能正确地识别“项目的配置”与"依赖环境的配置"。

> Maven in Java is similar to NPM in JavaScript.

简单写个使用 Maven 的 Quick Start 教程来帮助有需要的人脱离依赖配置的苦海。

*本文只讨论 maven 的一部分依赖管理功能。*

<!--more-->

# 安装

> 如果你使用 Intellij IDEA，应该是自带 maven 的，可以直接跳过安装。
>
> 当然 IDEA 也支持你安装一个 maven 然后指定 IDEA 不要用内置的 maven 而使用你安装的 maven。

首先你需要安装 maven。

去 [maven 的官网 https://maven.apache.org/](https://maven.apache.org/) 下载 maven 安装包。

跟某些绿色软件一样，maven 解压即用，它提供了一个命令行工具 `mvn`。

根据 maven 官网的[安装教程](https://maven.apache.org/install.html)，你应该确保 `JAVA_HOME` 环境变量正确地指向你的JDK 的目录。

**为了方便地使用命令行工具，你应该将解压后的执行文件目录(bin)添加到环境变量 PATH 中。**

在命令行中测试命令

```bash
$ mvn -v
```

查看 maven 的版本信息，应该跟下面这段类似：

```
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: C:\Program Files\apache-maven-3.3.9\bin\..
Java version: 1.8.0_111, vendor: Oracle Corporation
Java home: C:\Program Files\Java\jdk1.8.0_111\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "dos"
```

如果成功了就算是安装 maven 成功了。

# 使用

使用 maven 创建一个项目的方法有很多：

+ 直接使用 `mvn ...` 命令创建。
+ 使用 IDE 支持的 maven 项目初始化。
+ 直接创建 `pom.xml`

其实不管你如何创建一个 maven 管理的项目，这一动作的最终结果就是产生了一个 `pom.xml` ，用什么方式创建并无实质区别。

> POM: Project Object Model 项目对象模型，是 Maven 对“项目”的模型。
>
> `pom.xml` 中包含了项目的名称、标识、版本、依赖包等等项目的基本配置。之后可以根据这个配置直接自动导入 jar 包，并且根据配置自动构建项目。

既然是怎么创建 `pom.xml` 是无所谓的，那么你现在可以直接新建一个 `pom.xml`，并粘贴如下代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.function-x</groupId>
    <artifactId>test</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <org.springframework.version>4.3.6.RELEASE</org.springframework.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.40</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${org.springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${org.springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${org.springframework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>5.2.8.Final</version>
        </dependency>
    </dependencies>
</project>
```

仔细看一看这就是一个 Spring + Spring MVC + Hibernate 的配置文件。

那么接下来运行一个命令就可以自动导入包：

```bash
$ mvn install
```

> 在 Intellij IDEA 中，检测到 `pom.xml` 后会询问你是否设置自动导入(Enable Auto-Import)，选了就可以自动导 jar 包了。