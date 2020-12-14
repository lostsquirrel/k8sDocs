---
title: 卷动态供应
content_type: concept
weight: 70
---
<!--
---
reviewers:
- saad-ali
- jsafrane
- thockin
- msau42
title: Dynamic Volume Provisioning
content_type: concept
weight: 70
---
 -->
<!-- overview -->
<!--
Dynamic volume provisioning allows storage volumes to be created on-demand.
Without dynamic provisioning, cluster administrators have to manually make
calls to their cloud or storage provider to create new storage volumes, and
then create [`PersistentVolume` objects](/docs/concepts/storage/persistent-volumes/)
to represent them in Kubernetes. The dynamic provisioning feature eliminates
the need for cluster administrators to pre-provision storage. Instead, it
automatically provisions storage when it is requested by users.
 -->

卷动态供应允许存储卷可以根据需要创建。 如果没有动态供应，集群管理员必须要手动调用云存储或其它存储
提供者来创建新的存储卷，然后再创建
[`PersistentVolume` 对象](/k8sDocs/docs/concepts/storage/persistent-volumes/)来
在 k8s 中代表他们。动态供应特性就是将管理员从这些预创建的任务中解放出来。 系统会根据用户的请求
自动供应存储

<!-- body -->
<!--
## Background

The implementation of dynamic volume provisioning is based on the API object `StorageClass`
from the API group `storage.k8s.io`. A cluster administrator can define as many
`StorageClass` objects as needed, each specifying a *volume plugin* (aka
*provisioner*) that provisions a volume and the set of parameters to pass to
that provisioner when provisioning.
A cluster administrator can define and expose multiple flavors of storage (from
the same or different storage systems) within a cluster, each with a custom set
of parameters. This design also ensures that end users don't have to worry
about the complexity and nuances of how storage is provisioned, but still
have the ability to select from multiple storage options.

More information on storage classes can be found
[here](/docs/concepts/storage/storage-classes/).
 -->

## 背景

卷动态供应是基于来自 `storage.k8s.io` API 组的 `StorageClass` API 对象。 集群管理员可以
根据需要创建任意数量的 `StorageClass` 对象， 每个对象中都会指定一个 *卷插件*(也就是 *供应者*)
来提供卷并且包含供应卷时需要传递的参数集。集群管理员可以在集群中定义并暴露多个偏好的存储(来自相同
  或不同的存储系统)，每种都有一套自定义的参数。 这种设计也保证终端用户不需要关心存储供应的复杂性和差异性
的同时还能提供多种存储选项的选择。

更多关于存储类别的信息见
[这里](/k8sDocs/docs/concepts/storage/storage-classes/).
<!--
## Enabling Dynamic Provisioning

To enable dynamic provisioning, a cluster administrator needs to pre-create
one or more StorageClass objects for users.
StorageClass objects define which provisioner should be used and what parameters
should be passed to that provisioner when dynamic provisioning is invoked.
The name of a StorageClass object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

The following manifest creates a storage class "slow" which provisions standard
disk-like persistent disks.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```

The following manifest creates a storage class "fast" which provisions
SSD-like persistent disks.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
 -->

## 启用动态供应 {#enabling-dynamic-provisioning}

为启用动态供应，集群管理员需要预先为用户创建好一个或多个 `StorageClass` 对象。`StorageClass`
对象定义需要使用哪个供应者和在动态供应调用时需要传递给供应者的参数。 `StorageClass` 对象的名称
必须是一个有效的 [DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

以下的配置中定义了一个叫 "slow" `StorageClass`， 它提供了一个标准的类磁盘的持久下磁盘。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```

以下的配置中定义了一个叫 "fast" `StorageClass`， 它提供了一个类SSD盘的持久下磁盘。
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
<!--
## Using Dynamic Provisioning

Users request dynamically provisioned storage by including a storage class in
their `PersistentVolumeClaim`. Before Kubernetes v1.6, this was done via the
`volume.beta.kubernetes.io/storage-class` annotation. However, this annotation
is deprecated since v1.6. Users now can and should instead use the
`storageClassName` field of the `PersistentVolumeClaim` object. The value of
this field must match the name of a `StorageClass` configured by the
administrator (see [below](#enabling-dynamic-provisioning)).

To select the "fast" storage class, for example, a user would create the
following PersistentVolumeClaim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

This claim results in an SSD-like Persistent Disk being automatically
provisioned. When the claim is deleted, the volume is destroyed.
-->

## 使用动态供应 {#using-dynamic-provisioning}

用户通过在 `PersistentVolumeClaim` 引用一个 `StorageClass` 申请动态供应存储。 在 k8s
v1.6 之前， 这是通过添加注解 `volume.beta.kubernetes.io/storage-class` 实现的。但是
这个注解在 k8s v1.6 之后已经废弃。在这之后的版本用户可以在 `PersistentVolumeClaim` 对象
中使用 `storageClassName` 代替这个注解。这个字段值必须与管理员配置的 `StorageClass` 中的一个
 (见 [上面](#enabling-dynamic-provisioning)).

例如，想要使用 "fast" `StorageClass`， 用户可以创建以下 PersistentVolumeClaim:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```

这个配置的 PVC 会自动供应一个类 SSD 的持久化磁盘。 当 PVC 被删除后，对应卷也会被删除。
<!--
## Defaulting Behavior

Dynamic provisioning can be enabled on a cluster such that all claims are
dynamically provisioned if no storage class is specified. A cluster administrator
can enable this behavior by:

- Marking one `StorageClass` object as *default*;
- Making sure that the [`DefaultStorageClass` admission controller](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
  is enabled on the API server.

An administrator can mark a specific `StorageClass` as default by adding the
`storageclass.kubernetes.io/is-default-class` annotation to it.
When a default `StorageClass` exists in a cluster and a user creates a
`PersistentVolumeClaim` with `storageClassName` unspecified, the
`DefaultStorageClass` admission controller automatically adds the
`storageClassName` field pointing to the default storage class.

Note that there can be at most one *default* storage class on a cluster, or
a `PersistentVolumeClaim` without `storageClassName` explicitly specified cannot
be created.
 -->

## 默认行为

可以在集群中启用动态供应，这样所有的PVC 都可以在没有指定 StorageClass 的情况下也是动态供应的。
集群管理员可以通过以下方式启动:

- 将一个 `StorageClass` 对象设置为 *默认*;
  is enabled on the API server.
- 确保 API server 上的
  [`DefaultStorageClass` 准入控制](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
  启用了

管理员可以通过在一个 `StorageClass` 添加 `storageclass.kubernetes.io/is-default-class`
注解的方式将其设置为默认。 当集群中有默认的 `StorageClass` 存在时，用户创建 `PersistentVolumeClaim`
没有指定 `storageClassName` ，`DefaultStorageClass` 准入控制自动添加 `storageClassName`
字段，并将其指向默认的 `StorageClass`。

要注意一个集群中最多只能有一个 *默认* `StorageClass`， 否则没有显示指定 `storageClassName`
的 `PersistentVolumeClaim` 是不能创建的。
<!--
## Topology Awareness

In [Multi-Zone](/docs/setup/multiple-zones) clusters, Pods can be spread across
Zones in a Region. Single-Zone storage backends should be provisioned in the Zones where
Pods are scheduled. This can be accomplished by setting the [Volume Binding
Mode](/docs/concepts/storage/storage-classes/#volume-binding-mode).
 -->
## 拓扑自洽

在 [多区域](/k8sDocs/docs/setup/multiple-zones) 集群中，Pod 可以在同一个地区的不同区域中分布。
单区域存储后端应该供应到 Pod 被调度到的区域中。这可以通过设置
[卷绑定模式](/k8sDocs/docs/concepts/storage/storage-classes/#volume-binding-mode)
实现。
