---
layout: page
title: Startup Tasks
---

# 启动任务

在许多情况下，某些任务需要在思洛存储器可用时自动执行。*启动任务*提供此功能。

一些用例包括但不限于：

-   启动后台计时器以执行定期内务处理任务
-   使用从外部备份存储下载的数据预加载一些缓存颗粒

启动期间从启动任务引发的任何异常都将在思洛存储器日志中报告，并将停止思洛存储器。

这种快速故障方法是Orleans处理思洛存储器启动问题的标准方法，其目的是允许在测试阶段轻松检测思洛存储器配置和/或引导逻辑的任何问题，而不是在思洛存储器生命周期的后期被忽略并导致意外问题。

## 配置启动任务

启动任务可以使用`IsiloHostBuilder公司`通过注册要在启动期间调用的委托，或通过注册`伊斯塔普塔斯克`是的。

### 示例：注册委托

```csharp
siloHostBuilder.AddStartupTask(
  async (IServiceProvider services, CancellationToken cancellation) =>
  {
    // Use the service provider to get the grain factory.
    var grainFactory = services.GetRequiredService<IGrainFactory>();

    // Get a reference to a grain and call a method on it.
    var grain = grainFactory.GetGrain<IMyGrain>(0);
    await grain.Initialize();
});
```

### 示例：注册`伊斯塔普塔斯克`实施

首先，我们必须定义`伊斯塔普塔斯克`以下内容：

```csharp
public class CallGrainStartupTask : IStartupTask
{
    private readonly IGrainFactory grainFactory;

    public CallGrainStartupTask(IGrainFactory grainFactory)
    {
        this.grainFactory = grainFactory;
    }

    public async Task Execute(CancellationToken cancellationToken)
    {
        var grain = this.grainFactory.GetGrain<IMyGrain>(0);
        await grain.Initialize();
    }
}
```

然后必须在`IsiloHostBuilder公司`以下内容：

```csharp
siloHostBuilder.AddStartupTask<CallGrainStartupTask>();
```
