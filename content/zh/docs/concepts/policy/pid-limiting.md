---
title: 进程 ID 的限制与保留
content_type: concept
weight: 40
---
<!--
---
reviewers:
- derekwaynecarr
title: Process ID Limits And Reservations
content_type: concept
weight: 40
---
-->

<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.20" state="stable" >}}

Kubernetes allow you to limit the number of process IDs (PIDs) that a {{< glossary_tooltip term_id="Pod" text="Pod" >}} can use.
You can also reserve a number of allocatable PIDs for each {{< glossary_tooltip term_id="node" text="node" >}}
for use by the operating system and daemons (rather than by Pods).
 -->

{{< feature-state for_k8s_version="v1.20" state="stable" >}}

k8s 允许用户限制
{{< glossary_tooltip term_id="Pod" text="Pod" >}}
可以使用的进程 ID(PID) 数量。 也可以在每个
{{< glossary_tooltip term_id="node" text="node" >}}
上保留一定数量的可分配 PID，让操作系统或守护进程(而不是 Pod)来使用

<!-- body -->
<!--
Process IDs (PIDs) are a fundamental resource on nodes. It is trivial to hit the
task limit without hitting any other resource limits, which can then cause
instability to a host machine.
 -->

进程 ID (PID)是节点上的基础资源。 它可以会在不触发任意其它资源限制的情况下出现任务受限，这可能
会导致宿主机的稳定性变差。

<!--
Cluster administrators require mechanisms to ensure that Pods running in the
cluster cannot induce PID exhaustion that prevents host daemons (such as the
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} or
{{< glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}},
and potentially also the container runtime) from running.
In addition, it is important to ensure that PIDs are limited among Pods in order
to ensure they have limited impact on other workloads on the same node.
 -->

集群管理员需要一种机制来确保集群中的 Pod 不会导致 PID 耗尽，这样可以预防宿主机守护进程(诸如
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}}
或
{{< glossary_tooltip text="kube-proxy" term_id="kube-proxy" >}}
和容器运行时
) 被挤掉。 另外，限制 Pod 使用的 PID 数量很重要是因为限制了它们使用 PID 的数量就减少了他们
与同一个节点上的其它工作负载之间的冲突可能。

<!--
{{< note >}}
On certain Linux installations, the operating system sets the PIDs limit to a low default,
such as `32768`. Consider raising the value of `/proc/sys/kernel/pid_max`.
{{< /note >}}
 -->

{{< note >}}
在特定 Linux 安装上， 操作系统设置的 PID 限制的默认值是比较低的， 如果 `32768`. 可以考虑增加
`/proc/sys/kernel/pid_max` 的值.
{{< /note >}}

<!--
You can configure a kubelet to limit the number of PIDs a given Pod can consume.
For example, if your node's host OS is set to use a maximum of `262144` PIDs and
expect to host less than `250` Pods, one can give each Pod a budget of `1000`
PIDs to prevent using up that node's overall number of available PIDs. If the
admin wants to overcommit PIDs similar to CPU or memory, they may do so as well
with some additional risks. Either way, a single Pod will not be able to bring
the whole machine down. This kind of resource limiting helps to prevent simple
fork bombs from affecting operation of an entire cluster.
 -->

用户可以配置 kubelet 来限制 Pod 可以使用的 PID 的数量。 例如， 如果节点主机系统设置的最大 PID
数量是 `262144` 并期望托管不超过 `250` 个 Pod， 那么能给予每个 Pod 的 PID 数量的预算就是
`1000`， 这样可以防止节点的 PID 用尽。 如果管理想与 CPU 或内存类似的超限使用 PID，那么同时就
要承担额外的风险。 无论哪种方式，一个 Pod 不是能把整个宿主机搞挂的。 这种类型的资源限制可以防止
因为一个简单的 `fork` 为笮就影响整个集群的运行。

<!--
Per-Pod PID limiting allows administrators to protect one Pod from another, but
does not ensure that all Pods scheduled onto that host are unable to impact the node overall.
Per-Pod limiting also does not protect the node agents themselves from PID exhaustion.
 -->

针对单个 Pod 的 PID 限制允许管理员 Pod 之间的保障， 但并不能确保调度到一个节点上的所有的 Pod
不会炸掉节点的的 PID 总和。 针对单个 Pod 的 PID 限制也不能保护节点不出现 PID 耗尽的情况。

<!--
You can also reserve an amount of PIDs for node overhead, separate from the
allocation to Pods. This is similar to how you can reserve CPU, memory, or other
resources for use by the operating system and other facilities outside of Pods
and their containers.
 -->

用户也可以在 Pod 可分配的 PID 外保留一定数量的 PID 给节点。 这也与用户可以保留 CPU，内存，或
其它资源类似， 这些保留的资源会用于 Pod 及其容器外的操作系统和其它设备

<!--
PID limiting is a an important sibling to [compute
resource](/docs/concepts/configuration/manage-resources-containers/) requests
and limits. However, you specify it in a different way: rather than defining a
Pod's resource limit in the `.spec` for a Pod, you configure the limit as a
setting on the kubelet. Pod-defined PID limits are not currently supported.
 -->

PID 限制是与
[计算资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/)
保底请求与限制同样重要的一员。 但用户要以另一个不同的方式来指定: 相较与在 Pod 的 `.spec` 上定义
Pod 的资源限制。用户可以在 kubelet 设置这个限制。 基于 Pod 定义的 PID 限制目前还不支持

<!--
{{< caution >}}
This means that the limit that applies to a Pod may be different depending on
where the Pod is scheduled. To make things simple, it's easiest if all Nodes use
the same PID resource limits and reservations.
{{< /caution >}}
 -->

{{< caution >}}
这种方式定义的含义就是 Pod 所受到的限制可能因它被调度的节点不同而不同。 要让这个变简单，最简单
的试就是让所有的节点使用同样的 PID 资源限制和保留配置。
{{< /caution >}}
<!--
## Node PID limits

Kubernetes allows you to reserve a number of process IDs for the system use. To
configure the reservation, use the parameter `pid=<number>` in the
`--system-reserved` and `--kube-reserved` command line options to the kubelet.
The value you specified declares that the specified number of process IDs will
be reserved for the system as a whole and for Kubernetes system daemons
respectively.

{{< note >}}
Before Kubernetes version 1.20, PID resource limiting with Node-level
reservations required enabling the [feature
gate](/docs/reference/command-line-tools-reference/feature-gates/)
`SupportNodePidsLimit` to work.
{{< /note >}}
 -->

## 节点 PID 限制 {#node-pid-limits}

k8s 允许用户保留一定数量的 PID 以备系统使用。 要配置这个保留值， 可以使用 kubelet 的全久标记
`--system-reserved` 和 `--kube-reserved` 中设置参数 `pid=<number>`， 这里配置的值
分别表示为系统保留的 PID 数量和为 k8s 系统守护进程保留的 PID 数量

{{< note >}}
在 k8s v1.20 之前，想要配置节点级别的 PID 资源限制需要启用 `SupportNodePidsLimit`
[功能阀](/docs/reference/command-line-tools-reference/feature-gates/)
才行
{{< /note >}}
<!--
## Pod PID limits

Kubernetes allows you to limit the number of processes running in a Pod. You
specify this limit at the node level, rather than configuring it as a resource
limit for a particular Pod. Each Node can have a different PID limit.  
To configure the limit, you can specify the command line parameter `--pod-max-pids` to the kubelet, or set `PodPidsLimit` in the kubelet [configuration file](/docs/tasks/administer-cluster/kubelet-config-file/).

{{< note >}}
Before Kubernetes version 1.20, PID resource limiting for Pods required enabling
the [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
`SupportPodPidsLimit` to work.
{{< /note >}}
 -->

## Pod PID 限制 {#pod-pid-limits}

k8s 允许用户限制一个 Pod 中运行的进程的数量。 需要在节点级别批定这个限制，而不是在哪个 Pod
以资源限制的方式指定。 每个节点可以有不同的 PID 限制。 要配置这个限制，可以在 kubelet 的命令行
参数 `--pod-max-pids` 上指定，也可以在 kubelet 的
[配置文件](/k8sDocs/docs/tasks/administer-cluster/kubelet-config-file/)
上通过 `PodPidsLimit` 上设置。

{{< note >}}
在 k8s v1.20 之前，要配置 Pod 的 PID 资源限制，需要启用 `SupportPodPidsLimit`
[功能阀](/docs/reference/command-line-tools-reference/feature-gates/)
才行
{{< /note >}}
<!--
## PID based eviction

You can configure kubelet to start terminating a Pod when it is misbehaving and consuming abnormal amount of resources.
This feature is called eviction. You can [Configure Out of Resource Handling](/docs/tasks/administer-cluster/out-of-resource) for various eviction signals.
Use `pid.available` eviction signal to configure the threshold for number of PIDs used by Pod.
You can set soft and hard eviction policies. However, even with the hard eviction policy, if the number of PIDs growing very fast,
node can still get into unstable state by hitting the node PIDs limit.
Eviction signal value is calculated periodically and does NOT enforce the limit.

PID limiting - per Pod and per Node sets the hard limit.
Once the limit is hit, workload will start experiencing failures when trying to get a new PID.
It may or may not lead to rescheduling of a Pod,
depending on how workload reacts on these failures and how liveleness and readiness
probes are configured for the Pod. However, if limits were set correctly,
you can guarantee that other Pods workload and system processes will not run out of PIDs
when one Pod is misbehaving.
 -->

## 基于 PID 的驱逐 {#pid-based-eviction}

管理员可以配置 kubelet 在 Pod 有不当行为并使用异常数量的资源是开始终止它。 这个特性叫驱逐。
可以通过
[配置资源用尽处理](/k8sDocs/docs/tasks/administer-cluster/out-of-resource)
来应用多种驱逐信号。
使用 `pid.available` 驱逐信息来配置 Pod 使用的 PID 数量的阈值。 可以配置软硬两种驱逐策略。
但，即便是硬驱逐策略，如果使用的 PID 的数量涨得太快，节点还是会因为达到节点 PID 上限而进入不稳定状态。
驱逐信息值是定时计算而且并 **不是** 强制执行这个限制

PID  限制的单 Pod 和单 节点设置都是硬限制。 当达到限制时，工作负载在尝试获取新的 PID 时就会失败。
它可以导致也可能不导致 Pod 的重新调度， 这取决取工作负载是怎么应对这些失败的和 Pod 的存活和就绪
探针是怎么配置的。 但，如果限制配置是正确的， 这可以保证其它的 Pod 工作负载和系统进程不会因为一个
Pod 的行为异常而导致 PID 耗尽然后一起受影响。

## {{% heading "whatsnext" %}}
<!--
- Refer to the [PID Limiting enhancement document](https://github.com/kubernetes/enhancements/blob/097b4d8276bc9564e56adf72505d43ce9bc5e9e8/keps/sig-node/20190129-pid-limiting.md) for more information.
- For historical context, read [Process ID Limiting for Stability Improvements in Kubernetes 1.14](/blog/2019/04/15/process-id-limiting-for-stability-improvements-in-kubernetes-1.14/).
- Read [Managing Resources for Containers](/docs/concepts/configuration/manage-resources-containers/).
- Learn how to [Configure Out of Resource Handling](/docs/tasks/administer-cluster/out-of-resource).
 -->
- 更多信息见 [PID 限制增加文档](https://github.com/kubernetes/enhancements/blob/097b4d8276bc9564e56adf72505d43ce9bc5e9e8/keps/sig-node/20190129-pid-limiting.md)
- 关于历史背景, 见[Process ID Limiting for Stability Improvements in Kubernetes 1.14](https://kubernetes.io/blog/2019/04/15/process-id-limiting-for-stability-improvements-in-kubernetes-1.14/).
- 概念 [管理容器资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/).
- 学习怎么 [配置资源用尽处理](/k8sDocs/docs/tasks/administer-cluster/out-of-resource).
