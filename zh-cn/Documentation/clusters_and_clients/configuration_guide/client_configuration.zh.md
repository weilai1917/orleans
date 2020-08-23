---
layout: page
title: Client Configuration
---

> [啊！注意!]如果只想启动本地silos和本地客户端进行开发，请查看本地开发配置页。

# 客户端配置

连接到silos集群并向Grains发送请求的客户端是通过`客户端生成器`以及一些补充选项类。与思洛选项一样，客户端选项类遵循[ASP.NET选项](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)是的。

客户端配置有几个关键方面：

-   Orleans聚类信息
-   群集提供程序
-   应用程序部件

客户端配置示例：

```csharp
var client = new ClientBuilder()
    // Clustering information
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "my-first-cluster";
        options.ServiceId = "MyOrleansService";
    })
    // Clustering provider
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    // Application parts: just reference one of the grain interfaces that we use
    .ConfigureApplicationParts(parts => parts.AddApplicationPart(typeof(IValueGrain).Assembly))
    .Build();
```

让我们分解此示例中使用的步骤：

## Orleans聚类信息

```csharp
    [...]
    // Clustering information
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "orleans-docker";
        options.ServiceId = "AspNetSampleApp";
    })
    [...]
```

这里我们设定了两件事：

-   这个`棒状的`到`“我的第一个群集”`：这是Orleans集群的唯一ID。使用此ID的所有客户端和silos将能够直接相互通信。有些人会选择使用不同的`棒状的`例如，对于每个部署。
-   这个`服务ID`到`“aspnetsampleap”`：这是应用程序的唯一ID，某些提供程序（例如，持久性提供程序）将使用它。**此ID在部署期间应该是稳定的（而不是更改的）**是的。

## 群集提供程序

```csharp
    [...]
    // Clustering provider
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    [...]
```

客户端将使用此提供程序发现群集中可用的所有网关。有几个提供程序可用，在此示例中，我们使用azure表提供程序。

要获得更多详细信息，请查看服务器配置页中的匹配部分。

## 应用程序部件

```csharp
    [...]
    // Application parts: just reference one of the grain interfaces that we use
    .ConfigureApplicationParts(parts => parts.AddApplicationPart(typeof(IValueGrain).Assembly)).WithReferences())
    [...];
```

要获得更多详细信息，请查看服务器配置页中的匹配部分。
