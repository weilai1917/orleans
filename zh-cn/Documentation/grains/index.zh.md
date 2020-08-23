---
layout: page
title: Developing a Grain
---

# 建立

在编写代码以实现谷物类之前，请创建一个针对.NET Standard（首选）或.NET Framework 4.6.1或更高版本的新类库项目（如果由于依赖性而无法使用.NET Standard）。可以在同一个“类库”项目中或在两个不同的项目中定义晶粒接口和晶粒类，以更好地将接口与实现分开。无论哪种情况，项目都需要参考`微软奥尔良核心抽象`和`Microsoft.Orleans.CodeGenerator.MSBuild`NuGet软件包。

有关更详尽的说明，请参见[项目设置](../tutorials_and_samples/tutorial_1.md#project-setup)的部分[教程一–奥尔良基础](../tutorials_and_samples/tutorial_1.md)。

# 晶粒界面和类

晶粒相互交互，并通过调用声明为各个晶粒接口一部分的方法从外部调用。谷物类实现一个或多个先前声明的谷物接口。Grain接口的所有方法都必须返回`任务`（对于`虚空`方法），一个`任务<T>`或一个`ValueTask <T>`（对于返回类型为值的方法`Ť`）。

以下是Orleans 1.5 Presence Service示例的摘录：

```csharp
//an example of a Grain Interface
public interface IPlayerGrain : IGrainWithGuidKey
{
  Task<IGameGrain> GetCurrentGame();
  Task JoinGame(IGameGrain game);
  Task LeaveGame(IGameGrain game);
}

//an example of a Grain class implementing a Grain Interface
public class PlayerGrain : Grain, IPlayerGrain
{
    private IGameGrain currentGame;

    // Game the player is currently in. May be null.
    public Task<IGameGrain> GetCurrentGame()
    {
       return Task.FromResult(currentGame);
    }

    // Game grain calls this method to notify that the player has joined the game.
    public Task JoinGame(IGameGrain game)
    {
       currentGame = game;
       Console.WriteLine(
           "Player {0} joined game {1}",
           this.GetPrimaryKey(),
           game.GetPrimaryKey());

       return Task.CompletedTask;
    }

   // Game grain calls this method to notify that the player has left the game.
   public Task LeaveGame(IGameGrain game)
   {
       currentGame = null;
       Console.WriteLine(
           "Player {0} left game {1}",
           this.GetPrimaryKey(),
           game.GetPrimaryKey());

       return Task.CompletedTask;
   }
}
```

# 谷物方法的返回值

返回类型值的谷类方法`Ť`在谷粒接口中定义为返回a`任务<T>`。对于未标有`异步的`关键字，当返回值可用时，通常通过以下语句返回：

```csharp
public Task<SomeType> GrainMethod1()
{
    ...
    return Task.FromResult(<variable or constant with result>);
}
```

没有返回值的grain方法（实际上是void方法）在grain接口中定义为返回a`任务`。返回的`任务`指示方法的异步执行和完成。对于未标有`异步的`关键字，当“无效”方法完成执行时，需要返回的特殊值`Task.CompletedTask`：

```csharp
public Task GrainMethod2()
{
    ...
    return Task.CompletedTask;
}
```

标记为`异步的`直接返回值：

```csharp
public async Task<SomeType> GrainMethod3()
{
    ...
    return <variable or constant with result>;
}
```

一种“无效”的谷物方法标记为`异步的`不返回值的代码只是在执行结束时返回：

```csharp
public async Task GrainMethod4()
{
    ...
    return;
}
```

如果grain方法从另一个异步方法调用接收到的返回值（是否返回grain），并且不需要对该调用执行错误处理，则只需返回`任务`它从该异步调用接收作为其返回值：

```csharp
public Task<SomeType> GrainMethod5()
{
    ...
    Task<SomeType> task = CallToAnotherGrain();
    return task;
}
```

同样，“无效”粒度方法可以返回`任务`通过另一个调用返回给它，而不是等待它。

```csharp
public Task GrainMethod6()
{
    ...
    Task task = CallToAsyncAPI();
    return task;
}
```

`ValueTask <T>`可以代替`任务<T>`

### 谷物参考

谷物引用是实现与相应谷物类相同的谷物接口的代理对象。它封装了目标粒度的逻辑标识（类型和唯一键）。谷物参考是用于调用目标谷物的工具。每个谷物参考都针对单个谷物（谷物类的单个实例），但是可以为同一个谷物创建多个独立的引用。

由于谷物参考代表目标谷物的逻辑标识，因此它与谷物的物理位置无关，即使在系统完全重启后也仍然有效。开发人员可以像其他任何.NET对象一样使用谷物引用。它可以传递给方法，用作方法的返回值等，甚至可以保存到持久性存储中。

可以通过将谷物的身份传递给`GrainFactory.GetGrain <T>（键）`方法，在哪里`Ť`是谷物界面，`键`是该类型中纹理的唯一键。

以下是如何获取`IPlayerGrain`上面定义的接口。

从谷物类内部：

```csharp
    //construct the grain reference of a specific player
    IPlayerGrain player = GrainFactory.GetGrain<IPlayerGrain>(playerId);
```

来自奥尔良客户代码。

```csharp
    IPlayerGrain player = client.GetGrain<IPlayerGrain>(playerId);
```

### 粒度方法调用

奥尔良编程模型基于[异步编程](https://docs.microsoft.com/en-us/dotnet/csharp/async)。

使用上一个示例中的grain引用，这里是执行grain方法调用的方法：

```csharp
//Invoking a grain method asynchronously
Task joinGameTask = player.JoinGame(this);
//The await keyword effectively makes the remainder of the method execute asynchronously at a later point (upon completion of the Task being awaited) without blocking the thread.
await joinGameTask;
//The next line will execute later, after joinGameTask is completed.
players.Add(playerId);
```

可以加入两个或多个`任务`;联接操作创建一个新的`任务`当所有组成部分都解决时`任务`s完成。当谷物需要启动多个计算并等待所有计算完成后再继续操作时，这是一种有用的模式。例如，生成由许多部分组成的网页的前端纹理可能会进行多个后端调用，每个部分一个，并接收一个`任务`对于每个结果。然后谷物将等待所有这些的加入`任务`;当加入`任务`解决了，个人`任务`已完成，并且已收到格式化网页所需的所有数据。

例：

```csharp
List<Task> tasks = new List<Task>();
Message notification = CreateNewMessage(text);

foreach (ISubscriber subscriber in subscribers)
{
   tasks.Add(subscriber.Notify(notification));
}

// WhenAll joins a collection of tasks, and returns a joined Task that will be resolved when all of the individual notification Tasks are resolved.
Task joinedTask = Task.WhenAll(tasks);
await joinedTask;

// Execution of the rest of the method will continue asynchronously after joinedTask is resolve.
```

### 虚方法

谷物类可以选择覆盖`OnActivateAsync`和`OnDeactivateAsync`在激活和停用类的每个粒度时，由Orleans运行时调用的虚拟方法。这使谷物代码有机会执行其他初始化和清理操作。引发的异常`OnActivateAsync`无法激活过程。而`OnActivateAsync`，如果被覆盖，则始终被称为谷物激活过程的一部分，`OnDeactivateAsync`不能保证在所有情况下（例如在服务器故障或其他异常事件的情况下）都会被调用。因此，应用程序不应依赖`OnDeactivateAsync`用于执行关键操作，例如状态变化的持久性。他们应仅将其用于尽力而为的操作。
