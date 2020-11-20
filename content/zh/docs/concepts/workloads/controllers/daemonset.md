---
title: DaemonSet
date: 2020-08-24
publishdate: 2020-08-25
weight: 2030103
---
<!--
---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
title: DaemonSet
content_type: concept
weight: 40
---
 -->
<!-- overview -->
<!--
A _DaemonSet_ ensures that all (or some) Nodes run a copy of a Pod.  As nodes are added to the
cluster, Pods are added to them.  As nodes are removed from the cluster, those Pods are garbage
collected.  Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

- running a cluster storage daemon on every node
- running a logs collection daemon on every node
- running a node monitoring daemon on every node

In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon.
A more complex setup might use multiple DaemonSets for a single type of daemon, but with
different flags and/or different memory and cpu requests for different hardware types.
 -->
_DaemonSet_ 确保所以(或部分)节点上都会运行一个副本的 Pod。 当有节点加入到集群时，这个 Pod 也会自动加到该节点上。
当节点从集群移除时， 这些 Pod 也会被垃圾清理掉。 删除一个 DaemonSet 也会清除其所创建的 Pod。

DaemonSet 常见使用场景:

- 在每个节点上运行集群存储守护进程
- 在每个节点上运行日志收集守护进程
- 在每个节点上运行监控守护进程

一个简单的用法是对于每个类型的守护进程使用一个 DaemonSet 运行在每一个节点。
复杂点的配置就是对一个类型的守护进程使用多个 DaemonSet， 针对不同的硬件类型使用 不同的标志， 内存， CPU 需求的 DaemonSet
<!-- body -->
<!--
## Writing a DaemonSet Spec

### Create a DaemonSet

You can describe a DaemonSet in a YAML file. For example, the `daemonset.yaml` file below describes a DaemonSet that runs the fluentd-elasticsearch Docker image:

{{< codenew file="controllers/daemonset.yaml" >}}

Create a DaemonSet based on the YAML file:

```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```
 -->
## 编写 DaemonSet 配制

### 创建 DaemonSet

用户可以通过一个 YAML 文件在描述一个 DaemonSet。 例如， 下面的 `daemonset.yaml` 中就描述了一个运行 `fluentd-elasticsearch` Docker 镜像的 DaemonSet:

{{< codenew file="controllers/daemonset.yaml" >}}

基于 YAML 文件创建一个　DaemonSet
```
kubectl apply -f daemonset.yaml
```

<!--
### Required Fields

As with all other Kubernetes config, a DaemonSet needs `apiVersion`, `kind`, and `metadata` fields.  For
general information about working with config files, see
[running stateless applications](/docs/tasks/run-application/run-stateless-application-deployment/),
[configuring containers](/docs/tasks/), and [object management using kubectl](/docs/concepts/overview/working-with-objects/object-management/) documents.

The name of a DaemonSet object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A DaemonSet also needs a [`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) section.
 -->
### 必要字段

与其它所有其它的 k8s 配置一样， DaemonSet 必须有 `apiVersion`, `kind`, `metadata` 字段，
关于配置文件的通用信息见 [运行无状态应用](/k8sDocs/tasks/run-application/run-stateless-application-deployment/),
[配置容器](/k8sDocs/tasks/), [使用 kubectl 管理对象](/k8sDocs/docs/concepts/overview/working-with-objects/object-management/)
DaemonSet 的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

DaemonSet 也是必须要有一个 [`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) 配置区.

<!--
### Pod Template

The `.spec.template` is one of the required fields in `.spec`.

The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a {{< glossary_tooltip text="Pod" term_id="pod" >}}, except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a Pod template in a DaemonSet has to specify appropriate
labels (see [pod selector](#pod-selector)).

A Pod Template in a DaemonSet must have a [`RestartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)
 equal to `Always`, or be unspecified, which defaults to `Always`.
 -->
### Pod 模板

`.spec.template` 是 `.spec` 中的一个必要字段.

`.spec.template` 是一个 [pod 模板](/k8sDocs/docs/concepts/workloads/pods/#pod-templates).
除了因为嵌套没有 `apiVersion` 或 `kind`字段外，与 {{< glossary_tooltip term_id="pod" >}}的定义完全相同,

相较与裸 Pod 增加的字段还有 DaemonSet 需要设置恰当的标签 (见 [Pod 选择器](#pod-selector))

DaemonSet 中的 Pod 模板必须要有 [`RestartPolicy`](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)
且值为 `Always` 或 留空，然后默认为 `Always`
<!--
### Pod Selector

The `.spec.selector` field is a pod selector.  It works the same as the `.spec.selector` of
a [Job](/docs/concepts/workloads/controllers/job/).

As of Kubernetes 1.8, you must specify a pod selector that matches the labels of the
`.spec.template`. The pod selector will no longer be defaulted when left empty. Selector
defaulting was not compatible with `kubectl apply`. Also, once a DaemonSet is created,
its `.spec.selector` can not be mutated. Mutating the pod selector can lead to the
unintentional orphaning of Pods, and it was found to be confusing to users.

The `.spec.selector` is an object consisting of two fields:

* `matchLabels` - works the same as the `.spec.selector` of a [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/).
* `matchExpressions` - allows to build more sophisticated selectors by specifying key,
  list of values and an operator that relates the key and values.

When the two are specified the result is ANDed.

If the `.spec.selector` is specified, it must match the `.spec.template.metadata.labels`. Config with these not matching will be rejected by the API.

Also you should not normally create any Pods whose labels match this selector, either directly, via
another DaemonSet, or via another workload resource such as ReplicaSet.  Otherwise, the DaemonSet
{{< glossary_tooltip term_id="controller" >}} will think that those Pods were created by it.
Kubernetes will not stop you from doing this. One case where you might want to do this is manually
create a Pod with a different value on a node for testing.
 -->
### Pod 选择器 {#pod-selector}

`.spec.selector` 字段是一个 Pod 选择器.  与 [Job](/k8sDocs/docs/concepts/workloads/controllers/job/) 的 `.spec.selector` 是一样的。

从 k8s 1.8 开始，用户必须要设置一个与 `.spec.template` 中标签相匹配的标签选择器。不再是留空会添加默认值。
选择器默认添加的行为与 `kubectl apply` 不兼容。 并且，当一个 DaemonSet 创建后，`.spec.selector` 字段将不可变。
对标签选择器的修改可能无意间导致产生孤儿 Pod， 这样会让用户费解。

`.spec.selector` 对象由以下两个字段组成:

* `matchLabels` - 与 [ReplicationController](/k8sDocs/docs/concepts/workloads/controllers/replicationcontroller/) 中的 `.spec.selector` 作用一样。
* `matchExpressions` - 能通过键与对应的值列表，操作符组成更复杂的选择器
如果以上两个字段都有设置，它们之间是逻辑与关系

如果 `.spec.selector` 设置了值(对应 1.8 之前的版本), 则必须与 `.spec.template.metadata.labels`匹配. 如果配置不匹配会被 api-server 拒绝.

在此之外用户不应该再直接或间接(如其它的 DaemonSet 或如 ReplicaSet 之类的工作负载)创建与该选择器匹配的 Pod，
否则 DaemonSet {{< glossary_tooltip term_id="controller" >}} 会认为这些 Pod 也是它创建的。
k8s 不会阻止用户这么干。 能这么干的和种场景是手动创建与这个不一样配置的 Pod 用于测试
<!--
### Running Pods on select Nodes

If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will
create Pods on nodes which match that [node
selector](/docs/concepts/scheduling-eviction/assign-pod-node/). Likewise if you specify a `.spec.template.spec.affinity`,
then DaemonSet controller will create Pods on nodes which match that [node affinity](/docs/concepts/scheduling-eviction/assign-pod-node/).
If you do not specify either, then the DaemonSet controller will create Pods on all nodes.
 -->
### 只在选定的节点上运行 Pod

如果指定了 `.spec.template.spec.nodeSelector`， 则 DaemonSet 控制器只会在那些匹配 [节点选择器](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/)
节点上创建 Pod。 类似地，如果指定了 `.spec.template.spec.affinity` 控制器只会在那些匹配 [node affinity](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/)
节点上创建 Pod。如果一个都没指定，则 DaemonSet 控制器会在所有的节点上创建 Pod。
<!--
## How Daemon Pods are scheduled

### Scheduled by default scheduler

{{< feature-state state="stable" for-kubernetes-version="1.17" >}}

A DaemonSet ensures that all eligible nodes run a copy of a Pod. Normally, the
node that a Pod runs on is selected by the Kubernetes scheduler. However,
DaemonSet pods are created and scheduled by the DaemonSet controller instead.
That introduces the following issues:

 * Inconsistent Pod behavior: Normal Pods waiting to be scheduled are created
   and in `Pending` state, but DaemonSet pods are not created in `Pending`
   state. This is confusing to the user.
 * [Pod preemption](/docs/concepts/configuration/pod-priority-preemption/)
   is handled by default scheduler. When preemption is enabled, the DaemonSet controller
   will make scheduling decisions without considering pod priority and preemption.

`ScheduleDaemonSetPods` allows you to schedule DaemonSets using the default
scheduler instead of the DaemonSet controller, by adding the `NodeAffinity` term
to the DaemonSet pods, instead of the `.spec.nodeName` term. The default
scheduler is then used to bind the pod to the target host. If node affinity of
the DaemonSet pod already exists, it is replaced (the original node affinity was taken into account before selecting the target host). The DaemonSet controller only
performs these operations when creating or modifying DaemonSet pods, and no
changes are made to the `spec.template` of the DaemonSet.

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```

In addition, `node.kubernetes.io/unschedulable:NoSchedule` toleration is added
automatically to DaemonSet Pods. The default scheduler ignores
`unschedulable` Nodes when scheduling DaemonSet Pods.
 -->
## DaemonSet 的 Pod 是怎么调度的

### 由默认调度器调度

{{< feature-state state="stable" for-kubernetes-version="1.17" >}}

DaemonSet 会确保所有合适的节点者会运行一个 Pod 的副本。 通常 Pod 应该运行在哪个节点上是由 k8s 调度器来选的。
但是 DaemonSet 的 Pod 也可以由 DaemonSet 控制器来创建和调度。
由此也引出了一些问题:

- Pod 的行为不一致: 普通的 Pod 在创建和等待调度时的状态是 `Pending`，但 DaemonSet 的 Pod 创建时不是 `Pending` 状态，这对用户来说比较费解。
- [Pod preemption](/k8sDocs/docs/concepts/configuration/pod-priority-preemption/) 是由默认调度器处理的。 当 优先权被开启时， DaemonSet
控制器在进行调度决策时不会考虑 Pod 的优先级(priority)和优先权(preemption)(这俩有啥区别？)

`ScheduleDaemonSetPods` 允许用户可以让默认调度器来调度 DaemonSet 的 Pod，而不是 DaemonSet 的控制来调度，
只需要向 DaemonSet 的 Pod 模板添加 `NodeAffinity` 项， 而不是添加 `.spec.nodeName` 项就可以。
这时默认调度就与 Pod 的目标主机绑定了。 如果 DaemonSet Pod 已经存在了节点亲和性，会被替换(原本的节点亲和性在选择目标主机之间也会被考量)
DaemonSet 控制器只会在创建或修改 DaemonSet Pod 时执行这些操作， 不会对 DaemonSet 的 `spec.template` 作任何修改。

{{ <todo-optimize> }}

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```

In addition, `node.kubernetes.io/unschedulable:NoSchedule` toleration is added
automatically to DaemonSet Pods. The default scheduler ignores
`unschedulable` Nodes when scheduling DaemonSet Pods.
另外 `node.kubernetes.io/unschedulable:NoSchedule` 耐受会默认添加到 DaemonSet 的 Pod 上。
默认调度器在调度 DaemonSet 的 Pod时会忽略 `unschedulable` 节点

<!--
### Taints and Tolerations

Although Daemon Pods respect
[taints and tolerations](/docs/concepts/scheduling-eviction/taint-and-toleration/),
the following tolerations are added to DaemonSet Pods automatically according to
the related features.

| Toleration Key                           | Effect     | Version | Description |
| ---------------------------------------- | ---------- | ------- | ----------- |
| `node.kubernetes.io/not-ready`           | NoExecute  | 1.13+   | DaemonSet pods will not be evicted when there are node problems such as a network partition. |
| `node.kubernetes.io/unreachable`         | NoExecute  | 1.13+   | DaemonSet pods will not be evicted when there are node problems such as a network partition. |
| `node.kubernetes.io/disk-pressure`       | NoSchedule | 1.8+    | |
| `node.kubernetes.io/memory-pressure`     | NoSchedule | 1.8+    | |
| `node.kubernetes.io/unschedulable`       | NoSchedule | 1.12+   | DaemonSet pods tolerate unschedulable attributes by default scheduler. |
| `node.kubernetes.io/network-unavailable` | NoSchedule | 1.12+   | DaemonSet pods, who uses host network, tolerate network-unavailable attributes by default scheduler. |
 -->
### 毒点(Taint)与耐受(Toleration)

尽管 DaemonSet 的 Pod 是遵守 [毒点与耐受](/k8sDocs/docs/concepts/scheduling-eviction/taint-and-toleration/)的,
但以下耐受会根据相关的特性自动添加到 DaemonSet 的 Pod 上。

| Toleration Key                           | Effect     | Version | Description |
| ---------------------------------------- | ---------- | ------- | ----------- |
| `node.kubernetes.io/not-ready`           | NoExecute  | 1.13+   | DaemonSet 的 Pod 在遇到如网络分区的节点问题时不会衩上踢出去 |
| `node.kubernetes.io/unreachable`         | NoExecute  | 1.13+   | DaemonSet 的 Pod 在遇到如网络分区的节点问题时不会衩上踢出去 |
| `node.kubernetes.io/disk-pressure`       | NoSchedule | 1.8+    | |
| `node.kubernetes.io/memory-pressure`     | NoSchedule | 1.8+    | |
| `node.kubernetes.io/unschedulable`       | NoSchedule | 1.12+   | DaemonSet 的 Pod 由默认调度器调度时耐受 `unschedulable` 属性 |
| `node.kubernetes.io/network-unavailable` | NoSchedule | 1.12+   | DaemonSet 使用主机网络的 Pod 由默认调度器调度时耐受 `network-unavailable` 属性|
<!--
## Communicating with Daemon Pods

Some possible patterns for communicating with Pods in a DaemonSet are:

- **Push**: Pods in the DaemonSet are configured to send updates to another service, such
  as a stats database.  They do not have clients.
- **NodeIP and Known Port**: Pods in the DaemonSet can use a `hostPort`, so that the pods are reachable via the node IPs.  Clients know the list of node IPs somehow, and know the port by convention.
- **DNS**: Create a [headless service](/docs/concepts/services-networking/service/#headless-services) with the same pod selector,
  and then discover DaemonSets using the `endpoints` resource or retrieve multiple A records from
  DNS.
- **Service**: Create a service with the same Pod selector, and use the service to reach a
  daemon on a random node. (No way to reach specific node.)
 -->
## 与 DaemonSet 的 Pod 通信

与 DaemonSet 的 Pod 通信 可行模式有:

- **Push**: DaemonSet 中的 Pod 配置为向另一个服务发送更新， 如 一个状态数据。 它们没有客户端。
- **NodeIP and Known Port**: DaemonSet 中的 Pod 可以使用 `hostPort`， 这样就可以通过节点IP来访问这些 Pod。 客户端可以通过某些方便的方式获得节点的IP 和端口。
- **DNS**: 创建一个有相同 Pod 选择器的 [headless Service](/k8sDocs/docs/concepts/services-networking/service/#headless-services), 这样就可以通过 `endpoints` 找到 DaemonSet 或 通过 DNS 找到一个 A 记录列表
- **Service**: 创建个拥有相同 Pod 选择器的 Service , 通过 Service 随机访问一个节点上的 Pod(但没办法直接访问指定节点)

<!--
## Updating a DaemonSet

If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete
Pods from newly not-matching nodes.

You can modify the Pods that a DaemonSet creates.  However, Pods do not allow all
fields to be updated.  Also, the DaemonSet controller will use the original template the next
time a node (even with the same name) is created.

You can delete a DaemonSet.  If you specify `--cascade=false` with `kubectl`, then the Pods
will be left on the nodes.  If you subsequently create a new DaemonSet with the same selector,
the new DaemonSet adopts the existing Pods. If any Pods need replacing the DaemonSet replaces
them according to its `updateStrategy`.

You can [perform a rolling update](/docs/tasks/manage-daemon/update-daemon-set/) on a DaemonSet.
 -->
## 更新 DaemonSet

如果节点的标签发生变更，则 DaemonSet 及时时在新匹配的节点上添加 Pod，将不匹配的节点上的 Pod 删掉。

用户可以修改由 DaemonSet 创建的 Pod， 但不是所以的 Pod 字段都是可以更新的。 同时 DaemonSet 会使用原版的模板来创建新加入的节点(即使节点名是一样的)

用户可以删除一个 DaemonSet。 如果在删除时 `kubectl` 添加了参数 `--cascade=false`， 则对应的 Pod 会被保留在节点上。
如果用户接着又以相同的选择器创建了新的 DaemonSet， 这个新的 DaemonSet 会接管这些 Pod。 如果有 Pod 需要被替换则会根据 `updateStrategy` 进行替换

用户也可以在 DaemonSet 上[执行滚动更新](/k8sDocs/tasks/manage-daemon/update-daemon-set/)
<!--
## Alternatives to DaemonSet

### Init scripts

It is certainly possible to run daemon processes by directly starting them on a node (e.g. using
`init`, `upstartd`, or `systemd`).  This is perfectly fine.  However, there are several advantages to
running such processes via a DaemonSet:

- Ability to monitor and manage logs for daemons in the same way as applications.
- Same config language and tools (e.g. Pod templates, `kubectl`) for daemons and applications.
- Running daemons in containers with resource limits increases isolation between daemons from app
  containers.  However, this can also be accomplished by running the daemons in a container but not in a Pod
  (e.g. start directly via Docker).
 -->
## DaemonSet 的替代方案

### 初始化脚本

有时可以直接在节点上(通过如 `init`, `upstartd`, `systemd`)的方式直接运行守护进程。 这样也是很好的方案。
但是使用 DaemonSet 运行这些进程有如下优势:

- 可以与应用相同的方式来实现这些进程的监控和日志管理
- 守护进程与应用使用同样的配置语言与工具(如， Pod 模板 `kubectl`)
- 在容器中运行带资源限制的空城计进程可以提与应用容器之间的隔离级别。 当然，这也可以通过在容器中运行守护进程而不是 Pod(如直接通过 Docker 启动)来运行守护进程
<!--
### Bare Pods

It is possible to create Pods directly which specify a particular node to run on.  However,
a DaemonSet replaces Pods that are deleted or terminated for any reason, such as in the case of
node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, you should
use a DaemonSet rather than creating individual Pods.
 -->
### 裸 Pod

可以直接在指定节点上直接创建 Pod。 但这些 Pod 可能因为某些如节点挂掉或节点维护(升级内核)引起的故障原因被删除或终止。
因此应该使用 DaemonSet，而不是单独使用 Pod。
<!--
### Static Pods

It is possible to create Pods by writing a file to a certain directory watched by Kubelet.  These
are called [static pods](/docs/tasks/configure-pod-container/static-pod/).
Unlike DaemonSet, static Pods cannot be managed with kubectl
or other Kubernetes API clients.  Static Pods do not depend on the apiserver, making them useful
in cluster bootstrapping cases.  Also, static Pods may be deprecated in the future.
-->
### 静态 Pod

可以直接在 kubelet 监控的目录中创建 Pod 配置文件. 这些被称为 [静态 Pod](/k8sDocs/tasks/configure-pod-container/static-pod/).
与 DaemonSet 不现, 静态 Pod 不能通过 kubectl 或其它 k8s API 客户端管理. 静态 Pod 也不信赖于 api-server, 让它在
集群初始化的时候很有用. 并且,静态 Pod 可以在未来的版本中被废弃.
<!--
### Deployments

DaemonSets are similar to [Deployments](/docs/concepts/workloads/controllers/deployment/) in that
they both create Pods, and those Pods have processes which are not expected to terminate (e.g. web servers,
storage servers).

Use a Deployment for stateless services, like frontends, where scaling up and down the
number of replicas and rolling out updates are more important than controlling exactly which host
the Pod runs on.  Use a DaemonSet when it is important that a copy of a Pod always run on
all or certain hosts, and when it needs to start before other Pods.
 -->
### Deployment

DaemonSet 与 [Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/) 类似, 它们都可以创建 Pod
并且这些 Pod 运行的都是不应该被终止的进程(如, web 服务器,存储服务器)

Deployment 用于无状态的服务, 如前端,可以通过副本数来扩容或缩减容量, 发布更新比控制 Pod 在哪个主机上运行更重要.
在 Pod 需要始终在所有或特定主机上运行进更重要些时,并且需要它们在其它 Pod 之前启动时,应该使用 DaemonSet
