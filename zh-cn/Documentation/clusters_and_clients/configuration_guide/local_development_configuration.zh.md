---
layout: page
title: Local development configuration
---

# 本地开发配置

有关针对Orleans 2.0的工作示例应用程序，请参见：[https://github.com/dotnet/orleans/tree/master/samples/2.0/helloworld网站](https://github.com/dotnet/orleans/tree/master/Samples/2.0/HelloWorld)此示例托管了在不同平台上工作的.NET Core控制台应用程序中的客户端和silos，但对于.NET Framework 4.6.1+控制台应用程序（仅在Windows上工作）也可以这样做。

## silos配置

对于本地开发，请参阅下面的示例，了解如何为这种情况配置silos。它配置并启动一个silos，分别作为silos和网关端口监听“环回”地址和11111和30000。

添加`Microsoft.Orleans.Server公司`项目的nuget元包。在您熟悉了api之后，可以选择包含在`Microsoft.Orleans.Server公司`你真的需要，并参考他们。

```PowerShell
PM> Install-Package Microsoft.Orleans.Server
```

您需要配置`离合器选项`通过`ISiloBuilder.Configure`方法，指定您想要的`发展集群`将此silos作为主要的群集选择，然后配置silos终结点。

`配置应用程序部件`call显式地将具有grain类的程序集添加到应用程序设置中。它还添加任何引用的程序集，因为`有参考文献`分机。完成这些步骤后，将生成silos主机并启动silos。

您可以创建一个空的控制台应用程序项目，目标是.NETFramework4.6.1或更高版本，用于托管silos和.NETCore控制台应用程序。

以下是如何启动本地silos的示例：

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        try
        {
            var host = await StartSilo();
            Console.WriteLine("Press Enter to terminate...");
            Console.ReadLine();

            await host.StopAsync();

            return;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex);
            return;
        }
    }

   private static async Task<ISiloHost> StartSilo()
    {
        var builder = new SiloHostBuilder()
	    // Use localhost clustering for a single local silo
            .UseLocalhostClustering()
            // Configure ClusterId and ServiceId
            .Configure<ClusterOptions>(options =>
            {
                options.ClusterId = "dev";
                options.ServiceId = "MyAwesomeService";
            })
            // Configure connectivity
	    .Configure<EndpointOptions>(options => options.AdvertisedIPAddress = IPAddress.Loopback)
            // Configure logging with any logging framework that supports Microsoft.Extensions.Logging.
            // In this particular case it logs using the Microsoft.Extensions.Logging.Console package.
            .ConfigureLogging(logging => logging.AddConsole());

        var host = builder.Build();
        await host.StartAsync();
        return host;
    }
}
```

## 客户端配置

对于本地开发，请参阅下面的示例，了解如何为这种情况配置客户端。它配置将连接到`回送`silos。

添加`Microsoft.Orleans.client公司`项目的nuget元包。在您熟悉了api之后，可以选择包含在`Microsoft.Orleans.client公司`你真的需要，并参考他们。

```PowerShell
PM> Install-Package Microsoft.Orleans.Client
```

您需要配置`客户端生成器`使用与为本地silos指定的群集ID匹配的群集ID，并将静态群集指定为指向silos网关端口的群集选择

`配置应用程序部件`call显式地将具有grain接口的程序集添加到应用程序设置中。

完成这些步骤后，我们可以构建客户机并`连接（）`方法连接到群集。

您可以创建一个空的控制台应用程序项目，目标是.net framework 4.6.1或更高版本以运行客户端，或者重用为托管silos而创建的控制台应用程序项目。

以下是客户端如何连接到本地silos的示例：

```csharp
client = new ClientBuilder()
    // Use localhost clustering for a single local silo
    .UseLocalhostClustering()
    // Configure ClusterId and ServiceId
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "dev";
        options.ServiceId = "MyAwesomeService";
    })
    .ConfigureLogging(logging => logging.AddConsole())
var client = builder.Build();
await client.Connect();
```
