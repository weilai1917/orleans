---
layout: page
title: Migration from Orleans 1.5 to 2.0 when using Azure
---

# 使用Azure从Orleans 1.5迁移到2.0

在奥尔良2.0中，筒仓和客户端的配置已更改。在奥尔良1.5中，我们曾经有一个用于处理所有配置项的整体对象，而且提供程序也已添加到该配置对象中。在奥尔良2.0中，配置过程围绕`SiloHostBuilder`，类似于在ASP.NET Core中使用`WebHostBuilder`。

在奥尔良1.5中，Azure的配置如下所示：

```csharp
    var config = AzureSilo.DefaultConfiguration();
    config.AddMemoryStorageProvider();
    config.AddAzureTableStorageProvider("AzureStore", RoleEnvironment.GetConfigurationSettingValue("DataConnectionString"));
```

的`AzureSilo`类公开了一个名为DefaultConfiguration（）的静态方法，该方法用于加载配置XML文件。不建议使用这种配置筒仓的方式，但仍通过[旧版支持包](https://www.nuget.org/packages/Microsoft.Orleans.Core.Legacy/)。

在奥尔良2.0中，配置完全是编程的。新的配置API如下所示：

```csharp
    //Load the different settings from the services configuration
    var proxyPort = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["OrleansProxyEndpoint"].IPEndpoint.Port;
    var siloEndpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["OrleansSiloEndpoint"].IPEndpoint;
    var connectionString = RoleEnvironment.GetConfigurationSettingValue("DataConnectionString");
    var deploymentId = RoleEnvironment.DeploymentId;


    var builder = new SiloHostBuilder()
        //Set service ID and cluster ID
        .Configure<ClusterOptions>(options => 
            {
                options.ClusterId = deploymentId;
                options.ServiceIs = "my-app";
            })
        // Set silo name
        .Configure<SiloOptions>(options => options.SiloName = this.Name)
        //Then, we can configure the different endpoints
        .ConfigureEndpoints(siloEndpoint.Address, siloEndpoint.Port, proxyPort)
        //Then, we set the connection string for the storage
        .UseAzureStorageClustering(options => options.ConnectionString = connectionString)
        //If reminders are needed, add the service, the connection string is required
        .UseAzureTableReminderService(connectionString)
        //If Queues are needed, add the service, set the name and the Adapter, the one shown here
        //is the one provided with Orleans, but it can be a custom one
        .AddAzureQueueStreams<AzureQueueDataAdapterV2>("StreamProvider",
            configurator => configurator.Configure(configure =>
            {
                configure.ConnectionString = connectionString;
            }))
        //If Grain Storage is needed, add the service and set the name
        .AddAzureTableGrainStorage("AzureTableStore");
```

# AzureSilo到ISiloHost

在奥尔良1.5，`AzureSilo`类是在Azure Worker角色中托管筒仓的推荐方法。仍然可以通过[`Microsoft.Orleans.Hosting.AzureCloudServices`NuGet包](https://www.nuget.org/packages/Microsoft.Orleans.Hosting.AzureCloudServices/)。

```csharp
public class WorkerRole : RoleEntryPoint
{
    AzureSilo silo;

    public override bool OnStart()
    {
        // Do other silo initialization – for example: Azure diagnostics, etc
        return base.OnStart();
    }

    public override void OnStop()
    {
        silo.Stop();
        base.OnStop();
    }

    public override void Run()
    {
        var config = AzureSilo.DefaultConfiguration();
        config.AddMemoryStorageProvider();
        config.AddAzureTableStorageProvider("AzureStore", RoleEnvironment.GetConfigurationSettingValue("DataConnectionString"));

        // Configure storage providers
        silo = new AzureSilo();
        bool ok = silo.Start(config);

        silo.Run(); // Call will block until silo is shutdown
    }
}
```

奥尔良2.0提供了更灵活的模块化API，用于通过以下方式配置和托管筒仓`SiloHostBuilder`和`ISiloHost`。

```csharp
    public class WorkerRole : RoleEntryPoint
    {
        private ISiloHost host;
        private ISiloHostBuilder builder;
        private readonly CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();
        private readonly ManualResetEvent runCompleteEvent = new ManualResetEvent(false);

        public override void Run()
        {
            try
            {
                this.RunAsync(this.cancellationTokenSource.Token).Wait();
                runCompleteEvent.WaitOne();
            }
            finally
            {
                this.runCompleteEvent.Set();
            }
        }

        public override bool OnStart()
        {
            //builder is the SiloHostBuilder from the first section
            // Build silo host, so that any errors will restart the role instance
            this.host = this.builder.Build();

            return base.OnStart();
        }

        public override void OnStop()
        {
            this.cancellationTokenSource.Cancel();
            this.runCompleteEvent.WaitOne();

            this.host.StopAsync().Wait();

            base.OnStop();
        }

        private Task RunAsync(CancellationToken cancellationToken)
        {
            return this.host.StartAsync(cancellationToken);
        }
    }
```
