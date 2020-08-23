---
layout: page
title: Troubleshooting Deployments
---

# 部署疑难解答

此页面提供了一些常规指南，用于解决部署到azure云服务时出现的任何问题。这些都是非常常见的问题需要注意。一定要检查日志以获取更多信息。

## 获取silounavailableexception

在尝试初始化客户机之前，首先检查以确保实际启动了silos。有时，silos需要很长时间才能启动，因此尝试多次初始化客户机是有益的。如果它仍然抛出异常，那么silos可能还有另一个问题。

检查silos配置并确保silos正确启动。

## 常见的连接字符串问题

-   在部署到azure时使用本地连接字符串-网站将无法连接
-   为silos和前端（web和worker角色）使用不同的连接字符串-网站将无法初始化客户端，因为它无法连接到silos

可以在azure门户中检查连接字符串配置。如果连接字符串设置不正确，日志可能无法正确显示。

## 不正确地修改配置文件

请确保在serviceDefinition.csdef文件中配置了正确的终结点，否则部署将无法工作。它将给出错误，说明它无法获取端点信息。

## 丢失的日志

确保连接字符串设置正确。

很可能Web角色中的web.config文件或工作角色中的app.config文件修改不正确。这些文件中的不正确版本可能导致部署问题。处理更新时要小心。

## 版本问题

确保解决方案中的每个项目都使用相同版本的Orleans。不这样做会导致工人角色的回收。查看日志以获取更多信息。visual studio在部署历史记录中提供了一些silos启动错误消息。

## 角色不断循环

-   检查所有适当的orleans程序集是否在解决方案中，并将copy local设置为true。
-   检查日志以查看初始化时是否存在未处理的异常。
-   确保连接字符串正确。
-   有关详细信息，请查看azure云服务疑难解答页面。

## 如何检查日志

-   使用visual studio中的云资源管理器导航到存储帐户中相应的存储表或blob。Wadlogstable是查看日志的良好起点。
-   您可能只记录了错误。如果还需要信息日志，则需要修改配置以设置日志严重性级别。

编程配置：

-   当创建`群集配置`对象，设置`config.defaults.defaulttracelevel=严重性。信息`是的。
-   当创建`客户端配置`对象，设置`config.defaulttracelevel=严重性。信息`是的。

声明性配置：

-   添加`<tracing defaulttracelevel=“info”/>`致`orleansconfiguration.xml文件`和/或`客户端配置.xml`文件夹。

在`诊断.wadcfgx`文件中的web和worker角色，请确保设置`scheduledTransferLogLevelFilter`中的属性`原木`元素到`问询处`，因为这是跟踪筛选的附加层，它定义将哪些跟踪发送到`瓦德洛斯泰布尔`在azure存储中。

您可以在《配置指南》中找到有关此的更多信息。

## 与asp.net的兼容性

包含在asp.net中的razor视图引擎使用与orleans相同的代码生成程序集（`Microsoft.code分析`和`微软代码分析.csharp`）中。这可能会在运行时出现版本兼容性问题。

若要解决此问题，请尝试升级`Microsoft.codedom.providers.dotnetCompilerPlatform公司`（这是ASP.NET用于包含上述程序集的NuGet包）到最新版本，并设置绑定重定向，如下所示：

```xml
<dependentAssembly>
  <assemblyIdentity name="Microsoft.CodeAnalysis.CSharp" publicKeyToken="31bf3856ad364e35" culture="neutral" />
  <bindingRedirect oldVersion="0.0.0.0-2.0.0.0" newVersion="1.3.1.0" />
</dependentAssembly>
<dependentAssembly>
  <assemblyIdentity name="Microsoft.CodeAnalysis" publicKeyToken="31bf3856ad364e35" culture="neutral" />
  <bindingRedirect oldVersion="0.0.0.0-2.0.0.0" newVersion="1.3.1.0" />
</dependentAssembly>
```
