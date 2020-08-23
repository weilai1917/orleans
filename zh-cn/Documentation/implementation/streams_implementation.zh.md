---
layout: page
title: Streams Implementation Details
---

# 奥尔良流实施细节

本节提供了Orleans Stream实施的高级概述。它描述了在应用程序级别上不可见的概念和细节。如果仅计划使用流，则不必阅读本节。

*术语*：

我们用“队列”一词来指代任何可以吸收流事件并允许提取事件或提供基于推送的机制来使用事件的持久存储技术。通常，为了提供可伸缩性，这些技术提供了分片/分区队列。例如，Azure队列允许创建多个队列，事件中心具有多个中心，Kafka主题，...

## 持久流<a name="Persistent-Streams"></a>

所有奥尔良持久流提供者共享一个共同的实现[**`PersistentStreamProvider`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/PersistentStreams/PersistentStreamProvider.cs)。该通用流提供者需要配置有特定于技术的[**`IQueueAdapterFactory`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/QueueAdapters/IQueueAdapterFactory.cs)。

例如，出于测试目的，我们有队列适配器，它们生成自己的测试数据，而不是从队列中读取数据。下面的代码显示了我们如何配置持久流提供程序以使用我们的自定义（生成器）队列适配器。它通过使用用于创建适配器的工厂功能配置持久流提供程序来实现。

```csharp
hostBuilder.AddPersistentStreams(StreamProviderName, GeneratorAdapterFactory.Create);
```

当流生产者生成新的流项目并调用时`stream.OnNext（）`，Orleans Streaming Runtime在上调用适当的方法`IQueueAdapter`该流提供程序将条目直接排队到适当的队列中。

### 牵引剂<a name="Pulling-Agents"></a>

持久流提供者的核心是拉动代理。拉动代理程序从一组持久队列中拉出事件，然后将事件以消耗它们的方式传递给应用程序代码。可以将拉动代理视为一种分布式“微服务”，即一种分区的，高度可用的弹性分布式组件。拉动代理在托管应用程序粒度的相同筒仓中运行，并由Orleans Streaming Runtime完全管理。

### StreamQueueMapper和StreamQueueBalancer<a name="StreamQueueMapper-and-StreamQueueBalancer"></a>

牵引剂的参数设置为`IStreamQueueMapper`和`IStreamQueueBalancer`。[**`IStreamQueueMapper`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/QueueAdapters/IStreamQueueMapper.cs)提供所有队列的列表，还负责将流映射到队列。这样，持久流提供者的生产者方就知道将消息放入哪个队列中。

[**`IStreamQueueBalancer`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/PersistentStreams/IStreamQueueBalancer.cs)表示队列在奥尔良筒仓和特工之间平衡的方式。目标是以平衡的方式为代理分配队列，以防止瓶颈并支持弹性。将新的筒仓添加到Orleans集群后，新旧筒仓之间的队列会自动重新平衡。StreamQueueBalancer允许自定义该过程。奥尔良具有许多内置的StreamQueueBalancers，以支持不同的平衡方案（队列数量大而少）和不同的环境（Azure，Prem，静态）。

使用上面的测试生成器示例，以下代码显示了如何配置队列映射器和队列平衡器。

```csharp
hostBuilder
  .AddPersistentStreams(StreamProviderName, GeneratorAdapterFactory.Create,
    providerConfigurator=>providerConfigurator
      .Configure<HashRingStreamQueueMapperOptions>(ob=>ob.Configure(
        options=>{ options.TotalQueueCount = 8; }))
      .UseDynamicClusterConfigDeploymentBalancer()
);
```

上面的代码将GeneratorAdapter配置为使用具有8个队列的队列映射器，并使用`DynamicClusterConfigDeploymentBalancer`。

### 拉协议<a name="Pulling-Protocol"></a>

每个筒仓都运行一组拉动代理，每个代理都从一个队列中拉出。拉动代理程序本身由内部运行时组件（称为**SystemTarget**。SystemTargets本质上是运行时粒度，受单线程并发性的影响，可以使用常规的粒度消息传递，并且轻巧。与谷物相反，SystemTarget不是虚拟的：它们是由运行时显式创建的，并且位置不透明。通过将拉动代理实现为SystemTargets，Orleans Streaming Runtime可以依赖于内置的Orleans功能并可以扩展到大量队列，因为创建新的拉动代理与创建新的谷物一样便宜。

每个拉动代理都运行一个定期计时器，该定时从队列中拉出（通过调用[**`IQueueAdapterReceiver`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/QueueAdapters/IQueueAdapterReceiver.cs)）`GetQueueMessagesAsync（）`方法。返回的消息放在内部的每个代理的数据结构中，称为`IQueueCache`。检查每个消息以找出其目标流。代理使用Pub Sub来查找订阅此流的流使用者列表。检索到使用者列表后，代理会将其存储在本地（在其pub-sub缓存中），因此无需在每条消息上都与Pub Sub进行协商。代理还订阅pub-sub，以接收有关订阅该流的任何新使用者的通知。代理与pub-sub保证之间的这种握手**强大的流订阅语义**：*消费者订阅了流之后，它将看到订阅后生成的所有事件*。另外，使用`StreamSequenceToken`允许其过去订阅。

### 队列缓存<a name="Queue-Cache"></a>

[**`IQueueCache`**](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/QueueAdapters/IQueueCache.cs)是内部的每个代理程序数据结构，该结构允许将新事件从队列中分离出来并将它们传递给使用者。它还允许将传递到不同流和不同消费者的耦合解耦。

想象这样一种情况，其中一个流有3个流使用者，其中一个很慢。如果不注意，这种缓慢的使用者可能会影响代理的进度，减慢该流的其他使用者的消耗，甚至减慢其他流的事件的出队和传递。为了避免这种情况并允许代理中的最大并行度，我们使用`IQueueCache`。

`IQueueCache`缓冲流事件，并为代理提供一种以自己的节奏将事件传递给每个使用者的方式。每个消费者的交付都是通过内部组件实现的`IQueueCacheCursor`，它跟踪每个消费者的进度。这样，每个使用者就可以按照自己的节奏接收事件：快速的使用者在从队列中出队后就可以尽快接收事件，而较慢的使用者则在以后接收事件。一旦消息传递给所有使用者，就可以从缓存中将其删除。

### 背压<a name="Backpressure"></a>

奥尔良流运行时中的背压在两个地方适用：**将流事件从队列带到代理**和**将事件从代理传递到流消费者**。

后者由内置的Orleans消息传递机制提供。每个流事件都通过标准的Orleans谷物消息传递一次从代理传递到消费者。也就是说，代理将一个事件（或数量有限的事件）发送给每个单独的流使用者，并等待此呼叫。在解决或破坏上一个事件的任务之前，下一个事件将不会开始传递。这样一来，我们自然会将每次消费者的投放速度限制为一次只发送一条消息。

关于将流事件从队列传递到代理，Orleans Streaming提供了一种新的特殊背压机制。由于代理将事件的出队从队列中解耦并传递给使用者，因此单个缓慢的使用者可能会落后很多，以至于`IQueueCache`将填满。阻止`IQueueCache`为了避免无限期增长，我们限制了它的大小（大小限制是可配置的）。但是，代理永远不会丢弃未交付的事件。相反，当缓存开始填满时，代理会减慢事件从队列中出队的速度。这样，我们可以通过调整从队列中消耗的速率（“背压”）来“摆脱”缓慢的交付周期，然后稍后恢复为快速的消耗速率。要检测“缓慢投放”的山谷`IQueueCache`使用高速缓存存储区的内部数据结构来跟踪将事件交付给各个流使用者的进度。这导致了一个非常敏感和自我调整的系统。
