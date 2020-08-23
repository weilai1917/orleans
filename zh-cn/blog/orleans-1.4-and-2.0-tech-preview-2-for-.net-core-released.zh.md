# 面向.NET Core的Orleans 1.4和2.0 Tech Preview 2发布

[朱利安·多明格斯（Julian Dominguez）](https://github.com/jdom)2017/3/2下午5:54:19

* * *

# 奥尔良1.4.0

几周前，我们向NuGet.org发布了Orleans 1.4.0，其中的主要新主题是：

-   改进了JournaledGrain，用于事件源，并支持基于地理分布的基于日志的一致性提供程序。
-   具有固定放置的每仓库应用程序组件的Grain Services的抽象，其工作负载通过集群一致性环进行分区。
-   支持不均匀分布的可用粮仓的异构筒仓。
-   Service Fabric的群集成员资格提供程序。

当然，还有许多其他改进和错误修复，您可以在此处阅读：[Orleans v1.4.0发行说明](https://github.com/dotnet/orleans/releases/tag/v1.4.0)

# .NET Core的Orleans 2.0 Tech Preview 2

除了我们的标准版本外，我们还在使用支持.NET Standard（和.NET Core主机）的vNext功能。与TP1相似，此新预览版本与Orleans 1.X版本并不完全完全相同，但已经接近。自上次预览以来，我们已完成了许多错误修复，并且该版本是master分支中的最新版本（比1.4.0提前了一点）。

### 与奥尔良1.X的差异

此预发行版本中的一些显着差异或待处理事项：

-   奥尔良代码生成
    -   仅当使用Visual Studio 2017或最新的dotnet CLI在Windows上进行构建时，构建时间代码源（Microsoft.Orleans.OrleansCodeGenerator.Build nuget程序包）才有效。
    -   但是，运行时代码生成是跨平台工作的可行替代方法（通过引用Silo宿主和客户端项目中的Microsoft.Orleans.OrleansCodeGenerator程序包）。
-   .NET Standard中尚不提供BinaryFormatter（内置的.NET序列化），它已在Orleans中用作默认的后备序列化器（通常在序列化异常时使用）。现在，我们有了一个自定义的基于IL的后备序列化器，该序列化器应快速而强大，但如果您现有的代码依赖于它，则其行为可能会有所不同`[可序列化]`。
-   .NET标准中不支持System.Diagnostic.Trace.CorrelationManager.ActivityId。如果您依赖于此来关联谷物调用，请考虑改用Orleans.Runtime.RequestContext.ActivityId。

### 奥尔良2.0 TP2的生产准备就绪了吗？

还没。大声明：我们在.NET中进行CI测试（因为我们的测试严重依赖AppDomains创建内存孤岛群集，而.NET Core不支持这些孤岛，但我们计划尽快解决）。我们已经在.NET Core（包括Windows和Linux）中进行了一些基本的手动测试，并且我们的一些贡献者正在使用它开发新的服务。获得反馈（和PR！）是此版本的主要目标之一，尚未在生产中使用。此外，即使对于已完全移植的功能，也无法保证此技术预览版与Orleans 1.4完全向后兼容。在接近稳定版本之后，我们将列出所有已知的重大更改，以防您有兴趣将应用程序从1.X升级到2.0。

### 从哪里获得

因为此技术预览版不如1.X版本具有完整功能或稳定性，所以我们现在仅在MyGet中发布。您可以按照以下步骤在此处配置Feed来获取NuGet软件包：<https://dotnet.myget.org/gallery/orleans-ci>

### HelloWorld示例

现在，我们的存储库中有一个非常简单的示例，您可以使用它在.NET Core（无论是Windows，Linux还是MacOS中）中试用Orleans。样本位于<https://github.com/dotnet/orleans/tree/master/Samples/HelloWorld.NetCore>。

享受它，玩弄它，并让我们知道您的想法，无论是GitHub问题，PR还是只是在我们的Gitter频道中闲逛。
