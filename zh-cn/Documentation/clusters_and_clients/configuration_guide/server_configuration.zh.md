---
layout: page
title: Server Configuration
---

> [啊！注意!]如果要启动本地silos和本地客户机进行开发，请查看[本地开发配置页](local_development_configuration.md)

# 服务器配置

silos通过编程方式配置`silohostbuilder软件`以及一些补充选项类。Orleans的选项类遵循[ASP.NET选项](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)模式，可以通过文件、环境变量等加载。

silos配置有几个关键方面：

-   Orleans聚类信息
-   群集提供程序
-   用于silos到silos和客户端到silos通信的终结点
-   应用程序部件

这是一个silos配置的示例，它定义群集信息、使用azure群集并配置应用程序部分：

```csharp
var silo = new SiloHostBuilder()
    // Clustering information
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "my-first-cluster";
        options.ServiceId = "AspNetSampleApp";
    })
    // Clustering provider
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    // Endpoints
    .ConfigureEndpoints(siloPort: 11111, gatewayPort: 30000)
    // Application parts: just reference one of the grain implementations that we use
    .ConfigureApplicationParts(parts => parts.AddApplicationPart(typeof(ValueGrain).Assembly).WithReferences())
    // Now create the silo!
    .Build();
```

让我们分解此示例中使用的步骤：

## Orleans聚类信息

```csharp
    [...]
    // Clustering information
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "my-first-cluster";
        options.ServiceId = "AspNetSampleApp";
    })
    [...]
```

这里我们做两件事：

-   设置`棒状的`到`“我的第一个群集”`：这是Orleans集群的唯一ID。使用此ID的所有客户机和silos都可以直接相互通信。你可以选择使用不同的`棒状的`不过，对于不同的部署。
-   设置`服务ID`到`“aspnetsampleap”`：这是应用程序的唯一ID，某些提供程序（如持久性提供程序）将使用它。**此ID应保持稳定，并且在部署期间不会更改**是的。

## 群集提供程序

```csharp
    [...]
    // Clustering provider
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    [...]
```

通常，在orleans上构建的服务部署在节点集群上，或者部署在专用硬件上，或者部署在azure中。为了进行开发和基本测试，可以在单节点配置中部署Orleans。当部署到一个节点集群时，orleans在内部实现了一组协议来发现和维护集群中orleans silo的成员关系，包括检测节点故障和自动重新配置。

为了可靠地管理集群成员关系，orleans使用azure表、sql server或apache zookeeper来同步节点。

在这个示例中，我们使用azure表作为成员资格提供程序。

## 端点

```csharp
var silo = new SiloHostBuilder()
    [...]
    // Endpoints
    .ConfigureEndpoints(siloPort: 11111, gatewayPort: 30000)
    [...]
```

Orleanssilos有两种典型的端点配置：

-   silos到silos端点，用于同一集群中silos之间的通信
-   客户端到silos端点（或网关），用于在同一集群中的客户端和silos之间进行通信

在示例中，我们使用helper方法`.配置终结点（siloport:11111，gatewayport:30000）`它将用于silos到silos通信的端口设置为`11111个`和网关的端口`30000个`是的。此方法将检测要侦听的接口。

这种方法在大多数情况下应该足够了，但是如果需要的话，可以进一步自定义它。下面是如何将外部IP地址与某些端口转发一起使用的示例：

```csharp
[...]
.Configure<EndpointOptions>(options =>
{
    // Port to use for Silo-to-Silo
    options.SiloPort = 11111;
    // Port to use for the gateway
    options.GatewayPort = 30000;
    // IP Address to advertise in the cluster
    options.AdvertisedIPAddress = IPAddress.Parse("172.16.0.42");
    // The socket used for silo-to-silo will bind to this endpoint
    options.GatewayListeningEndpoint = new IPEndPoint(IPAddress.Any, 40000);
    // The socket used by the gateway will bind to this endpoint
    options.SiloListeningEndpoint = new IPEndPoint(IPAddress.Any, 50000);

})
[...]
```

在内部，silos会监听`0.0.0.0:40000个`和`0.0.0.0:50000`，但在成员资格提供程序中发布的值将是`172.16.0.42:11111`和`172.16.0.42:30000`是的。

## 应用程序部件

```csharp
    [...]
    // Application parts: just reference one of the grain implementations that we use
    .ConfigureApplicationParts(parts => parts.AddApplicationPart(typeof(ValueGrain).Assembly).WithReferences())
    [...];
```

虽然技术上不需要此步骤（如果未配置，Orleans将扫描当前文件夹中的所有程序集），但鼓励开发人员对此进行配置。此步骤将帮助Orleans加载用户程序集和类型。这些组件称为应用程序部件。所有粒度、粒度接口和序列化器都是使用应用程序部件发现的。

应用程序部件的配置使用`IApplicationPartsManager应用程序部件管理器`，可以使用`配置应用程序部件`上的扩展方法`IClientBuilder公司`和`IsiloHostBuilder公司`是的。这个`配置应用程序部件`方法接受委托，`操作<iapplicationpartmanager>`是的。

上的以下扩展方法`IApplicationPartManager`支持常见用途：

-   `addapplicationpart（程序集）`使用此扩展方法可以添加单个程序集。
-   `AddFromAppdomain（）`添加当前加载在`应用程序域`是的。
-   `AddFromApplicationBaseDirectory（）`加载并添加当前基路径中的所有程序集（请参见`appdomain.basedirectory目录`）中。

通过上述方法添加的程序集可以在其返回类型上使用以下扩展方法进行补充，`带程序集的IApplicationPartManager`：

-   `引用（）`从添加的零件添加所有引用的部件。这将立即加载任何可传递引用的程序集。忽略程序集加载错误。
-   `使用代码生成（）`为添加的部件生成支持代码并将其添加到部件管理器中。注意，这需要`Microsoft.Orleans.Orleanscodegenerator公司`要安装的包，通常称为运行时代码生成。

类型发现要求提供的应用程序部分包含特定属性。添加生成时代码生成包（`Microsoft.Orleans.CodeGenerator.MSBuild`或`Microsoft.Orleans.OrleansCodeGenerator.Build公司`）对于包含粒度、粒度接口或序列化程序的每个项目，建议使用确保这些属性存在的方法。生成生成时代码只支持C。对于F、Visual Basic和其他.NET语言，可以在配置期间通过`使用代码生成（）`上述方法。有关代码生成的更多信息，请参见[相应部分](../../grains/code_generation.md)是的。
