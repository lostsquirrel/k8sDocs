---
title: 存储容量
content_type: concept
weight: 80
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- msau42
- xing-yang
- pohly
title: Storage Capacity
content_type: concept
weight: 45
---
 -->
<!-- overview -->
<!--
Storage capacity is limited and may vary depending on the node on
which a pod runs: network-attached storage might not be accessible by
all nodes, or storage is local to a node to begin with.

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

This page describes how Kubernetes keeps track of storage capacity and
how the scheduler uses that information to schedule Pods onto nodes
that have access to enough storage capacity for the remaining missing
volumes. Without storage capacity tracking, the scheduler may choose a
node that doesn't have enough capacity to provision a volume and
multiple scheduling retries will be needed.

Tracking storage capacity is supported for {{< glossary_tooltip
text="Container Storage Interface" term_id="csi" >}} (CSI) drivers and
[needs to be enabled](#enabling-storage-capacity-tracking) when installing a CSI driver.
 -->
存储容量是受限的并且还基于 Pod 运行的节点: 网络存储可以不能在所有的节点上访问，或者存储只属于
Pod 启动的节点上。

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

本文主要介绍 k8s 是怎么保持对存储容量的跟踪并且调度器使用这些信息来把 Pod 调度到能够访问到
剩余没使用的卷有足够存储容量。 如果没有对存储容量的跟踪， 造成调度器因调度到一个没有足够存储容量
的节点而造成多次尝试重新调度。

跟踪存储容量是支持
{{< glossary_tooltip term_id="csi" >}} (CSI) 驱动并且在安装 CSI 驱动时
[需要开启](#enabling-storage-capacity-tracking)

<!-- body -->
<!--
## API

There are two API extensions for this feature:
- [CSIStorageCapacity](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#csistoragecapacity-v1alpha1-storage-k8s-io) objects:
  these get produced by a CSI driver in the namespace
  where the driver is installed. Each object contains capacity
  information for one storage class and defines which nodes have
  access to that storage.
- [The `CSIDriverSpec.StorageCapacity` field](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#csidriverspec-v1-storage-k8s-io):
  when set to `true`, the Kubernetes scheduler will consider storage
  capacity for volumes that use the CSI driver.
 -->

## API

对于这个特性有两个 API 扩展:
- [CSIStorageCapacity](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#csistoragecapacity-v1alpha1-storage-k8s-io) 对象:
  这些是当驱动被安装时由 CSI 驱动在命名空间中生成的。 每个对象包含一个 StorageClass 的容量信息
  并定义哪个节点可以访问这个存储。
- [`CSIDriverSpec.StorageCapacity` 字段](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#csidriverspec-v1-storage-k8s-io):
  当设置为 `true`， k8s 调度器会考量使用 CSI 驱动的卷的存储容量
<!--
## Scheduling

Storage capacity information is used by the Kubernetes scheduler if:
- the `CSIStorageCapacity` feature gate is true,
- a Pod uses a volume that has not been created yet,
- that volume uses a {{< glossary_tooltip text="StorageClass" term_id="storage-class" >}} which references a CSI driver and
  uses `WaitForFirstConsumer` [volume binding
  mode](/docs/concepts/storage/storage-classes/#volume-binding-mode),
  and
- the `CSIDriver` object for the driver has `StorageCapacity` set to
  true.

In that case, the scheduler only considers nodes for the Pod which
have enough storage available to them. This check is very
simplistic and only compares the size of the volume against the
capacity listed in `CSIStorageCapacity` objects with a topology that
includes the node.

For volumes with `Immediate` volume binding mode, the storage driver
decides where to create the volume, independently of Pods that will
use the volume. The scheduler then schedules Pods onto nodes where the
volume is available after the volume has been created.

For [CSI ephemeral volumes](/docs/concepts/storage/volumes/#csi),
scheduling always happens without considering storage capacity. This
is based on the assumption that this volume type is only used by
special CSI drivers which are local to a node and do not need
significant resources there.
 -->

## 调度 {#scheduling}

k8s 调度器会使用存储容量信息如果:
- `CSIStorageCapacity` 功能阀启用
- 一个 Pod 使用了一个还没有被创建的卷
- 这卷使用一个 {{< glossary_tooltip text="StorageClass" term_id="storage-class" >}}
  它使用 CSI 驱动并设置 `WaitForFirstConsumer`
  [卷绑定模式](/k8sDocs/docs/concepts/storage/storage-classes/#volume-binding-mode)
- 驱动的 `CSIDriver` 对象上的 `StorageCapacity` 被设置为 true

在那种情况下，调度器只考虑那些对于该 Pod 有足够存储的节点。这种检测是相当简单的也就是只对卷的大小
和 包含节点拓扑的 `CSIStorageCapacity` 对象中列举的容量进行对比。

对于那些使用 `Immediate` 卷绑定模式的卷，由存储驱动决定在哪创建卷， Pod 会独立地使用这些卷。
然后调度器再把 Pod 调度到那些已经创建好卷的节点上。

对于
[CSI 临时卷](/k8sDocs/docs/concepts/storage/volumes/#csi),
在调度时始终不考虑存储容量的。 这么做是因为有这么一个假设: 这个卷类型只被一个在节点范围内，不
需要用到很多资源的特殊 CSI 驱动。
<!--
## Rescheduling

When a node has been selected for a Pod with `WaitForFirstConsumer`
volumes, that decision is still tentative. The next step is that the
CSI storage driver gets asked to create the volume with a hint that the
volume is supposed to be available on the selected node.

Because Kubernetes might have chosen a node based on out-dated
capacity information, it is possible that the volume cannot really be
created. The node selection is then reset and the Kubernetes scheduler
tries again to find a node for the Pod.
 -->

## 再调度 {#rescheduling}

当一个节点被一个使用 `WaitForFirstConsumer` 卷的 Pod 选中时，这个决策也是选择性的。 下一步
就是 CSI 存储驱动被告知选择的节点应该是有足够的空间来创建这相卷

因为 k8s 可能基于过期的容量信息选择一个节点，这就可能这个卷不能被真正创建。然后这次节点选择就
被重围，k8s 调度器再次尝试为 Pod 找一个节点。
<!--
## Limitations

Storage capacity tracking increases the chance that scheduling works
on the first try, but cannot guarantee this because the scheduler has
to decide based on potentially out-dated information. Usually, the
same retry mechanism as for scheduling without any storage capacity
information handles scheduling failures.

One situation where scheduling can fail permanently is when a Pod uses
multiple volumes: one volume might have been created already in a
topology segment which then does not have enough capacity left for
another volume. Manual intervention is necessary to recover from this,
for example by increasing capacity or deleting the volume that was
already created. [Further
work](https://github.com/kubernetes/enhancements/pull/1703) is needed
to handle this automatically.
 -->

## 限制 {#limitations}

存储容量的跟踪会增加调度在第一次调度时成功的机会，但不能保证，因为调度器可能是基于潜在的过期信息。
通常，这与没有容量信息调度时失败后的重试机制是一样的。

有一种情况是当一个 Pod 使用两个卷时调度可能会永久失： 一个卷可能已经在拓扑信息中创建，但这里
没有足够的空间来创建另一个卷。 这时间就需要人工介入来修复这种问题， 例如增加容量或删除已经创建的卷。
[剩下的工件](https://github.com/kubernetes/enhancements/pull/1703) 就会自动地被处理。
<!--
## Enabling storage capacity tracking

Storage capacity tracking is an *alpha feature* and only enabled when
the `CSIStorageCapacity` [feature
gate](/docs/reference/command-line-tools-reference/feature-gates/) and
the `storage.k8s.io/v1alpha1` {{< glossary_tooltip text="API group" term_id="api-group" >}} are enabled. For details on
that, see the `--feature-gates` and `--runtime-config` [kube-apiserver
parameters](/docs/reference/command-line-tools-reference/kube-apiserver/).

A quick check
whether a Kubernetes cluster supports the feature is to list
CSIStorageCapacity objects with:
```shell
kubectl get csistoragecapacities --all-namespaces
```

If your cluster supports CSIStorageCapacity, the response is either a list of CSIStorageCapacity objects or:
```
No resources found
```

If not supported, this error is printed instead:
```
error: the server doesn't have a resource type "csistoragecapacities"
```

In addition to enabling the feature in the cluster, a CSI
driver also has to
support it. Please refer to the driver's documentation for
details.
 -->

## 启用容量跟踪 {#enabling-storage-capacity-tracking}

存储容量跟踪是一个 *alpha 特性*， 只有在
{{< glossary_tooltip text="API group" term_id="api-group" >}}
中的 `storage.k8s.io/v1alpha1` 被启用，同时还启用了 `CSIStorageCapacity`
[功能阀](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)
更多相关信息，见
[kube-apiserver 参数](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
中的  `--feature-gates` 和 `--runtime-config`

快速检查 k8s 集群是否支持该特性的办法就是列举 `CSIStorageCapacity` 对象:
```shell
kubectl get csistoragecapacities --all-namespaces
```

如果集群支持 CSIStorageCapacity， 返回内容可能是一个 CSIStorageCapacity 列表或者:
```
No resources found
```

如果不支持则会出现以下错误:
```
error: the server doesn't have a resource type "csistoragecapacities"
```

另外，要在集群中启用该功能还需要所使用的 CSI 驱动也是支持。 具体请参阅驱动的文档。
## {{% heading "whatsnext" %}}
<!--
- For more information on the design, see the
[Storage Capacity Constraints for Pod Scheduling KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-storage/1472-storage-capacity-tracking/README.md).
- For more information on further development of this feature, see the [enhancement tracking issue #1472](https://github.com/kubernetes/enhancements/issues/1472).
- Learn about [Kubernetes Scheduler](/docs/concepts/scheduling-eviction/kube-scheduler/)
 -->
- 更多关于设计的信息见
  [Storage Capacity Constraints for Pod Scheduling KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-storage/1472-storage-capacity-tracking/README.md).
- 更多关于该特性的开发展望见
  [enhancement tracking issue #1472](https://github.com/kubernetes/enhancements/issues/1472).
- 概念学习 [k8s 调度器](/k8sDocs/docs/concepts/scheduling-eviction/kube-scheduler/)
