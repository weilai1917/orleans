---
layout: page
title: Using Consul as a Membership Provider
---

# 使用consul作为成员资格提供程序

## 领事介绍

[领事](https://www.consul.io)是一个分布式、高可用和数据中心感知的服务发现平台，包括简单的服务注册、运行状况检查、故障检测和密钥/值存储。它是在数据中心的每个节点都运行一个consu代理的前提下构建的，consun代理可以充当服务器，也可以作为客户端，通过一个可伸缩的八卦协议进行通信。

consul有一个非常详细的概述，包括与类似解决方案的比较[在这里](https://www.consul.io/intro/index.html)是的。

执政官写的是go[开源](https://github.com/hashicorp/consul)；编译的下载可用于[Mac OS X、FreeBSD、Linux、solaris和Windows](https://www.consul.io/downloads.html)

## 为什么选择领事？

作为[Orleans会员服务商](../implementation/cluster_management.md)，当您需要提供**内部解决方案**这并不要求潜在客户拥有现有的基础设施。**和**一家合作的it提供商。consul是一个非常轻量级的单一可执行文件，没有依赖关系，因此可以很容易地构建到您自己的中间件解决方案中。当consul已经是您发现、检查和维护微服务的解决方案时，完全集成orleans成员关系以简化操作是有意义的。因此，我们在consul(也称为“orleans custom system store”)中实现了一个成员表，它与orleans的[群集管理](../implementation/cluster_management.md)是的。

## 设立领事

有很多关于[IO领事](https://www.consul.io)关于建立一个稳定的领事集群，在这里重复是没有意义的；但是为了您的方便，我们提供了这个指南，这样您可以很快让Orleans运行一个独立的领事代理。

1)创建一个文件夹以将consul安装到其中，例如c:\\ consul

2)创建子文件夹：C:\\Cuxel\\Data(如果不存在，Cuxl将不创建此文件)

三)[下载](https://www.consul.io/downloads.html)并将consul.exe解压到c:\\ consul\\

4)在C:\\ consul打开命令提示符\\

5)进入`consun.exe代理-服务器-引导-数据目录“C:\ consun\data”-客户端=0.0.0.0`

`代理人`指示consul运行托管服务的代理进程。如果不这样做，consul进程将尝试使用rpc来配置正在运行的代理。

`-服务器`将代理定义为服务器而不是客户端(consul*客户*是托管所有服务和数据的代理，但没有投票权来决定，并且不能成为群集的领导者

`-自举`第一个(而且只有第一个！)必须引导群集中的节点，使其承担群集领导地位。

`-数据目录[路径]`指定存储所有consun数据的路径，包括cluster membership表

`-客户端=0.0.0.0`通知领事在哪个IP上打开服务。

还有许多其他参数，以及使用json配置文件的选项。有关选项的完整列表，请参阅consul文档。

6)通过打开[服务](http://localhost:8500/v1/catalog/services)浏览器中的终结点。

## Orleans的结构

### 服务器

“自定义”成员资格提供程序orleansconfiguration.xml配置文件当前存在一个已知问题，无法正确解析该文件。因此，在启动silos之前，必须在XML中提供一个占位符SystemStore，然后在代码中配置提供程序。

**orleansconfiguration.xml文件**

```xml
<OrleansConfiguration xmlns="urn:orleans">
    <Globals>
        <SystemStore SystemStoreType="None" DataConnectionString="http://localhost:8500" DeploymentId="MyOrleansDeployment" />
    </Globals>
    <Defaults>
        <Networking Address="localhost" Port="22222" />
        <ProxyingGateway Address="localhost" Port="30000" />
    </Defaults>    
</OrleansConfiguration>
```

**代码**

```csharp
public void Start(ClusterConfiguration config)
{
    _siloHost = new SiloHost(System.Net.Dns.GetHostName(), config);

    _siloHost.Config.Globals.LivenessType = GlobalConfiguration.LivenessProviderType.Custom;
    _siloHost.Config.Globals.MembershipTableAssembly = "OrleansConsulUtils";
    _siloHost.Config.Globals.ReminderServiceType = GlobalConfiguration.ReminderServiceProviderType.Disabled;

    _siloHost.InitializeOrleansSilo();
    var startedok = _siloHost.StartOrleansSilo();
    if (!startedok)
        throw new SystemException(String.Format("Failed to start Orleans silo '{0}' as a {1} node", _siloHost.Name, _siloHost.Type));

    Log.Information("Orleans Silo is running.\n");
}
```

或者，您可以完全用代码配置silos。

### 顾客

客户端配置要简单得多

**客户端配置.xml**

```xml
<ClientConfiguration xmlns="urn:orleans">
    <SystemStore SystemStoreType="Custom" CustomGatewayProviderAssemblyName="OrleansConsulUtils" DataConnectionString="http://192.168.1.26:8500" DeploymentId="MyOrleansDeployment" />
</ClientConfiguration>
```

## 客户端sdk

如果您有兴趣使用领事馆为您自己的服务发现有[客户SDK](https://www.consul.io/downloads_tools.html)最流行的语言。

## 实施细节

成员资格表提供程序使用[领事钥匙/价值商店](https://www.consul.io/intro/getting-started/kv.html)cas的功能。当每个silos启动时，它会注册两个KV条目，一个包含silos的详细信息，另一个保存silos上次报告它处于活动状态时的信息(后者是指诊断“I am alive”条目，而不是指直接在silos之间发送且未写入表中的故障检测Hearbeats)。根据orleans的需要，对表的所有写操作都由cas执行，以提供并发控制。[群集管理协议](../implementation/cluster_management.md)是的。silos运行后，您可以在Web浏览器中查看这些条目[在这里](http://localhost:8500/v1/kv/?keys)，将显示如下内容：

```js
[
    "orleans/MyOrleansDeployment/192.168.1.26:11111@191780753",
    "orleans/MyOrleansDeployment/192.168.1.26:11111@191780753/iamalive"
]
```

您会注意到这些键的前缀是*“Orleans/”*这是在提供者中硬编码的，旨在避免与consul的其他用户发生密钥空间冲突。每个键都可以通过附加它们的键名来读取*(当然没有引用)*致[领事KV根](http://localhost:8500/v1/kv/)是的。这样做将为您提供以下信息：

```js
[
    {
        "LockIndex": 0,
        "Key": "orleans/MyOrleansDeployment/192.168.1.26:22222@191780753",
        "Flags": 0,
        "Value": "[BASE64 UTF8 Encoded String]",
        "CreateIndex": 10,
        "ModifyIndex": 12
    }
]
```

解码字符串将提供实际的Orleans成员资格数据：

**<http://localhost:8500/v1/KV/orleans/MyOrleansDeployment/[SiloAddress>]**

```
{
    "Hostname": "[YOUR_MACHINE_NAME]",
    "ProxyPort": 22222,
    "StartTime": "2016-01-29T16:25:54.9538838Z",
    "Status": 3,
    "SuspectingSilos": []
}
```

**<http://localhost:8500/v1/KV/orleans/MyOrleansDeployment/[SiloAddress]/IAmAlive>**

```
"2016-01-29T16:35:58.9193803Z"
```

当客户端连接时，他们通过使用uri在一个http get中读取集群中所有silos的kvs`http://192.168.1.26:8500/v1/kv/orleans/myorleansdeployment/？递归`是的。

## 限制

### Orleans扩展成员协议(表版本和ETag)

consul kv currently目前不支持原子更新。因此，orleans consul成员资格提供者只实现了orleans基本成员资格协议，如前所述[在这里](../implementation/cluster_management.md)不支持扩展成员身份协议。这个扩展的协议被引入作为一个额外的，但不是必需的，silos连通性验证和作为尚未实现的功能的基础。如果您的基础设施配置正确，您将不会因缺乏支持而受到任何不利影响。

### 多个数据中心

consun中的键值对当前未在consun数据中心之间复制。有一个[单独项目](https://github.com/hashicorp/consul-replicate)为了解决这个问题，但它还没有被证明支持Orleans。

### 在Windows上运行时

当consul在windows上启动时，它会记录以下消息：

```
==> WARNING: Windows is not recommended as a Consul server. Do not use in production.
```

这仅仅是因为在windows环境中运行时缺乏对测试的关注，而不是因为任何实际的已知问题。阅读[此处讨论](https://groups.google.com/forum/#!topic/consul-tool/DvXYgZtUZyU)在决定领事是否适合你之前。

## 未来的潜在增强

1)证明consun-kv复制项目能够在多个consun数据中心之间的广域网环境中支持一个orleans集群。

2)在领事馆执行提醒表。

3)实现扩展成员协议。consul背后的团队确实计划实现原子操作，一旦此功能可用，就可以消除提供者中的限制。
