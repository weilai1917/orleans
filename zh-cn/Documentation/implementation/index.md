---
layout: page
title: Implementation Details
---

# 实施细节概述

## [Orleans生命周期](orleans_lifecycle.md)

Orleans的某些行为非常复杂，因此需要有序地启动和关闭。为了解决这个问题，引入了通用组件生命周期模式。

## [消息传递保证](messaging_delivery_guarantees.md)

Orleans消息传递保证是**最多一次**， 默认。(可选)如果配置为在超时后重试，则Orleans提供至少一次交付。

## [排程器](scheduler.md)

Orleans Scheduler是Orleans运行时中的一个组件，负责执行应用程序代码和部分运行时代码，以确保单线程执行语义。

## [集群管理](cluster_management.md)

Orleans通过内置的成员资格协议(有时称为“silos成员资格”)提供集群管理。该协议的目标是让所有孤岛(Orleans服务器)就当前活动的孤岛集达成共识，检测出故障的孤岛，并允许新的孤岛加入集群。

## [流实施](streams_implementation.md)

本节提供了Orleans Stream实施的高级概述。它描述了在应用程序级别上不可见的概念和细节。

## [负载均衡](load_balancing.md)

从广义上讲，负载平衡是Orleans运行时的支柱之一。

## [单元测试](testing.md)

本节说明如何对Grains进行单元测试，以确保其行为正确。
