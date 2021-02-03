---
title: 毒点(Taint)与耐受性(Toleration)
content_type: concept
weight: 40
---
<!--
---
reviewers:
- davidopp
- kevin-wangzefeng
- bsalamat
title: Taints and Tolerations
content_type: concept
weight: 40
---
 -->

<!-- overview -->
<!--
[_Node affinity_](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity),
is a property of {{< glossary_tooltip text="Pods" term_id="pod" >}} that *attracts* them to
a set of {{< glossary_tooltip text="nodes" term_id="node" >}} (either as a preference or a
hard requirement). _Taints_ are the opposite -- they allow a node to repel a set of pods.

_Tolerations_ are applied to pods, and allow (but do not require) the pods to schedule
onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled
onto inappropriate nodes. One or more taints are applied to a node; this
marks that the node should not accept any pods that do not  the taints.
-->

[_节点亲和性_](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity),
是
{{< glossary_tooltip term_id="pod" >}}
的一个属性，这个属性会 *吸引* 它们到一个
{{< glossary_tooltip term_id="node" >}}
集合(无论是偏好还是硬性要求)。
_毒点(Taint)_ 则正好相反 -- 它们允许一个节点排斥特定集合内的 Pod。

_耐受性(Toleration)_ 是会应用到 Pod 上的， 它允许(但不是必须)调度到拥有对应 毒点(Taint)的节点上。

毒点(Taint)和耐受性(Toleration) 一起工作，以保证 Pod 不会调度到不合适的节点上。一个节点可以
应用一个或多个毒点(Taint)；这个标记表示节点不应该接收任意不能耐受(tolerate) 这些毒点(Taint)
的 Pod。
<!-- body -->

<!--
## Concepts

You add a taint to a node using [kubectl taint](/docs/reference/generated/kubectl/kubectl-commands#taint).
For example,

```shell
kubectl taint nodes node1 key1=value1:NoSchedule
```

places a taint on node `node1`. The taint has key `key1`, value `value1`, and taint effect `NoSchedule`.
This means that no pod will be able to schedule onto `node1` unless it has a matching toleration.

To remove the taint added by the command above, you can run:
```shell
kubectl taint nodes node1 key1=value1:NoSchedule-
```

You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the
taint created by the `kubectl taint` line above, and thus a pod with either toleration would be able
to schedule onto `node1`:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

```yaml
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

Here's an example of a pod that uses tolerations:

{{< codenew file="pods/pod-with-toleration.yaml" >}}

The default value for `operator` is `Equal`.

A toleration "matches" a taint if the keys are the same and the effects are the same, and:

* the `operator` is `Exists` (in which case no `value` should be specified), or
* the `operator` is `Equal` and the `value`s are equal.

{{< note >}}

There are two special cases:

An empty `key` with operator `Exists` matches all keys, values and effects which means this
will tolerate everything.

An empty `effect` matches all effects with key `key1`.

{{< /note >}}

The above example used `effect` of `NoSchedule`. Alternatively, you can use `effect` of `PreferNoSchedule`.
This is a "preference" or "soft" version of `NoSchedule` -- the system will *try* to avoid placing a
pod that does not tolerate the taint on the node, but it is not required. The third kind of `effect` is
`NoExecute`, described later.

You can put multiple taints on the same node and multiple tolerations on the same pod.
The way Kubernetes processes multiple taints and tolerations is like a filter: start
with all of a node's taints, then ignore the ones for which the pod has a matching toleration; the
remaining un-ignored taints have the indicated effects on the pod. In particular,

* if there is at least one un-ignored taint with effect `NoSchedule` then Kubernetes will not schedule
the pod onto that node
* if there is no un-ignored taint with effect `NoSchedule` but there is at least one un-ignored taint with
effect `PreferNoSchedule` then Kubernetes will *try* to not schedule the pod onto the node
* if there is at least one un-ignored taint with effect `NoExecute` then the pod will be evicted from
the node (if it is already running on the node), and will not be
scheduled onto the node (if it is not yet running on the node).

For example, imagine you taint a node like this

```shell
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

And a pod has two tolerations:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```

In this case, the pod will not be able to schedule onto the node, because there is no
toleration matching the third taint. But it will be able to continue running if it is
already running on the node when the taint is added, because the third taint is the only
one of the three that is not tolerated by the pod.

Normally, if a taint with effect `NoExecute` is added to a node, then any pods that do
not tolerate the taint will be evicted immediately, and pods that do tolerate the
taint will never be evicted. However, a toleration with `NoExecute` effect can specify
an optional `tolerationSeconds` field that dictates how long the pod will stay bound
to the node after the taint is added. For example,

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

means that if this pod is running and a matching taint is added to the node, then
the pod will stay bound to the node for 3600 seconds, and then be evicted. If the
taint is removed before that time, the pod will not be evicted. -->


## 概念 {#concepts}

用户可以使用
[kubectl taint](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint)
来给一个节点添加一个毒点(Taint),
例如，

```shell
kubectl taint nodes node1 key1=value1:NoSchedule
```

在节点 `node1` 上放置一个毒点(Taint)。 这个 毒点(Taint)的键是 `key1`, 值是 `value1`，
效果是 `NoSchedule`。 它的含义是如果 Pod 是没有对应的耐受性(Toleration) 就不能调度到节点
`node1` 上。

要移除上面的命令添加的 毒点(Taint)，可以使用:

```shell
kubectl taint nodes node1 key1=value1:NoSchedule-
```

用户可以在 Pod 的 `.spec` 中指定耐受性(Toleration)。 下面的两个耐受性(Toleration) 就可以
匹配由上面`kubectl taint` 命令定义的耐受性(Toleration)，也就是一个 Pod 如果包含以下耐受性(Toleration)
中的任意一个就可以调度到节点  `node1` 上:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

```yaml
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```

现在是一个使用耐受性(Toleration)的示例:

{{< codenew file="pods/pod-with-toleration.yaml" >}}

`operator` 的默认值是 `Equal`.

一个耐受性(Toleration)匹配一个毒点(Taint)条件是 键要相同并且效果要一样，同时:
- `operator` is `Exists` (这种情况不应该指定值), 或
- `operator` is `Equal` 并且 `value` 是相等的。

{{< note >}}

还有两种特殊情况:

`operator` is `Exists`, `key` 是空表示匹配所有的键，值和效果，也就代表耐受所有毒点(Taint)

如果 `effect` 是空则匹配键 `key1` 的所有效果

{{< /note >}}

上面示例果使用的 `effect` 是 `NoSchedule`. 还有另一个选择是让 `effect` 使用 `PreferNoSchedule`.
这是 `NoSchedule` 的 "偏好" 或 "软" 版本 -- 也就是系统会 *尝试* 避免将那些不能耐受这个
毒点(Taint) 的 Pod 放到这个节点上，但不是一定不能放。 `effect` 第三个选择是 `NoExecute` 后面
会介绍。

用户可以在一个节点上添加多个毒点(Taint)，也可以在一个 Pod 上定义多个 耐受性(Toleration).
k8s 处理多个 毒点(Taint)和 耐受性(Toleration)的方式与过虑器类似: 从节点的所有 毒点(Taint)
开始， 然后忽略掉那些与 Pod 耐受性(Toleration)匹配的；剩下的就是对 Pod 生效的毒点(Taint)。
特别是，

- 如果剩下未忽略的毒点(Taint)中至少有一个效果是 `NoSchedule` 则 k8s 不会将这个 Pod 调度到这个节点上
- 如果剩下未忽略的毒点(Taint)中没有效果是 `NoSchedule`， 但是至少有一个效果是 `PreferNoSchedule`
  则 k8s 会 *尝试* 不将这个 Pod 调度到这个节点上
- 如果剩下未忽略的毒点(Taint)中至少有一个效果是 `NoExecute`，则 Pod 会被从节点驱逐(如果 Pod
  已经在节点上运行)，或不会被调度到这个节点(如果它还没有在这个节点上运行)

例如，假设将一个节点添加如下毒点(Taint)

```shell
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

和一个有以下两个耐受性(Toleration) Pod:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```

在这种情况下， Pod 是会能调度到这个节点上的， 因为没有一个 耐受性(Toleration)能匹配到第三个
毒点(Taint)。 但如果这个 Pod 在这些毒点(Taint)添加前已经在这个节点上运行，则它还是可以继续在
上面运行，因为 Pod 不耐受的毒点(Taint)只有第三个。

通常情况下，如果一个效果是 `NoExecute` 的毒点(Taint)被添加到节点上，则所有不能耐受这个毒点(Taint)
的 Pod 立马就会被驱逐，同时耐受这个毒点(Taint) Pod 则永远不会被驱逐。 但是，效果是  `NoExecute`
的 耐受性(Toleration) 还可以指定一个可选的 `tolerationSeconds` 字段，表示在节点添加 毒点(Taint)
后 Pod 还可以在节点上保持绑定状态多久。例如，

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

的含义就是如果这个 Pod 运行在一个添加了这个耐受性(Toleration)匹配 毒点(Taint)的节点， 则
Pod 会继续与节点保持绑定 3600 秒，然后被驱逐。 如果在这个时间之前，这个 毒点(Taint) 被从该
节点移除，则 Pod 不会被驱逐。

<!--
## Example Use Cases

Taints and tolerations are a flexible way to steer pods *away* from nodes or evict
pods that shouldn't be running. A few of the use cases are

* **Dedicated Nodes**: If you want to dedicate a set of nodes for exclusive use by
a particular set of users, you can add a taint to those nodes (say,
`kubectl taint nodes nodename dedicated=groupName:NoSchedule`) and then add a corresponding
toleration to their pods (this would be done most easily by writing a custom
[admission controller](/docs/reference/access-authn-authz/admission-controllers/)).
The pods with the tolerations will then be allowed to use the tainted (dedicated) nodes as
well as any other nodes in the cluster. If you want to dedicate the nodes to them *and*
ensure they *only* use the dedicated nodes, then you should additionally add a label similar
to the taint to the same set of nodes (e.g. `dedicated=groupName`), and the admission
controller should additionally add a node affinity to require that the pods can only schedule
onto nodes labeled with `dedicated=groupName`.

* **Nodes with Special Hardware**: In a cluster where a small subset of nodes have specialized
hardware (for example GPUs), it is desirable to keep pods that don't need the specialized
hardware off of those nodes, thus leaving room for later-arriving pods that do need the
specialized hardware. This can be done by tainting the nodes that have the specialized
hardware (e.g. `kubectl taint nodes nodename special=true:NoSchedule` or
`kubectl taint nodes nodename special=true:PreferNoSchedule`) and adding a corresponding
toleration to pods that use the special hardware. As in the dedicated nodes use case,
it is probably easiest to apply the tolerations using a custom
[admission controller](/docs/reference/access-authn-authz/admission-controllers/).
For example, it is recommended to use [Extended
Resources](/docs/concepts/configuration/manage-resources-containers/#extended-resources)
to represent the special hardware, taint your special hardware nodes with the
extended resource name and run the
[ExtendedResourceToleration](/docs/reference/access-authn-authz/admission-controllers/#extendedresourcetoleration)
admission controller. Now, because the nodes are tainted, no pods without the
toleration will schedule on them. But when you submit a pod that requests the
extended resource, the `ExtendedResourceToleration` admission controller will
automatically add the correct toleration to the pod and that pod will schedule
on the special hardware nodes. This will make sure that these special hardware
nodes are dedicated for pods requesting such hardware and you don't have to
manually add tolerations to your pods.

* **Taint based Evictions**: A per-pod-configurable eviction behavior
when there are node problems, which is described in the next section.
 -->

## 使用场景示例 {#example-use-cases}

毒点(Taint) 和 耐受性(Toleration) 是控制 Pod *远离* 某些节点或驱逐那些不应该运行的 Pod。
一些使用场景:

- **专用节点**: 如果想要将一组专用节点让一组特定用户独占，可以在这个节点上添加一个毒点(Taint)
(就比如, `kubectl taint nodes nodename dedicated=groupName:NoSchedule`)然后在他们
的 Pod 上添加对应的耐受性(Toleration)(实现这个最简单的办法是写一个自定义
[准入控制](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
)。
这些有这个耐受性(Toleration)的 Pod 就被允许使用这些毒掉的(专用)节点和集群中的其它节点。
如果想将这些节点给他们独占 *并且* 确保他们也 *只能* 使用这些专用节点, 还应该在这一组节点上添加
与毒点(Taint)类似的标签(例如 `dedicated=groupName`), 然后在准入控制器上另外添加一个节点亲和性
来标这些 Pod 只能调度到那些有 `dedicated=groupName` 标签的节点。

- **拥有特殊硬件的节点**: 在一个集群中，有一小部分节点有专用的硬件(例如 GPU)，这时就希望不要
将那些不需要这些专用硬件的 Pod 调度到这些节点上， 这样就可以把这些节点留给后来的为那些需要这些专用
硬件的 Pod。 要做到这一点，可以在这些有专用硬件的节点上加毒点(Taint)(例如，
  `kubectl taint nodes nodename special=true:NoSchedule` 或
  `kubectl taint nodes nodename special=true:PreferNoSchedule`
)再在需要这些专用硬件的 Pod 上添加对应的耐受性(Toleration)。 与上面的专用节点使用场景一样，
应用这些 耐受性(Toleration)最简单的方式就是使用自定义
[准入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).
例如推荐使用
[扩展资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/#extended-resources)
来表示特殊的硬件， 使用扩展资源的名称作为节点的 毒点(Taint) 然后运行
[ExtendedResourceToleration](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#extendedresourcetoleration)
准入控制器。 此时， 因为节点上已经打了 毒点(Taint)没有对应耐受性(Toleration)的 Pod 不会调度
到这些节点上。 但当提供一个要求这些扩展资源的 Pod 时， `ExtendedResourceToleration` 准入
控制器会自动为这些 Pod 添加对应的耐受性(Toleration)，这样这些 Pod 就会被调度到这些有特殊硬件
的节点上。 这会保证这些特殊硬件的节点只会被这些要求这个硬件的 Pod 单独使用，并且不需要手动为这些
Pod 添加耐受性(Toleration)

* **Taint based Evictions**: A per-pod-configurable eviction behavior
when there are node problems, which is described in the next section.

- **基于毒点(Taint)的驱逐行为**: 一个每个 Pod 都可配置的驱逐行为就是当节点出问题时进行的，
 这会在下一节点介绍。

<!--
## Taint based Evictions

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

The `NoExecute` taint effect, mentioned above, affects pods that are already
running on the node as follows

 * pods that do not tolerate the taint are evicted immediately
 * pods that tolerate the taint without specifying `tolerationSeconds` in
   their toleration specification remain bound forever
 * pods that tolerate the taint with a specified `tolerationSeconds` remain
   bound for the specified amount of time

The node controller automatically taints a Node when certain conditions
are true. The following taints are built in:

 * `node.kubernetes.io/not-ready`: Node is not ready. This corresponds to
   the NodeCondition `Ready` being "`False`".
 * `node.kubernetes.io/unreachable`: Node is unreachable from the node
   controller. This corresponds to the NodeCondition `Ready` being "`Unknown`".
 * `node.kubernetes.io/out-of-disk`: Node becomes out of disk.
 * `node.kubernetes.io/memory-pressure`: Node has memory pressure.
 * `node.kubernetes.io/disk-pressure`: Node has disk pressure.
 * `node.kubernetes.io/network-unavailable`: Node's network is unavailable.
 * `node.kubernetes.io/unschedulable`: Node is unschedulable.
 * `node.cloudprovider.kubernetes.io/uninitialized`: When the kubelet is started
    with "external" cloud provider, this taint is set on a node to mark it
    as unusable. After a controller from the cloud-controller-manager initializes
    this node, the kubelet removes this taint.

In case a node is to be evicted, the node controller or the kubelet adds relevant taints
with `NoExecute` effect. If the fault condition returns to normal the kubelet or node
controller can remove the relevant taint(s).

{{< note >}}
The control plane limits the rate of adding node new taints to nodes. This rate limiting
manages the number of evictions that are triggered when many nodes become unreachable at
once (for example: if there is a network disruption).
{{< /note >}}

You can specify `tolerationSeconds` for a Pod to define how long that Pod stays bound
to a failing or unresponsive Node.

For example, you might want to keep an application with a lot of local state
bound to node for a long time in the event of network partition, hoping
that the partition will recover and thus the pod eviction can be avoided.
The toleration you set for that Pod might look like:

```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

{{< note >}}
Kubernetes automatically adds a toleration for
`node.kubernetes.io/not-ready` and `node.kubernetes.io/unreachable`
with `tolerationSeconds=300`,
unless you, or a controller, set those tolerations explicitly.

These automatically-added tolerations mean that Pods remain bound to
Nodes for 5 minutes after one of these problems is detected.
{{< /note >}}

[DaemonSet](/docs/concepts/workloads/controllers/daemonset/) pods are created with
`NoExecute` tolerations for the following taints with no `tolerationSeconds`:

  * `node.kubernetes.io/unreachable`
  * `node.kubernetes.io/not-ready`

This ensures that DaemonSet pods are never evicted due to these problems.
 -->

## 基于毒点(Taint) 的驱逐 {#taint-based-evictions}

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

上面提到的影响为 `NoExecute` 毒点(Taint)，对已经在节点上运行的 Pod 的影响如下

- 不能耐受这个 毒点(Taint) 的 Pod 会立马被踢出
- 能够耐受这个毒点(Taint)并且在耐受定义中没有指定 `tolerationSeconds` 的 Pod 会一直继续在这个节点上运行
- 能够耐受这个毒点(Taint)并且在耐受定义中指定了 `tolerationSeconds` 则会继续在这个节点上运行指定的这个时间

节点控制器会在特定条件满足是自动给节点添加 毒点(Taint). 以下为内置毒点(Taint):

- `node.kubernetes.io/not-ready`: 节点还没有就绪。 这对应的节点状态(NodeCondition) `Ready` 是 "`False`"
- `node.kubernetes.io/unreachable`: 节点控制器连接不到节点。 这对应的节点状态(NodeCondition) `Ready` 是 "`Unknown`"
- `node.kubernetes.io/out-of-disk`: 节点硬盘不足
- `node.kubernetes.io/memory-pressure`: 节点有内存使用紧张
- `node.kubernetes.io/disk-pressure`: 节点有硬盘使用紧张
- `node.kubernetes.io/network-unavailable`: 节点网络不可用
- `node.kubernetes.io/unschedulable`: 节点不能作为调度目标
- `node.cloudprovider.kubernetes.io/uninitialized`: 当 kubelet 使用"外部"云提供商启动时，
  使用这个 毒点(Taint) 将其标记为不可用。 在 `cloud-controller-manager` 中的控制器初始化这
  个节点之后， kubelet 会移除这个毒点(Taint)

有一种情况是一个节点被驱逐了，节点控制器或 kubelet 会添加相应效果为 `NoExecute` 的毒点(Taint)。
如果这个节点从错误状态重新变回正常状态节点控制器或 kubelet 会移除相应的毒点(Taint)。

{{< note >}}
控制中心会限制添加到节点上的 毒点(Taint) 的速率。这个速率限制是在许多节点一下都变得不可达时管理
被驱逐的数量(例如: 发生了网络抖动)
{{< /note >}}

可以为 Pod 指定 `tolerationSeconds` 来定义在节点失效或不响应时继续让这个 Pod 保持在这个节点
上的时间。

例如，当一个有许多本地状态应用与在出现网络分区的情况下继续保持在这个节点上相当长一段时间，以期望
这个分区在些期间能够恢复，这样可以避免 Pod 被驱逐。 设置在这个 Pod 上的耐受性(Toleration)可能
会长成这个样子:

```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

{{< note >}}
除非有用户或控制器显示地设置这些 耐受性(Toleration)，否则 k8s 会自动地在添加
`node.kubernetes.io/not-ready` 和 `node.kubernetes.io/unreachable` 在其中设置
`tolerationSeconds=300`

这些自动添加耐受性(Toleration)表示，在侦测到这个问题时 Pod 最多还会在这个节点上保留 5 分钟。
{{< /note >}}

[DaemonSet](/k8sDocs/docs/concepts/workloads/controllers/daemonset/) 的 Pod 在创建时会指定
下面这些毒点(Taint)对应的效果为 `NoExecute` 并且没有 `tolerationSeconds` 的耐受性(Toleration):

- `node.kubernetes.io/unreachable`
- `node.kubernetes.io/not-ready`

这会确保 DaemonSet 的 Pod 永远不会因为这些原因而被驱逐。
<!--
## Taint Nodes by Condition

The node lifecycle controller automatically creates taints corresponding to
Node conditions with `NoSchedule` effect.
Similarly the scheduler does not check Node conditions; instead the scheduler checks taints. This assures that Node conditions don't affect what's scheduled onto the Node. The user can choose to ignore some of the Node's problems (represented as Node conditions) by adding appropriate Pod tolerations.

The DaemonSet controller automatically adds the following `NoSchedule`
tolerations to all daemons, to prevent DaemonSets from breaking.

  * `node.kubernetes.io/memory-pressure`
  * `node.kubernetes.io/disk-pressure`
  * `node.kubernetes.io/out-of-disk` (*only for critical pods*)
  * `node.kubernetes.io/unschedulable` (1.10 or later)
  * `node.kubernetes.io/network-unavailable` (*host network only*)

Adding these tolerations ensures backward compatibility. You can also add
arbitrary tolerations to DaemonSets.
 -->

## 根据条件给节点上毒点(Taint) {#taint-nodes-by-condition}

节点的生命周期控制自动地根据节点状况创建 `NoSchedule` 效果的毒点(Taint)。
类似地，调度是不会检查节点状况的；替代方式是调度器检查毒点(Taint)。 这样做能保证节点状况不会影响
已经调度到这个节点上的工作负载。 用户也可以通过向 Pod 上添加对应的耐受性(Toleration)
选择忽略一个节点上的问题(以节点状况表示的)。

DaemonSet 控制器自动添加以下 `NoSchedule` 耐受性(Toleration)到所有的守护进程， 防止 DaemonSet 挂掉。

- `node.kubernetes.io/memory-pressure`
- `node.kubernetes.io/disk-pressure`
- `node.kubernetes.io/out-of-disk` (*仅对关键 Pod*)
- `node.kubernetes.io/unschedulable` (v1.10+)
- `node.kubernetes.io/network-unavailable` (*仅主机网络*)

添加这些 耐受性(Toleration) 能确保向后兼容。 用户还可以向 DaemonSet 添加任意 耐受性(Toleration)

## {{% heading "whatsnext" %}}
<!--
* Read about [out of resource handling](/docs/tasks/administer-cluster/out-of-resource/) and how you can configure it
* Read about [pod priority](/docs/concepts/configuration/pod-priority-preemption/)
 -->
* 实践 [资源不足的处理](/k8sDocs/docs/tasks/administer-cluster/out-of-resource/) 与配置
* 概念 [Pod 优先级](/k8sDocs/docs/concepts/configuration/pod-priority-preemption/)
