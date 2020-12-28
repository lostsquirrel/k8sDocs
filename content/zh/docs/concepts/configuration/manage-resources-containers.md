---
title: 管理容器资源
content_type: concept
weight: 40
---
<!--
---
title: Managing Resources for Containers
content_type: concept
weight: 40
feature:
  title: Automatic bin packing
  description: >
    Automatically places containers based on their resource requirements and other constraints, while not sacrificing availability. Mix critical and best-effort workloads in order to drive up utilization and save even more resources.
---
 -->
<!-- overview -->
<!--
When you specify a {{< glossary_tooltip term_id="pod" >}}, you can optionally specify how
much of each resource a {{< glossary_tooltip text="Container" term_id="container" >}} needs.
The most common resources to specify are CPU and memory (RAM); there are others.

When you specify the resource _request_ for Containers in a Pod, the scheduler uses this
information to decide which node to place the Pod on. When you specify a resource _limit_
for a Container, the kubelet enforces those limits so that the running container is not
allowed to use more of that resource than the limit you set. The kubelet also reserves
at least the _request_ amount of that system resource specifically for that container
to use.
 -->

在配置一个
{{< glossary_tooltip term_id="pod" >}}
时，用户可以选择对每个
{{< glossary_tooltip text="Container" term_id="container" >}}
设置其所使用的资源。
最常设置的资源是 CPU 和 内存 (RAM); 但还是有其他的。

当用户为一个 Pod 中的容器设置资源 _请求_ ，调度器会使用这些信息来决定将 Pod 放到哪个节点上。
当为容器设置资源 _限制_ 时， kubelet 会执行这些限制以保证容器在运行时使用的资源不会超过这个设置
的资源限制。 kubelet 也会为这些容器保留资源 _请求_ 所设置的的资源的数量给予容器使用。


<!-- body -->
<!--
## Requests and limits

If the node where a Pod is running has enough of a resource available, it's possible (and
allowed) for a container to use more resource than its `request` for that resource specifies.
However, a container is not allowed to use more than its resource `limit`.

For example, if you set a `memory` request of 256 MiB for a container, and that container is in
a Pod scheduled to a Node with 8GiB of memory and no other Pods, then the container can try to use
more RAM.

If you set a `memory` limit of 4GiB for that Container, the kubelet (and
{{< glossary_tooltip text="container runtime" term_id="container-runtime" >}}) enforce the limit.
The runtime prevents the container from using more than the configured resource limit. For example:
when a process in the container tries to consume more than the allowed amount of memory,
the system kernel terminates the process that attempted the allocation, with an out of memory
(OOM) error.

Limits can be implemented either reactively (the system intervenes once it sees a violation)
or by enforcement (the system prevents the container from ever exceeding the limit). Different
runtimes can have different ways to implement the same restrictions.

{{< note >}}
If a Container specifies its own memory limit, but does not specify a memory request, Kubernetes
automatically assigns a memory request that matches the limit. Similarly, if a Container specifies its own
CPU limit, but does not specify a CPU request, Kubernetes automatically assigns a CPU request that matches
the limit.
{{< /note >}}
 -->

## 请求与限制 {#requests-and-limits}

如果 Pod 运行的节点上的某种资源足够， 这就使得这个容器可能(也允许)使用比 `request` 所设置的资源
更多的资源，但容器不能使用比 `limit` 设置的资源的限制。

例如， 如果设置容器的 `memory` 资源请求为 256 MiB，然后容器所在的 Pod 被调度到一个有 8GiB
内存的节点上并且这个节点上没有其它的 Pod， 这时容器就可以常用使用更多的 RAM.

如果设置了容器 `memory` 限制为 4GiB， kubelet
(和 {{< glossary_tooltip term_id="container-runtime" >}})
会执行这个限制。容器运行环境会防止容器使用超过容器资源限制所配置的资源数额。 例如，当容器中的一
个进行尝试使用超过允许的内存数量，系统内核就会终止这个尝试申请的进程，错误信息的内存不足(OOM).

这些限制的实现可以是反应式的(当发现起限时系统干涉)或通过强制(系统防止容器使用超过限制资源)。
不同容器运行环境可能以不同的方式实现同样的限制。

{{< note >}}
如果一个容器设置内存限制(`limit`)，但没有设置内存请求(`request`)，k8s 会自动为其分配一个与限制数额相同的请求。类似地
如果容器指定 CPU 限制(`limit`)，但没设置 CPU 请求(`request`)， k8s 会自动为其分配一个与限制数额相同的请求。
{{< /note >}}
<!--
## Resource types

*CPU* and *memory* are each a *resource type*. A resource type has a base unit.
CPU represents compute processing and is specified in units of [Kubernetes CPUs](#meaning-of-cpu).
Memory is specified in units of bytes.
If you're using Kubernetes v1.14 or newer, you can specify _huge page_ resources.
Huge pages are a Linux-specific feature where the node kernel allocates blocks of memory
that are much larger than the default page size.

For example, on a system where the default page size is 4KiB, you could specify a limit,
`hugepages-2Mi: 80Mi`. If the container tries allocating over 40 2MiB huge pages (a
total of 80 MiB), that allocation fails.

{{< note >}}
You cannot overcommit `hugepages-*` resources.
This is different from the `memory` and `cpu` resources.
{{< /note >}}

CPU and memory are collectively referred to as *compute resources*, or just
*resources*. Compute
resources are measurable quantities that can be requested, allocated, and
consumed. They are distinct from
[API resources](/docs/concepts/overview/kubernetes-api/). API resources, such as Pods and
[Services](/docs/concepts/services-networking/service/) are objects that can be read and modified
through the Kubernetes API server.
 -->

## 资源类型 {#resource-types}

*CPU* 和 *memory* 都是 *资源类型* 的一种。 每种资源类型都有一个基础单位。CPU 代表计算处理
并以
[Kubernetes CPUs](#meaning-of-cpu)
为设置的基础单位。 内存是以字节为单位来设置的。 如果使用的是 k8s v1.14+, 可以设置 _huge page_ 资源。
Huge page 是一个 Linux 的特性，当节点内存分配内存块时可以多默认的 page size 大很多

例如，在一个系统中默认的 page size 是 4KiB, 设置了一个限制为 `hugepages-2Mi: 80Mi`.
如果容器尝试分配了超出 40 个 2Mib 的 huge page(总共就是 80 Mib), 这个分配就会失败。

{{< note >}}
用户并不能过量使用 `hugepages-*` 资源。 这与 `memory` 与 `cpu` 资源不同。
{{< /note >}}

CPU 和 内存都可以被认为是 *计算资源*, 或者直接称为 *资源*。 计算资源作为可以请求，分配，和使用
的可量化资源。 他们与
[API 资源](/k8sDocs/docs/concepts/overview/kubernetes-api/) 不同的。 API 资源，如
Pod 和
[Services](/k8sDocs/docs/concepts/services-networking/service/)
是可以通过 k8s API 服务读取和修改的对象。

<!--
## Resource requests and limits of Pod and Container

Each Container of a Pod can specify one or more of the following:

* `spec.containers[].resources.limits.cpu`
* `spec.containers[].resources.limits.memory`
* `spec.containers[].resources.limits.hugepages-<size>`
* `spec.containers[].resources.requests.cpu`
* `spec.containers[].resources.requests.memory`
* `spec.containers[].resources.requests.hugepages-<size>`

Although requests and limits can only be specified on individual Containers, it
is convenient to talk about Pod resource requests and limits. A
*Pod resource request/limit* for a particular resource type is the sum of the
resource requests/limits of that type for each Container in the Pod.
-->

## Pod 和 容器对资源的请求和限制 {#resource-requests-and-limits-of-pod-and-container}

一个 Pod 中的每个容器都可以指定以下配置中的一个或多个:

* `spec.containers[].resources.limits.cpu`
* `spec.containers[].resources.limits.memory`
* `spec.containers[].resources.limits.hugepages-<size>`
* `spec.containers[].resources.requests.cpu`
* `spec.containers[].resources.requests.memory`
* `spec.containers[].resources.requests.hugepages-<size>`

尽管资源请求和限制只能被设置在独立的容器上，对 Pod 资源的请求和限制也是很方便的。
对于一个特定类别的 *Pod 资源请求/限制*就是 Pod 中所有容器该类型的资源 请求/限制 的总和。

<!--
## Resource units in Kubernetes

### Meaning of CPU

Limits and requests for CPU resources are measured in *cpu* units.
One cpu, in Kubernetes, is equivalent to **1 vCPU/Core** for cloud providers and **1 hyperthread** on bare-metal Intel processors.

Fractional requests are allowed. A Container with
`spec.containers[].resources.requests.cpu` of `0.5` is guaranteed half as much
CPU as one that asks for 1 CPU. The expression `0.1` is equivalent to the
expression `100m`, which can be read as "one hundred millicpu". Some people say
"one hundred millicores", and this is understood to mean the same thing. A
request with a decimal point, like `0.1`, is converted to `100m` by the API, and
precision finer than `1m` is not allowed. For this reason, the form `100m` might
be preferred.

CPU is always requested as an absolute quantity, never as a relative quantity;
0.1 is the same amount of CPU on a single-core, dual-core, or 48-core machine.
 -->

## k8s 中的资源单元 {#resource-units-in-kubernetes}

### CPU 的含义 {#meaning-of-cpu}

对 CPU 资源的请求和限制是以 *cpu* 的单元来计量的。在 k8s 中 1 个单位的 CPU， 与其等效的是
云提供商的 **1 个核心(vCPU/Core)** 和 是裸金属的 Intel 处理器的 **1 个超线程(hyperthread)**

允许使用小数的请求。 一个容器中如果设置 `spec.containers[].resources.requests.cpu` 为
`0.5` 就表示它至少要保证提供给容器二分之一个 CPU 的资源。 `0.1` 等同与 `100m`， 可以被读作
"一百微 CPU". 也有人说的是 "一百微核心"，只要知道这说的是一个意思就行。 配置中如果设置为像
`0.1` 这样的小数，会被 API 转化为 `100m`，不能设置比 `1m` 更小的粒度。所以 `100m` 这种格式
更合用。

CPU 始终是以绝对数量请求的，绝不是相对数量； 0.1 在单核，双核，48 核的机器表示是一样的数量。

<!--
### Meaning of memory

Limits and requests for `memory` are measured in bytes. You can express memory as
a plain integer or as a fixed-point number using one of these suffixes:
E, P, T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi,
Mi, Ki. For example, the following represent roughly the same value:

```shell
128974848, 129e6, 129M, 123Mi
```

Here's an example.
The following Pod has two Containers. Each Container has a request of 0.25 cpu
and 64MiB (2<sup>26</sup> bytes) of memory. Each Container has a limit of 0.5
cpu and 128MiB of memory. You can say the Pod has a request of 0.5 cpu and 128
MiB of memory, and a limit of 1 cpu and 256MiB of memory.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
 -->

### 内存的含义 {#meaning-of-memory}

对 `memory` 的请求和限制是字节来计量的。 可以直接以整数或小数加以下后缀中的一个: E, P, T, G, M, K.
也可以使用 2 的幂的计量单位: Ei, Pi, Ti, Gi, Mi, Ki. 例如，下面几个值的值大约是相等的:

```
128974848, 129e6, 129M, 123Mi
```

下面是一个例子。下面这个 Pod 中有两个容器。 每个容器请求 0.25 CPU 和 64MiB (2<sup>26</sup> 字节)内存。
每个容器限制 0.5 CPU 和 128MiB 内存。 这样就可以说这个 Pod 请求了 0.5 CPU 和 128 MiB 内存，
限制为 1 CPU 和 256MiB 内存。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

<!--
## How Pods with resource requests are scheduled

When you create a Pod, the Kubernetes scheduler selects a node for the Pod to
run on. Each node has a maximum capacity for each of the resource types: the
amount of CPU and memory it can provide for Pods. The scheduler ensures that,
for each resource type, the sum of the resource requests of the scheduled
Containers is less than the capacity of the node. Note that although actual memory
or CPU resource usage on nodes is very low, the scheduler still refuses to place
a Pod on a node if the capacity check fails. This protects against a resource
shortage on a node when resource usage later increases, for example, during a
daily peak in request rate.
 -->

## 带有资源请求的 Pod 是怎么调度的 {#how-pods-with-resource-requests-are-scheduled}

当用户创建一个 Pod 时， k8s 调度器会为 Pod 选择一个节点让它在上面运行。 每个节点都有对每个资源
类型的最大容量: 可以供给 Pod 运行的 CPU 的数量和内存数量。调度器会确保被调度的容器所请求的
各种资源的总和要小于节点对应资源的容量。 即便节点上实际内存或 CPU 资源都很低，调度器依然会在容量
检查失败后拒绝将 Pod 放在这个节点上。 这是为了防止后续资源使用增加而导致资源短缺，例如，每天的
请求峰值。

<!--
## How Pods with resource limits are run

When the kubelet starts a Container of a Pod, it passes the CPU and memory limits
to the container runtime.

When using Docker:

- The `spec.containers[].resources.requests.cpu` is converted to its core value,
  which is potentially fractional, and multiplied by 1024. The greater of this number
  or 2 is used as the value of the
  [`--cpu-shares`](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)
  flag in the `docker run` command.

- The `spec.containers[].resources.limits.cpu` is converted to its millicore value and
  multiplied by 100. The resulting value is the total amount of CPU time that a container can use
  every 100ms. A container cannot use more than its share of CPU time during this interval.

  {{< note >}}
  The default quota period is 100ms. The minimum resolution of CPU quota is 1ms.
  {{</ note >}}

- The `spec.containers[].resources.limits.memory` is converted to an integer, and
  used as the value of the
  [`--memory`](https://docs.docker.com/engine/reference/run/#/user-memory-constraints)
  flag in the `docker run` command.

If a Container exceeds its memory limit, it might be terminated. If it is
restartable, the kubelet will restart it, as with any other type of runtime
failure.

If a Container exceeds its memory request, it is likely that its Pod will
be evicted whenever the node runs out of memory.

A Container might or might not be allowed to exceed its CPU limit for extended
periods of time. However, it will not be killed for excessive CPU usage.

To determine whether a Container cannot be scheduled or is being killed due to
resource limits, see the
[Troubleshooting](#troubleshooting) section.
 -->

## 带有资源限制的 Pod 是怎么运行的 {#how-pods-with-resource-limits-are-run}

当 kubelet 为一个 Pod 启动一个容器时，它会给容器运行时传递 CPU 和 内存的限制。

在使用 Docker 时:

- `spec.containers[].resources.requests.cpu` 会被转化为核心值，一般来说它很可能是个小数，
  并乘上 1024，得出的结果与 2 相比比较大的一个会作为 `docker run` 命令的
  [`--cpu-shares`](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)
  的值。

- `spec.containers[].resources.limits.cpu` 会被转化为微核心值并乘以 100. 得出的结果就是
  每 100ms 中这个容器可以使用的 CPU 时间。容器在每个时间段不能使用超出其限制的 CPU 时间

  {{< note >}}
  默认的配额时间段是 100ms， CPU 配置的最小粒度是 1ms
  {{</ note >}}

- `spec.containers[].resources.limits.memory` 会被转化为一个整数，并作为 `docker run`
  命令的
  [`--memory`](https://docs.docker.com/engine/reference/run/#/user-memory-constraints)
  标记的值。

如果容器超出了内存限制，它就可能被终止。 如果容器是可以重启的，kubelet 就会把它重启了，就像任意
其它类型的运行失败一样。

如果一个容器超出了其请求的内存，它会在节点内存耗尽时被踢出去。

一个容器可能允许也可能不允许超出 CPU 使用时间限制。 但是它不会因为 CPU 使用超限而被杀掉。

决定容器不能被调度或因资源限制而被杀掉的因素见 [故障检查](#troubleshooting) 章节

<!--
### Monitoring compute & memory resource usage

The resource usage of a Pod is reported as part of the Pod status.

If optional [tools for monitoring](/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
are available in your cluster, then Pod resource usage can be retrieved either
from the [Metrics API](/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#the-metrics-api)
directly or from your monitoring tools.
 -->

### 监控计算和内存资源使用 {#monitoring-compute-memory-resource-usage}

Pod 所使用的资源会作为 Pod 状态报告的一部分出现。

如果集群中有可选的
[监控工具](/k8sDocs/docs/tasks/debug-application-cluster/resource-usage-monitoring/)，
Pod 资源使用可以通过
[Metrics API](/k8sDocs/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#the-metrics-api)
或直接通过监控工作中的一种中获取。
<!-- feature gate LocalStorageCapacityIsolation -->
<!--
## Local ephemeral storage


{{< feature-state for_k8s_version="v1.10" state="beta" >}}

Nodes have local ephemeral storage, backed by
locally-attached writeable devices or, sometimes, by RAM.
"Ephemeral" means that there is no long-term guarantee about durability.

Pods use ephemeral local storage for scratch space, caching, and for logs.
The kubelet can provide scratch space to Pods using local ephemeral storage to
mount [`emptyDir`](/docs/concepts/storage/volumes/#emptydir)
 {{< glossary_tooltip term_id="volume" text="volumes" >}} into containers.

The kubelet also uses this kind of storage to hold
[node-level container logs](/docs/concepts/cluster-administration/logging/#logging-at-the-node-level),
container images, and the writable layers of running containers.

{{< caution >}}
If a node fails, the data in its ephemeral storage can be lost.  
Your applications cannot expect any performance SLAs (disk IOPS for example)
from local ephemeral storage.
{{< /caution >}}

As a beta feature, Kubernetes lets you track, reserve and limit the amount
of ephemeral local storage a Pod can consume.
 -->

## 本地临时存储 {#local-ephemeral-storage }

<!-- feature gate LocalStorageCapacityIsolation -->
{{< feature-state for_k8s_version="v1.10" state="beta" >}}

节点有本地临时存储，这些存储由本地挂载可写设备或有时候是 RAM 来提供。"临时" 的意思就是对
数据的持久性没有长期保证。

Pod 可以使用临时本地存储作为暂存空间，缓存或日志。 kubelet 可以使用临时本地存储提供暂存空间
来挂载
[`emptyDir`](/docs/concepts/storage/volumes/#emptydir)
到容器中的
{{< glossary_tooltip term_id="volume" >}}

kubelet 还可以使用这种类型的存储来放置
[节点级别的容器日志](/k8sDocs/docs/concepts/cluster-administration/logging/#logging-at-the-node-level),
容器镜像，运行容器的可写层。

{{< caution >}}

如果节点挂了，临时存储中的数据就可能丢失。并且应用并不能对本地临时存储有任何性能 SLA(例如,磁盘 IOPS)
有任何要求
{{< /caution >}}

作为一个 beta 特性， k8s 可以让用户跟踪，保留，限制一个 Pod 可以使用的临时本地存储。

### Configurations for local ephemeral storage

Kubernetes supports two ways to configure local ephemeral storage on a node:
{{< tabs name="local_storage_configurations" >}}
{{% tab name="Single filesystem" %}}
In this configuration, you place all different kinds of ephemeral local data
(`emptyDir` volumes, writeable layers, container images, logs) into one filesystem.
The most effective way to configure the kubelet means dedicating this filesystem
to Kubernetes (kubelet) data.

The kubelet also writes
[node-level container logs](/docs/concepts/cluster-administration/logging/#logging-at-the-node-level)
and treats these similarly to ephemeral local storage.

The kubelet writes logs to files inside its configured log directory (`/var/log`
by default); and has a base directory for other locally stored data
(`/var/lib/kubelet` by default).

Typically, both `/var/lib/kubelet` and `/var/log` are on the system root filesystem,
and the kubelet is designed with that layout in mind.

Your node can have as many other filesystems, not used for Kubernetes,
as you like.
{{% /tab %}}
{{% tab name="Two filesystems" %}}
You have a filesystem on the node that you're using for ephemeral data that
comes from running Pods: logs, and `emptyDir` volumes. You can use this filesystem
for other data (for example: system logs not related to Kubernetes); it can even
be the root filesystem.

The kubelet also writes
[node-level container logs](/docs/concepts/cluster-administration/logging/#logging-at-the-node-level)
into the first filesystem, and treats these similarly to ephemeral local storage.

You also use a separate filesystem, backed by a different logical storage device.
In this configuration, the directory where you tell the kubelet to place
container image layers and writeable layers is on this second filesystem.

The first filesystem does not hold any image layers or writeable layers.

Your node can have as many other filesystems, not used for Kubernetes,
as you like.
{{% /tab %}}
{{< /tabs >}}

The kubelet can measure how much local storage it is using. It does this provided
that:

- the `LocalStorageCapacityIsolation`
  [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
  is enabled (the feature is on by default), and
- you have set up the node using one of the supported configurations
  for local ephemeral storage.

If you have a different configuration, then the kubelet does not apply resource
limits for ephemeral local storage.

{{< note >}}
The kubelet tracks `tmpfs` emptyDir volumes as container memory use, rather
than as local ephemeral storage.
{{< /note >}}

### Setting requests and limits for local ephemeral storage

You can use _ephemeral-storage_ for managing local ephemeral storage. Each Container of a Pod can specify one or more of the following:

* `spec.containers[].resources.limits.ephemeral-storage`
* `spec.containers[].resources.requests.ephemeral-storage`

Limits and requests for `ephemeral-storage` are measured in bytes. You can express storage as
a plain integer or as a fixed-point number using one of these suffixes:
E, P, T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi,
Mi, Ki. For example, the following represent roughly the same value:

```shell
128974848, 129e6, 129M, 123Mi
```

In the following example, the Pod has two Containers. Each Container has a request of 2GiB of local ephemeral storage. Each Container has a limit of 4GiB of local ephemeral storage. Therefore, the Pod has a request of 4GiB of local ephemeral storage, and a limit of 8GiB of local ephemeral storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
```

### How Pods with ephemeral-storage requests are scheduled

When you create a Pod, the Kubernetes scheduler selects a node for the Pod to
run on. Each node has a maximum amount of local ephemeral storage it can provide for Pods. For more information, see [Node Allocatable](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable).

The scheduler ensures that the sum of the resource requests of the scheduled Containers is less than the capacity of the node.

### Ephemeral storage consumption management {#resource-emphemeralstorage-consumption}

If the kubelet is managing local ephemeral storage as a resource, then the
kubelet measures storage use in:

- `emptyDir` volumes, except _tmpfs_ `emptyDir` volumes
- directories holding node-level logs
- writeable container layers

If a Pod is using more ephemeral storage than you allow it to, the kubelet
sets an eviction signal that triggers Pod eviction.

For container-level isolation, if a Container's writable layer and log
usage exceeds its storage limit, the kubelet marks the Pod for eviction.

For pod-level isolation the kubelet works out an overall Pod storage limit by
summing the limits for the containers in that Pod. In this case, if the sum of
the local ephemeral storage usage from all containers and also the Pod's `emptyDir`
volumes exceeds the overall Pod storage limit, then the kubelet also marks the Pod
for eviction.

{{< caution >}}
If the kubelet is not measuring local ephemeral storage, then a Pod
that exceeds its local storage limit will not be evicted for breaching
local storage resource limits.

However, if the filesystem space for writeable container layers, node-level logs,
or `emptyDir` volumes falls low, the node
{{< glossary_tooltip text="taints" term_id="taint" >}} itself as short on local storage
and this taint triggers eviction for any Pods that don't specifically tolerate the taint.

See the supported [configurations](#configurations-for-local-ephemeral-storage)
for ephemeral local storage.
{{< /caution >}}

The kubelet supports different ways to measure Pod storage use:

{{< tabs name="resource-emphemeralstorage-measurement" >}}
{{% tab name="Periodic scanning" %}}
The kubelet performs regular, scheduled checks that scan each
`emptyDir` volume, container log directory, and writeable container layer.

The scan measures how much space is used.

{{< note >}}
In this mode, the kubelet does not track open file descriptors
for deleted files.

If you (or a container) create a file inside an `emptyDir` volume,
something then opens that file, and you delete the file while it is
still open, then the inode for the deleted file stays until you close
that file but the kubelet does not categorize the space as in use.
{{< /note >}}
{{% /tab %}}
{{% tab name="Filesystem project quota" %}}

{{< feature-state for_k8s_version="v1.15" state="alpha" >}}

Project quotas are an operating-system level feature for managing
storage use on filesystems. With Kubernetes, you can enable project
quotas for monitoring storage use. Make sure that the filesystem
backing the `emptyDir` volumes, on the node, provides project quota support.
For example, XFS and ext4fs offer project quotas.

{{< note >}}
Project quotas let you monitor storage use; they do not enforce limits.
{{< /note >}}

Kubernetes uses project IDs starting from `1048576`. The IDs in use are
registered in `/etc/projects` and `/etc/projid`. If project IDs in
this range are used for other purposes on the system, those project
IDs must be registered in `/etc/projects` and `/etc/projid` so that
Kubernetes does not use them.

Quotas are faster and more accurate than directory scanning. When a
directory is assigned to a project, all files created under a
directory are created in that project, and the kernel merely has to
keep track of how many blocks are in use by files in that project.  
If a file is created and deleted, but has an open file descriptor,
it continues to consume space. Quota tracking records that space accurately
whereas directory scans overlook the storage used by deleted files.

If you want to use project quotas, you should:

* Enable the `LocalStorageCapacityIsolationFSQuotaMonitoring=true`
  [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
  in the kubelet configuration.

* Ensure that the root filesystem (or optional runtime filesystem)
  has project quotas enabled. All XFS filesystems support project quotas.
  For ext4 filesystems, you need to enable the project quota tracking feature
  while the filesystem is not mounted.
  ```bash
  # For ext4, with /dev/block-device not mounted
  sudo tune2fs -O project -Q prjquota /dev/block-device
  ```

* Ensure that the root filesystem (or optional runtime filesystem) is
  mounted with project quotas enabled. For both XFS and ext4fs, the
  mount option is named `prjquota`.

{{% /tab %}}
{{< /tabs >}}

## Extended resources

Extended resources are fully-qualified resource names outside the
`kubernetes.io` domain. They allow cluster operators to advertise and users to
consume the non-Kubernetes-built-in resources.

There are two steps required to use Extended Resources. First, the cluster
operator must advertise an Extended Resource. Second, users must request the
Extended Resource in Pods.

### Managing extended resources

#### Node-level extended resources

Node-level extended resources are tied to nodes.

##### Device plugin managed resources
See [Device
Plugin](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)
for how to advertise device plugin managed resources on each node.

##### Other resources
To advertise a new node-level extended resource, the cluster operator can
submit a `PATCH` HTTP request to the API server to specify the available
quantity in the `status.capacity` for a node in the cluster. After this
operation, the node's `status.capacity` will include a new resource. The
`status.allocatable` field is updated automatically with the new resource
asynchronously by the kubelet. Note that because the scheduler uses the	node
`status.allocatable` value when evaluating Pod fitness, there may be a short
delay between patching the node capacity with a new resource and the first Pod
that requests the resource to be scheduled on that node.

**Example:**

Here is an example showing how to use `curl` to form an HTTP request that
advertises five "example.com/foo" resources on node `k8s-node-1` whose master
is `k8s-master`.

```shell
curl --header "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path": "/status/capacity/example.com~1foo", "value": "5"}]' \
http://k8s-master:8080/api/v1/nodes/k8s-node-1/status
```

{{< note >}}
In the preceding request, `~1` is the encoding for the character `/`
in the patch path. The operation path value in JSON-Patch is interpreted as a
JSON-Pointer. For more details, see
[IETF RFC 6901, section 3](https://tools.ietf.org/html/rfc6901#section-3).
{{< /note >}}

#### Cluster-level extended resources

Cluster-level extended resources are not tied to nodes. They are usually managed
by scheduler extenders, which handle the resource consumption and resource quota.

You can specify the extended resources that are handled by scheduler extenders
in [scheduler policy
configuration](https://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/scheduler/api/v1/types.go#L31).

**Example:**

The following configuration for a scheduler policy indicates that the
cluster-level extended resource "example.com/foo" is handled by the scheduler
extender.

- The scheduler sends a Pod to the scheduler extender only if the Pod requests
     "example.com/foo".
- The `ignoredByScheduler` field specifies that the scheduler does not check
     the "example.com/foo" resource in its `PodFitsResources` predicate.

```json
{
  "kind": "Policy",
  "apiVersion": "v1",
  "extenders": [
    {
      "urlPrefix":"<extender-endpoint>",
      "bindVerb": "bind",
      "managedResources": [
        {
          "name": "example.com/foo",
          "ignoredByScheduler": true
        }
      ]
    }
  ]
}
```

### Consuming extended resources

Users can consume extended resources in Pod specs just like CPU and memory.
The scheduler takes care of the resource accounting so that no more than the
available amount is simultaneously allocated to Pods.

The API server restricts quantities of extended resources to whole numbers.
Examples of _valid_ quantities are `3`, `3000m` and `3Ki`. Examples of
_invalid_ quantities are `0.5` and `1500m`.

{{< note >}}
Extended resources replace Opaque Integer Resources.
Users can use any domain name prefix other than `kubernetes.io` which is reserved.
{{< /note >}}

To consume an extended resource in a Pod, include the resource name as a key
in the `spec.containers[].resources.limits` map in the container spec.

{{< note >}}
Extended resources cannot be overcommitted, so request and limit
must be equal if both are present in a container spec.
{{< /note >}}

A Pod is scheduled only if all of the resource requests are satisfied, including
CPU, memory and any extended resources. The Pod remains in the `PENDING` state
as long as the resource request cannot be satisfied.

**Example:**

The Pod below requests 2 CPUs and 1 "example.com/foo" (an extended resource).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: myimage
    resources:
      requests:
        cpu: 2
        example.com/foo: 1
      limits:
        example.com/foo: 1
```

## PID limiting

Process ID (PID) limits allow for the configuration of a kubelet to limit the number of PIDs that a given Pod can consume. See [Pid Limiting](/docs/concepts/policy/pid-limiting/) for information.

## Troubleshooting

### My Pods are pending with event message failedScheduling

If the scheduler cannot find any node where a Pod can fit, the Pod remains
unscheduled until a place can be found. An event is produced each time the
scheduler fails to find a place for the Pod, like this:

```shell
kubectl describe pod frontend | grep -A 3 Events
```
```
Events:
  FirstSeen LastSeen   Count  From          Subobject   PathReason      Message
  36s   5s     6      {scheduler }              FailedScheduling  Failed for reason PodExceedsFreeCPU and possibly others
```

In the preceding example, the Pod named "frontend" fails to be scheduled due to
insufficient CPU resource on the node. Similar error messages can also suggest
failure due to insufficient memory (PodExceedsFreeMemory). In general, if a Pod
is pending with a message of this type, there are several things to try:

- Add more nodes to the cluster.
- Terminate unneeded Pods to make room for pending Pods.
- Check that the Pod is not larger than all the nodes. For example, if all the
  nodes have a capacity of `cpu: 1`, then a Pod with a request of `cpu: 1.1` will
  never be scheduled.

You can check node capacities and amounts allocated with the
`kubectl describe nodes` command. For example:

```shell
kubectl describe nodes e2e-test-node-pool-4lw4
```
```
Name:            e2e-test-node-pool-4lw4
[ ... lines removed for clarity ...]
Capacity:
 cpu:                               2
 memory:                            7679792Ki
 pods:                              110
Allocatable:
 cpu:                               1800m
 memory:                            7474992Ki
 pods:                              110
[ ... lines removed for clarity ...]
Non-terminated Pods:        (5 in total)
  Namespace    Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------    ----                                  ------------  ----------  ---------------  -------------
  kube-system  fluentd-gcp-v1.38-28bv1               100m (5%)     0 (0%)      200Mi (2%)       200Mi (2%)
  kube-system  kube-dns-3297075139-61lj3             260m (13%)    0 (0%)      100Mi (1%)       170Mi (2%)
  kube-system  kube-proxy-e2e-test-...               100m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system  monitoring-influxdb-grafana-v4-z1m12  200m (10%)    200m (10%)  600Mi (8%)       600Mi (8%)
  kube-system  node-problem-detector-v0.1-fj7m3      20m (1%)      200m (10%)  20Mi (0%)        100Mi (1%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests    CPU Limits    Memory Requests    Memory Limits
  ------------    ----------    ---------------    -------------
  680m (34%)      400m (20%)    920Mi (11%)        1070Mi (13%)
```

In the preceding output, you can see that if a Pod requests more than 1120m
CPUs or 6.23Gi of memory, it will not fit on the node.

By looking at the `Pods` section, you can see which Pods are taking up space on
the node.

The amount of resources available to Pods is less than the node capacity, because
system daemons use a portion of the available resources. The `allocatable` field
[NodeStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#nodestatus-v1-core)
gives the amount of resources that are available to Pods. For more information, see
[Node Allocatable Resources](https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md).

The [resource quota](/docs/concepts/policy/resource-quotas/) feature can be configured
to limit the total amount of resources that can be consumed. If used in conjunction
with namespaces, it can prevent one team from hogging all the resources.

### My Container is terminated

Your Container might get terminated because it is resource-starved. To check
whether a Container is being killed because it is hitting a resource limit, call
`kubectl describe pod` on the Pod of interest:

```shell
kubectl describe pod simmemleak-hra99
```
```
Name:                           simmemleak-hra99
Namespace:                      default
Image(s):                       saadali/simmemleak
Node:                           kubernetes-node-tf0f/10.240.216.66
Labels:                         name=simmemleak
Status:                         Running
Reason:
Message:
IP:                             10.244.2.75
Replication Controllers:        simmemleak (1/1 replicas created)
Containers:
  simmemleak:
    Image:  saadali/simmemleak
    Limits:
      cpu:                      100m
      memory:                   50Mi
    State:                      Running
      Started:                  Tue, 07 Jul 2015 12:54:41 -0700
    Last Termination State:     Terminated
      Exit Code:                1
      Started:                  Fri, 07 Jul 2015 12:54:30 -0700
      Finished:                 Fri, 07 Jul 2015 12:54:33 -0700
    Ready:                      False
    Restart Count:              5
Conditions:
  Type      Status
  Ready     False
Events:
  FirstSeen                         LastSeen                         Count  From                              SubobjectPath                       Reason      Message
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {scheduler }                                                          scheduled   Successfully assigned simmemleak-hra99 to kubernetes-node-tf0f
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   pulled      Pod container image "k8s.gcr.io/pause:0.8.0" already present on machine
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   created     Created with docker id 6a41280f516d
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   started     Started with docker id 6a41280f516d
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    spec.containers{simmemleak}         created     Created with docker id 87348f12526a
```

In the preceding example, the `Restart Count:  5` indicates that the `simmemleak`
Container in the Pod was terminated and restarted five times.

You can call `kubectl get pod` with the `-o go-template=...` option to fetch the status
of previously terminated Containers:

```shell
kubectl get pod -o go-template='{{range.status.containerStatuses}}{{"Container Name: "}}{{.name}}{{"\r\nLastState: "}}{{.lastState}}{{end}}'  simmemleak-hra99
```
```
Container Name: simmemleak
LastState: map[terminated:map[exitCode:137 reason:OOM Killed startedAt:2015-07-07T20:58:43Z finishedAt:2015-07-07T20:58:43Z containerID:docker://0e4095bba1feccdfe7ef9fb6ebffe972b4b14285d5acdec6f0d3ae8a22fad8b2]]
```

You can see that the Container was terminated because of `reason:OOM Killed`, where `OOM` stands for Out Of Memory.






## {{% heading "whatsnext" %}}


* Get hands-on experience [assigning Memory resources to Containers and Pods](/docs/tasks/configure-pod-container/assign-memory-resource/).

* Get hands-on experience [assigning CPU resources to Containers and Pods](/docs/tasks/configure-pod-container/assign-cpu-resource/).

* For more details about the difference between requests and limits, see
  [Resource QoS](https://git.k8s.io/community/contributors/design-proposals/node/resource-qos.md).

* Read the [Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core) API reference

* Read the [ResourceRequirements](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#resourcerequirements-v1-core) API reference

* Read about [project quotas](https://xfs.org/docs/xfsdocs-xml-dev/XFS_User_Guide/tmp/en-US/html/xfs-quotas.html) in XFS
