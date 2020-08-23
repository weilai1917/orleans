---
layout: page
title: Declarative Persistence
---

# 声明性持久性

在第二个教程中，我们看到了grain state是如何在客户机被关闭的情况下幸存下来的，这为许多类似缓存的场景打开了大门，在这种场景中，Orleans被视为一种“带有行为的缓存”，一种面向对象的缓存，如果你愿意的话。这已经是非常有价值的了，并且通过一个简单、熟悉的编程模型和内置的单线程执行保证来实现服务器端的可伸缩性。

但是，有时，您正在累积的某些状态属于某种形式的永久存储，因此它可以在silos关闭或从一个silos迁移到另一个silos以实现负载平衡或服务完全重新启动/关机后继续存在。我们迄今所看到的情况将不支持这种情况。

幸运的是，Orleans提供了一个简单的声明性模型，用于标识需要存储在永久位置的状态，同时将何时保存和恢复状态的决定权交给编程控制。您不需要使用声明性持久性机制，并且仍然可以直接从grain代码访问存储，但这是一种很好的方法，可以为您节省一些样板代码，并构建可跨各种存储服务移植的应用程序。

## 入门

我们将继续以我们的员工和经理样本为基础。

我们需要做的第一件事是让我们的员工和管理者的身份更加可预测。在这个示例中，使用`Guid.NewGuid()`，这很方便，但不允许我们在后续运行中找到它们。因此，我们将首先创建一组guid，然后将它们用作工作者标识。

修改后的客户端程序如下所示：

```csharp
private static async Task DoClientWork(IClusterClient client)
{
     ...
    var ids = new string[] {
        "42783519-d64e-44c9-9c29-399e3afaa625",
        "d694a4e0-1bc3-4c3f-a1ad-ba95103622bc",
        "9a72b0c6-33df-49db-ac05-14316edd332d",
        "6526a751-b9ac-4881-9bfb-836ecce2ca9f",
        "ae4b106f-3c96-464a-b48d-3583ed584b17",
        "b715c40f-d8d2-424d-9618-76afbc0a2a0a",
        "5ad92744-a0b1-487b-a9e7-e6b91e9a9826",
        "e23a55af-217c-4d76-8221-c2b447bf04c8",
        "2eef0ac5-540f-4421-b9a9-79d89400f7ab"
    };

    var e0 = client.GetGrain<IEmployee>(Guid.Parse(ids[0]));
    var e1 = client.GetGrain<IEmployee>(Guid.Parse(ids[1]));
    var e2 = client.GetGrain<IEmployee>(Guid.Parse(ids[2]));
    var e3 = client.GetGrain<IEmployee>(Guid.Parse(ids[3]));
    var e4 = client.GetGrain<IEmployee>(Guid.Parse(ids[4]));

    var m0 = client.GetGrain<IManager>(Guid.Parse(ids[5]));
    var m1 = client.GetGrain<IManager>(Guid.Parse(ids[6]));
     ...
}
```

> 注意：如果您从Orleans1.5过渡，您会注意到客户机不再是静态的。请参考[从Orleans迁移到0.5](../Documentation/resources/Migration/Migration1.5.zh.md)第页。

下一步，我们将进行一些silos配置，以便配置将允许我们访问持久存储的存储提供程序。SiloHost项目包括一个文件*程序.cs*我们可以在这里找到以下部分：

```csharp
var builder = new SiloHostBuilder()
                .UseLocalhostClustering()
                .Configure<EndpointOptions>(options => options.AdvertisedIPAddress = IPAddress.Loopback)
                .ConfigureLogging(logging => logging.AddConsole());

var host = builder.Build();
await host.StartAsync();
return host;
```

如果这是托管在Azure云服务中，则可以在调用之前使用以下任一选项`建造者。建造者()`要使用Azure来保持grains状态，请执行以下操作：

```csharp
// Stores grains as composition of fields
builder.AddAzureTableGrainStorage(option => option.ConnectionString = your_connection_string);
// Stores grains as blobs
builder.AddAzureBlobGrainStorage(option => option.ConnectionString = your_connection_string);
```

这个`记忆库`provider相当无趣，因为它实际上不提供任何永久存储；它的目的是调试持久性grains，而不能访问持久性存储。在我们的例子中，这使得很难证明持久性，所以我们将依赖于一个真正的存储提供者。

根据您是否已经设置（并希望使用）Azure存储帐户，还是希望依赖Azure存储仿真程序，您应该添加其他两行中的一行，但不能同时添加这两行。您可以使用`AddAzureTableStorage提供程序（）`函数或`AddAzureBlobStorageProvider（）`函数取决于您希望如何存储信息。

对于前者，您必须在安装最新版本的azuresdk之后启动Azure存储仿真器。对于后者，您必须创建一个Azure存储帐户，并在配置文件中输入名称和密钥。

启用其中一个，我们就可以处理grain代码了。

> 注意：在Orleans 2.0中，许多功能被拆分成更小的包，以允许更细粒度的配置和部署。这包括Azure存储提供程序。请安装[Orleans持久性](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.AzureStorage/)如果要将Azure用作存储提供程序，请打包。您可以通过[寻找Orleans。坚持](https://www.nuget.org/packages?q=Microsoft.Orleans.Persistence).

## 申报国

确定grains应使用持久状态需要三个步骤：

1.  为州声明一个类，
2.  改变Grains基类，以及
3.  正在标识存储提供程序。

第一步，在grain实现项目中声明一个state类，只意味着标识应该持久化的参与者的信息，并创建一个看起来像持久数据记录的内容——每个状态组件都由一个带有getter和setter的属性表示。

对于员工，我们希望保持所有状态：

```csharp
public class  EmployeeState
{
    public int Level { get; set; }
    public IManager Manager { get; set; }
}
```

对于经理来说，我们必须存储直接下属，但是`_我`在激活期间可能会继续创建引用。

```csharp
public class ManagerState
{
    public List<IEmployee> Reports { get; set; }
}
```

然后，我们更改grain类声明以标识状态接口（例如，从`Orleans。Grains`到`Orleans。Grains<EmployeeState>`)并删除我们想要持久化的变量。确保移除`水平`，和`经理`从`雇员`类和`_报告`从`经理`班级。除此之外，我们还必须更新这些功能。

我们还添加了一个属性来标识存储提供程序：

```csharp
[StorageProvider(ProviderName = "AzureStore")]
public class Employee : Orleans.Grain<EmployeeState>, Interfaces.IEmployee
```

和

```csharp
[StorageProvider(ProviderName="AzureStore")]
public class Manager : Orleans.Grain<ManagerState>, IManager
```

在声明显而易见的风险下，存储提供程序属性的名称应该与配置silos时使用的名称相匹配。这种间接性使您能够在部署之前延迟关于在哪里存储粒度状态的选择。

考虑到这些声明性的更改，grain不应该再依赖私有字段来保持补偿级别和管理器。相反，grain基类让我们通过`州`可用于Grains的属性。

例如：

```csharp
public Task SetManager(IManager manager)
{
   State.Manager = manager;
   return TaskDone.Done;
}
```

## 控制检查点

剩下的问题是持久状态何时保存到存储提供程序。

Orleans设计人员可以做出的一个选择是在每次方法调用后都设置运行时保存状态，但结果证明这是不可取的，因为它过于保守——并非所有调用都会修改所有调用的状态，而且有些调用永远不会修改它。Orleans没有在每个方法之后使用复杂的系统来评估状态差异，而是要求grain开发人员添加必要的逻辑来确定是否需要保存状态。

使用存储提供程序保存状态很容易通过调用`base.WriteStateAsync()`.

因此，最终版本的`提升（）`和`设置管理器（）`方法如下：

```csharp
public Task Promote(int newLevel)
{
    State.Level = newLevel;
    return base.WriteStateAsync();
}

public Task SetManager(IManager manager)
{
    State.Manager = manager;
    return base.WriteStateAsync();
}
```

在`经理`类，只有一个方法需要修改才能写出数据，`AddDirectReport（）`. 应该是这样的：

```csharp
public async Task AddDirectReport(IEmployee employee)
{
    if (State.Reports == null)
    {
        State.Reports = new List<IEmployee>();
    }
    State.Reports.Add(employee);
    await employee.SetManager(this);
    var data = new GreetingData { From = this.GetPrimaryKey(), Message = "Welcome to my team!" };
    await employee.Greeting(data);
    Console.WriteLine("{0} said: {1}",
                        data.From.ToString(),
                        data.Message);

    await base.WriteStateAsync();
}
```

让我们试试这个！

在中设置断点`员工。晋升()`.

当我们第一次运行客户端代码并到达断点时，level字段应该是`0`以及newLevel参数`10`或`11`:

!\[]（../Images/Persistence 2.PNG）

让应用程序完成（到达“点击回车…”提示符）并退出。再次运行它，并比较第二次查看状态时发生的情况：

!\[]（../Images/Persistence 3.PNG）

## 只是确保。。。

值得检查一下Azure对数据的看法。使用存储资源管理器（如Azure存储资源管理器（ASE）或Visual Studio 2013中内置的服务器资源管理器），打开存储帐户（或模拟器的开发人员存储）并找到“OrleansGrainState”表。它应该是这样的（你必须在ASE中点击'Query'：

!\[]（../Images/Persistence 4.PNG）

如果一切正常，则纹理键应该出现在`分区键`列中，并且grains的限定类名应出现在`行键`列。

## 混合的东西

grains可能包含持久状态和瞬态状态的组合。任何瞬态都应该用grain类中的私有字段来表示。混合使用这两者的一个常见用途是，当持久化状态存在于内存中时，将它的某些计算版本缓存到私有字段中。例如，一堆元素可以外部表示为`列表<T>`，但在内部，作为`堆栈<T>`.

如果我们`经理`同学们`_我`field只是一个缓存的值，一开始我们甚至不需要将其作为字段保存，它可以在任何需要它的时候创建，但是由于它将是一个常用的值，所以值得将它保存在一个临时字段中。

## 状态自动加载

如果Grains类型具有状态，则在激活时该状态将从存储中加载，然后`非激活异步`调用，以便您可以确保在初始化Grains时加载了状态。这是Orleans唯一打电话来的案子`读状态异步`自动地。如果你想写州或在其他地方读，你应该自己写。通常你不需要打电话`读状态异步`你自己，除非你正在做一些关于处理损坏状态或其他事情的具体事情。

## 使用持久性处理故障

一般来说，读写Grains的状态是一个很好的机制来处理失败和服务于它的初衷。由于不同的原因，grain调用可能会在方法的中间失败，最终导致状态更改一半。在这种情况下，从存储器中读取可以将状态返回到上一个正确的状态。或者，进入这种状态后，grain可以通过调用DeactivateOnIdle（）请求立即停用，这样它的下一个请求将触发grain的重新激活，这将重新读取持久状态并重建其在内存中的副本。停用是将grains重置为其最后已知良好状态的最干净的方法，但是如果要避免重新激活过程的成本，可以重置其状态并重新运行任何初始化逻辑（例如，通过调用`非激活异步`)而不是使Grains失活。

## 下一个

接下来，我们将了解如何从mvcweb应用程序调用grains。

[处理失败](../1.5/Tutorials/Failure-Handling.zh.md)
