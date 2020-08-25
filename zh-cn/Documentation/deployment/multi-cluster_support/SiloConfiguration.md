---
layout: page
title: Multi-Cluster Silo Configuration
---

## Orleanssilos配置

为了快速了解，我们在下面的XML语法中显示了所有相关的配置参数(包括可选参数)：

```html
<?xml version="1.0" encoding="utf-8"?>
<OrleansConfiguration xmlns="urn:orleans">
  <Globals>
    <MultiClusterNetwork
      ClusterId="clusterid"
      DefaultMultiCluster="uswest,europewest,useast"
      BackgroundGossipInterval="30s"
      UseGlobalSingleInstanceByDefault="false"
      GlobalSingleInstanceRetryInterval="30s"
      GlobalSingleInstanceNumberRetries="3"
      MaxMultiClusterGateways="10">
         <GossipChannel  Type="..."  ConnectionString="..."/>      
         <GossipChannel  Type="..."  ConnectionString="..."/>      
    </MultiClusterNetwork>
    <SystemStore ... ServiceId="some-guid" .../>
  </Globals>
</OrleansConfiguration>
```

```csharp
var silo = new SiloHostBuilder()
  [...]
  .Configure<ClusterInfo>(options =>
  {
    options.ClusterId = "us3";
    options.ServiceId = "myawesomeservice";
  })
  .Configure<MultiClusterOptions>(options => 
  {
    options.HasMultiClusterNetwork = true;
    options.DefaultMultiCluster = new[] { "us1", "eu1", "us2" };
    options.BackgroundGossipInterval = TimeSpan.FromSeconds(30);
    options.UseGlobalSingleInstanceByDefault = false;
    options.GlobalSingleInstanceRetryInterval = TimeSpan.FromSeconds(30);
    options.GlobalSingleInstanceNumberRetries = 3;
    options.MaxMultiClusterGateways = 10;
    options.GossipChannels.Add("AzureTable", "DefaultEndpointsProtocol=https;AccountName=usa;AccountKey=...");
    options.GossipChannels.Add("AzureTable", "DefaultEndpointsProtocol=https;AccountName=europe;AccountKey=...")
    [...]
  })
  [...]
```

与往常一样，所有配置设置也可以通过`全球配置`上课。

这个`服务ID`是用于标识此服务的任意ID。所有集群和所有silos必须相同。

这个`多集群网络`节是可选的-如果不存在，则禁用此silos的所有多群集支持。

这个**所需参数** `棒状的`和`八卦频道`在[多集群通信](GossipChannels.md)是的。

可选参数`MaxMultiClusterGateways系列`和`背景消息间隔`在[多集群通信](GossipChannels.md)是的。

可选参数`默认多群集`在[多群集配置](MultiClusterConfiguration.md)是的。

可选参数`默认为UseGlobalSingleInstances`，请`GlobalSingleInstanceRetryInterval`和`GlobalSingleInstanceNumberEntries全局`在[全局单实例Grain](GlobalSingleInstance.md)是的。

## Orleans客户端配置

Orleans客户端不需要额外配置。同一客户端可能无法连接到不同集群中的silos(silos在这种情况下拒绝连接)。
