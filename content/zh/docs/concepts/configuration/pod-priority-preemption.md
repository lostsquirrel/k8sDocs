---
title: Pod 优先级与抢占
content_type: concept
weight: 70
---
<!--
---
reviewers:
- davidopp
- wojtek-t
title: Pod Priority and Preemption
content_type: concept
weight: 70
---
 -->
<!-- overview -->

{{< feature-state for_k8s_version="v1.14" state="stable" >}}
<!--
[Pods](/docs/concepts/workloads/pods/) can have _priority_. Priority indicates the
importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the
scheduler tries to preempt (evict) lower priority Pods to make scheduling of the
pending Pod possible.
 -->

[Pod](/k8sDocs/docs/concepts/workloads/pods/) 是可以有 _优先级的_。 优先级表示一个 Pod
与其它 Pod 之间的相对重要程度。 如果一个 Pod 不能被调度， 调度器会尝试抢占(驱逐)一些(个)低优先
级的 Pod， 让这个等待的 Pod 有可能正常调度。

<!-- body -->

{{< warning >}}
<!--
In a cluster where not all users are trusted, a malicious user could create Pods
at the highest possible priorities, causing other Pods to be evicted/not get
scheduled.
An administrator can use ResourceQuota to prevent users from creating pods at
high priorities.

See [limit Priority Class consumption by default](/docs/concepts/policy/resource-quotas/#limit-priority-class-consumption-by-default)
for details.
 -->
当在一个并不是所有用户都可信的集群中，一个恶意用户可能创建拥有最高可能优先级的 Pod，导致其它的
Pod 被驱逐或无法调度。 管理员可以使用资源配额(ResourceQuota)来防止用户创建高优先级的 Pod.

详见
[对默认使用的 Priority Class进行限制](/k8sDocs/docs/concepts/policy/resource-quotas/#limit-priority-class-consumption-by-default)
{{< /warning >}}
<!--
## How to use priority and preemption

To use priority and preemption:

1.  Add one or more [PriorityClasses](#priorityclass).

1.  Create Pods with[`priorityClassName`](#pod-priority) set to one of the added
    PriorityClasses. Of course you do not need to create the Pods directly;
    normally you would add `priorityClassName` to the Pod template of a
    collection object like a Deployment.

Keep reading for more information about these steps.

{{< note >}}
Kubernetes already ships with two PriorityClasses:
`system-cluster-critical` and `system-node-critical`.
These are common classes and are used to [ensure that critical components are always scheduled first](/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/).
{{< /note >}}
 -->

## 怎么使用优先级和抢占 {#how-to-use-priority-and-preemption}

要使用优先级和抢占:

1. 添加一个或多个 [PriorityClasses](#priorityclass).

2. 在创建 Pod 时通过 [`priorityClassName`](#pod-priority) 为它设置一个上一步创建的
   PriorityClasses。 当然不需要直接创建 Pod； 通常是将 `priorityClassName` 添加到像
   Deployment 这一类对象的 Pod 模板中。

下面的章节会更多地介绍这些步骤。

{{< note >}}
k8s 已经自带了两个 PriorityClasses: `system-cluster-critical` 和 `system-node-critical`.
这些是通用类别，被用于
[确保关键组件始终先调度](/k8sDocs/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/).
{{< /note >}}

<!--
## PriorityClass

A PriorityClass is a non-namespaced object that defines a mapping from a
priority class name to the integer value of the priority. The name is specified
in the `name` field of the PriorityClass object's metadata. The value is
specified in the required `value` field. The higher the value, the higher the
priority.
The name of a PriorityClass object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names),
and it cannot be prefixed with `system-`.

A PriorityClass object can have any 32-bit integer value smaller than or equal
to 1 billion. Larger numbers are reserved for critical system Pods that should
not normally be preempted or evicted. A cluster admin should create one
PriorityClass object for each such mapping that they want.

PriorityClass also has two optional fields: `globalDefault` and `description`.
The `globalDefault` field indicates that the value of this PriorityClass should
be used for Pods without a `priorityClassName`. Only one PriorityClass with
`globalDefault` set to true can exist in the system. If there is no
PriorityClass with `globalDefault` set, the priority of Pods with no
`priorityClassName` is zero.

The `description` field is an arbitrary string. It is meant to tell users of the
cluster when they should use this PriorityClass.
 -->

## PriorityClass {#priority-class}

PriorityClass 是一个无命名空间字段的对象，它定义了一个优先级类别名称与一个整数值的优先级的映射
关系。 名称定义在 PriorityClass 对象元数据的 `name` 字段中。 值则定义在对象的必要字段 `value`
中。 这个值越大表示优先级越高。

PriorityClass 对象可以将任意小于 10 亿的 32 位整数作为其值。 更大的数字则被保留作为系统关键
Pod 使用，这些 Pod 通常不应该被挤掉或驱逐。

PriorityClass 还有两个可选字段: `globalDefault` 和 `description`. `globalDefault`
字段表示这个 PriorityClass 的值会用作那些没有 `priorityClassName` Pod 的优先级。 系统中
只能存在一个 `globalDefault` 设置为 true 的 PriorityClass。 如果集群中没有
`globalDefault` 设置为 true 的 PriorityClass。 则那些没有 `priorityClassName` 的 Pod
的优先级值为 0

`description` 可以是任意字符串。但通常用来告诉集群用户，啥时候应该用这个 PriorityClass.

<!--
### Notes about PodPriority and existing clusters

-   If you upgrade an existing cluster without this feature, the priority
    of your existing Pods is effectively zero.

-   Addition of a PriorityClass with `globalDefault` set to `true` does not
    change the priorities of existing Pods. The value of such a PriorityClass is
    used only for Pods created after the PriorityClass is added.

-   If you delete a PriorityClass, existing Pods that use the name of the
    deleted PriorityClass remain unchanged, but you cannot create more Pods that
    use the name of the deleted PriorityClass.
 -->

### Pod 优先级和已有集群需要注意的一些地方 {#notes-about-podpriority-and-existing-clusters}

- 如果从一个没有该特性的集群升级，集群中现在的 Pod 的优先级会被设置为 0

- 添加一个 `globalDefault` 设置为 `true` 的 PriorityClass， 不会改变已有 Pod 的优先级。
  这个 PriorityClass 的优先级值只会被应用到在它之后创建的 Pod 上。

- 如果删除了一个 PriorityClass， 已有的在使用这个 PriorityClass 的 Pod 不会发生变化，
  但新创建 Pod 时就不能再使用这个 PriorityClass 了。

<!--
### Example PriorityClass

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```
 -->

### `PriorityClass` 示例 {#example-priority-class}

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

<!--
## Non-preempting PriorityClass {#non-preempting-priority-class}

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

Pods with `PreemptionPolicy: Never` will be placed in the scheduling queue
ahead of lower-priority pods,
but they cannot preempt other pods.
A non-preempting pod waiting to be scheduled will stay in the scheduling queue,
until sufficient resources are free,
and it can be scheduled.
Non-preempting pods,
like other pods,
are subject to scheduler back-off.
This means that if the scheduler tries these pods and they cannot be scheduled,
they will be retried with lower frequency,
allowing other pods with lower priority to be scheduled before them.

Non-preempting pods may still be preempted by other,
high-priority pods.

`PreemptionPolicy` defaults to `PreemptLowerPriority`,
which will allow pods of that PriorityClass to preempt lower-priority pods
(as is existing default behavior).
If `PreemptionPolicy` is set to `Never`,
pods in that PriorityClass will be non-preempting.

An example use case is for data science workloads.
A user may submit a job that they want to be prioritized above other workloads,
but do not wish to discard existing work by preempting running pods.
The high priority job with `PreemptionPolicy: Never` will be scheduled
ahead of other queued pods,
as soon as sufficient cluster resources "naturally" become free.
 -->

## 非抢占式 PriorityClass {#non-preempting-priority-class}

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

设置有 `PreemptionPolicy: Never` 的 Pod 在调度队列中会放在低优先级 Pod 的前面， 但不会
挤掉其它的 Pod。 一个非抢占式 Pod 会一直待在调度队列中直至有足够的空闲资源才会被调度。 非抢占式
Pod 与其它的 Pod 一样服务调度器补尝。 意思是如果调度器在尝试调度这些 Pod 时如果他们不能被调度，
则尝试调度的频率会越来越低，这会使得其它低优先级的 Pod 可能在他们之间完成调度。

Non-preempting pods may still be preempted by other,
high-priority pods.
非抢占式 Pod 也可能被其它高优先级的 Pod 摔掉。

`PreemptionPolicy` 默认为 `PreemptLowerPriority`, 也就是允许这个 `PriorityClass` 的 Pod
可以抢占优先级比它低的 Pod (作为默认存在的行为)。 如果 `PreemptionPolicy` 设置为 `Never`,
使用这个 `PriorityClass` 就是非抢占式的。

一个使用场景示例就是数据科学的工作负载。 一个用户可能提供一个任务(job), 他想要这个任务的优先级
比其它所有工作负载的都高， 但不希望因抢占其它正在运行的 Pod 而导致废弃已经在进行的工作。 拥有
高优先级但使用 `PreemptionPolicy: Never` 的任务会被放在调度队列中其它 Pod 之前， 当有足够
集群资源"自然"空闲时就会被调度。

### 非抢占式 PriorityClass 示例 {#example-non-preempting-priority-class}

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: "This priority class will not cause other pods to be preempted."
```

<!--
## Pod priority

After you have one or more PriorityClasses, you can create Pods that specify one
of those PriorityClass names in their specifications. The priority admission
controller uses the `priorityClassName` field and populates the integer value of
the priority. If the priority class is not found, the Pod is rejected.

The following YAML is an example of a Pod configuration that uses the
PriorityClass created in the preceding example. The priority admission
controller checks the specification and resolves the priority of the Pod to
1000000.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```
 -->

## Pod 优先级 {#pod-priority}

在创建了一个或多个 PriorityClasses 后，在创建 Pod 时就可以在定义中指定一个 PriorityClass
名称。 优先级准入控制器使用 `priorityClassName` 字段转㧪为优先级的整数值。 如果没有找到这个
PriorityClasses 则拒绝创建这个 Pod。

下面的 YAML 中是一个 Pod 配置的示例，其中使用的前面示例中创建的 PriorityClass。优先级准入
控制器检查定义并解析设置 Pod 的优先级值为 1000000.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

### Pod 优先级对调度顺序的影响 {#effect-of-pod-priority-on-scheduling-order}

当启用优先级以后， 调度器会以优先级来组织等待调度的 Pod，高优先级的 Pod 在队列中要比低优先级的
排位靠前。 结果就是如果调度要求满足时高优先级的 Pod 可能会比低优先级的 Pod 先调度。 但如果
这个高优先级的 Pod 不能调度，调度器会继续尝试调度其它优先级比它低的 Pod。

<!--
## Preemption

When Pods are created, they go to a queue and wait to be scheduled. The
scheduler picks a Pod from the queue and tries to schedule it on a Node. If no
Node is found that satisfies all the specified requirements of the Pod,
preemption logic is triggered for the pending Pod. Let's call the pending Pod P.
Preemption logic tries to find a Node where removal of one or more Pods with
lower priority than P would enable P to be scheduled on that Node. If such a
Node is found, one or more lower priority Pods get evicted from the Node. After
the Pods are gone, P can be scheduled on the Node.
 -->

## 抢占 {#preemption}

当 Pod 被创建后，就会被塞进一个队列并等待调度。 调度器会从队列中挑一个出来并尝试将其调度到一个
节点上。 如果没有找到一个能满足这个 Pod 指定的所有要求的节点，为这个等待 Pod 的抢占逻辑就会
触发。 假设这个等待的 Pod 叫 P。 抢占逻辑尝试找一个可以通过移除一个或几个优先级比 P 低的 Pod
就能让 P 调度到该节点上。 如果有这样一个节点， 一个或几个低优先级的 Pod 就会被从该节点驱逐。
在这些 Pod 消失后， P 就可以被调度到这个节点上了。

<!--
### User exposed information

When Pod P preempts one or more Pods on Node N, `nominatedNodeName` field of Pod
P's status is set to the name of Node N. This field helps scheduler track
resources reserved for Pod P and also gives users information about preemptions
in their clusters.

Please note that Pod P is not necessarily scheduled to the "nominated Node".
After victim Pods are preempted, they get their graceful termination period. If
another node becomes available while scheduler is waiting for the victim Pods to
terminate, scheduler will use the other node to schedule Pod P. As a result
`nominatedNodeName` and `nodeName` of Pod spec are not always the same. Also, if
scheduler preempts Pods on Node N, but then a higher priority Pod than Pod P
arrives, scheduler may give Node N to the new higher priority Pod. In such a
case, scheduler clears `nominatedNodeName` of Pod P. By doing this, scheduler
makes Pod P eligible to preempt Pods on another Node.
 -->

### 暴露给用户的信息 {#user-exposed-information}

当 Pod P 抢占节点 N 上一个或多个 Pod 的地盘， Pod P 的状态上的 `nominatedNodeName` 字段
就会设置了 Node N 的名称。 这个字段帮助调度器跟踪为 Pod P 保留的资源同时也为用户提供集群中
的抢占信息。

要注意 Pod P 不是必须要被调度到"提名节点"的。 在受害 Pod 被抢后，他们会得到一个优雅停止的时限。
如果在等待这些 Pod 停止的过程中有另一个节点变得可用，调度器就会使用另一个节点来调度 Pod . 这
就会导致 Pod 定义中的 `nominatedNodeName` 和 `nodeName` 并不会始终相同。 还有，如果调度
器将节点 N 上的 Pod 驱逐后，这时候出来一个优先级比 Pod P 高的 Pod 冒出来, 调度器就可能会把
这个高优先级的 Pod 调度到节点 N 上。 在这种情况下，调度器会清除 Pod P 的 `nominatedNodeName`。
通过这种方式，调度让 Pod P 再次拥有抢占其它节点上 Pod 的权利。

<!--
### Limitations of preemption

#### Graceful termination of preemption victims

When Pods are preempted, the victims get their
[graceful termination period](/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).
They have that much time to finish their work and exit. If they don't, they are
killed. This graceful termination period creates a time gap between the point
that the scheduler preempts Pods and the time when the pending Pod (P) can be
scheduled on the Node (N). In the meantime, the scheduler keeps scheduling other
pending Pods. As victims exit or get terminated, the scheduler tries to schedule
Pods in the pending queue. Therefore, there is usually a time gap between the
point that scheduler preempts victims and the time that Pod P is scheduled. In
order to minimize this gap, one can set graceful termination period of lower
priority Pods to zero or a small number.
 -->

### 抢占式的限制 {#limitations-of-preemption}

#### 优雅的停止抢占的受害 Pod {#graceful-termination-of-preemption-victims}

当 Pod 被抢地盘之后，受害都会有
[优雅退场时间](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).
他们会有一定的时候来完成他们的工作并退出。 如果到了时间还没退出，就会被杀掉。 这个优雅退场时间
就造成了在调度器抢占 Pod 地盘与等待 Pod (P) 可以被调度到节点(N)上之间有一个时间窗口。 在这个
时间窗口中，调度器会继续调度等待队列中其它等待的 Pod。 当受害 Pod 退出或被终止时，调度器在
尝试调度等待队列中的 Pod。因而， 通常在调度器抢占受害 Pod 地盘与 Pod P 被调度之间也有一个时间
窗口。 为了减少这个窗口的长度， 可以将低优先级的 Pod 的优雅退场的时间设置为 0 或一个较小的数字。

<!--
#### PodDisruptionBudget is supported, but not guaranteed

A [PodDisruptionBudget](/docs/concepts/workloads/pods/disruptions/) (PDB)
allows application owners to limit the number of Pods of a replicated application
that are down simultaneously from voluntary disruptions. Kubernetes supports
PDB when preempting Pods, but respecting PDB is best effort. The scheduler tries
to find victims whose PDB are not violated by preemption, but if no such victims
are found, preemption will still happen, and lower priority Pods will be removed
despite their PDBs being violated.
 -->

#### 支持 PodDisruptionBudget，但不承诺会遵守 {#poddisruptionbudget-is-supported-but-not-guaranteed}

[PodDisruptionBudget](/k8sDocs/docs/concepts/workloads/pods/disruptions/) (PDB)
允许应用所属者限制在计划内故障时多副本应用同时挂掉的 Pod 的数量。 在 Pod 抢占时 k8s 也是支持
PDB 的，但只是尽最大努力遵守。  调度器尝试寻找那些在抢占时不会违反PDB 的 Pod， 但是如果找不到
这样的 Pod。 抢占还是会发生，即便违反了 PDB 低优先级的 Pod 还是会被驱逐。

<!--
#### Inter-Pod affinity on lower-priority Pods

A Node is considered for preemption only when the answer to this question is
yes: "If all the Pods with lower priority than the pending Pod are removed from
the Node, can the pending Pod be scheduled on the Node?"

{{< note >}}
Preemption does not necessarily remove all lower-priority
Pods. If the pending Pod can be scheduled by removing fewer than all
lower-priority Pods, then only a portion of the lower-priority Pods are removed.
Even so, the answer to the preceding question must be yes. If the answer is no,
the Node is not considered for preemption.
{{< /note >}}

If a pending Pod has inter-pod affinity to one or more of the lower-priority
Pods on the Node, the inter-Pod affinity rule cannot be satisfied in the absence
of those lower-priority Pods. In this case, the scheduler does not preempt any
Pods on the Node. Instead, it looks for another Node. The scheduler might find a
suitable Node or it might not. There is no guarantee that the pending Pod can be
scheduled.

Our recommended solution for this problem is to create inter-Pod affinity only
towards equal or higher priority Pods.
 -->

#### 低优先级 Pod 的 Pod 内亲和 {#inter-Pod-affinity-on-lower-priority-Pods}

一个只能只有在这样一个问题的答案是肯定时才会开始抢占: 如果把节点上所有优先级比等待 Pod 低的
Pod 都从节点移除，等待 Pod 是不是就能调度到这个节点上?

{{< note >}}

抢占并不是必须要把所有低优先级的 Pod 都干掉。 如果只在移除部分低优先级的 Pod 就可以让等待的 Pod
调度，则只有一部分低优先级的 Pod 会被干掉。 即便如此，对于上面提到的问题答案也必须是肯定的。
如果答案是否定的，则节点不会考虑抢占。
{{< /note >}}

如果等待的 Pod 在这个节点上有一个或多个低优先级的 Pod 存在 Pod 内亲和，少了这些低优先级的 Pod
就不能满足 Pod 内亲和规则。 在这种情况下， 调度器不会抢占节点上的任何 Pod。 而是会去打其它的节点。
调度器可能找到也可能找不到这样的节点。 这就不能承诺对等待 Pod 完成调度。

对于这种情况，我们推荐的解决方案会，在创建 Pod 内亲和时只指定相同或更高优先级的 Pod。

<!--
#### Cross node preemption

Suppose a Node N is being considered for preemption so that a pending Pod P can
be scheduled on N. P might become feasible on N only if a Pod on another Node is
preempted. Here's an example:

*   Pod P is being considered for Node N.
*   Pod Q is running on another Node in the same Zone as Node N.
*   Pod P has Zone-wide anti-affinity with Pod Q (`topologyKey:
    topology.kubernetes.io/zone`).
*   There are no other cases of anti-affinity between Pod P and other Pods in
    the Zone.
*   In order to schedule Pod P on Node N, Pod Q can be preempted, but scheduler
    does not perform cross-node preemption. So, Pod P will be deemed
    unschedulable on Node N.

If Pod Q were removed from its Node, the Pod anti-affinity violation would be
gone, and Pod P could possibly be scheduled on Node N.

We may consider adding cross Node preemption in future versions if there is
enough demand and if we find an algorithm with reasonable performance.
 -->

#### 跨节点抢占 {#cross-node-preemption}

假设一个节点 N 被考虑作为等待的 Pod N 在其上进行抢占并调度。 P 只有在另一个节点上的一个 Pod 也
被抢占才可能被调度到节点 N 上。 下面是一个例子:

-  考虑将 Pod P 放到节点 N 上。
-  Pod Q 运行在与节点 N 同一个区域的另一个节点上。
-  Pod P 与 Pod Q 之间有一个区域级的反亲和规则(`topologyKey: topology.kubernetes.io/zone`).
-  在这个区域中 Pod P 与 Pod Q 之间没有其它的反亲和情况。
-  要把 Pod P 调度到节点 N 上， Pod Q 就要被抢占， 但调度器不会执行跨节点的抢占。 所以 Pod P
   就认为是不可以调度到节点 N 上。

如果 Pod Q 被从这个节点上移除， 违反反亲和性这个情况就不存在了， Pod P 就是可以被调度到节点 N 上的。

如果有足够量的要求，我们考虑在未来的版本中加入跨节点的抢占，还得我们找到一个性能可以的算法。

<!--
## Troubleshooting

Pod priority and pre-emption can have unwanted side effects. Here are some
examples of potential problems and ways to deal with them.
 -->

## 故障排查 {#troubleshooting}

Pod 优先级和抢占行为可能会引起一些不希望的副作用。 下面是一些潜在问题的示例以及对应的解决办法。

<!--
### Pods are preempted unnecessarily

Preemption removes existing Pods from a cluster under resource pressure to make
room for higher priority pending Pods. If you give high priorities to
certain Pods by mistake, these unintentionally high priority Pods may cause
preemption in your cluster. Pod priority is specified by setting the
`priorityClassName` field in the Pod's specification. The integer value for
priority is then resolved and populated to the `priority` field of `podSpec`.

To address the problem, you can change the `priorityClassName` for those Pods
to use lower priority classes, or leave that field empty. An empty
`priorityClassName` is resolved to zero by default.

When a Pod is preempted, there will be events recorded for the preempted Pod.
Preemption should happen only when a cluster does not have enough resources for
a Pod. In such cases, preemption happens only when the priority of the pending
Pod (preemptor) is higher than the victim Pods. Preemption must not happen when
there is no pending Pod, or when the pending Pods have equal or lower priority
than the victims. If preemption happens in such scenarios, please file an issue.
 -->

### 不必要的抢占 {#pods-are-preempted-unnecessarily}

抢占就是集群在资源压力下移除现有的 Pod 为更高优先级的 Pod 腾空间。 如果误操作给一个 Pod 不该
有的高优先级，这些无意中搞出来的高优先级 Pod 就可能在集群中触发抢占。 Pod 的优先级是通过在 Pod
的定义中的 `priorityClassName` 字段来实现的。 整数值的优先级则是解析 PriorityClass 得到
并注入到 `podSpec` 的 `priority` 字段中。

为解决这个问题，可以通过修改这些 Pod 的 `priorityClassName` 使用低优先级的类别， 或不设置
那个字段。 空的 `priorityClassName` 默认解析出来的优先级值是 0.

当一个 Pod 被抢占后， 被抢占的 Pod 就会有相应的事件记录。 抢占只应该发生在集群中没有足够的资源
来调度等待的 Pod。 在这种情况下， 抢占只发生在等待 Pod (抢占者)的优先级比被抢占者的要高。抢占
不能发生在没有等待 Pod， 或与等待的 Pod 拥有与被抢占 Pod 相等或更低的优先级的的情况。如果这种
情况下都发生了抢占，请到官方仓库提交问题单。

<!--
### Pods are preempted, but the preemptor is not scheduled

When pods are preempted, they receive their requested graceful termination
period, which is by default 30 seconds. If the victim Pods do not terminate within
this period, they are forcibly terminated. Once all the victims go away, the
preemptor Pod can be scheduled.

While the preemptor Pod is waiting for the victims to go away, a higher priority
Pod may be created that fits on the same Node. In this case, the scheduler will
schedule the higher priority Pod instead of the preemptor.

This is expected behavior: the Pod with the higher priority should take the place
of a Pod with a lower priority.
 -->

### 受害者已经被抢，抢占者又没去占地盘 {#pods-are-preempted-but-the-preemptor-is-not-scheduled}

当 Pod 被抢时，他们会得到其所请求的优雅退场时间，这个时间默认为 30 秒。 如果被抢的 Pod 在这个
时间内没有终止，则它们会被强制终止。 当所有的被抢 Pod 都退出后，抢占者就可以被调度了。

当抢占者在等待被抢占者退场时， 可能会有比抢占都优先级更高的 Pod 被创建并适合被调度到同一个节点。
在这种情况下，调度会将后来的这个更高优先级的 Pod 调度过去，而不是原本的抢占者。

这是预期的行为: 高优先级的 Pod 应该占掉低优先级的 Pod 的地盘。

### 高优先级 Pod 比低优先级先被抢占 {#higher-priority-pods-are-preempted-before-lower-priority-pods}

调度器在尝试为等待 Pod 找可以运行的节点时。 如果没有找到，调度器就会尝试在任意一个节点上移除
低优先级的 Pod 为这个等待的 Pod 腾空间。 如果一个有低优先级 Pod 的节点不适合这个等待的 Pod，
调度可能会选择另一个(相对于刚才那个节点上的 Pod)优先级更高的 Pod 进行抢占。但被抢占者依然是
比抢占 Pod 的优先级要低。

当有多个节点满足被抢占的条件时， 调度器会选择被抢占 Pod 集合优先级最低的那个节点。 但是如果这些
Pod 有 PodDisruptionBudget 并且，如果它们被抢占会违反其 PDB， 则调度器可能会选择一个优先级
更高一些的 Pod 的节点。

当存在多个节点可以抢占而又没有以上提到的情况时， 调度器会选择 Pod 优先级最低的那个节点。

## Pod 优先级与服务质量之间的相互关系 {#interactions-of-pod-priority-and-qos}

Pod 优先级和
{{< glossary_tooltip term_id="qos-class" >}}
是两个交互和关系都比较少的两个特性， 在设置 Pod 优先级级时是否基于其 QoS 类别是没有默认限制的。
调度器在进行抢占逻辑时选择抢占目标时不会考虑 QoS。 抢占逻辑只会考虑 Pod 优先级并且尝试选择优先级
最低的目标集。 高优先级的 Pod 只有在如果移除最低优先级并不能满足调度器调度抢占 Pod 的时候，或
如果最低优先级的 Pod 受到 `PodDisruptionBudget` 保护。

同时考虑 QoS 和 Pod 优先级的唯一组件是
[kubelet 资源耗尽(out-of-resource) 驱逐](/k8sDocs/docs/tasks/administer-cluster/out-of-resource/).
kubelet 排列驱逐列表时，第一是他们使用的紧缺资源是否超出了请求， 然后是优先级， 然后是消耗的
紧缺计算资源相对了 Pod 调度请求的量。更多信息见
[驱逐终端用户 Pod](/k8sDocs/docs/tasks/administer-cluster/out-of-resource/#evicting-end-user-pods)

kubelet 资源耗尽(out-of-resource) 驱逐不会驱逐那些使用没有超出其请求值的 Pod。 如果一个
低优先级的 Pod 没有超出它的请求， 它就不会被驱逐。 另一个优先级相对更高的 Pod 如果超出了其请求则
可能被驱逐。

## {{% heading "whatsnext" %}}

<!--
* Read about using ResourceQuotas in connection with PriorityClasses: [limit Priority Class consumption by default](/docs/concepts/policy/resource-quotas/#limit-priority-class-consumption-by-default)
 -->

- 使用与 PriorityClasses 有联系的 PriorityClasses
  [对默认使用的 Priority Class进行限制](/k8sDocs/docs/concepts/policy/resource-quotas/#limit-priority-class-consumption-by-default)
