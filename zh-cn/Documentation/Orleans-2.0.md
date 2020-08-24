---
layout: page
title: Orleans 2.0
---

# Orleans2.0

2.0是Orleans的一个主要版本，其主要目标是使其与.NET标准2.0兼容并跨平台（通过.NET Core）。作为这项工作的一部分，OrleansAPI进行了几次现代化，以使其更符合像ASP.NET这样的技术在当今的配置和托管方式。

因为它与.NET标准2.0兼容，所以Orleans 2.0可以被针对.NET核心或完整.NET框架的应用程序使用。核心团队对这个版本进行测试的重点是在.NETFramework框架上，以确保现有的应用程序可以轻松地从1.5迁移到2，并具有完全向后兼容性。

2.0中最显著的变化如下：

-   完全转移到利用fluid builder模式api的依赖注入的编程配置。

基于配置对象和xml文件的旧api是为了向后兼容而保留的，但不会向前移动，将来会被弃用。有关详细信息，请参见[配置](clusters_and_clients/configuration_guide/index.md)章节。

-   应用程序程序集的显式编程规范，它取代了Orleans运行时在silos或客户端初始化时自动扫描文件夹的功能。

orleans仍然会在指定的程序集中自动查找相关类型，如grain接口和类、序列化程序等，但它不再尝试加载在文件夹中可以找到的每个程序集。为了向后兼容，提供了用于加载文件夹中所有程序集的可选助手方法：`iapplicationpartmanager.addfromapplicationbasedirectory()`是的。

见[配置](clusters_and_clients/configuration_guide/index.md)和[迁移](resources/Migration/Migration1.5.md)部分了解更多详细信息。

-   代码生成的彻底检查。

虽然代码生成对开发人员来说基本上是不可见的，但是在处理各种可能类型的序列化时，代码生成变得更加健壮。F组件需要特殊处理。见[代码生成](resources/Migration/Codegen.md)部分了解更多详细信息。

-   创建了`Microsoft.Orleans.core.abstractions公司`nuget包并将几个类型移动/重构到其中。

grain代码很可能只需要引用这些抽象，而silo主机和客户端将引用更多的orleans包。我们计划不经常更新这个包。

-   添加对作用域服务的支持。

这意味着每个grain激活都有自己的作用域服务提供者，而orleans注册了一个上下文`IGrainActivationContext`可以注入*瞬变的*或*范围*获取激活特定信息和Grains激活生命周期事件访问权限的服务。这与asp.net core 2.0为每个请求创建作用域上下文的方式类似，但对于orleans，它适用于grain激活的整个生命周期。见[使用寿命和注册选项](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#service-lifetimes-and-registration-options)在ASP.NET核心文档中获取有关服务生命周期的详细信息。

-   迁移了要使用的日志基础结构`Microsoft.Extensions.Logging`（与ASP.NET Core 2.0相同的摘要）。

-   2.0包含了对acid分布式跨粒度事务的测试版支持。

该功能将为原型设计和开发做好准备，并将在2.0版本发布后的某个时间毕业用于生产。见[交易](grains/transactions.md)更多细节。
