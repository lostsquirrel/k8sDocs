---
title: StorageClass
content_type: concept
weight: 50
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- thockin
- msau42
title: Storage Classes
content_type: concept
weight: 30
---
 -->
<!-- overview -->
<!--
This document describes the concept of a StorageClass in Kubernetes. Familiarity
with [volumes](/docs/concepts/storage/volumes/) and
[persistent volumes](/docs/concepts/storage/persistent-volumes) is suggested.
 -->

本文介绍 k8s 中的 StorageClass 这个概念。 建议先熟悉
[卷(Volume)](/k8sDocs/docs/concepts/storage/volumes/)
和
[持久化卷(PV)](/k8sDocs/docs/concepts/storage/persistent-volumes)
<!-- body -->
<!--
## Introduction

A StorageClass provides a way for administrators to describe the "classes" of
storage they offer. Different classes might map to quality-of-service levels,
or to backup policies, or to arbitrary policies determined by the cluster
administrators. Kubernetes itself is unopinionated about what classes
represent. This concept is sometimes called "profiles" in other storage
systems.
-->

## 介绍

StorageClass 为管理员提供了一种描述不同存储类别的方式. 不同的类别与 服务质量(QoS)级别，备份策略，
或其它由管理决定的任意策略关联。k8s 本身没不固化出现的类别。 这个概念在某些存储系统中被称为 "profiles"
<!--
## The StorageClass Resource

Each StorageClass contains the fields `provisioner`, `parameters`, and
`reclaimPolicy`, which are used when a PersistentVolume belonging to the
class needs to be dynamically provisioned.

The name of a StorageClass object is significant, and is how users can
request a particular class. Administrators set the name and other parameters
of a class when first creating StorageClass objects, and the objects cannot
be updated once they are created.

Administrators can specify a default StorageClass just for PVCs that don't
request any particular class to bind to: see the
[PersistentVolumeClaim section](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
for details.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```
 -->

## `StorageClass` 资源

每个 `StorageClass` 对象包含 `provisioner`, `parameters`, `reclaimPolicy` 字段，
当有被该类别的 PV 需要被动态供给时会用到。

`StorageClass` 对象的名称是有意义的，它会被用户在申请该类别存储时用到。 管理员在第一次创建
`StorageClass` 对象时设置名称和其它的参数， 这些对象在创建后将不可修改。

管理员可以指定一个默认的 `StorageClass`，那些没有指定类别的 PVC 就会使用这个默认的类别:
详细信息见
[PersistentVolumeClaim 章节](/k8sDocs/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```
<!--
### Provisioner

Each StorageClass has a provisioner that determines what volume plugin is used
for provisioning PVs. This field must be specified.

| Volume Plugin        | Internal Provisioner| Config Example                       |
| :---                 |     :---:           |    :---:                             |
| AWSElasticBlockStore | &#x2713;            | [AWS EBS](#aws-ebs)                          |
| AzureFile            | &#x2713;            | [Azure File](#azure-file)            |
| AzureDisk            | &#x2713;            | [Azure Disk](#azure-disk)            |
| CephFS               | -                   | -                                    |
| Cinder               | &#x2713;            | [OpenStack Cinder](#openstack-cinder)|
| FC                   | -                   | -                                    |
| FlexVolume           | -                   | -                                    |
| Flocker              | &#x2713;            | -                                    |
| GCEPersistentDisk    | &#x2713;            | [GCE PD](#gce-pd)                          |
| Glusterfs            | &#x2713;            | [Glusterfs](#glusterfs)              |
| iSCSI                | -                   | -                                    |
| Quobyte              | &#x2713;            | [Quobyte](#quobyte)                  |
| NFS                  | -                   | -                                    |
| RBD                  | &#x2713;            | [Ceph RBD](#ceph-rbd)                |
| VsphereVolume        | &#x2713;            | [vSphere](#vsphere)                  |
| PortworxVolume       | &#x2713;            | [Portworx Volume](#portworx-volume)  |
| ScaleIO              | &#x2713;            | [ScaleIO](#scaleio)                  |
| StorageOS            | &#x2713;            | [StorageOS](#storageos)              |
| Local                | -                   | [Local](#local)              |

You are not restricted to specifying the "internal" provisioners
listed here (whose names are prefixed with "kubernetes.io" and shipped
alongside Kubernetes). You can also run and specify external provisioners,
which are independent programs that follow a [specification](https://git.k8s.io/community/contributors/design-proposals/storage/volume-provisioning.md)
defined by Kubernetes. Authors of external provisioners have full discretion
over where their code lives, how the provisioner is shipped, how it needs to be
run, what volume plugin it uses (including Flex), etc. The repository
[kubernetes-sigs/sig-storage-lib-external-provisioner](https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner)
houses a library for writing external provisioners that implements the bulk of
the specification. Some external provisioners are listed under the repository
[kubernetes-sigs/sig-storage-lib-external-provisioner](https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner).

For example, NFS doesn't provide an internal provisioner, but an external
provisioner can be used. There are also cases when 3rd party storage
vendors provide their own external provisioner.
 -->

### 供应者(Provisioner)

每个 `StorageClass` 都有一个供应者，这个供应者决定供给 PV 的卷插件。 这个字段必须指定。

| Volume Plugin        | Internal Provisioner| Config Example                       |
| :---                 |     :---:           |    :---:                             |
| AWSElasticBlockStore | &#x2713;            | [AWS EBS](#aws-ebs)                          |
| AzureFile            | &#x2713;            | [Azure File](#azure-file)            |
| AzureDisk            | &#x2713;            | [Azure Disk](#azure-disk)            |
| CephFS               | -                   | -                                    |
| Cinder               | &#x2713;            | [OpenStack Cinder](#openstack-cinder)|
| FC                   | -                   | -                                    |
| FlexVolume           | -                   | -                                    |
| Flocker              | &#x2713;            | -                                    |
| GCEPersistentDisk    | &#x2713;            | [GCE PD](#gce-pd)                          |
| Glusterfs            | &#x2713;            | [Glusterfs](#glusterfs)              |
| iSCSI                | -                   | -                                    |
| Quobyte              | &#x2713;            | [Quobyte](#quobyte)                  |
| NFS                  | -                   | -                                    |
| RBD                  | &#x2713;            | [Ceph RBD](#ceph-rbd)                |
| VsphereVolume        | &#x2713;            | [vSphere](#vsphere)                  |
| PortworxVolume       | &#x2713;            | [Portworx Volume](#portworx-volume)  |
| ScaleIO              | &#x2713;            | [ScaleIO](#scaleio)                  |
| StorageOS            | &#x2713;            | [StorageOS](#storageos)              |
| Local                | -                   | [Local](#local)              |

用户并不仅限于上面列举的"内部"供应者(这些名称以 "kubernetes.io" 前缀的是随同 k8s 发行一起的)。
也可以运行和指定外部的供应者，这些是依照由 k8s 定义的
[specification](https://git.k8s.io/community/contributors/design-proposals/storage/volume-provisioning.md)
独立程序。 外部供应者的开发者可以完全自主地决定代码存入在哪， 供应者程序是什么发布的， 运行需要什么，
使用什么卷插件(包括 Flex)，等等。
[kubernetes-sigs/sig-storage-lib-external-provisioner](https://github.com/kubernetes-sigs/sig-storage-lib-external-provisioner)
仓库中包含了编写外部供应者需要实现的一系列规格说明。 一些外部提供也列举在这个仓库中

例如， NFS 没有提供内部的供应者，就可以使用一个外部的供应都。 还有种情况是第三方存储提供都
也会提供自己的外部供应者。
<!--
### Reclaim Policy

PersistentVolumes that are dynamically created by a StorageClass will have the
reclaim policy specified in the `reclaimPolicy` field of the class, which can be
either `Delete` or `Retain`. If no `reclaimPolicy` is specified when a
StorageClass object is created, it will default to `Delete`.

PersistentVolumes that are created manually and managed via a StorageClass will have
whatever reclaim policy they were assigned at creation.
 -->

### 回收策略

由 `StorageClass` 动态创建的持久化卷(PV)将通过 `StorageClass` 的 `reclaimPolicy` 字段
设备回收策略，这些策略可以是 `Delete` 或 `Retain`。 如果在创建 `StorageClass` 对象的时候
没有指定 `reclaimPolicy`， 默认回收策略为 `Delete`

由手动创建并通过 `StorageClass` 管理的 持久化卷(PV) 会在创建的时候指定回收策略
<!--
### Allow Volume Expansion

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

PersistentVolumes can be configured to be expandable. This feature when set to `true`,
allows the users to resize the volume by editing the corresponding PVC object.

The following types of volumes support volume expansion, when the underlying
StorageClass has the field `allowVolumeExpansion` set to true.

{{< table caption = "Table of Volume types and the version of Kubernetes they require"  >}}

Volume type | Required Kubernetes version
:---------- | :--------------------------
gcePersistentDisk | 1.11
awsElasticBlockStore | 1.11
Cinder | 1.11
glusterfs | 1.11
rbd | 1.11
Azure File | 1.11
Azure Disk | 1.11
Portworx | 1.11
FlexVolume | 1.13
CSI | 1.14 (alpha), 1.16 (beta)

{{< /table >}}


{{< note >}}
You can only use the volume expansion feature to grow a Volume, not to shrink it.
{{< /note >}}
 -->

### 允许卷扩容

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

持久化卷(PV) 可以设置为可扩展的。当开启该特性后允许用户通过修改对应 PVC 对象的方式修改卷的容量。

以下类型的卷在底层 `StorageClass` 的 `allowVolumeExpansion` 字段设置为 `true`,时支持卷扩展。

{{< table caption = "Table of Volume types and the version of Kubernetes they require"  >}}

卷类型       | 需要的 k8s 版本
:---------- | :--------------------------
gcePersistentDisk | 1.11
awsElasticBlockStore | 1.11
Cinder | 1.11
glusterfs | 1.11
rbd | 1.11
Azure File | 1.11
Azure Disk | 1.11
Portworx | 1.11
FlexVolume | 1.13
CSI | 1.14 (alpha), 1.16 (beta)

{{< /table >}}


{{< note >}}
只能使用卷扩展特性扩充卷，不能缩小
{{< /note >}}
<!--
### Mount Options

PersistentVolumes that are dynamically created by a StorageClass will have the
mount options specified in the `mountOptions` field of the class.

If the volume plugin does not support mount options but mount options are
specified, provisioning will fail. Mount options are not validated on either
the class or PV, so mount of the PV will simply fail if one is invalid.
 -->

### 挂载选项

由 `StorageClass` 动态创建的持久化卷(PV)会拥有由 `StorageClass` `mountOptions` 字段指定
的挂载选项。

如果卷插件不支持挂载选项但又指定了挂载选项，供应就会失败。 `StorageClass` 或 PV 挂载选项
不是有效的， 如果其中有一个无效则挂载就会失败。
<!--
### Volume Binding Mode

The `volumeBindingMode` field controls when [volume binding and dynamic
provisioning](/docs/concepts/storage/persistent-volumes/#provisioning) should occur.

By default, the `Immediate` mode indicates that volume binding and dynamic
provisioning occurs once the PersistentVolumeClaim is created. For storage
backends that are topology-constrained and not globally accessible from all Nodes
in the cluster, PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling
requirements. This may result in unschedulable Pods.

A cluster administrator can address this issue by specifying the `WaitForFirstConsumer` mode which
will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.
PersistentVolumes will be selected or provisioned conforming to the topology that is
specified by the Pod's scheduling constraints. These include, but are not limited to, [resource
requirements](/docs/concepts/configuration/manage-resources-containers/),
[node selectors](/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector),
[pod affinity and
anti-affinity](/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity),
and [taints and tolerations](/docs/concepts/scheduling-eviction/taint-and-toleration).

The following plugins support `WaitForFirstConsumer` with dynamic provisioning:

* [AWSElasticBlockStore](#aws-ebs)
* [GCEPersistentDisk](#gce-pd)
* [AzureDisk](#azure-disk)

The following plugins support `WaitForFirstConsumer` with pre-created PersistentVolume binding:

* All of the above
* [Local](#local)

{{< feature-state state="stable" for_k8s_version="v1.17" >}}
[CSI volumes](/docs/concepts/storage/volumes/#csi) are also supported with dynamic provisioning
and pre-created PVs, but you'll need to look at the documentation for a specific CSI driver
to see its supported topology keys and examples.
 -->

### 卷绑定模式

当
[卷绑定和动态供应](/docs/concepts/storage/persistent-volumes/#provisioning)
发生时由 `volumeBindingMode` 字段控制。

默认情况下使用的是 `Immediate` 模式，这种模式表示在 PersistentVolumeClaim 对象创建后立即
进行卷绑定和动态供应。对于有拓扑限制的存储后台和并不是集群中所有节点都可以访问的存储时，
持久化卷(PV) 会在不知道 Pod 调度要求的情况下绑定或供应。这可能会导致 Pod 不可调度。

要避免这个问题，管理员可以将卷模式设置为 `WaitForFirstConsumer`，这样持久化卷(PV)的绑定和
供应会推迟到使用这个 PVC 的 Pod 创建之后。此时持久化卷(PV)在供应时会确认由 Pod 指定的调度约束。
这些限制包括但不限于
[资源需求](/k8sDocs/docs/concepts/configuration/manage-resources-containers/),
[节点选择器](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector),
[Pod 亲和性和反亲和性](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity),
[毒点和耐受](/k8sDocs/docs/concepts/scheduling-eviction/taint-and-toleration).

以下插件支持带动态供应的 `WaitForFirstConsumer`:

* [AWSElasticBlockStore](#aws-ebs)
* [GCEPersistentDisk](#gce-pd)
* [AzureDisk](#azure-disk)

以下插件支持在预先创建 持久化卷(PV)绑定的 `WaitForFirstConsumer`

* 上面所有
* [Local](#local)

{{< feature-state state="stable" for_k8s_version="v1.17" >}}

[CSI 卷](/docs/concepts/storage/volumes/#csi) 也支持动态供应和预创建 PV，但需要先
查看对应 CSI 驱动的文档，看看支持的拓扑键和示例。
<!--
### Allowed Topologies

When a cluster operator specifies the `WaitForFirstConsumer` volume binding mode, it is no longer necessary
to restrict provisioning to specific topologies in most situations. However,
if still required, `allowedTopologies` can be specified.

This example demonstrates how to restrict the topology of provisioned volumes to specific
zones and should be used as a replacement for the `zone` and `zones` parameters for the
supported plugins.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - us-central1-a
    - us-central1-b
```
 -->
### 允许的拓扑 {#allowed-topologies}

当集群中的卷绑定模式被设置为 `WaitForFirstConsumer` 时，在大多数情况下就不供应严格受限于
指定拓扑。 但如果仍然需要这些限制，可以通过 `allowedTopologies` 指定。

以下的示例中展示的是怎么通过设置区域来限制供应的卷拓扑，如果插件支持，这些限制会用来替换插件中的 `zone` 和 `zones` 参数
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - us-central1-a
    - us-central1-b
```
<!--
## Parameters

Storage Classes have parameters that describe volumes belonging to the storage
class. Different parameters may be accepted depending on the `provisioner`. For
 example, the value `io1`, for the parameter `type`, and the parameter
`iopsPerGB` are specific to EBS. When a parameter is omitted, some default is
used.

There can be at most 512 parameters defined for a StorageClass.
The total length of the parameters object including its keys and values cannot
exceed 256 KiB.
 -->

## 参数

`StorageClass` 有些参数，这些参数描述属于该存储类别的卷。 基于不同的 `provisioner` 可以接受
不同的参数。 例如， 对于 `type` 的参数值为 `io1`， `iopsPerGB` 参数的值为 `EBS`。 当一个参数
没有设置时，就会使用默认值。

对于每个 `StorageClass` 最多可以定义 `512` 个参数。参数对象的总长度，包含其键和值不能超过
256 KiB
<!--
### AWS EBS

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

* `type`: `io1`, `gp2`, `sc1`, `st1`. See
  [AWS docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)
  for details. Default: `gp2`.
* `zone` (Deprecated): AWS zone. If neither `zone` nor `zones` is specified, volumes are
  generally round-robin-ed across all active zones where Kubernetes cluster
  has a node. `zone` and `zones` parameters must not be used at the same time.
* `zones` (Deprecated): A comma separated list of AWS zone(s). If neither `zone` nor `zones`
  is specified, volumes are generally round-robin-ed across all active zones
  where Kubernetes cluster has a node. `zone` and `zones` parameters must not
  be used at the same time.
* `iopsPerGB`: only for `io1` volumes. I/O operations per second per GiB. AWS
  volume plugin multiplies this with size of requested volume to compute IOPS
  of the volume and caps it at 20 000 IOPS (maximum supported by AWS, see
  [AWS docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html).
  A string is expected here, i.e. `"10"`, not `10`.
* `fsType`: fsType that is supported by kubernetes. Default: `"ext4"`.
* `encrypted`: denotes whether the EBS volume should be encrypted or not.
  Valid values are `"true"` or `"false"`. A string is expected here,
  i.e. `"true"`, not `true`.
* `kmsKeyId`: optional. The full Amazon Resource Name of the key to use when
  encrypting the volume. If none is supplied but `encrypted` is true, a key is
  generated by AWS. See AWS docs for valid ARN value.

{{< note >}}
`zone` and `zones` parameters are deprecated and replaced with
[allowedTopologies](#allowed-topologies)
{{< /note >}}
 -->

### AWS EBS {#aws-ebs}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

- `type`: `io1`, `gp2`, `sc1`, `st1`. 默认: `gp2`
  详细信息见 [AWS 文档](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)

- `zone` (废弃): AWS 区域。如果 `zone` 和 `zones` 都没有设置， 卷会在 k8s 集群中所有有节点的
  活跃区别之间随机调度

- `zones` (废弃): 一个用逗号分隔的 AWS 区域列表。 如果没有设置，卷会在 k8s 集群中所有有节点的
  活跃区别之间随机调度。 `zone` 和 `zones` 参数一定不要同时使用。

- `iopsPerGB`: 仅限 `io1` 卷。每秒每GiB I/O 操作数。AWS 会将这个值乘以申请的卷大小得出
  卷的 IOPS， 最高为 20 000 IOPS (AWS 支持的最大值, 见
  [AWS 文档](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html).
  这个的值是一个字段串，也就是这样 `"10"`， 而不是 `10`.
- `fsType`: k8s 支持的文件系统类型。 默认: `"ext4"`.

- `encrypted`: 表示这个 EBS 卷是否使用加密。 有效的值是 `"true"` 或 `"false"`
  这里的值也是字符串，也就是 `"true"`, 而不是 `true`.

- `kmsKeyId`: 可选。 在对卷加密时且的亚马逊资源全名的键。 如果这个值没提供但 `encrypted`
  设置为 "true", AWS 就会生成一个键。 关于有效的 ARN 值见 AWS 文档。

{{< note >}}
`zone` 和 `zones` 参数已经废弃，被 [允许的拓扑](#allowed-topologies) 替换
{{< /note >}}
<!--
### GCE PD

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  fstype: ext4
  replication-type: none
```

* `type`: `pd-standard` or `pd-ssd`. Default: `pd-standard`
* `zone` (Deprecated): GCE zone. If neither `zone` nor `zones` is specified, volumes are
  generally round-robin-ed across all active zones where Kubernetes cluster has
  a node. `zone` and `zones` parameters must not be used at the same time.
* `zones` (Deprecated): A comma separated list of GCE zone(s). If neither `zone` nor `zones`
  is specified, volumes are generally round-robin-ed across all active zones
  where Kubernetes cluster has a node. `zone` and `zones` parameters must not
  be used at the same time.
* `fstype`: `ext4` or `xfs`. Default: `ext4`. The defined filesystem type must be supported by the host operating system.

* `replication-type`: `none` or `regional-pd`. Default: `none`.

If `replication-type` is set to `none`, a regular (zonal) PD will be provisioned.

If `replication-type` is set to `regional-pd`, a
[Regional Persistent Disk](https://cloud.google.com/compute/docs/disks/#repds)
will be provisioned. It's highly recommended to have
`volumeBindingMode: WaitForFirstConsumer` set, in which case when you create
a Pod that consumes a PersistentVolumeClaim which uses this StorageClass, a
Regional Persistent Disk is provisioned with two zones. One zone is the same
as the zone that the Pod is scheduled in. The other zone is randomly picked
from the zones available to the cluster. Disk zones can be further constrained
using `allowedTopologies`.

{{< note >}}
`zone` and `zones` parameters are deprecated and replaced with
[allowedTopologies](#allowed-topologies)
{{< /note >}}
 -->

### GCE PD {#gce-pd}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  fstype: ext4
  replication-type: none
```

- `type`: `pd-standard` 或 `pd-ssd`. 默认: `pd-standard`

- `zone` (废弃): GCE 区域. 如果 `zone` 和 `zones` 都没有设置， 卷会在 k8s 集群中所有有节点的
  活跃区别之间随机调度， `zone` 和 `zones` 参数一定不要同时使用。

- `zones` (废弃): 一个用逗号分隔的 GCE 区域列表。 如果没有设置，卷会在 k8s 集群中所有有节点的
  活跃区别之间随机调度。 `zone` 和 `zones` 参数一定不要同时使用。`zone` 和 `zones` 参数一定不要同时使用。
- `fstype`: `ext4` 或 `xfs`. 默认: `ext4`. 定义的文件系统类型必须被主机操作系统支持
- `replication-type`: `none` 或 `regional-pd`. 默认: `none`.

如果 `replication-type` 设置为 `none`， 会供应一个常规的 PD。

如果 `replication-type` 设置为 `regional-pd`， 会供应一个
[Regional Persistent Disk](https://cloud.google.com/compute/docs/disks/#repds)
强烈推荐同时设置 `volumeBindingMode: WaitForFirstConsumer`。 在这种情况下，当创建一个
消费使用该 `StorageClass` 的 PVC 时， 会供应两个区域持久化盘。 一个区域与 Pod 调度的区域相同，
另一个则随机到集群中其它的可用区域。 硬盘区域还可以使用  `allowedTopologies` 添加更多多限制。

{{< note >}}
`zone` 和 `zones` 参数已经废弃，被 [允许的拓扑](#allowed-topologies) 替换
{{< /note >}}
<!--
### Glusterfs

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://127.0.0.1:8081"
  clusterid: "630372ccdc720a92c681fb928f27b53f"
  restauthenabled: "true"
  restuser: "admin"
  secretNamespace: "default"
  secretName: "heketi-secret"
  gidMin: "40000"
  gidMax: "50000"
  volumetype: "replicate:3"
```

* `resturl`: Gluster REST service/Heketi service url which provision gluster
  volumes on demand. The general format should be `IPaddress:Port` and this is
  a mandatory parameter for GlusterFS dynamic provisioner. If Heketi service is
  exposed as a routable service in openshift/kubernetes setup, this can have a
  format similar to `http://heketi-storage-project.cloudapps.mystorage.com`
  where the fqdn is a resolvable Heketi service url.
* `restauthenabled` : Gluster REST service authentication boolean that enables
  authentication to the REST server. If this value is `"true"`, `restuser` and
  `restuserkey` or `secretNamespace` + `secretName` have to be filled. This
  option is deprecated, authentication is enabled when any of `restuser`,
  `restuserkey`, `secretName` or `secretNamespace` is specified.
* `restuser` : Gluster REST service/Heketi user who has access to create volumes
  in the Gluster Trusted Pool.
* `restuserkey` : Gluster REST service/Heketi user's password which will be used
  for authentication to the REST server. This parameter is deprecated in favor
  of `secretNamespace` + `secretName`.
* `secretNamespace`, `secretName` : Identification of Secret instance that
  contains user password to use when talking to Gluster REST service. These
  parameters are optional, empty password will be used when both
  `secretNamespace` and `secretName` are omitted. The provided secret must have
  type `"kubernetes.io/glusterfs"`, for example created in this way:

    ```
    kubectl create secret generic heketi-secret \
      --type="kubernetes.io/glusterfs" --from-literal=key='opensesame' \
      --namespace=default
    ```

    Example of a secret can be found in
    [glusterfs-provisioning-secret.yaml](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/glusterfs/glusterfs-secret.yaml).

* `clusterid`: `630372ccdc720a92c681fb928f27b53f` is the ID of the cluster
  which will be used by Heketi when provisioning the volume. It can also be a
  list of clusterids, for example:
  `"8452344e2becec931ece4e33c4674e4e,42982310de6c63381718ccfa6d8cf397"`. This
  is an optional parameter.
* `gidMin`, `gidMax` : The minimum and maximum value of GID range for the
  StorageClass. A unique value (GID) in this range ( gidMin-gidMax ) will be
  used for dynamically provisioned volumes. These are optional values. If not
  specified, the volume will be provisioned with a value between 2000-2147483647
  which are defaults for gidMin and gidMax respectively.
* `volumetype` : The volume type and its parameters can be configured with this
  optional value. If the volume type is not mentioned, it's up to the provisioner
  to decide the volume type.

    For example:
    * Replica volume: `volumetype: replicate:3` where '3' is replica count.
    * Disperse/EC volume: `volumetype: disperse:4:2` where '4' is data and '2' is the redundancy count.
    * Distribute volume: `volumetype: none`

    For available volume types and administration options, refer to the
    [Administration Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Storage/3.1/html/Administration_Guide/part-Overview.html).

    For further reference information, see
    [How to configure Heketi](https://github.com/heketi/heketi/wiki/Setting-up-the-topology).

    When persistent volumes are dynamically provisioned, the Gluster plugin
    automatically creates an endpoint and a headless service in the name
    `gluster-dynamic-<claimname>`. The dynamic endpoint and service are automatically
    deleted when the persistent volume claim is deleted.
 -->

### Glusterfs {#glusterfs}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://127.0.0.1:8081"
  clusterid: "630372ccdc720a92c681fb928f27b53f"
  restauthenabled: "true"
  restuser: "admin"
  secretNamespace: "default"
  secretName: "heketi-secret"
  gidMin: "40000"
  gidMax: "50000"
  volumetype: "replicate:3"
```

- `resturl`: Gluster REST 服务/Heketi 服务 url, 用来根据需要供应 gluster 卷。 通常格式为
  `IPaddress:Port` 这是 GlusterFS 动态供应的必要参数。 如果 Heketi 服务是可路由的服务
  在 openshift/kubernetes 设置中提供。 这会有一个类似 `http://heketi-storage-project.cloudapps.mystorage.com`
  格式的地址，其中的 fqdn 是 Heketi 服务的可解析 url.

- `restauthenabled` :Gluster REST 服务是否开启认证。 如果这个值是 `"true"`， 则必须要提供
  `restuser` 和 `restuserkey` 或 `secretNamespace` + `secretName`。 这个选项已经废弃
  当 `restuser`, `restuserkey`, `secretName` 或 `secretNamespace` 有值时，默认就启用

- `restuser` : Gluster REST 服务/Heketi 中可以访问并在 Gluster Trusted Pool 中创建卷的用户。

- `restuserkey` : Gluster REST 服务/Heketi 用户的密码，用于 REST 服务认证。 这个参数
  已经废弃，推荐使用 `secretNamespace` + `secretName`

- `secretNamespace`, `secretName` :  确定在访问 Gluster REST 服务用户密码的 Secret
  实例是哪个。这两个参数为可选， 如果它们都没有设置，则会使用空密码。 提供的 Secret 必要是
  `"kubernetes.io/glusterfs"` 类型的。 创建示例

    ```
    kubectl create secret generic heketi-secret \
      --type="kubernetes.io/glusterfs" --from-literal=key='opensesame' \
      --namespace=default
    ```

    使用 Secret 的示例在这里
    [glusterfs-provisioning-secret.yaml](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/glusterfs/glusterfs-secret.yaml).

- `clusterid`: `630372ccdc720a92c681fb928f27b53f` 集群 ID，会在 Heketi 供应卷时用到。
  也可以是集群 ID 的列表，例如: `"8452344e2becec931ece4e33c4674e4e,42982310de6c63381718ccfa6d8cf397"`
  这个参数为可选

- `gidMin`, `gidMax` : StorageClass GID 范围的最小值和最大值， 在这个范围(`gidMin`-`gidMax`)
  中的一个唯一值(GID) 会用于动态供应卷。 这两个参数为可选。 如果没有指定，被供应卷的范围值为
  `2000-2147483647` 也就对应着默认的最小值(`gidMin`)和最大值(`gidMax`)

- `volumetype` : 卷类型及其参数可以使用这个参数配置，这是一个可选参数。 如果卷类型没有指定，
  则其类型由供应者决定。

    例如:
    - 复制型卷: `volumetype: replicate:3` 其中 3 是副本数
    - Disperse/EC 卷 : `volumetype: disperse:4:2` 其中 4 是数据 2 是冗余数
    - 分布式卷: `volumetype: none`

    对于可用的卷类型及其管理选项，见
    [Administration Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Storage/3.1/html/Administration_Guide/part-Overview.html).

    更多信息见
    [怎么配置 Heketi](https://github.com/heketi/heketi/wiki/Setting-up-the-topology).

    当持久化卷是被动态供应时， Gluster 会自动创建一个 Endpoint 和一个无头 Service, 名称都是
    `gluster-dynamic-<claimname>`. 当 PVC 被删除时，对应的 Endpoint 和 Service 也会
    动态自动删除。

<!--
### OpenStack Cinder

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/cinder
parameters:
  availability: nova
```

* `availability`: Availability Zone. If not specified, volumes are generally
  round-robin-ed across all active zones where Kubernetes cluster has a node.

{{< note >}}
{{< feature-state state="deprecated" for_k8s_version="v1.11" >}}
This internal provisioner of OpenStack is deprecated. Please use [the external cloud provider for OpenStack](https://github.com/kubernetes/cloud-provider-openstack).
{{< /note >}}
-->

### OpenStack Cinder {#openstack-cinder}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/cinder
parameters:
  availability: nova
```

- `availability`: 可用区域。 如果没有设置， 卷会在所有有 k8s 节点的区域中随机

{{< note >}}
{{< feature-state state="deprecated" for_k8s_version="v1.11" >}}

这个 OpenStack 内部供应都已经废弃。 请使用
[OpenStack 外部云提供者](https://github.com/kubernetes/cloud-provider-openstack).
{{< /note >}}
<!--
### vSphere

There are two types of provisioners for vSphere storage classes:

- [CSI provisioner](#csi-provisioner): `csi.vsphere.vmware.com`
- [vCP provisioner](#vcp-provisioner): `kubernetes.io/vsphere-volume`

In-tree provisioners are [deprecated](/blog/2019/12/09/kubernetes-1-17-feature-csi-migration-beta/#why-are-we-migrating-in-tree-plugins-to-csi). For more information on the CSI provisioner, see [Kubernetes vSphere CSI Driver](https://vsphere-csi-driver.sigs.k8s.io/) and [vSphereVolume CSI migration](/docs/concepts/storage/volumes/#csi-migration-5).
 -->

### vSphere {#vsphere}

vSphere 存储类别有两种类型的供应者:
- [CSI 供应者](#vsphere-provisioner-csi): `csi.vsphere.vmware.com`
- [vCP 供应者](#vcp-provisioner): `kubernetes.io/vsphere-volume`

内部供应都已经
[废弃](https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-csi-migration-beta/#why-are-we-migrating-in-tree-plugins-to-csi)
更多关于 CSI 供应者的信息见
[Kubernetes vSphere CSI Driver](https://vsphere-csi-driver.sigs.k8s.io/) and [vSphereVolume CSI migration](/docs/concepts/storage/volumes/#csi-migration-5).
<!--
#### CSI Provisioner {#vsphere-provisioner-csi}

The vSphere CSI StorageClass provisioner works with Tanzu Kubernetes clusters. For an example, refer to the [vSphere CSI repository](https://raw.githubusercontent.com/kubernetes-sigs/vsphere-csi-driver/master/example/vanilla-k8s-file-driver/example-sc.yaml).
 -->

#### CSI 供应者 {#vsphere-provisioner-csi}

vSphere CSI StorageClass 供应者工作于 Tanzu k8s 集群中。 示例见
[vSphere CSI 仓库](https://raw.githubusercontent.com/kubernetes-sigs/vsphere-csi-driver/master/example/vanilla-k8s-file-driver/example-sc.yaml).
<!--
#### vCP Provisioner

The following examples use the VMware Cloud Provider (vCP) StorageClass provisioner.  

1. Create a StorageClass with a user specified disk format.

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/vsphere-volume
    parameters:
      diskformat: zeroedthick
    ```

    `diskformat`: `thin`, `zeroedthick` and `eagerzeroedthick`. Default: `"thin"`.

2. Create a StorageClass with a disk format on a user specified datastore.

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/vsphere-volume
    parameters:
        diskformat: zeroedthick
        datastore: VSANDatastore
    ```

    `datastore`: The user can also specify the datastore in the StorageClass.
    The volume will be created on the datastore specified in the StorageClass,
    which in this case is `VSANDatastore`. This field is optional. If the
    datastore is not specified, then the volume will be created on the datastore
    specified in the vSphere config file used to initialize the vSphere Cloud
    Provider.

3. Storage Policy Management inside kubernetes

    * Using existing vCenter SPBM policy

        One of the most important features of vSphere for Storage Management is
        policy based Management. Storage Policy Based Management (SPBM) is a
        storage policy framework that provides a single unified control plane
        across a broad range of data services and storage solutions. SPBM enables
        vSphere administrators to overcome upfront storage provisioning challenges,
        such as capacity planning, differentiated service levels and managing
        capacity headroom.

        The SPBM policies can be specified in the StorageClass using the
        `storagePolicyName` parameter.

    * Virtual SAN policy support inside Kubernetes

        Vsphere Infrastructure (VI) Admins will have the ability to specify custom
        Virtual SAN Storage Capabilities during dynamic volume provisioning. You
        can now define storage requirements, such as performance and availability,
        in the form of storage capabilities during dynamic volume provisioning.
        The storage capability requirements are converted into a Virtual SAN
        policy which are then pushed down to the Virtual SAN layer when a
        persistent volume (virtual disk) is being created. The virtual disk is
        distributed across the Virtual SAN datastore to meet the requirements.

        You can see [Storage Policy Based Management for dynamic provisioning of volumes](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/policy-based-mgmt.html)
        for more details on how to use storage policies for persistent volumes
        management.

There are few
[vSphere examples](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere)
which you try out for persistent volume management inside Kubernetes for vSphere.
 -->

#### vCP 供应者 {#vcp-provisioner}

The following examples use the VMware Cloud Provider (vCP) StorageClass provisioner.  
以下示例使用 VMware Cloud Provider (vCP) StorageClass 应用者。

1. 创建一个用户指定硬盘模式的 StorageClass

    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/vsphere-volume
    parameters:
      diskformat: zeroedthick
    ```

    `diskformat`: `thin`, `zeroedthick` 和 `eagerzeroedthick`. 默认: `"thin"`.

2. 创建一个用于指定数据源和基于该数据库的磁盘模式的 StorageClass
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/vsphere-volume
    parameters:
        diskformat: zeroedthick
        datastore: VSANDatastore
    ```

    `datastore`:  用户也可以在 StorageClass 中指定数据源(datastore). 卷会在 StorageClass
    指定的数据源上创建，本例中就是 `VSANDatastore`. 该字段为可选。 如果没有指定数据源，
    则会使用 vSphere Cloud Provider 初始化时使用的 vSphere 配置文件中指定的数据源来创建卷。
3. 在 k8s 中的存储策略管理

    - 使用已经存在的 vCenter SPBM 策略

        vSphere 对于存储管理的重要特性之一就是基于策略的管理。 基于策略的存储管理(SPBM)是一个
        存储策略框架， 它为大范围的数据服务和存储方案提供一个统一的控制台。 SPBM 让 vSphere
        管理员能够应对存储供应的挑战，如 容量计划，细分服务级别，可用空间管理(capacity headroom)

        SPBM 可能通过 StorageClass 中的 `storagePolicyName` 来指定。
    - k8s 内部支持的虚拟 SAN 策略

        Vsphere 基础设施 (VI) 管理员可以在动态卷供应时指定自定义的虚拟 SAN 存储能力。
        这时候可以在动态供应时以存储能力的形式定义存储要求， 如性能和可用性。 在一个持久化卷(虚拟磁盘)
        被创建时，存储能力要求会被转换成虚拟 SAN 策略，再被推到虚拟 SAN 层。虚拟磁盘会发布在
        满足要求的虚拟 SAN 数据中。

        更多关于怎么使用持久化卷管理的存储策略见
        [Storage Policy Based Management for dynamic provisioning of volumes](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/policy-based-mgmt.html)

这些
[vSphere 示例](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere)
可以用来熟悉用于 vSphere 的 k8s 集群中的持久化卷管理
### Ceph RBD

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/rbd
parameters:
  monitors: 10.16.153.105:6789
  adminId: kube
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: kube
  userId: kube
  userSecretName: ceph-secret-user
  userSecretNamespace: default
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"
```

* `monitors`: Ceph monitors, comma delimited. This parameter is required.
* `adminId`: Ceph client ID that is capable of creating images in the pool.
  Default is "admin".
* `adminSecretName`: Secret Name for `adminId`. This parameter is required.
  The provided secret must have type "kubernetes.io/rbd".
* `adminSecretNamespace`: The namespace for `adminSecretName`. Default is "default".
* `pool`: Ceph RBD pool. Default is "rbd".
* `userId`: Ceph client ID that is used to map the RBD image. Default is the
  same as `adminId`.
* `userSecretName`: The name of Ceph Secret for `userId` to map RBD image. It
  must exist in the same namespace as PVCs. This parameter is required.
  The provided secret must have type "kubernetes.io/rbd", for example created in this
  way:

    ```shell
    kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
      --from-literal=key='QVFEQ1pMdFhPUnQrSmhBQUFYaERWNHJsZ3BsMmNjcDR6RFZST0E9PQ==' \
      --namespace=kube-system
    ```
* `userSecretNamespace`: The namespace for `userSecretName`.
* `fsType`: fsType that is supported by kubernetes. Default: `"ext4"`.
* `imageFormat`: Ceph RBD image format, "1" or "2". Default is "2".
* `imageFeatures`: This parameter is optional and should only be used if you
  set `imageFormat` to "2". Currently supported features are `layering` only.
  Default is "", and no features are turned on.

### Quobyte

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: slow
provisioner: kubernetes.io/quobyte
parameters:
    quobyteAPIServer: "http://138.68.74.142:7860"
    registry: "138.68.74.142:7861"
    adminSecretName: "quobyte-admin-secret"
    adminSecretNamespace: "kube-system"
    user: "root"
    group: "root"
    quobyteConfig: "BASE"
    quobyteTenant: "DEFAULT"
```

* `quobyteAPIServer`: API Server of Quobyte in the format
  `"http(s)://api-server:7860"`
* `registry`: Quobyte registry to use to mount the volume. You can specify the
  registry as ``<host>:<port>`` pair or if you want to specify multiple
  registries you just have to put a comma between them e.q.
  ``<host1>:<port>,<host2>:<port>,<host3>:<port>``.
  The host can be an IP address or if you have a working DNS you can also
  provide the DNS names.
* `adminSecretNamespace`: The namespace for `adminSecretName`.
  Default is "default".
* `adminSecretName`: secret that holds information about the Quobyte user and
  the password to authenticate against the API server. The provided secret
  must have type "kubernetes.io/quobyte" and the keys `user` and `password`,
  for example:

    ```shell
    kubectl create secret generic quobyte-admin-secret \
      --type="kubernetes.io/quobyte" --from-literal=user='admin' --from-literal=password='opensesame' \
      --namespace=kube-system
    ```

* `user`: maps all access to this user. Default is "root".
* `group`: maps all access to this group. Default is "nfsnobody".
* `quobyteConfig`: use the specified configuration to create the volume. You
  can create a new configuration or modify an existing one with the Web
  console or the quobyte CLI. Default is "BASE".
* `quobyteTenant`: use the specified tenant ID to create/delete the volume.
  This Quobyte tenant has to be already present in Quobyte.
  Default is "DEFAULT".

### Azure Disk

#### Azure Unmanaged Disk storage class {#azure-unmanaged-disk-storage-class}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Standard_LRS
  location: eastus
  storageAccount: azure_storage_account_name
```

* `skuName`: Azure storage account Sku tier. Default is empty.
* `location`: Azure storage account location. Default is empty.
* `storageAccount`: Azure storage account name. If a storage account is provided,
  it must reside in the same resource group as the cluster, and `location` is
  ignored. If a storage account is not provided, a new storage account will be
  created in the same resource group as the cluster.

#### Azure Disk storage class (starting from v1.7.2) {#azure-disk-storage-class}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Standard_LRS
  kind: Shared
```

* `storageaccounttype`: Azure storage account Sku tier. Default is empty.
* `kind`: Possible values are `shared` (default), `dedicated`, and `managed`.
  When `kind` is `shared`, all unmanaged disks are created in a few shared
  storage accounts in the same resource group as the cluster. When `kind` is
  `dedicated`, a new dedicated storage account will be created for the new
  unmanaged disk in the same resource group as the cluster. When `kind` is
  `managed`, all managed disks are created in the same resource group as
  the cluster.
* `resourceGroup`: Specify the resource group in which the Azure disk will be created.
   It must be an existing resource group name. If it is unspecified, the disk will be
   placed in the same resource group as the current Kubernetes cluster.

- Premium VM can attach both Standard_LRS and Premium_LRS disks, while Standard
  VM can only attach Standard_LRS disks.
- Managed VM can only attach managed disks and unmanaged VM can only attach
  unmanaged disks.

### Azure File

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  skuName: Standard_LRS
  location: eastus
  storageAccount: azure_storage_account_name
```

* `skuName`: Azure storage account Sku tier. Default is empty.
* `location`: Azure storage account location. Default is empty.
* `storageAccount`: Azure storage account name.  Default is empty. If a storage
  account is not provided, all storage accounts associated with the resource
  group are searched to find one that matches `skuName` and `location`. If a
  storage account is provided, it must reside in the same resource group as the
  cluster, and `skuName` and `location` are ignored.
* `secretNamespace`: the namespace of the secret that contains the Azure Storage
  Account Name and Key. Default is the same as the Pod.
* `secretName`: the name of the secret that contains the Azure Storage Account Name and
  Key. Default is `azure-storage-account-<accountName>-secret`
* `readOnly`: a flag indicating whether the storage will be mounted as read only.
  Defaults to false which means a read/write mount. This setting will impact the
  `ReadOnly` setting in VolumeMounts as well.

During storage provisioning, a secret named by `secretName` is created for the
mounting credentials. If the cluster has enabled both
[RBAC](/docs/reference/access-authn-authz/rbac/) and
[Controller Roles](/docs/reference/access-authn-authz/rbac/#controller-roles),
add the `create` permission of resource `secret` for clusterrole
`system:controller:persistent-volume-binder`.

In a multi-tenancy context, it is strongly recommended to set the value for
`secretNamespace` explicitly, otherwise the storage account credentials may
be read by other users.

### Portworx Volume

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: portworx-io-priority-high
provisioner: kubernetes.io/portworx-volume
parameters:
  repl: "1"
  snap_interval:   "70"
  priority_io:  "high"

```

* `fs`: filesystem to be laid out: `none/xfs/ext4` (default: `ext4`).
* `block_size`: block size in Kbytes (default: `32`).
* `repl`: number of synchronous replicas to be provided in the form of
  replication factor `1..3` (default: `1`) A string is expected here i.e.
  `"1"` and not `1`.
* `priority_io`: determines whether the volume will be created from higher
  performance or a lower priority storage `high/medium/low` (default: `low`).
* `snap_interval`: clock/time interval in minutes for when to trigger snapshots.
  Snapshots are incremental based on difference with the prior snapshot, 0
  disables snaps (default: `0`). A string is expected here i.e.
  `"70"` and not `70`.
* `aggregation_level`: specifies the number of chunks the volume would be
  distributed into, 0 indicates a non-aggregated volume (default: `0`). A string
  is expected here i.e. `"0"` and not `0`
* `ephemeral`: specifies whether the volume should be cleaned-up after unmount
  or should be persistent. `emptyDir` use case can set this value to true and
  `persistent volumes` use case such as for databases like Cassandra should set
  to false, `true/false` (default `false`). A string is expected here i.e.
  `"true"` and not `true`.

### ScaleIO

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/scaleio
parameters:
  gateway: https://192.168.99.200:443/api
  system: scaleio
  protectionDomain: pd0
  storagePool: sp1
  storageMode: ThinProvisioned
  secretRef: sio-secret
  readOnly: false
  fsType: xfs
```

* `provisioner`: attribute is set to `kubernetes.io/scaleio`
* `gateway`: address to a ScaleIO API gateway (required)
* `system`: the name of the ScaleIO system (required)
* `protectionDomain`: the name of the ScaleIO protection domain (required)
* `storagePool`: the name of the volume storage pool (required)
* `storageMode`: the storage provision mode: `ThinProvisioned` (default) or
  `ThickProvisioned`
* `secretRef`: reference to a configured Secret object (required)
* `readOnly`: specifies the access mode to the mounted volume (default false)
* `fsType`: the file system to use for the volume (default ext4)

The ScaleIO Kubernetes volume plugin requires a configured Secret object.
The secret must be created with type `kubernetes.io/scaleio` and use the same
namespace value as that of the PVC where it is referenced
as shown in the following command:

```shell
kubectl create secret generic sio-secret --type="kubernetes.io/scaleio" \
--from-literal=username=sioadmin --from-literal=password=d2NABDNjMA== \
--namespace=default
```

### StorageOS

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/storageos
parameters:
  pool: default
  description: Kubernetes volume
  fsType: ext4
  adminSecretNamespace: default
  adminSecretName: storageos-secret
```

* `pool`: The name of the StorageOS distributed capacity pool to provision the
  volume from.  Uses the `default` pool which is normally present if not specified.
* `description`: The description to assign to volumes that were created dynamically.
  All volume descriptions will be the same for the storage class, but different
  storage classes can be used to allow descriptions for different use cases.
  Defaults to `Kubernetes volume`.
* `fsType`: The default filesystem type to request. Note that user-defined rules
  within StorageOS may override this value.  Defaults to `ext4`.
* `adminSecretNamespace`: The namespace where the API configuration secret is
  located. Required if adminSecretName set.
* `adminSecretName`: The name of the secret to use for obtaining the StorageOS
  API credentials. If not specified, default values will be attempted.

The StorageOS Kubernetes volume plugin can use a Secret object to specify an
endpoint and credentials to access the StorageOS API. This is only required when
the defaults have been changed.
The secret must be created with type `kubernetes.io/storageos` as shown in the
following command:

```shell
kubectl create secret generic storageos-secret \
--type="kubernetes.io/storageos" \
--from-literal=apiAddress=tcp://localhost:5705 \
--from-literal=apiUsername=storageos \
--from-literal=apiPassword=storageos \
--namespace=default
```

Secrets used for dynamically provisioned volumes may be created in any namespace
and referenced with the `adminSecretNamespace` parameter. Secrets used by
pre-provisioned volumes must be created in the same namespace as the PVC that
references it.

### Local

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Local volumes do not currently support dynamic provisioning, however a StorageClass
should still be created to delay volume binding until Pod scheduling. This is
specified by the `WaitForFirstConsumer` volume binding mode.

Delaying volume binding allows the scheduler to consider all of a Pod's
scheduling constraints when choosing an appropriate PersistentVolume for a
PersistentVolumeClaim.
