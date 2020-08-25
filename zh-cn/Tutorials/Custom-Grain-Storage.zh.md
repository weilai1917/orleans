---
layout: page
title: Custom Grain Storage
---

# 自定义Grains存储

## 编写自定义Grains存储

在有关声明性参与者存储的教程中，我们研究了允许Grains使用内置存储提供程序之一将其状态存储在Azure表中。尽管Azure是松散数据的好地方，但还有许多替代方法。实际上，有太多的人无法支持所有人。取而代之的是，Orleans旨在让您通过编写Grains存储来轻松添加对您自己的存储形式的支持。

在本教程中，我们将逐步介绍如何编写基于文件的简单Grains存储。文件系统不是存储Grains状态的最佳位置，因为它是本地的，文件锁可能存在问题，并且最后更新日期不足以防止不一致。但这是一个简单的示例，可以帮助我们说明`Grains储存`。

## 入门

OrleansGrains仓库是实现`IGrain存储`包含在其中[Microsoft.Orleans.Core NuGet程序包](https://www.nuget.org/packages/Microsoft.Orleans.Core/)。

我们也从`ILifecycleParticipant <ISiloLifecycle>`这将使我们能够订阅孤岛生命周期中的特定事件。

我们首先创建一个名为`FileGrainStorage`。

```csharp
using Orleans;
using System;
using Orleans.Storage;
using Orleans.Runtime;
using System.Threading.Tasks;

namespace GrainStorage
{
    public class FileGrainStorage : IGrainStorage, ILifecycleParticipant<ISiloLifecycle>
    {
        private readonly string _storageName;
        private readonly FileGrainStorageOptions _options;
        private readonly ClusterOptions _clusterOptions;
        private readonly IGrainFactory _grainFactory;
        private readonly ITypeResolver _typeResolver;
        private JsonSerializerSettings _jsonSettings;

        public FileGrainStorage(string storageName, FileGrainStorageOptions options, IOptions<ClusterOptions> clusterOptions, IGrainFactory grainFactory, ITypeResolver typeResolver)
        {
            _storageName = storageName;
            _options = options;
            _clusterOptions = clusterOptions.Value;
            _grainFactory = grainFactory;
            _typeResolver = typeResolver;
        }

        public Task ClearStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
        {
            throw new NotImplementedException();
        }

        public Task ReadStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
        {
            throw new NotImplementedException();
        }

        public Task WriteStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
        {
            throw new NotImplementedException();
        }
  
        public void Participate(ISiloLifecycle lifecycle)
        {
            throw new NotImplementedException();
        }

        public void Participate(ISiloLifecycle lifecycle)
        {
            throw new NotImplementedException();
        }
    }
}
```

在开始实施之前，我们创建一个包含根目录的选项类，Grains状态文件将存储在该目录下。为此，我们将创建一个选项文件`FileGrainStorageOptions`：

```csharp
public class FileGrainStorageOptions
{
    public string RootDirectory { get; set; }
}
```

创建一个包含两个字段的构造函数，`storageName`指定使用此存储应该写入哪些纹理`[StorageProvider(ProviderName =“文件”)]`和`目录`这将是保存grains状态的目录。

`IGrain工厂`，`ITypeResolver`将在下一部分中使用，我们将在其中初始化存储。

我们也有两个选择，我们自己`FileGrainStorageOptions`和`集群选项`。实现存储功能将需要这些。

我们还需要`JsonSerializerSettings`因为我们正在以Json格式进行序列化和反序列化。

*Json是一个实现细节，由开发人员决定哪种串行化/反序列化协议适合该应用程序。另一种常见格式是二进制格式。*

## 初始化存储

要初始化存储，我们注册一个`在里面`功能上`应用服务`生命周期。

```csharp
public void Participate(ISiloLifecycle lifecycle)
{
    lifecycle.Subscribe(OptionFormattingUtilities.Name<FileGrainStorage>(_storageName), ServiceLifecycleStage.ApplicationServices, Init);
}
```

的`在里面`功能用于设置`_jsonSettings`将用于配置`杰森`序列化器。同时，我们创建文件夹来存储grains状态(如果尚不存在)。

```csharp
private Task Init(CancellationToken ct)
{
    // Settings could be made configurable from Options.
    _jsonSettings = OrleansJsonSerializer.UpdateSerializerSettings(OrleansJsonSerializer.GetDefaultSerializerSettings(_typeResolver, _grainFactory), false, false, null);

    var directory = new System.IO.DirectoryInfo(_rootDirectory);
    if (!directory.Exists)
        directory.Create();

    return Task.CompletedTask;
}
```

我们还提供了一个通用函数来构造文件名，以确保每个服务，GrainID和Grain类型的唯一性。

```csharp
private string GetKeyString(string grainType, GrainReference grainReference)
{
    return $"{_clusterOptions.ServiceId}.{grainReference.ToKeyString()}.{grainType}";
}
```

## 阅读状态

要读取grains状态，我们使用先前定义的函数获取文件名，并将其组合到来自选项的根目录中。

```csharp
public async Task ReadStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
{
    var fName = GetKeyString(grainType, grainReference);
    var path = Path.Combine(_options.RootDirectory, fName);

    var fileInfo = new FileInfo(path);
    if (!fileInfo.Exists)
    {
        grainState.State = Activator.CreateInstance(grainState.State.GetType());
        return;
    }

    using (var stream = fileInfo.OpenText())
    {
        var storedData = await stream.ReadToEndAsync();
        grainState.State = JsonConvert.DeserializeObject(storedData, _jsonSettings);
    }

    grainState.ETag = fileInfo.LastWriteTimeUtc.ToString();
}
```

我们使用`fileInfo.LastWriteTimeUtc`作为ETag，其他功能将使用该ETag进行不一致检查以防止数据丢失。

请注意，对于反序列化，我们使用`_jsonSettings`这是在`在里面`功能。这对于能够正确地序列化/反序列化状态很重要。

## 写作状态

写入状态类似于读取状态。

```csharp
public async Task WriteStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
{
    var storedData = JsonConvert.SerializeObject(grainState.State, _jsonSettings);

    var fName = GetKeyString(grainType, grainReference);
    var path = Path.Combine(_options.RootDirectory, fName);

    var fileInfo = new FileInfo(path);

    if (fileInfo.Exists && fileInfo.LastWriteTimeUtc.ToString() != grainState.ETag)
    {
        throw new InconsistentStateException($"Version conflict (WriteState): ServiceId={_clusterOptions.ServiceId} ProviderName={_storageName} GrainType={grainType} GrainReference={grainReference.ToKeyString()}.");
    }

    using (var stream = new StreamWriter(fileInfo.Open(FileMode.Create, FileAccess.Write)))
    {
        await stream.WriteAsync(storedData);
    }

    fileInfo.Refresh();
    grainState.ETag = fileInfo.LastWriteTimeUtc.ToString();
}
```

与阅读类似，我们使用`_jsonSettings`写状态。当前的ETag用于检查文件的UTC中的最后更新时间。如果日期不同，则意味着同一粒Grains的另一次激活会同时更改状态。在这种情况下，我们将`InconsistentStateException`这将导致当前激活被杀死，以防止覆盖先前由其他激活grains保存的状态。

## 清算国

如果文件存在，则清除状态将删除该文件。

```csharp
public Task ClearStateAsync(string grainType, GrainReference grainReference, IGrainState grainState)
{
    var fName = GetKeyString(grainType, grainReference);
    var path = Path.Combine(_options.RootDirectory, fName);

    var fileInfo = new FileInfo(path);
    if (fileInfo.Exists)
    {
        if (fileInfo.LastWriteTimeUtc.ToString() != grainState.ETag)
        {
            throw new InconsistentStateException($"Version conflict (ClearState): ServiceId={_clusterOptions.ServiceId} ProviderName={_storageName} GrainType={grainType} GrainReference={grainReference.ToKeyString()}.");
        }

        grainState.ETag = null;
        grainState.State = Activator.CreateInstance(grainState.State.GetType());
        fileInfo.Delete();
    }

    return Task.CompletedTask;
}
```

出于同样的原因`写状态`，我们在继续删除文件并重置ETag之前检查是否存在不一致，并检查当前ETag是否与上次写入时间UTC相同。

## 把它放在一起

之后，我们将创建一个工厂，该工厂将使我们可以将选项设置的范围限定在提供程序名称上，同时创建一个实例。`FileGrainStorage`简化向服务集合的注册。

```csharp
public static class FileGrainStorageFactory
{
    internal static IGrainStorage Create(IServiceProvider services, string name)
    {
        IOptionsSnapshot<FileGrainStorageOptions> optionsSnapshot = services.GetRequiredService<IOptionsSnapshot<FileGrainStorageOptions>>();
        return ActivatorUtilities.CreateInstance<FileGrainStorage>(services, name, optionsSnapshot.Get(name), services.GetProviderClusterOptions(name));
    }
}
```

最后，要注册Grains储存库，我们在`ISiloHostBuilder`在内部使用以下方式将Grains存储注册为命名服务`.AddSingletonNamedService(...)`，由提供的扩展`Orleans`。

```csharp
public static class FileSiloBuilderExtensions
{
    public static ISiloHostBuilder AddFileGrainStorage(this ISiloHostBuilder builder, string providerName, Action<FileGrainStorageOptions> options)
    {
        return builder.ConfigureServices(services => services.AddFileGrainStorage(providerName, options));
    }

    public static IServiceCollection AddFileGrainStorage(this IServiceCollection services, string providerName, Action<FileGrainStorageOptions> options)
    {
        services.AddOptions<FileGrainStorageOptions>(providerName).Configure(options);
        return services
            .AddSingletonNamedService(providerName, FileGrainStorageFactory.Create)
            .AddSingletonNamedService(providerName, (s, n) => (ILifecycleParticipant<ISiloLifecycle>)s.GetRequiredServiceByName<IGrainStorage>(n));
    }
}
```

我们的`FileGrainStorage`实现两个接口，`IGrain存储`和`ILifecycleParticipant <ISiloLifecycle>`因此，我们需要为每个接口注册两个命名服务：

```csharp
return services
    .AddSingletonNamedService(providerName, FileGrainStorageFactory.Create)
    .AddSingletonNamedService(providerName, (s, n) => (ILifecycleParticipant<ISiloLifecycle>)s.GetRequiredServiceByName<IGrainStorage>(n));
```

这使我们能够使用扩展名添加文件存储。`ISiloHostBuilder`：

```csharp
var silo = new SiloHostBuilder()
    .UseLocalhostClustering()
    .AddFileGrainStorage("File", opts =>
    {
        opts.RootDirectory = "C:/TestFiles";
    })
    .Build();
```

现在，我们将可以与供应商一起装饰Grains`[StorageProvider(ProviderName =“文件”)]`它将以谷粒状态存储在选项中设置的根目录中。
