# 最新版本-1.3.1

[谢尔盖·拜科夫（Sergey Bykov）](https://github.com/sergeybykov)2016/12/1下午5:48:39

* * *

11月15日，我们发布了我们的[最新版本-1.3.1](https://github.com/dotnet/orleans/releases/tag/v1.3.1)。此修补程序版本自1.3.0起已合并到master中，其中包含许多错误修复和改进。1.3.1有两个主要原因。

-   343 Industries需要一个在流传输方面进行了一些改进的版本，以及EventHub流提供程序，以简化从Halo 5发布之前一直在运行的流发行堆栈的预发行版本的迁移。
-   [奥尔良卡](https://github.com/OrleansContrib/Orleankka)需要一个相当高级的功能，使他们可以在每个消息的基础上控制请求的交织。[@yevhen](https://github.com/yevhen)[提交了一份公关](https://github.com/dotnet/orleans/pull/2246)为此，经过几次设计和实现迭代。

因此，1.3.1不是纯补丁程序发行版，因为它包含一项新功能。我们认为这里还可以，因为该功能实际上对其他人没有影响。

如果要从1.2.x或更早的版本升级到1.3.1，请注意在1.3.0中所做的细微更改。的[1.3.0发行说明](https://github.com/dotnet/orleans/releases/tag/v1.3.0)喊出来：

**注意：此版本中有一个微妙的重大更改，很容易错过。**

*如果您正在使用`AzureSilo.Start（ClusterConfiguration配置，字符串DeploymentId）`在您的代码中，该重载已被删除，但是替换它的新重载具有相同的参数签名，但具有不同的第二个参数：`（ClusterConfiguration配置，字符串connectionString）`。现在必须将部署ID作为config参数的一部分传递：config.Globals.DeploymentId。这样就消除了传递两个不同的部署ID的不确定性，但是不幸的是，这需要付出破坏性的API更改的代价。*

1.3.0是一个相当大的版本，具有许多改进，错误修复以及地理分布的多集群的主要新功能。它的大部分内容都列在[1.3.0-beta2发行说明](https://github.com/dotnet/orleans/releases/tag/v1.3.0-beta2)。地理分布功能在[多集群支持部分](http://dotnet.github.io/orleans/Documentation/Multi-Cluster/Overview.html)文档。
