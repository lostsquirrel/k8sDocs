---
title: Pod 拓扑分布约束条件
date: 2020-07-29
publishdate: 2020-08-05
weight: 2030002
---
<!--
---
title: Pod Topology Spread Constraints
content_type: concept
weight: 40
---
 -->
<!-- overview -->
<!--  
{{< feature-state for_k8s_version="v1.18" state="beta" >}}

You can use _topology spread constraints_ to control how {{< glossary_tooltip text="Pods" term_id="Pod" >}} are spread across your cluster among failure-domains such as regions, zones, nodes, and other user-defined topology domains. This can help to achieve high availability as well as efficient resource utilization.
-->
{{< feature-state for_k8s_version="v1.18" state="beta" >}}
用户可以通过 _拓扑分布约束条件_ 来控制 {{< glossary_tooltip text="Pods" term_id="Pod" >}}
在包含集群中故障域 （如 地区，分区，节点和其它用户定义拓扑域）中是怎样分布的。
该功能可以帮助用户在实现高可用的同时充分利用资源。

<!-- body -->
<!--
## Prerequisites

### Enable Feature Gate

The `EvenPodsSpread` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
must be enabled for the
{{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}} **and**
{{< glossary_tooltip text="scheduler" term_id="kube-scheduler" >}}.

### Node Labels

Topology spread constraints rely on node labels to identify the topology domain(s) that each Node is in. For example, a Node might have labels: `node=node1,zone=us-east-1a,region=us-east-1`

Suppose you have a 4-node cluster with the following labels:

```
NAME    STATUS   ROLES    AGE     VERSION   LABELS
node1   Ready    <none>   4m26s   v1.16.0   node=node1,zone=zoneA
node2   Ready    <none>   3m58s   v1.16.0   node=node2,zone=zoneA
node3   Ready    <none>   3m17s   v1.16.0   node=node3,zone=zoneB
node4   Ready    <none>   2m43s   v1.16.0   node=node4,zone=zoneB
```

Then the cluster is logically viewed as below:

```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
```

Instead of manually applying labels, you can also reuse the [well-known labels](/docs/reference/kubernetes-api/labels-annotations-taints/) that are created and populated automatically on most clusters.
 -->
## 准备工作

### 打开功能开关

需要打开
{{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}} **和**
{{< glossary_tooltip text="scheduler" term_id="kube-scheduler" >}}
中叫 `EvenPodsSpread` 的[功能阀](/k8sDocs/reference/command-line-tools-reference/feature-gates/)

### 为节点添加恰当的标签

拓扑分布约束条件信赖于节点标签来区分其所在的拓扑域。 例如， 某节点标签可以为:
`node=node1,zone=us-east-1a,region=us-east-1`
假设集群中有4个节点，标签如下:
```
NAME    STATUS   ROLES    AGE     VERSION   LABELS
node1   Ready    <none>   4m26s   v1.16.0   node=node1,zone=zoneA
node2   Ready    <none>   3m58s   v1.16.0   node=node2,zone=zoneA
node3   Ready    <none>   3m17s   v1.16.0   node=node3,zone=zoneB
node4   Ready    <none>   2m43s   v1.16.0   node=node4,zone=zoneB
```
那么该集群的逻辑视图如下:

```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
```
相较于手动添加标签，可以重用在大多数集群会自动创建和添加的
[常用标签](/k8sDocs/reference/kubernetes-api/labels-annotations-taints/)
<!--
## Spread Constraints for Pods

### API

The field `pod.spec.topologySpreadConstraints` is introduced in 1.16 as below:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  topologySpreadConstraints:
    - maxSkew: <integer>
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
```

You can define one or multiple `topologySpreadConstraint` to instruct the kube-scheduler how to place each incoming Pod in relation to the existing Pods across your cluster. The fields are:

- **maxSkew** describes the degree to which Pods may be unevenly distributed. It's the maximum permitted difference between the number of matching Pods in any two topology domains of a given topology type. It must be greater than zero.
- **topologyKey** is the key of node labels. If two Nodes are labelled with this key and have identical values for that label, the scheduler treats both Nodes as being in the same topology. The scheduler tries to place a balanced number of Pods into each topology domain.
- **whenUnsatisfiable** indicates how to deal with a Pod if it doesn't satisfy the spread constraint:
  - `DoNotSchedule` (default) tells the scheduler not to schedule it.
  - `ScheduleAnyway` tells the scheduler to still schedule it while prioritizing nodes that minimize the skew.
- **labelSelector** is used to find matching Pods. Pods that match this label selector are counted to determine the number of Pods in their corresponding topology domain. See [Label Selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors) for more details.

You can read more about this field by running `kubectl explain Pod.spec.topologySpreadConstraints`.
 -->
## Pod 的扩散约束

### API

在 `1.16` 版本中加入了 `pod.spec.topologySpreadConstraints` 字段，如下:

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  topologySpreadConstraints:
    - maxSkew: <integer>
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
```

用户可以在 Pod 上定义一个或多个 `topologySpreadConstraint`， 用于指导 `kube-scheduler`
在集群中有与已经存在的 Pod 相关的新的 Pod 时应该怎么放置。有如下字段：

- **maxSkew** 该字段描述 Pod 分布不均匀的程度。 在指定拓扑类型的两个拓扑域中特定 Pod 数量相差数允许的最大值，这个值必须大于 0
- **topologyKey** 该字段使用节点的标签键， 如果有两个节点包含一个键，且该键值也相同，
  调度器会将这两个节点认为在同一个拓扑。 调度器会尝试让两个拓扑域中的 Pod 数量平衡。
- **whenUnsatisfiable** 该字段设置怎么处理不满足分布约束的Pod
  - `DoNotSchedule` (默认) 让调度器不要调度该 Pod
  - `ScheduleAnyway` 让调度器仍然调度，但调度到不均匀度(skew)最低的节点
- **labelSelector** 用于找到匹配的 Pod。 匹配到的 Pod 会作为对应拓扑域的的一员(参与数量统计)
  更多标签和选择器见 [标签和选择器](/k8sDocs/concepts/overview/working-with-objects/labels/#label-selectors)

更新多关于该字段的信息请查看 `kubectl explain Pod.spec.topologySpreadConstraints` 命令结果。
<!--
### Example: One TopologySpreadConstraint

Suppose you have a 4-node cluster where 3 Pods labeled `foo:bar` are located in node1, node2 and node3 respectively (`P` represents Pod):

```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+
```

If we want an incoming Pod to be evenly spread with existing Pods across zones, the spec can be given as:

{{< codenew file="pods/topology-spread-constraints/one-constraint.yaml" >}}

`topologyKey: zone` implies the even distribution will only be applied to the nodes which have label pair "zone:&lt;any value&gt;" present. `whenUnsatisfiable: DoNotSchedule` tells the scheduler to let it stay pending if the incoming Pod can’t satisfy the constraint.

If the scheduler placed this incoming Pod into "zoneA", the Pods distribution would become [3, 1], hence the actual skew is 2 (3 - 1) - which violates `maxSkew: 1`. In this example, the incoming Pod can only be placed onto "zoneB":

```
+---------------+---------------+      +---------------+---------------+
|     zoneA     |     zoneB     |      |     zoneA     |     zoneB     |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |  OR  | node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
|   P   |   P   |   P   |   P   |      |   P   |   P   |  P P  |       |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
```

You can tweak the Pod spec to meet various kinds of requirements:

- Change `maxSkew` to a bigger value like "2" so that the incoming Pod can be placed onto "zoneA" as well.
- Change `topologyKey` to "node" so as to distribute the Pods evenly across nodes instead of zones. In the above example, if `maxSkew` remains "1", the incoming Pod can only be placed onto "node4".
- Change `whenUnsatisfiable: DoNotSchedule` to `whenUnsatisfiable: ScheduleAnyway` to ensure the incoming Pod to be always schedulable (suppose other scheduling APIs are satisfied). However, it’s preferred to be placed onto the topology domain which has fewer matching Pods. (Be aware that this preferability is jointly normalized with other internal scheduling priorities like resource usage ratio, etc.)
 -->
### 示例: 单个 TopologySpreadConstraint

假设有一个4节点的集群中有三个标签包含标签为 `foo:bar`的 Pod， 分别分布在 1，2，3 号节点上(一个 `P` 代表一个 Pod)
```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+
```
如果想要新加入的 Pod 与之前的三个节点均匀的分布在不同的区域内， 可以使用如下配置:

{{< codenew file="pods/topology-spread-constraints/one-constraint.yaml" >}}

配置中 `topologyKey: zone` 表示均匀分布只针对包含标签 `zone:<any value>`的节点
`whenUnsatisfiable: DoNotSchedul` 表示针对不满足约束的的 Pod， 调度器应该让其挂起

如果调度器将 Pod 分配的 “zoneA”中， 则 Pod 分布就变成 [3,1], 这时偏差(skew)就为 2(3 - 1)
这就与 `maxSkew: 1` 相违背。所以在本例中，新加入的 Pod 就只能被分配到  “zoneB”:

```
+---------------+---------------+      +---------------+---------------+
|     zoneA     |     zoneB     |      |     zoneA     |     zoneB     |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |  OR  | node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
|   P   |   P   |   P   |   P   |      |   P   |   P   |  P P  |       |
+-------+-------+-------+-------+      +-------+-------+-------+-------+
```

用户也可以通过调整配置实现不同的需求

- 将 `maxSkew` 设置为大于 `2`， 这样新加入 Pod 也可以分配到 “zoneA”中
- 将 `topologyKey` 设置为 "node", 则 Pod 的均匀分布范围就从区域变为节点
- 将 `whenUnsatisfiable` 设置为 `ScheduleAnyway` 来保证新加入的 Pod 都能被调度(假设满足其它的调度 API)
  但是，会被优先调度到匹配 Pod 少的拓扑域中(也要注意这个优先还需要连同其它内部调度优先级如资源使用率等一起考量)。

<!--
### Example: Multiple TopologySpreadConstraints

This builds upon the previous example. Suppose you have a 4-node cluster where 3 Pods labeled `foo:bar` are located in node1, node2 and node3 respectively (`P` represents Pod):

```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+
```

You can use 2 TopologySpreadConstraints to control the Pods spreading on both zone and node:

{{< codenew file="pods/topology-spread-constraints/two-constraints.yaml" >}}

In this case, to match the first constraint, the incoming Pod can only be placed onto "zoneB"; while in terms of the second constraint, the incoming Pod can only be placed onto "node4". Then the results of 2 constraints are ANDed, so the only viable option is to place on "node4".

Multiple constraints can lead to conflicts. Suppose you have a 3-node cluster across 2 zones:

```
+---------------+-------+
|     zoneA     | zoneB |
+-------+-------+-------+
| node1 | node2 | node3 |
+-------+-------+-------+
|  P P  |   P   |  P P  |
+-------+-------+-------+
```

If you apply "two-constraints.yaml" to this cluster, you will notice "mypod" stays in `Pending` state. This is because: to satisfy the first constraint, "mypod" can only be put to "zoneB"; while in terms of the second constraint, "mypod" can only put to "node2". Then a joint result of "zoneB" and "node2" returns nothing.

To overcome this situation, you can either increase the `maxSkew` or modify one of the constraints to use `whenUnsatisfiable: ScheduleAnyway`.
 -->
### 示例: 多个 TopologySpreadConstraint

与上一个示例相同， 假设有一个4节点的集群中有三个标签包含标签为 `foo:bar`的 Pod，
分别分布在 1，2，3 号节点上 (一个 `P` 代表一个 Pod)

```
+---------------+---------------+
|     zoneA     |     zoneB     |
+-------+-------+-------+-------+
| node1 | node2 | node3 | node4 |
+-------+-------+-------+-------+
|   P   |   P   |   P   |       |
+-------+-------+-------+-------+
```

这次用两个 TopologySpreadConstraints， 同时通过 区域 和节点为控制 Pod 的分布

{{< codenew file="pods/topology-spread-constraints/two-constraints.yaml" >}}

在这种情况下， 要符合第一个约束， 新加入的 Pod 就分被分配到 “zoneB”
再要符合第二个约束，新加入的 Pod 就分被分配到  “node4”
而这两个约束之间是逻辑与关系，也就最终可分配的就 “node4”。

多个约束可能产生冲突。比如集群在两个区域中有三个节点:

```
+---------------+-------+
|     zoneA     | zoneB |
+-------+-------+-------+
| node1 | node2 | node3 |
+-------+-------+-------+
|  P P  |   P   |  P P  |
+-------+-------+-------+
```

如果在这个集群中执行 `two-constraints.yaml`， 就会发现名称为 `mypod` 的 Pod 状态一直是 `Pending`。
这是因为，要符合第一个约束， 就只能分配到 “zoneB”， 同时要符合第二个约束， 就只能分配到 “node2”
而 “zoneB” 与 “node2” 的交集为空集。

要解决这种情况， 可以通过增加 `maxSkew` 的值，
或 修改其中一个约束的 `whenUnsatisfiable`值为`ScheduleAnyway`
<!--  
### Conventions

There are some implicit conventions worth noting here:

- Only the Pods holding the same namespace as the incoming Pod can be matching candidates.

- Nodes without `topologySpreadConstraints[*].topologyKey` present will be bypassed. It implies that:

  1. the Pods located on those nodes do not impact `maxSkew` calculation - in the above example, suppose "node1" does not have label "zone", then the 2 Pods will be disregarded, hence the incoming Pod will be scheduled into "zoneA".
  2. the incoming Pod has no chances to be scheduled onto this kind of nodes - in the above example, suppose a "node5" carrying label `{zone-typo: zoneC}` joins the cluster, it will be bypassed due to the absence of label key "zone".

- Be aware of what will happen if the incomingPod’s `topologySpreadConstraints[*].labelSelector` doesn’t match its own labels. In the above example, if we remove the incoming Pod’s labels, it can still be placed onto "zoneB" since the constraints are still satisfied. However, after the placement, the degree of imbalance of the cluster remains unchanged - it’s still zoneA having 2 Pods which hold label {foo:bar}, and zoneB having 1 Pod which holds label {foo:bar}. So if this is not what you expect, we recommend the workload’s `topologySpreadConstraints[*].labelSelector` to match its own labels.

- If the incoming Pod has `spec.nodeSelector` or `spec.affinity.nodeAffinity` defined, nodes not matching them will be bypassed.

    Suppose you have a 5-node cluster ranging from zoneA to zoneC:

    ```
    +---------------+---------------+-------+
    |     zoneA     |     zoneB     | zoneC |
    +-------+-------+-------+-------+-------+
    | node1 | node2 | node3 | node4 | node5 |
    +-------+-------+-------+-------+-------+
    |   P   |   P   |   P   |       |       |
    +-------+-------+-------+-------+-------+
    ```

    and you know that "zoneC" must be excluded. In this case, you can compose the yaml as below, so that "mypod" will be placed onto "zoneB" instead of "zoneC". Similarly `spec.nodeSelector` is also respected.

    {{< codenew file="pods/topology-spread-constraints/one-constraint-with-nodeaffinity.yaml" >}}
-->
### 约定

以下为一些值得注意的隐性约定:

- 只有在同一个名字空间中的 Pod 才能作为匹配候选者
- 没有 `topologySpreadConstraints[*].topologyKey` 的节点会被当作旁路，隐含的意思为:
  1. 这些只点上的 Pod 不会被用于计算 `maxSkew`， 在上面的例子中，假设 `node1` 上没有标签 `zone`，
    这时其上的两个 Pod 会被忽略， 这样新加入的 Pod 就会被分配到  “zoneA”
  2. 新加入的 Pod 也不会有机会被分配到此类节点上， 在上面的例子中，
    假设集群中加入了一个 “node5” 上面有个标签为 `zone-typo: zoneC`
    这个节点(区域)会因为没有标签键 `zone` 而被忽略
- 还可能发生的一种情况是 新加入的 Pod 上的 `topologySpreadConstraints[*].labelSelector`
  与自身的标签不匹配。在上面的例子中， 如果删除新加入 Pod 上的标签，该 Pod 也会被分配到 “zoneB”。
  因为约束条件是满足的。 但是在 Pod 分配之后，集群的均衡程度并没有改变， 也就是 `zoneA` 中
  有两个包含标签 `foo:bar`的 Pod， `zoneB` 中 有一个包含标签 `foo:bar`的 Pod。 如果这不是预期的行为，
  官方推荐工作负载的 `topologySpreadConstraints[*].labelSelector` 需要匹配自身的标签。
- 如果新加入的 Pod 还定义了 `spec.nodeSelector` 或 `spec.affinity.nodeAffinity`
  不匹配的节点也会被忽略。

  假如一个包含5个节点的集群，有A，B，C三个分区:
  ```
  +---------------+---------------+-------+
  |     zoneA     |     zoneB     | zoneC |
  +-------+-------+-------+-------+-------+
  | node1 | node2 | node3 | node4 | node5 |
  +-------+-------+-------+-------+-------+
  |   P   |   P   |   P   |       |       |
  +-------+-------+-------+-------+-------+
  ```

  如果想让 zoneC 被排除， 这时可以使用以下配置，让 `mypod` 被分配到 `zoneB` 而不是 `zoneC`
  同样的 spec.nodeSelector 也要考量

  {{< codenew file="pods/topology-spread-constraints/one-constraint-with-nodeaffinity.yaml" >}}

<!--
### Cluster-level default constraints

{{< feature-state for_k8s_version="v1.18" state="alpha" >}}

It is possible to set default topology spread constraints for a cluster. Default
topology spread constraints are applied to a Pod if, and only if:

- It doesn't define any constraints in its `.spec.topologySpreadConstraints`.
- It belongs to a service, replication controller, replica set or stateful set.

Default constraints can be set as part of the `PodTopologySpread` plugin args
in a [scheduling profile](/docs/reference/scheduling/profiles).
The constraints are specified with the same [API above](#api), except that
`labelSelector` must be empty. The selectors are calculated from the services,
replication controllers, replica sets or stateful sets that the Pod belongs to.

An example configuration might look like follows:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha2
kind: KubeSchedulerConfiguration

profiles:
  pluginConfig:
    - name: PodTopologySpread
      args:
        defaultConstraints:
          - maxSkew: 1
            topologyKey: failure-domain.beta.kubernetes.io/zone
            whenUnsatisfiable: ScheduleAnyway
```

{{< note >}}
The score produced by default scheduling constraints might conflict with the
score produced by the
[`DefaultPodTopologySpread` plugin](/docs/reference/scheduling/profiles/#scheduling-plugins).
It is recommended that you disable this plugin in the scheduling profile when
using default constraints for `PodTopologySpread`.
{{< /note >}}
 -->
### 集群级的默认约束

{{< feature-state for_k8s_version="v1.18" state="alpha" >}}

可以为一个集群设置默认的拓扑分布约束条件，默认拓扑分布约束条件能且仅能适用于:

- Pod 没有在 `.spec.topologySpreadConstraints` 中定义任何约束条件
- Pod 属于
  {{<glossary_tooltip term_id="service">}}
  {{<glossary_tooltip term_id="replication-controller">}}
  {{<glossary_tooltip term_id="replica-set">}}
  {{<glossary_tooltip term_id="statefulset">}} 之一

默认的约束条件可以作为 [scheduling profile](/k8sDocs/reference/scheduling/profiles)
中 `PodTopologySpread` 插件参数的一部分。 这些约束条件可以能过同 [API](#api) 一样设置，
除了 `labelSelector` 必须为空。 选择器通过 Pod 所属的   
{{<glossary_tooltip term_id="service">}}
{{<glossary_tooltip term_id="replication-controller">}}
{{<glossary_tooltip term_id="replica-set">}}
{{<glossary_tooltip term_id="statefulset">}}
计算得出。

以下为示例:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha2
kind: KubeSchedulerConfiguration

profiles:
  pluginConfig:
    - name: PodTopologySpread
      args:
        defaultConstraints:
          - maxSkew: 1
            topologyKey: failure-domain.beta.kubernetes.io/zone
            whenUnsatisfiable: ScheduleAnyway
```

{{< note >}}
由默认调度约束计算的结果可能与[`DefaultPodTopologySpread` plugin](/k8sDocs/reference/scheduling/profiles/#scheduling-plugins)计算结果相冲突。
建议用户在使用`PodTopologySpread`的默认约束时，关掉调度配置中的插件。

{{< /note >}}
<!--
## Comparison with PodAffinity/PodAntiAffiniaty

In Kubernetes, directives related to "Affinity" control how Pods are
scheduled - more packed or more scattered.

- For `PodAffinity`, you can try to pack any number of Pods into qualifying
  topology domain(s)
- For `PodAntiAffinity`, only one Pod can be scheduled into a
  single topology domain.

The "EvenPodsSpread" feature provides flexible options to distribute Pods evenly across different
topology domains - to achieve high availability or cost-saving. This can also help on rolling update
workloads and scaling out replicas smoothly. See [Motivation](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/895-pod-topology-spread#motivation) for more details.
 -->
## 约束条件 vs `PodAffinity/PodAntiAffiniaty`

在 k8s 中， 与 `Affinity` 相关用于控制 Pod 怎么调度的指令，或集中或分散

- 对于 `PodAffinity`， 用户可以尝试向有资格的拓扑域中塞进任意数量的 Pod
- 对于 `PodAntiAffinity`， 一个 Pod 只能被调度到一个拓扑域中

`EvenPodsSpread` 特性提供了灵活的选项来让 Pod 均匀的分布到不同的拓扑域中，来达到高可用或减少开支的目的。
这也可以让滚动发布和动态扩容变得更平滑。更多信息见
[Motivation](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/895-pod-topology-spread#motivation)
<!--
## Known Limitations

As of 1.18, at which this feature is Beta, there are some known limitations:

- Scaling down a Deployment may result in imbalanced Pods distribution.
- Pods matched on tainted nodes are respected. See [Issue 80921](https://github.com/kubernetes/kubernetes/issues/80921)
 -->

## 已知的限制

到 `1.18`，该特性还是 `Beta` 状态， 还有以下已知的限制:

- 收缩 Deployment 的容量可能导致 Pod 分布的不均匀。
- 匹配到有 {{<glossary_tooltip term_id="taint">}} 节点也会被计入， 见[Issue 80921](https://github.com/kubernetes/kubernetes/issues/80921)
