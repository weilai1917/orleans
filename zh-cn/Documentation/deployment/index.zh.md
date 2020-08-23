---
layout: page
title: Running the Application
---

### Orleans应用

典型的Orleans应用程序由一组服务器进程（silo）和一组客户端进程（通常是web服务器）组成，这些进程接收外部请求，将它们转换为grain方法调用，并返回结果。因此，运行Orleans应用程序首先需要启动一个silos集群。出于测试目的，集群可以由单个silos组成。对于可靠的生产部署，我们显然希望集群中有多个silos用于容错和扩展。

集群运行后，我们可以启动一个或多个客户端进程，这些进程连接到集群并可以向grains发送请求。客户端连接到silos-gateway上的特殊tcp端点。默认情况下，群集中的每个silos都启用了客户端网关。因此，客户可以同时连接到所有silos，以获得更好的性能和弹性。

### 配置和启动silos

silos通过`群集配置`反对。它可以直接实例化和填充，从文件中加载设置，或者使用多个适用于不同部署环境的可用帮助器方法创建。对于本地测试，最简单的方法是使用`clusterconfiguration.localhostprimarysilo（）`助手方法。然后将配置对象传递给`silos`类，可以在该类之后初始化和启动。

您可以创建一个空的控制台应用程序项目，目标是.NETFramework4.6.1或更高版本，用于托管silos。添加`Microsoft.Orleans.Server公司`项目的nuget元包。

```
PM> Install-Package Microsoft.Orleans.Server
```

以下是如何启动本地silos的示例：

```csharp
var siloConfig = ClusterConfiguration.LocalhostPrimarySilo(); 
var silo = new SiloHost("Test Silo", siloConfig); 
silo.InitializeOrleansSilo(); 
silo.StartOrleansSilo();

Console.WriteLine("Press Enter to close."); 
// wait here
Console.ReadLine(); 

// shut the silo down after we are done.
silo.ShutdownOrleansSilo();
```

### 配置和连接客户端

通过一个`客户端配置`对象和`客户端生成器`是的。`客户端配置`对象可以直接实例化和填充，从文件加载设置，或者使用多个适用于不同部署环境的可用帮助器方法创建。对于本地测试，最简单的方法是使用`clientconfiguration.localhostsilo（）`助手方法。然后将配置对象传递给`客户端生成器`上课。

`客户端生成器`公开配置其他客户端功能的更多方法。之后`建立`方法`客户端生成器`调用对象以获取`IClusterClient公司`接口。最后，我们打电话`连接（）`方法连接到群集。

您可以创建一个空的控制台应用程序项目，目标是.net framework 4.6.1或更高版本以运行客户端，或者重用为托管silos而创建的控制台应用程序项目。添加`Microsoft.Orleans.client公司`项目的nuget元包。

```
PM> Install-Package Microsoft.Orleans.Client
```

以下是客户端如何连接到本地silos的示例：

```csharp
var config = ClientConfiguration.LocalhostSilo();
var builder = new ClientBuilder().UseConfiguration(config).
var client = builder.Build();
await client.Connect();
```

### 生产配置

我们在这里使用的配置示例用于测试与`本地服务器`是的。在生产环境中，silos和客户机通常运行在不同的服务器上，并使用可靠的群集配置选项之一进行配置。您可以在《配置指南》]（../clusters\\u and\\u clients/configuration\\u guide/index.md）和[群集管理](../implementation/cluster_management.zh.md)是的。
