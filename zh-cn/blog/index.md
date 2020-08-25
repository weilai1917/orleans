---
layout: page
title: Orleans Blog
---

## [解决交易绩效之谜](solving-a-transactions-performance-mystery.md)

[鲁本·邦德](https://github.com/ReubenBond)2018/12/7上午10:08:58

* * *

到达雷德蒙德并完成强制性的新员工入职培训后，我在Orleans团队的首要任务是协助进行一些持续的绩效调查，以确保内部人员可以使用Orleans的交易支持并因此而释放。

在针对我们的测试集群的压力/负载测试中，我们看到了严重的性能问题和大量事务失败。很大一部分事务一直暂停直到超时。

## [德米特里·瓦库连科(Dmitry Vakulenko)](dmitry-vakulenko.md)

[谢尔盖·拜科夫(Sergey Bykov)](https://github.com/sergeybykov)2018/11/19下午1:57:59

* * *

Dmitry Vakulenko三年前加入了Orleans开源社区，并开始提交侧重于提高Orleans代码库性能的请求。在核心团队的现任和前任成员之外，他成为了最多产的贡献者。

德米特里(Dmitry)也做出了其他改进，但他的激情始终是表现。由于增量优化的复合性质，随着时间的流逝，这些改进加起来达到了惊人的总和。我们的保守估计是，德米特里的贡献使Orleans的表现提高了约2.6倍。

## [宣布Orleans2.1](announcing-orleans-2.1.md)

[鲁本·邦德](https://github.com/ReubenBond)2018/10/1下午7:17:59

* * *

今天，我们宣布了Orleans2.1。此版本包括对2.0的重大性能改进，对分布式事务支持的重大更新，新的代码生成器以及用于共同托管方案的新功能，以及较小的修复和改进。阅读[在这里发布说明](https://github.com/dotnet/orleans/releases/tag/v2.1.0)。
