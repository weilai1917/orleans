---
layout: page
title: Configuring ADO.NET Providers
---

# 配置ADO.NET提供程序

任何可靠的Orleans部署都需要使用持久存储来保持系统状态，特别是Orleans集群成员表和提醒。可用的选项之一是通过ADO.NET提供程序使用SQL数据库。

为了使用ado.net进行持久化、集群或提醒，需要将ado.net提供程序配置为silos配置的一部分，如果是集群，则还需要配置为客户端配置的一部分。

silos配置代码应如下所示：

```c#
var siloHostBuilder = new SiloHostBuilder();
var invariant = "System.Data.SqlClient"; // for Microsoft SQL Server
var connectionString = "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Orleans;Integrated Security=True;Pooling=False;Max Pool Size=200;Asynchronous Processing=True;MultipleActiveResultSets=True";
//use AdoNet for clustering 
siloHostBuilder.UseAdoNetClustering(options =>
            {
                options.Invariant = invariant;
                options.ConnectionString = connectionString;
            });
//use AdoNet for reminder service
siloHostBuilder.UseAdoNetReminderService(options =>
            {
                options.Invariant = invariant;
                options.ConnectionString = connectionString;
            });
//use AdoNet for Persistence
siloHostBuilder.AddAdoNetGrainStorage("GrainStorageForTest", options =>
            {
                options.Invariant = invariant;
                options.ConnectionString = connectionString;
            });
```

客户端配置代码应如下所示：

```c#
var siloHostBuilder = new SiloHostBuilder();
var invariant = "System.Data.SqlClient";
var connectionString = "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Orleans;Integrated Security=True;Pooling=False;Max Pool Size=200;Asynchronous Processing=True;MultipleActiveResultSets=True";
//use AdoNet for clustering 
siloHostBuilder.UseAdoNetClustering(options =>
            {
                options.Invariant = invariant;
                options.ConnectionString = connectionString;
            });
```

其中`连接串`设置为有效的ADONET服务器连接字符串。

为了使用ado.net提供程序进行持久化、提醒或集群，有用于创建数据库工件的脚本，所有将托管Orleanssilos的服务器都需要访问这些脚本。缺乏对目标数据库的访问是我们看到的开发人员所犯的一个典型错误。

在ADONET扩展名NUGETS上安装或执行NUGET还原后，脚本将复制到项目目录或leansADONETcontent，其中每个受支持的ADO.NET扩展名都有自己的目录。我们将adonet nuget分为每个功能nuget：`Microsoft.Orleans.Clustering.adonet公司`对于集群，`Microsoft.Orleans.Persistence.adonet公司`为了坚持和`Microsoft.Orleans.Reminders.adonet公司`作为提醒。
