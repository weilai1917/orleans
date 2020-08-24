---
layout: page
title: Typical Configurations
---

# 典型配置

下面是可用于开发和生产部署的典型配置示例。

## 地方发展

见[本地开发配置](local_development_configuration.md)

## 使用azure的可靠生产部署

要使用azure进行可靠的生产部署，需要使用azure表选项作为群集成员身份。此配置是部署到本地服务器、容器或azure虚拟机实例的典型配置。

数据连接字符串的格式为`“DefaultEndpointsProtocol=https；帐户名=<azure存储帐户>；帐户密钥=<azure表存储帐户密钥>”`

silos配置：

```csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var silo = new SiloHostBuilder()
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "Cluster42";
        options.ServiceId = "MyAwesomeService";
    })
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    .ConfigureEndpoints(siloPort: 11111, gatewayPort: 30000)
    .ConfigureLogging(builder => builder.SetMinimumLevel(LogLevel.Warning).AddConsole())
    .Build();
```

客户端配置：

```csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var client = new ClientBuilder()
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "Cluster42";
        options.ServiceId = "MyAwesomeService";
    })
    .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
    .ConfigureLogging(builder => builder.SetMinimumLevel(LogLevel.Warning).AddConsole())
    .Build();
```

## 使用SQL Server进行可靠的生产部署

为了使用SQL Server进行可靠的生产部署，需要提供SQL Server连接字符串。

silos配置：

```csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var silo = new SiloHostBuilder()
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "Cluster42";
        options.ServiceId = "MyAwesomeService";
    })
    .UseAdoNetClustering(options =>
    { 
      options.ConnectionString = connectionString;
      options.Invariant = "System.Data.SqlClient";
    })
    .ConfigureEndpoints(siloPort: 11111, gatewayPort: 30000)
    .ConfigureLogging(builder => builder.SetMinimumLevel(LogLevel.Warning).AddConsole())
    .Build();
```

客户端配置：

```csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var client = new ClientBuilder()
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "Cluster42";
        options.ServiceId = "MyAwesomeService";
    })
    .UseAdoNetClustering(options =>
    { 
      options.ConnectionString = connectionString;
      options.Invariant = "System.Data.SqlClient";
    })
    .ConfigureLogging(builder => builder.SetMinimumLevel(LogLevel.Warning).AddConsole())
    .Build();
```

## 在专用服务器群集上部署不可靠

对于在不考虑可靠性的情况下在专用服务器集群上进行测试，可以利用MembershipTableGrain并避免对azure表的依赖。您只需要将其中一个节点指定为主节点。

在silos上：

```csharp
var primarySiloEndpoint = new IPEndpoint(PRIMARY_SILO_IP_ADDRESS, 11111);
var silo = new SiloHostBuilder()
  .UseDevelopmentClustering(primarySiloEndpoint)
  .Configure<ClusterOptions>(options =>
  {
    options.ClusterId = "Cluster42";
    options.ServiceId = "MyAwesomeService";
  })
  .ConfigureEndpoints(siloPort: 11111, gatewayPort: 30000)
  .ConfigureLogging(logging => logging.AddConsole())
  .Build();
```

对客户：

```csharp
var gateways = new IPEndPoint[]
{
    new IPEndPoint(PRIMARY_SILO_IP_ADDRESS, 30000),
    new IPEndPoint(OTHER_SILO__IP_ADDRESS_1, 30000),
    [...]
    new IPEndPoint(OTHER_SILO__IP_ADDRESS_N, 30000),
};
var client = new ClientBuilder()
    .UseStaticClustering(gateways)
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "dev";
        options.ServiceId = "AdventureApp";
    })
    .ConfigureLogging(logging => logging.AddConsole())
    .Build();
```
