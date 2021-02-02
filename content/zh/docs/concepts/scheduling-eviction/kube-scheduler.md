---
title: k8s 中的调度器
content_type: concept
weight: 10
---
<!--
---
title: Kubernetes Scheduler
content_type: concept
weight: 10
---
 -->
<!-- overview -->

在 k8s 中， 调度指的是搞清楚
{{< glossary_tooltip term_id="pod" >}}
是不是与
{{< glossary_tooltip term_id="node" >}}
匹配，当匹配了
{{< glossary_tooltip term_id="kubelet" >}}
才能在这个节点运行这个 Pod。
<!-- body -->
<!--
## Scheduling overview {#scheduling}

A scheduler watches for newly created Pods that have no Node assigned. For
every Pod that the scheduler discovers, the scheduler becomes responsible
for finding the best Node for that Pod to run on. The scheduler reaches
this placement decision taking into account the scheduling principles
described below.

If you want to understand why Pods are placed onto a particular Node,
or if you're planning to implement a custom scheduler yourself, this
page will help you learn about scheduling.
 -->

## 调度概览 {#scheduling}

一个调度器会监测新创建但还没分配节点的 Pod。对于这个调度器发现的每一个 Pod， 它就将负责为这个
Pod 找到其运行的最佳节点。 调度器在进程调度决策时会考量现面介绍的调度原则。

如果你想为明白为啥这个 Pod 会放到一个特定的节点，或你计划自己实现一个自定义调度器，本文会帮助你
学习调度。
<!--
## kube-scheduler

[kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/)
is the default scheduler for Kubernetes and runs as part of the
{{< glossary_tooltip text="control plane" term_id="control-plane" >}}.
kube-scheduler is designed so that, if you want and need to, you can
write your own scheduling component and use that instead.

For every newly created pod or other unscheduled pods, kube-scheduler
selects an optimal node for them to run on. However, every container in
pods has different requirements for resources and every pod also has
different requirements. Therefore, existing nodes need to be filtered
according to the specific scheduling requirements.

In a cluster, Nodes that meet the scheduling requirements for a Pod
are called _feasible_ nodes. If none of the nodes are suitable, the pod
remains unscheduled until the scheduler is able to place it.

The scheduler finds feasible Nodes for a Pod and then runs a set of
functions to score the feasible Nodes and picks a Node with the highest
score among the feasible ones to run the Pod. The scheduler then notifies
the API server about this decision in a process called _binding_.

Factors that need taken into account for scheduling decisions include
individual and collective resource requirements, hardware / software /
policy constraints, affinity and anti-affinity specifications, data
locality, inter-workload interference, and so on.
 -->

## kube-scheduler

[kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
是 k8s 默认的调度器，并作为
{{< glossary_tooltip term_id="control-plane" >}}
的一部分运行。
kube-scheduler 在设计时就考虑到，如果用户希望并且有必要就可以编写并使用自己的调度模块，来代替默认
的调度器。

对于每新创建的 Pod 或其它未调度的 Pod，kube-scheduler 选择一个最合适它们节点让他们运行。但是，
Pod 中的每个容器对资源有不同的保底要求同时每个 Pod 也有不同的要求。 因此，会根据这些调度要求过
滤现有的节点。

在一个集群中， 那些满足 Pod 调度要求的节点就叫做 _可用的_ 节点。 如果没有一个节点合适，则 Pod
在调度器为其找到可用的节点之前就是未调度状态。

调度器为一个 Pod 找到一系列可用的节点，然后对这些可用节点运行一系统函数进行算分，选择这些可用节点
中得分最高的可用节点来运行这个 Pod。 这时候调度器会通过一个叫做 _绑定(binding)_ 的过程通知 API 服务关于这次的决定

在做调度决策时需要考量的因素包含 单个各总体的资源需求， 硬件 / 软件 策略约束， 亲和性和反亲和性规范，
数据地区，内部工作负载干扰，等等。

<!--
### Node selection in kube-scheduler {#kube-scheduler-implementation}

kube-scheduler selects a node for the pod in a 2-step operation:

1. Filtering
1. Scoring

The _filtering_ step finds the set of Nodes where it's feasible to
schedule the Pod. For example, the PodFitsResources filter checks whether a
candidate Node has enough available resource to meet a Pod's specific
resource requests. After this step, the node list contains any suitable
Nodes; often, there will be more than one. If the list is empty, that
Pod isn't (yet) schedulable.

In the _scoring_ step, the scheduler ranks the remaining nodes to choose
the most suitable Pod placement. The scheduler assigns a score to each Node
that survived filtering, basing this score on the active scoring rules.

Finally, kube-scheduler assigns the Pod to the Node with the highest ranking.
If there is more than one node with equal scores, kube-scheduler selects
one of these at random.

There are two supported ways to configure the filtering and scoring behavior
of the scheduler:


1. [Scheduling Policies](/docs/reference/scheduling/policies) allow you to configure _Predicates_ for filtering and _Priorities_ for scoring.
1. [Scheduling Profiles](/docs/reference/scheduling/config/#profiles) allow you to configure Plugins that implement different scheduling stages, including: `QueueSort`, `Filter`, `Score`, `Bind`, `Reserve`, `Permit`, and others. You can also configure the kube-scheduler to run different profiles.

 -->

### kube-scheduler 中的节点选择 {#kube-scheduler-implementation}

kube-scheduler 为 Pod 选择一个节点有 2 步操作:

1. 过滤
2. 算分

_过滤_ 这一步是找到那些可以调度这个 Pod 的节点集合。 例如， PodFitsResources 过虑器检查这个
候选节点的可用资源是否满足 Pod 配置的资源要求。 在这一步之后，节点列表中还包含有任意可用节点；
通常会有不止一个。 如果列表是空，则这个 Pod 就不可调度。

在 _算分_ 这一步，调度器会将剩余的节点排名挑选最适合放置 Pod 的地方。 调度会为每个在上步中留在
列表中的节点分配一个分数， 基于这个分数来激活算分规则

最终， kube-scheduler 将这个 Pod 分配给排名第一的节点。如果有多个节点并列第一(分数一样)，
kube-scheduler 会从中随机选一个。

支持以下两种配置调度器的过滤和算分行为:

1. [调度策略](https://kubernetes.io/docs/reference/scheduling/policies)
  允许用户为过滤的 _Predicates_ 和为算分的 _优先级_
2. [调度方案](https://kubernetes.io/docs/reference/scheduling/config/#profiles)
  允许用户配置不同的插件，这些插件实现了不同的调度阶段，包括:
  `QueueSort`, `Filter`, `Score`, `Bind`, `Reserve`, `Permit`, 等。
  也可以配置 kube-scheduler 运行不同的方案

## {{% heading "whatsnext" %}}

<!--
* Read about [scheduler performance tuning](/docs/concepts/scheduling-eviction/scheduler-perf-tuning/)
* Read about [Pod topology spread constraints](/docs/concepts/workloads/pods/pod-topology-spread-constraints/)
* Read the [reference documentation](/docs/reference/command-line-tools-reference/kube-scheduler/) for kube-scheduler
* Learn about [configuring multiple schedulers](/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)
* Learn about [topology management policies](/docs/tasks/administer-cluster/topology-manager/)
* Learn about [Pod Overhead](/docs/concepts/scheduling-eviction/pod-overhead/)
* Learn about scheduling of Pods that use volumes in:
  * [Volume Topology Support](/docs/concepts/storage/storage-classes/#volume-binding-mode)
  * [Storage Capacity Tracking](/docs/concepts/storage/storage-capacity/)
  * [Node-specific Volume Limits](/docs/concepts/storage/storage-limits/)
 -->

* 概念 [调度器性能优化](/k8sDocs/docs/concepts/scheduling-eviction/scheduler-perf-tuning/)
* 概念 [Pod 拓扑散布约束](/k8sDocs/docs/concepts/workloads/pods/pod-topology-spread-constraints/)
* 文档 [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)
* 实践 [配置多个调度器](/k8sDocs/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)
* 实践 [拓扑管理策略](/k8sDocs/docs/tasks/administer-cluster/topology-manager/)
* 概念 [Pod 上限](/k8sDocs/docs/concepts/scheduling-eviction/pod-overhead/)
* 关于使用卷的 Pod 的调度:
  * [卷对拓扑的支持](/k8sDocs/docs/concepts/storage/storage-classes/#volume-binding-mode)
  * [存储容量跟踪](/k8sDocs/docs/concepts/storage/storage-capacity/)
  * [节点级别的卷限制](/k8sDocs/docs/concepts/storage/storage-limits/)
