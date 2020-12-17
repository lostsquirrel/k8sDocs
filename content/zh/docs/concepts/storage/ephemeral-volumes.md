---
title: 临时卷
content_type: concept
weight: 90
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- msau42
- xing-yang
- pohly
title: Ephemeral Volumes
content_type: concept
weight: 50
---
 -->
<!-- overview -->
<!--
This document describes _ephemeral volumes_ in Kubernetes. Familiarity
with [volumes](/docs/concepts/storage/volumes/) is suggested, in
particular PersistentVolumeClaim and PersistentVolume.
 -->
本文主要介绍 k8s 中的 _临时卷(ephemeral volume)_。 建议先熟悉
[卷(Volume)](/k8sDocs/docs/concepts/storage/volumes/)
特别是 PersistentVolumeClaim 和 持久化卷(PV)
<!-- body -->

Some application need additional storage but don't care whether that
data is stored persistently across restarts. For example, caching
services are often limited by memory size and can move infrequently
used data into storage that is slower than memory with little impact
on overall performance.

有些应用需要额外的存储，但不关心其中的数据在重启之后是不是还在。 例如，缓存服务经常受限于内存大小
可以将不常用的数据移到其它比内存慢的存储中，这样对全局情况影响比较小。
Other applications expect some read-only input data to be present in
files, like configuration data or secret keys.

另一些应用可能需要一个存放在文件中的只读输入数据， 例如配置数据或 Secret 的键。

_Ephemeral volumes_ are designed for these use cases. Because volumes
follow the Pod's lifetime and get created and deleted along with the
Pod, Pods can be stopped and restarted without being limited to where
some persistent volume is available.

_临时卷_ 就是为这些应用场景设计。因为卷跟随 Pod 的生命周期，随同 Pod 一起创建和删除， Pod 可以
在不再受限于那些只有可用卷的地方停止或重启。

Ephemeral volumes are specified _inline_ in the Pod spec, which
simplifies application deployment and management.

临时卷是在 Pod 定义中单行配置，这样可以简化应用的部署和管理。
<!--
### Types of ephemeral volumes

Kubernetes supports several different kinds of ephemeral volumes for
different purposes:
- [emptyDir](/docs/concepts/storage/volumes/#emptydir): empty at Pod startup,
  with storage coming locally from the kubelet base directory (usually
  the root disk) or RAM
- [configMap](/docs/concepts/storage/volumes/#configmap),
  [downwardAPI](/docs/concepts/storage/volumes/#downwardapi),
  [secret](/docs/concepts/storage/volumes/#secret): inject different
  kinds of Kubernetes data into a Pod
- [CSI ephemeral volumes](#csi-ephemeral-volumes):
  similar to the previous volume kinds, but provided by special
  [CSI drivers](https://github.com/container-storage-interface/spec/blob/master/spec.md)
  which specifically [support this feature](https://kubernetes-csi.github.io/docs/drivers.html)
- [generic ephemeral volumes](#generic-ephemeral-volumes), which
  can be provided by all storage drivers that also support persistent volumes

`emptyDir`, `configMap`, `downwardAPI`, `secret` are provided as
[local ephemeral
storage](/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage).
They are managed by kubelet on each node.

CSI ephemeral volumes *must* be provided by third-party CSI storage
drivers.

Generic ephemeral volumes *can* be provided by third-party CSI storage
drivers, but also by any other storage driver that supports dynamic
provisioning. Some CSI drivers are written specifically for CSI
ephemeral volumes and do not support dynamic provisioning: those then
cannot be used for generic ephemeral volumes.

The advantage of using third-party drivers is that they can offer
functionality that Kubernetes itself does not support, for example
storage with different performance characteristics than the disk that
is managed by kubelet, or injecting different data.
 -->

### 临时卷的类型

k8s 支持几种不同类型的临时卷用于不同目的:
- [emptyDir](/docs/concepts/storage/volumes/#emptydir): 在 Pod 启动时是空的，来自
  kubelet 基础目录的本地存储(通常是系统盘)或内存

- [configMap](/docs/concepts/storage/volumes/#configmap),
  [downwardAPI](/docs/concepts/storage/volumes/#downwardapi),
  [secret](/docs/concepts/storage/volumes/#secret): 注入不同类型的 k8s 数据到 Pod 中

- [CSI 临时卷](#csi-ephemeral-volumes): 与上一个类型相似， 但由特殊的
  [CSI 驱动](https://github.com/container-storage-interface/spec/blob/master/spec.md)
  提供，并且要
  [支持该特性](https://kubernetes-csi.github.io/docs/drivers.html)
- [通用临时卷](#generic-ephemeral-volumes)， 可以由所有可以支持持久化卷(PV)的存储驱动提供

`emptyDir`, `configMap`, `downwardAPI`, `secret`  是以
[本地临时卷](/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage)
提供。
它们由每个节点上的 kubelet 管理。

CSI 临时卷*必须*由第三方 CSI 存储驱动提供的。

通用临时卷*可以*由第三方 CSI 存储驱动提供的，但也可以由其它支持动态供应的存储驱动提供。
有些 CSI 驱动是 CSI 临时卷专用的，并不支持动态供应: 这些就不能用于通用临时卷。

用第三方驱动的好处是它们能提供一个 k8s 本身不支持的功能， 例如除由 kubelet 管理外的磁盘外的
其它拥有不同性能特性的存储或插入不同的数据。
<!--
### CSI ephemeral volumes

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

This feature requires the `CSIInlineVolume` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) to be enabled. It
is enabled by default starting with Kubernetes 1.16.

{{< note >}}
CSI ephemeral volumes are only supported by a subset of CSI drivers.
The Kubernetes CSI [Drivers list](https://kubernetes-csi.github.io/docs/drivers.html)
shows which drivers support ephemeral volumes.
{{< /note >}}

Conceptually, CSI ephemeral volumes are similar to `configMap`,
`downwardAPI` and `secret` volume types: the storage is managed locally on each
 node and is created together with other local resources after a Pod has been
scheduled onto a node. Kubernetes has no concept of rescheduling Pods
anymore at this stage. Volume creation has to be unlikely to fail,
otherwise Pod startup gets stuck. In particular, [storage capacity
aware Pod scheduling](/docs/concepts/storage/storage-capacity/) is *not*
supported for these volumes. They are currently also not covered by
the storage resource usage limits of a Pod, because that is something
that kubelet can only enforce for storage that it manages itself.


Here's an example manifest for a Pod that uses CSI ephemeral storage:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-csi-app
spec:
  containers:
    - name: my-frontend
      image: busybox
      volumeMounts:
      - mountPath: "/data"
        name: my-csi-inline-vol
      command: [ "sleep", "1000000" ]
  volumes:
    - name: my-csi-inline-vol
      csi:
        driver: inline.storage.kubernetes.io
        volumeAttributes:
          foo: bar
```

The `volumeAttributes` determine what volume is prepared by the
driver. These attributes are specific to each driver and not
standardized. See the documentation of each CSI driver for further
instructions.

As a cluster administrator, you can use a [PodSecurityPolicy](/docs/concepts/policy/pod-security-policy/) to control which CSI drivers can be used in a Pod, specified with the
[`allowedCSIDrivers` field](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podsecuritypolicyspec-v1beta1-policy).
 -->

### CSI 临时卷

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

This feature requires the `CSIInlineVolume` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) to be enabled. It
is enabled by default starting with Kubernetes 1.16.
这个特性需要启用 `CSIInlineVolume`
[功能阀](/docs/reference/command-line-tools-reference/feature-gates/)。
从 k8s v1.16 开始该功能默认是开启的。

{{< note >}}
CSI 临时卷只被一部分 CSI 驱动支持。 这个 k8s CSI
[驱动列表](https://kubernetes-csi.github.io/docs/drivers.html)
中显示了哪些驱动支持临时卷。
{{< /note >}}

Conceptually, CSI ephemeral volumes are similar to `configMap`,
`downwardAPI` and `secret` volume types: the storage is managed locally on each
 node and is created together with other local resources after a Pod has been
scheduled onto a node. Kubernetes has no concept of rescheduling Pods
anymore at this stage. Volume creation has to be unlikely to fail,
otherwise Pod startup gets stuck. In particular, [storage capacity
aware Pod scheduling](/docs/concepts/storage/storage-capacity/) is *not*
supported for these volumes. They are currently also not covered by
the storage resource usage limits of a Pod, because that is something
that kubelet can only enforce for storage that it manages itself.

在概念上， CSI 临时卷与 `configMap`, `downwardAPI` 和 `secret` 这些类型的卷相似:
存储由每个节点本地管理并且在 Pod 被调度到节点上时随同其它本地资源一起创建。

Here's an example manifest for a Pod that uses CSI ephemeral storage:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-csi-app
spec:
  containers:
    - name: my-frontend
      image: busybox
      volumeMounts:
      - mountPath: "/data"
        name: my-csi-inline-vol
      command: [ "sleep", "1000000" ]
  volumes:
    - name: my-csi-inline-vol
      csi:
        driver: inline.storage.kubernetes.io
        volumeAttributes:
          foo: bar
```

The `volumeAttributes` determine what volume is prepared by the
driver. These attributes are specific to each driver and not
standardized. See the documentation of each CSI driver for further
instructions.

As a cluster administrator, you can use a [PodSecurityPolicy](/docs/concepts/policy/pod-security-policy/) to control which CSI drivers can be used in a Pod, specified with the
[`allowedCSIDrivers` field](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podsecuritypolicyspec-v1beta1-policy).

### Generic ephemeral volumes

{{< feature-state for_k8s_version="v1.19" state="alpha" >}}

This feature requires the `GenericEphemeralVolume` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) to be
enabled. Because this is an alpha feature, it is disabled by default.

Generic ephemeral volumes are similar to `emptyDir` volumes, just more
flexible:
- Storage can be local or network-attached.
- Volumes can have a fixed size that Pods are not able to exceed.
- Volumes may have some initial data, depending on the driver and
  parameters.
- Typical operations on volumes are supported assuming that the driver
  supports them, including
  ([snapshotting](/docs/concepts/storage/volume-snapshots/),
  [cloning](/docs/concepts/storage/volume-pvc-datasource/),
  [resizing](/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims),
  and [storage capacity tracking](/docs/concepts/storage/storage-capacity/).

Example:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: my-app
spec:
  containers:
    - name: my-frontend
      image: busybox
      volumeMounts:
      - mountPath: "/scratch"
        name: scratch-volume
      command: [ "sleep", "1000000" ]
  volumes:
    - name: scratch-volume
      ephemeral:
        volumeClaimTemplate:
          metadata:
            labels:
              type: my-frontend-volume
          spec:
            accessModes: [ "ReadWriteOnce" ]
            storageClassName: "scratch-storage-class"
            resources:
              requests:
                storage: 1Gi
```

### Lifecycle and PersistentVolumeClaim

The key design idea is that the
[parameters for a volume claim](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ephemeralvolumesource-v1alpha1-core)
are allowed inside a volume source of the Pod. Labels, annotations and
the whole set of fields for a PersistentVolumeClaim are supported. When such a Pod gets
created, the ephemeral volume controller then creates an actual PersistentVolumeClaim
object in the same namespace as the Pod and ensures that the PersistentVolumeClaim
gets deleted when the Pod gets deleted.

That triggers volume binding and/or provisioning, either immediately if
the {{< glossary_tooltip text="StorageClass" term_id="storage-class" >}} uses immediate volume binding or when the Pod is
tentatively scheduled onto a node (`WaitForFirstConsumer` volume
binding mode). The latter is recommended for generic ephemeral volumes
because then the scheduler is free to choose a suitable node for
the Pod. With immediate binding, the scheduler is forced to select a node that has
access to the volume once it is available.

In terms of [resource ownership](/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents),
a Pod that has generic ephemeral storage is the owner of the PersistentVolumeClaim(s)
that provide that ephemeral storage. When the Pod is deleted,
the Kubernetes garbage collector deletes the PVC, which then usually
triggers deletion of the volume because the default reclaim policy of
storage classes is to delete volumes. You can create quasi-ephemeral local storage
using a StorageClass with a reclaim policy of `retain`: the storage outlives the Pod,
and in this case you need to ensure that volume clean up happens separately.

While these PVCs exist, they can be used like any other PVC. In
particular, they can be referenced as data source in volume cloning or
snapshotting. The PVC object also holds the current status of the
volume.

### PersistentVolumeClaim naming

Naming of the automatically created PVCs is deterministic: the name is
a combination of Pod name and volume name, with a hyphen (`-`) in the
middle. In the example above, the PVC name will be
`my-app-scratch-volume`.  This deterministic naming makes it easier to
interact with the PVC because one does not have to search for it once
the Pod name and volume name are known.

The deterministic naming also introduces a potential conflict between different
Pods (a Pod "pod-a" with volume "scratch" and another Pod with name
"pod" and volume "a-scratch" both end up with the same PVC name
"pod-a-scratch") and between Pods and manually created PVCs.

Such conflicts are detected: a PVC is only used for an ephemeral
volume if it was created for the Pod. This check is based on the
ownership relationship. An existing PVC is not overwritten or
modified. But this does not resolve the conflict because without the
right PVC, the Pod cannot start.

{{< caution >}}
Take care when naming Pods and volumes inside the
same namespace, so that these conflicts can't occur.
{{< /caution >}}

### Security

Enabling the GenericEphemeralVolume feature allows users to create
PVCs indirectly if they can create Pods, even if they do not have
permission to create PVCs directly. Cluster administrators must be
aware of this. If this does not fit their security model, they have
two choices:
- Explicitly disable the feature through the feature gate, to avoid
  being surprised when some future Kubernetes version enables it
  by default.
- Use a [Pod Security
  Policy](/docs/concepts/policy/pod-security-policy/) where the
  `volumes` list does not contain the `ephemeral` volume type.

The normal namespace quota for PVCs in a namespace still applies, so
even if users are allowed to use this new mechanism, they cannot use
it to circumvent other policies.

## {{% heading "whatsnext" %}}

### Ephemeral volumes managed by kubelet

See [local ephemeral storage](/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage).

### CSI ephemeral volumes

- For more information on the design, see the [Ephemeral Inline CSI
  volumes KEP](https://github.com/kubernetes/enhancements/blob/ad6021b3d61a49040a3f835e12c8bb5424db2bbb/keps/sig-storage/20190122-csi-inline-volumes.md).
- For more information on further development of this feature, see the [enhancement tracking issue #596](https://github.com/kubernetes/enhancements/issues/596).

### Generic ephemeral volumes

- For more information on the design, see the
[Generic ephemeral inline volumes KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-storage/1698-generic-ephemeral-volumes/README.md).
- For more information on further development of this feature, see the [enhancement tracking issue #1698](https://github.com/kubernetes/enhancements/issues/1698).
