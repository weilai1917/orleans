---
layout: page
title: Orleans NuGet Packages
---

# OrleansNuGet软件包

## 关键套餐

在大多数情况下，您需要使用5个关键的NuGet软件包：

### [微软Orleans核心抽象](https://www.nuget.org/packages/Microsoft.Orleans.Core.Abstractions/)

```powershell
PM> Install-Package Microsoft.Orleans.Core.Abstractions
```

包含Orleans.Core.Abstractions.dll，该文件定义开发应用程序代码（grains接口和类）所需的Orleans公共类型。任何Orleans项目都需要直接或间接引用此软件包。将其添加到定义grain接口和类的项目中。

### Microsoft Orleans构建时代码生成

-   [Microsoft.Orleans.OrleansCodeGenerator.Build](https://www.nuget.org/packages/Microsoft.Orleans.OrleansCodeGenerator.Build/)。

    ```powershell
    PM> Install-Package Microsoft.Orleans.OrleansCodeGenerator.Build
    ```

    出现在Orleans1.2.0中。为Grains接口和实施项目提供时间支持。将其添加到您的grain接口和实现项目中，以启用grain引用和序列化器的代码生成。

-   [Microsoft.Orleans.CodeGenerator.MSBuild](https://www.nuget.org/packages/Microsoft.Orleans.CodeGenerator.MSBuild/)。

    ```powershell
    PM> Install-Package Microsoft.Orleans.CodeGenerator.MSBuild
    ```

    出现在[Orleans2.1.0](https://blogs.msdn.microsoft.com/orleans/2018/10/01/announcing-orleans-2-1/)。替代`Microsoft.Orleans.OrleansCodeGenerator.Build`包。利用Roslyn进行代码分析，以避免加载应用程序二进制文件并改善对增量构建的支持，这将缩短构建时间。

### [Microsoft Orleans服务器库](https://www.nuget.org/packages/Microsoft.Orleans.Server/)

```powershell
PM> Install-Package Microsoft.Orleans.Server
```

一个易于构建和启动silos的元软件包。包括以下软件包：

-   微软Orleans核心抽象
-   微软Orleans核心
-   Microsoft.Orleans.OrleansRuntime
-   Microsoft.Orleans.OrleansProviders

### [Microsoft Orleans客户库](https://www.nuget.org/packages/Microsoft.Orleans.Client/)

```powershell
PM> Install-Package Microsoft.Orleans.Client
```

一个用于轻松构建和启动Orleans客户（前端）的元软件包。包括以下软件包：

-   微软Orleans核心抽象
-   微软Orleans核心
-   Microsoft.Orleans.OrleansProviders

### [微软Orleans核心库](https://www.nuget.org/packages/Microsoft.Orleans.Core/)

```powershell
PM> Install-Package Microsoft.Orleans.Core
```

包含应用程序代码和Orleans客户（前端）使用的大多数Orleans公共类型的实现。引用它以构建使用Orleans类型但不处理托管或孤岛的库和客户端应用程序。包含在Microsoft.Orleans.Client和Microsoft.Orleans.Server元软件包中，并且大多数其他软件包直接或间接地对其进行引用。

## 代管

### [微软Orleans运行时](https://www.nuget.org/packages/Microsoft.Orleans.OrleansRuntime/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansRuntime 
```

用于配置和启动silos的库。在您的silos主机项目中引用它。包含在Microsoft.Orleans.Server元软件包中。

### [Microsoft Orleans运行时抽象](https://www.nuget.org/packages/Microsoft.Orleans.Runtime.Abstractions/)

```powershell
PM> Install-Package Microsoft.Orleans.Runtime.Abstractions 
```

包含Microsoft.Orleans.OrleansRuntime中实现的类型的接口和抽象。

### [Microsoft Orleans在Azure云服务上托管](https://www.nuget.org/packages/Microsoft.Orleans.Hosting.AzureCloudServices/)

```powershell
PM> Install-Package Microsoft.Orleans.Hosting.AzureCloudServices
```

包含用于将silos和Orleans客户端托管为Azure云服务的帮助程序类（工作人员角色和Web角色）。

### [Microsoft Orleans Service Fabric托管支持](https://www.nuget.org/packages/Microsoft.Orleans.Hosting.ServiceFabric/)

```powershell
PM> Install-Package Microsoft.Orleans.Hosting.ServiceFabric 
```

包含用于将silos作为无状态Service Fabric服务托管的帮助程序类。

## 集群提供商

以下软件包包括用于在各种存储技术中持久存储集群成员数据的插件。

### [Microsoft Orleans群集提供者，用于Azure表存储](https://www.nuget.org/packages/Microsoft.Orleans.Clustering.AzureStorage/)

```powershell
PM> Install-Package Microsoft.Orleans.Clustering.AzureStorage
```

包括用于使用Azure表存储群集成员资格数据的插件。

### [Microsoft Orleans ADO.NET提供程序的群集提供程序](https://www.nuget.org/packages/Microsoft.Orleans.Clustering.AdoNet/)

```powershell
PM> Install-Package Microsoft.Orleans.Clustering.AdoNet
```

包括用于使用ADO.NET在支持的数据库之一中存储集群成员资格数据的插件。

### [Microsoft Orleans领事实用程序](https://www.nuget.org/packages/Microsoft.Orleans.OrleansConsulUtils/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansConsulUtils
```

包括用于使用Consul存储集群成员数据的插件。

### [Microsoft Orleans ZooKeeper实用程序](https://www.nuget.org/packages/Microsoft.Orleans.OrleansZooKeeperUtils/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansZooKeeperUtils
```

包括用于使用ZooKeeper存储集群成员数据的插件。

### [适用于AWS DynamoDB的Microsoft Orleans群集提供程序](https://www.nuget.org/packages/Microsoft.Orleans.Clustering.DynamoDB/)

```powershell
PM> Install-Package Microsoft.Orleans.Clustering.DynamoDB
```

包括用于使用AWS DynamoDB存储集群成员数据的插件。

## 提醒提供者

以下软件包包括用于在各种存储技术中持久保存提醒的插件。

### [Microsoft Orleans提醒Azure表存储](https://www.nuget.org/packages/Microsoft.Orleans.Reminders.AzureStorage/)

```powershell
PM> Install-Package Microsoft.Orleans.Reminders.AzureStorage
```

包括用于使用Azure表存储提醒的插件。

### [Microsoft Orleans提醒ADO.NET提供程序](https://www.nuget.org/packages/Microsoft.Orleans.reminders.AdoNet/)

```powershell
PM> Install-Package Microsoft.Orleans.Reminders.AdoNet
```

包括用于使用ADO.NET在一个受支持的数据库之一中存储提醒的插件。

### [Microsoft Orleans提醒提供者，适用于AWS DynamoDB](https://www.nuget.org/packages/Microsoft.Orleans.Reminders.DynamoDB/)

```powershell
PM> Install-Package Microsoft.Orleans.Reminders.DynamoDB
```

包括用于使用AWS DynamoDB存储提醒的插件。

## grain储存供应商

以下软件包包括用于在各种存储技术中保持grains状态的插件。

### [Microsoft Orleans Persistence Azure存储](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.AzureStorage/)

```powershell
PM> Install-Package Microsoft.Orleans.Persistence.AzureStorage
```

包括用于使用Azure表或Azure Blob存储Grain状态的插件。

### [Microsoft Orleans Persistence ADO.NET提供程序](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.AdoNet/)

```powershell
PM> Install-Package Microsoft.Orleans.Persistence.AdoNet
```

包括用于使用ADO.NET在支持的数据库之一中存储grains状态的插件。

### [MicrosoftOrleans持久性DynamoDB](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.DynamoDB/)

```powershell
PM> Install-Package Microsoft.Orleans.Persistence.DynamoDB
```

包括用于使用AWS DynamoDB存储Grain状态的插件。

## 流提供者

以下软件包包括用于传递流事件的插件。

### [Microsoft Orleans ServiceBus实用程序](https://www.nuget.org/packages/Microsoft.Orleans.OrleansServiceBus/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansServiceBus
```

包括Azure事件中心的流提供程序。

### [Microsoft Orleans流式Azure存储](https://www.nuget.org/packages/Microsoft.Orleans.Streaming.AzureStorage/)

```powershell
PM> Install-Package Microsoft.Orleans.Streaming.AzureStorage
```

包括Azure队列的流提供程序。

### [Microsoft Orleans Streaming AWS SQS](https://www.nuget.org/packages/Microsoft.Orleans.Streaming.SQS/)

```powershell
PM> Install-Package Microsoft.Orleans.Streaming.SQS
```

包括AWS SQS服务的流提供程序。

### [Microsoft Orleans Google Cloud Platform实用程序](https://www.nuget.org/packages/Microsoft.Orleans.OrleansGCPUtils/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansGCPUtils
```

包括GCP PubSub服务的流提供程序。

## 其他套餐

### [微软Orleans代码生成](https://www.nuget.org/packages/Microsoft.Orleans.OrleansCodeGenerator/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansCodeGenerator
```

包括运行时代码生成器。

### [微软Orleans事件来源](https://www.nuget.org/packages/Microsoft.Orleans.EventSourcing/)

```powershell
PM> Install-Package Microsoft.Orleans.EventSourcing 
```

包含一组用于创建具有事件源状态的Grains类的基本类型。

## 开发与测试

### [微软Orleans供应商](https://www.nuget.org/packages/Microsoft.Orleans.OrleansProviders/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansProviders
```

包含一组将数据保留在内存中的持久性和流提供程序。用于测试。通常，不建议将其用于生产，除非可以接受数据丢失以防止孤岛故障。

### [Microsoft Orleans测试主机库](https://www.nuget.org/packages/Microsoft.Orleans.TestingHost/)

```powershell
PM> Install-Package Microsoft.Orleans.TestingHost
```

包括用于在测试项目中托管孤岛和客户端的库。

## 序列化器

### [微软Orleans债券序列化器](https://www.nuget.org/packages/Microsoft.Orleans.Serialization.Bond/)

```powershell
PM> Install-Package Microsoft.Orleans.Serialization.Bond
```

包括对[债券序列化器](https://github.com/microsoft/bond)。

### [Microsoft Orleans Google实用工具](https://www.nuget.org/packages/Microsoft.Orleans.OrleansGoogleUtils/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansGoogleUtils
```

包括Google协议缓冲区序列化程序。

### [Microsoft Orleans protobuf-net序列化器](https://www.nuget.org/packages/Microsoft.Orleans.ProtobufNet/)

```powershell
PM> Install-Package Microsoft.Orleans.ProtobufNet
```

包括协议缓冲区串行器的protobuf-net版本。

## 遥测

### [MicrosoftOrleans遥测消费者-性能计数器](https://www.nuget.org/packages/Microsoft.Orleans.OrleansTelemetryConsumers.Counters/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansTelemetryConsumers.Counters
```

Orleans Telemetry API的Windows Performance Counter实现。

### [Microsoft Orleans Telemetry使用者-Azure应用程序见解](https://www.nuget.org/packages/Microsoft.Orleans.OrleansTelemetryConsumers.AI/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansTelemetryConsumers.AI
```

包括用于Azure Application Insights的遥测使用者。

### [微软Orleans遥测消费者-NewRelic](https://www.nuget.org/packages/Microsoft.Orleans.OrleansTelemetryConsumers.NewRelic/)

```powershell
PM> Install-Package Microsoft.Orleans.OrleansTelemetryConsumers.NewRelic
```

包括NewRelic的遥测用户。

## 工具类

### [Microsoft Orleans性能计数器工具](https://www.nuget.org/packages/Microsoft.Orleans.CounterControl/)

```powershell
PM> Install-Package Microsoft.Orleans.CounterControl
```

包括OrleansCounterControl.exe，该文件为Orleans统计信息和已部署的Grain类注册Windows性能计数器类别。需要海拔。可以作为角色启动任务的一部分在Azure中执行。

## 交易次数

### [Microsoft Orleans Transactions支持](https://www.nuget.org/packages/Microsoft.Orleans.Transactions/)

```powershell
PM> Install-Package Microsoft.Orleans.Transactions
```

包括对跨Grains交易（测试版）的支持。

### [Microsoft在Azure上进行Orleans交易](https://www.nuget.org/packages/Microsoft.Orleans.Transactions.AzureStorage/)

```powershell
PM> Install-Package Microsoft.Orleans.Transactions.AzureStorage
```

包括用于在Azure表（beta）中持久保存事务日志的插件。
