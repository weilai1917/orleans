---
layout: page
title: List of Options Classes
---

# 选项类列表

用于配置Orleans的所有选项类都应该在`Orleans。配置`命名空间。他们中的许多人在`Orleans。接待`命名空间。

## iclientbuilder和isilohostbuilder的通用核心选项

| 选项类型 | 用于 |
| ---- | --- |
| `离合器选项` | 设置`棒状的`以及`服务ID` |
| `网络选项` | 为套接字和打开的连接设置超时值 |
| `序列化提供程序选项` | 设置序列化提供程序 |
| `类型管理选项` | 设置类型映射的刷新周期(请参阅异构silos和版本控制) |

## iclientbuilder特定选项

| 选项类型 | 用于 |
| ---- | --- |
| `客户信息选项` | 设置要保持打开的连接数，并指定要使用的网络接口 |
| `客户统计选项` | 设置与统计输出相关的各种设置 |
| `网关选项` | 设置可用网关列表的刷新周期 |
| `StaticGatewayListProviderOptions` | 设置客户端用于连接到群集的uri |

## IsiloHostBuilder特定选项

| 选项类型 | 用于 |
| ---- | --- |
| `ClusterMembership选项` | 群集成员身份的设置 |
| `一致性选项` | 一致散列算法的配置选项，用于平衡集群中的资源分配。 |
| `端点选项` | 设置silos终结点选项 |
| `GrainCollection选项` | Grains垃圾收集选项 |
| `grains转化选项` | 管理异构部署中的Grain实现选择 |
| `加载选项` | 减载配置设置。必须有`ihostenvironmentstatistics`例如通过`builder.UsePerfCounterEnvironmentStatistics()`(仅限Windows)用于`减载`发挥作用。 |
| `多色光` | 配置多群集支持的选项 |
| `性能调整选项` | 性能调整选项(网络、线程数) |
| `进程Exthand选项` | 在进程出口配置silos行为 |
| `计划选项` | 配置计划程序行为 |
| `Silomessagingoptions` | 配置与silos相关的全局消息传递选项。 |
| `silos选项` | 设置silos的名称 |
| `silos统计选项` | 设置与统计输出相关的各种设置 |
| `遥测选项` | 设置遥测用户设置 |
