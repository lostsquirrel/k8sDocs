---
title: VolumeSnapshotClass
content_type: concept
weight: 60
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
title: Volume Snapshot Classes
content_type: concept
weight: 30
---
-->

<!-- overview -->
<!--
This document describes the concept of VolumeSnapshotClass in Kubernetes. Familiarity
with [volume snapshots](/docs/concepts/storage/volume-snapshots/) and
[storage classes](/docs/concepts/storage/storage-classes) is suggested.
-->

本文介绍 k8s 中的 VolumeSnapshotClass 概念。建议先熟悉
[卷快照](/k8sDocs/docs/concepts/storage/volume-snapshots/) 和
[StorageClass](/k8sDocs/docs/concepts/storage/storage-classes)

<!-- body -->
<!--
## Introduction

Just like StorageClass provides a way for administrators to describe the "classes"
of storage they offer when provisioning a volume, VolumeSnapshotClass provides a
way to describe the "classes" of storage when provisioning a volume snapshot.
 -->

<!--
## Introduction
Just like StorageClass provides a way for administrators to describe the "classes"
of storage they offer when provisioning a volume, VolumeSnapshotClass provides a
way to describe the "classes" of storage when provisioning a volume snapshot.
-->

## 介绍

与 StorageClass 为管理员提供了一种描述他们供应的存储的类别(class)一样。
VolumeSnapshotClass 提供了一种描述他们供应的卷快照的类别(class)

<!--
## The VolumeSnapshotClass Resource

Each VolumeSnapshotClass contains the fields `driver`, `deletionPolicy`, and `parameters`,
which are used when a VolumeSnapshot belonging to the class needs to be
dynamically provisioned.

The name of a VolumeSnapshotClass object is significant, and is how users can
request a particular class. Administrators set the name and other parameters
of a class when first creating VolumeSnapshotClass objects, and the objects cannot
be updated once they are created.

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-hostpath-snapclass
driver: hostpath.csi.k8s.io
deletionPolicy: Delete
parameters:
```

Administrators can specify a default VolumeSnapshotClass for VolumeSnapshots
that don't request any particular class to bind to by adding the
`snapshot.storage.kubernetes.io/is-default-class: "true"` annotation:

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-hostpath-snapclass
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: hostpath.csi.k8s.io
deletionPolicy: Delete
parameters:
```
 -->

## VolumeSnapshotClass 资源

每个 `VolumeSnapshotClass` 包含的字段有 `driver`, `deletionPolicy`, 和 `parameters`,
这个对象被属于该类别的 VolumeSnapshot 动态供应时用到。

VolumeSnapshotClass 对象的名称是有意义的，它是用户可以怎么申请一个指定的类别。 管理员在第一次创建
VolumeSnapshotClass 指定该类别的名称和其它参数，这个对象在创建之后将不能被修改。
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-hostpath-snapclass
driver: hostpath.csi.k8s.io
deletionPolicy: Delete
parameters:
```

管理员可以能过如下设置注解
`snapshot.storage.kubernetes.io/is-default-class: "true"`
为那些没有指定任何类别的 VolumeSnapshot 指定默认的 VolumeSnapshotClass，
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-hostpath-snapclass
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: hostpath.csi.k8s.io
deletionPolicy: Delete
parameters:
```
<!--
### Driver

Volume snapshot classes have a driver that determines what CSI volume plugin is
used for provisioning VolumeSnapshots. This field must be specified.
 -->

### 驱动(`driver`)

卷快照类别有一个 driver 字段来决定使用哪个 CSI 卷插件来供应卷快照(VolumeSnapshot).
这是一相必要字段。
<!--
### DeletionPolicy

Volume snapshot classes have a deletionPolicy. It enables you to configure what happens to a VolumeSnapshotContent when the VolumeSnapshot object it is bound to is to be deleted. The deletionPolicy of a volume snapshot can either be `Retain` or `Delete`. This field must be specified.

If the deletionPolicy is `Delete`, then the underlying storage snapshot will be deleted along with the VolumeSnapshotContent object. If the deletionPolicy is `Retain`, then both the underlying snapshot and VolumeSnapshotContent remain.
 -->

### 删除策略(`deletionPolicy`)

卷快照类别有一个 `deletionPolicy`。 它让用户可以配置当与 VolumeSnapshotContent 绑定
VolumeSnapshot 对象被删除时，怎么处理 VolumeSnapshotContent。 卷快照类别的删除策略(`deletionPolicy`)
可以是 `Retain` 或 `Delete`， 这是一个必要字段。

如果删除策略(`deletionPolicy`) 是 `Delete`，底层的存储快照会随同 VolumeSnapshotContent
对象的删除一起删除。 如果删除策略(`deletionPolicy`) 是 `Retain`，
底层的存储快照 和 VolumeSnapshotContent 都会保留。
<!--
## Parameters

Volume snapshot classes have parameters that describe volume snapshots belonging to
the volume snapshot class. Different parameters may be accepted depending on the
`driver`.
 -->
## 其它参数 (`parameters`)

卷快照类别有描述属于该类别的卷快照的参数。 可以接受哪些的参数基于不同的驱动 `driver`.
