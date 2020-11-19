---
title: 持久化卷(PV)
feature:
  title: 存储编排
  description: >
    根据选择自动挂载存储系统，无论是本地存储，诸如 href="https://cloud.google.com/storage/">GCP</a>
    或 <a href="https://aws.amazon.com/products/storage/">AWS</a> 的公有云存储，
    或诸如 NFS, iSCSI, Gluster, Ceph, Cinder, Flocker 的网络存储。
content_type: concept
weight: 20
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- thockin
- msau42
- xing-yang
title: Persistent Volumes
feature:
  title: Storage orchestration
  description: >
    Automatically mount the storage system of your choice, whether from local storage, a public cloud provider such as <a href="https://cloud.google.com/storage/">GCP</a> or <a href="https://aws.amazon.com/products/storage/">AWS</a>, or a network storage system such as NFS, iSCSI, Gluster, Ceph, Cinder, or Flocker.

content_type: concept
weight: 20
---
 -->
<!-- overview -->
<!--
This document describes the current state of _persistent volumes_ in Kubernetes. Familiarity with [volumes](/docs/concepts/storage/volumes/) is suggested.
 -->
本文主要介绍当前 k8s 中 _持久化卷(persistent volume)_ 的状态。
建议先熟悉 [volumes](/k8sDocs/concepts/storage/volumes/)
<!-- body -->
<!--
## Introduction

Managing storage is a distinct problem from managing compute instances. The PersistentVolume subsystem provides an API for users and administrators that abstracts details of how storage is provided from how it is consumed. To do this, we introduce two new API resources:  PersistentVolume and PersistentVolumeClaim.

A _PersistentVolume_ (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using [Storage Classes](/docs/concepts/storage/storage-classes/). It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

A _PersistentVolumeClaim_ (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory).  Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany or ReadWriteMany, see [AccessModes](#access-modes)).

While PersistentVolumeClaims allow a user to consume abstract storage resources, it is common that users need PersistentVolumes with varying properties, such as performance, for different problems. Cluster administrators need to be able to offer a variety of PersistentVolumes that differ in more ways than just size and access modes, without exposing users to the details of how those volumes are implemented. For these needs, there is the _StorageClass_ resource.

See the [detailed walkthrough with working examples](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/).
 -->

## 介绍

管理存储和管理实例是两个不同的问题。 PersistentVolume 子系统为用户和管理员提供了一套 API 将
存储是怎么提供的细节从存储使用中抽象出来。 而为了做到这一个我们要介绍两个新的 API 资源:
PersistentVolume 和 PersistentVolumeClaim.

_PersistentVolume_ (PV) 是集群中的一块存储，它可能由管理员管理或使用
[StorageClass](/docs/concepts/storage/storage-classes/)
动态管理。它是集群中的一个资源，就像节点是集群中的一个资源一样。 PV 是与卷(Volume)类似的卷插件，
但它的生命周期与使用它的 Pod 的生命周期是相互独立的。这个 API 对象中包含了存储的实现细节，包含
NFS, iSCSI, 云服务提供的存储系统。

_PersistentVolumeClaim_ (PVC) 是一个用户对存储的请求。 它就像是一个 Pod。Pod 使用的是节点上的资源
而 PVC 使用的是 PV 资源。 Pod 可以申请指定级别的资源(CPU 和 Memory)。 PVC 可以申请指定容量的存储
和访问模式 (如，它们可以以 `ReadWriteOnce`, `ReadOnlyMany` 或 `ReadWriteMany` 方式挂载，
见 [AccessMode](#access-modes))

因为 `PersistentVolumeClaim` 允许用户使用抽象的存储资源，通常用户都需要 PersistentVolume
中包含许多属性，如性能，来应对不同的问题。 集群管理员需要能够提供多样的 PersistentVolume，而不
仅仅是容量和访问模式，还需要向用户提供这些卷的实现细节。 为了满足这些需求，我们就有了 _StorageClass_ 资源。

[亲自上手试试](/k8sDocs/tasks/configure-pod-container/configure-persistent-volume-storage/).
<!--
## Lifecycle of a volume and claim

PVs are resources in the cluster. PVCs are requests for those resources and also act as claim checks to the resource. The interaction between PVs and PVCs follows this lifecycle:
 -->

## PV 和 PVC 的生命周期

PV 是集群中的资源。 PVC 是对这些资源的申请同时也表现为对这些资源的检查声明。PV 和 PVC 之间的
相互影响遵守以下生命周期:
<!--
### Provisioning

There are two ways PVs may be provisioned: statically or dynamically.
 -->
### 供给

PV 可以被两个方式提供: 静态供给 或 动态供给
<!--
#### Static

A cluster administrator creates a number of PVs. They carry the details of the real storage, which is available for use by cluster users. They exist in the Kubernetes API and are available for consumption.
 -->

#### 静态供给

集群管理创建一系列 PV。 它们包含真实存储的细节，可以被集群用户使用。
它们存在于 k8s API 中，可以被取用。
<!--
#### Dynamic

When none of the static PVs the administrator created match a user's PersistentVolumeClaim,
the cluster may try to dynamically provision a volume specially for the PVC.
This provisioning is based on StorageClasses: the PVC must request a
[storage class](/docs/concepts/storage/storage-classes/) and
the administrator must have created and configured that class for dynamic
provisioning to occur. Claims that request the class `""` effectively disable
dynamic provisioning for themselves.

To enable dynamic storage provisioning based on storage class, the cluster administrator
needs to enable the `DefaultStorageClass` [admission controller](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
on the API server. This can be done, for example, by ensuring that `DefaultStorageClass` is
among the comma-delimited, ordered list of values for the `--enable-admission-plugins` flag of
the API server component. For more information on API server command-line flags,
check [kube-apiserver](/docs/admin/kube-apiserver/) documentation.
 -->

#### 动态供给

当管理创建的 PV 不能满足用户的 PersistentVolumeClaim 时，集群可能就会尝试为这个 PVC 动态
提供一个卷。这种供给基于 `StorageClass`: PVC 必须要申请一个
[StorageClass](/k8sDocs/concepts/storage/storage-classes/)
并且在动态供给发生之前管理员必须要完成该 `StorageClass` 的创建和配置。
如果 PVC 的 `StorageClass` 被设置为 `""` 实际上表示关闭自身的动态供给。

要启用基于 `StorageClass` 的动态存储供给需要集群管理在 api-server 中的
[准入控制](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
中启用 `DefaultStorageClass`。 具体的配置方法就是确认 api-server 组件的 `--enable-admission-plugins` 选项
的值中有没有 `DefaultStorageClass`。 更多关于 api-server 命令行参数见
[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver/) 文档
<!--
### Binding

A user creates, or in the case of dynamic provisioning, has already created, a PersistentVolumeClaim with a specific amount of storage requested and with certain access modes. A control loop in the master watches for new PVCs, finds a matching PV (if possible), and binds them together. If a PV was dynamically provisioned for a new PVC, the loop will always bind that PV to the PVC. Otherwise, the user will always get at least what they asked for, but the volume may be in excess of what was requested. Once bound, PersistentVolumeClaim binds are exclusive, regardless of how they were bound. A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.

Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.
 -->

### 绑定 {#binding}

当一个指定容量和访问模式的 `PersistentVolumeClaim` 被创建后。 主控中心中的一个控制回环会监控新创建的 PVC，
如果有匹配的 PV 存在，则将它们绑定在一起。 如果这个 PV 是动态供给给这个新建的 PVC 的， 那么控制
回环将会始终将这个 PV 绑定到这个 PVC。 否则用户会得到至少满足其请求的卷，但这个卷容量可能实际是超出请求的。
当绑定完成，就会确定绑定关系，不管它们是怎么绑定的。 PVC 到 PV 的绑定是一对一关系，PV 使用的 `spec.claimRef`
实现 `PersistentVolume` 与 `PersistentVolumeClaim` 的双向绑定。

如果没有匹配的 PV 存在，PVC 会永远保持在未绑定状态。 当有可用的匹配 PV 出现时 PVC 就会绑定。
例如， 集群中提供许多 50Gi PV 是不会被一个申请 100Gi 的 PVC 匹配到的。当集群中添加了一个
100Gi PV 时，这个 PVC 就可以绑定了。
<!--
### Using

Pods use claims as volumes. The cluster inspects the claim to find the bound volume and mounts that volume for a Pod. For volumes that support multiple access modes, the user specifies which mode is desired when using their claim as a volume in a Pod.

Once a user has a claim and that claim is bound, the bound PV belongs to the user for as long as they need it. Users schedule Pods and access their claimed PVs by including a `persistentVolumeClaim` section in a Pod's `volumes` block. See [Claims As Volumes](#claims-as-volumes) for more details on this.
 -->

### 使用

Pod 会将 PVC 当作卷来使用。 集群会检视这个 PVC 找到它绑定的 PV 然后将这个 PV 挂载到 Pod 中。
对于支持多次访问模式的卷，用户在 Pod 当作卷的 PVC 中指定需要的访问模式。

当一个用户拥有一个 PVC 并且这个 PVC 完成绑定，则这个被绑定的 PV 在用户需要时始终属于该用户。
用户通过 Pod 中的 `volumes` 配置区中添加 `persistentVolumeClaim` 配置区来访问这些由
PVC 管理的 PV。
更多信息见 [将 PVC 当作卷(PV)](#claims-as-volumes)
<!--
### Storage Object in Use Protection
The purpose of the Storage Object in Use Protection feature is to ensure that PersistentVolumeClaims (PVCs) in active use by a Pod and PersistentVolume (PVs) that are bound to PVCs are not removed from the system, as this may result in data loss.

{{< note >}}
PVC is in active use by a Pod when a Pod object exists that is using the PVC.
{{< /note >}}

If a user deletes a PVC in active use by a Pod, the PVC is not removed immediately. PVC removal is postponed until the PVC is no longer actively used by any Pods. Also, if an admin deletes a PV that is bound to a PVC, the PV is not removed immediately. PV removal is postponed until the PV is no longer bound to a PVC.

You can see that a PVC is protected when the PVC's status is `Terminating` and the `Finalizers` list includes `kubernetes.io/pvc-protection`:

```shell
kubectl describe pvc hostpath
Name:          hostpath
Namespace:     default
StorageClass:  example-hostpath
Status:        Terminating
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-class=example-hostpath
               volume.beta.kubernetes.io/storage-provisioner=example.com/hostpath
Finalizers:    [kubernetes.io/pvc-protection]
...
```

You can see that a PV is protected when the PV's status is `Terminating` and the `Finalizers` list includes `kubernetes.io/pv-protection` too:

```shell
kubectl describe pv task-pv-volume
Name:            task-pv-volume
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Terminating
Claim:
Reclaim Policy:  Delete
Access Modes:    RWO
Capacity:        1Gi
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/data
    HostPathType:
Events:            <none>
```
 -->

### 使用保护模式的存储对象

保护模式的存储对象特性的目的是保证被 Pod 使用的有效的 PVC 和与 PVC 绑定的 PV 能会被系统删除，
从而可能导致数据丢失。
{{< note >}}
当一个使用 PVC 的 Pod 对象存在时就表示 PVC 被 Pod 有效使用。
{{< /note >}}

如果用户删除了一个被 Pod 有效使用的 PVC， 这个 PVC 不会立马被删除。 PVC 会延迟到没有被任何
Pod 有效使用后才会删除。 同样，如果管理员删除了一个与 PVC 绑定的 PV， 这个 PV 也不会被立马删除，
PV 的删除行为会被延迟到不再与 PVC 绑定时。

当 PVC 的状态是 `Terminating` 并且 `Finalizers` 列表中包含 `kubernetes.io/pvc-protection`
就表示这个 PVC 是被保护的。

```shell
kubectl describe pvc hostpath
Name:          hostpath
Namespace:     default
StorageClass:  example-hostpath
Status:        Terminating
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-class=example-hostpath
               volume.beta.kubernetes.io/storage-provisioner=example.com/hostpath
Finalizers:    [kubernetes.io/pvc-protection]
...
```

同样，当 PV 的状态是 `Terminating` 并且 `Finalizers` 列表中包含 `kubernetes.io/pvc-protection`
就表示这个 PV 是被保护的。

```shell
kubectl describe pv task-pv-volume
Name:            task-pv-volume
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Terminating
Claim:
Reclaim Policy:  Delete
Access Modes:    RWO
Capacity:        1Gi
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/data
    HostPathType:
Events:            <none>
```
<!--
### Reclaiming

When a user is done with their volume, they can delete the PVC objects from the API that allows reclamation of the resource. The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim. Currently, volumes can either be Retained, Recycled, or Deleted.
 -->

### 回收

当用于不再需要一个卷时，可以通过 API 删除这个 PVC 对象，这样就允许对对应资源的回收。 PV 的回收
策略让集群可以在 PV 释放以后使用对应的方式回收。 目前，卷的回收策略有 保留(`Retain`),
循环使用(`Recycle`), 删除 (`Delete`).
<!--
#### Retain

The `Retain` reclaim policy allows for manual reclamation of the resource. When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". But it is not yet available for another claim because the previous claimant's data remains on the volume. An administrator can manually reclaim the volume with the following steps.

1. Delete the PersistentVolume. The associated storage asset in external infrastructure (such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume) still exists after the PV is deleted.
1. Manually clean up the data on the associated storage asset accordingly.
1. Manually delete the associated storage asset, or if you want to reuse the same storage asset, create a new PersistentVolume with the storage asset definition.
 -->
#### 保留(`Retained`)

保留(`Retained`) 回收策略允许对资源手动回收。 当 PVC 被删除后，其之前绑定的 PV 依然存在并被
认为是已经释放。 但是它还不被另一个 PVC 使用，因为其中还有上一个 PVC 时的数据。 管理员可以
通过以下步骤手动回收这个卷:

1. 删除 PV 对象。 它所关联的外部存储设施(如 AWS EBS, GCE PD, Azure Disk, Cinder 卷)依然存在。
2. 根据需要手动清理对应存储资源上的数据
3. 手动删除对应的存储资源，如果想要重新使用该存储资源，可以再创建一个新的 PV 对象与之关联。
<!--
#### Delete

For volume plugins that support the `Delete` reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure, such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume. Volumes that were dynamically provisioned inherit the [reclaim policy of their StorageClass](#reclaim-policy), which defaults to `Delete`. The administrator should configure the StorageClass according to users' expectations; otherwise, the PV must be edited or patched after it is created. See [Change the Reclaim Policy of a PersistentVolume](/docs/tasks/administer-cluster/change-pv-reclaim-policy/).
 -->

#### 删除 (`Delete`)

对于支持 删除 (`Delete`) 回收策略的卷插件，在删除 PVC 对象之后与其郑的 PV 对象也会被删除，
而且对应的外部存储资源，如 AWS EBS, GCE PD, Azure Disk, Cinder 卷也会一同被删除。
动态提供的卷的回收策略继承自 [它们的 StorageClass 的回收策略](#reclaim-policy)，默认为 删除 (`Delete`)
管理应该根据用户期望设置 StorageClass 的回收策略，否则 PV 在创建后再需要再次修改才行。
见 [修改 PersistentVolume 的回收策略](/k8sDocs/tasks/administer-cluster/change-pv-reclaim-policy/).
<!--
#### Recycle

{{< warning >}}
The `Recycle` reclaim policy is deprecated. Instead, the recommended approach is to use dynamic provisioning.
{{< /warning >}}

If supported by the underlying volume plugin, the `Recycle` reclaim policy performs a basic scrub (`rm -rf /thevolume/*`) on the volume and makes it available again for a new claim.

However, an administrator can configure a custom recycler Pod template using
the Kubernetes controller manager command line arguments as described in the
[reference](/docs/reference/command-line-tools-reference/kube-controller-manager/).
The custom recycler Pod template must contain a `volumes` specification, as
shown in the example below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-recycler
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: vol
    hostPath:
      path: /any/path/it/will/be/replaced
  containers:
  - name: pv-recycler
    image: "k8s.gcr.io/busybox"
    command: ["/bin/sh", "-c", "test -e /scrub && rm -rf /scrub/..?* /scrub/.[!.]* /scrub/*  && test -z \"$(ls -A /scrub)\" || exit 1"]
    volumeMounts:
    - name: vol
      mountPath: /scrub
```

However, the particular path specified in the custom recycler Pod template in the `volumes` part is replaced with the particular path of the volume that is being recycled.
-->

#### 循环使用(`Recycle`)

{{< warning >}}
循环使用(`Recycle`) 回收策略已经废弃。 推荐使用动态供给方式。
{{< /warning >}}

如果底层卷插件支持， 循环使用(`Recycle`) 回收策略在卷上执行基础的清理操作 (`rm -rf /thevolume/*`)
然后它就可以再次被新的 PVC 使用了。

但是，管理也可以使用 k8s 控制管理器的命令行参数配置一个自定义的回收器 Pod 模板。具体见
[这里]({{<base-origin path="/docs/reference/command-line-tools-reference/kube-controller-manager/">}}).
这个自定义的回收器 Pod 模板必须要包含 `volumes` 定义，下面就是一个示例:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-recycler
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: vol
    hostPath:
      path: /any/path/it/will/be/replaced
  containers:
  - name: pv-recycler
    image: "k8s.gcr.io/busybox"
    command: ["/bin/sh", "-c", "test -e /scrub && rm -rf /scrub/..?* /scrub/.[!.]* /scrub/*  && test -z \"$(ls -A /scrub)\" || exit 1"]
    volumeMounts:
    - name: vol
      mountPath: /scrub
```

回收器 Pod 模板中的 `volumes` 部分指定的路径，就是要被回收的卷
<!--
### Reserving a PersistentVolume

The control plane can [bind PersistentVolumeClaims to matching PersistentVolumes](#binding) in the
cluster. However, if you want a PVC to bind to a specific PV, you need to pre-bind them.

By specifying a PersistentVolume in a PersistentVolumeClaim, you declare a binding between that specific PV and PVC.
If the PersistentVolume exists and has not reserved PersistentVolumeClaims through its `claimRef` field, then the PersistentVolume and PersistentVolumeClaim will be bound.

The binding happens regardless of some volume matching criteria, including node affinity.
The control plane still checks that [storage class](/docs/concepts/storage/storage-classes/), access modes, and requested storage size are valid.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
  namespace: foo
spec:
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: foo-pv
  ...
```

This method does not guarantee any binding privileges to the PersistentVolume. If other PersistentVolumeClaims could use the PV that you specify, you first need to reserve that storage volume. Specify the relevant PersistentVolumeClaim in the `claimRef` field of the PV so that other PVCs can not bind to it.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
  ...
```

This is useful if you want to consume PersistentVolumes that have their `claimPolicy` set
to `Retain`, including cases where you are reusing an existing PV.
 -->

### 保留 `PersistentVolume`

控制中心可以在集群中将[ `PersistentVolumeClaim` 与匹配的 `PersistentVolume` 绑定](#binding)。
但是，如果想要将 PVC 与指定 PV 绑定， 则需要预先绑定(而不是自动绑定)。

通过在 `PersistentVolumeClaim` 中指定一个 `PersistentVolume`，就可以将 PV 绑定到这个 PVC 上。
如果 PV 已经存在， 可能通过设置它的 `claimRef` 字段指定一个 `PersistentVolumeClaim`，
这样 PV 和 PVC 就会绑定。

这种绑定会忽略一些卷匹配条件，包括节点亲和性。 控制中心还是会检测，
[StorageClass](/k8sDocs/concepts/storage/storage-classes/), 访问模式，申请容量是否是有效的。

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
  namespace: foo
spec:
  storageClassName: "" # 这里必须要显示地设置，否则就会使用默认的 StorageClass
  volumeName: foo-pv
  ...
```

这种方式不对绑定 PV 的优先级做任何保证。 如果其它的 PVC 能够使用过这个 PV，必须要先保留存储卷。
在需要绑定的 PV `claimRef` 的 PVC 这样其它的 PVC 才不能绑定这个 PV。


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
  ...
```

如果想要使用 `claimPolicy` 是 `Retain` 的 PV 这一招很有用， 包括重复使用那些已经存在的 PV。
<!--
### Expanding Persistent Volumes Claims

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

Support for expanding PersistentVolumeClaims (PVCs) is now enabled by default. You can expand
the following types of volumes:

* gcePersistentDisk
* awsElasticBlockStore
* Cinder
* glusterfs
* rbd
* Azure File
* Azure Disk
* Portworx
* FlexVolumes
* CSI

You can only expand a PVC if its storage class's `allowVolumeExpansion` field is set to true.

``` yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gluster-vol-default
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

To request a larger volume for a PVC, edit the PVC object and specify a larger
size. This triggers expansion of the volume that backs the underlying PersistentVolume. A
new PersistentVolume is never created to satisfy the claim. Instead, an existing volume is resized.
 -->

### 扩展 PVC

{{< feature-state for_k8s_version="v1.11" state="beta" >}}

对 PVC 的扩展默认是启用的。 可以对以下类型的卷进行扩展:

* gcePersistentDisk
* awsElasticBlockStore
* Cinder
* glusterfs
* rbd
* Azure File
* Azure Disk
* Portworx
* FlexVolumes
* CSI

只能对 StorageClass 的 `allowVolumeExpansion` 字段为 `true` PVC 进行扩展

``` yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gluster-vol-default
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true
```

想要为 PVC 申请一个更大的卷，只需要修改 PVC 对象，设置一个更大的容量。 这会触发卷底层的 PV 的扩充。
这样做不会创建一个新的 PV 来满足这个 PVC， 而是通过修改原来的容量来实现。
<!--
#### CSI Volume expansion

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

Support for expanding CSI volumes is enabled by default but it also requires a specific CSI driver to support volume expansion. Refer to documentation of the specific CSI driver for more information.
 -->

#### CSI 卷扩容

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

对 CSI 卷扩展 扩容默认是开启的，但也是需要相应的 CSI 驱动支持卷扩容。 具体信息请参阅相应
CSI 驱动的文档。
<!--
#### Resizing a volume containing a file system

You can only resize volumes containing a file system if the file system is XFS, Ext3, or Ext4.

When a volume contains a file system, the file system is only resized when a new Pod is using
the PersistentVolumeClaim in `ReadWrite` mode. File system expansion is either done when a Pod is starting up
or when a Pod is running and the underlying file system supports online expansion.

FlexVolumes allow resize if the driver is set with the `RequiresFSResize` capability to `true`.
The FlexVolume can be resized on Pod restart.
 -->

#### 修改包含文件系统的卷的容量

如果郑文件系统是  XFS, Ext3, Ext4 之一，则可以修改其容量。

当卷中包含文件文件系统时，只有在
新的 Pod 使用 `ReadWrite` 模式的 PVC 时才会变更容量。文件系统的扩容只有在 Pod 启动或
正在运行的 Pod 底层的文件系统支持在线扩容时才能生效。

FlexVolume 允许在 驱动的 `RequiresFSResize` 设置为 `true` 时变更容量。
FlexVolume 的容量变更只能在 Pod 重启时生效。

<!--
#### Resizing an in-use PersistentVolumeClaim

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

{{< note >}}
Expanding in-use PVCs is available as beta since Kubernetes 1.15, and as alpha since 1.11. The `ExpandInUsePersistentVolumes` feature must be enabled, which is the case automatically for many clusters for beta features. Refer to the [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) documentation for more information.
{{< /note >}}

In this case, you don't need to delete and recreate a Pod or deployment that is using an existing PVC.
Any in-use PVC automatically becomes available to its Pod as soon as its file system has been expanded.
This feature has no effect on PVCs that are not in use by a Pod or deployment. You must create a Pod that
uses the PVC before the expansion can complete.


Similar to other volume types - FlexVolume volumes can also be expanded when in-use by a Pod.

{{< note >}}
FlexVolume resize is possible only when the underlying driver supports resize.
{{< /note >}}

{{< note >}}
Expanding EBS volumes is a time-consuming operation. Also, there is a per-volume quota of one modification every 6 hours.
{{< /note >}}
 -->

#### 对使用中的 PersistentVolumeClaim 变更容量

{{< feature-state for_k8s_version="v1.15" state="beta" >}}

{{< note >}}
对使用中的 PersistentVolumeClaim 变更容量 在 k8s `v1.15` 是 `beta` 状态，
`v1.11` 是 `alpha` 状态, `ExpandInUsePersistentVolumes` 在 `beta` 时是默认打开的。
更多信息请见
[功能阀]({{<base-origin path="/docs/reference/command-line-tools-reference/feature-gates/">}})
{{< /note >}}

在这种情况下，不需要删除或重新创建使用现有 PVC 的 Pod 或 Deployment. 任意使用中的 PVC 在
文件系统扩容后在 Pod 中自动变得可用。 这个特性对那些没有被 Pod 或 Deployment 的 PVC 无效。
要想扩容生效必须要有一个使用 PVC 的 Pod。

与其它的卷类型类似， FlexVolume 也可以在被 Pod 使用时扩容。
{{< note >}}
FlexVolume 变量容量只能在底层驱动支持的情况下才行。
{{< /note >}}

{{< note >}}
对 EBS 卷扩容是一个耗时的操作。并且还有一个每个卷每 6 个小时只能扩容的限制
{{< /note >}}
<!--
#### Recovering from Failure when Expanding Volumes

If expanding underlying storage fails, the cluster administrator can manually recover the Persistent Volume Claim (PVC) state and cancel the resize requests. Otherwise, the resize requests are continuously retried by the controller without administrator intervention.

1. Mark the PersistentVolume(PV) that is bound to the PersistentVolumeClaim(PVC) with `Retain` reclaim policy.
2. Delete the PVC. Since PV has `Retain` reclaim policy - we will not lose any data when we recreate the PVC.
3. Delete the `claimRef` entry from PV specs, so as new PVC can bind to it. This should make the PV `Available`.
4. Re-create the PVC with smaller size than PV and set `volumeName` field of the PVC to the name of the PV. This should bind new PVC to existing PV.
5. Don't forget to restore the reclaim policy of the PV.
 -->

#### 从卷扩展失败中恢复

如果底层存储扩容的失败，集群管理可以通过手动恢复 PVC 的状态，取消扩容请求。否则，在没管理员中止
的情况下控制器会不断地重试容量变更请求。

1. 将与 PersistentVolumeClaim(PVC) 绑定的 PersistentVolume(PV) 回收策略改为 `Retain`
2. 删除 PVC。 因为 PV 的回收策略是 `Retain`，所以在重新创建 PVC 之前数据不会丢失。
3. 删除 PV 对象中的 `claimRef` 实体， 这样新的 PVC 才可以与它绑定， 可以可以将 PV 变为 `Available`
4. 以小于 PV 的容量重新创建一个 PVC，这个 PVC 的 `volumeName` 设置 PV 的名称。这样就可以将
原来的 PV 绑定到新的 PVC 上
5. 不要忘记将 PV 的回收策略也改加原来的值
<!--
## Types of Persistent Volumes

PersistentVolume types are implemented as plugins.  Kubernetes currently supports the following plugins:

* GCEPersistentDisk
* AWSElasticBlockStore
* AzureFile
* AzureDisk
* CSI
* FC (Fibre Channel)
* FlexVolume
* Flocker
* NFS
* iSCSI
* RBD (Ceph Block Device)
* CephFS
* Cinder (OpenStack block storage)
* Glusterfs
* VsphereVolume
* Quobyte Volumes
* HostPath (Single node testing only -- local storage is not supported in any way and WILL NOT WORK in a multi-node cluster)
* Portworx Volumes
* ScaleIO Volumes
* StorageOS
 -->

## 持久化卷的类型

`PersistentVolume` 的类型随同实现插件。 目前 k8s 支持以下插件:

* GCEPersistentDisk
* AWSElasticBlockStore
* AzureFile
* AzureDisk
* CSI
* FC (Fibre Channel)
* FlexVolume
* Flocker
* NFS
* iSCSI
* RBD (Ceph Block Device)
* CephFS
* Cinder (OpenStack block storage)
* Glusterfs
* VsphereVolume
* Quobyte Volumes
* HostPath (只在单节点上测试过 -- 本地存储现在不支持多节点集群，将来也永远不会支持)
* Portworx Volumes
* ScaleIO Volumes
* StorageOS
<!--
## Persistent Volumes

Each PV contains a spec and status, which is the specification and status of the volume.
The name of a PersistentVolume object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

{{< note >}}
Helper programs relating to the volume type may be required for consumption of a PersistentVolume within a cluster.  In this example, the PersistentVolume is of type NFS and the helper program /sbin/mount.nfs is required to support the mounting of NFS filesystems.
{{< /note >}}
 -->

## PersistentVolume

每个 PV 包含一个 `spec` 和 `status`, 分别是对这个卷的配置定义和状态。 `PersistentVolume`
对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/concepts/overview/working-with-objects/names#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

{{< note >}}
Helper programs relating to the volume type may be required for consumption of a PersistentVolume within a cluster.  In this example, the PersistentVolume is of type NFS and the helper program /sbin/mount.nfs is required to support the mounting of NFS filesystems.
集群中辅助程序应该能够正确处理相关类型的卷。 在上面的例子中，PersistentVolume 的类型是 NFS
辅助程序 `/sbin/mount.nfs` 就需要支持挂载 NFS 文件系统
{{< /note >}}
<!--
### Capacity

Generally, a PV will have a specific storage capacity.  This is set using the PV's `capacity` attribute.  See the Kubernetes [Resource Model](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md) to understand the units expected by `capacity`.

Currently, storage size is the only resource that can be set or requested.  Future attributes may include IOPS, throughput, etc.
 -->

### 容量(Capacity)

通常，一个 PV 是有指定存储容量的。 这是通过 PV 的 `capacity` 属性设置的。 理解受 `capacity`
见 [Resource Model](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md)
Currently, storage size is the only resource that can be set or requested.  Future attributes may include IOPS, throughput, etc.
<!--
### Volume Mode

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

Kubernetes supports two `volumeModes` of PersistentVolumes: `Filesystem` and `Block`.

`volumeMode` is an optional API parameter.
`Filesystem` is the default mode used when `volumeMode` parameter is omitted.

A volume with `volumeMode: Filesystem` is *mounted* into Pods into a directory. If the volume
is backed by a block device and the device is empty, Kuberneretes creates a filesystem
on the device before mounting it for the first time.

You can set the value of `volumeMode` to `Block` to use a volume as a raw block device.
Such volume is presented into a Pod as a block device, without any filesystem on it.
This mode is useful to provide a Pod the fastest possible way to access a volume, without
any filesystem layer between the Pod and the volume. On the other hand, the application
running in the Pod must know how to handle a raw block device.
See [Raw Block Volume Support](#raw-block-volume-support)
for an example on how to use a volume with `volumeMode: Block` in a Pod.
 -->

### 卷模式

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

k8s 支持两种 PersistentVolume 卷模式(`volumeModes`): 文件系统(`Filesystem`) 和 块设备(`Block`)

`volumeMode` 是一个可选 API 参数
如果没有设置 `volumeMode`， 则默认使用 `Filesystem`

一个卷模式为 `volumeMode: Filesystem` 的卷 *挂载* 到 Pod 中的一个目录。 如果卷的后端是
一个块设备并且这个设备是空的， k8s 会在第一次挂载之前在块设备上创建一个文件系统。

也可以将 `volumeMode` 的值设置为 `Block` 以块设备的方式来使用这个卷。 这个卷就会以块设备的
形式存在于 Pod 中， 其中不会有任何文件系统。 这种方式在为 Pod 提供该卷可能最佳的访问速度，
在 Pod 和 卷之间没有任何文件系统层。 另一方面， Pod 中运行的应用必须要能正确使用块设备。

可以在
[块设备卷支持](#raw-block-volume-support)
查看使用 `volumeMode: Block` 卷模式卷的 Pod 。
<!--
### Access Modes

A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume.  For example, NFS can support multiple read/write clients, but a specific NFS PV might be exported on the server as read-only. Each PV gets its own set of access modes describing that specific PV's capabilities.

The access modes are:

* ReadWriteOnce -- the volume can be mounted as read-write by a single node
* ReadOnlyMany -- the volume can be mounted read-only by many nodes
* ReadWriteMany -- the volume can be mounted as read-write by many nodes

In the CLI, the access modes are abbreviated to:

* RWO - ReadWriteOnce
* ROX - ReadOnlyMany
* RWX - ReadWriteMany

> __Important!__ A volume can only be mounted using one access mode at a time, even if it supports many.  For example, a GCEPersistentDisk can be mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes, but not at the same time.


| Volume Plugin        | ReadWriteOnce          | ReadOnlyMany          | ReadWriteMany|
| :---                 | :---:                  | :---:                 | :---:        |
| AWSElasticBlockStore | &#x2713;               | -                     | -            |
| AzureFile            | &#x2713;               | &#x2713;              | &#x2713;     |
| AzureDisk            | &#x2713;               | -                     | -            |
| CephFS               | &#x2713;               | &#x2713;              | &#x2713;     |
| Cinder               | &#x2713;               | -                     | -            |
| CSI                  | depends on the driver  | depends on the driver | depends on the driver |
| FC                   | &#x2713;               | &#x2713;              | -            |
| FlexVolume           | &#x2713;               | &#x2713;              | depends on the driver |
| Flocker              | &#x2713;               | -                     | -            |
| GCEPersistentDisk    | &#x2713;               | &#x2713;              | -            |
| Glusterfs            | &#x2713;               | &#x2713;              | &#x2713;     |
| HostPath             | &#x2713;               | -                     | -            |
| iSCSI                | &#x2713;               | &#x2713;              | -            |
| Quobyte              | &#x2713;               | &#x2713;              | &#x2713;     |
| NFS                  | &#x2713;               | &#x2713;              | &#x2713;     |
| RBD                  | &#x2713;               | &#x2713;              | -            |
| VsphereVolume        | &#x2713;               | -                     | - (works when Pods are collocated)  |
| PortworxVolume       | &#x2713;               | -                     | &#x2713;     |
| ScaleIO              | &#x2713;               | &#x2713;              | -            |
| StorageOS            | &#x2713;               | -                     | -            |
 -->

### 访问模式

A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume.  For example, NFS can support multiple read/write clients, but a specific NFS PV might be exported on the server as read-only. Each PV gets its own set of access modes describing that specific PV's capabilities.

`PersistentVolume` 可以以任意资源提供者支持的方式挂载到主机中。 如下表所示， 不同的提供者
拥有不同能力，每个 PV 的模式可以在对应卷上设置支持的模式。 例如， NFS 支持多客户端读写,但是可以
将一个 NFS PV 以只读方式提供。 每个 PV 都可以独立设置各自的方式模式。

访问模式有:

* ReadWriteOnce -- 卷只能被一个消费者以只读方式挂载
* ReadOnlyMany -- 卷能被多个消费者以只读方式挂载
* ReadWriteMany -- 卷能被多个消费者以读写方式挂载

在命令，这些模式的简写如下:

* RWO - ReadWriteOnce
* ROX - ReadOnlyMany
* RWX - ReadWriteMany

> __重要!__ 一个卷一次只能以一种访问模式挂载，即便它是支持多种的。 例如， 一个 GCEPersistentDisk 可以
在一个消费者挂载为 ReadWriteOnce 或可以在多个消费者挂载为 ReadOnlyMany， 但不能同时挂载
ReadWriteOnce 和 ReadOnlyMany。

| Volume Plugin        | ReadWriteOnce          | ReadOnlyMany          | ReadWriteMany|
| :---                 | :---:                  | :---:                 | :---:        |
| AWSElasticBlockStore | &#x2713;               | -                     | -            |
| AzureFile            | &#x2713;               | &#x2713;              | &#x2713;     |
| AzureDisk            | &#x2713;               | -                     | -            |
| CephFS               | &#x2713;               | &#x2713;              | &#x2713;     |
| Cinder               | &#x2713;               | -                     | -            |
| CSI                  | 基于驱动                 | 基于驱动               | 基于驱动      |
| FC                   | &#x2713;               | &#x2713;              | -            |
| FlexVolume           | &#x2713;               | &#x2713;              | 基于驱动      |
| Flocker              | &#x2713;               | -                     | -            |
| GCEPersistentDisk    | &#x2713;               | &#x2713;              | -            |
| Glusterfs            | &#x2713;               | &#x2713;              | &#x2713;     |
| HostPath             | &#x2713;               | -                     | -            |
| iSCSI                | &#x2713;               | &#x2713;              | -            |
| Quobyte              | &#x2713;               | &#x2713;              | &#x2713;     |
| NFS                  | &#x2713;               | &#x2713;              | &#x2713;     |
| RBD                  | &#x2713;               | &#x2713;              | -            |
| VsphereVolume        | &#x2713;               | -                     | - (works when Pods are collocated)  |
| PortworxVolume       | &#x2713;               | -                     | &#x2713;     |
| ScaleIO              | &#x2713;               | &#x2713;              | -            |
| StorageOS            | &#x2713;               | -                     | -            |
<!--
### Class

A PV can have a class, which is specified by setting the
`storageClassName` attribute to the name of a
[StorageClass](/docs/concepts/storage/storage-classes/).
A PV of a particular class can only be bound to PVCs requesting
that class. A PV with no `storageClassName` has no class and can only be bound
to PVCs that request no particular class.

In the past, the annotation `volume.beta.kubernetes.io/storage-class` was used instead
of the `storageClassName` attribute. This annotation is still working; however,
it will become fully deprecated in a future Kubernetes release.
 -->
### 类别 {#class}

一个 PV 可以有一个类别， 可以通过 `storageClassName` 属性来设置一个
[StorageClass](/k8sDocs/concepts/storage/storage-classes/).
一个指定类别的 PV 只能与请求对应类的 PVC 相绑定。 没有设置 `storageClassName` 的 PV 是没有类别的
并且只能与没有请求类别的 PVC 绑定。

在过去是通过 `volume.beta.kubernetes.io/storage-class` 注解设置类别而不是 `storageClassName`。
这个注解目前还能用；但在未来的版本中会完全废弃。
<!--
### Reclaim Policy

Current reclaim policies are:

* Retain -- manual reclamation
* Recycle -- basic scrub (`rm -rf /thevolume/*`)
* Delete -- associated storage asset such as AWS EBS, GCE PD, Azure Disk, or OpenStack Cinder volume is deleted

Currently, only NFS and HostPath support recycling. AWS EBS, GCE PD, Azure Disk, and Cinder volumes support deletion.
 -->

### 回收策略 {#reclain-policy}

目前支持的回收策略:

* Retain -- 手动回收
* Recycle -- 基础清理 (`rm -rf /thevolume/*`)
* Delete -- 关联存储资源如 AWS EBS, GCE PD, Azure Disk, OpenStack Cinder 卷也会被删除。

目前只有 NFS 和 HostPath 支持 `Recycle`
AWS EBS, GCE PD, Azure Disk, and Cinder 卷支持 `Delete`
<!--
### Mount Options

A Kubernetes administrator can specify additional mount options for when a Persistent Volume is mounted on a node.

{{< note >}}
Not all Persistent Volume types support mount options.
{{< /note >}}

The following volume types support mount options:

* AWSElasticBlockStore
* AzureDisk
* AzureFile
* CephFS
* Cinder (OpenStack block storage)
* GCEPersistentDisk
* Glusterfs
* NFS
* Quobyte Volumes
* RBD (Ceph Block Device)
* StorageOS
* VsphereVolume
* iSCSI

Mount options are not validated, so mount will simply fail if one is invalid.

In the past, the annotation `volume.beta.kubernetes.io/mount-options` was used instead
of the `mountOptions` attribute. This annotation is still working; however,
it will become fully deprecated in a future Kubernetes release.
 -->

### 挂载选项

k8s 管理员可以在 PV 挂载时指定额外的挂载选项。
{{< note >}}
不是所有的 PV 类型都支持挂载选项
{{< /note >}}

以下卷类型支持挂载选项:

* AWSElasticBlockStore
* AzureDisk
* AzureFile
* CephFS
* Cinder (OpenStack block storage)
* GCEPersistentDisk
* Glusterfs
* NFS
* Quobyte Volumes
* RBD (Ceph Block Device)
* StorageOS
* VsphereVolume
* iSCSI

挂载选项不会验证，如果其中有无效选项就会挂载失败。

在过去 使用的是 `volume.beta.kubernetes.io/mount-options` 注解设置挂载选项，而不是 `mountOptions`
这个注解目前还可以用，但在未来的版本中会完全废弃。
<!--
### Node Affinity

{{< note >}}
For most volume types, you do not need to set this field. It is automatically populated for [AWS EBS](/docs/concepts/storage/volumes/#awselasticblockstore), [GCE PD](/docs/concepts/storage/volumes/#gcepersistentdisk) and [Azure Disk](/docs/concepts/storage/volumes/#azuredisk) volume block types. You need to explicitly set this for [local](/docs/concepts/storage/volumes/#local) volumes.
{{< /note >}}

A PV can specify [node affinity](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#volumenodeaffinity-v1-core) to define constraints that limit what nodes this volume can be accessed from. Pods that use a PV will only be scheduled to nodes that are selected by the node affinity.
 -->

### 节点亲和性

{{< note >}}
对于大多数卷类型， 是不需要设置这个字段的。
[AWS EBS](/k8sDocs/concepts/storage/volumes/#awselasticblockstore),
[GCE PD](/k8sDocs/concepts/storage/volumes/#gcepersistentdisk),
[Azure Disk](/k8sDocs/concepts/storage/volumes/#azuredisk)
卷块设备会自动添加。
但需要为
[local](/k8sDocs/concepts/storage/volumes/#local)
卷需要显示设置该字段
{{< /note >}}

PV 可以通过设置
[节点亲和性](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param \"version\" >}}/#volumenodeaffinity-v1-core)
来定义限制哪些节点能够访问这个卷。
Pod 只会被调度到节点亲和性选择的节点上。
<!--
### Phase

A volume will be in one of the following phases:

* Available -- a free resource that is not yet bound to a claim
* Bound -- the volume is bound to a claim
* Released -- the claim has been deleted, but the resource is not yet reclaimed by the cluster
* Failed -- the volume has failed its automatic reclamation

The CLI will show the name of the PVC bound to the PV.
 -->

### 阶段

一个卷会处于以下阶段中的一个:

* Available -- 一个空闲资源，还没有被绑定到 PVC
* Bound -- 这个卷被绑定了一个 PVC
* Released -- 绑定的 PVC 已经被删除，但资源还没有被集群回收
* Failed -- 这个卷自动回收失败

命令行可以显示与 PV 绑定的 PVC 的名称
<!--
## PersistentVolumeClaims

Each PVC contains a spec and status, which is the specification and status of the claim.
The name of a PersistentVolumeClaim object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```
 -->

## PersistentVolumeClaim

每个 PVC 包含 `spec` 和 `status`, 其中包含 PVC 的配置定义和状态。
`PersistentVolumeClaim` 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/concepts/overview/working-with-objects/names#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```
<!--
### Access Modes

Claims use the same conventions as volumes when requesting storage with specific access modes.
 -->

### 访问模式

Claims use the same conventions as volumes when requesting storage with specific access modes.
PVC 的访问模式与卷在请求存储时指定的访问模式一至
<!--
### Volume Modes

Claims use the same convention as volumes to indicate the consumption of the volume as either a filesystem or block device.
 -->

### 卷模式

PVC 的卷模式与卷指定的卷模式一至，可以是 文件系统 或 块设备
<!--
### Resources

Claims, like Pods, can request specific quantities of a resource. In this case, the request is for storage. The same [resource model](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md) applies to both volumes and claims.
 -->

### 资源

PVC 与 Pod 类似， 可以请求指定数量的资源。 在这种情况下， 请求是提存储。
[资源模式](https://git.k8s.io/community/contributors/design-proposals/scheduling/resources.md)
可应用到卷和 PVC
<!--
### Selector

Claims can specify a [label selector](/docs/concepts/overview/working-with-objects/labels/#label-selectors) to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim. The selector can consist of two fields:

* `matchLabels` - the volume must have a label with this value
* `matchExpressions` - a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include In, NotIn, Exists, and DoesNotExist.

All of the requirements, from both `matchLabels` and `matchExpressions`, are ANDed together – they must all be satisfied in order to match.
 -->

### 选择器

PVC 可以指定一个
[标签选择器](/k8sDocs/concepts/overview/working-with-objects/labels/#label-selectors)
来选择卷集合。 只有与选择器匹配的卷可以与 PVC 绑定。 选择器可以包含以下两个字段

* `matchLabels` - 卷必须要拥有与其对应的标签
* `matchExpressions` - 一个由指定键和值列表，以及键和值相应的操作符组成的条件列表，
可用的操作符包含 `In`, `NotIn`, `Exists`, `DoesNotExist`

来自 `matchLabels` 和 `matchExpressions` 的所有条件以逻辑与方式组合。
必须要满足所有条件才能匹配到。
<!--
### Class

A claim can request a particular class by specifying the name of a
[StorageClass](/docs/concepts/storage/storage-classes/)
using the attribute `storageClassName`.
Only PVs of the requested class, ones with the same `storageClassName` as the PVC, can
be bound to the PVC.

PVCs don't necessarily have to request a class. A PVC with its `storageClassName` set
equal to `""` is always interpreted to be requesting a PV with no class, so it
can only be bound to PVs with no class (no annotation or one set equal to
`""`). A PVC with no `storageClassName` is not quite the same and is treated differently
by the cluster, depending on whether the
[`DefaultStorageClass` admission plugin](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
is turned on.

* If the admission plugin is turned on, the administrator may specify a
  default StorageClass. All PVCs that have no `storageClassName` can be bound only to
  PVs of that default. Specifying a default StorageClass is done by setting the
  annotation `storageclass.kubernetes.io/is-default-class` equal to `true` in
  a StorageClass object. If the administrator does not specify a default, the
  cluster responds to PVC creation as if the admission plugin were turned off. If
  more than one default is specified, the admission plugin forbids the creation of
  all PVCs.
* If the admission plugin is turned off, there is no notion of a default
  StorageClass. All PVCs that have no `storageClassName` can be bound only to PVs that
  have no class. In this case, the PVCs that have no `storageClassName` are treated the
  same way as PVCs that have their `storageClassName` set to `""`.

Depending on installation method, a default StorageClass may be deployed
to a Kubernetes cluster by addon manager during installation.

When a PVC specifies a `selector` in addition to requesting a StorageClass,
the requirements are ANDed together: only a PV of the requested class and with
the requested labels may be bound to the PVC.

{{< note >}}
Currently, a PVC with a non-empty `selector` can't have a PV dynamically provisioned for it.
{{< /note >}}

In the past, the annotation `volume.beta.kubernetes.io/storage-class` was used instead
of `storageClassName` attribute. This annotation is still working; however,
it won't be supported in a future Kubernetes release.
 -->

### 类别

PVC 可以通过设置 `storageClassName` 的值为一个
[StorageClass](/docs/concepts/storage/storage-classes/)
的名称来指定其使用该类别。
PV 与 PVC 只有在拥有相同 `storageClassName` 的情况下才能绑定。

PVC 并不是必须要设置一个类别。 PVC 可以将 `storageClassName` 设置为  `""`， 表示请求一个
没有类别的 PV，因此它也只能与没有类别的 PV 绑定(没有类别注解或类别注解值为 `""`)。
如果 PVC 没有设置 `storageClassName`，则会基于是否开启
[`DefaultStorageClass` admission plugin](/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass)
以下情况而有不同的表现行为:

- 如果准入插件是开启的， 则管理员可以设置一个默认的 `StorageClass`。 所有没有设置 `storageClassName`
  的 PVC 就只能与这个默认类别的 PV 绑定。 通过将 `StorageClass` 对象的
  `storageclass.kubernetes.io/is-default-class` 注解值设置为 `true` 可以将其设置默认。
  如果管理不有设置默认类别， 集群应答 PVC 创建操作与准入插件关闭相同。 如果设置了不只一个默认
  类别，则准入插件会阻止所有 PVC 的创建
- 如果准入插件没有打开， 那就没有默认 `StorageClass` 这回事。 所有没有设置 `storageClassName`
  的 PVC 就只能与没有设置类型的 PV 绑定。 在这种情况下， 没有设置 `storageClassName` 与
  将 `storageClassName` 设置为 `""` 的处理方式是一样的。

基于安装方式， 默认 `StorageClass` 可以被插件管理器在安装时加入集群。

当 PVC 设置 `selector` 来请求 StorageClass， 所有条件是逻辑与关系: 只有包含所有选择器需要
的标签的类型才能被绑定的 PVC。
{{< note >}}
目前，如果 PVC 的 `selector` 是空，则不能实现动态管理 PV
{{< /note >}}

在过去， 用的是 `volume.beta.kubernetes.io/storage-class` 注解而不是 `storageClassName` 属性。
这个注解目前还有用，但在未来版本中会完全废弃。


<!--
## Claims As Volumes

Pods access storage by using the claim as a volume. Claims must exist in the same namespace as the Pod using the claim. The cluster finds the claim in the Pod's namespace and uses it to get the PersistentVolume backing the claim. The volume is then mounted to the host and into the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
 -->

## 将 PVC 当作卷(PV) {#claims-as-volumes}

Pod 可以将 PVC 当作卷(PV) 来用作存储。 被引用 PVC 必须要要与 Pod 在同一个命名空间。
集群会在 Pod 所在的命名空间中寻找 PVC 然后使用与其绑定的 PV。 最后将 PV 挂载到主机，最终挂载到
Pod 中。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
<!--
### A Note on Namespaces

PersistentVolumes binds are exclusive, and since PersistentVolumeClaims are namespaced objects, mounting claims with "Many" modes (`ROX`, `RWX`) is only possible within one namespace.
 -->
### 一个需要注意命名空间的问题

PersistentVolume 的绑定是独占的，而又因为 PVC 是命名空间级别的对象，因此在对 PVC 进行多节点
模式(`ROX`, `RWX`)挂载时只能针对同一个命名空间的节点。
<!--
## Raw Block Volume Support

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

The following volume plugins support raw block volumes, including dynamic provisioning where
applicable:

* AWSElasticBlockStore
* AzureDisk
* CSI
* FC (Fibre Channel)
* GCEPersistentDisk
* iSCSI
* Local volume
* OpenStack Cinder
* RBD (Ceph Block Device)
* VsphereVolume
 -->

## 块设备卷支持 {#raw-block-volume-support}

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

以下卷插件支持块设备卷，包含适配的动态管理:

* AWSElasticBlockStore
* AzureDisk
* CSI
* FC (Fibre Channel)
* GCEPersistentDisk
* iSCSI
* Local volume
* OpenStack Cinder
* RBD (Ceph Block Device)
* VsphereVolume
<!--
### PersistentVolume using a Raw Block Volume {#persistent-volume-using-a-raw-block-volume}

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```
 -->
### 使用块设备卷的 PersistentVolume {#persistent-volume-using-a-raw-block-volume}

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```
<!--
### PersistentVolumeClaim requesting a Raw Block Volume {#persistent-volume-claim-requesting-a-raw-block-volume}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```
 -->

### 申请块设备卷的 PersistentVolumeClaim {#persistent-volume-claim-requesting-a-raw-block-volume}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```
<!--
### Pod specification adding Raw Block Device path in container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```

{{< note >}}
When adding a raw block device for a Pod, you specify the device path in the container instead of a mount path.
{{< /note >}}
 -->

### 在 Pod 定义中添加容器中的块设备路径

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```

{{< note >}}
在为一个 Pod 添加块设备时，在容器中指定的是设备路径，而不是挂载路径
{{< /note >}}
<!--
### Binding Block Volumes

If a user requests a raw block volume by indicating this using the `volumeMode` field in the PersistentVolumeClaim spec, the binding rules differ slightly from previous releases that didn't consider this mode as part of the spec.
Listed is a table of possible combinations the user and admin might specify for requesting a raw block device. The table indicates if the volume will be bound or not given the combinations:
Volume binding matrix for statically provisioned volumes:

| PV volumeMode | PVC volumeMode  | Result           |
| --------------|:---------------:| ----------------:|
|   unspecified | unspecified     | BIND             |
|   unspecified | Block           | NO BIND          |
|   unspecified | Filesystem      | BIND             |
|   Block       | unspecified     | NO BIND          |
|   Block       | Block           | BIND             |
|   Block       | Filesystem      | NO BIND          |
|   Filesystem  | Filesystem      | BIND             |
|   Filesystem  | Block           | NO BIND          |
|   Filesystem  | unspecified     | BIND             |

{{< note >}}
Only statically provisioned volumes are supported for alpha release. Administrators should take care to consider these values when working with raw block devices.
{{< /note >}}
 -->

### 绑定块设备

如果用户在 PVC 配置中使用 `volumeMode` 字段指定申请一个块设备卷，则绑定规则与之前版本中
没有在配置中指定 `volumeMode` 是有点不一样的。

以下为用户/管理员在申请块设备时可能的组合列表。 这个表展示了这个组合卷是否能绑定
静态管理卷的绑定矩阵:

| PV volumeMode | PVC volumeMode  | Result           |
| --------------|:---------------:| ----------------:|
|   未指定       | 未指定           | BIND             |
|   未指定       | Block           | NO BIND          |
|   未指定       | Filesystem      | BIND             |
|   Block       | 未指定           | NO BIND          |
|   Block       | Block           | BIND             |
|   Block       | Filesystem      | NO BIND          |
|   Filesystem  | Filesystem      | BIND             |
|   Filesystem  | Block           | NO BIND          |
|   Filesystem  | 未指定           | BIND             |

{{< note >}}
在 alpha 特性中只有静态管理的卷被支持。 管理员在操作块设备需要仔细考虑这些值。
{{< /note >}}
<!--
## Volume Snapshot and Restore Volume from Snapshot Support

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

Volume snapshot feature was added to support CSI Volume Plugins only. For details, see [volume snapshots](/docs/concepts/storage/volume-snapshots/).

To enable support for restoring a volume from a volume snapshot data source, enable the
`VolumeSnapshotDataSource` feature gate on the apiserver and controller-manager.
 -->

## 卷快照和快照恢复支持

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

卷快照特性只对 CSI 卷插件支持。详细信息见
 [卷快照](/k8sDocs/concepts/storage/volume-snapshots/).
要启用对从卷快照恢复的支持，需要在 `apiserver` `controller-manager` 打开
`VolumeSnapshotDataSource` 功能阀
<!--
### Create a PersistentVolumeClaim from a Volume Snapshot {#create-persistent-volume-claim-from-volume-snapshot}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restore-pvc
spec:
  storageClassName: csi-hostpath-sc
  dataSource:
    name: new-snapshot-test
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
 -->

### 基于卷快照创建 PVC {#create-persistent-volume-claim-from-volume-snapshot}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restore-pvc
spec:
  storageClassName: csi-hostpath-sc
  dataSource:
    name: new-snapshot-test
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
<!--
## Volume Cloning

[Volume Cloning](/docs/concepts/storage/volume-pvc-datasource/) only available for CSI volume plugins.
 -->

## 卷克隆

[卷克隆](/k8sDocs/concepts/storage/volume-pvc-datasource/) 只存在于 CSI 卷插件
<!--
### Create PersistentVolumeClaim from an existing PVC {#create-persistent-volume-claim-from-an-existing-pvc}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  storageClassName: my-csi-plugin
  dataSource:
    name: existing-src-pvc-name
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
 -->

### 基于存在的 PVC 创建新的 PVC {#create-persistent-volume-claim-from-an-existing-pvc}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  storageClassName: my-csi-plugin
  dataSource:
    name: existing-src-pvc-name
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
<!--
## Writing Portable Configuration

If you're writing configuration templates or examples that run on a wide range of clusters
and need persistent storage, it is recommended that you use the following pattern:

- Include PersistentVolumeClaim objects in your bundle of config (alongside
  Deployments, ConfigMaps, etc).
- Do not include PersistentVolume objects in the config, since the user instantiating
  the config may not have permission to create PersistentVolumes.
- Give the user the option of providing a storage class name when instantiating
  the template.
  - If the user provides a storage class name, put that value into the
    `persistentVolumeClaim.storageClassName` field.
    This will cause the PVC to match the right storage
    class if the cluster has StorageClasses enabled by the admin.
  - If the user does not provide a storage class name, leave the
    `persistentVolumeClaim.storageClassName` field as nil. This will cause a
    PV to be automatically provisioned for the user with the default StorageClass
    in the cluster. Many cluster environments have a default StorageClass installed,
    or administrators can create their own default StorageClass.
- In your tooling, watch for PVCs that are not getting bound after some time
  and surface this to the user, as this may indicate that the cluster has no
  dynamic storage support (in which case the user should create a matching PV)
  or the cluster has no storage system (in which case the user cannot deploy
  config requiring PVCs).

  ## {{% heading "whatsnext" %}}


* Learn more about [Creating a PersistentVolume](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).
* Learn more about [Creating a PersistentVolumeClaim](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim).
* Read the [Persistent Storage design document](https://git.k8s.io/community/contributors/design-proposals/storage/persistent-storage.md).
 -->

## 编写可移植的配置

如果要编写运行在大范围集群中并且需要使用到持久化存储的模板或示例配置，建议依照以下模式:

- 在配置 (随同 Deployments, ConfigMaps, 等)时在同一个配置文件中包含其使用的 PVC 对象。

- 如果使用配置的用户可能没有创建 PV 的权限，则不要在配置中包含 PV 对象。

- 在用户使用模板时，提供设置 StorageClass 名称的选项
  - 如果用户通过 `persistentVolumeClaim.storageClassName`  字段设置了 StorageClass 名称。
    当集群管理员启用了该 StorageClasses 时， PVC 就能正确使用存储类别。
  - 如果用户没提供 `StorageClass` 名称。 这会导致 `persistentVolumeClaim.storageClassName`
    值为空。 这样集群中 PV 就会使用默认 `StorageClass` 自动管理。 许多集群环境中都都有安装一个
    默认的 `StorageClass`或管理可能创建自己的默认 `StorageClass`

- 在工具中，在一个时间后观测未绑定的 PVC 并将其传递给用户。 这可能是集群不支持动态存储(这样用户
  就要自己创建相应的 PV) 或者集群中没有存储系统(这种情况用户就不能部署包含 PVC 的配置)
## {{% heading "whatsnext" %}}

<!--
* Learn more about [Creating a PersistentVolume](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).
* Learn more about [Creating a PersistentVolumeClaim](/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim).
* Read the [Persistent Storage design document](https://git.k8s.io/community/contributors/design-proposals/storage/persistent-storage.md).
 -->
* 实践 [创建 PersistentVolume](/k8sDocs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume).
* 实践 [创建 PersistentVolumeClaim](/k8sDocs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim).
* 参阅 [持久化存储设计文稿](https://git.k8s.io/community/contributors/design-proposals/storage/persistent-storage.md).

### Reference

* [PersistentVolume](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#persistentvolume-v1-core)
* [PersistentVolumeSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#persistentvolumespec-v1-core)
* [PersistentVolumeClaim](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#persistentvolumeclaim-v1-core)
* [PersistentVolumeClaimSpec](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#persistentvolumeclaimspec-v1-core)
