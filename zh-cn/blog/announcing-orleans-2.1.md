# 宣布Orleans2.1

[鲁本·邦德](https://github.com/ReubenBond)2018/10/1下午7:17:59

* * *

今天，我们宣布了Orleans2.1。此版本包括对2.0的重大性能改进，对分布式事务支持的重大更新，新的代码生成器以及用于共同托管方案的新功能，以及较小的修复和改进。阅读[在这里发布说明](https://github.com/dotnet/orleans/releases/tag/v2.1.0)。

## 新调度程序

从Orleans2.1开始，我们有一个[重写核心调度程序](https://github.com/dotnet/orleans/pull/3792)利用[.NET Core的ThreadPool的性能改进](https://blogs.msdn.microsoft.com/dotnet/2017/06/07/performance-improvements-in-net-core/)。新的调度程序使用具有本地队列相似性的工作窃取队列来减少争用并提高吞吐量和延迟。这些改进适用于所有平台/框架。社区成员报告说他们的服务具有显着的响应能力和吞吐量改进，而我们的测试表明吞吐量提高了30％。这种新的调度程序在低负载期间的CPU使用率也要低得多，这有利于共同托管方案并改善CPU性能分析体验。特别感谢[德米特罗·瓦库连科(Dmytro Vakulenko)](https://twitter.com/dVakulen)从社区推动这项工作从理论到实验和完成。

## 分布式交易

Orleans起源于Microsoft Research，我们将继续与MSR和产品团队合作，将可扩展的分布式事务等功能引入生产环境。分布式事务支持最初是在Orleans2.0中作为实验性功能引入的，而在2.1中，我们将使用新的完全分散的事务管理器来刷新发行版。除了对交易管理器进行了改进之外，整个交易系统仍在继续进行大量投资，以准备生产代码并在Orleans的未来版本中稳定发布。我们认为分布式事务在2.1中具有“候选发布”质量。从Sergey的演讲中了解Orleans的分布式交易，[分布式事务已死，分布式事务万岁！来自J On the Beach 2018](https://www.youtube.com/watch?v=8A5bRdyZXJw)。阅读有关交易，面向参与者的数据库系统以及其他主题的研究论文。[OrleansMicrosoft Research网站](https://www.microsoft.com/en-us/research/project/orleans-virtual-actors/#!publications)。

## 直接客户

Orleans2.1引入了一种与Grains进行交互并与ASP.NET或gRPC等框架进行互操作的新方法。该功能称为*直接客户*并且它允许以一种方式共同托管客户端和孤岛，从而使客户端不仅可以与其连接的孤岛，而且可以与整个集群进行更有效的通信。启用直接客户端后，`ICluster客户端`和`IGrain工厂`可以从silos容器中解析出来，并用于创建可以调用的Grains引用。这些调用使用本地silos对群集和grains位置的了解来避免不必要的复制，序列化和网络跃点。另外，由于此功能与silos本身具有相同的内部结构，因此在线程间传递Grains引用时，它提供了无缝的体验。在Orleans2.1中，我们将直接客户作为选择加入的功能。通过调用启用它`ISiloHostBuilder.EnableDirectClient()`在silos配置期间。

## 新代码生成器

此版本包括一个新的代码生成包，`Microsoft.Orleans.CodeGenerator.MSBuild`，是现有软件包的替代品，`Microsoft.Orleans.OrleansCodeGenerator.Build`。新的代码生成器利用Roslyn进行代码分析，以避免加载应用程序二进制文件。结果，它避免了因相互依赖的版本冲突和目标框架不同而导致的问题。如果您遇到问题，请通过打开一个问题告知我们[的GitHub](https://github.com/dotnet/orleans/)。新的代码生成器还改善了对增量构建的支持，这将缩短构建时间。

## 其他改进

-   [Grains方法可以返回`ValueTask <T>`](https://github.com/dotnet/orleans/pull/4562)- 谢谢[@kutensky](https://twitter.com/kutensky)
-   [删除了每次访问计时器分配](https://github.com/dotnet/orleans/pull/4399)，减少.NET Timer队列争用
-   [修正](https://github.com/dotnet/orleans/pull/4853)
    [对于](https://github.com/dotnet/orleans/pull/4883)Silo关闭行为-谢谢[@yevhen](https://twitter.com/yevhen)用于报告和[调查中](https://github.com/dotnet/orleans/issues/4757)
-   [配置Grains收集](https://github.com/dotnet/orleans/pull/4890)空闲时间使用`[CollectionAgeLimit(分钟= x)]`- 谢谢[@aRajeshKumar](https://github.com/arajeshkumar)

## .NET Core 2.1的已知问题

用户[孙忠峰报告了问题](https://github.com/dotnet/orleans/issues/4990)在启用TieredCompilation的.NET Core 2.1上运行Orleans。我们将此归结为[CoreCLR中的JIT问题](https://github.com/dotnet/coreclr/issues/20040)，CoreCLR团队迅速对其进行了诊断和修复。.NET Core 2.1默认情况下未启用TieredCompilation，并且此修复程序是[预计不会出现在.NET Core 2.1中，但将包含在.NET Core 2.2中](https://github.com/dotnet/coreclr/pull/20083#issuecomment-424464934)。**如果您在.NET Core 2.1上运行Orleans，请不要启用TieredCompilation。**我们要感谢社区中为该版本做出贡献并帮助测试预发行版本的每个人。
