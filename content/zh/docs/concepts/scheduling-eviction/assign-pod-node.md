---
title: 分配 Pod 到节点
content_type: concept
weight: 30
---

<!--
---
reviewers:
- davidopp
- kevin-wangzefeng
- bsalamat
title: Assigning Pods to Nodes
content_type: concept
weight: 50
---
 -->

<!-- overview -->
<!--
You can constrain a {{< glossary_tooltip text="Pod" term_id="pod" >}} to only be able to run on particular
{{< glossary_tooltip text="Node(s)" term_id="node" >}}, or to prefer to run on particular nodes.
There are several ways to do this, and the recommended approaches all use
[label selectors](/docs/concepts/overview/working-with-objects/labels/) to make the selection.
Generally such constraints are unnecessary, as the scheduler will automatically do a reasonable placement
(e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.)
but there are some circumstances where you may want more control on a node where a pod lands, for example to ensure
that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different
services that communicate a lot into the same availability zone.
 -->

用户可以约束一个
{{< glossary_tooltip text="Pod" term_id="pod" >}}
只能运行在特定一个(些)
{{< glossary_tooltip term_id="node" >}}
上, 或偏好运行在特定一个(些)节点上。 有几种方式可以做到这一点， 推荐的方式都使用
[标签选择器](/k8sDocs/docs/concepts/overview/working-with-objects/labels/)
进行选择。 通常这种约束不是必要的，因为调度器会自动地进行合理调度(例如: 将 Pod 分散到节点中，
不会将 Pod 放在资源不足的节点上，等) 但也有些情况希望控制 Pod 调度的节点，例如，确保 Pod
最终调度到一个使用 SSD 的节点， 或将两有大量通信的 `Service` 的 Pod 放置在同一个可用区

<!-- body -->
<!--
## nodeSelector

`nodeSelector` is the simplest recommended form of node selection constraint.
`nodeSelector` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible
to run on a node, the node must have each of the indicated key-value pairs as labels (it can have
additional labels as well). The most common usage is one key-value pair.

Let's walk through an example of how to use `nodeSelector`.
 -->

## `nodeSelector`

`nodeSelector` 是推荐的最简单的节点选择的形式。
`nodeSelector` 是 Pod `.spec` 的一个字段。 它是一个键值对形式的字典。 用户选择 Pod 运行的节点。
这个节点必须要有 `nodeSelector` 上指定的每一个键值对作为标签(它还可以有额外的标签)。 最常用的是一个键值对。

让我们使用一个示例来展示 `nodeSelector` 是怎么用的。

<!--
### Step Zero: Prerequisites

This example assumes that you have a basic understanding of Kubernetes pods and that you have [set up a Kubernetes cluster](/docs/setup/).
 -->

### 第零步: 准备 {#step-zero-prerequisites}

这个示例假定你对 k8s Pod  有基本理解，并且你需要
[搭建一个 k8s 集群](/docs/setup/).

<!--
### Step One: Attach label to the node

Run `kubectl get nodes` to get the names of your cluster's nodes. Pick out the one that you want to add a label to, and then run `kubectl label nodes <node-name> <label-key>=<label-value>` to add a label to the node you've chosen. For example, if my node name is 'kubernetes-foo-node-1.c.a-robinson.internal' and my desired label is 'disktype=ssd', then I can run `kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`.

You can verify that it worked by re-running `kubectl get nodes --show-labels` and checking that the node now has a label. You can also use `kubectl describe node "nodename"` to see the full list of labels of the given node.
 -->

### 第一步: 给节点打标签 {#step-one-attach-label-to-the-node}

运行
```shell
kubectl get nodes
```
查看集群中的节点。 选一个喜欢的， 运行
```
kubectl label nodes <node-name> <label-key>=<label-value>
```

在你选择的节点上加一个标签。 例如， 如果我的节点名称为
'kubernetes-foo-node-1.c.a-robinson.internal'
我想要加的标签是 'disktype=ssd'， 要运行的命令就是
```shell
`kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`
```

要检查标签是不是加上了，可以运行
```shell
kubectl get nodes --show-labels
```
看看那个节点是不是
有这个标签。 也可以使用
```
kubectl describe node "nodename"
```
 查看这个节点的完整标签列表。

### 第二步: 在 Pod 配置中添加 `nodeSelector` 字段 {#step-two-add-a-nodeselector-field-to-your-pod-configuration}

选一个你想运行的 Pod 配置文件， 然后给它加一个下面这样的 `nodeSelector`。
例如， 如果 Pod 的配置是这样的

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
```

加上 `nodeSelector` 后就是这样:

{{< codenew file="pods/pod-nginx.yaml" >}}

当运行
```shell
kubectl apply -f https://k8s.io/examples/pods/pod-nginx.yaml
```
后， 这个 Pod 就会被调度到那个打过标签的节点。 要验证可以运行命令
```shell
kubectl get pods -o wide
```
查看输出结果中这个 Pod 的 "NODE" 列是不是指定向的是那个节点
<!--
## Interlude: built-in node labels {#built-in-node-labels}

In addition to labels you [attach](#step-one-attach-label-to-the-node), nodes come pre-populated
with a standard set of labels. See [Well-Known Labels, Annotations and Taints](/docs/reference/kubernetes-api/labels-annotations-taints/) for a list of these.

{{< note >}}
The value of these labels is cloud provider specific and is not guaranteed to be reliable.
For example, the value of `kubernetes.io/hostname` may be the same as the Node name in some environments
and a different value in other environments.
{{< /note >}}
 -->

## 插一嘴: 内置节点标签 {#built-in-node-labels}

在你
[加](#step-one-attach-label-to-the-node)
的标签外。 节点也可能预先就有一些标签标签。 详见
 [常见 标签, 注解和毒点(Taint)](https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/)
{{< note >}}
这些标签的值是与云提供商关联的，而且并不保证可靠。 例如， 在一些环境中 `kubernetes.io/hostname`
的值可能与节点名称相同，另一些环境则不同
{{< /note >}}

<!--
## Node isolation/restriction

Adding labels to Node objects allows targeting pods to specific nodes or groups of nodes.
This can be used to ensure specific pods only run on nodes with certain isolation, security, or regulatory properties.
When using labels for this purpose, choosing label keys that cannot be modified by the kubelet process on the node is strongly recommended.
This prevents a compromised node from using its kubelet credential to set those labels on its own Node object,
and influencing the scheduler to schedule workloads to the compromised node.

The `NodeRestriction` admission plugin prevents kubelets from setting or modifying labels with a `node-restriction.kubernetes.io/` prefix.
To make use of that label prefix for node isolation:

1. Ensure you are using the [Node authorizer](/docs/reference/access-authn-authz/node/) and have _enabled_ the [NodeRestriction admission plugin](/docs/reference/access-authn-authz/admission-controllers/#noderestriction).
2. Add labels under the `node-restriction.kubernetes.io/` prefix to your Node objects, and use those labels in your node selectors.
For example, `example.com.node-restriction.kubernetes.io/fips=true` or `example.com.node-restriction.kubernetes.io/pci-dss=true`.
 -->

## 节点的限离与限制 {#node-isolation-restriction}

在节点对象上加标签可以使得 Pod 运行在指定的节点或一组节点。 这可以用来确保指定 Pod 只能运行中
有特定隔离，安全或监控性质的节点上。
当将节点用于这个目的时， 在选择标签键时强烈推荐那些不能被节点上运行 kubelet 进程修改的那些键。
这能防止在节点被黑后，攻击者使用节点上 kubelet 的凭据在它的节点对象上设置这些标签，然后影响
调度器将工作负载调度到被黑的节点。

`NodeRestriction` 准入插件会防止 kubelet 设置或修改使用 `node-restriction.kubernetes.io/`
前缀的标签。要使用这个标签前缀来实现节点隔离:

1. 确保你在使用 [Node authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/) 和
  _启用了_
  [NodeRestriction 准入插件](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#noderestriction).
2. 在节点对象上添加 `node-restriction.kubernetes.io/` 前缀的标签，并在节点选择器中使用这些标签。
  例如， `example.com.node-restriction.kubernetes.io/fips=true` 或 `example.com.node-restriction.kubernetes.io/pci-dss=true`.

<!--
## Affinity and anti-affinity

`nodeSelector` provides a very simple way to constrain pods to nodes with particular labels. The affinity/anti-affinity
feature, greatly expands the types of constraints you can express. The key enhancements are

1. The affinity/anti-affinity language is more expressive. The language offers more matching rules
   besides exact matches created with a logical AND operation;
2. you can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler
   can't satisfy it, the pod will still be scheduled;
3. you can constrain against labels on other pods running on the node (or other topological domain),
   rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located

The affinity feature consists of two types of affinity, "node affinity" and "inter-pod affinity/anti-affinity".
Node affinity is like the existing `nodeSelector` (but with the first two benefits listed above),
while inter-pod affinity/anti-affinity constrains against pod labels rather than node labels, as
described in the third item listed above, in addition to having the first and second properties listed above.
 -->

## 亲和性与反亲和性 {#affinity-and-anti-affinity}

`nodeSelector` 提供了一个十分简单的方式可以使用特定标签将 Pod 约束到某些节点上。 而亲和性与反亲和性
特性则是大大地扩展了对约束的表达能力。 主要增加有

1. 亲和性/反亲和性 语言有更好的表达能力。 在精确匹配的基础上，通过逻辑与(`AND`)操作提供更多的匹配规则
2. 可以将规则设置为 "软"/"偏好"，而不是强制要求，这样如果调度器找不到满足这个规则的节点也可以
  让 Pod 完成调度
3. 还可以将约束应用在运行在同一个节点(或其它拓扑域)的 Pod 上，而不止是节点本身的标签。 这使得
  这些规则可以约束哪些 Pod 不能待在一起

亲和特性由两种类型的亲和性组成， "节点亲和性" 和 "Pod 间亲和性/反亲和性"。
节点亲和性与 `nodeSelector` 相似(但是有上面提到的增加中的前两条优势)，
而 Pod 间亲和性/反亲和性是基于 Pod 的标签而不是节点的标签进行约束的， 就如上面第三条提到的，
同时第一二条的优势也是有的

<!--
### Node affinity

Node affinity is conceptually similar to `nodeSelector` -- it allows you to constrain which nodes your
pod is eligible to be scheduled on, based on labels on the node.

There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution`. You can think of them as "hard" and "soft" respectively,
in the sense that the former specifies rules that *must* be met for a pod to be scheduled onto a node (just like
`nodeSelector` but using a more expressive syntax), while the latter specifies *preferences* that the scheduler
will try to enforce but will not guarantee. The "IgnoredDuringExecution" part of the names means that, similar
to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer
met, the pod will still continue to run on the node. In the future we plan to offer
`requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution`
except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.

Thus an example of `requiredDuringSchedulingIgnoredDuringExecution` would be "only run the pod on nodes with Intel CPUs"
and an example `preferredDuringSchedulingIgnoredDuringExecution` would be "try to run this set of pods in failure
zone XYZ, but if it's not possible, then allow some to run elsewhere".

Node affinity is specified as field `nodeAffinity` of field `affinity` in the PodSpec.

Here's an example of a pod that uses node affinity:

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

This node affinity rule says the pod can only be placed on a node with a label whose key is
`kubernetes.io/e2e-az-name` and whose value is either `e2e-az1` or `e2e-az2`. In addition,
among nodes that meet that criteria, nodes with a label whose key is `another-node-label-key` and whose
value is `another-node-label-value` should be preferred.

You can see the operator `In` being used in the example. The new node affinity syntax supports the following operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.
You can use `NotIn` and `DoesNotExist` to achieve node anti-affinity behavior, or use
[node taints](/docs/concepts/scheduling-eviction/taint-and-toleration/) to repel pods from specific nodes.

If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the pod
to be scheduled onto a candidate node.

If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of the** `nodeSelectorTerms` can be satisfied.

If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` is satisfied.

If you remove or change the label of the node where the pod is scheduled, the pod won't be removed. In other words, the affinity selection works only at the time of scheduling the pod.

The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding "weight" to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.
 -->

<!--
### Node affinity

Node affinity is conceptually similar to `nodeSelector` -- it allows you to constrain which nodes your
pod is eligible to be scheduled on, based on labels on the node.

There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution`. You can think of them as "hard" and "soft" respectively,
in the sense that the former specifies rules that *must* be met for a pod to be scheduled onto a node (just like
`nodeSelector` but using a more expressive syntax), while the latter specifies *preferences* that the scheduler
will try to enforce but will not guarantee. The "IgnoredDuringExecution" part of the names means that, similar
to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer
met, the pod will still continue to run on the node. In the future we plan to offer
`requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution`
except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.

Thus an example of `requiredDuringSchedulingIgnoredDuringExecution` would be "only run the pod on nodes with Intel CPUs"
and an example `preferredDuringSchedulingIgnoredDuringExecution` would be "try to run this set of pods in failure
zone XYZ, but if it's not possible, then allow some to run elsewhere".

Node affinity is specified as field `nodeAffinity` of field `affinity` in the PodSpec.

Here's an example of a pod that uses node affinity:

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

This node affinity rule says the pod can only be placed on a node with a label whose key is
`kubernetes.io/e2e-az-name` and whose value is either `e2e-az1` or `e2e-az2`. In addition,
among nodes that meet that criteria, nodes with a label whose key is `another-node-label-key` and whose
value is `another-node-label-value` should be preferred.

You can see the operator `In` being used in the example. The new node affinity syntax supports the following operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.
You can use `NotIn` and `DoesNotExist` to achieve node anti-affinity behavior, or use
[node taints](/docs/concepts/scheduling-eviction/taint-and-toleration/) to repel pods from specific nodes.

If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the pod
to be scheduled onto a candidate node.

If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of the** `nodeSelectorTerms` can be satisfied.

If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` is satisfied.

If you remove or change the label of the node where the pod is scheduled, the pod won't be removed. In other words, the affinity selection works only at the time of scheduling the pod.

The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding "weight" to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.
 -->

### 节点亲和性 {#node-affinity}

节点亲和性概念上与 `nodeSelector` 类似 -- 它允许用户基于节点上的标签来约束哪些 Pod 应该调度
到哪个节点上。

目前有两种类型的节点亲和性，叫做 `requiredDuringSchedulingIgnoredDuringExecution` 和
`preferredDuringSchedulingIgnoredDuringExecution`. 可以认为他们分别相当于 "硬" 和 "软"，
从这个意义上说，前一个类型指定规则 *必须要* 被满足了 Pod 才会被调度到一个节点(就像 `nodeSelector`
但使用表达能力更强的语法)， 而后者指定的是 *优先*，也是就调度会尝试满足但不保证会满足。
名称中的  "IgnoredDuringExecution" 的含义是如果节点的标签在运行时改变，Pod 上的节点亲和性
规则不再满足，Pod 还是继续会在节点上运行，就和 `nodeSelector` 的工作方式类似。在未来还计划
提供 `requiredDuringSchedulingRequiredDuringExecution` 它就和
`requiredDuringSchedulingIgnoredDuringExecution` 差不多，区别就是它会在节点不满足 Pod
上的节点亲和性时会将 Pod 从节点上驱逐。

`requiredDuringSchedulingIgnoredDuringExecution` 的一个示例用法: 只能把 Pod 运行在
使用英特尔 CPU 的节点上。
`preferredDuringSchedulingIgnoredDuringExecution` 的一个示例用法: 尝试把这组 Pod 运行在
XYZ 这个失效区，但如果办不到， 也允许在其它地方运行。

节点亲和性是通过 Pod `.spec.affinity.nodeAffinity` 字段指定的。

下面是一个使用节点亲和性的 Pod 的示例:

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

这个节点亲和性规则的要求是 Pod 只能被放在一个拥有一个标签，这个标签的键是 `kubernetes.io/e2e-az-name`
，标签的值是 `e2e-az1` 或 `e2e-az2`. 另外，在满足上一条件的节点中，包含一个键为 `another-node-label-key`
值为 `another-node-label-value` 的标签的节点优先。

可以看到这个示例中使用了 `In` 操作符。 新的节点亲和性语法支持以下操作符: `In`, `NotIn`,
`Exists`, `DoesNotExist`, `Gt`, `Lt`. 可以使用 `NotIn` 和 `DoesNotExist` 来达成反
亲和行为， 或使用
[节点毒点(Taint)](/k8sDocs/docs/concepts/scheduling-eviction/taint-and-toleration/) 让 Pod
避开指定节点。

如果同时指定了 `nodeSelector` 和 `nodeAffinity`，只有同时满足的候选节点才能让 Pod 调度上去。

如果在 `nodeAffinity` 中配置了多个 `nodeSelectorTerms`，当 **有一个** `nodeSelectorTerms`
被满足 Pod 就可以调度到这个节点。(这个有点歧义)

如果在 `nodeSelectorTerms` 中指定了多个 `matchExpressions`， Pod **在且仅在所有**
 `matchExpressions` 都满足时才能被调度到这个节点上

当移除或修改 Pod 运行的节点上的标签时， Pod 不会被移除。 换一句话说， 亲和性配置只有在 Pod
调度时生效。

`preferredDuringSchedulingIgnoredDuringExecution` 中 `weight` 字段的范围是 `1-100`
对于满足所有调度要求(资源要求， `RequiredDuringScheduling` 亲和性表达式，等)，如果节点满足
对应的匹配表达式，调度器会迭代元素中的这个字段，并将值加到 "权重" 得分中。再将这个分与节点的
其它优先级函数得分组合。 最终得分最就的节点(一个或多个)就有最高的优先级

{{<todo-optimize>}}

<!--
#### Node affinity per scheduling profile

{{< feature-state for_k8s_version="v1.20" state="beta" >}}

When configuring multiple [scheduling profiles](/docs/reference/scheduling/config/#multiple-profiles), you can associate
a profile with a Node affinity, which is useful if a profile only applies to a specific set of Nodes.
To do so, add an `addedAffinity` to the args of the [`NodeAffinity` plugin](/docs/reference/scheduling/config/#scheduling-plugins)
in the [scheduler configuration](/docs/reference/scheduling/config/). For example:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
  - schedulerName: foo-scheduler
    pluginConfig:
      - name: NodeAffinity
        args:
          addedAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: scheduler-profile
                  operator: In
                  values:
                  - foo
```

The `addedAffinity` is applied to all Pods that set `.spec.schedulerName` to `foo-scheduler`, in addition to the
NodeAffinity specified in the PodSpec.
That is, in order to match the Pod, Nodes need to satisfy `addedAffinity` and the Pod's `.spec.NodeAffinity`.

Since the `addedAffinity` is not visible to end users, its behavior might be unexpected to them. We
recommend to use node labels that have clear correlation with the profile's scheduler name.

{{< note >}}
The DaemonSet controller, which [creates Pods for DaemonSets](/docs/concepts/workloads/controllers/daemonset/#scheduled-by-default-scheduler)
is not aware of scheduling profiles. For this reason, it is recommended that you keep a scheduler profile, such as the
`default-scheduler`, without any `addedAffinity`. Then, the Daemonset's Pod template should use this scheduler name.
Otherwise, some Pods created by the Daemonset controller might remain unschedulable.
{{< /note >}}
 -->

#### 每个调度方案的节点亲和性 {#node-affinity-per-scheduling-profile}

{{< feature-state for_k8s_version="v1.20" state="beta" >}}

在配置多个
[调度方案](https://kubernetes.io/docs/reference/scheduling/config/#multiple-profiles)
时，可以在调度方案是添加节点亲和性，如果一个方案只会应用到指定的节点集时，这么做是相当有用的。
而要在方案中加亲和性则要在
[调度器配置](https://kubernetes.io/docs/reference/scheduling/config/)
中的
[`NodeAffinity` 插件](https://kubernetes.io/docs/reference/scheduling/config/#scheduling-plugins)
的参数中添加 `addedAffinity`。 例如:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
  - schedulerName: foo-scheduler
    pluginConfig:
      - name: NodeAffinity
        args:
          addedAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: scheduler-profile
                  operator: In
                  values:
                  - foo
```

`addedAffinity` 会应用到所有设置 `.spec.schedulerName` 为 `foo-scheduler` 的 Pod,
再加上这些 Pod `.spec` 中指定的节点亲和性配置为最终的节点亲和性。

因为 `addedAffinity` 对终端用户是不可见的，导致最终行为不符合用户预期。
建议使用的标签要与方案的调度器名称有清晰的关联。


{{< note >}}
`DaemonSet` 控制器，是用来
[为 DaemonSet 创建 Pod 的](/k8sDocs/docs/concepts/workloads/controllers/daemonset/#scheduled-by-default-scheduler)
，它是不能感知到调度方案的。 因为这个原因， 推荐保留一个没有任何 `addedAffinity` 的调度方案，就
如上面例子中的 `default-scheduler`。 这样 `Daemonset` 的 Pod 模板就可以使用这个调度器名称。
否则，有些由 `DaemonSet` 控制器创建的 Pod 就能就一直是不可调度状态。
{{< /note >}}

<!--
### Inter-pod affinity and anti-affinity

Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on
labels on pods that are already running on the node* rather than based on labels on nodes. The rules are of the form
"this pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y".
Y is expressed as a LabelSelector with an optional associated list of namespaces; unlike nodes, because pods are namespaced
(and therefore the labels on pods are implicitly namespaced),
a label selector over pod labels must specify which namespaces the selector should apply to. Conceptually X is a topology domain
like node, rack, cloud provider zone, cloud provider region, etc. You express it using a `topologyKey` which is the
key for the node label that the system uses to denote such a topology domain; for example, see the label keys listed above
in the section [Interlude: built-in node labels](#built-in-node-labels).

{{< note >}}
Inter-pod affinity and anti-affinity require substantial amount of
processing which can slow down scheduling in large clusters significantly. We do
not recommend using them in clusters larger than several hundred nodes.
{{< /note >}}

{{< note >}}
Pod anti-affinity requires nodes to be consistently labelled, in other words every node in the cluster must have an appropriate label matching `topologyKey`. If some or all nodes are missing the specified `topologyKey` label, it can lead to unintended behavior.
{{< /note >}}

As with node affinity, there are currently two types of pod affinity and anti-affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution` which denote "hard" vs. "soft" requirements.
See the description in the node affinity section earlier.
An example of `requiredDuringSchedulingIgnoredDuringExecution` affinity would be "co-locate the pods of service A and service B
in the same zone, since they communicate a lot with each other"
and an example `preferredDuringSchedulingIgnoredDuringExecution` anti-affinity would be "spread the pods from this service across zones"
(a hard requirement wouldn't make sense, since you probably have more pods than zones).

Inter-pod affinity is specified as field `podAffinity` of field `affinity` in the PodSpec.
And inter-pod anti-affinity is specified as field `podAntiAffinity` of field `affinity` in the PodSpec.
 -->

### Pod 间的亲和性与反亲和性 {#inter-pod-affinity-and-anti-affinity}

Pod 间的亲和性与反亲和性允许用户通过 *已经运行在节点上的 Pod 的标签* 而不是节点的标签来决定一个 Pod
应该调度到哪个节点上。
其中的规则的形式是这样的: 这个 Pod 应该(或，在反亲和性的情况下，不应该)运行在 X 上，如果
X 上已经运行了一个或多个匹配 规则 Y 的 Pod。 规则 Y 是以标签选择器的形式存在的，其它可以还有
一个可选的命名空间列表，因为与节点不一样， Pod 是有命名空间的(因此 Pod 上的标签隐形的就是有命名空间的)
对于 Pod 标签的标签选择器必须指定这个选择器应用的命名空间。 概念 X 是一个与节点, 机架，云提供商
的区域，云提供商的地区，等一样的拓扑域，可以使用一个 `topologyKey` 来表示它，也就是系统用来
表示这个拓扑域的节点标签的键。 例子见上面的
[插一嘴: 内置节点标签](#built-in-node-labels).

{{< note >}}
Pod 间的亲和性与反亲和性需要大量的处理过程，如果在大集群中会极是的减慢调度的速度。 不建议使用在
集群节点大于几百的节点中。
{{< /note >}}

{{< note >}}
Pod 反亲和性需要节点都要打上标签，也就是集群中的每个节点必须要有与 `topologyKey` 匹配的适当的
标签。 如果其中的一些或全部没有指定 `topologyKey` 标签，如能会导致意外的情况。
{{< /note >}}

与节点亲和性一样， Pod 的亲和性与反亲和性目前也有两种类型，
`requiredDuringSchedulingIgnoredDuringExecution` 和
`preferredDuringSchedulingIgnoredDuringExecution`
分别表示为 "硬" vs. "软" 要求。
`requiredDuringSchedulingIgnoredDuringExecution` 亲和性的一个应用示例:
因为 Service A 和 Service B 之间有大量通信，把它们的 Pod 放置在同一个可用区。
`preferredDuringSchedulingIgnoredDuringExecution` 反亲和性的一个应用示例:
把这个 Service 的 Pod 分散到不同的区域中(在这种情况下，硬性要求就可能会出问题了， 因为 Pod 的
数量可能是大于区域的)

Pod 间的亲和性是通过 Pod `.spec.affinity.podAffinity` 来指定的。 而反亲和性则是由
`.spec.affinity.podAntiAffinity` 来指定的。

#### 一个使用 Pod 亲和性的示例: {#an-example-of-a-pod-that-uses-pod-affinity}

{{< codenew file="pods/pod-with-pod-affinity.yaml" >}}

这个 Pod 中的亲和性定义有一个 Pod 亲和性规则和一个 Pod 反亲和性规则， 在这个例子中
`podAffinity` 是 `requiredDuringSchedulingIgnoredDuringExecution` 而
`podAntiAffinity` 是 `preferredDuringSchedulingIgnoredDuringExecution`.
Pod 亲和性规则的含义是: Pod 仅能被调度到一个所在区域中至少有一个已经运行了包含标签键为
`security` 值为 `S1` 的 Pod 的节点(也就是如果节点 N 有一个标签键为  `topology.kubernetes.io/zone`
假定标签值为 `V`, 然后集群中至少还有另一个节点标签键为  `topology.kubernetes.io/zone`
标签值为 `V`, 后面这个节点上运行了一个 Pod 包含了标签 `security: S1`).
Pod 反亲和性规则的含义是: Pod 不应当被调度到节点所在区域中的其它节点上有运行 Pod 包含标签为
`security: S2` 的节点上。 更多关于 Pod 亲和性与反亲和性的示例见
[设计文稿](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)
其中
`requiredDuringSchedulingIgnoredDuringExecution`
和
`preferredDuringSchedulingIgnoredDuringExecution`
都有

Pod 亲和性与反亲和性和合法操作符有 `In`, `NotIn`, `Exists`, `DoesNotExist`.

原则上， `topologyKey` 可以是任意合法的标签键， 但为了性能和安全的原因，对 `topologyKey`
有些约束:

1. 对于 Pod 亲和性，`requiredDuringSchedulingIgnoredDuringExecution` 和
  `preferredDuringSchedulingIgnoredDuringExecution` 中都不允许有空的 `topologyKey`
2. 对于 Pod 反亲和性，`requiredDuringSchedulingIgnoredDuringExecution` 和
  `preferredDuringSchedulingIgnoredDuringExecution` 中也都不允许有空的 `topologyKey`
3. 对于 `requiredDuringSchedulingIgnoredDuringExecution` Pod 反亲和性， 引入
`LimitPodHardAntiAffinityTopology` 准入控制器来限制 `topologyKey` 为 `kubernetes.io/hostname`
如果想要用在自定义拓扑上，则需要修改准入控制器或直接禁用它
4. 除了上面的情况， `topologyKey` 可以是任意合法的标签键

在 `labelSelector` 和 `topologyKey` 外，可以选择指定一个可选的 `namespaces` 列表，使得
`labelSelector` 需要来匹配它们(与 `labelSelector` 和 `topologyKey` 定义同级)。
如果没有指定或值为空，默认为 亲和性与反亲和性 所在的命名空间中的 Pod。

`requiredDuringSchedulingIgnoredDuringExecution` 亲和性与反亲和性的所有 `matchExpressions`
必须要求被满足时 Pod 才能被调度到这个节点上

<!--
#### More Practical Use-cases

Interpod Affinity and AntiAffinity can be even more useful when they are used with higher
level collections such as ReplicaSets, StatefulSets, Deployments, etc.  One can easily configure that a set of workloads should
be co-located in the same defined topology, eg., the same node.
 -->

#### 更实用的应用场景 {#more-practical-use-cases}

Pod 亲和性与反亲和性在被用在如 ReplicaSet, StatefulSet, Deployment, 等这些更高一级的资源上
时可以发挥更大的作用。 这样可以很空间地配置将一组工作负载放在同一个拓扑中，例如， 同一个节点上。

<!--
##### Always co-located in the same node

In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.

Here is the yaml snippet of a simple redis deployment with three replicas and selector label `app=store`. The deployment has `PodAntiAffinity` configured to ensure the scheduler does not co-locate replicas on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

The below yaml snippet of the webserver deployment has `podAntiAffinity` and `podAffinity` configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label `app=store`. This will also ensure that each web-server replica does not co-locate on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

If we create the above two deployments, our three node cluster should look like below.

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| *webserver-1*        |   *webserver-2*     |    *webserver-3*   |
|  *cache-1*           |     *cache-2*       |     *cache-3*      |

As you can see, all the 3 replicas of the `web-server` are automatically co-located with the cache as expected.

```
kubectl get pods -o wide
```
The output is similar to this:
```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```
 -->

##### 始终调度在同一个节点上 {#always-co-located-in-the-same-node}

在一个3节点的集群中，有一个使用的内存缓存如 redis 的 web 应用。 我们希望 web 服务尽量与缓存
协同调度。

下面是一个简单的三副本的 redis Deployment 的 yaml 片断， 其中选择器标签为 `app=store`。
Deployment 中有 `PodAntiAffinity` 配置，确保调度器不会同副本调度在同一个节点上。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

下面是 web 服务的 Deployment yaml 片断， 其中配置了 `podAntiAffinity` 和 `podAffinity`
这些配置告知调度器它的所有副本都要调度到与标签为 `app=store` 的 Pod 在一起。这也会保证每个
 web 服务的副本不会被调度到同一个节点上

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

如果我们创建了上面的两个 Deployment，我们集群中的三个节点看起来应该就是下面这样。

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| *webserver-1*        |   *webserver-2*     |    *webserver-3*   |
|  *cache-1*           |     *cache-2*       |     *cache-3*      |

如你所见，就如预期中的那样 `web-server` 的三个副本自动地与缓存调度在一起。

```
kubectl get pods -o wide
```
输出结果类似如下:
```
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```

<!--
##### Never co-located in the same node

The above example uses `PodAntiAffinity` rule with `topologyKey: "kubernetes.io/hostname"` to deploy the redis cluster so that
no two instances are located on the same host.
See [ZooKeeper tutorial](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)
for an example of a StatefulSet configured with anti-affinity for high availability, using the same technique.
 -->

##### 永远不要调度到同一个节点上 {#never-co-located-in-the-same-node}

上面的例子中使用包含 `topologyKey: "kubernetes.io/hostname"` 的 `PodAntiAffinity` 来
部署 redis 集群，这样任意两个实例都不会被调度到同一个主机上。
[ZooKeeper 教程](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure)
中的一个示例中，一个 StatefulSet 配置了反亲和性来实现高可用，其中使用了同样的技巧。

<!--
## nodeName

`nodeName` is the simplest form of node selection constraint, but due
to its limitations it is typically not used.  `nodeName` is a field of
PodSpec.  If it is non-empty, the scheduler ignores the pod and the
kubelet running on the named node tries to run the pod.  Thus, if
`nodeName` is provided in the PodSpec, it takes precedence over the
above methods for node selection.

Some of the limitations of using `nodeName` to select nodes are:

-   If the named node does not exist, the pod will not be run, and in
    some cases may be automatically deleted.
-   If the named node does not have the resources to accommodate the
    pod, the pod will fail and its reason will indicate why,
    for example OutOfmemory or OutOfcpu.
-   Node names in cloud environments are not always predictable or
    stable.

Here is an example of a pod config file using the `nodeName` field:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```

The above pod will run on the node kube-01.
 -->

## nodeName

`nodeName` 是实现节点选择约束的最简单方式，但因为它的限制，通常是不会使用的。
`nodeName` 是 Pod  `.spec` 的一个字段。 如果它不是空， 调度器就会忽略这个 Pod， 由 `nodeName`
所设置的名称的节点上的 `kubelet` 来尝试运行这个 Pod。 这样， 如果 Pod `.spec`  中定义了
`nodeName`， 则它会比上面提到的所有节点选择的优先级都高。

使用 `nodeName` 进行节点选择的限制有:

- 如果没有一个名称是这个的节点， 则 Pod 不会被运行，在某些情况下可以会被自动删除。
- 如果叫这个名称的节点上资源不足以运行这个节点，这个 Pod 就会失败，它的状态信息的原因中会显示为啥，
  比如 `OutOfmemory` 或 `OutOfcpu`.
- 云环境中的节点名称可能不是始终可预期的或稳定的。

下面就是一个使用 `nodeName` 字段的 Pod 配置的示例:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```

上面这个 Pod 会运行在节点 `kube-01` 上


## {{% heading "whatsnext" %}}

<!--
[Taints](/docs/concepts/scheduling-eviction/taint-and-toleration/) allow a Node to *repel* a set of Pods.

The design documents for
[node affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)
and for [inter-pod affinity/anti-affinity](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md) contain extra background information about these features.

Once a Pod is assigned to a Node, the kubelet runs the Pod and allocates node-local resources.
The [topology manager](/docs/tasks/administer-cluster/topology-manager/) can take part in node-level
resource allocation decisions.
 -->
[毒点(Taint)](/k8sDocs/docs/concepts/scheduling-eviction/taint-and-toleration/)
允许节点 *排斥* 一些 Pod.


[节点亲和性](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md)
的设计文稿

[Pod 间的 亲和性/反亲和性](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md) 包含了这个特性额外的背景信息

当一个 Pod 被分配给一个节点时， kubelet 运行这个节点并分配节点级别的资源。
[拓扑管理器](/k8sDocs/docs/tasks/administer-cluster/topology-manager/)  可以参与节点级别资源
分配的决定。
