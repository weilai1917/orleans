---
layout: page
title: Scheduler
---

# 排程器

Orleans Scheduler是Orleans运行时中的一个组件，负责执行应用程序代码和部分运行时代码，以确保**单线程执行语义**。它实现了自定义的TPL任务计划程序。

Orleans任务计划程序是一个2级层次的计划程序。在第一层是全球**OrleansTaskScheduler**负责执行系统活动。在第二级，每个Grains活化都有自己的**ActivationTaskScheduler**，它提供了单线程执行语义。

### 从高层次上讲，执行路径如下：

1.  请求到达正确的silos，并找到目标激活。
2.  请求在其ActivationTaskScheduler上转换为一个任务，排队等待该激活执行。
3.  通过标准TaskScheduler机制，作为谷类方法执行的一部分而创建的任何后续Task都会自然地排队到同一ActivationTaskScheduler中。
4.  每个ActivationTaskScheduler都有一个排队等待执行的任务队列。
5.  Orleans Scheduler具有一组由所有激活调度程序共同使用的工作线程。这些线程定期扫描所有调度程序队列以执行工作。
6.  线程接受一个队列（每个队列一次由一个线程接受）并开始按FIFO顺序在该队列中执行Tasks。
7.  一次将一个线程排入队列，然后依次执行Tasks的线程的组合提供了单线程执行语义。

### 工作项目：

Orleans使用工作项的概念来指定调度程序的入口点。最初，每个新请求都作为工作项入队，该工作项仅包装该请求的第一个任务的执行。工作项只是提供有关调度活动的更多上下文信息（调用方，活动的名称，日志记录），有时还需要代表该调度活动进行一些额外的工作（Invoke工作项中的调用后活动）。当前有以下工作项类型：1.调用工作项–这是最常用的工作项类型。它表示应用程序请求的执行。2.请求/响应工作项–执行系统请求（对SystemTarget的请求）。3. TaskWorkItem –代表排队到最高OrleansTaskScheduler的任务。只是为了方便数据结构而使用它而不是直接任务（下面有更多详细信息）。4. WorkItemGroup –共享同一调度程序的工作项组。用于为每个ActivationTaskScheduler包装任务队列。5. ClosureWorkItem –封装在排队到系统上下文的闭包（任意lambda）周围的包装器。

### 调度上下文：

调度上下文是一个标签，只是一个不透明的对象，代表调度目标–激活数据，系统目标或系统空上下文。

### 高级原则：

1.  任务总是排队到正确的调度程序

    1.1任务永远不会从一个调度程序转移到另一个调度程序。

    1.2我们绝不会代表其他任务创建任务来执行它们。

    1.3工作项包装在Task中（也就是说，为了执行工作项，我们创建了一个Task，其lambda函数将仅运行工作项lambda）。通过始终执行任务，我们确保通过适当的任务计划程序执行任何活动。

2.  使用base.TryExecute（而不是RunSynchronously）在计划了队列的调度程序上执行任务
3.  ATS，WorkItem组和调度上下文之间存在一对一的映射：

    3.1激活任务计划程序（ATS）是自定义的TPL计划程序。我们保持ATS稀薄，并将所有数据存储在WorkItemGroup中。ATS指向其WorkItemGroup。

    3.2 WorkItem组是激活任务的实际持有人（数据对象）。任务存储在列表中<Task>-ATS的所有任务的队列。WorkItemGroup指向其ATS。

### 数据流以及任务和工作项的执行：

1.  入口点始终是排队到OrleansTaskScheduler中的工作项。它可以是“调用/请求/响应/关闭工作项”之一。
2.  根据Task.Start的上下文包装到Task中，并放入正确的ActivationTaskScheduler中。
3.  排队到其ActivationTaskScheduler的任务被放入WorkItemGroup队列。
4.  将任务放入WorkItemGroup队列时，WorkItemGroup确保将其显示在OrleansTaskScheduler全局RunQueue中。RunQueue是可运行的WorkItemGroups的全局队列，这些队列中至少有一个Task已排队，因此可以执行。
5.  辅助线程扫描OrleansTaskScheduler的RunQueue，该队列保存WorkItemGroups并调用WorkItemGroups.Execute。
6.  WorkItemGroups.Execute扫描其任务队列，并通过ActivationTaskScheduler.RunTask（Task）执行它们

    6.1 ActivationTaskScheduler.RunTask（Task）调用base.TryExecute。

    6.2通过TPL直接排队到调度程序的任务将立即执行。

    6.3包装工作项的任务将调用workItem.Execute，它将执行Closure工作项委托。

### 低层设计–工作项目：

1.  在OrleansTaskScheduler中排队工作项是如何在Orleans运行时中开始每个请求的整个执行链的。这是我们进入调度程序的入口。
2.  工作项首先提交给OrleansTaskScheduler（因为这是呈现给系统其余部分的接口）。

    2.1这样只能提交关闭/调用/恢复工作项目。

    2.2无法将TaskWorkItem直接提交给OrleansTaskScheduler（有关处理TaskWorkItem的更多信息，请参见下文）。

3.  每个工作项都必须包装到Task中，并通过Task.Start排队到正确的调度程序中。

    3.1这将确保在执行此workItem期间隐式创建的任何Task上正确设置TaskScheduler.Current。

    3.2包装是通过WrapWorkItemAsTask创建一个Task来完成的，该Task将执行工作项，并通过Task.Start（scheduler）将其排队到正确的调度程序中。

    3.3空上下文的工作项将排队到OrleansTaskScheduler。

    3.4非空上下文的工作项将排队到ActivationTaskScheduler中。

### 低层设计–排队任务：

1.  任务直接排队到正确的调度程序中

    1.1任务由TPL通过受保护的重写void QueueTask（任务任务）隐式排队。

    1.2具有非空上下文的任务始终排队到ActivationTaskScheduler中

    1.3具有空上下文的任务始终排队到OrleansTaskScheduler

2.  将任务排队到ActivationTaskScheduler：

    2.1我们绝不会将一个任务包装在另一个任务中。任务被直接添加到工作项组队列中

3.  向OrleansTaskScheduler排队任务：

    3.1当Task排入OrleansTaskScheduler时，我们将其包装到TaskWorkItem中，并将其放入此计划程序的工作项队列中。

    3.2这仅是数据结构问题，无任何内在因素：

    3.3 OrleansTaskScheduler通常保存工作项组以安排它们，因此其RunQueue具有BlockingCollection<IWorkItem>。

    3.4由于空上下文的任务也排队到OrleansTaskScheduler中，因此我们重复使用相同的数据结构，因此必须将每个Task包装在TaskWorkItem中。

    3.5我们应该能够通过调整RunQueue数据结构完全摆脱这种包装。这可以稍微简化代码，但通常不重要。而且，将来无论如何我们都应该离开null上下文，所以无论如何这个问题都将消失

### 内联任务：

由于任务总是排队在正确的调度程序上，因此从理论上讲，内联任何任务应该总是安全的。
