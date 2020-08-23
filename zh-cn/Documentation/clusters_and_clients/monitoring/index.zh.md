---
layout: page
title: Runtime Monitoring
---

# 运行时监视

Orleans通过`ITelemetryConsumer公司`接口。应用程序可以向一个或多个遥测用户注册它们的silos和客户机，以接收Orleans Runtime Periotic发布的统计信息和度量。它们可以是流行的遥测分析解决方案的消费者，也可以是任何其他目的地和用途的自定义解决方案的消费者。三个遥测用户目前包含在Orleans代码库中。

它们作为单独的nuget包发布：

-   `Microsoft.Orleans.OrleanstelemtryConsumers.ai公司`发布到[应用洞察](https://azure.microsoft.com/en-us/services/application-insights/)是的。

-   `Microsoft.Orleans.OrleanstelemtryConsumers.Counters公司`用于发布到Windows性能计数器。Orleans运行时会不断更新其中的一些。CounterControl.exe工具，包含在[`Microsoft.Orleans.CounterControl公司`](https://www.nuget.org/packages/Microsoft.Orleans.CounterControl/)nuget包，帮助注册必要的性能计数器类别。它必须以提升的权限运行。可以使用任何标准监视工具监视性能计数器。

-   `microsoft.orleans.orleanstelemtryconsumers.newrelic公司`，用于发布到[新文物](https://newrelic.com/)是的。

要将silos和客户端配置为使用遥测用户，silos配置代码如下所示：

```c#
var siloHostBuilder = new SiloHostBuilder();
//configure the silo with AITelemetryConsumer
siloHostBuilder.AddApplicationInsightsTelemetryConsumer("INSTRUMENTATION_KEY");
```

客户端配置代码如下所示：

```c#
var clientBuilder = new ClientBuilder();
//configure the clientBuilder with AITelemetryConsumer
clientBuilder.AddApplicationInsightsTelemetryConsumer("INSTRUMENTATION_KEY");
```
