---
layout: page
title: Shutting down Orleans
---

本文档解释了如何在应用程序出口之前优雅地关闭Orleanssilos，首先是作为控制台应用程序，然后作为一个反编译容器应用程序。

# 正常关机-控制台应用程序

下面的代码显示了如何在用户按下ctrl+c时优雅地关闭Orleans思洛控制台应用程序，这将生成`控制台取消按键`事件

通常，当事件处理程序返回时，应用程序将立即退出，导致灾难性的Orleanssilos崩溃和内存状态丢失。但是在下面的示例代码中，我们设置`a.取消=真；`以防止应用程序在Orleanssilos完成正常关闭之前关闭。

```csharp
using Microsoft.Extensions.Logging;
using Orleans.Configuration;
using Orleans.Hosting;
using System;
using System.Net;
using System.Threading;
using System.Threading.Tasks;

namespace MySiloHost {

    class Program {

        static readonly ManualResetEvent _siloStopped = new ManualResetEvent(false);

        static ISiloHost silo;
        static bool siloStopping = false;
        static readonly object syncLock = new object();

        static void Main(string[] args) {

            SetupApplicationShutdown();

            silo = CreateSilo();
            silo.StartAsync().Wait();

            /// Wait for the silo to completely shutdown before exiting. 
            _siloStopped.WaitOne();
        }

        static void SetupApplicationShutdown() {
            /// Capture the user pressing Ctrl+C
            Console.CancelKeyPress += (s, a) => {
                /// Prevent the application from crashing ungracefully.
                a.Cancel = true;
                /// Don't allow the following code to repeat if the user presses Ctrl+C repeatedly.
                lock (syncLock) {
                    if (!siloStopping) {
                        siloStopping = true;
                        Task.Run(StopSilo).Ignore();
                    }
                }
                /// Event handler execution exits immediately, leaving the silo shutdown running on a background thread,
                /// but the app doesn't crash because a.Cancel has been set = true
            };
        }

        static ISiloHost CreateSilo() {
            return new SiloHostBuilder()
                .Configure(options => options.ClusterId = "MyTestCluster")
                /// Prevent the silo from automatically stopping itself when the cancel key is pressed.
                .Configure<ProcessExitHandlingOptions>(options => options.FastKillOnProcessExit = false)
                .UseDevelopmentClustering(options => options.PrimarySiloEndpoint = new IPEndPoint(IPAddress.Loopback, 11111))
                .ConfigureLogging(b => b.SetMinimumLevel(LogLevel.Debug).AddConsole())
                .Build();
        }

        static async Task StopSilo() {
            await silo.StopAsync();
            _siloStopped.Set();
        }
    }
}
```

当然，还有很多其他方法可以达到同样的目标。下面是一个方式，流行的在线，误导，这是不正常的工作。它不起作用，因为它在试图退出的两种方法之间设置一个竞争条件：`控制台取消按键`事件处理程序方法，以及`静态void main（字符串[]参数）`方法。当事件处理方法首先完成时，至少发生一半的时间，应用程序将挂起而不是顺利退出。

```csharp
class Program {

    static readonly ManualResetEvent _siloStopped = new ManualResetEvent(false);

    static ISiloHost silo;
    static bool siloStopping = false;
    static readonly object syncLock = new object();

    static void Main(string[] args) {

        Console.CancelKeyPress += (s, a) => {
            Task.Run(StopSilo);
            /// Wait for the silo to completely shutdown before exiting. 
            _siloStopped.WaitOne();
            /// Now race to finish ... who will finish first?
            /// If I finish first, the application will hang! :(
        };

        silo = CreateSilo();
        silo.StartAsync().Wait();

        /// Wait for the silo to completely shutdown before exiting. 
        _siloStopped.WaitOne();
        /// Now race to finish ... who will finish first?
    }

    static async Task StopSilo() {
        await silo.StopAsync();
        _siloStopped.Set();
    }
}
```

# 正常关机-Docker应用程序

待完成。
