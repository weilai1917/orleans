---
layout: page
title: Student Projects
---

# 学生项目

我们为学生推荐两种类型的项目：

1.  第一种类型包括探索性的，开放式的，**研究性项目**目标是在Orleans建立新的能力。这些项目的范围通常很广，适合硕士、博士或高级本科生最后一年的学习。这些项目的最终目标是为Orleans提供创意和设计。我们不一定期望在这些项目中生成的代码直接贡献到这个存储库中，但是这将是很好的。

2.  第二种类型包括**学生教育理念**. 这些都是可以在Orleans之上构建的有趣应用程序的想法，或者是为Orleans开发的一些新功能。这些项目适合在高级本科或研究生课程中授课，学生们学习云计算和现代分布式技术，并希望获得构建云应用程序的实际动手经验。我们不希望在这些项目中生成的代码直接贡献给这个存储库。

### 研究项目：

1.  **自动缩放。**在这个项目中，学生可以从探索现有的自动伸缩机制开始，以控制windowsazure中的资源分配([自动调整应用程序块](http://azure.microsoft.com/en-us/documentation/articles/cloud-services-dotnet-autoscaling-application-block/)). 下一步包括探索由Orleans收集的各种统计数据和资源消耗指标，并将它们用作Azure自动缩放的输入。该项目的高级阶段可能涉及改善Orleans内部对弹性变化作出反应的机制，例如通过实施实时演员迁移来减少利用新资源所需的时间。

2.  **为基于Orleans的云服务自动生成前端**. 这个项目将Orleans actor模型无缝地扩展到HTTP世界中。项目的升级部分包括根据参与者的.NET接口和元数据动态生成HTTP端点。主要部分包括自动生成前端以支持web套接字和双向数据流，这需要复杂的代码生成和优化以获得高性能。它还需要注意容错，在服务器重新启动、客户端重新连接和迁移过程中保持流式会话的高可用性，这是一个重大的研究挑战。

3.  **实体框架的存储提供程序**. 这个项目包括使Orleans对象能够将它们的状态存储在数据库中，并随后对其进行查询。这可能包括使用EntityFramework（EF）在SQLAzure数据库上添加对Orleans对象持久性的支持，EF是Microsoft针对.NET的开源对象关系映射器，并通过LINQ查询公开这些数据。可以使用标准数据库基准和/或定制的Orleans应用程序来评估和调整实现。

4.  **分布式系统基准测试**. 定义一个适用于像Orleans这样的分布式系统的基准测试列表。基准应用程序在精神上可能类似于[TPC数据库基准](http://www.tpc.org/information/benchmarks.asp)或[UCB“平行矮星”](https://paralleldwarfs.codeplex.com/)实施[在这里](http://view.eecs.berkeley.edu/wiki/Dwarfs)可以用来描述分布式框架的性能和可伸缩性。例如，考虑为Orleans开发一个新的基准，以比较存储供应商的性能。

5.  **流上的声明性数据流语言**. 定义并构建[三叉戟风暴](https://storm.apache.org/documentation/Trident-tutorial.html)就像OrleansStreams上的陈述性语言。开发一个优化程序来配置流处理以最小化总体成本。

6.  **客户端设备的编程模型**. 将Orleans扩展到客户端设备，如传感器、电话、平板电脑和台式机。启用粒度逻辑在客户端上执行。潜在地支持层分割，也就是说，动态地决定哪些部分的代码在设备上执行，哪些部分被卸载到云上。

7.  **对grain/actor类、二级索引的查询**. 建立一个分布式的、可扩展的、可靠的Grains指数。这包括正式定义查询模型和实现分布式索引。索引本身可以实现为OrleansGrains和/或存储在数据库中。

8.  **大规模模拟**. Orleans适合大规模的模拟建筑。探索Orleans在不同模拟中的应用，例如，蛋白质相互作用、网络模拟、模拟退火等。

### 课程项目：

1.  **物联网应用**. 例如，该应用程序可以使传感器/设备向云端报告其状态，在云中，每个设备都由Orleans的参与者表示。用户可以通过web浏览器连接到代表其设备的actor，并检查其状态或控制它。这个项目需要掌握许多现代云技术，包括[Windows Azure](http://azure.microsoft.com/)，Orleans，[网络API](http://www.asp.net/web-api)或者ASP.NET,[信号员](http://signalr.net/)用于将命令从云端传输回设备，并编写传感器/设备/手机应用程序。

2.  **基于Orleans的云中类似Twitter的大型可伸缩聊天服务**. 每个用户都可以由一个Orleans演员代表，该演员包含其追随者列表。

3.  **基于Orleans的Faceboook类社交应用**. 每个用户可以由一个Orleans演员代表，其中包括一个朋友列表和朋友可以在上面写字的墙。

4.  **简单存储提供程序**. 为存储系统（如键值存储或数据库系统）添加存储提供程序。一个简单的人可以用[Orleans序列化程序](https://github.com/dotnet/orleans/tree/master/src/Orleans/Serialization)，如现有的[Azure表存储提供程序](https://github.com/dotnet/orleans/blob/master/src/OrleansProviders/Storage/AzureTableStorage.cs). 更复杂的方法是将Orleans类的状态变量映射到存储系统的细粒度结构。复杂的是上面提到的实体框架存储提供程序*研究项目*. 比较不同类型和大小的参与者状态的不同存储提供程序的性能。

5.  **与其他分布式应用框架的比较**. 以为另一个应用程序框架编写的示例应用程序为例，例如[谷歌应用引擎](https://cloud.google.com/appengine/docs)或[阿克卡](http://akka.io/)把它翻译成Orleans。通过比较应用程序，总结每个框架的相对优势和劣势。

* * *

### 已完成的研究项目：

下面是一些以前成功的研究项目的例子。

1.  **分布式日志分析、关联与调试**. 大型分布式系统的调试是一项具有挑战性的任务，因为在不同进程和不同机器上运行的分布式组件之间存在着大量的数据和复杂的动态交互。本项目的目标是分析现有技术，提出解决方案，然后实现原型工具，用于收集、关联和分析多机器分布式应用程序运行时环境中的应用程序错误日志文件数据。这包括从多个角度探索问题空间，包括：

    ```
      a. Approaches to efficient logging, collection and analysis of failure information from various log-capture mechanisms in a distributed Orleans runtime environment.

      b. Possible applications of machine learning to find log patterns that signal serious production issues, and then detecting these patterns in near real time as a production monitoring utility.

      c. Ways to help individual developers perform real-time debugging of run-time issues with their applications.

     This project was performed successfully and result in a published paper [PAD: Performance Anomaly Detection in Multi-Server Distributed Systems](http://research.microsoft.com/apps/pubs/?id=217109) and a proof of concept implementation of a distributed log analysis tool.
    ```

2.  **Horton-分布式图形数据库**. Horton是一个研究项目，目标是建立一个存储、管理和查询大规模分布式图形的系统。它完全是作为一个Orleans应用程序实现的。该项目的结果是[出版物](http://research.microsoft.com/en-us/projects/ldg/)以及一些非常成功的学生项目。
