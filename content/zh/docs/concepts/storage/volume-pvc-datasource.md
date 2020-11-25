---
title: CSI 卷克隆
content_type: concept
weight: 40
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- thockin
- msau42
title: CSI Volume Cloning
content_type: concept
weight: 30
---
 -->
<!-- overview -->
<!--
This document describes the concept of cloning existing CSI Volumes in Kubernetes.  Familiarity with [Volumes](/docs/concepts/storage/volumes) is suggested.
 -->

本文主要介绍在 k8s 中克隆已有的 CSI 卷的概念. 建议先熟悉
[Volumes](/k8sDocs/docs/concepts/storage/volumes)

<!-- body -->
<!--
## Introduction

The {{< glossary_tooltip text="CSI" term_id="csi" >}} Volume Cloning feature adds support for specifying existing {{< glossary_tooltip text="PVC" term_id="persistent-volume-claim" >}}s in the `dataSource` field to indicate a user would like to clone a {{< glossary_tooltip term_id="volume" >}}.

A Clone is defined as a duplicate of an existing Kubernetes Volume that can be consumed as any standard Volume would be.  The only difference is that upon provisioning, rather than creating a "new" empty Volume, the back end device creates an exact duplicate of the specified Volume.

The implementation of cloning, from the perspective of the Kubernetes API, simply adds the ability to specify an existing PVC as a dataSource during new PVC creation. The source PVC must be bound and available (not in use).

Users need to be aware of the following when using this feature:

* Cloning support (`VolumePVCDataSource`) is only available for CSI drivers.
* Cloning support is only available for dynamic provisioners.
* CSI drivers may or may not have implemented the volume cloning functionality.
* You can only clone a PVC when it exists in the same namespace as the destination PVC (source and destination must be in the same namespace).
* Cloning is only supported within the same Storage Class.
    - Destination volume must be the same storage class as the source
    - Default storage class can be used and storageClassName omitted in the spec
* Cloning can only be performed between two volumes that use the same VolumeMode setting (if you request a block mode volume, the source MUST also be block mode)
 -->

## 介绍

{{< glossary_tooltip text="CSI" term_id="csi" >}} 卷克隆的特性。它支持在 `dataSource`
字段指定现有的
{{< glossary_tooltip text="PVC" term_id="persistent-volume-claim" >}}
表示用户希望克隆一个
{{< glossary_tooltip term_id="volume" >}}

这个克隆体是对现有的 k8s 卷的复制，可以用于任意标准卷可用的地方。 区别只有提供时不是创建一个新
的空卷，而对源卷底层存储设备完全复制的卷

以 k8s API 的角度来看，克隆的实现就是，在创建新的 PVC 时，可以在 `dataSource` 字段指定一个
现有的 PVC 作为数据源。 源 PVC 必须绑定且可用(不能在使用中)

用户在使用该特性时需要注意以下几点:

- 只有 CSI 驱动才支持克隆 (`VolumePVCDataSource`)
- 只支持在动态供应中使用克隆
- CSI 驱动可能实现或可能没实现克隆功能
- 只有源 PVC 必须与目标 PVC 在同一个命名空间才能克隆(源和目标必须在一个命名空间)
- 克隆只能发生一同一个存储类别中(StorageClass)
  - 目标卷的 存储类别必须与源相同
  - 当定义中没有指定 `storageClassName` 时使用默认存储类别(StorageClass)
- 克隆只能在两个相同卷类别(VolumeMode)的卷之间操作(如果请求的是一个块设备卷，源就必须是一个块设备卷)
<!--
## Provisioning

Clones are provisioned just like any other PVC with the exception of adding a dataSource that references an existing PVC in the same namespace.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: clone-of-pvc-1
    namespace: myns
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: cloning
  resources:
    requests:
      storage: 5Gi
  dataSource:
    kind: PersistentVolumeClaim
    name: pvc-1
```

{{< note >}}
You must specify a capacity value for `spec.resources.requests.storage`, and the value you specify must be the same or larger than the capacity of the source volume.
{{< /note >}}

The result is a new PVC with the name `clone-of-pvc-1` that has the exact same content as the specified source `pvc-1`.
 -->

## 供应

 克隆体除了在 `dataSource` 指定一个同一命名空间的 PVC 外与任意其它的 PVC 的供应完全相同，

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: clone-of-pvc-1
    namespace: myns
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: cloning
  resources:
    requests:
      storage: 5Gi
  dataSource:
    kind: PersistentVolumeClaim
    name: pvc-1
```

{{< note >}}
必须要为 `spec.resources.requests.storage` 指定一个值，并且这个值必须大于或等于源卷的值
{{< /note >}}

最终结果就是新创建一个叫  `clone-of-pvc-1` 的PVC 其内容与它的源 `pvc-1` 完全一样。
<!--
## Usage

Upon availability of the new PVC, the cloned PVC is consumed the same as other PVC.  It's also expected at this point that the newly created PVC is an independent object.  It can be consumed, cloned, snapshotted, or deleted independently and without consideration for it's original dataSource PVC.  This also implies that the source is not linked in any way to the newly created clone, it may also be modified or deleted without affecting the newly created clone.
 -->
## 使用

在可用性上，克隆体的 PVC 与其它的 PVC 一样。 新创建的 PVC 是一个独立的对象。 它可以消费，克隆，创建快照，
并在不影响它源 PVC 的情况下独立的删除。 这就是说克隆的源和目标之间没有任意形式的链接，对源的
修改或删除也不会影响它克隆体
