# Orleans和米多里

[贝科夫](https://github.com/sergeybykov)2016年12月11日下午10:56:13

* * *

读乔·达菲的史诗[共事15年](http://joeduffyblog.com/2016/11/30/15-years-of-concurrency/)波斯特带来了Orleans早期的一些往事。它甚至迫使我挖掘并尝试编译2009年的代码。这是一个有趣的练习。

当我们刚开始Orleans项目时，我们会定期与Midori人会面和交谈。这是很自然的，不仅是因为问题空间有一些明显的重叠，而且因为孕育Orleans的吉姆·拉鲁斯是[奇点](https://www.microsoft.com/en-us/research/project/singularity/)米多里的起点。我们立即借用了Midori的promises库，因为我们想使用基于promise的并发来安全执行和高效的RPC。我们没有费心尝试集成代码，只是简单地获取了二进制文件并将它们签入到我们的源代码树中。我们处于早期原型阶段，还不必担心长期性。

当时，grains界面是这样的：

```csharp
[Eventual]
public interface ISimpleGrain : IEventual
{
    [Eventual]
    PVoid SetA(int a);

    [Eventual]
    PVoid SetB(int b);

    [Eventual]
    PInt32 GetAxB();
}
```

`PVoid公司`和`针脚32`在道德上等同于[任务和任务\\\<int>在第三方物流](https://msdn.microsoft.com/en-us/library/dd537609(v=vs.110).aspx). 与Tasks不同，它们有许多静态方法，其中一个更简单的重载使用两个lambda：一个用于成功案例，另一个用于处理抛出的异常：

```csharp
public static PVoid When(PVoid target, Action fn, Action<Exception> catchFn);
```

一个简单的纹理方法看起来像：

```csharp
public PVoid SetA(int a)
{
    this.a = a;
    return PVoid.DONE;
}
```

你可以在这里看到任务完成了。完成了来自。一个简单的单元测试方法看起来相当复杂：

```csharp
[TestMethod]
public void SimpleGrainDataFlow()
{
    result = new ResultHandle();
    Runner.Enqueue(new SimpleTodo(() =>
    {

       Promise<SimpleGrainReference> clientPromise = SimpleGrainReference.GetReference("foo");
       PVoid.When(clientPromise,
           reference =>
           {
                grain = reference;
                Assert.IsNotNull(grain);
                PVoid setPromise = grain.SetA(3);
                PVoid.When(setPromise,
                    () =>
                    {
                        setPromise = grain.SetB(4);
                        PVoid.When(setPromise,
                            () =>
                            {
                                PInt32 intPromise = grain.GetAxB();
                                PVoid.When<Int32>(intPromise,
                                    x =>
                                    {
                                        result.Result = x;
                                        result.Done = true;
                                    },
                                    exc =>
                                    {
                                        Assert.Fail("Exception thrown by GetAxB: " + exc.Message);
                                        return PVoid.DONE;
                                    });
                            },
                            exc =>
                            {
                                Assert.Fail("Exception thrown by SetB: " + exc.Message);
                                return PVoid.DONE;
                            });
                    },
                    exc =>
                    {
                        Assert.Fail("Exception thrown by SetA: " + exc.Message);
                        return PVoid.DONE;
                    });
            },
            exc =>
            {
                result.Exception =  exc;
                result.Done = true;
                return PVoid.DONE;
            });
    })); 

    Assert.IsTrue(result.WaitForFinished(timeout));
    Assert.IsNotNull(result.Result);
    Assert.AreEqual(12, result.Result);
}
```

嵌套when是组织数据流执行管道所必需的。`跑步者`是`外国投资者`，这是注入异步任务的方法之一(`托多`s） 变成一个`托多经理`. `托多经理`是一个单线程执行管理器，也叫增值税，[来自E语言的概念](http://www.erights.org/elib/concurrency/vat.html). 增值税执行系统的初始化是几行代码：

```csharp
todoManager = new TodoManager();

Thread t = new Thread(todoManager.Run);
t.Name = "Unit test TodoManager";
t.Start();

runner = new ForeignTodoRunner(todoManager);
```

在一个silos中，我们还使用vats来管理Grains转向的单线程执行。作为silos启动的一部分，我们将其中的N个设置为与可用CPU核心的数量相匹配：

```csharp
for (int i = 0; i < nTodoManagers; i++)
{
    todoManagers[i] = new TodoManager();

    for (int j = 0; j < runnerFactor; j++)
        todoRunners[i \* runnerFactor + j] = new ForeignTodoRunner(todoManagers[i]);

    Thread t = new Thread(todoManagers[i].Run);
    t.Name = String.Format("TodoManager: {0}", i);
    t.Start();
}
```

当时我们和迪恩·特里布尔争论说，在我们看来，在承诺上使用静态方法对大多数开发人员来说太不方便了。我们希望它们成为实例方法。几个月后，我们推出了自己的承诺AsyncCompletion和AsyncValue<T>. 他们是任务和任务的包装<T>和had实例方法。这将代码压缩了不少：

```csharp
[TestMethod]
public void SimpleGrainDataFlow()
{
    ResultHandle result = new ResultHandle(); 
    SimpleGrainReference grain = SimpleGrainReference.GetReference("foo");

    AsyncCompletion setPromise = grain.SetA(3);
    setPromise.ContinueWith(() =>
    {
        setPromise = grain.SetB(4);
        setPromise.ContinueWith(
        () =>
        {
            AsyncValue<int> intPromise = grain.GetAxB();
            intPromise.ContinueWith(
            x =>
            {
                result.Result = x;
                result.Done = true;
            });
        });
    });

    Assert.IsTrue(result.WaitForFinished(timeout));
    Assert.IsNotNull(result.Result);
    Assert.AreEqual(12, result.Result);
}
```

最初，我们允许grain方法是同步的，并将grain引用作为它们的异步代理。

```csharp
public class SimpleGrain : GrainBase
{
    public void SetA(int a)

    public void SetB(int b)

    public int GetAxB()
}

public class SimpleGrainReference : GrainReference
{
    public AsyncCompletion SetA(int a)

    public AsyncCompletion SetB(int b)

    public AsyncValue<int> GetAxB()
}
```

我们很快意识到这是个坏主意，于是改用Grains的方法退货`异步完成`/`异步值<T>`. 我们反复考虑，最终放弃了一些其他的坏主意。我们支持Grains类的属性。异步设置器是一个问题，一般来说，异步属性具有相当的误导性，与显式getter方法相比没有任何好处。我们最初支持Grains上的.NET事件。由于.NET中+=和-=操作基本上是同步的，所以不得不废弃它们。

为什么我们不简单地使用`任务`/`任务<T>`而不是`异步完成`/`异步值<T>`?

为了保证单线程执行，我们需要拦截每个调度和连续调用。`任务`是一个密封类，因此我们无法将其子类化以重写所需的键方法。我们也没有一个定制的TPL调度程序。

在我们转而使用我们自己的承诺之后，我们失去了使用Midori的一些高级功能的机会。例如，他们支持三方承诺切换协议。如果节点A调用了节点B并持有该调用的允诺，但作为处理请求的一部分，B向C发出了对最终值的调用，则B可以传递对A持有的允诺的引用，这样C就可以直接响应A，而不是额外跳回B。在性能和复杂性之间的折衷中，我们选择优先考虑简单性。

我们从与Midori人交谈中得到的另一个教训是，在他们的代码库中，一些最难跟踪的bug的来源是交错执行。尽管vat只有一个线程来执行所有的turns（在屈服点之间同步的代码段），但它以任意顺序执行属于不同请求的turns是完全合法的。

假设您的组件正在处理一个请求，并且需要调用另一个组件，例如，在其中进行IO调用。您发出那个IO调用，收到完成或返回值的承诺，并使用`什么时候？`或`继续`打电话。这里的陷阱是，当IO调用完成并且计划的延续开始执行时，很容易假设组件的状态自发出IO调用以来没有改变。事实上，组件可能在异步等待IO调用时接收并处理了许多其他请求，而处理这些请求可能会以不明显的方式改变组件的状态。Midori团队非常资深。当时，他们中的大多数是校长和合伙人级别的工程师和建筑师。我们想知道，如果穿插对这种才干和经验的人来说是如此的危险，那对我们这样的凡人来说肯定更糟。这导致后来决定让Orleans的Grains在默认情况下不可再入。

大约在同一时间，Niklas Gustafsson参与了这个项目[大师](https://channel9.msdn.com/shows/Going+Deep/Maestro-A-Managed-Domain-Specific-Language-For-Concurrent-Programming/)后来改名为[阿克苏姆](https://web.archive.org/web/20090511155406/http:/msdn.microsoft.com/en-us/devlabs/dd795202.aspx). 我们在Axum上有一个实习生原型（早期的Orleans应用程序之一）来比较编程体验和2009年春季基于promise的应用程序。我们得出的结论是，对于开发人员来说，promises模型更容易实现。与此同时，尼格拉斯提出了一个建议和一个原型，在他说服了安德斯·赫兹伯格和其他人之后，最终成为了`异步`/`等待`C中的关键字#. 到现在，它传播到了更多的语言。

在发布了带有async和await的.net4.5之后，我们最终放弃了`异步完成`/`异步值<T>`赞成`任务`/`任务<T>`利用等待的力量。这是另一个折衷方案，它让我们重写了几次调度程序（不是一个小任务），并放弃了我们承诺中的一些好特性。例如，在我们可以轻松检测到grain代码是否试图通过调用`结果`或`等等（）`在一个未解决的承诺上`InvalidOperationException异常`表示在silos的协作多任务环境中不允许这样做。我们不能再这样了。但我们获得了今天更干净的编程模型：

```csharp
public interface ISimpleGrain : IGrainWithIntegerKey
{
    Task SetA(int a);

    Task SetB(int b);

    Task<int> GetAxB();
}

[Fact, TestCategory("BVT"), TestCategory("Functional")]
public async Task SimpleGrainDataFlow()
{
    var grain = GrainFactory.GetGrain<ISimpleGrain>(GetRandomGrainId());

    Task setAPromise = grain.SetA(3);
    Task setBPromise = grain.SetB(4);

    await Task.WhenAll(setAPromise, setBPromise);

    var x = await grain.GetAxB();
    Assert.Equal(12, x);
}
```

Midori是一个非常有意义的有趣实验，试图构建一个具有异步性和自上而下隔离的“安全构建”操作系统。从成功、失败和错失的机会来判断这些努力总是很困难的。有一点很清楚，Midori确实影响了Orleans关于异步和并发的早期思考和设计，并帮助引导了它的初始原型。
