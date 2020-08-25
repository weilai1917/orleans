---
layout: page
title: Reentrancy
---

# 可重入

Grain激活是单线程的，默认情况下，在下一个请求可以开始处理之前，从头到尾处理每个请求。在某些情况下，当一个请求等待异步操作完成时，可能需要激活来处理其他请求。由于这个和其他原因，Orleans给了开发人员一些控制请求交错行为的权限。在以下情况下，多个请求可能被交错：

-   Grains等级标记为`[可重入]`
-   接口方法标记为`[总是在屋檐下]`
-   同一调用链中的请求
-   Grains的*五月交错*谓词返回`是的`

以下各节将讨论这些情况。

## 重入grains

`Grains`实现类可以用`[可重入]`属性指示不同的请求可以自由交错。

换言之，可重入激活可能在前一个请求尚未完成处理时开始执行另一个请求。执行仍然局限于单个线程，因此激活仍然一次执行一个回合，并且每个回合只代表激活的一个请求执行。

可重入Grain代码永远不会并行运行多个Grain代码（Grain代码的执行将始终是单线程的），但可重入Grain代码**可以**请参阅不同请求交错执行代码。也就是说，不同请求的续转可以交错。

例如，使用下面的伪代码，当Foo和Bar是同一grain类的2个方法时：

```csharp
Task Foo()
{
    await task1;    // line 1
    return Do2();   // line 2
}

Task Bar()
{
    await task2;   // line 3
    return Do2();  // line 4
}
```

如果这个grains有标记`[可重入]`，Foo和Bar的执行可以交错执行。

例如，可以按以下顺序执行：

1号线、3号线、2号线和4号线。也就是说，来自不同请求的圈数交错。

如果grain不可重入，唯一可能的执行将是：第1行、第2行、第3行、第4行或：第3行、第4行、第1行、第2行（在前一个请求完成之前，新请求无法启动）。

在选择可重入和不可重入grains时，主要的折衷是使交织正确工作的代码复杂度和对此进行推理的困难。

在一个很小的例子中，当Grain是无状态的，逻辑也很简单时，更少（但不是太少，以便使用所有的硬件线程）可重入grains通常会稍微更有效一些。

如果代码更复杂，那么大量的不可重入grains，即使总体效率稍低，也可以避免您在解决不明显的交错问题时的许多痛苦。

最终答案将取决于具体的应用程序。

## 交错法

grains接口法`[总是在屋檐下]`无论grains是否可重入，都将被交错。考虑以下示例：

```csharp
public interface ISlowpokeGrain : IGrainWithIntegerKey
{
    Task GoSlow();

    [AlwaysInterleave]
    Task GoFast();
}

public class SlowpokeGrain : Grain, ISlowpokeGrain
{
    public async Task GoSlow()
    {
        await Task.Delay(TimeSpan.FromSeconds(10));
    }

    public async Task GoFast()
    {
        await Task.Delay(TimeSpan.FromSeconds(10));
    }
}
```

现在考虑由以下客户端请求启动的调用流：

```csharp
var slowpoke = client.GetGrain<ISlowpokeGrain>(0);

// A) This will take around 20 seconds
await Task.WhenAll(slowpoke.GoSlow(), slowpoke.GoSlow());

// B) This will take around 10 seconds.
await Task.WhenAll(slowpoke.GoFast(), slowpoke.GoFast(), slowpoke.GoFast());
```

访问`戈斯洛`不会交错，所以执行两个`戈斯洛()`打电话大约需要20秒。另一方面，因为`戈法斯特`有标记`[总是在屋檐下]`，对它的三个调用将同时执行，并将在大约10秒内完成，而不是至少需要30秒才能完成。

## 访问链中的可重入性

为了避免死锁，调度器允许在给定的调用链中进行重入。考虑以下两个grains的例子，它们具有相互递归的方法，`伊文`和`IsOdd公司`:

```csharp
public interface IEvenGrain : IGrainWithIntegerKey
{
    Task<bool> IsEven(int num);
}

public interface IOddGrain : IGrainWithIntegerKey
{
    Task<bool> IsOdd(int num);
}

public class EvenGrain : Grain, IEvenGrain
{
    public async Task<bool> IsEven(int num)
    {
        if (num == 0) return true;
        var oddGrain = this.GrainFactory.GetGrain<IOddGrain>(0);
        return await oddGrain.IsOdd(num - 1);
    }
}

public class OddGrain : Grain, IOddGrain
{
    public async Task<bool> IsOdd(int num)
    {
        if (num == 0) return false;
        var evenGrain = this.GrainFactory.GetGrain<IEvenGrain>(0);
        return await evenGrain.IsEven(num - 1);
    }
}
```

现在考虑由以下客户端请求启动的调用流：

```csharp
var evenGrain = client.GetGrain<IEvenGrain>(0);
await evenGrain.IsEven(2);
```

上面的代码调用`伊文格伦（二）`，调用`IOddGrain.IsOdd（一）`，调用`伊文格伦（0）`，返回`是的`将访问链备份到客户端。如果没有调用链可重入，上述代码将在以下情况下导致死锁`碘酒`电话`伊文格伦（0）`. 然而，对于调用链可重入，调用被认为是开发人员的意图，因此允许继续进行。

可以通过设置来禁用此行为`SchedulingOptions.AllowCallChainEntrancy`到`假`. 例如：

```csharp
siloHostBuilder.Configure<SchedulingOptions>(
    options => options.AllowCallChainReentrancy = false);
```

## 使用谓词的可重入性

Grain类可以指定一个谓词，用于通过检查请求逐个调用确定交错。这个`[MayInterleave（字符串方法名）]`属性提供此功能。属性的参数是grain类中接受`调用方法请求`对象并返回`布尔`指示是否应交错请求。

下面是一个示例，如果请求参数类型具有`[交错]`属性：

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct)]
public sealed class InterleaveAttribute : Attribute { }

// Specify the may-interleave predicate.
[MayInterleave(nameof(ArgHasInterleaveAttribute))]
public class MyGrain : Grain, IMyGrain
{
    public static bool ArgHasInterleaveAttribute(InvokeMethodRequest req)
    {
        // Returning true indicates that this call should be interleaved with other calls.
        // Returning false indicates the opposite.
        return req.Arguments.Length == 1
            && req.Arguments[0]?.GetType().GetCustomAttribute<InterleaveAttribute>() != null;
    }

    public Task Process(object payload)
    {
        // Process the object.
    }
}
```
