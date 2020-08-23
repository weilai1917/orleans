---
layout: page
title: Orleans Lifecycle
---

# Orleans生命周期

## 总览

Orleans的某些行为非常复杂，因此需要有序地启动和关闭。具有此类行为的某些组件包括Grains，silos和客户。为了解决这个问题，引入了通用组件生命周期模式。此模式包括一个可观察的生命周期，该生命周期负责在组件启动和关闭的各个阶段发出信号，而生命周期观察器则负责在特定阶段执行启动或关闭操作。

也可以看看[Grains生命周期](../grains/grain_lifecycle.zh.md)和[silos生命周期](../clusters_and_clients/silo_lifecycle.zh.md)。

## 可观察的生命周期

需要顺序启动和关闭的组件可以使用可观察的生命周期，该周期可允许其他组件观察LiveCycle并在启动或关闭过程中到达某个阶段时接收通知。

```csharp
    public interface ILifecycleObservable
    {
        IDisposable Subscribe(string observerName, int stage, ILifecycleObserver observer);
    }
```

订阅呼叫在启动或停止时达到阶段时会注册观察者以进行通知。观察者名称用于报告。指示在启动/关闭顺序中的哪一点将通知观察者的阶段。生命周期的每个阶段都是可观察的。当开始和停止到达阶段时，将通知所有观察者。阶段以升序开始，以降序停止。观察者可以通过处置返回的一次性物品来退订。

## 生命周期观察者

需要参与另一个组件生命周期的组件需要为其启动和关闭行为提供挂钩，并订阅可观察到的生命周期的特定阶段。

```csharp
    public interface ILifecycleObserver
    {
        Task OnStart(CancellationToken ct);
        Task OnStop(CancellationToken ct);
    }
```

`OnStart / OnStop`在启动/关闭期间达到订阅的阶段时将调用。

## 实用工具

为了方便起见，已经为常见的生命周期使用模式创建了辅助功能。

### 扩展名

存在用于订阅可观察生命周期的扩展功能，这些功能不需要订阅组件实现ILifecycleObserver。而是，这些允许组件传递lambda或在预订阶段调用成员函数。

```csharp
IDisposable Subscribe(this ILifecycleObservable observable, string observerName, int stage, Func<CancellationToken, Task> onStart, Func<CancellationToken, Task> onStop);

IDisposable Subscribe(this ILifecycleObservable observable, string observerName, int stage, Func<CancellationToken, Task> onStart);
```

类似的扩展功能允许使用通用类型参数来代替观察者名称。

```csharp
IDisposable Subscribe<TObserver>(this ILifecycleObservable observable, int stage, Func<CancellationToken, Task> onStart, Func<CancellationToken, Task> onStop);

IDisposable Subscribe<TObserver>(this ILifecycleObservable observable, int stage, Func<CancellationToken, Task> onStart);
```

### 生命周期参与

一些可扩展性点需要一种方法来识别哪些组件对参与生命周期感兴趣。为此，引入了生命周期参与者标记界面。探索silos和Grains的生命周期时，将详细介绍如何使用它。

```csharp
    public interface ILifecycleParticipant<TLifecycleObservable>
        where TLifecycleObservable : ILifecycleObservable
    {
        void Participate(TLifecycleObservable lifecycle);
    }
```

## 例

根据我们的生命周期测试，以下是在生命周期的多个阶段参与可观察生命周期的组件示例。

```csharp
enum TestStages
{
    Down,
    Initialize,
    Configure,
    Run,
}

class MultiStageObserver : ILifecycleParticipant<ILifecycleObservable>
{
    public Dictionary<TestStages,bool> Started { get; } = new Dictionary<TestStages, bool>();
    public Dictionary<TestStages, bool> Stopped { get; } = new Dictionary<TestStages, bool>();

    private Task OnStartStage(TestStages stage)
    {
        this.Started[stage] = true;
        return Task.CompletedTask;
    }

    private Task OnStopStage(TestStages stage)
    {
        this.Stopped[stage] = true;
        return Task.CompletedTask;
    }

    public void Participate(ILifecycleObservable lifecycle)
    {
        lifecycle.Subscribe<MultiStageObserver>((int)TestStages.Down, ct => OnStartStage(TestStages.Down), ct => OnStopStage(TestStages.Down));
        lifecycle.Subscribe<MultiStageObserver>((int)TestStages.Initialize, ct => OnStartStage(TestStages.Initialize), ct => OnStopStage(TestStages.Initialize));
        lifecycle.Subscribe<MultiStageObserver>((int)TestStages.Configure, ct => OnStartStage(TestStages.Configure), ct => OnStopStage(TestStages.Configure));
        lifecycle.Subscribe<MultiStageObserver>((int)TestStages.Run, ct => OnStartStage(TestStages.Run), ct => OnStopStage(TestStages.Run));
    }
}
```
