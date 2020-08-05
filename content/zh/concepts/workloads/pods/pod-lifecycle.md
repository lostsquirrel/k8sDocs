---
title: Pod 生命周期
date: 2020-07-27
publishdate: 2020-08-04
weight: 1
---
<!--
---
title: Pod Lifecycle
content_type: concept
weight: 30
---
 -->
<!-- overview -->
<!--
This page describes the lifecycle of a Pod. Pods follow a defined lifecycle, starting
in the `Pending` [phase](#pod-phase), moving through `Running` if at least one
of its primary containers starts OK, and then through either the `Succeeded` or
`Failed` phases depending on whether any container in the Pod terminated in failure.

Whilst a Pod is running, the kubelet is able to restart containers to handle some
kind of faults. Within a Pod, Kubernetes tracks different container
[states](#container-states) and handles

In the Kubernetes API, Pods have both a specification and an actual status. The
status for a Pod object consists of a set of [Pod conditions](#pod-conditions).
You can also inject [custom readiness information](#pod-readiness-gate) into the
condition data for a Pod, if that is useful to your application.

Pods are only [scheduled](/k8sDocs/concepts/scheduling-eviction/) once in their lifetime.
Once a Pod is scheduled (assigned) to a Node, the Pod runs on that Node until it stops
or is [terminated](#pod-termination).


 -->
本文介经 Pod 的生命周期。 Pod 有一个既定的生命周期，开启后进行 `Pending` [阶段](#pod-phase)，
当其中至少有一个主要容器正常启动后变更为 `Running` 阶段， 如果所有容器全部正常启动则进入 `Succeeded` 阶段，
如果有任意容器启动失败则进行 `Failed` 阶段。

当一个 Pod 在运行中， kubelet 可以在某些情况下容器挂掉后将其重启。 在 Pod 中， k8s 会跟踪和处理容器的 [状态](#container-states)

在 k8s 的 API 对象中， Pod 对象拥有定义明细和实时状态。 Pod 对象的状态上包含一系列 [Pod 条件](#pod-conditions)
如果应用有需要，可以向 Pod 中加入 [自定义就绪信息](#pod-readiness-gate) 到条件子对象。

在 Pod 的整个生命周期中只会被[调度](../../../scheduling-eviction/)一次，
当一个 Pod 被调度(分配)到一个节点后，就会一直运行在这个节点上，直接被停止或被[终止](#pod-termination)
<!-- body -->
<!--
## Pod lifetime

Like individual application containers, Pods are considered to be relatively
ephemeral (rather than durable) entities. Pods are created, assigned a unique
ID ([UID](/k8sDocs/concepts/overview/working-with-objects/names/#uids)), and scheduled
to nodes where they remain until termination (according to restart policy) or
deletion.  
If a {{< glossary_tooltip term_id="node" >}} dies, the Pods scheduled to that node
are [scheduled for deletion](#pod-garbage-collection) after a timeout period.

Pods do not, by themselves, self-heal. If a Pod is scheduled to a
{{< glossary_tooltip text="node" term_id="node" >}} that then fails,
or if the scheduling operation itself fails, the Pod is deleted; likewise, a Pod won't
survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a
higher-level abstraction, called a
{{< glossary_tooltip term_id="controller" text="controller" >}}, that handles the work of
managing the relatively disposable Pod instances.

A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead,
that Pod can be replaced by a new, near-identical Pod, with even the same name i
desired, but with a different UID.

When something is said to have the same lifetime as a Pod, such as a
{{< glossary_tooltip term_id="volume" text="volume" >}},
that means that the thing exists as long as that specific Pod (with that exact UID)
exists. If that Pod is deleted for any reason, and even if an identical replacement
is created, the related thing (a volume, in this example) is also destroyed and
created anew.

{{< figure src="/images/docs/pod.svg" title="Pod diagram" width="50%" >}}

*A multi-container Pod that contains a file puller and a
web server that uses a persistent volume for shared storage between the containers.*

 -->
## Pod 的一生

与单独使用应用容器一样, Pod 可以被认为是一个相对临时(而不是长期存在)的实体. Pod 在创建时会被分配
一个唯一的 ID([UID](/k8sDocs/concepts/overview/working-with-objects/names/#uids)),
然后被到一个 {{< glossary_tooltip text="node" term_id="node" >}} 上,直到被终止(依照重启策略)或者被删除.
如果一个 {{< glossary_tooltip text="node" term_id="node" >}} 挂了, 这个节点上的 Pod 会在超时后
[因删除被调度](#pod-garbage-collection).

Pod 并不能独自实现自愈. 如果 Pod 被调度的 {{< glossary_tooltip text="node" term_id="node" >}} 挂了,
或者是调度操作本身失败, Pod 就被删除了, Pod 也不会在因为资源不足或 {{< glossary_tooltip text="node" term_id="node" >}} 节点被驱逐中幸存.
k8s 通过一个叫 {{< glossary_tooltip term_id="controller" >}} 的更高层级的抽象,来处理这些相对来说是一次的的 Pod 实例的管理工作.

某个 Pod(拥有特定 UID) 是永远不会被重新调度到另一个节点上; 而是被一个基本相同, 甚至可以名称也相同,但 UID 不同的 Pod 所取代.

当某些对象(如 {{< glossary_tooltip term_id="volume" text="volume" >}}) 被描述为与 Pod 拥有一致的生命期,
表示这些对象会与指定的 (拥有那个 UID 的) {{< glossary_tooltip term_id="pod" >}} 同时存在,
如果那个 {{< glossary_tooltip term_id="pod" >}} 因为某些原为被删除, 即便同样的代替 {{< glossary_tooltip term_id="pod" >}} 已经被创建,
相关的对象(如 {{< glossary_tooltip term_id="volume">}} 也会被随同 Pod 一起被销毁重建).

{{< figure src="/k8sDocs/images/docs/pod.svg" alt="Pod diagram" width="50%" >}}
 *有一个容器作为 web 服务，为共享数据卷的文件提供访问服务，另一个独立的容器作为 边车，负责从远程的源更新这些文件*



<!--  
## Pod phase

A Pod's `status` field is a
[PodStatus](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
object, which has a `phase` field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its
lifecycle. The phase is not intended to be a comprehensive rollup of observations
of container or Pod state, nor is it intended to be a comprehensive state machine.

The number and meanings of Pod phase values are tightly guarded.
Other than what is documented here, nothing should be assumed about Pods that
have a given `phase` value.

Here are the possible values for `phase`:

Value | Description
:-----|:-----------
`Pending` | The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to bescheduled as well as the time spent downloading container images over the network.
`Running` | The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.
`Succeeded` | All containers in the Pod have terminated in success, and will not be restarted.
`Failed` | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system.
`Unknown` | For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.

If a node dies or is disconnected from the rest of the cluster, Kubernetes
applies a policy for setting the `phase` of all Pods on the lost node to Failed.
-->
## Pod 的人生阶段

在 Pod 的 `status` 字段是一个 [PodStatus](https://kubernetes.io/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
对象, 上面有一个 `phase` 字段.

Pod 的人生阶段是对 {{< glossary_tooltip term_id="pod" >}} 生命周期的调度总结.
Pod 的人生阶段并不是对容器或 Pod 状态的容易理解的总结, 也不是一个容易理解的状态机.

Pod 的人生阶段的数量与意义与其值都是很有限的. 除了以下对各阶段的说明, Pod 不会有其它的阶段.
以下为 `phase` 字段的可能值:

字段值 | 描述
:-----|:-----------
`Pending` | Pod 已经被集群确立， 但是其中的一个或多个容器还没有完成配置并准备就绪。 包括 Pod 等调度的时间和从网上下载容器镜像的时间
`Running` | Pod 已经在节点上，并且其中的所有容器已经完成创建，至少有一个容器正在运行，或在启动或重启的过程中
`Succeeded` | Pod 中所有的容器都已经成功运行完成并终止，并且不会再被重启
`Failed` | Pod 中所有的容器被终止，且至少有一个容器是因为失败而被终止的。 容器失败的原因可能是因返回非零而退出或被系统终止
`Unknown` | 因为某些原因导至无法获取 Pod 的状态。 这个阶段一般是因为与 Pod 所在的节点无法通信。

如果一个节点挂了或者与集群失联， k8s 会执行一个策略，让节点上所有的 Pod 的 `phase` 字段设置为 `Failed`。
<!--
## Container states

As well as the [phase](#pod-phase) of the Pod overall, Kubernetes tracks the state of
each container inside a Pod. You can use
[container lifecycle hooks](/k8sDocs/concepts/containers/container-lifecycle-hooks/) to
trigger events to run at certain points in a container's lifecycle.

Once the {{< glossary_tooltip text="scheduler" term_id="kube-scheduler" >}}
assigns a Pod to a Node, the kubelet starts creating containers for that Pod
using a {{< glossary_tooltip text="container runtime" term_id="container-runtime" >}}.
There are three possible container states: `Waiting`, `Running`, and `Terminated`.

To the check state of a Pod's containers, you can use
`kubectl describe pod <name-of-pod>`. The output shows the state for each container
within that Pod.

Each state has a specific meaning:

 -->
## 容器的状态

与 Pod 存在几个[阶段](#pod-phase)一个样，k8s 也会跟踪 Pod 内的容器的状态。 用户可以通过
[容器的生命周期钩子](../../../containers/container-lifecycle-hooks/)
来以容器生命周期事件来触发一些需要工作的运行

当一个 {{< glossary_tooltip term_id="kube-scheduler" >}}
将一个 Pod 调度到一个节点时， kubelet 就会通过 {{< glossary_tooltip term_id="container-runtime" >}}
为该 Pod 创建对应的容器。 容器可能存在三种状态 `Waiting`, `Running`, `Terminated`.

用户可以通过命令 `kubectl describe pod <name-of-pod>` 查看 Pod 中容器的状态。
命令输出结果会包含其中所有容器的状态。
接下来介绍每一种状的具体含义

### `Waiting` {#container-state-waiting}
<!--
If a container is not in either the `Running` or `Terminated` state, it `Waiting`.
A container in the `Waiting` state is still running the operations it requires in
order to complete start up: for example, pulling the container image from a container
image registry, or applying {{< glossary_tooltip text="Secret" term_id="secret" >}}
data.
When you use `kubectl` to query a Pod with a container that is `Waiting`, you also see
a Reason field to summarize why the container is in that state.
 -->
当一个容器的状不是 `Running` 或 `Terminated` 就是 `Waiting`。当一个容器状态为 `Waiting`
表示容器正在进行启动需要的前置操作: 如， 从镜像仓库拉取容器所需要的镜像， 或配置
{{< glossary_tooltip text="Secret" term_id="secret" >}} 数据。

当 {{< glossary_tooltip term_id="kubelet" >}} 查询到Pod中的状是 `Waiting`， 同时也会看到
一个 `Reason` 字段，其值是对容器保持在该状态原因的总结

### `Running` {#container-state-running}
<!--
The `Running` status indicates that a container is executing without issues. If there
was a `postStart` hook configured, it has already executed and executed. When you use
`kubectl` to query a Pod with a container that is `Running`, you also see information
about when the container entered the `Running` state.

 -->
`Running` 状态表示容器正在欢快地运行，没啥毛病。 如果配置了钩子 `postStart`， 这个钩子的处理器也
已经执行而且成功完成。 当使用 `kubectl` 查询容器为 `Running` 的 Pod 时，同时可以看到容器进入
`Running` 状态的时长。

### `Terminated` {#container-state-terminated}
<!--
A container in the `Terminated` state has begin execution and has then either run to
completion or has failed for some reason. When you use `kubectl` to query a Pod with
a container that is `Terminated`, you see a reason, and exit code, and the start and
finish time for that container's period of execution.

If a container has a `preStop` hook configured, that runs before the container enters
the `Terminated` state.
 -->
当一个容器状态为 `Terminated` 时，表示任务已经执行了，要么执行完成，要么因为某些原因失败了。
当使用 `kubectl` 查看容器为 `Terminated` 状态的 Pod 时， 可以看到原因和退出码， 还有
容器内任务执行的开始和结束时间。

如果一个容器配置了钩子 `preStop`， 那么钩子对应的处理器会在容器进行`Terminated` 状态之前执行。
<!--
## Container restart policy {#restart-policy}

The `spec` of a Pod has a `restartPolicy` field with possible values Always, OnFailure,
and Never. The default value is Always.

The `restartPolicy` applies to all containers in the Pod. `restartPolicy` only
refers to restarts of the containers by the kubelet on the same node. After containers
in a Pod exit, the kubelet restarts them with an exponential back-off delay (10s, 20s,
40s, …), that is capped at five minutes. Once a container has executed with no problems
for 10 minutes without any problems, the kubelet resets the restart backoff timer for
that container.
 -->
## 容器的重启策略 {#restart-policy}

在 Pod 的 `spec` 子对象上有一个 `restartPolicy` 字段，可能的值有 `Always`, `OnFailure`, `Never`
默认为 `Always`。

`restartPolicy` 适用于 Pod 中的所有容器。 `restartPolicy` 只能让一个节点上的 {{< glossary_tooltip term_id="kubelet" >}}
来重启其上的容器。 当 Pod 中的容器退出后， {{< glossary_tooltip term_id="kubelet" >}}
会以指数延迟(10s, 20s, 40s, …)补偿机制来重启容器，延迟时间最长为 5 分钟。 当一个容器正常运行 10 分钟
后，  {{< glossary_tooltip term_id="kubelet" >}} 才会重置该容器的补偿时钟。
<!--
## Pod conditions

A Pod has a PodStatus, which has an array of
[PodConditions](https://kubernetes.io/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core)
through which the Pod has or has not passed:

* `PodScheduled`: the Pod has been scheduled to a node.
* `ContainersReady`: all containers in the Pod are ready.
* `Initialized`: all [init containers](/k8sDocs/concepts/workloads/pods/init-containers/)
  have started successfully.
* `Ready`: the Pod is able to serve requests and should be added to the load
  balancing pools of all matching Services.

Field name           | Description
:--------------------|:-----------
`type`               | Name of this Pod condition.
`status`             | Indicates whether that condition is applicable, with possible values "`True`", "`False`", or "`Unknown`".
`lastProbeTime`      | Timestamp of when the Pod condition was last probed.
`lastTransitionTime` | Timestamp for when the Pod last transitioned from one status to another.
`reason`             | Machine-readable, UpperCamelCase text indicating the reason for the condition's last transition.
`message`            | Human-readable message indicating details about the last status transition.
 -->
## Pod 的就绪条件
在 Pod 上面有一个 `PodStatus` 子对象，其中包含一个
[PodConditions](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core)
的数组。表示这个 Pod 有没有通过这些条件， 具体如下:

* `PodScheduled`: Pod 已经被调度到节点上.
* `ContainersReady`: Pod 中所有的容器都已经就绪.
* `Initialized`: 所有 [初始化容器](../init-containers/)
  都启动成功.
* `Ready`: Pod 已经能够处理请求，应该被加入对应 Service 的负载均衡池中。

字段名称              | 说明
:--------------------|:-----------
`type`               | 这个 Pod 条件的名称
`status`             | 表示这个条件的达成情况， 可能的值有 "`True`", "`False`", 或 "`Unknown`".
`lastProbeTime`      | 该条件上次探测的时间戳
`lastTransitionTime` | 该条件的值最近发生变更的时间戳
`reason`             | 机器可读, 大写驼峰的文本，说明最近一次值变化的原因
`message`            | 人类可读的消息，详细说明最近一次值变化的原因

<!--
### Pod readiness {#pod-readiness-gate}

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

Your application can inject extra feedback or signals into PodStatus:
_Pod readiness_. To use this, set `readinessGates` in the Pod's `spec` to
specify a list of additional conditions that the kubelet evaluates for Pod readiness.

Readiness gates are determined by the current state of `status.condition`
fields for the Pod. If Kubernetes cannot find such a condition in the
`status.conditions` field of a Pod, the status of the condition
is defaulted to "`False`".

Here is an example:

```yaml
kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready                              # a built in PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"        # an extra PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```

The Pod conditions you add must have names that meet the Kubernetes [label key format](/k8sDocs/concepts/overview/working-with-objects/labels/#syntax-and-character-set).
  -->

### Pod 就绪阀 {#pod-readiness-gate}
用户可以向应用中注入额外的反馈或信号到 `PodStatus`: Pod readiness. 要使用该特性，需要在 Pod 的 `spec` 子对象上设置 `readinessGates`，定义追加额外的就绪条件到 k8s 检测 Pod 就绪条件列表中。

就绪阀由 Pod 当前 `status.condition` 的状态决定。 如果 k8s 不能在 Pod 的 `status.condition` 中找到该条件， 条件的默认值为 `False`
以下为示例:
```yaml
kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready                              # 内置的 PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"        # 外挂的 PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```
用户添加的条件在命名是需要符合 k8s 的 [标签命名格式](/k8sDocs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)

<!--
### Status for Pod readiness {#pod-readiness-status}

The `kubectl patch` command does not support patching object status.
To set these `status.conditions` for the pod, applications and
{{< glossary_tooltip term_id="operator-pattern" text="operators">}} should use
the `PATCH` action.
You can use a [Kubernetes client library](/k8sDocs/reference/using-api/client-libraries/) to
write code that sets custom Pod conditions for Pod readiness.

For a Pod that uses custom conditions, that Pod is evaluated to be ready **only**
when both the following statements apply:

* All containers in the Pod are ready.
* All conditions specified in `readinessGates` are `True`.

When a Pod's containers are Ready but at least one custom condition is missing or
`False`, the kubelet sets the Pod's [condition](#pod-condition) to `ContainersReady`.
 -->
### Pod 就绪条件的状态 {#pod-readiness-status}

`kubectl patch` 命令不支持对对象状态的修改。 想要对 Pod 的 `status.conditions`， 应用，或其它进行 `PATCH` 的操作
可以通过 [k8s 客户端库](/k8sDocs/reference/using-api/client-libraries/)
写代码的方式来自定义 Pod 就绪条件。

对于使用自定义就绪条件的 Pod， 只有在达成以下条件时才能进入就绪状态:
- Pod 中所有的容器都已经就绪
- 所有有容器配置的 `readinessGates` 的值都为 `True`

当一个 Pod 中所有容器已经就绪，但至少有一个自定义条件不存在或值为 `False`，
kubelet 会将 Pod 的[就绪条件](#pod-condition)值设置为 `ContainersReady`
<!--
## Container probes

A [Probe](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#probe-v1-core) is a diagnostic
performed periodically by the [kubelet](/k8sDocs/admin/kubelet/)
on a Container. To perform a diagnostic,
the kubelet calls a
[Handler](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#handler-v1-core) implemented by
the container. There are three types of handlers:

* [ExecAction](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#execaction-v1-core):
  Executes a specified command inside the container. The diagnostic
  is considered successful if the command exits with a status code of 0.

* [TCPSocketAction](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#tcpsocketaction-v1-core):
  Performs a TCP check against the Pod's IP address on
  a specified port. The diagnostic is considered successful if the port is open.

* [HTTPGetAction](/k8sDocs/reference/generated/kubernetes-api/{{< param "version" >}}/#httpgetaction-v1-core):
  Performs an HTTP `GET` request against the Pod's IP
  address on a specified port and path. The diagnostic is considered successful
  if the response has a status code greater than or equal to 200 and less than 400.

Each probe has one of three results:

* `Success`: The container passed the diagnostic.
* `Failure`: The container failed the diagnostic.
* `Unknown`: The diagnostic failed, so no action should be taken.

The kubelet can optionally perform and react to three kinds of probes on running
containers:

* `livenessProbe`: Indicates whether the container is running. If
   the liveness probe fails, the kubelet kills the container, and the container
   is subjected to its [restart policy](#restart-policy). If a Container does not
   provide a liveness probe, the default state is `Success`.

* `readinessProbe`: Indicates whether the container is ready to respond to requests.
   If the readiness probe fails, the endpoints controller removes the Pod's IP
   address from the endpoints of all Services that match the Pod. The default
   state of readiness before the initial delay is `Failure`. If a Container does
   not provide a readiness probe, the default state is `Success`.

* `startupProbe`: Indicates whether the application within the container is started.
   All other probes are disabled if a startup probe is provided, until it succeeds.
   If the startup probe fails, the kubelet kills the container, and the container
   is subjected to its [restart policy](#restart-policy). If a Container does not
   provide a startup probe, the default state is `Success`.

For more information about how to set up a liveness, readiness, or startup probe,
see [Configure Liveness, Readiness and Startup Probes](/k8sDocs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).
 -->

## 容器探针

[探针](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#probe-v1-core)
就是由 [kubelet](/k8sDocs/admin/kubelet/) 定时对容器进行诊断操作，
诊断操作则是由 kubelet 调用一个由容器实现的
[处理器](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#handler-v1-core)。
有以下三种类型处理器:
* [ExecAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#execaction-v1-core):
  在容器内执行一个指定命令. 如果命令执行结束代码为 0 则表示诊断结果为成功

* [TCPSocketAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#tcpsocketaction-v1-core):
  向指定 IP 地址和端口发起 TCP 请求。 如果成功打开端口，表示诊断结果为成功
* [HTTPGetAction](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#httpgetaction-v1-core):
  向指定IP 地址，端口和路径发起 HTTP `GET` 请求。如果响应码在 200 <= code < 400
  表示诊断结果为成功

以上探针的结果的值可能为以下任意一个:

* `Success`: 容器通过了诊断.
* `Failure`: 容器没有通过诊断.
* `Unknown`: 诊断过程失败，不执行任何操作.

kubelet 可以选择是否对容器中以下探针的结果作出相应的操作：

* `livenessProbe`(存活探针): 指示容器是否正在运行.

   如果存活探针的诊断结果为未通过， 则 kubelet 会杀掉这个容器，而后容器操作则由其 [重启策略](#restart-policy)决定。
   如果容器没有配置存活探针，则默认状态为成功。

* `readinessProbe`(就绪探针): 指示容器是否可以响应请求
   如果就绪探针的诊断结果为未通过， 则 {{<glossary_tooltip term_id="endpoint">}} 控制器
   就会把该 Pod 从所有配置该 Pod 的 {{<glossary_tooltip term_id="service">}} 的
   {{<glossary_tooltip term_id="endpoint">}} 中移出。
   如果容器没有配置就绪探针，则默认状态为成功。

* `startupProbe`(启动探针): 指示容器内的应用是否启动
   如果配置了启动探针，除非启动探针诊断结果为通过，否则所有其它探针都不会工作。 如果启动探针的诊断
   结果为未通过，则 kubelet 会杀掉是这个容器， 而后容器操作则由其 [重启策略](#restart-policy)决定。
   如果容器没有配置启动探针，则默认状态为成功。

了角更多关于如何配置 存活探针，就绪探针，启动探针，见
[存活探针，就绪探针，启动探针配置](/k8sDocs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).
<!--
### When should you use a liveness probe?

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

If the process in your container is able to crash on its own whenever it
encounters an issue or becomes unhealthy, you do not necessarily need a liveness
probe; the kubelet will automatically perform the correct action in accordance
with the Pod's `restartPolicy`.

If you'd like your container to be killed and restarted if a probe fails, then
specify a liveness probe, and specify a `restartPolicy` of Always or OnFailure.
  -->
### 为啥需要用就绪(readiness)探针?

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

如果应用内的进程在出毛病或变得不健康是就能够自己挂掉，那么就是需要配置生存探针， kubelet 会
自动根据 Pod 的 `restartPolicy` 正确处理这些问题。

如果用户需要在探针诊断结果为未通过时杀掉容器并重启，就可以配置一个存活探针，并将 `restartPolicy`
的值设置为 `Always` 或 `OnFailure`.
<!--
### When should you use a readiness probe?

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

If you'd like to start sending traffic to a Pod only when a probe succeeds,
specify a readiness probe. In this case, the readiness probe might be the same
as the liveness probe, but the existence of the readiness probe in the spec means
that the Pod will start without receiving any traffic and only start receiving
traffic after the probe starts succeeding.
If your container needs to work on loading large data, configuration files, or
migrations during startup, specify a readiness probe.

If you want your container to be able to take itself down for maintenance, you
can specify a readiness probe that checks an endpoint specific to readiness that
is different from the liveness probe.

{{< note >}}
If you just want to be able to drain requests when the Pod is deleted, you do not
necessarily need a readiness probe; on deletion, the Pod automatically puts itself
into an unready state regardless of whether the readiness probe exists.
The Pod remains in the unready state while it waits for the containers in the Pod
to stop.
{{< /note >}}
 -->
### 啥时候应该用就绪(readiness)探针?

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

如果用户期望仅在探针诊断状态为通过时才向 Pod 调度流量，此时应该配置就绪探针。
在这种情况下，就绪探针可能与存活探针区别不大， 但就绪探针存在的意义在于 Pod 不会在就绪探针通过之前
接收到任何流量，只有在通过之后才会开始接收流量。
如果用户容器在启动时需要加载大量数据，配置文件，或迁移数据，这时就需要配置就绪探针。

如果用户期望在需要维护容器可以自挂东南枝， 就可以设置一个与存活探针不同的探测接口作为就绪探针。

{{< note >}}
如果用户期望仅在 Pod 被删除是不接收流量， 则不需要配置就绪探针。 在 Pod 被删除时，无论有没有就绪探针都自动将其状态
设置为未就绪状态。 Pod 在等待其中容器正常停止的过程中状态一直也都是未就绪。
{{< /note >}}
<!--
### When should you use a startup probe?

{{< feature-state for_k8s_version="v1.16" state="alpha" >}}

Startup probes are useful for Pods that have containers that take a long time to
come into service. Rather than set a long liveness interval, you can configure
a separate configuration for probing the container as it starts up, allowing
a time longer than the liveness interval would allow.

If your container usually starts in more than
`initialDelaySeconds + failureThreshold × periodSeconds`, you should specify a
startup probe that checks the same endpoint as the liveness probe. The default for
`periodSeconds` is 30s. You should then set its `failureThreshold` high enough to
allow the container to start, without changing the default values of the liveness
probe. This helps to protect against deadlocks.
 -->
### 啥时候应该用启动探针？

{{< feature-state for_k8s_version="v1.16" state="alpha" >}}

启动探针对于那些其中容器需要花费很长时间才能提供服务的 Pod 是相当有用的。并不需要配置一个长时间间隔的存活探针，
只需要配置一个独立的配置的探测容器的启动。而这样可以允许比存活探针时间间隔更长的时间来等待容器启动。

如果用户容器通过启动时间大于 `initialDelaySeconds + failureThreshold × periodSeconds`，
就应该配置一个与存活探针检查点相同的启动探针。 `periodSeconds` 默认为 30秒。 所以需要设置一个
足够大的 `failureThreshold` 值，以保证容器能够有足够的时间启动， 而不需要修改存活探针的默认配置。
这也能避免出现死锁。
<!--
## Termination of Pods {#pod-termination}

Because Pods represent processes running on nodes in the cluster, it is important to
allow those processes to gracefully terminate when they are no longer needed (rather
than being abruptly stopped with a `KILL` signal and having no chance to clean up).

The design aim is for you to be able to request deletion and know when processes
terminate, but also be able to ensure that deletes eventually complete.
When you request deletion of a Pod, the cluster records and tracks the intended grace period
before the Pod is allowed to be forcefully killed. With that forceful shutdown tracking in
place, the {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} attempts graceful
shutdown.

Typically, the container runtime sends a a TERM signal is sent to the main process in each
container. Once the grace period has expired, the KILL signal is sent to any remainig
processes, and the Pod is then deleted from the
{{< glossary_tooltip text="API Server" term_id="kube-apiserver" >}}. If the kubelet or the
container runtime's management service is restarted while waiting for processes to terminate, the
cluster retries from the start including the full original grace period.

An example flow:

1. You use the `kubectl` tool to manually delete a specific Pod, with the default grace period
   (30 seconds).
1. The Pod in the API server is updated with the time beyond which the Pod is considered "dead"
   along with the grace period.  
   If you use `kubectl describe` to check on the Pod you're deleting, that Pod shows up as
   "Terminating".  
   On the node where the Pod is running: as soon as the kubelet sees that a Pod has been marked
   as terminating (a graceful shutdown duration has been set), the kubelet begins the local Pod
   shutdown process.
   1. If one of the Pod's containers has defined a `preStop`
      [hook](/k8sDocs/concepts/containers/container-lifecycle-hooks/#hook-details), the kubelet
      runs that hook inside of the container. If the `preStop` hook is still running after the
      grace period expires, the kubelet requests a small, one-off grace period extension of 2
      seconds.
      {{< note >}}
      If the `preStop` hook needs longer to complete than the default grace period allows,
      you must modify `terminationGracePeriodSeconds` to suit this.
      {{< /note >}}
   1. The kubelet triggers the container runtime to send a TERM signal to process 1 inside each
      container.
      {{< note >}}
      The containers in the Pod receive the TERM signal at different times and in an arbitrary
      order. If the order of shutdowns matters, consider using a `preStop` hook to synchronize.
      {{< /note >}}
1. At the same time as the kubelet is starting graceful shutdown, the control plane removes that
   shutting-down Pod from Endpoints (and, if enabled, EndpointSlice) objects where these represent
   a {{< glossary_tooltip term_id="service" text="Service" >}} with a configured
   {{< glossary_tooltip text="selector" term_id="selector" >}}.
   {{< glossary_tooltip text="ReplicaSets" term_id="replica-set" >}} and other workload resources
   no longer treat the shutting-down Pod as a valid, in-service replica. Pods that shut down slowly
   cannot continue to serve traffic as load balancers (like the service proxy) remove the Pod from
   the list of endpoints as soon as the termination grace period _begins_.
1. When the grace period expires, the kubelet triggers forcible shutdown. The container runtime sends
   `SIGKILL` to any processes still running in any container in the Pod.
   The kubelet also cleans up a hidden `pause` container if that container runtime uses one.
1. The kubelet triggers forcible removal of Pod object from the API server, by setting grace period
   to 0 (immediate deletion).  
1. The API server deletes the Pod's API object, which is then no longer visible from any client.
 -->
## Pod 的终结过程 {#pod-termination}

因为 Pod 代表运行在集群节点上的一系列进程， 而要让这些进程在不需要时能够死得瞑目(而不是通过 KILL 信号突然被停止，连收尾的机会都没得)。

在设计上旨在用户能够在发起删除请求并能够知晓啥时候进程终止， 但最终还要保证删除操作最终需要完成。
当用户发起删除一个 Pod 的请求， 集群会在预期的时间内跟踪和记录，如果超过这个时间则会强制终止 Pod 的进程。
在强制终止之前， kubelet 都会尝试平滑关闭。

通常情况下，{{< glossary_tooltip term_id="container-runtime">}} 会向每个容器的主进程
发送一个 `TERM` 信号。如果超过预期时间，再和仍然存在的进程发送 `KILL` 信号， 然后从
{{< glossary_tooltip term_id="kube-apiserver">}} 中删除该 Pod 对象。 如果在这个等待过程中
kubelet 或 {{< glossary_tooltip term_id="container-runtime">}} 发生重启， 集群会尝试对该删除操作
重启开启计时。

以下为一个示例：

1. 用户使用 kubectl 命令手动删除一个 Pod，使用默认的预期时间(30s).
2. 从命令执行开始到预期时间内 {{< glossary_tooltip term_id="kube-apiserver">}} 中的
  Pod 对象就会更新，并标记为已经挂了。 如果使用 `kubectl describe` 查看正在删除的 Pod，
  看到它的状态应该是 `Terminating`。 在 Pod 所在的节点上： 当 kubelet 看到 Pod 被标记为终止时(添加一个平滑关闭标记)
  kubelet 就开始并本地的 Pod 的进程。
    1. 如果 Pod 中有任意容器配置了[钩子](/k8sDocs/concepts/containers/container-lifecycle-hooks/#hook-details) `preStop`,
      kubelet 会在对应容器中执行这个钩子。 如果在预期时间到达时 `preStop` 钩子仍在运行，则 kubelet 一次性多给 2 秒。
      {{< note >}}
      如果 `preStop` 需要比默认的预期时间更长的时间，则需要设置一个合适的 `terminationGracePeriodSeconds` 值
      {{< /note >}}
    2. kubelet 触发 {{< glossary_tooltip term_id="container-runtime">}} 向 Pod 中的每个容器的 1 号进程
      发送 TERM 信号
      {{< note >}}
      Pod 中的容器可能会以不同的顺序和时间接收到 TERM 信号， 如果需要进行有序关闭，考虑使用 `preStop` 钩子来实现同步锁
      {{< /note >}}
3. 在 kubelet 开始平滑关闭 Pod 的进程的同时， 控制中心将正在删除的 Pod 从 对应配置选择的 {{< glossary_tooltip term_id="service">}}
    所代表的 {{< glossary_tooltip term_id="endpoint">}} (如果开启也可能是 EndpointSlice)中移出。
    {{< glossary_tooltip term_id="replica-set">}} 和其它的工作负载资源都会将该Pod认作是失效的，对于那些半天关不掉又不能提供服务的Pod
    负载均衡(如 service proxy)会在删除预期时间开始时就从 {{< glossary_tooltip term_id="endpoint">}} 列表中移出。
4. 当预期时间用完后，就会触发 kubelet 强制删除。 {{< glossary_tooltip term_id="container-runtime">}}
  会向所有剩余的进程发送 `SIGKILL` 信号。 如果容器用到了隐藏的 `pause` 容器 kubelet 也会一起清理
5. kubectl 通过将预期时间设置为0(立马删除)触发从 {{< glossary_tooltip term_id="kube-apiserver">}}
  中强制删除 Pod 对象。
6. {{< glossary_tooltip term_id="kube-apiserver">}} 会在 Pod 对象对所有客户端不可见时，将其删除

<!--
### Forced Pod termination {#pod-termination-forced}

{{< caution >}}
Forced deletions can be potentially disruptiove for some workloads and their Pods.
{{< /caution >}}

By default, all deletes are graceful within 30 seconds. The `kubectl delete` command supports
the `--grace-period=<seconds>` option which allows you to override the default and specify your
own value.

Setting the grace period to `0` forcibly and immediately deletes the Pod from the API
server. If the pod was still running on a node, that forcible deletion triggers the kubelet to
begin immediate cleanup.

{{< note >}}
You must specify an additional flag `--force` along with `--grace-period=0` in order to perform force deletions.
{{< /note >}}

When a force deletion is performed, the API server does not wait for confirmation
from the kubelet that the Pod has been terminated on the node it was running on. It
removes the Pod in the API immediately so a new Pod can be created with the same
name. On the node, Pods that are set to terminate immediately will still be given
a small grace period before being force killed.

If you need to force-delete Pods that are part of a StatefulSet, refer to the task
documentation for
[deleting Pods from a StatefulSet](/k8sDocs/tasks/run-application/force-delete-stateful-set-pod/).
 -->
### Pod 的强制删除 {#pod-termination-forced}
{{< caution >}}
强制删除存在破坏一些工作负载或其 Pod的潜在风险。
{{< /caution >}}

在默认情况下，所有的删除操作的预期时间都是 30 秒，`kubectl delete` 命令支持通过
`--grace-period=<seconds>` 选择来自定义预期时间。

将预期时间设置为 0， 会强制立马从 {{< glossary_tooltip term_id="kube-apiserver">}} 中删除 Pod 对象。
如果 Pod 仍然运行在某个节点上， 这种强制删除会触发 kubelet 开始立即清理。

{{< note >}}
需要同时使用 `--force` 和 `--grace-period=0` 在能实现强制删除。
{{< /note >}}

当一个强制删除被执行时， {{< glossary_tooltip term_id="kube-apiserver">}} 不会等待
Pod 所在节点的的 kubelet 确认 Pod 已经被终止。 只是立马删除 Pod 对象，这时可以马上创建一个同名的新 Pod
而在节点上，被设置为立马终止的 Pod 在被强制杀死前也会给予一小会时间，以期可能平滑关闭。

如果用户需要强制删除一个 StatefulSet 的 Pod，
请见 [删除一个属于StatefulSet的Pod](/k8sDocs/tasks/run-application/force-delete-stateful-set-pod/).
<!--  
### Garbage collection of failed Pods {#pod-garbage-collection}

For failed Pods, the API objects remain in the cluster's API until a human or
{{< glossary_tooltip term_id="controller" text="controller" >}} process
explicitly removes them.

The control plane cleans up terminated Pods (with a phase of `Succeeded` or
`Failed`), when the number of Pods exceeds the configured threshold
(determined by `terminated-pod-gc-threshold` in the kube-controller-manager).
This avoids a resource leak as Pods are created and terminated over time.
-->
### 对失效 Pod 的垃圾回收 {#pod-garbage-collection}

对于失效的 Pod, 其对应会存在于集群 {{< glossary_tooltip term_id="kube-apiserver">}} 中，
直至人工或 {{< glossary_tooltip term_id="controller" text="controller" >}} 明确的删除它们

控制中心清理终止的Pod (阶段的 `Succeeded` 或 `Failed`)， 如 Pod 的数量超过配置的阈值(由 kube-controller-manager 中的`terminated-pod-gc-threshold`配置决定)
这会在长时间 Pod 创建和终止的过程中避免出现资源泄漏。


## {{% heading "whatsnext" %}}

* 实践
  [attaching handlers to Container lifecycle events](/k8sDocs/tasks/configure-pod-container/attach-handler-lifecycle-event/).

* 这溃
  [configuring Liveness, Readiness and Startup Probes](/k8sDocs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).

* 了解[container lifecycle hooks](/k8sDocs/concepts/containers/container-lifecycle-hooks/).

* 更多关于 Pod / Container 状态的 API, 见 [PodStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core),
[ContainerStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core).
