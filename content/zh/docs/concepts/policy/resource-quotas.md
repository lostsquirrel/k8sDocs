---
title: 资源配额
content_type: concept
weight: 20
---
<!--
---
reviewers:
- derekwaynecarr
title: Resource Quotas
content_type: concept
weight: 20
---
 -->
<!-- overview -->

当多个用户或小组共享一个固定数量节点的集群时，就可能要担心一个小组可能使用超出公开分离的资源。

资源配额就一个为管理员解决定个问题的工具。
<!-- body -->

一个资源配额，是由一个 `ResourceQuota` 对象定义的，它提供了对每个命名空间可以使用的资源总和约束。
它可以限制在一个命名空间中可以创建该类型的对象的数量，也可以限制命名空间中资源可以使用的计算资源的
总数。

资源配额是这样工作的：

- 不同的组在不同的命名空间干活。 目前这是自愿的，但计划支持通过 ACL 将它变成强制的。
- 管理为每个命名空间创建一个 `ResourceQuota`
- 如果在创建或修改一个资源时违反一配额限制，则这个请求就会失败，HTTP 状态码为`403 FORBIDDEN`
  错误信息中解释了违反了哪个限制
- 如果命名空间中启用了对像 `cpu` 和 `memory` 这些资源的配额， 用户必须设置 保底(`request`)
  和 上限(`limit`) 的值， 否则，配额系统可能会拒绝这些 Pod 的创建。提示: 使用 `LimitRanger`
  准入控制器为那些没有设置计算资源要求的 Pod 强制添加默认的配置。
  怎么避免这个问题见
  [示例](/k8sDocs/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)

ResourceQuota 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

可以使用命名空间和配额创建的策略的示例:

- 在一个拥有 32 GiB RAM, 和 16 核心容量的集群中，让 A 组使用 20 GiB and 10 核心,
  让 B 组使用 10 GiB and 4 核心, 保留 2GiB 和 2 核心将来分配。

- 限制叫 "testing" 命名空间只能使用 1 核心和 1GiB RAM.  让叫 "production" 的命名空间可以使用任意数量。
<!--
## Enabling Resource Quota

Resource Quota support is enabled by default for many Kubernetes distributions.  It is
enabled when the API server `--enable-admission-plugins=` flag has `ResourceQuota` as
one of its arguments.

A resource quota is enforced in a particular namespace when there is a
ResourceQuota in that namespace.
 -->

## 启用资源配额 {#enabling-resource-quota}

对支持配额的支持在许多 k8s 发行中是默认开启的。 当 API 服务在其 `--enable-admission-plugins=`
中包含 `ResourceQuota` 就用户了该特性。

当一个命名空间中有一个 `ResourceQuota` 对象时，就会对这个命名空间执行资源配额。

<!--
## Compute Resource Quota

You can limit the total sum of [compute resources](/docs/concepts/configuration/manage-resources-containers/) that can be requested in a given namespace.

The following resource types are supported:

| Resource Name | Description |
| --------------------- | ----------------------------------------------------------- |
| `limits.cpu` | Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value. |
| `limits.memory` | Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value. |
| `requests.cpu` | Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value. |
| `requests.memory` | Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value. |
| `hugepages-<size>` | Across all pods in a non-terminal state, the number of huge page requests of the specified size cannot exceed this value. |
| `cpu` | Same as `requests.cpu` |
| `memory` | Same as `requests.memory` |
 -->

## 计算资源配额 {#compute-resource-quota}

用户可以限制在指定命名空间可以请求的
[计算资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/)
的总和

支持以下资源类型:

|资源名称 | 描述 |
| --------------------- | ----------------------------------------------------------- |
| `limits.cpu` | 所有不是终止状态的 Pod 的 CPU 上限总和不能超过这个值|
| `limits.memory` | 所有不是终止状态的 Pod 的 内存 上限总和不能超过这个值 |
| `requests.cpu` | 所有不是终止状态的 Pod 的 CPU 保底要求总和不能超过这个值 |
| `requests.memory` | 所有不是终止状态的 Pod 的 内存 保底要求总和不能超过这个值 |
| `hugepages-<size>` | 所有不是终止状态的 Pod 的这个尺寸的保底要求总和不能超过这个值|
| `cpu` | 同 `requests.cpu` |
| `memory` | 同 `requests.memory` |

<!--
### Resource Quota For Extended Resources

In addition to the resources mentioned above, in release 1.10, quota support for
[extended resources](/docs/concepts/configuration/manage-resources-containers/#extended-resources) is added.

As overcommit is not allowed for extended resources, it makes no sense to specify both `requests`
and `limits` for the same extended resource in a quota. So for extended resources, only quota items
with prefix `requests.` is allowed for now.

Take the GPU resource as an example, if the resource name is `nvidia.com/gpu`, and you want to
limit the total number of GPUs requested in a namespace to 4, you can define a quota as follows:

* `requests.nvidia.com/gpu: 4`

See [Viewing and Setting Quotas](#viewing-and-setting-quotas) for more detail information.
 -->


### 扩展资源的资源配额 {#resource-quota-for-extended-resources}

除了上面提到的资源外， 在 v1.10 版本中添加了对
[扩展资源](/docs/concepts/configuration/manage-resources-containers/#extended-resources)
的配额支持。

因为扩展资源不能超量使用， 所以在配额中对于都一个扩展资源同时指定 `requests` 和 `limits` 是
没有意义的，目前只允许使用 `requests.` 开头的配额条目。

就拿 GPU 资源来举例， 如果资源名称是 `nvidia.com/gpu`，然后希望在命名空间中对 GPU 的申请总和
为 4， 就可以定义下面这样一个配额:

- `requests.nvidia.com/gpu: 4`

更多信息见 [查看和设置配额](#viewing-and-setting-quotas)

<!--
## Storage Resource Quota

You can limit the total sum of [storage resources](/docs/concepts/storage/persistent-volumes/) that can be requested in a given namespace.

In addition, you can limit consumption of storage resources based on associated storage-class.

| Resource Name | Description |
| --------------------- | ----------------------------------------------------------- |
| `requests.storage` | Across all persistent volume claims, the sum of storage requests cannot exceed this value. |
| `persistentvolumeclaims` | The total number of [PersistentVolumeClaims](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) that can exist in the namespace. |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage` | Across all persistent volume claims associated with the `<storage-class-name>`, the sum of storage requests cannot exceed this value. |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | Across all persistent volume claims associated with the storage-class-name, the total number of [persistent volume claims](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) that can exist in the namespace. |

For example, if an operator wants to quota storage with `gold` storage class separate from `bronze` storage class, the operator can
define a quota as follows:

* `gold.storageclass.storage.k8s.io/requests.storage: 500Gi`
* `bronze.storageclass.storage.k8s.io/requests.storage: 100Gi`

In release 1.8, quota support for local ephemeral storage is added as an alpha feature:

| Resource Name | Description |
| ------------------------------- |----------------------------------------------------------- |
| `requests.ephemeral-storage` | Across all pods in the namespace, the sum of local ephemeral storage requests cannot exceed this value. |
| `limits.ephemeral-storage` | Across all pods in the namespace, the sum of local ephemeral storage limits cannot exceed this value. |
| `ephemeral-storage` | Same as `requests.ephemeral-storage`. |
 -->

## 存储资源配额 {#storage-resource-quota}

用户可以限制在指定命名空间中能够要求
[计算资源](/docs/concepts/storage/persistent-volumes/)
的总和。

另外，也可以基于对应的存储类别(StorageClass)来限制消耗的存储资源上限。

| 资源名称 | 描述 |
| --------------------- | ----------------------------------------------------------- |
| `requests.storage` | 包括所有的 PVC， 存储请求的问题不能超过这个值|
| `persistentvolumeclaims` | 在命名空间中可以存在 [PersistentVolumeClaims](/k8sDocs/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 的数量不能超过这个值 |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage` | 所有使用这个 `<storage-class-name>` 的  PVC 所请求的存储资源总和不能超过这个值. |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | 所有使用这个 `<storage-class-name>` 的 [PVC](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 的数量不能超过这个值 |

例如, 如果一个运维人员想要分别定义 `gold` 存储类别与 `bronze` 存储类别配额，则可以定义如下配额:

- `gold.storageclass.storage.k8s.io/requests.storage: 500Gi`
- `bronze.storageclass.storage.k8s.io/requests.storage: 100Gi`

In release 1.8, quota support for local ephemeral storage is added as an alpha feature:
在 v1.8 中，配额作为 alpha 特性支持本地临时存储

| 资源名称 | 描述 |
| ------------------------------- |----------------------------------------------------------- |
| `requests.ephemeral-storage` |命名空间内所有的 Pod 请求本地临时存储资源的总和不能超过这个值 |
| `limits.ephemeral-storage` | 命名空间内所有的 Pod 本地临时存储配置上限的总和不能超过这个值|
| `ephemeral-storage` | 同 `requests.ephemeral-storage`. |

<!--
## Object Count Quota

You can set quota for the total number of certain resources of all standard,
namespaced resource types using the following syntax:

* `count/<resource>.<group>` for resources from non-core groups
* `count/<resource>` for resources from the core group

Here is an example set of resources users may want to put under object count quota:

* `count/persistentvolumeclaims`
* `count/services`
* `count/secrets`
* `count/configmaps`
* `count/replicationcontrollers`
* `count/deployments.apps`
* `count/replicasets.apps`
* `count/statefulsets.apps`
* `count/jobs.batch`
* `count/cronjobs.batch`

The same syntax can be used for custom resources.
For example, to create a quota on a `widgets` custom resource in the `example.com` API group, use `count/widgets.example.com`.

When using `count/*` resource quota, an object is charged against the quota if it exists in server storage.
These types of quotas are useful to protect against exhaustion of storage resources.  For example, you may
want to limit the number of Secrets in a server given their large size. Too many Secrets in a cluster can
actually prevent servers and controllers from starting. You can set a quota for Jobs to protect against
a poorly configured CronJob. CronJobs that create too many Jobs in a namespace can lead to a denial of service.

It is also possible to do generic object count quota on a limited set of resources.
The following types are supported:

| Resource Name | Description |
| ------------------------------- | ------------------------------------------------- |
| `configmaps` | The total number of ConfigMaps that can exist in the namespace. |
| `persistentvolumeclaims` | The total number of [PersistentVolumeClaims](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) that can exist in the namespace. |
| `pods` | The total number of Pods in a non-terminal state that can exist in the namespace.  A pod is in a terminal state if `.status.phase in (Failed, Succeeded)` is true.  |
| `replicationcontrollers` | The total number of ReplicationControllers that can exist in the namespace. |
| `resourcequotas` | The total number of ResourceQuotas that can exist in the namespace. |
| `services` | The total number of Services that can exist in the namespace. |
| `services.loadbalancers` | The total number of Services of type `LoadBalancer` that can exist in the namespace. |
| `services.nodeports` | The total number of Services of type `NodePort` that can exist in the namespace. |
| `secrets` | The total number of Secrets that can exist in the namespace. |

For example, `pods` quota counts and enforces a maximum on the number of `pods`
created in a single namespace that are not terminal. You might want to set a `pods`
quota on a namespace to avoid the case where a user creates many small pods and
exhausts the cluster's supply of Pod IPs.
 -->

## 对象数量配额 {#object-count-quota}

用户可以使用下面的语法为所有标准的特定资源，有命名空间的资源类型设置配额总数:
{{<todo-optimize>}}

- `count/<resource>.<group>` 用于非核心组的资源
- `count/<resource>` 用于核心组的资源

以下是一些用户可能希望添加数量配额的资源集合示例:

* `count/persistentvolumeclaims`
* `count/services`
* `count/secrets`
* `count/configmaps`
* `count/replicationcontrollers`
* `count/deployments.apps`
* `count/replicasets.apps`
* `count/statefulsets.apps`
* `count/jobs.batch`
* `count/cronjobs.batch`

自定义资源也可以使用同样的语法。 例如，要为 `example.com` API 组中的  `widgets` 自定义资源
创建一个配额，可以用 `count/widgets.example.com`.

当使用 `count/*` 资源配额时， 如果它存在于服务器存储它就会消耗对应的配额。  这些类别的配额在
防止存储资源耗尽上是相当有用的。 例如， 用户可以希望限制给予其大尺寸的服务器上的 Secret 的数量。
如果集群中有太多的 Secret 就会阻止服务和控制器的启动。 用户也可以对 Job 数量设置配额来应对
配置不良的 CronJob。 CronJob 可能在一个命名空间中创建太多 Job 而导致拒绝服务。

也可以为资源集添加通用对象数量配额。
支持以下资源类型:

| 资源名称 | 描述 |
| ------------------------------- | ------------------------------------------------- |
| `configmaps` | 这个命名空间中的 ConfigMap 的总数不能超过这个值 |
| `persistentvolumeclaims` | 这个命名空间中的 [PersistentVolumeClaim](/k8sDocs/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 的总数不能超过这个值 |
| `pods` | 这个命名空间中不是终止状态的 Pod 的总数不能超过这个值.  如果 Pod 的 `.status.phase 是 (Failed, Succeeded)` 则就是终止状态  |
| `replicationcontrollers` | 这个命名空间中 ReplicationController 的总数不能超过这个值 |
| `resourcequotas` | 这个命名空间中 ResourceQuota 的总数不能超过这个值|
| `services` | 这个命名空间中 Service 的总数不能超过这个值 |
| `services.loadbalancers` | 这个命名空间中 `LoadBalancer` 类型的 Service 的总数不能超过这个值 |
| `services.nodeports` | 这个命名空间中 `NodePort` 类型的 Service 的总数不能超过这个值|
| `secrets` | 这个命名空间中 Secret 的总数不能超过这个值 |

例如， `pods` 配额数量就是确保在一个命名空间中不是终止状态的 Pod 的最大数量。 用户可能希望在
命名空间上设置一个 `pods` 配置为防止因为一个用户创建太多小的 Pod 而耗用集群的 Pod IP。

<!--
## Quota Scopes

Each quota can have an associated set of `scopes`. A quota will only measure usage for a resource if it matches
the intersection of enumerated scopes.

When a scope is added to the quota, it limits the number of resources it supports to those that pertain to the scope.
Resources specified on the quota outside of the allowed set results in a validation error.

| Scope | Description |
| ----- | ----------- |
| `Terminating` | Match pods where `.spec.activeDeadlineSeconds >= 0` |
| `NotTerminating` | Match pods where `.spec.activeDeadlineSeconds is nil` |
| `BestEffort` | Match pods that have best effort quality of service. |
| `NotBestEffort` | Match pods that do not have best effort quality of service. |
| `PriorityClass` | Match pods that references the specified [priority class](/docs/concepts/configuration/pod-priority-preemption). |

The `BestEffort` scope restricts a quota to tracking the following resource:

* `pods`

The `Terminating`, `NotTerminating`, `NotBestEffort` and `PriorityClass`
scopes restrict a quota to tracking the following resources:

* `pods`
* `cpu`
* `memory`
* `requests.cpu`
* `requests.memory`
* `limits.cpu`
* `limits.memory`

Note that you cannot specify both the `Terminating` and the `NotTerminating`
scopes in the same quota, and you cannot specify both the `BestEffort` and
`NotBestEffort` scopes in the same quota either.

The `scopeSelector` supports the following values in the `operator` field:

* `In`
* `NotIn`
* `Exists`
* `DoesNotExist`

When using one of the following values as the `scopeName` when defining the
`scopeSelector`, the `operator` must be `Exists`.

* `Terminating`
* `NotTerminating`
* `BestEffort`
* `NotBestEffort`

If the `operator` is `In` or `NotIn`, the `values` field must have at least
one value. For example:

```yaml
  scopeSelector:
    matchExpressions:
      - scopeName: PriorityClass
        operator: In
        values:
          - middle
```

If the `operator` is `Exists` or `DoesNotExist`, the `values` field must *NOT* be
specified.
 -->

## 配额作用域 {#quota-scopes}

每个配额都可以有一个相应的作用域(`scopes`)集合。 一个配额只会度量在其作用域集合交集内的资源使用。

当一个配额添加了作用域后，它就只会限制在它作用域内的资源使用数量。
配额上指定的资源是允许集合之外就会导致验证错误。

{{<todo-optimize>}}

| 作用域 | 描述 |
| ----- | ----------- |
| `Terminating` | 匹配 `.spec.activeDeadlineSeconds >= 0` 的 Pod  |
| `NotTerminating` | 匹配 `.spec.activeDeadlineSeconds is nil` 的 Pod |
| `BestEffort` | 匹配服务质量是尽最大努力的 Pod |
| `NotBestEffort` | 匹配服务质量不是尽最大努力的 Pod |
| `PriorityClass` | 匹配引用了指定 [PriorityClass](/k8sDocs/docs/concepts/configuration/pod-priority-preemption) 的 Pod |

`BestEffort` 作用域限制配额跟踪下面的资源:

* `pods`

`Terminating`, `NotTerminating`, `NotBestEffort` `PriorityClass` 作用域限制配额
跟踪下面的资源:

* `pods`
* `cpu`
* `memory`
* `requests.cpu`
* `requests.memory`
* `limits.cpu`
* `limits.memory`

要注意不能在同一个配额中同时指定 `Terminating` 和 `NotTerminating` 作用域，也不能
在同一个配额中同时指定 `BestEffort` 和 `NotBestEffort` 作用域.

`scopeSelector` 的 `operator` 字段支持以下值:
* `In`
* `NotIn`
* `Exists`
* `DoesNotExist`

When using one of the following values as the `scopeName` when defining the
`scopeSelector`, the `operator` must be `Exists`.

在使用以下任意一个值作为 `scopeName` 定义 `scopeSelector` 时， `operator` 必须是 `Exists`.

* `Terminating`
* `NotTerminating`
* `BestEffort`
* `NotBestEffort`

如果 `operator` 是 `In` 或 `NotIn`, `values` 字段就至少有一个值。 例如:

```yaml
  scopeSelector:
    matchExpressions:
      - scopeName: PriorityClass
        operator: In
        values:
          - middle
```

如果 `operator` 是 `Exists` 或 `DoesNotExist`, `values` 字段就 *不能* 设置

<!--
### Resource Quota Per PriorityClass

{{< feature-state for_k8s_version="v1.17" state="stable" >}}

Pods can be created at a specific [priority](/docs/concepts/configuration/pod-priority-preemption/#pod-priority).
You can control a pod's consumption of system resources based on a pod's priority, by using the `scopeSelector`
field in the quota spec.

A quota is matched and consumed only if `scopeSelector` in the quota spec selects the pod.

When quota is scoped for priority class using `scopeSelector` field, quota object is restricted to track only following resources:

* `pods`
* `cpu`
* `memory`
* `ephemeral-storage`
* `limits.cpu`
* `limits.memory`
* `limits.ephemeral-storage`
* `requests.cpu`
* `requests.memory`
* `requests.ephemeral-storage`

This example creates a quota object and matches it with pods at specific priorities. The example
works as follows:

- Pods in the cluster have one of the three priority classes, "low", "medium", "high".
- One quota object is created for each priority.

Save the following YAML to a file `quota.yml`.

```yaml
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-high
  spec:
    hard:
      cpu: "1000"
      memory: 200Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["high"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-medium
  spec:
    hard:
      cpu: "10"
      memory: 20Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["medium"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-low
  spec:
    hard:
      cpu: "5"
      memory: 10Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["low"]
```

Apply the YAML using `kubectl create`.

```shell
kubectl create -f ./quota.yml
```

```
resourcequota/pods-high created
resourcequota/pods-medium created
resourcequota/pods-low created
```

Verify that `Used` quota is `0` using `kubectl describe quota`.

```shell
kubectl describe quota
```

```
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     1k
memory      0     200Gi
pods        0     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

Create a pod with priority "high". Save the following YAML to a
file `high-priority-pod.yml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority
spec:
  containers:
  - name: high-priority
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
    resources:
      requests:
        memory: "10Gi"
        cpu: "500m"
      limits:
        memory: "10Gi"
        cpu: "500m"
  priorityClassName: high
```

Apply it with `kubectl create`.

```shell
kubectl create -f ./high-priority-pod.yml
```

Verify that "Used" stats for "high" priority quota, `pods-high`, has changed and that
the other two quotas are unchanged.

```shell
kubectl describe quota
```

```
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         500m  1k
memory      10Gi  200Gi
pods        1     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```
-->

### 每个 PriorityClass 的资源配额 {#resource-quota-per-priorityclass}

{{< feature-state for_k8s_version="v1.17" state="stable" >}}

Pods can be created at a specific [priority](/docs/concepts/configuration/pod-priority-preemption/#pod-priority).
You can control a pod's consumption of system resources based on a pod's priority, by using the `scopeSelector`
field in the quota spec.

Pod 可以以一个指定的
[优先级](/docs/concepts/configuration/pod-priority-preemption/#pod-priority)
来创建。
用户可以通过使用配额定义中的 `scopeSelector` 字段来基于 Pod 的优先级来控制一个 Pod 可以消耗
的系统资源。

A quota is matched and consumed only if `scopeSelector` in the quota spec selects the pod.
配额只有在其定义的 `scopeSelector` 匹配的 Pod 才会消耗。

当配额使用 `scopeSelector` 字段设置优先级类别作用域， 配额对象就被限制在只能跟踪以下资源:

* `pods`
* `cpu`
* `memory`
* `ephemeral-storage`
* `limits.cpu`
* `limits.memory`
* `limits.ephemeral-storage`
* `requests.cpu`
* `requests.memory`
* `requests.ephemeral-storage`

这个示例创建一个配额对象，匹配指定优先级的 Pod。 工作内容如下:

- 集群中的 Pod 有这三种优先级类别， "low", "medium", "high" 中的一种。
- 每个优先级类别创建一个对应的配额对象

将以下 YAML 保存到文件 `quota.yml`.

```yaml
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-high
  spec:
    hard:
      cpu: "1000"
      memory: 200Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["high"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-medium
  spec:
    hard:
      cpu: "10"
      memory: 20Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["medium"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-low
  spec:
    hard:
      cpu: "5"
      memory: 10Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["low"]
```

使用 `kubectl create` 应用 YAML.

```shell
kubectl create -f ./quota.yml
```

```
resourcequota/pods-high created
resourcequota/pods-medium created
resourcequota/pods-low created
```

使用  `kubectl describe quota` 验证对配额的使用(`Used`) 都是 `0`

```shell
kubectl describe quota
```

```
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     1k
memory      0     200Gi
pods        0     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

创建一个优先级是 "high" 的 Pod. 将以下 YAML 保存到文件 `high-priority-pod.yml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority
spec:
  containers:
  - name: high-priority
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
    resources:
      requests:
        memory: "10Gi"
        cpu: "500m"
      limits:
        memory: "10Gi"
        cpu: "500m"
  priorityClassName: high
```

通过 `kubectl create` 命令应用.

```shell
kubectl create -f ./high-priority-pod.yml
```

验证 "high" 优先级的配额的使用状态，这时 `pods-high` 配额发生变化，其它两个不变

```shell
kubectl describe quota
```

```
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         500m  1k
memory      10Gi  200Gi
pods        1     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```
<!--
## Requests compared to Limits {#requests-vs-limits}

When allocating compute resources, each container may specify a request and a limit value for either CPU or memory.
The quota can be configured to quota either value.

If the quota has a value specified for `requests.cpu` or `requests.memory`, then it requires that every incoming
container makes an explicit request for those resources.  If the quota has a value specified for `limits.cpu` or `limits.memory`,
then it requires that every incoming container specifies an explicit limit for those resources.
 -->

## 比较保底要求与使用上限 {#requests-vs-limits}

在分配计算资源时，每个容器都可以对CPU 或/和 memory 指定一个 保底要求 和一个使用上限值。
配额也可以配置任意一个值的额度。

如果配额为 `requests.cpu` 或 `requests.memory` 指定了一个值，则就要求每个新进的容器需要
显示的设置这些资源的保底要求。
如果配额为 `limits.cpu` 或 `limits.memory` 指定了一个值，则就要求每个新进的容器需要
显示的设置这些资源的使用上限。

<!--
## Viewing and Setting Quotas

Kubectl supports creating, updating, and viewing quotas:

```shell
kubectl create namespace myspace
```

```shell
cat <<EOF > compute-resources.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
EOF
```

```shell
kubectl create -f ./compute-resources.yaml --namespace=myspace
```

```shell
cat <<EOF > object-counts.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
EOF
```

```shell
kubectl create -f ./object-counts.yaml --namespace=myspace
```

```shell
kubectl get quota --namespace=myspace
```

```
NAME                    AGE
compute-resources       30s
object-counts           32s
```

```shell
kubectl describe quota compute-resources --namespace=myspace
```

```
Name:                    compute-resources
Namespace:               myspace
Resource                 Used  Hard
--------                 ----  ----
limits.cpu               0     2
limits.memory            0     2Gi
requests.cpu             0     1
requests.memory          0     1Gi
requests.nvidia.com/gpu  0     4
```

```shell
kubectl describe quota object-counts --namespace=myspace
```

```
Name:                   object-counts
Namespace:              myspace
Resource                Used    Hard
--------                ----    ----
configmaps              0       10
persistentvolumeclaims  0       4
pods                    0       4
replicationcontrollers  0       20
secrets                 1       10
services                0       10
services.loadbalancers  0       2
```

Kubectl also supports object count quota for all standard namespaced resources
using the syntax `count/<resource>.<group>`:

```shell
kubectl create namespace myspace
```

```shell
kubectl create quota test --hard=count/deployments.apps=2,count/replicasets.apps=4,count/pods=3,count/secrets=4 --namespace=myspace
```

```shell
kubectl create deployment nginx --image=nginx --namespace=myspace --replicas=2
```

```shell
kubectl describe quota --namespace=myspace
```

```
Name:                         test
Namespace:                    myspace
Resource                      Used  Hard
--------                      ----  ----
count/deployments.apps        1     2
count/pods                    2     3
count/replicasets.apps        1     4
count/secrets                 1     4
```
 -->

## 查看和使用配额 {#viewing-and-setting-quotas}

kubectl 支持创建，修改，查看配额:

```shell
kubectl create namespace myspace
```

```shell
cat <<EOF > compute-resources.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
EOF
```

```shell
kubectl create -f ./compute-resources.yaml --namespace=myspace
```

```shell
cat <<EOF > object-counts.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
EOF
```

```shell
kubectl create -f ./object-counts.yaml --namespace=myspace
```

```shell
kubectl get quota --namespace=myspace
```

```
NAME                    AGE
compute-resources       30s
object-counts           32s
```

```shell
kubectl describe quota compute-resources --namespace=myspace
```

```
Name:                    compute-resources
Namespace:               myspace
Resource                 Used  Hard
--------                 ----  ----
limits.cpu               0     2
limits.memory            0     2Gi
requests.cpu             0     1
requests.memory          0     1Gi
requests.nvidia.com/gpu  0     4
```

```shell
kubectl describe quota object-counts --namespace=myspace
```

```
Name:                   object-counts
Namespace:              myspace
Resource                Used    Hard
--------                ----    ----
configmaps              0       10
persistentvolumeclaims  0       4
pods                    0       4
replicationcontrollers  0       20
secrets                 1       10
services                0       10
services.loadbalancers  0       2
```
qq
kubectl 也支持对所有的标准有命名空间资源使用 `count/<resource>.<group>` 语法设置对象数配额:

```shell
kubectl create namespace myspace
```

```shell
kubectl create quota test --hard=count/deployments.apps=2,count/replicasets.apps=4,count/pods=3,count/secrets=4 --namespace=myspace
```

```shell
kubectl create deployment nginx --image=nginx --namespace=myspace --replicas=2
```

```shell
kubectl describe quota --namespace=myspace
```

```
Name:                         test
Namespace:                    myspace
Resource                      Used  Hard
--------                      ----  ----
count/deployments.apps        1     2
count/pods                    2     3
count/replicasets.apps        1     4
count/secrets                 1     4
```
<!--
## Quota and Cluster Capacity

ResourceQuotas are independent of the cluster capacity. They are
expressed in absolute units.  So, if you add nodes to your cluster, this does *not*
automatically give each namespace the ability to consume more resources.

Sometimes more complex policies may be desired, such as:

- Proportionally divide total cluster resources among several teams.
- Allow each tenant to grow resource usage as needed, but have a generous
  limit to prevent accidental resource exhaustion.
- Detect demand from one namespace, add nodes, and increase quota.

Such policies could be implemented using `ResourceQuotas` as building blocks, by
writing a "controller" that watches the quota usage and adjusts the quota
hard limits of each namespace according to other signals.

Note that resource quota divides up aggregate cluster resources, but it creates no
restrictions around nodes: pods from several namespaces may run on the same node.
 -->

## 配额与集群容量 {#quota-and-cluster-capacity}

ResourceQuotas are independent of the cluster capacity. They are
expressed in absolute units.  So, if you add nodes to your cluster, this does *not*
automatically give each namespace the ability to consume more resources.

资源配额是独立于集群容量的。他们是以绝对单位来表达的。 所以，如果向集群中添加了节点，并 *不* 能
自动地给予每个命名空间消耗更多资源的能力。

Sometimes more complex policies may be desired, such as:
有时候还需要更加复杂的策略，如:
- Proportionally divide total cluster resources among several teams.
- Allow each tenant to grow resource usage as needed, but have a generous
  limit to prevent accidental resource exhaustion.
- Detect demand from one namespace, add nodes, and increase quota.

- 在几个团队之间按比例分配集群总资源。
- 允许每个租户根据需要增加资源，但有一个相对较充裕的限制阴谋诡计意外的资源耗尽
- 侦测一个命名空间的要求，添加节点，再增加配额。

Such policies could be implemented using `ResourceQuotas` as building blocks, by
writing a "controller" that watches the quota usage and adjusts the quota
hard limits of each namespace according to other signals.

这些策略可以使用 `ResourceQuotas` 作为实现基础，通过编写控制器("controller") 来监测配额的
使用情况，并根据其它信息自适应每个命名空间的配额的硬限制

Note that resource quota divides up aggregate cluster resources, but it creates no
restrictions around nodes: pods from several namespaces may run on the same node.

要注意资源配额分割了集群的资源，但它并不会在节点上创建限制：来自不同命名空间的 Pod 可以运行在同
一个节点上。

<!--
## Limit Priority Class consumption by default

It may be desired that pods at a particular priority, eg. "cluster-services",
should be allowed in a namespace, if and only if, a matching quota object exists.

With this mechanism, operators are able to restrict usage of certain high
priority classes to a limited number of namespaces and not every namespace
will be able to consume these priority classes by default.

To enforce this, `kube-apiserver` flag `--admission-control-config-file` should be
used to pass path to the following configuration file:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: "ResourceQuota"
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: ResourceQuotaConfiguration
    limitedResources:
    - resource: pods
      matchScopes:
      - scopeName: PriorityClass
        operator: In
        values: ["cluster-services"]
```

Now, "cluster-services" pods will be allowed in only those namespaces where a quota object with a matching `scopeSelector` is present.
For example:

```yaml
    scopeSelector:
      matchExpressions:
      - scopeName: PriorityClass
        operator: In
        values: ["cluster-services"]
```
 -->

## 限制指定优先级类别的默认消耗 {#limit-priority-class-consumption-by-default}

还有种可能是希望指定优先级的 Pod，就例如，"cluster-services",就允许且仅允许它们在有匹配配额
对象存在的命名空间中。

通过这种机制，运维人员可以使得在默认情况下限制特定高优先级类别的使用在有限的几个命名空间中，
而不是每个命名空间都可以消费这些优先级类别。

而要实现这一点，`kube-apiserver` 的 `--admission-control-config-file` 应该传入包含下面
内容的文件的路径:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: "ResourceQuota"
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: ResourceQuotaConfiguration
    limitedResources:
    - resource: pods
      matchScopes:
      - scopeName: PriorityClass
        operator: In
        values: ["cluster-services"]
```

此时，使用 "cluster-services" 优先级类别的 Pod 就允许且仅允许它们在这些命名空间中，这些命名
空间中要有一个配额的 `scopeSelector` 匹配 "cluster-services" 这个优先级类别的。

```yaml
    scopeSelector:
      matchExpressions:
      - scopeName: PriorityClass
        operator: In
        values: ["cluster-services"]
```

## {{% heading "whatsnext" %}}

- [ResourceQuota 设计文稿](https://git.k8s.io/community/contributors/design-proposals/resource-management/admission_control_resource_quota.md)
- [怎么使用资源配额的详细示例](/k8sDocs/docs/tasks/administer-cluster/quota-api-object/).
- [对优先级类别配额支持的设计文稿](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/pod-priority-resourcequota.md).
- [LimitedResources](https://github.com/kubernetes/kubernetes/pull/36765)
