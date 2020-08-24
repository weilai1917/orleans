---
layout: page
title: Orleans Streams Quick Start
---

# OrleansStreams快速入门

本指南将向您展示设置和使用Orleans Streams的快速方法。要了解有关流功能的详细信息，请阅读本文档的其他部分。

## 所需配置

在本指南中，我们将使用基于简单消息的流，该流使用Grains消息传递将流数据发送给订阅者。我们将使用内存中的存储提供程序来存储订阅列表，因此对于实际的生产应用程序来说，这不是明智的选择。

在silos上，其中hostBuilder是ISiloHostBuilder

```csharp
hostBuilder.AddSimpleMessageStreamProvider("SMSProvider")
           .AddMemoryGrainStorage("PubSubStore");
```

在群集客户端上，其中clientBuilder是IClientBuilder

```csharp
clientBuilder.AddSimpleMessageStreamProvider("SMSProvider");
```

现在，我们可以创建流，使用它们作为生产者发送数据，也可以作为订阅者接收数据。

## 生产活动

为流生成事件相对容易。您应该首先访问在上述配置中定义的流提供程序（`短信提供商`），然后选择一个流并将数据推送到该流。

```csharp
//Pick a guid for a chat room grain and chat room stream
var guid = some guid identifying the chat room
//Get one of the providers which we defined in config
var streamProvider = GetStreamProvider("SMSProvider");
//Get the reference to a stream
var stream = streamProvider.GetStream<int>(guid, "RANDOMDATA");
```

如您所见，我们的流具有GUID和名称空间。这将使识别唯一流变得容易。例如，一个聊天室的名称空间可以是“ Rooms”，GUID可以是拥有的RoomGrain的GUID。

在这里，我们使用一些已知聊天室的GUID。现在使用`OnNext`流的方法，我们可以将数据推送到它。让我们在计时器内并使用随机数进行操作。您也可以对流使用任何其他数据类型。

```csharp
RegisterTimer(s =>
        {
            return stream.OnNextAsync(new System.Random().Next());
        }, null, TimeSpan.FromMilliseconds(1000), TimeSpan.FromMilliseconds(1000));
```

## 订阅和接收流数据

为了接收数据，我们可以使用隐式/显式订阅，这在手册的其他页面中有完整介绍。在这里，我们使用隐式订阅，这更容易。当Grains类型想要隐式订阅流时，它使用属性`ImplicitStreamSubscription（命名空间）]`。

对于我们的情况，我们将定义如下的ReceiverGrain：

```csharp
[ImplicitStreamSubscription("RANDOMDATA")]
public class ReceiverGrain : Grain, IRandomReceiver
```

现在，每当像计时器中一样将某些数据推送到名称空间RANDOMDATA的流中时，`接收器grain`具有相同GUID的流将接收到该消息。即使当前不存在任何激活的Grains，运行时也会自动创建一个新的Grains并将消息发送给它。

为了使其正常工作，我们需要通过设置我们的订阅来完成订阅过程`OnNext`接收数据的方法。所以我们`接收器grain`应该打电话给它`OnActivateAsync`像这样的东西

```csharp
//Create a GUID based on our GUID as a grain
var guid = this.GetPrimaryKey();
//Get one of the providers which we defined in config
var streamProvider = GetStreamProvider("SMSProvider");
//Get the reference to a stream
var stream = streamProvider.GetStream<int>(guid, "RANDOMDATA");
//Set our OnNext method to the lambda which simply prints the data, this doesn't make new subscriptions
await stream.SubscribeAsync<int>(async (data, token) => Console.WriteLine(data));
```

我们都准备好了。唯一的要求是，某些事情会触发我们的生产者Grains的创建，然后它将注册计时器并开始向所有感兴趣的方发送随机整数。

同样，本指南跳过了许多细节，仅适合于展示大图。阅读本手册的其他部分以及有关RX的其他资源，以更好地了解可用的内容和方法。

反应式编程可以是解决许多问题的非常有效的方法。例如，您可以在订户中使用LINQ来过滤数字并做各种有趣的事情。

## 下一个

[Orleans流编程API](streams_programming_APIs.md)
