---
layout: page
title: Orleans Stream Providers
---

# 流提供者

溪流可以有不同的形状和形式。一些流可能通过直接TCP链接传递事件，而另一些流则通过持久队列传递事件。不同的流类型可能使用不同的批处理策略，不同的缓存算法或不同的反压程序。为避免将流应用程序限制为仅这些行为选择的一部分，**流提供者**是Orleans Streaming Runtime的可扩展性点，允许用户实现任何类型的流。这个扩展点在精神上类似于[奥尔良存储提供商](https://github.com/dotnet/orleans/wiki/Custom%20Storage%20Providers)。奥尔良目前与许多流提供程序一起提供，包括：[简单消息流提供者](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/SimpleMessageStream/SimpleMessageStreamProvider.cs)和[Azure队列流提供程序](https://github.com/dotnet/orleans/tree/master/src/Azure/Orleans.Streaming.AzureStorage/Providers/Streams/AzureQueue)。

## 简单消息流提供者

简单消息流提供程序，也称为SMS提供程序，通过利用常规的奥尔良谷物消息传递通过TCP传递事件。由于SMS中的事件是通过不可靠的TCP链接传递的，因此SMS可以*不*确保可靠的事件传递，并且不会自动重新发送SMS流失败的消息。默认情况下，生产者的调用`stream.OnNextAsync`返回一个`任务`代表流使用者的处理状态，它告诉生产者使用者是否成功接收并处理了事件。如果此任务失败，则生产者可以决定再次发送同一事件，从而在应用程序级别上实现可靠性。尽管流消息传递是尽力而为的，但SMS流本身是可靠的。也就是说，Pub-Sub执行的订户到生产者绑定是完全可靠的。

## Azure队列（AQ）流提供程序

Azure队列（AQ）流提供程序通过Azure队列传递事件。在生产者端，AQ流提供程序将事件直接排队到Azure队列中。在消费者方面，AQ Stream Provider管理一组**拉剂**从一组Azure队列中提取事件，并将事件传递给使用事件的应用程序代码。可以将拉动代理视为一种分布式“微服务”，即一种分区的，高度可用的弹性分布式组件。牵引剂在承载应用程序粒度的相同筒仓中运行。因此，无需运行单独的Azure工作角色来从队列中拉出。拉动代理程序，它们的管理，背压，平衡它们之间的队列以及将队列从故障代理程序切换到另一个代理程序的存在由Orleans Streaming Runtime完全管理，并且对于使用流的应用程序代码是透明的。

## 队列适配器

通过持久队列传递事件的不同流提供程序表现出相似的行为，并受到相似的实现。因此，我们提供了通用的可扩展[`PersistentStreamProvider`](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/PersistentStreams/PersistentStreamProvider.cs)允许开发人员插入不同类型的队列，而无需从头开始编写全新的流提供程序。`PersistentStreamProvider`使用一个[`IQueueAdapter`](https://github.com/dotnet/orleans/blob/master/src/Orleans.Core/Streams/QueueAdapters/IQueueAdapter.cs)组件，它抽象特定的队列实现细节，并提供使事件入队和出队的方法。其余的全部由内部逻辑处理`PersistentStreamProvider`。上面提到的Azure队列提供程序也以这种方式实现：它是`PersistentStreamProvider`使用一个`AzureQueueAdapter`。

## 下一个

[奥尔良流实施细节](../implementation/streams_implementation.md)
