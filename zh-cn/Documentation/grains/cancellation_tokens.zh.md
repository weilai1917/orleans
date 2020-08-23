---
layout: page
title: Grain cancellation tokens
---

# Grains取消代币

orleans运行时提供了一种称为grain cancellation token的机制，使开发人员能够取消正在执行的grain操作。

## 说明

**GrainCancellationToken公司**是标准的包装**.NET系统.Threading.CancellationToken**，它启用线程、线程池工作项或任务对象之间的协作取消，并且可以作为grain方法参数传递。

一个**GrainCancellationTokenSource**是通过其token属性提供取消令牌并通过调用`取消`方法。

## 用法

-   实例化CancellationTokenSource对象，该对象管理并向各个取消令牌发送取消通知。

```csharp
        var tcs = new GrainCancellationTokenSource();
```

-   将GrainCancellationTokenSource.Token属性返回的令牌传递给侦听取消的每个Grain方法。

```csharp
        var waitTask = grain.LongIoWork(tcs.Token, TimeSpan.FromSeconds(10));
```

-   可取消的Grains操作需要处理底层**取消令牌**财产**GrainCancellationToken公司**就像在其他.NET代码中一样。

```csharp
        public async Task LongIoWork(GrainCancellationToken tc, TimeSpan delay)
        {
            while(!tc.CancellationToken.IsCancellationRequested)
            {
                 await IoOperation(tc.CancellationToken);
            }
        }
```

-   打电话给`GrainCancellationTokenSource.Cancel`方法启动取消。

```csharp
        await tcs.Cancel();
```

-   打电话给`处置`方法完成**GrainCancellationTokenSource**反对。

```csharp
        tcs.Dispose();
```

#### 重要注意事项：

-   这个`GrainCancellationTokenSource.Cancel`方法返回**任务**，并且为了确保取消，必须在短暂通信失败的情况下重试取消调用。
-   在基础中注册的回调**系统.threading.cancellationtoken**在注册它们的grain activation中受单线程执行保证的约束。
-   每个**GrainCancellationToken公司**可以通过多个方法调用传递。
