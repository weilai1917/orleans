---
layout: page
title: Global-Single-Instance Grains
---

### grains协调属性

开发人员可以指示集群应该在何时以及如何根据特定的grain类协调其grain目录。这个`[全球照明]`属性意味着我们需要与在单个全局集群中运行orleans时相同的行为：即将所有调用路由到一个单一的grain激活。相反地，`[OneInstancePerCluster]`属性指示每个群集可以有自己的独立激活。如果集群之间的通信是不需要的，这是合适的。

属性放在grain实现上。例如：

```csharp
using Orleans.MultiCluster;

[GlobalSingleInstance]
public class MyGlobalGrain : Orleans.Grain, IMyGrain {
   ...
}

[OneInstancePerCluster]
public class MyLocalGrain : Orleans.Grain, IMyGrain {
   ...
}
```

如果grain类没有指定这些属性中的任何一个，则默认为`[OneInstancePerCluster]`或`[全球照明]`如果配置参数`默认为UseGlobalSingleInstances`设置为true。

#### 全局单实例Grain协议

当访问全局单实例(GSI)grains，并且不知道存在激活时，在激活新实例之前执行特殊GSI激活协议。特别地，请求被发送到当前[多群集配置](MultiClusterConfiguration.md)检查他们是否已经激活了这种Grains。如果所有响应均为负，则在此群集中创建新激活。否则，将使用远程激活(并在本地目录中缓存对它的引用)。

#### 每个集群Grain一个实例的协议

对于每个集群Grain的一个实例，没有集群间通信。它们只需在每个集群中独立使用标准的orleans机制。在orleans框架本身中，以下grain类用`[OneInstancePerCluster]`属性：`管理grain`，请`grains数据库成员表`，和`粒状`是的。

#### 可疑激活

如果GSI协议在3次重试(或配置参数指定的任何数目)后没有从所有集群接收到结论性响应`GlobalSingleInstanceNumberEntries全局`，它乐观地创建了一个新的本地“可疑”激活，支持可用性而不是一致性。

可疑的激活可能是重复的(因为某些在GSI协议激活期间没有响应的远程集群可能仍然激活了此grains)。因此，每隔30秒(或配置参数指定的任何间隔)定期`GlobalSingleInstanceRetryInterval`)对于所有可疑的激活，再次运行gsi协议。这确保一旦恢复集群之间的通信，就可以检测并删除重复的激活。
