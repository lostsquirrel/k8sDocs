---
title: 范围限制
content_type: concept
weight: 10
---
<!--
---
reviewers:
- nelvadas
title: Limit Ranges
content_type: concept
weight: 10
---
 -->
<!-- overview -->
<!--
By default, containers run with unbounded [compute resources](/docs/concepts/configuration/manage-resources-containers/) on a Kubernetes cluster.
With resource quotas, cluster administrators can restrict resource consumption and creation on a {{< glossary_tooltip text="namespace" term_id="namespace" >}} basis.
Within a namespace, a Pod or Container can consume as much CPU and memory as defined by the namespace's resource quota. There is a concern that one Pod or Container could monopolize all available resources. A LimitRange is a policy to constrain resource allocations (to Pods or Containers) in a namespace.
 -->

默认情况下，容器在集群中运行是没有限制其能使用的
[计算资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/)
的. 通过资源配额， 集群管理可以创建
{{< glossary_tooltip term_id="namespace" >}}
级别的资源使用限制。
在一个命名空间中， Pod 或容器可以使用命名空间资源配额规定的 CPU 和内存。 这是担心一个 Pod 或
容器可能会独占所有可用资源。`LimitRange` 就是一个在命名空间中约束(Pod或容器)资源占用的策略。

<!-- body -->
<!--
A _LimitRange_ provides constraints that can:

- Enforce minimum and maximum compute resources usage per Pod or Container in a namespace.
- Enforce minimum and maximum storage request per PersistentVolumeClaim in a namespace.
- Enforce a ratio between request and limit for a resource in a namespace.
- Set default request/limit for compute resources in a namespace and automatically inject them to Containers at runtime.
 -->

_LimitRange_ 提供的限制可以做到以下几点:

- 规定一个命名空间中的每个 Pod 或容器能够使用计算资源的上限和下限
- 规定一个命名空间中的每个 PersistentVolumeClaim 能够存储的上限和下限
- 规定一个命名空间中保底(`request`) 和 上限(`limit`) 之间的比率
- 设置命名空间中计算资源的默认 保底(`request`) / 上限(`limit`) 并在容器运行时自动注入。

<!--
## Enabling LimitRange

LimitRange support has been enabled by default since Kubernetes 1.10.

A LimitRange is enforced in a particular namespace when there is a
LimitRange object in that namespace.

The name of a LimitRange object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
 -->

## 启用 `LimitRange` {#enabling-limitrange}

从 k8s v1.10 开始默认开启 `LimitRange`。

当命名空间中有一个 `LimitRange` 对象时， 这个命名空间就执行这个 `LimitRange`

`LimitRange` 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
<!--
### Overview of Limit Range

- The administrator creates one LimitRange in one namespace.
- Users create resources like Pods, Containers, and PersistentVolumeClaims in the namespace.
- The `LimitRanger` admission controller enforces defaults and limits for all Pods and Containers that do not set compute resource requirements and tracks usage to ensure it does not exceed resource minimum, maximum and ratio defined in any LimitRange present in the namespace.
- If creating or updating a resource (Pod, Container, PersistentVolumeClaim) that violates a LimitRange constraint, the request to the API server will fail with an HTTP status code `403 FORBIDDEN` and a message explaining the constraint that have been violated.
- If a LimitRange is activated in a namespace for compute resources like `cpu` and `memory`, users must specify
  requests or limits for those values. Otherwise, the system may reject Pod creation.
- LimitRange validations occurs only at Pod Admission stage, not on Running Pods.

Examples of policies that could be created using limit range are:

- In a 2 node cluster with a capacity of 8 GiB RAM and 16 cores, constrain Pods in a namespace to request 100m of CPU with a max limit of 500m for CPU and request 200Mi for Memory with a max limit of 600Mi for Memory.
- Define default CPU limit and request to 150m and memory default request to 300Mi for Containers started with no cpu and memory requests in their specs.

In the case where the total limits of the namespace is less than the sum of the limits of the Pods/Containers,
there may be contention for resources. In this case, the Containers or Pods will not be created.

Neither contention nor changes to a LimitRange will affect already created resources.
 -->

### 范围限制概览 {#overview-of-limit-range}

- 管理员在一个命名空间创建一个 `LimitRange`
- 用户在这个命名空间创建像 Pod， 容器， PersistentVolumeClaim 这些资源
- `LimitRanger` 准入控制器规定所有没有设置计算资源要求的 Pod 和容器的默认设置和限制并且跟踪
和保证它们不会超出命名空间中存在的任意 `LimitRange` 资源限制的最小值，最大值和比例
- 如果在创建或修改一个资源(Pod， 容器， PersistentVolumeClaim)会导致违反 LimitRange，这个
到 API 服务的请求就会失败，HTTP 状态码为 `403 FORBIDDEN` 同时还有一个解释其所违反的约束的信息。
- 如果命名空间中激活了对 `cpu` 和 `memory` 这样计算资源的 `LimitRange`， 用户必须设置
保底(`request`) 和 上限(`limit`) 的值， 否则， 系统可能会拒绝这些 Pod 的创建。
- `LimitRange` 验证只发生在 Pod 的准入阶段， 不会在 Pod 运行时进行。

可以使用创建 `LimitRange` 的策略示例:

- 在一个 2 节点的集群中，资源容量为 8 GiB 内存和 16 核心 CPU, 一个命名空间中对 Pod 的约束是
CPU 保底(`request`) 为 `100m` 上限(`limit`) 为 `500m`, 内存 保底(`request`) 为 `200Mi` 上限(`limit`) 为 `600Mi`
- 为那些在启动时在其定义中没有设置 CPU 和 内存的容器定义默认的 CPU 保底(`request`)和上限(`limit`)
为 `150m`， 内存 内存 保底(`request`)和上限(`limit`) 为 `300Mi`

在这种情况下，命名空间的上限小于 Pod/容器 上限的总和， 这时就会有资源争夺。 这时候， 容器或 Pod
可以就没法创建。

无论资源争夺还是修改 `LimitRange` 都不会影响已经创建好的资源。

## {{% heading "whatsnext" %}}

<!--
Refer to the [LimitRanger design document](https://git.k8s.io/community/contributors/design-proposals/resource-management/admission_control_limit_range.md) for more information.

For examples on using limits, see:

- [how to configure minimum and maximum CPU constraints per namespace](/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/).
- [how to configure minimum and maximum Memory constraints per namespace](/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/).
- [how to configure default CPU Requests and Limits per namespace](/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/).
- [how to configure default Memory Requests and Limits per namespace](/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/).
- [how to configure minimum and maximum Storage consumption per namespace](/docs/tasks/administer-cluster/limit-storage-consumption/#limitrange-to-limit-requests-for-storage).
- a [detailed example on configuring quota per namespace](/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/).
 -->

更多信息见
[LimitRanger 设计文稿](https://git.k8s.io/community/contributors/design-proposals/resource-management/admission_control_limit_range.md)。

使用限制示例见:

- [怎么设置命名空间 CPU 的最小和最大约束](/k8sDocs/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/).
- [怎么设置命名空间内存的最小和最大约束](/k8sDocs/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/).
- [怎么设置命名空间的 CPU 默认保底(`request`)和上限(`limit`)](/k8sDocs/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/).
- [怎么设置命名空间的内存默认保底(`request`)和上限(`limit`)](/k8sDocs/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/).
- [怎么设置命名空间存储的最小和最大消耗](/k8sDocs/docs/tasks/administer-cluster/limit-storage-consumption/#limitrange-to-limit-requests-for-storage).
- 一个[命名空间配置配额的详细示例](/k8sDocs/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/).
