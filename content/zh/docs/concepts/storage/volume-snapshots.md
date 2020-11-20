---
title: 卷快照
content_type: concept
weight: 30
---
<!--
---
reviewers:
- saad-ali
- thockin
- msau42
- jingxu97
- xing-yang
- yuxiangqian
title: Volume Snapshots
content_type: concept
weight: 20
---
-->

<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.17" state="beta" >}}
In Kubernetes, a _VolumeSnapshot_ represents a snapshot of a volume on a storage system. This document assumes that you are already familiar with Kubernetes [persistent volumes](/docs/concepts/storage/persistent-volumes/).
 -->

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

在 k8s 中， _VolumeSnapshot_ 代表存储系统中一个卷的快照。 在阅读此文之间需要熟悉 k8s
[持久化卷(PV)](/k8sDocs/docs/concepts/storage/persistent-volumes/).


<!-- body -->
<!--
## Introduction

Similar to how API resources `PersistentVolume` and `PersistentVolumeClaim` are used to provision volumes for users and administrators, `VolumeSnapshotContent` and `VolumeSnapshot` API resources are provided to create volume snapshots for users and administrators.

A `VolumeSnapshotContent` is a snapshot taken from a volume in the cluster that has been provisioned by an administrator. It is a resource in the cluster just like a PersistentVolume is a cluster resource.

A `VolumeSnapshot` is a request for snapshot of a volume by a user. It is similar to a PersistentVolumeClaim.

`VolumeSnapshotClass` allows you to specify different attributes belonging to a `VolumeSnapshot`. These attributes may differ among snapshots taken from the same volume on the storage system and therefore cannot be expressed by using the same `StorageClass` of a `PersistentVolumeClaim`.

Volume snapshots provide Kubernetes users with a standardized way to copy a volume's contents at a particular point in time without creating an entirely new volume. This functionality enables, for example, database administrators to backup databases before performing edit or delete modifications.

Users need to be aware of the following when using this feature:

* API Objects `VolumeSnapshot`, `VolumeSnapshotContent`, and `VolumeSnapshotClass` are {{< glossary_tooltip term_id="CustomResourceDefinition" text="CRDs" >}}, not part of the core API.
* `VolumeSnapshot` support is only available for CSI drivers.
* As part of the deployment process in the beta version of `VolumeSnapshot`, the Kubernetes team provides a snapshot controller to be deployed into the control plane, and a sidecar helper container called csi-snapshotter to be deployed together with the CSI driver.  The snapshot controller watches `VolumeSnapshot` and `VolumeSnapshotContent` objects and is responsible for the creation and deletion of `VolumeSnapshotContent` object in dynamic provisioning.  The sidecar csi-snapshotter watches `VolumeSnapshotContent` objects and triggers `CreateSnapshot` and `DeleteSnapshot` operations against a CSI endpoint.
* CSI drivers may or may not have implemented the volume snapshot functionality. The CSI drivers that have provided support for volume snapshot will likely use the csi-snapshotter. See [CSI Driver documentation](https://kubernetes-csi.github.io/docs/) for details.
* The CRDs and snapshot controller installations are the responsibility of the Kubernetes distribution.
 -->

## 介绍

与 `PersistentVolume` 和 `PersistentVolumeClaim` API 资源用来为用户和管理员管理卷相似，
`VolumeSnapshotContent` 和 `VolumeSnapshot` API 资源是用来为用户和管理员创建卷快照的。

一个 `VolumeSnapshotContent` 是一个由管理员管理的从集群中一个卷创建的快照。
它是一个与  `PersistentVolume` 类似的集群资源。

一个 `VolumeSnapshot` 是由用户申请的对一个卷的快照。 与 PVC 类似。

`VolumeSnapshotClass` 可以指定属于 `VolumeSnapshot` 的许多属性。 这些属性可能与其它来自
同一个卷打的快照不一样， 因此不能由使用同样 `StorageClass` 的 `PersistentVolumeClaim` 表现。

卷快照为 k8s 用户提供了一种拷贝特定时间点的卷内容，而不用创建新的卷的标准方式。 这个功能可以
用于数据库管理员在做变更或删除操作之前备份数据库。

用户在使用该特性时要注意以下几点:


- `VolumeSnapshot`, `VolumeSnapshotContent`, `VolumeSnapshotClass` API 对象是
  {{< glossary_tooltip term_id="CustomResourceDefinition" text="CRDs" >}}，
  不是核心 API 的对象。
- `VolumeSnapshot` 只支持 CSI 驱动。
- 作为 `VolumeSnapshot` beta 版本的部署里程， k8s 团队提供了一个部署到控制中心的快照控制器，
  和一个叫做 `csi-snapshotter` 的边车工具容器和 CSI 驱动部署在一起。 快照控制器监测
  `VolumeSnapshot` 和 `VolumeSnapshotContent` 对象，并负责动态管理的 `VolumeSnapshotContent`
  对象的创建和删除。 csi-snapshotter 边车，则监测 `VolumeSnapshotContent` 对象并触发
  到 CSI 接口的 `CreateSnapshot` 和 `DeleteSnapshot` 操作。
- CSI 驱动有可能实现也有可能没有实现快照功能。 提供了卷快照支持的 CSI 驱动通常会使用 `csi-snapshotter`。
  详细信息见 [CSI 驱动文档](https://kubernetes-csi.github.io/docs/)
- CRD 和 快照控制器的安装由 k8s 发行版本负责
<!--
## Lifecycle of a volume snapshot and volume snapshot content

`VolumeSnapshotContents` are resources in the cluster. `VolumeSnapshots` are requests for those resources. The interaction between `VolumeSnapshotContents` and `VolumeSnapshots` follow this lifecycle:
 -->

## `VolumeSnapshot` 和 `VolumeSnapshotContent` 的生命周期

`VolumeSnapshotContents` 是集群中的资源。 `VolumeSnapshots` 是对这些资源的申请。
`VolumeSnapshotContents` 与 `VolumeSnapshots` 交互遵循以下周期:
<!--
### Provisioning Volume Snapshot

There are two ways snapshots may be provisioned: pre-provisioned or dynamically provisioned.
 -->
### 卷快照管理

卷快照管理有两种方式: 静态管理 或 动态管理
<!--
#### Pre-provisioned {#static}
A cluster administrator creates a number of `VolumeSnapshotContents`. They carry the details of the real volume snapshot on the storage system which is available for use by cluster users. They exist in the Kubernetes API and are available for consumption.
 -->

#### 静态管理 {#static}

由集群管理员创建一定数据的 `VolumeSnapshotContents`。 其中包含存储系统中可以被集群用户使用的
真实卷快照的详细信息。 它们存在于 k8s API 并可被消费。
<!--
#### Dynamic
Instead of using a pre-existing snapshot, you can request that a snapshot to be dynamically taken from a PersistentVolumeClaim. The [VolumeSnapshotClass](/docs/concepts/storage/volume-snapshot-classes/) specifies storage provider-specific parameters to use when taking a snapshot.
 -->

#### 动态管理

与使用静态快照管理不同，可以申请从 PVC 动态创建快照。
[VolumeSnapshotClass](/k8sDocs/docs/concepts/storage/volume-snapshot-classes/) 可以在
做快照时指定存储提供者相关的参数
<!--
### Binding

The snapshot controller handles the binding of a `VolumeSnapshot` object with an appropriate `VolumeSnapshotContent` object, in both pre-provisioned and dynamically provisioned scenarios. The binding is a one-to-one mapping.

In the case of pre-provisioned binding, the VolumeSnapshot will remain unbound until the requested VolumeSnapshotContent object is created.
 -->

### 绑定

在静态管理和动态管理的情况下，快照控制器会处理 `VolumeSnapshot` 对象与恰当的 `VolumeSnapshotContent`
的绑定。 这种绑定关系是一对一的。

在静态管理绑定时， `VolumeSnapshot` 在 `VolumeSnapshotContent` 创建之前都是未绑定状态

<!--
### Persistent Volume Claim as Snapshot Source Protection

The purpose of this protection is to ensure that in-use
{{< glossary_tooltip text="PersistentVolumeClaim" term_id="persistent-volume-claim" >}}
API objects are not removed from the system while a snapshot is being taken from it (as this may result in data loss).

While a snapshot is being taken of a PersistentVolumeClaim, that PersistentVolumeClaim is in-use. If you delete a PersistentVolumeClaim API object in active use as a snapshot source, the PersistentVolumeClaim object is not removed immediately. Instead, removal of the PersistentVolumeClaim object is postponed until the snapshot is readyToUse or aborted.
 -->

### 当 PVC 作为快照源的保护机制

这个保护机制的目的是确保使用中的
{{< glossary_tooltip text="PersistentVolumeClaim" term_id="persistent-volume-claim" >}}
API 对象在做快照的过程中不会被从系统中删除(因为这可能会导致数据丢失)。

当一个 `PersistentVolumeClaim` 在用于做快照，则它就在使用中状态。 如果删除了一个正在作为快照
源的 `PersistentVolumeClaim` 对象，它不会马上被删除。 这个删除行为会被延迟到快照的状态
变为 `readyToUse` 或 中止(`aborted`) 之后。
<!--
### Delete

Deletion is triggered by deleting the `VolumeSnapshot` object, and the `DeletionPolicy` will be followed. If the `DeletionPolicy` is `Delete`, then the underlying storage snapshot will be deleted along with the `VolumeSnapshotContent` object. If the `DeletionPolicy` is `Retain`, then both the underlying snapshot and `VolumeSnapshotContent` remain.
 -->

### 删除

删除由删除 `VolumeSnapshot` 对象触发， 会依照 `DeletionPolicy` 进行。 如果 `DeletionPolicy`
是删除(`Delete`)， 则底层的存储快照会连同 `VolumeSnapshotContent` 对象一起删除。 如果
`DeletionPolicy` 是保留(`Retain`), 则 底层的存储快照 和 `VolumeSnapshotContent` 对象都保留
<!--
## VolumeSnapshots

Each VolumeSnapshot contains a spec and a status.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: new-snapshot-test
spec:
  volumeSnapshotClassName: csi-hostpath-snapclass
  source:
    persistentVolumeClaimName: pvc-test
```

`persistentVolumeClaimName` is the name of the PersistentVolumeClaim data source for the snapshot. This field is required for dynamically provisioning a snapshot.

A volume snapshot can request a particular class by specifying the name of a
[VolumeSnapshotClass](/docs/concepts/storage/volume-snapshot-classes/)
using the attribute `volumeSnapshotClassName`. If nothing is set, then the default class is used if available.

For pre-provisioned snapshots, you need to specify a `volumeSnapshotContentName` as the source for the snapshot as shown in the following example. The `volumeSnapshotContentName` source field is required for pre-provisioned snapshots.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: test-snapshot
spec:
  source:
    volumeSnapshotContentName: test-content
```
 -->

## `VolumeSnapshot`

每个 `VolumeSnapshot` 包含一个 `spec` 和一个 `status`

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: new-snapshot-test
spec:
  volumeSnapshotClassName: csi-hostpath-snapclass
  source:
    persistentVolumeClaimName: pvc-test
```

`persistentVolumeClaimName` is the name of the PersistentVolumeClaim data source for the snapshot. This field is required for dynamically provisioning a snapshot.
`persistentVolumeClaimName` 是作为快照数据源的 PVC 的名称。 这个字段在动态管理快照时是必要的。


A volume snapshot can request a particular class by specifying the name of a
[VolumeSnapshotClass](/docs/docs/concepts/storage/volume-snapshot-classes/)
using the attribute `volumeSnapshotClassName`. If nothing is set, then the default class is used if available.
卷快照时可以通过 `volumeSnapshotClassName` 设置一个
[VolumeSnapshotClass](/k8sDocs/docs/concepts/storage/volume-snapshot-classes/)
名称来指定特定的类别。
如果没有设置，在有默认的类别时就使用默认的

对于静态管理的快照，需要向下面例子中所示的指定一个 `volumeSnapshotContentName` 作为快照源。
在静态管理快照时 `volumeSnapshotContentName` 源字段是必要的。

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: test-snapshot
spec:
  source:
    volumeSnapshotContentName: test-content
```
<!--
## Volume Snapshot Contents

Each VolumeSnapshotContent contains a spec and status. In dynamic provisioning, the snapshot common controller creates `VolumeSnapshotContent` objects. Here is an example:

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotContent
metadata:
  name: snapcontent-72d9a349-aacd-42d2-a240-d775650d2455
spec:
  deletionPolicy: Delete
  driver: hostpath.csi.k8s.io
  source:
    volumeHandle: ee0cfb94-f8d4-11e9-b2d8-0242ac110002
  volumeSnapshotClassName: csi-hostpath-snapclass
  volumeSnapshotRef:
    name: new-snapshot-test
    namespace: default
    uid: 72d9a349-aacd-42d2-a240-d775650d2455
```

`volumeHandle` is the unique identifier of the volume created on the storage backend and returned by the CSI driver during the volume creation. This field is required for dynamically provisioning a snapshot. It specifies the volume source of the snapshot.

For pre-provisioned snapshots, you (as cluster administrator) are responsible for creating the `VolumeSnapshotContent` object as follows.

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotContent
metadata:
  name: new-snapshot-content-test
spec:
  deletionPolicy: Delete
  driver: hostpath.csi.k8s.io
  source:
    snapshotHandle: 7bdd0de3-aaeb-11e8-9aae-0242ac110002
  volumeSnapshotRef:
    name: new-snapshot-test
    namespace: default
```

`snapshotHandle` is the unique identifier of the volume snapshot created on the storage backend. This field is required for the pre-provisioned snapshots. It specifies the CSI snapshot id on the storage system that this `VolumeSnapshotContent` represents.
 -->

## `VolumeSnapshotContent`

每个 VolumeSnapshotContent 都包含一个 `spec` 和一个 `status`， 在动态管理时，
快照控制器创建 `VolumeSnapshotContent` 对象，下面是一个示例:

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotContent
metadata:
  name: snapcontent-72d9a349-aacd-42d2-a240-d775650d2455
spec:
  deletionPolicy: Delete
  driver: hostpath.csi.k8s.io
  source:
    volumeHandle: ee0cfb94-f8d4-11e9-b2d8-0242ac110002
  volumeSnapshotClassName: csi-hostpath-snapclass
  volumeSnapshotRef:
    name: new-snapshot-test
    namespace: default
    uid: 72d9a349-aacd-42d2-a240-d775650d2455
```

`volumeHandle` 是卷在存储后台上的唯一标识，由 CSI 创建卷时返回。 这个字段在动态管理快照时是必要的。
它指定了快照的源(卷)

对于静态管理的乜是， 集群管理员负责创建如下 `VolumeSnapshotContent` 对象。
```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotContent
metadata:
  name: new-snapshot-content-test
spec:
  deletionPolicy: Delete
  driver: hostpath.csi.k8s.io
  source:
    snapshotHandle: 7bdd0de3-aaeb-11e8-9aae-0242ac110002
  volumeSnapshotRef:
    name: new-snapshot-test
    namespace: default
```

`volumeHandle` 是卷在存储后台上的唯一标识， 这个字段在静态管理快照时是必要的。 它指定这个
`VolumeSnapshotContent` 在存储系统所代表的 CSI 快照的ID
<!--
## Provisioning Volumes from Snapshots

You can provision a new volume, pre-populated with data from a snapshot, by using
the *dataSource* field in the `PersistentVolumeClaim` object.

For more details, see
[Volume Snapshot and Restore Volume from Snapshot](/docs/concepts/storage/persistent-volumes/#volume-snapshot-and-restore-volume-from-snapshot-support).
 -->
## 基于快照创建卷

You can provision a new volume, pre-populated with data from a snapshot, by using
the *dataSource* field in the `PersistentVolumeClaim` object.
可以通过 `PersistentVolumeClaim` 对象的 *dataSource* 指定一个快照，将快照的数据预先添加
到新创建的卷中。

更多详细信息见
[卷快照和从快照中恢复卷](/k8sDocs/docs/concepts/storage/persistent-volumes/#volume-snapshot-and-restore-volume-from-snapshot-support).
