---
layout: page
title: Service Fabric Hosting
---

# 服务结构宿主

奥尔良可以使用`Microsoft.Orleans.hosting.ServiceFabric公司`包裹。筒仓应该作为未分区的无状态服务托管，因为Orleans使用细粒度的动态分布来管理谷物本身的分布。其他托管选项（分区的、有状态的）当前未经测试且不受支持。

一个演示在服务结构上托管的示例位于[示例/2.0/ServiceFabric](https://github.com/dotnet/orleans/tree/master/Samples/2.0/ServiceFabric)是的。

托管支持在`Microsoft.Orleans.hosting.ServiceFabric公司`包裹。它允许奥尔良筒仓作为服务结构运行`图像通信侦听器`是的。思洛存储器生命周期遵循典型的通信侦听器生命周期：它通过`ICommunicationListener.OpenAsync`方法，并通过`ICommunicationListener.CloseAsync`方法或通过`ICommunication侦听器。中止`方法。

`OrleansCommunication侦听器`提供`图像通信侦听器`实施。推荐的方法是使用`orleansServiceListener.createStateless（操作<statelessServiceContext，isiloHostBuilder>configure）`在`Orleans.Hosting.ServiceFabric`命名空间。

每次打开通信侦听器时，`配置`委托传递给`创建无状态`调用以配置新思洛存储器。

## 示例：配置服务结构宿主

下面的示例演示了一个服务结构`无状态服务`类，该类托管奥尔良思洛存储器。完整的样本可以在[示例/2.0/ServiceFabric](https://github.com/dotnet/orleans/tree/master/Samples/2.0/ServiceFabric)奥尔良仓库的目录。

```csharp
/// <summary>
/// An instance of this class is created for each service instance by the Service Fabric runtime.
/// </summary>
internal sealed class StatelessCalculatorService : StatelessService
{
    public StatelessCalculatorService(StatelessServiceContext context)
        : base(context)
    {
    }

    /// <summary>
    /// Optional override to create listeners (e.g., TCP, HTTP) for this service replica to handle
    /// client or user requests.
    /// </summary>
    /// <returns>A collection of listeners.</returns>
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        // Listeners can be opened and closed multiple times over the lifetime of a service instance.
        // A new Orleans silo will be both created and initialized each time the listener is opened
        // and will be shutdown when the listener is closed.
        var listener = OrleansServiceListener.CreateStateless(
            (fabricServiceContext, builder) =>
            {
                builder.Configure<ClusterOptions>(options =>
                {
                    // The service id is unique for the entire service over its lifetime. This is
                    // used to identify persistent state such as reminders and grain state.
                    options.ServiceId = fabricServiceContext.ServiceName.ToString();

                    // The cluster id identifies a deployed cluster. Since Service Fabric uses rolling
                    // upgrades, the cluster id can be kept constant. This is used to identify which
                    // silos belong to a particular cluster.
                    options.ClusterId = "development";
                });

                // Configure clustering. Other clustering providers are available, but for the purpose
                // of this sample we will use Azure Storage.
                // TODO: Pick a clustering provider and configure it here.
                builder.UseAzureStorageClustering(
                    options => options.ConnectionString = "UseDevelopmentStorage=true");

                // Optional: configure logging.
                builder.ConfigureLogging(logging => logging.AddDebug());

                builder.AddStartupTask<StartupTask>();

                // Service Fabric manages port allocations, so update the configuration using those
                // ports.
                // Gather configuration from Service Fabric.
                var activation = fabricServiceContext.CodePackageActivationContext;
                var endpoints = activation.GetEndpoints();

                // These endpoint names correspond to TCP endpoints specified in ServiceManifest.xml
                var siloEndpoint = endpoints["OrleansSiloEndpoint"];
                var gatewayEndpoint = endpoints["OrleansProxyEndpoint"];
                var hostname = fabricServiceContext.NodeContext.IPAddressOrFQDN;
                builder.ConfigureEndpoints(hostname, siloEndpoint.Port, gatewayEndpoint.Port);

                // Add your application assemblies.
                builder.ConfigureApplicationParts(parts =>
                {
                    parts.AddApplicationPart(typeof(CalculatorGrain).Assembly).WithReferences();

                    // Alternative: add all loadable assemblies in the current base path
                    // (see AppDomain.BaseDirectory).
                    parts.AddFromApplicationBaseDirectory();
                });
            });

        return new[] { listener };
    }

    /// <summary>
    /// This is the main entry point for your service instance.
    /// </summary>
    /// <param name="cancellationToken">
    /// Canceled when Service Fabric needs to shut down this service instance.
    /// </param>
    protected override async Task RunAsync(CancellationToken cancellationToken)
    {
        while (true)
        {
            cancellationToken.ThrowIfCancellationRequested();
            await Task.Delay(TimeSpan.FromSeconds(10), cancellationToken);
        }
    }
}
```
