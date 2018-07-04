---
title: GraphQL & ASP.NET Core Tutorial
date: 2018-5-13
tags:
  - Tutorial
  - GraphQL
  - CSharp
---

GraphQL 是个好东西，它可以让你从后端繁复的组合逻辑中解放出来，只要构建一套 Type System，前端们就可以各取所需。它很大程度保持了后端代码的健壮性，并在应用设计的早期就可以避免 API 过度/过少设计。你只要定义“我们有什么资源”、“资源之间的关系是怎么样的”以及“资源是如何变化的”就可以了，这听起来很 RESTful，但它比 REST 风格又好在了“一次请求”上，REST 由于设计的原子性，会带来一个 "1 + N" 的网络请求问题，而 GraphQL 通过引入了一个 DSL，即 Query Language 来解决了这个痛点，同时又保持了 REST 原子设计的优雅性。善哉！

现在我手头有一个基于 .NET 的 C# 项目，现在要做一个 GUI ，嗯， .NET Framework 做 Desktop GUI 肯定没有 Web 技术方便啊。所以现在需要构建一个 API 服务器，当然还是用 ASP.NET Core 了，至少跨平台嘛，然后造一下 GUI。那么关键在于如何设计 API 了，我想最省力的方式还是 GraphQL 这种（主要之前有一个 GraphQL 的实践经验，确实好用 www）

<!--more-->

---

# GraphQL Install

它有一个基于 .NET Stardard 的跨平台 C# 版本

源码：[graphql-dotnet](https://github.com/graphql-dotnet/graphql-dotnet)
NuGet: [GraphQL](https://www.nuget.org/packages/GraphQL/)

> 使用 NuGet 获取 GraphQL 的时候，要注意使用最新的 current 版本，书写本文的时候是 2.0.0-alpha-899。
> 如果你找不到 Schema.For 静态方法的话，可能是安装了错误的版本。[graphql-dotnet/issue #460](https://github.com/graphql-dotnet/graphql-dotnet/issues/460)

是可以先搞一个 Console Application 来玩耍一下 GraphQL for C#，但我们先考虑一下 ASP.NET Core 上的安装配置。

首先创建一个 ASP.NET Core 空项目，从一个最简的配置开始：

程序的入口是 `Program.cs`，这个文件不需要修改。

```cs
// Program.cs
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;

namespace QServer
{
    public class Program
    {
        public static void Main(string[] args) => BuildWebHost(args).Run();

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
}
```

`Program` 类引用了一个 `Startup` 类来进行配置：

```cs
// Startup.cs
using GraphQL;
using GraphQL.Http;
using GraphQL.Server.Transports.AspNetCore;
using GraphQL.Server.Ui.Playground;
using GraphQL.Types;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using QServer.GraphQL;

namespace QServer
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton<IDocumentExecuter, DocumentExecuter>();
            services.AddSingleton<IDocumentWriter, DocumentWriter>();
            services.AddSingleton<ISchema, QSchema>();
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseGraphQLHttp<ISchema>(new GraphQLHttpOptions());
            app.UseGraphQLPlayground(new GraphQLPlaygroundOptions() {Path = "/"});
        }
    }
}
```

可以看到这里有个类没有出现过：`QSchema`，这个类继承了 `Schema`，即实现了 `ISchema` 因此可以在这里按类型 `ISchema` 注入。

关于 `QSchema` 的细节可以之后再考虑。

> 关于运行时的 Web 服务监听的端口号等配置，在 `Properties/launchSettings.json` 中。

# GraphQL Schema

创建一个 GraphQL API 服务的第一步还是创建一个 Schema 对象作为根，有两种方案：


1. GraphQL Schema Text

    用专门的一个文件或者字符串描述 Schema 的结构。

    ```gql
    type query {
        greeting(name: String): String
    }
    ```

    然后编译这个文本，得到一个不含 Resolver 的 Schema 模型，然后补充定义 Resolver。

2. Native C# Object

    另外一种就是直接通过代码定义 Schema：

    ```cs
    public class QSchema : Schema {
        public QSchema() {
            Query = new QueryGraphType();
        }
    }

    public class QueryGraphType : ObjectGraphType {
        public QueryGraphType() {
            Field<StringGraphType>(
                "greeting",
                arguments: new QueryArguments(
                    new QueryArgument(typeof(StringGraphType)) { Name = "name" }
                ),
                resolve: ctx => $"hello, {ctx.GetArgument<string>("name")}!"
            );
        }
    }
    ```

    重点在于 `QSchema` 在其构造函数里将 `Query` 属性置为一个 `QQuery` 类的实例，一个 `ObjectGraphType` 实例，更精确地说，一个实现了 `IObjectGraphType` 接口的对象实例。

    `Query` 是 GraphQL 中一个特殊的对象，它是**查询类** API 的顶层入口。类似的还有 `Mutation` 表示**修改类** API 的顶层入口。

    > Query 表示**只读查询**，应保证其下的 API 不修改任何数据，保证纯度或者幂等性。
    > 
    > Mutation 表示**修改查询**，可以修改数据，没有纯度或幂等性的限制，也可以返回对象。
    > 



对比两种方法，你会发现第一种方法在快速的声明/塑造 API 时具有优势，但无法定义具体的逻辑，它更像是一种设计稿；
第二种方法的冗余代码较多，但类型强、提示强，还能附加具体的、能运行的实现。

> 本文推荐直接使用第二种方法。

# GraphQL Type

GraphQL 的亮点在于它的 Type System，一旦 Type 被完全定义，系统的 Query 只读查询部分基本随之完工。这是一个非常厉害的设计效果。

显然，我们需要自定义类型，在 GraphQL 的范畴下，所有的自定义类型的根是 `ObjectGraphType`，这个就像所有的类的根都是 `Object` 一样，但不同的是，GraphQL 自定义类都需要显式继承 `ObjectGraphType` 类。

```cs
public class QueryGraphType : ObjectGraphType { /* ... */ }
```

习惯上，这类直接或间接继承 `ObjectGraphType` 的类的命名都以 `GraphType` 结尾。

## Fields (字段)

在 GraphQL 中，字段对应了传统编程语言中的成员 (Member) 的概念，包含了传统编程语言中的字段、方法。

给自定义类型添加字段的方法：

```gql
type some {
    a: Int
    b: String
    # no Double in GraphQL
    c: Float
}
```

对应的是：

```cs
public class SomeGraphType : ObjectGraphType {
    public SomeGraphType() {
        Field<IntGraphType>("a", resolve: ctx => 1);
        Field<StringGraphType>("b", resolve: ctx => "well");
        Field<FloatGraphType>("c", "no Double in GraphQL", resolve: ctx => 0.1);
    }
}
```

可以看到一般都在一个公共的无参构造函数中调用 Field 方法来添加字段。
这个泛型方法有许多重载，这里我们使用的是：

```cs
public FieldType Field<TGraphType>(
    string name, // 字段名
    string description = null, // 描述
    QueryArguments arguments = null, // 参数
    Func<ResolveFieldContext<TSourceType>, object> resolve = null, // Resolver
    string deprecationReason = null // 废弃原因
) where TGraphType : IGraphType;
```

resolve 参数决定了对于一个特定的 GraphQL Resolve 上下文，应该返回什么对象。

含参数的情况之后讨论。

## Arguments (参数)

对于 Field 的无参数形式，可以认为是 C# 中的只读属性；当 Field 具有参数时，可以认为是 C# 中的方法。

```gql
type some {
    a: 
}
```