---
layout: page
title: Activation Garbage Collection
---

# 激活垃圾回收

如核心概念部分所述，*grains活化*是一个grain类的内存实例，由orleans运行时根据需要自动创建，作为grain的临时物理实施例。

激活垃圾收集（activation gc）是从内存中删除未使用的grains激活的过程。它在概念上类似于.net中的内存垃圾收集工作方式。然而，激活gc只考虑特定grains激活空闲的时间。内存使用率不是一个因素。

## 激活gc的工作原理

激活gc的一般过程包括在silos中的orleans运行时定期扫描在配置的时间段（收集期限）内根本没有使用的Grains激活。一旦Grains激活闲置了那么长时间，它就会被停用。停用过程开始于运行时调用grain的`OnDeactivateAsync()`方法，并通过从silos的所有数据结构中移除对Grain Activation对象的引用来完成，以便.NET GC回收内存。

因此，在不给应用程序代码增加负担的情况下，只有最近使用的grains激活会保留在内存中，而不再使用的激活会被自动删除，它们使用的系统资源会被运行时回收。

**对于Grains活化收集而言，什么算是“活跃”**

-   接收方法调用
-   收到提醒
-   通过流接收事件

**对于Grains活化收集而言，什么不算“活跃”**

-   执行访问（对另一个Grains或对一个Orleans客户）
-   定时器事件
-   不涉及orleans框架的任意io操作或外部调用

**收款年龄限制**

在这段时间之后，闲置Grains的激活会受到激活GC的影响，这段时间称为收集期限限制。默认的收集期限为2小时，但可以全局更改，也可以针对单个Grains类更改。

## 显式控制激活垃圾回收

### 延迟激活gc

Grains激活可以通过调用`this.delaydeactivation()`方法：

```csharp
protected void DelayDeactivation(TimeSpan timeSpan)
```

此调用将确保此激活至少在指定的持续时间内不被停用。它的优先级高于配置中指定的激活垃圾回收设置，但不会取消这些设置。因此，此调用为**将停用延迟到激活垃圾收集设置中指定的时间之外**是的。此调用不能用于加速激活垃圾收集。

积极的`时间跨度`值表示“在该时间段内阻止此激活的GC”。

否定的`时间跨度`值表示“取消`延迟停用`调用并使此激活行为基于常规激活垃圾收集设置”。

**情节：**

1）激活垃圾收集设置指定10分钟的期限，Grains正在调用`延迟停用（TimeSpan.FromMinutes（20））`，它将导致至少20分钟内无法收集此激活。

2）激活垃圾收集设置指定10分钟的期限，Grains正在调用`延迟停用（TimeSpan.FromMinutes（5））`，如果没有额外访问，则激活将在10分钟后收集。

3）激活垃圾收集设置指定10分钟的期限，Grains正在调用`延迟停用（TimeSpan.FromMinutes（5））`，7分钟后，如果没有额外的调用，则会在17分钟后从时间0开始收集激活。

4）激活垃圾收集设置指定10分钟的期限，Grains正在调用`延迟停用（TimeSpan.FromMinutes（20））`，7分钟后，此Grains上还有另一个调用，如果没有额外的调用，则将在从时间0开始的20分钟后收集激活。

请注意`延迟停用`无法100%保证在指定的时间段到期之前不会停用Grains激活。有些失效案例可能导致Grains过早失活。也就是说`延迟停用` **不能用作永久“固定”内存中的grains激活或固定到特定silos的方法**是的。`延迟停用`这仅仅是一种优化机制，可以帮助降低Grains随着时间的推移被停用和重新激活的总成本，如果这很重要的话。在大多数情况下，不需要使用`延迟停用`完全。

### 加速活化气相色谱

Grains激活还可以通过调用`这个。停用空闲()`方法：

```csharp
protected void DeactivateOnIdle()
```

如果此时不处理任何消息，则认为Grains激活处于空闲状态。如果你打电话`停用空闲`当Grains正在处理消息时，一旦当前消息的处理完成，它将被停用。

如果有任何请求排队等待Grains，它们将被转发到下一个激活。

`停用空闲`优先于配置或`延迟停用`是的。请注意，此设置仅适用于调用它的grains激活，而不适用于此类型的其他grains激活。

## 配置

可以使用`GrainCollection选项`选项：

```csharp
mySiloHostBuilder.Configure<GrainCollectionOptions>(options =>
{
  // Set the value of CollectionAge to 10 minutes for all grain
  options.CollectionAge = TimeSpan.FromMinutes(10);
  // Override the value of CollectionAge to 5 minutes for MyGrainImplementation
  options.ClassSpecificCollectionAge[typeof(MyGrainImplementation).FullName] = TimeSpan.FromMinutes(5);
})
```
