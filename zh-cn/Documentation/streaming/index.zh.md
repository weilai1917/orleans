---
layout: page
title: Orleans Streams
---

# OrleansStreams

Orleans v.1.0.0增加了对编程模型的流扩展的支持。流扩展提供了一组抽象和API，使对流的思考和使用变得更简单，更可靠。流扩展允许开发人员编写以结构化方式对一系列事件进行操作的响应式应用程序。流提供程序的可扩展性模型使编程模型可与多种现有排队技术兼容并可移植，例如[活动中心](http://azure.microsoft.com/en-us/services/event-hubs/)，[服务总线](http://azure.microsoft.com/en-us/services/service-bus/)，[Azure队列](http://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-how-to-use-queues/)和[阿帕奇·卡夫卡](http://kafka.apache.org/)。无需编写特殊代码或运行专用进程来与此类队列进行交互。

## 我为什么要在乎？

如果您已经知道所有[流处理](https://confluentinc.wordpress.com/2015/01/29/making-sense-of-stream-processing/)并且熟悉诸如[活动中心](http://azure.microsoft.com/en-us/services/event-hubs/)，[卡夫卡](http://kafka.apache.org/)，[Azure流分析](http://azure.microsoft.com/en-us/services/stream-analytics/)，[阿帕奇风暴](https://storm.apache.org/)，[Apache Spark流](https://spark.apache.org/streaming/)和[.NET中的反应性扩展（Rx）](https://msdn.microsoft.com/en-us/data/gg577609.aspx)，您可能会问为什么要关心。**为什么我们还需要另一个流处理系统，以及Actor与流之间的关系？** [“为什么OrleansStreams？”](streams_why.md)是要回答这个问题。

## 程式设计模型

Orleans Streams编程模型背后有许多原则。

1.  OrleansStreams是*虚拟*。即，流始终存在。它不是显式创建或销毁的，它永远不会失败。
2.  流是*由...确定*流ID，仅*逻辑名称*由GUID和字符串组成。
3.  OrleansStreams允许*在时间和空间上将数据生成与其处理脱钩*。这意味着流生产者和流使用者可能位于不同的服务器上，处于不同的时间，并且将承受故障。
4.  OrleansStreams是*轻巧而动态*。Orleans Streaming Runtime旨在处理大量高速率来来去去的流。
5.  Orleans流*绑定是动态的*。Orleans Streaming Runtime旨在处理Grains以高速率连接到流或从流中断开的情况。
6.  Orleans流媒体运行时*透明地管理流消耗的生命周期*。应用程序订阅流之后，从那时起，即使存在故障，它也将接收流的事件。
7.  OrleansStreams*跨Grains和Orleans客户统一工作*。

## 编程API

应用程序通过与众所周知的API非常相似的API与流进行交互[.NET中的反应性扩展（Rx）](https://msdn.microsoft.com/en-us/data/gg577609.aspx)， 通过使用[`Orleans.Streams.IAsyncStream <T>`](https://github.com/dotnet/orleans/blob/master/src/Orleans/Streams/Core/IAsyncStream.cs)实现\
[`Orleans.Streams.IAsyncObserver <T>`](https://github.com/dotnet/orleans/blob/master/src/Orleans/Streams/Core/IAsyncObserver.cs)和[`Orleans.Streams.IAsyncObservable <T>`](https://github.com/dotnet/orleans/blob/master/src/Orleans/Streams/Core/IAsyncObservable.cs)接口。

在下面的典型示例中，设备生成一些数据，这些数据作为HTTP请求发送到云中运行的服务。在前端服务器中运行的Orleans客户端收到此HTTP调用，并将数据发布到匹配的设备流中：

```csharp
public async Task OnHttpCall(DeviceEvent deviceEvent)
{
     // Post data directly into device's stream.
     IStreamProvider streamProvider = GrainClient.GetStreamProvider("myStreamProvider");
     IAsyncStream<DeviceEventData> deviceStream = streamProvider.GetStream<DeviceEventData>(deviceEvent.DeviceId);
     await deviceStream.OnNextAsync(deviceEvent.Data);
}
```

在下面的另一个示例中，聊天用户（实现为Orleans Grain）加入聊天室，获取该房间中所有其他用户生成的聊天消息流的句柄并进行订阅。请注意，聊天用户不需要了解聊天室的粒度本身（我们的系统中可能没有这种粒度）或该组中产生消息的其他用户。不用说，要生成聊天流，用户无需知道当前订阅了谁。这说明了如何将聊天用户在时间和空间上完全分离。

```csharp
public class ChatUser: Grain
{
    public async Task JoinChat(string chatGroupName)
    {
       IStreamProvider streamProvider = base.GetStreamProvider("myStreamProvider");
       IAsyncStream<string> chatStream = streamProvider.GetStream<string>(chatGroupName);
       await chatStream.SubscribeAsync((string chatEvent) => Console.Out.Write(chatEvent));
    }
}
```

* * *

## 快速入门样本

的[快速入门样本](streams_quick_start.md)是对在应用程序中使用流的总体工作流程的快速概述。阅读后，您应该阅读[流编程API](streams_programming_APIs.md)对概念有更深入的了解。

## 流编程API

一个[流编程API](streams_programming_APIs.md)提供了有关编程API的详细说明。

## 流提供者

流可以通过各种形状和形式的物理通道出现，并且可以具有不同的语义。Orleans Streaming旨在通过以下概念支持这种多样性**流提供者**，这是系统中的可扩展点。Orleans目前有两个流提供程序的实现：基于TCP**简单消息流提供者**和基于Azure队列**Azure队列流提供程序**。有关Steam Providers的更多详细信息，请访问：[流提供者](stream_providers.md)。

## 流语义

**流订阅语义**：Orleans Streams保证Stream Subscription操作的顺序一致性。具体来说，当消费者订阅流时，一旦`任务`如果代表订阅操作已成功解决，则使用者将看到订阅后生成的所有事件。此外，可倒带流允许通过使用以下内容从过去的任意时间点进行订阅`StreamSequenceToken`（可以找到更多详细信息[这里](stream_providers.md)）。

**个人流事件交付保证**：单个事件传递的保证取决于单个流提供者。一些提供仅尽最大可能的一次传送（例如简单消息流（SMS）），而其他一些提供至少一次的传送（例如Azure队列流）。甚至可以构建一个将保证一次交付的流提供程序（我们还没有这样的提供程序，但是可以构建一个）。

**活动交付单**：事件顺序还取决于特定的流提供程序。在SMS流中，生产者通过控制其发布方式来显式控制消费者看到的事件的顺序。Azure队列流不保证FIFO顺序，因为基础Azure队列在故障情况下不保证顺序。应用程序还可以通过使用以下命令控制自己的流交付顺序`StreamSequenceToken`。

## 流实施

的[Orleans流实施](../implementation/streams_implementation.md)提供了内部实现的高级概述。

## 代码样本

可以找到有关如何在Grains中使用流式API的更多示例。[这里](https://github.com/dotnet/orleans/blob/master/test/Grains/TestGrains/SampleStreamingGrain.cs)。我们计划在将来创建更多样本。

## 更多材料

-   [关于流的Orleans虚拟聚会](https://www.youtube.com/watch?v=eSepBlfY554)
-   [虚拟聚会的Orleans流媒体演示](http://dotnet.github.io/orleans/Presentations/Orleans%20Streaming%20-%20Virtual%20meetup%20-%205-22-2015.pptx)
