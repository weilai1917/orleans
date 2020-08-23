---
layout: page
title: Orleans Configuration Guide
---

# Orleans配置指南

本配置指南解释了关键配置参数以及它们在大多数典型使用场景中的使用方式。

orleans可以用于各种适合不同使用场景的配置，例如用于开发和测试的本地单节点部署、服务器群集、多实例azure工作者角色等。

本指南提供了在目标场景中运行Orleans所需的关键配置参数的说明。还有一些其他的配置参数，主要有助于微调Orleans以获得更好的性能。

通过`silohostbuilder软件`和`客户端生成器`以及一些补充期权类别。Orleans的选项类遵循[ASP.NET选项](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options/)模式，可通过文件、环境变量等加载。请参阅[选项模式文档](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options/)更多信息。

如果要为本地开发配置silos和客户端，请查看[本地开发配置](local_development_configuration.md)页：1茶[服务器配置](server_configuration.md)和[客户端配置](client_configuration.md)导游盖部分配置谷仓和顾客，各自。

第二节[典型配置](typical_configurations.md)提供一个少量共同配置的概要。

一份可配置的重要核心选项清单可在上面找到。[本节](list_of_options_classes.md)页：1

**重要的**确保你预先配置净垃圾邮件收藏[配置网收藏](configuring_.NET_garbage_collection.md)页：1
