---
layout: page
title: Configuring .NET Garbage Collection
---

# 配置.NET垃圾回收

为了获得良好的性能，必须以正确的方式为silos进程配置.NET垃圾回收。我们找到的最佳设置组合是设置gcserver=true和gcconcurrent=true。这些很容易通过应用程序csproj文件设置。示例如下：

## .NET框架和.NET核心

```csproj
// .csproj
<PropertyGroup>
  <ServerGarbageCollection>true</ServerGarbageCollection>
  <ConcurrentGarbageCollection>true</ConcurrentGarbageCollection>
</PropertyGroup>
```

## 具有旧.csproj项目格式的.NET框架

```xml
// App.config
<configuration>
  <runtime>
    <gcServer enabled="true"/>
    <gcConcurrent enabled="true"/>
  </runtime>
</configuration>
```

但是，如果silos作为azure工作器角色的一部分运行（默认情况下配置为使用工作站gc），这就不那么容易了。这篇博客文章展示了如何为azure工作者角色设置相同的配置-<https://blogs.msdn.microsoft.com/cclayton/2014/06/05/server-garbage-collection-mode-in-microsoft-azure/>

> [啊！注意][server garbage collection is available only on multiprocessor computers]（[https://msdn.microsoft.com/library/system.runtime.gcsettings.isservergc（v=vs.110）.aspx](https://msdn.microsoft.com/library/system.runtime.gcsettings.isservergc(v=vs.110).aspx)）中。因此，即使您通过应用程序csproj文件或通过引用的博客文章上的脚本配置垃圾回收，如果silos运行在具有单个核心的（虚拟）计算机上，您也不会从`gcserver=真`是的。
