---
title: Disruptions
date: 2020-07-29
publishdate: 2020-08-10
weight: 2030004
---
<!--
---
reviewers:
- erictune
- foxish
- davidopp
title: Disruptions
content_type: concept
weight: 60
---
 -->
<!-- overview -->
<!--
This guide is for application owners who want to build
highly available applications, and thus need to understand
what types of disruptions can happen to Pods.

It is also for cluster administrators who want to perform automated
cluster actions, like upgrading and autoscaling clusters.
 -->
本文主要给
那些需要构建高可用应用的应用所属者，需要理解 Pod 可能遇到哪些类型的故障
也适用于那些需要实现自动集群操作，如升级或集群自动扩容的集群管理员
<!-- body -->
<!--
## Voluntary and involuntary disruptions

Pods do not disappear until someone (a person or a controller) destroys them, or
there is an unavoidable hardware or system software error.

We call these unavoidable cases *involuntary disruptions* to
an application.  Examples are:

- a hardware failure of the physical machine backing the node
- cluster administrator deletes VM (instance) by mistake
- cloud provider or hypervisor failure makes VM disappear
- a kernel panic
- the node disappears from the cluster due to cluster network partition
- eviction of a pod due to the node being [out-of-resources](/docs/tasks/administer-cluster/out-of-resource/).

Except for the out-of-resources condition, all these conditions
should be familiar to most users; they are not specific
to Kubernetes.

We call other cases *voluntary disruptions*.  These include both
actions initiated by the application owner and those initiated by a Cluster
Administrator.  Typical application owner actions include:

- deleting the deployment or other controller that manages the pod
- updating a deployment's pod template causing a restart
- directly deleting a pod (e.g. by accident)

Cluster administrator actions include:

- [Draining a node](/docs/tasks/administer-cluster/safely-drain-node/) for repair or upgrade.
- Draining a node from a cluster to scale the cluster down (learn about
[Cluster Autoscaling](/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaler)
).
- Removing a pod from a node to permit something else to fit on that node.

These actions might be taken directly by the cluster administrator, or by automation
run by the cluster administrator, or by your cluster hosting provider.

Ask your cluster administrator or consult your cloud provider or distribution documentation
to determine if any sources of voluntary disruptions are enabled for your cluster.
If none are enabled, you can skip creating Pod Disruption Budgets.

{{< caution >}}
Not all voluntary disruptions are constrained by Pod Disruption Budgets. For example,
deleting deployments or pods bypasses Pod Disruption Budgets.
{{< /caution >}}

 -->

## 计划内和计划外的故障

Pod 只会在有人(真人或一个控制器)销毁时或有不可用的硬件或系统软件错误时才会消失。

我们把这类在应用上出现不可避免出现的情况称为 *计划外故障*， 例如:

- 节点所以的物理机出现硬件故障
- 集群管理误操作删除了虚拟机实例
- 云提供商或虚拟化软件故障导致虚拟机丢失
- 内核故障
- 因为网络分区导致节点与集群失联
- 因为节点[资源爆了](/k8sDocs/tasks/administer-cluster/out-of-resource/)导致 Pod 被驱逐

除了资源瀑掉的情况，其它的情况对大多数用户来说都是比较熟悉的， 并不是 k8s 独有的。

其它的情况就被称为 *计划内故障*， 包括由应用所有者发起的行为和集群管理员发起的行为。 常见的应用所有者行为有:

- 删除管理 Pod 的 Deployment 或其它控制器(controller)
- 更新 Pod 定义模板引起重启
- 直接删除 Pod (如，误删除)

集群管理发起的行为有:

- 因修复或升级 [节点清场](/k8sDocs/tasks/administer-cluster/safely-drain-node/)
- 因收缩集群容量而 节点清场
  (更多见[集群动态容量](/k8sDocs/tasks/administer-cluster/cluster-management/#cluster-autoscaler))
- 因要放入更适合该节点的其它东西而删除 Pod

询问系统管理或咨询云提供商或查看文档来确定集群是否开启了计划内故障

{{< caution >}}
不是所有的计划内故障都包含在 Pod 故障预算中。 例如， 删除 Deployment 或 Pod 绕过了 Pod 故障预算
{{< /caution >}}
<!--  
## Dealing with disruptions

Here are some ways to mitigate involuntary disruptions:

- Ensure your pod [requests the resources](/docs/tasks/configure-pod-container/assign-memory-resource) it needs.
- Replicate your application if you need higher availability.  (Learn about running replicated
  [stateless](/docs/tasks/run-application/run-stateless-application-deployment/)
  and [stateful](/docs/tasks/run-application/run-replicated-stateful-application/) applications.)
- For even higher availability when running replicated applications,
  spread applications across racks (using
  [anti-affinity](/docs/user-guide/node-selection/#inter-pod-affinity-and-anti-affinity-beta-feature))
  or across zones (if using a
  [multi-zone cluster](/docs/setup/multiple-zones).)

The frequency of voluntary disruptions varies.  On a basic Kubernetes cluster, there are
no voluntary disruptions at all.  However, your cluster administrator or hosting provider
may run some additional services which cause voluntary disruptions. For example,
rolling out node software updates can cause voluntary disruptions. Also, some implementations
of cluster (node) autoscaling may cause voluntary disruptions to defragment and compact nodes.
Your cluster administrator or hosting provider should have documented what level of voluntary
disruptions, if any, to expect.
-->
## 如何应对故障

可以通过以下方式缓解计划外故障:

- 保证 Pod [申请所需要的资源](/k8sDocs/tasks/configure-pod-container/assign-memory-resource)
- 如果需要高可以，部署多个应用副本。(更多关于如何运行
  [无状态](/k8sDocs/tasks/run-application/run-stateless-application-deployment/)
  [有状态](/k8sDocs/tasks/run-application/run-replicated-stateful-application/)
  应用
  )
- 如果多副本应用还需要更高的可用性， 让应用副本分布在不同的机架(
  通过 [anti-affinity](/k8sDocs/user-guide/node-selection/#inter-pod-affinity-and-anti-affinity-beta-feature)
  ) 或 不同的区域(
  如果使用 [多区域集群](/k8sDocs/setup/multiple-zones))
<!--
## Pod disruption budgets

{{< feature-state for_k8s_version="v1.5" state="beta" >}}

Kubernetes offers features to help you run highly available applications even when you
introduce frequent voluntary disruptions.

As an application owner, you can create a PodDisruptionBudget (PDB) for each application.
A PDB limits the number of Pods of a replicated application that are down simultaneously from
voluntary disruptions. For example, a quorum-based application would
like to ensure that the number of replicas running is never brought below the
number needed for a quorum. A web front end might want to
ensure that the number of replicas serving load never falls below a certain
percentage of the total.

Cluster managers and hosting providers should use tools which
respect PodDisruptionBudgets by calling the [Eviction API](/docs/tasks/administer-cluster/safely-drain-node/#the-eviction-api)
instead of directly deleting pods or deployments.

For example, the `kubectl drain` subcommand lets you mark a node as going out of
service. When you run `kubectl drain`, the tool tries to evict all of the Pods on
the Node you're taking out of service. The eviction request that `kubectl` submits on
your behalf may be temporarily rejected, so the tool periodically retries all failed
requests until all Pods on the target node are terminated, or until a configurable timeout
is reached.

A PDB specifies the number of replicas that an application can tolerate having, relative to how
many it is intended to have.  For example, a Deployment which has a `.spec.replicas: 5` is
supposed to have 5 pods at any given time.  If its PDB allows for there to be 4 at a time,
then the Eviction API will allow voluntary disruption of one (but not two) pods at a time.

The group of pods that comprise the application is specified using a label selector, the same
as the one used by the application's controller (deployment, stateful-set, etc).

The "intended" number of pods is computed from the `.spec.replicas` of the workload resource
that is managing those pods. The control plane discovers the owning workload resource by
examining the `.metadata.ownerReferences` of the Pod.

PDBs cannot prevent [involuntary disruptions](#voluntary-and-involuntary-disruptions) from
occurring, but they do count against the budget.

Pods which are deleted or unavailable due to a rolling upgrade to an application do count
against the disruption budget, but workload resources (such as Deployment and StatefulSet)
are not limited by PDBs when doing rolling upgrades. Instead, the handling of failures
during application updates is configured in the spec for the specific workload resource.

When a pod is evicted using the eviction API, it is gracefully
[terminated](/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination), honoring the
`terminationGracePeriodSeconds` setting in its [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core).)
 -->
## Pod 故障预算

{{< feature-state for_k8s_version="v1.5" state="beta" >}}

k8s 提供了即便在频繁有计划内故障的情况下依然可以运行高可用应用的特性。

作为一个应用所有者，用户可以为每个应用创建一个 `PodDisruptionBudget` (PDB)，
PDB 会限制在计划内故障时应用副本的 Pod 同时挂掉的数量。例如，一个基于选举的应用，就必须要保证
运行的副本数不能少于选举所需要的数量。 一个WEB前端可能需要保证提供服务的副本数量不能少于某个百分比

集群管理器和主机提供都应该使用工具来调用遵循 `PodDisruptionBudget` 的
[Eviction API](/k8sDocs/tasks/administer-cluster/safely-drain-node/#the-eviction-api)
而不是直接删除 Pod 或 Deployment.

例如，`kubectl drain` 命令将节点标签为即将停止服务。 当运行 `kubectl drain` 后，
工具将尝试将停止服务的节点上所有的 Pod 驱逐掉。 由 kubelet 发起的驱逐请求可能会临时被拒绝，
所以工具会对失败的请求周期性重试直到节点上所有的 Pod 被终止或最终超时(超时时间可配置)。

PDB 指定一个应用可以接受的最少同时正常运行的副本数量。 例如，一个 Deployment 上定义为 `.spec.replicas: 5`
也就是任意时间都要有5个正常运行的副本。 如果它的 PDB 允许同时至少要有 4 个副本， 则 驱逐 API
允许故同一时间内的计划内障数为一(不是二)

而应用是通过特定标签选择器匹配到的 Pod 组成的， 与应用的控制器(deployment, stateful-set,等)一样

Pod 的预期数量是通过管理这些 Pod 的工作负载资源上的 `.spec.replicas` 计算等到的。
控制中心通过 Pod 上的 `.metadata.ownerReferences` 来查看它的拥有者。

PDB 不能阻止 [计划内故障](#voluntary-and-involuntary-disruptions)的发生，
但可以让它发生在预算控制内。

由应用滚动更新造成的 Pod 删除或不可用是遵照故障预算的， 但工作负载(如 Deployment 和 StatefulSet)
的滚动更新则不受 PDB 限制。 而是由工作负载中的配置来处理更新失败的。

当一个 Pod 因使用 驱逐 API而被驱逐， 会平滑地被 [终止](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
或  [PodSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core).)
中配置的 `terminationGracePeriodSeconds` 后被杀死。

{{<todo-optimize>}}
<!--
## PodDisruptionBudget example {#pdb-example}

Consider a cluster with 3 nodes, `node-1` through `node-3`.
The cluster is running several applications.  One of them has 3 replicas initially called
`pod-a`, `pod-b`, and `pod-c`.  Another, unrelated pod without a PDB, called `pod-x`, is also shown.
Initially, the pods are laid out as follows:

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *available*   | pod-b *available*   | pod-c *available*  |
| pod-x  *available*   |                     |                    |

All 3 pods are part of a deployment, and they collectively have a PDB which requires
there be at least 2 of the 3 pods to be available at all times.

For example, assume the cluster administrator wants to reboot into a new kernel version to fix a bug in the kernel.
The cluster administrator first tries to drain `node-1` using the `kubectl drain` command.
That tool tries to evict `pod-a` and `pod-x`.  This succeeds immediately.
Both pods go into the `terminating` state at the same time.
This puts the cluster in this state:

|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* |                     |                    |

The deployment notices that one of the pods is terminating, so it creates a replacement
called `pod-d`.  Since `node-1` is cordoned, it lands on another node.  Something has
also created `pod-y` as a replacement for `pod-x`.

(Note: for a StatefulSet, `pod-a`, which would be called something like `pod-0`, would need
to terminate completely before its replacement, which is also called `pod-0` but has a
different UID, could be created.  Otherwise, the example applies to a StatefulSet as well.)

Now the cluster is in this state:

|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* | pod-d *starting*    | pod-y              |

At some point, the pods terminate, and the cluster looks like this:

|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *starting*    | pod-y              |

At this point, if an impatient cluster administrator tries to drain `node-2` or
`node-3`, the drain command will block, because there are only 2 available
pods for the deployment, and its PDB requires at least 2.  After some time passes, `pod-d` becomes available.

The cluster state now looks like this:

|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *available*   | pod-y              |

Now, the cluster administrator tries to drain `node-2`.
The drain command will try to evict the two pods in some order, say
`pod-b` first and then `pod-d`.  It will succeed at evicting `pod-b`.
But, when it tries to evict `pod-d`, it will be refused because that would leave only
one pod available for the deployment.

The deployment creates a replacement for `pod-b` called `pod-e`.
Because there are not enough resources in the cluster to schedule
`pod-e` the drain will again block.  The cluster may end up in this
state:

|    node-1 *drained*  |       node-2        |       node-3       | *no node*          |
|:--------------------:|:-------------------:|:------------------:|:------------------:|
|                      | pod-b *terminating* | pod-c *available*  | pod-e *pending*    |
|                      | pod-d *available*   | pod-y              |                    |

At this point, the cluster administrator needs to
add a node back to the cluster to proceed with the upgrade.

You can see how Kubernetes varies the rate at which disruptions
can happen, according to:

- how many replicas an application needs
- how long it takes to gracefully shutdown an instance
- how long it takes a new instance to start up
- the type of controller
- the cluster's resource capacity
 -->
## PodDisruptionBudget 使用示例

假设有一个三个节点的集群，节点名称依次为 `node-1` 到 `node-3`, 集群中运行了多个应用。
其中一个应用包含三个副本， 初始叫 `pod-a`, `pod-a`, `pod-c`. 还有一个不相关的 Pod 不包含 PDB
叫 pod-x,初始分布如下:

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *available*   | pod-b *available*   | pod-c *available*  |
| pod-x  *available*   |                     |                    |

三个Pod 都属于一个 Deployment, 且共同拥有一个PDB，要求3个中至少有2个 Pod 始终可用。

例如， 假设集群管理员想要更新内核版本以修复一个 bug, 所以需要重启。
集群管理员第一步通过 `kubectl drain` 命令 尝试对 node-1 清场。
这时会尝试驱逐 `pod-a` 和 `pod-x`。 这步操作应该立即能成功。 两个 Pod 都会同时进入 `terminating` 状态。
集群状态变更为:

|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* |                     |                    |

Deployment 发现它有一个 Pod 正在被终止， 所以它会创建一个替代 Pod 叫 `pod-d`,
因为 node-1 不可用， 所以会被调度到其它节点上。 其它某个控制器或工具也会创建一个 `pod-y` 替代 `pod-x`

{{<note>}}
对于一个 StatefulSet， pod-a 对应的名称应该是 pod-0, 且需要被替换的 Pod 完全终止后，才会创建替代的 Pod 仍叫 pod-0, 但 UID 不一样。
如此这样，这个示例也对 StatefulSet 适用。
{{</note>}}

集群状态再次变更为:

|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* | pod-d *starting*    | pod-y              |

一段时间后， Pod 被终止，集群状态变更为:

|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *starting*    | pod-y              |

这时如果遇到一个急躁的集群管理尝试去对 `node-2` 或 `node-3` 进行清场， 则清场命令会被阻塞，
因为 Deployment 只有两个可用的 Pod， 而 PDB 要求至少要两个。 一段时间之后， `pod-d` 状态变更为可用

集群状态变更为:

|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *available*   | pod-y              |

此时，假设集群管理员清场的是 node-2. 清场命令会以某个顺序驱逐这两个 Pod， 假设先是 pod-b, 然后是 pod-d.
对 pod-b 的驱逐会成功， 但是对 pod-d 的驱逐会被拒绝，因为如果 pod-d 被终止， Deployment 就只剩下一个可用 Pod。

Deployment 会创建一个叫 pod-e 来替代 pod-b. 因为集群中没有足够的资源来让 pod-e 被调度， 清场命令会再次被阻塞。
最终集群的状态为成为这样:

|    node-1 *drained*  |       node-2        |       node-3       | *no node*          |
|:--------------------:|:-------------------:|:------------------:|:------------------:|
|                      | pod-b *terminating* | pod-c *available*  | pod-e *pending*    |
|                      | pod-d *available*   | pod-y              |                    |

至些， 集群管理员需要把完成升级的节点加入到集群中。

k8s 可以接受的各种故障可能发生的频次，由如下因素决定:

- 应用必须要多少个副本
- 一个实例平滑关闭所需要的时间
- 一个新实例启动需要多少时间
- 控制器的类型
- 集群资源的容量
<!--
## Separating Cluster Owner and Application Owner Roles

Often, it is useful to think of the Cluster Manager
and Application Owner as separate roles with limited knowledge
of each other.   This separation of responsibilities
may make sense in these scenarios:

- when there are many application teams sharing a Kubernetes cluster, and
  there is natural specialization of roles
- when third-party tools or services are used to automate cluster management

Pod Disruption Budgets support this separation of roles by providing an
interface between the roles.

If you do not have such a separation of responsibilities in your organization,
you may not need to use Pod Disruption Budgets.
 -->
## 集群管理员和应用管理员的角色划分

通常，将集群管理员和应用管理当作不同的角色，是很有用的。
对责任和区分在以下场景很有意义：

- 当集群中有多个应用团队共同使用该集群， 只预置的角色。
- 当第三方工作或服务对集群进行自动化管理

Pod 故障预算通过角色之间的接口来实现对角色区分的支持

如果用户组织中没有对责任区分的需求，则可能不震要使用 Pod 故障预算

{{<todo-optimize>}}
<!--
## How to perform Disruptive Actions on your Cluster

If you are a Cluster Administrator, and you need to perform a disruptive action on all
the nodes in your cluster, such as a node or system software upgrade, here are some options:

- Accept downtime during the upgrade.
- Failover to another complete replica cluster.
   -  No downtime, but may be costly both for the duplicated nodes
     and for human effort to orchestrate the switchover.
- Write disruption tolerant applications and use PDBs.
   - No downtime.
   - Minimal resource duplication.
   - Allows more automation of cluster administration.
   - Writing disruption-tolerant applications is tricky, but the work to tolerate voluntary
     disruptions largely overlaps with work to support autoscaling and tolerating
     involuntary disruptions.
 -->

## 怎么在集群中执行干扰操作

如果用户为集群管理员，需要对集群中的所有节点执行干扰操作, 使用对节点或系统软件升级， 以下为一些选项：

- 可以接收升级期间停止服务
- 故障转移到另一个完整的副本集群
  - 没有宕机时间， 但可能需要双倍的节点和运行人员来实现精细的切换
- 编写可忍受干扰的应用并使用 PDB。
  - 没有宕机时间
  - 最少资源重复
  - 允许更多集群自动化管理
  - 编写可忍受干扰的应用是相当难的。但实现对计划内故障的忍受，则能够很大程度上覆盖了支持自动伸缩容量和忍受计划外故障的工作内容

{{<todo-optimize>}}
## {{% heading "whatsnext" %}}

<!--
* Follow steps to protect your application by [configuring a Pod Disruption Budget](/docs/tasks/run-application/configure-pdb/).

* Learn more about [draining nodes](/docs/tasks/administer-cluster/safely-drain-node/)

* Learn about [updating a deployment](/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
  including steps to maintain its availability during the rollout.
 -->
- 实践 [配置 PodDisruptionBudget](/k8sDocs/tasks/run-application/configure-pdb/).
- 实践 [节点清场](/k8sDocs/tasks/administer-cluster/safely-drain-node/)
- 了解 [更新 Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
  包括在回滚时用哪些步骤保持其可用性
