---
title: Volumes
date: 2020-10-12
draft: true
weight: 020500
content_type: concept
weight: 10
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- thockin
- msau42
title: Volumes
content_type: concept
weight: 10
--- -->

<!-- overview -->
<!--
On-disk files in a Container are ephemeral, which presents some problems for
non-trivial applications when running in Containers.  First, when a Container
crashes, kubelet will restart it, but the files will be lost - the
Container starts with a clean state.  Second, when running Containers together
in a `Pod` it is often necessary to share files between those Containers.  The
Kubernetes `Volume` abstraction solves both of these problems.

Familiarity with [Pods](/docs/concepts/workloads/pods/pod/) is suggested.
 -->
容器中写到硬盘的文件是临时的，这会导致有些需要写入硬盘文件的应用出现一些问题。第一个问题是当一个容器
崩溃后，kubelet 会将其重启，但其所写的文件会全部丢失 - 容器会以全新的状态启动。 第二个问题是
在一个 `Pod` 中的不同容器之间需要共享文件。 k8s 的 `Volume` 抽象概念就是解决这些问题的。

建议先看看 [Pods](/k8sDocs/docs/concepts/workloads/pods/pod/)
<!-- body -->
<!--
## Background

Docker also has a concept of
[volumes](https://docs.docker.com/storage/), though it is
somewhat looser and less managed.  In Docker, a volume is simply a directory on
disk or in another Container.  Lifetimes are not managed and until very
recently there were only local-disk-backed volumes.  Docker now provides volume
drivers, but the functionality is very limited for now (e.g. as of Docker 1.7
only one volume driver is allowed per Container and there is no way to pass
parameters to volumes).

A Kubernetes volume, on the other hand, has an explicit lifetime - the same as
the Pod that encloses it.  Consequently, a volume outlives any Containers that run
within the Pod, and data is preserved across Container restarts. Of course, when a
Pod ceases to exist, the volume will cease to exist, too.  Perhaps more
importantly than this, Kubernetes supports many types of volumes, and a Pod can
use any number of them simultaneously.

At its core, a volume is just a directory, possibly with some data in it, which
is accessible to the Containers in a Pod.  How that directory comes to be, the
medium that backs it, and the contents of it are determined by the particular
volume type used.

To use a volume, a Pod specifies what volumes to provide for the Pod (the
`.spec.volumes`
field) and where to mount those into Containers (the
`.spec.containers[*].volumeMounts`
field).

A process in a container sees a filesystem view composed from their Docker
image and volumes.  The [Docker
image](https://docs.docker.com/userguide/dockerimages/) is at the root of the
filesystem hierarchy, and any volumes are mounted at the specified paths within
the image.  Volumes can not mount onto other volumes or have hard links to
other volumes.  Each Container in the Pod must independently specify where to
mount each volume.
 -->

## 背景

Docker 同样有 [volumes](https://docs.docker.com/storage/) 概念，但是 Docker 的数据卷(volume)
不那么好管理。在 Docker 中，一个数据卷就是简单地在磁盘上或另一个容器中的一个目录。存在期不受管理，
并且直到最近(文档最早提交为 2018年5月6日)都只支持本地磁盘的数据卷。 现在 Docker 提供了数据卷
驱动， 但其功能也还是相当有限(例如，Docker v1.7 中每个容器只支持一个数据卷驱动，并且没办法向
数据卷传递参数)。

k8s 的数据卷，相对来说就更加强大，拥有明确的生命期 - 与其绑定的 Pod 相同。 因此，一个数据卷可能
比 Pod 中所有的容器的命都长， 并且在容器重启之后数据依然存在。 当然，当 Pod 不存在进，与其绑定的
数据卷也就不存在了。 可能比这些更重要的是 k8s 支持许多类型的数据卷，并且一个 Pod 可以同时使用
任意类型任意数量的数据卷。

数据卷的核心也只是一个目录，可能其中还有数据，它可以被 Pod 中的容器访问。 这个目录是怎么来的，
它所用的介质是什么，它其中的内存是什么，这些都是由数据卷所使用的类型所决定的。

要使用一个数据卷，需要在 Pod 配置文件中 `.spec.volumes` 字段上配置可用数据卷，并在容器的
`.spec.containers[*].volumeMounts` 字段指定数据卷和在容器内的挂载点

在一个容器内的进程看到的文件系统是由它们的 Docker 镜像和数据卷共同组成的。 [Docker
镜像](https://docs.docker.com/userguide/dockerimages/)位于文件系统层级的底层， 然后各个
数据卷挂载到镜像内相应的目录上。 数据卷不能再挂载到其它的数据卷上，也不创建硬连接到其它的数据卷。
Pod 中的每个容器都需要独立的指定每个数据卷的挂载点。

<!--
## Types of Volumes

Kubernetes supports several types of Volumes:

   * [awsElasticBlockStore](#awselasticblockstore)
   * [azureDisk](#azuredisk)
   * [azureFile](#azurefile)
   * [cephfs](#cephfs)
   * [cinder](#cinder)
   * [configMap](#configmap)
   * [csi](#csi)
   * [downwardAPI](#downwardapi)
   * [emptyDir](#emptydir)
   * [fc (fibre channel)](#fc)
   * [flexVolume](#flexVolume)
   * [flocker](#flocker)
   * [gcePersistentDisk](#gcepersistentdisk)
   * [gitRepo (deprecated)](#gitrepo)
   * [glusterfs](#glusterfs)
   * [hostPath](#hostpath)
   * [iscsi](#iscsi)
   * [local](#local)
   * [nfs](#nfs)
   * [persistentVolumeClaim](#persistentvolumeclaim)
   * [projected](#projected)
   * [portworxVolume](#portworxvolume)
   * [quobyte](#quobyte)
   * [rbd](#rbd)
   * [scaleIO](#scaleio)
   * [secret](#secret)
   * [storageos](#storageos)
   * [vsphereVolume](#vspherevolume)

We welcome additional contributions.

 -->
## 数据卷(Volume)的类型 {#types-of-volumes}

以下为 k8s 支持的数据卷(Volume)类型:

   * [awsElasticBlockStore](#awselasticblockstore)
   * [azureDisk](#azuredisk)
   * [azureFile](#azurefile)
   * [cephfs](#cephfs)
   * [cinder](#cinder)
   * [configMap](#configmap)
   * [csi](#csi)
   * [downwardAPI](#downwardapi)
   * [emptyDir](#emptydir)
   * [fc (fibre channel)](#fc)
   * [flexVolume](#flexVolume)
   * [flocker](#flocker)
   * [gcePersistentDisk](#gcepersistentdisk)
   * [gitRepo (deprecated)](#gitrepo)
   * [glusterfs](#glusterfs)
   * [hostPath](#hostpath)
   * [iscsi](#iscsi)
   * [local](#local)
   * [nfs](#nfs)
   * [persistentVolumeClaim](#persistentvolumeclaim)
   * [projected](#projected)
   * [portworxVolume](#portworxvolume)
   * [quobyte](#quobyte)
   * [rbd](#rbd)
   * [scaleIO](#scaleio)
   * [secret](#secret)
   * [storageos](#storageos)
   * [vsphereVolume](#vspherevolume)

欢迎贡献更多类型.
<!--
### awsElasticBlockStore {#awselasticblockstore}

An `awsElasticBlockStore` volume mounts an Amazon Web Services (AWS) [EBS
Volume](https://aws.amazon.com/ebs/) into your Pod.  Unlike
`emptyDir`, which is erased when a Pod is removed, the contents of an EBS
volume are preserved and the volume is merely unmounted.  This means that an
EBS volume can be pre-populated with data, and that data can be "handed off"
between Pods.

{{< caution >}}
You must create an EBS volume using `aws ec2 create-volume` or the AWS API before you can use it.
{{< /caution >}}

There are some restrictions when using an `awsElasticBlockStore` volume:

* the nodes on which Pods are running must be AWS EC2 instances
* those instances need to be in the same region and availability-zone as the EBS volume
* EBS only supports a single EC2 instance mounting a volume
 -->

### awsElasticBlockStore {#awselasticblockstore}

一个 `awsElasticBlockStore` 数据卷会挂载一个 AWS 的 [EBS 卷](https://aws.amazon.com/ebs/)
到 Pod 中。 与 `emptyDir` 在 Pod 删除时清除数据不同， 在 EBS 卷中的内存会在卸载后依然会保存
其中的内容。也就是说一个 EBS 卷 可以预先存在数据，其中的数据也可以在不同的 Pod 之间传递。

{{< caution >}}
必须要先使用 `aws ec2 create-volume` 或 AWS API 创建一个 EBS 卷 才能在 `awsElasticBlockStore` 中使用。
{{< /caution >}}

使用 `awsElasticBlockStore` 卷有如下限制:

- Pod 运行的节点必须是 AWS EC2 实例
- 节点实例必须与 EBS 卷 在同一个地址或可用域
- EBS 只支持一个 EC2 实例挂载一个卷
<!--
#### Creating an EBS volume

Before you can use an EBS volume with a Pod, you need to create it.

```shell
aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2
```

Make sure the zone matches the zone you brought up your cluster in.  (And also check that the size and EBS volume
type are suitable for your use!)
 -->

#### 创建一个 EBS 卷

在 Pod 中使用一个 EBS 卷 之前需要先创建。

```shell
aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2
```

需要确保卷所在的可用区域与集群所在的可用区域相同。(同时还要检查卷的大小和类型是合用的)
<!--
#### AWS EBS Example configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```
 -->

#### 一个使用 AWS EBS 的配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```
<!--
#### CSI Migration

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

The CSI Migration feature for awsElasticBlockStore, when enabled, shims all plugin operations
from the existing in-tree plugin to the `ebs.csi.aws.com` Container
Storage Interface (CSI) Driver. In order to use this feature, the [AWS EBS CSI
Driver](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationAWS`
Beta features must be enabled.
 -->

#### CSI 迁移

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

`awsElasticBlockStore` 的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到
`ebs.csi.aws.com` 容器存储接口(CSI)驱动。 为了使用该特性， 需要在集群中先安装
[AWS EBS CSI 驱动](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
同时启用 `CSIMigration` 和 `CSIMigrationAWS` 两个 `beta` 特性
<!--
#### CSI Migration Complete
{{< feature-state for_k8s_version="v1.17" state="alpha" >}}

To turn off the awsElasticBlockStore storage plugin from being loaded by controller manager and kubelet, you need to set this feature flag to true. This requires `ebs.csi.aws.com` Container Storage Interface (CSI) driver being installed on all worker nodes.
 -->

#### CSI 迁移完成
{{< feature-state for_k8s_version="v1.17" state="alpha" >}}

为了关闭由控制器管理器和 kubelet 加载的 `awsElasticBlockStore` 存储插件， 需要将这个标记设置
为 `true`. 这需要在所有的工作节点安装 `ebs.csi.aws.com` 容器存储接口(CSI)驱动
<!--
### azureDisk {#azuredisk}

A `azureDisk` is used to mount a Microsoft Azure [Data Disk](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/) into a Pod.

More details can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_disk/README.md).
 -->

### azureDisk {#azuredisk}

`azureDisk` 是用来将微软 Azure 的
[Data Disk](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/)
数据盘挂载到 Pod 中的。
更多信息见 [这里](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_disk/README.md).
<!--
#### CSI Migration

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

The CSI Migration feature for azureDisk, when enabled, shims all plugin operations
from the existing in-tree plugin to the `disk.csi.azure.com` Container
Storage Interface (CSI) Driver. In order to use this feature, the [Azure Disk CSI
Driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationAzureDisk`
features must be enabled.
 -->

#### CSI 迁移

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

`azureDisk` 的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到
`disk.csi.azure.com`  容器存储接口(CSI)驱动。 为了使用该特性， 需要在集群中先安装
[Azure Disk CSI Driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)
同时启用 `CSIMigration` 和 `CSIMigrationAzureDisk` 两个特性
<!--
### azureFile {#azurefile}

A `azureFile` is used to mount a Microsoft Azure File Volume (SMB 2.1 and 3.0)
into a Pod.

More details can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_file/README.md).
 -->

### azureFile {#azurefile}

A `azureFile` is used to mount a Microsoft Azure File Volume (SMB 2.1 and 3.0)
into a Pod.
`azureFile` 用于将微软 Azure 文件卷(SMB 2.1 and 3.0) 挂载到 Pod 中。
更多信息见 [这里](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/azure_file/README.md).
<!--
#### CSI Migration

{{< feature-state for_k8s_version="v1.15" state="alpha" >}}

The CSI Migration feature for azureFile, when enabled, shims all plugin operations
from the existing in-tree plugin to the `file.csi.azure.com` Container
Storage Interface (CSI) Driver. In order to use this feature, the [Azure File CSI
Driver](https://github.com/kubernetes-sigs/azurefile-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationAzureFile`
Alpha features must be enabled.
 -->

#### CSI 迁移

{{< feature-state for_k8s_version="v1.15" state="alpha" >}}

`azureFile` 的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到
`file.csi.azure.com`  容器存储接口(CSI)驱动。 为了使用该特性， 需要在集群中先安装
[Azure File CSI Driver](https://github.com/kubernetes-sigs/azurefile-csi-driver)
同时启用 `CSIMigration` 和 `CSIMigrationAzureFile` 两个 `Alpha` 特性

### cephfs {#cephfs}

`cephfs` 卷可以将一个已经存在的 CephFS 卷挂载到一个 Pod 中。
与 `emptyDir` 在 Pod 删除时清除数据不同， 在 `cephfs` 卷中的内存会在卸载后依然会保存
其中的内容。也就是说一个 `cephfs` 卷可以预先存在数据，其中的数据也可以在不同的 Pod 之间传递。
CephFS 还可以被同时多个挂载拥有写权限

{{< caution >}}
在使用之前需要有自己的 Ceph 服务在运行，并且相应的分离已经配置好
{{< /caution >}}

更多信息见 [CephFS 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/cephfs/)
<!--
### cinder {#cinder}

{{< note >}}
Prerequisite: Kubernetes with OpenStack Cloud Provider configured.
{{< /note >}}

`cinder` is used to mount OpenStack Cinder Volume into your Pod.
 -->

### cinder {#cinder}

{{< note >}}
前置条件: 配置好 OpenStack 云提供商的 k8s 集群。
{{< /note >}}

`cinder` 用于将 OpenStack Cinder 卷挂载到 Pod.
<!--
#### Cinder Volume Example configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-cinder
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-cinder-container
    volumeMounts:
    - mountPath: /test-cinder
      name: test-volume
  volumes:
  - name: test-volume
    # This OpenStack volume must already exist.
    cinder:
      volumeID: <volume-id>
      fsType: ext4
```
-->

#### Cinder Volume 配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-cinder
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-cinder-container
    volumeMounts:
    - mountPath: /test-cinder
      name: test-volume
  volumes:
  - name: test-volume
    # 这个 OpenStack 卷必须已经存在.
    cinder:
      volumeID: <volume-id>
      fsType: ext4
```

#### CSI 迁移

{{< feature-state for_k8s_version="v1.18" state="beta" >}}

`Cinder` 的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到 `cinder.csi.openstack.org`  容器存储接口(CSI)驱动。 为了使用该特性， 需要在集群中先安装
[Openstack Cinder CSI
Driver](https://github.com/kubernetes/cloud-provider-openstack/blob/master/docs/using-cinder-csi-plugin.md)
同时启用 `CSIMigration` 和 `CSIMigrationOpenStack` 两个 `beta` 特性
<!--
### configMap {#configmap}

The [`configMap`](/docs/tasks/configure-pod-container/configure-pod-configmap/) resource
provides a way to inject configuration data into Pods.
The data stored in a `ConfigMap` object can be referenced in a volume of type
`configMap` and then consumed by containerized applications running in a Pod.

When referencing a `configMap` object, you can simply provide its name in the
volume to reference it. You can also customize the path to use for a specific
entry in the ConfigMap.
For example, to mount the `log-config` ConfigMap onto a Pod called `configmap-pod`,
you might use the YAML below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

The `log-config` ConfigMap is mounted as a volume, and all contents stored in
its `log_level` entry are mounted into the Pod at path "`/etc/config/log_level`".
Note that this path is derived from the volume's `mountPath` and the `path`
keyed with `log_level`.

{{< caution >}}
You must create a [ConfigMap](/docs/tasks/configure-pod-container/configure-pod-configmap/) before you can use it.
{{< /caution >}}

{{< note >}}
A Container using a ConfigMap as a [subPath](#using-subpath) volume mount will not
receive ConfigMap updates.
{{< /note >}}

{{< note >}}
Text data is exposed as files using the UTF-8 character encoding. To use some other character encoding, use binaryData.
{{< /note >}}
 -->

### configMap {#configmap}

[`configMap`](/docs/tasks/configure-pod-container/configure-pod-configmap/) 资源提供了一种将配置数据注入 Pod 的方式。

存在于 `ConfigMap` 对象中的数据可以通过 `configMap` 类型的数据卷被 Pod 引用，然后被存在于 Pod 中的容器化应用所使用。

在引用 `configMap` 对象时，只需要提供这个对象的名称在引用它的卷上面。
也可以通过自定义路径的方式使用 ConfigMap 中指定的条目。
例如， 将一个叫 `log-config` ConfigMap 挂载到一个 叫 `configmap-pod` 的 Pod上，配置 YAML 如下:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

`log-config` ConfigMap 以卷的方式挂载， 其中所有存在于条目 `log_level` 的内容会被挂载到 Pod 的 `/etc/config/log_level` 上
其中这个路径继承了卷上面的 `mountPath` 和 键名为 `log_level` 的
`path` 的值 -  `log_level`

{{< caution >}}
在使用之前必须要先创建对应的
[ConfigMap](/k8sDocs/tasks/configure-pod-container/configure-pod-configmap/)
{{< /caution >}}

{{< note >}}
一个以 [subPath](#using-subpath) 卷方式使用 ConfigMap 的容器是
不能接收到 ConfigMap 的更新的
{{< /note >}}

{{< note >}}
文本内存数据以 UTF-8编码的文件方式提供。 要使用其它的编码方式请使用
二进制数据(`binaryData`)
{{< /note >}}

<!--
### downwardAPI {#downwardapi}

A `downwardAPI` volume is used to make downward API data available to applications.
It mounts a directory and writes the requested data in plain text files.

{{< note >}}
A Container using Downward API as a [subPath](#using-subpath) volume mount will not
receive Downward API updates.
{{< /note >}}

See the [`downwardAPI` volume example](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)  for more details.

 -->
### downwardAPI {#downwardapi}

`downwardAPI` 卷用来让 downward API 的数据在应用中可用。
它通过挂载一个目录然后将请求的数据以纯文本文件的形式写在这个目录中。
{{< note >}}
 一个以 [subPath](#using-subpath) 卷方式使用 Downward API 的容器是 不能接收到 Downward API 的更新的
{{< /note >}}

更多信息见 [`downwardAPI` 卷示例](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

<!--

### emptyDir {#emptydir}

An `emptyDir` volume is first created when a Pod is assigned to a Node, and
exists as long as that Pod is running on that node.  As the name says, it is
initially empty.  Containers in the Pod can all read and write the same
files in the `emptyDir` volume, though that volume can be mounted at the same
or different paths in each Container.  When a Pod is removed from a node for
any reason, the data in the `emptyDir` is deleted forever.

{{< note >}}
A Container crashing does *NOT* remove a Pod from a node, so the data in an `emptyDir` volume is safe across Container crashes.
{{< /note >}}

Some uses for an `emptyDir` are:

* scratch space, such as for a disk-based merge sort
* checkpointing a long computation for recovery from crashes
* holding files that a content-manager Container fetches while a webserver
  Container serves the data

By default, `emptyDir` volumes are stored on whatever medium is backing the
node - that might be disk or SSD or network storage, depending on your
environment.  However, you can set the `emptyDir.medium` field to `"Memory"`
to tell Kubernetes to mount a tmpfs (RAM-backed filesystem) for you instead.
While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on
node reboot and any files you write will count against your Container's
memory limit.
 -->
### emptyDir {#emptydir}

`emptyDir` 卷在 Pod 分配到节点时创建，只要这个 Pod 还在这个节点上运行，
那么这个郑就会存在。 就像名称所示，它就是个空目录，所以初始化时是空的。
Pod 中 所有的容器都可以读写这个 `emptyDir` 卷中的文件， 这个卷可以挂载到每个
容器的同一个路径或不同的路径。当 Pod 因为任何原因从节点移除时， 存在于  `emptyDir`
中的数据会被永久删除。
{{< note >}}
容器崩溃并 *不会* 导致 Pod 从节点删除，所以容器崩溃并不会导致 `emptyDir` 中的数据丢失。
{{< /note >}}

Some uses for an `emptyDir` are:
一些用到 `emptyDir` 的地方:

- 临时空间，例如基于磁盘的合并排序
- 作为一个恢复崩溃的长时计划的检查点
  Container serves the data
- 存放由 内容管理容器抓取然后用于 web 服务器使用的数据

默认情况下， `emptyDir` 卷使用所在节点所使用的介质存储数据，基于环境可能是机械硬盘
SSD 或 网络存储。 但是，用户可以通过设置 `emptyDir.medium` 字段值为 `"Memory"`
让 k8s 将其挂载为 tmpfs (基于内存的文件系统)。
因为 tmpfs 的速度很快，但要注意与磁盘不同， tmpfs 会在节点重启时被清除，
可用空间的大小受内存大小限制。

#### 示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### fc (fibre channel) {#fc}

`fc` 卷可以将光纤信道(fibre channel)卷挂载到 Pod 中。 可以在卷的配置中使用
`targetWWNs` 参数指定一个或多个目标全局名称(World Wide Names).
如果指定了多个 WWNs， targetWWNs 使用的是那些来自 multi-path 连接的  WWNs
{{< caution >}}
必须要预先配置 FC SAN Zoning 分配并且标记这些目标 WWN 的 LUNs (卷)
这样 k8s 主机才能访问它们
{{< /caution >}}

更多信息见 [FC 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/fibre_channel)

### flocker {#flocker}

[Flocker](https://github.com/ClusterHQ/flocker) 是一个开源的集群
容器数据卷管理器。 它提供了管理和编排多种存储后端的数据卷

`flocker` 让 Flocker 数据集可以被挂载到 Pod 中。 如果数据集还没有存在于 Flocker， 必须要使用 Flocker
CLI 或 Flocker API 创建。 如果数据集已经存在于 Flocker， 当 Pod 被调度到
节点时会被重新通过 Flocker 挂载。这就是说这些数据可以在需要的 Pod 传递。

{{< caution >}}
必须要在使用之前先安装运行 Flocker
{{< /caution >}}

更多信息见 [Flocker 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/flocker)

### gcePersistentDisk {#gcepersistentdisk}

`gcePersistentDisk` 卷可以将 Google 计算引擎(GCE)
[持久化磁盘(PD)](https://cloud.google.com/compute/docs/disks)
挂载到 Pod 中。 与 `emptyDir` 在 Pod 删除时同时删除不同， PD 中的内存会依旧保留 仅仅卸载卷。 这就是说 PD 可以预先添加数据，并且可以在不同 Pod 之间传递。

{{< caution >}}
必须要先使用 `gcloud` 或 GCE API 或 UI 创建 PD 才能使用。
{{< /caution >}}

使用 `gcePersistentDisk` 有如下限制:

- Pod 运行的节点必须是 GCE 虚拟机
- 这些虚拟机必须与 PD 在同一个 GCE 项目和区域

PD 有一个特性是如果要同时挂载到多个 Pod 只能以只读方式挂载。
也就是可以预先将数据存入 PD 中，然后就可以在需要的任意个多 Pod 中使用。
不幸的是只能以读写方式挂载一次，不能以写方式多次挂载
在一个用 ReplicationController 管理的 Pod 中使用 PD 时，只有在 PD 为只读
或副本数为 0 或 1 时才能成功否则就会失败。
<!--
#### Creating a PD

Before you can use a GCE PD with a Pod, you need to create it.

```shell
gcloud compute disks create --size=500GB --zone=us-central1-a my-data-disk
```

#### Example Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    # This GCE PD must already exist.
    gcePersistentDisk:
      pdName: my-data-disk
      fsType: ext4
```
 -->

#### 创建 PD

在 Pod 中使用 GCE PD 前需要先创建
```shell
gcloud compute disks create --size=500GB --zone=us-central1-a my-data-disk
```

#### 示例 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    # 这个 GCE PD 必须要先存在
    gcePersistentDisk:
      pdName: my-data-disk
      fsType: ext4
```
<!--
#### Regional Persistent Disks
The [Regional Persistent Disks](https://cloud.google.com/compute/docs/disks/#repds) feature allows the creation of Persistent Disks that are available in two zones within the same region. In order to use this feature, the volume must be provisioned as a PersistentVolume; referencing the volume directly from a pod is not supported.
 -->

#### 地区性持久化磁盘

[地区性持久化磁盘](https://cloud.google.com/compute/docs/disks/#repds) 特性允许在同一个地区的两个区域创建 地区性持久化磁盘。
要使用这个特性， 这个卷必须由 PersistentVolume 管理；不支持在 Pod 直接使用
<!--
#### Manually provisioning a Regional PD PersistentVolume
Dynamic provisioning is possible using a [StorageClass for GCE PD](/docs/concepts/storage/storage-classes/#gce).
Before creating a PersistentVolume, you must create the PD:
```shell
gcloud compute disks create --size=500GB my-data-disk
    --region us-central1
    --replica-zones us-central1-a,us-central1-b
```
Example PersistentVolume spec:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
spec:
  capacity:
    storage: 400Gi
  accessModes:
  - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: failure-domain.beta.kubernetes.io/zone
          operator: In
          values:
          - us-central1-a
          - us-central1-b
```
 -->

#### 手动管理一个地区性 PD PersistentVolume
动态管理使用 [StorageClass for GCE PD](/docs/concepts/storage/storage-classes/#gce).
在创建 PersistentVolume 之前，需要先创建 PD:
```shell
gcloud compute disks create --size=500GB my-data-disk
    --region us-central1
    --replica-zones us-central1-a,us-central1-b
```
示例 PersistentVolume spec:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
spec:
  capacity:
    storage: 400Gi
  accessModes:
  - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: failure-domain.beta.kubernetes.io/zone
          operator: In
          values:
          - us-central1-a
          - us-central1-b
```
<!--
#### CSI Migration

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

The CSI Migration feature for GCE PD, when enabled, shims all plugin operations
from the existing in-tree plugin to the `pd.csi.storage.gke.io` Container
Storage Interface (CSI) Driver. In order to use this feature, the [GCE PD CSI
Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationGCE`
Beta features must be enabled.
 -->

#### CSI 迁移

{{< feature-state for_k8s_version="v1.17" state="beta" >}}

GCE PD 的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到 `pd.csi.storage.gke.io`  容器存储接口(CSI)驱动。 为了使用该特性， 需要在集群中先安装
[GCE PD CSI
Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
同时启用 `CSIMigration` 和 `CSIMigrationGCE` 两个 `beta` 特性
<!--
### gitRepo (deprecated) {#gitrepo}

{{< warning >}}
The gitRepo volume type is deprecated. To provision a container with a git repo, mount an [EmptyDir](#emptydir) into an InitContainer that clones the repo using git, then mount the [EmptyDir](#emptydir) into the Pod's container.
{{< /warning >}}

A `gitRepo` volume is an example of what can be done as a volume plugin.  It
mounts an empty directory and clones a git repository into it for your Pod to
use.  In the future, such volumes may be moved to an even more decoupled model,
rather than extending the Kubernetes API for every such use case.

Here is an example of gitRepo volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: server
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /mypath
      name: git-volume
  volumes:
  - name: git-volume
    gitRepo:
      repository: "git@somewhere:me/my-git-repository.git"
      revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
```
 -->

### gitRepo (废弃) {#gitrepo}

{{< warning >}}
gitRepo 卷类型已经被废弃。 要管理一个包含 git 创建的容器，可以在初始化容器
(InitContainer) 中挂载一个 [EmptyDir](#emptydir) 使用 git 克隆这个仓库，
然后在需要使用的容器的 Pod 上挂载这个 [EmptyDir](#emptydir)
{{< /warning >}}

`gitRepo` 卷是一个能够使用卷插件的示例。 它挂载一个空目录然后在其中克隆一个
git 创建到其中，供 Pod 使用。 在未来，这些卷可能被改为更加松耦合的模式，
而不是每次在这些情况下都通过扩展 k8s  API 来实现。
以下为 gitRepo 卷示例:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: server
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /mypath
      name: git-volume
  volumes:
  - name: git-volume
    gitRepo:
      repository: "git@somewhere:me/my-git-repository.git"
      revision: "22f1d8406d464b0c0874075539c1f2e96c253775"
```
<!--
### glusterfs {#glusterfs}

A `glusterfs` volume allows a [Glusterfs](https://www.gluster.org) (an open
source networked filesystem) volume to be mounted into your Pod.  Unlike
`emptyDir`, which is erased when a Pod is removed, the contents of a
`glusterfs` volume are preserved and the volume is merely unmounted.  This
means that a glusterfs volume can be pre-populated with data, and that data can
be "handed off" between Pods.  GlusterFS can be mounted by multiple writers
simultaneously.

{{< caution >}}
You must have your own GlusterFS installation running before you can use it.
{{< /caution >}}

See the [GlusterFS example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/glusterfs) for more details.
 -->

### glusterfs {#glusterfs}

使用 `glusterfs` 卷可以将 [Glusterfs](https://www.gluster.org)
(一个开源的网络文件系统)卷挂载到 Pod 中。
与 `emptyDir` 在 Pod 删除时清除数据不同， 在 `glusterfs` 卷中的内存会在卸载后依然会保存
其中的内容。也就是说一个 `glusterfs` 卷可以预先存在数据，其中的数据也可以在不同的 Pod 之间传递。
GlusterFS 还可以被同时多个挂载拥有写权限
{{< caution >}}
在使用之前必须要先安装运行自己的 GlusterFS
{{< /caution >}}

更多个信息见 [GlusterFS 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/glusterfs)
<!--
### hostPath {#hostpath}

A `hostPath` volume mounts a file or directory from the host node's filesystem
into your Pod. This is not something that most Pods will need, but it offers a
powerful escape hatch for some applications.

For example, some uses for a `hostPath` are:

* running a Container that needs access to Docker internals; use a `hostPath`
  of `/var/lib/docker`
* running cAdvisor in a Container; use a `hostPath` of `/sys`
* allowing a Pod to specify whether a given `hostPath` should exist prior to the
  Pod running, whether it should be created, and what it should exist as

In addition to the required `path` property, user can optionally specify a `type` for a `hostPath` volume.

The supported values for field `type` are:


| Value | Behavior |
|:------|:---------|
| | Empty string (default) is for backward compatibility, which means that no checks will be performed before mounting the hostPath volume. |
| `DirectoryOrCreate` | If nothing exists at the given path, an empty directory will be created there as needed with permission set to 0755, having the same group and ownership with Kubelet. |
| `Directory` | A directory must exist at the given path |
| `FileOrCreate` | If nothing exists at the given path, an empty file will be created there as needed with permission set to 0644, having the same group and ownership with Kubelet. |
| `File` | A file must exist at the given path |
| `Socket` | A UNIX socket must exist at the given path |
| `CharDevice` | A character device must exist at the given path |
| `BlockDevice` | A block device must exist at the given path |

Watch out when using this type of volume, because:

* Pods with identical configuration (such as created from a podTemplate) may
  behave differently on different nodes due to different files on the nodes
* when Kubernetes adds resource-aware scheduling, as is planned, it will not be
  able to account for resources used by a `hostPath`
* the files or directories created on the underlying hosts are only writable by root. You
  either need to run your process as root in a
  [privileged Container](/docs/tasks/configure-pod-container/security-context/) or modify the file
  permissions on the host to be able to write to a `hostPath` volume
 -->

### hostPath {#hostpath}

`hostPath` 卷可以将节点上的一个文件或目录挂载到 Pod 中. 这种方式不是大多数
Pod 所需要要的， 但也能为有些应用提供强力支持。

For example, some uses for a `hostPath` are:
例如以下为可以用到 `hostPath` 的地方:
- 运行一个需要访问 Docker 内部的应用，使用 `hostPath` 挂载 `/var/lib/docker`

- 在容器中运行 `cAdvisor`； 使用 `hostPath` 挂载 `/sys`
- 允许 Pod 指定某个 `hostPath` 在 Pod 运行之前需要存在，这个目录是否需要创建，以什么方式存在(在说啥？)
In addition to the required `path` property, user can optionally specify a `type` for a `hostPath` volume.
在需要的 `path` 属性外， 用户还可以为 `hostPath` 指定一个可选的 `type` 属性。
The supported values for field `type` are:
`type` 字段支持的值有:

| 值     | 行为 |
|:------|:---------|
| |空字符器(默认)为了向后兼容，含义是在执行挂载 `hostPath` 卷之前不执行任何检查|
| `DirectoryOrCreate` |如果指定路径不存在，则创建一个空目录，权限为 0755，组和所属关系与 kubelet 相同 |
| `Directory` | 路径必须是一个已经存在的目录 |
| `FileOrCreate` | 如果指定路径不存在，则创建一个空文件，权限为 0644，组和所属关系与 kubelet 相同|
| `File` | 路径必须是一个已经存在的文件 |
| `Socket` | 路径必须是一个已经存在的 UNIX 套接字 |
| `CharDevice` | 路径必须是一个已经存在的 字符设备(character device) |
| `BlockDevice` | 路径必须是一个已经存在的 块设备(block device)  |

Watch out when using this type of volume, because:
在使用带类型的卷时需要小心，因为:
- 拥有相同配置(如由同一个 podTemplate 创建的) Pod 可以会因为不同节点上拥有的文件不同为表现不同行为
- 根据设计，当 k8s 添加资源感知(resource-aware) 调度时，不会计算 `hostPath` 使用的资源。
- 在底层主机上创建的文件或目录只有 root 拥有写权限。 所以必须要在一个
  [提权容器](/k8sDocs/tasks/configure-pod-container/security-context/)
  以 root 权限运行进程或在主机上修改文件权限以便可以让 `hostPath` 卷为可读
<!--
#### Example Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

{{< caution >}}
It should be noted that the `FileOrCreate` mode does not create the parent directory of the file. If the parent directory of the mounted file does not exist, the pod fails to start. To ensure that this mode works, you can try to mount directories and files separately, as shown below.
{{< /caution >}}
 -->

#### 示例 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # 主机上的目录
      path: /data
      # 该字段可选
      type: Directory
```

{{< caution >}}
要注意 `FileOrCreate` 模式时，如果文件的父目录不存在是不会自动创建的。 如果挂载文件的父目录不
存在， Pod 就会启动失败。 为了保证该模式正常工作，可以参考下面的方式，将目录和文件分开挂载。
{{< /caution >}}
<!--
#### Example Pod FileOrCreate

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: k8s.gcr.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # Ensure the file directory is created.
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```
 -->

#### FileOrCreate 示例 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: k8s.gcr.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # 确保文件所在的目录是存在的
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```
<!--
### iscsi {#iscsi}

An `iscsi` volume allows an existing iSCSI (SCSI over IP) volume to be mounted
into your Pod.  Unlike `emptyDir`, which is erased when a Pod is removed, the
contents of an `iscsi` volume are preserved and the volume is merely
unmounted.  This means that an iscsi volume can be pre-populated with data, and
that data can be "handed off" between Pods.

{{< caution >}}
You must have your own iSCSI server running with the volume created before you can use it.
{{< /caution >}}

A feature of iSCSI is that it can be mounted as read-only by multiple consumers
simultaneously.  This means that you can pre-populate a volume with your dataset
and then serve it in parallel from as many Pods as you need.  Unfortunately,
iSCSI volumes can only be mounted by a single consumer in read-write mode - no
simultaneous writers allowed.

See the [iSCSI example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/iscsi) for more details.
 -->
### iscsi {#iscsi}

`iscsi` 卷可以将已经存在的 iSCSI (SCSI over IP) 卷挂载到 Pod 中。
与 `emptyDir` 在 Pod 删除时清除数据不同， 在 `iscsi` 卷中的内存会在卸载后依然会保存
其中的内容。也就是说一个 `iscsi` 卷可以预先存在数据，其中的数据也可以在不同的 Pod 之间传递。

{{< caution >}}
在创建和使用卷之前必须为先有运行的 iSCSI 服务。
{{< /caution >}}

`iSCSI` 一个特性是可以以只读的方式同时挂载到多个消费者中。 这就代表着可以预先将数据集放入卷中
然后根据需要使用任意数量的 Pod 来提供访问服务。 不过 iSCSI 卷以读写方式挂载时只能有一个消费者，
不允许并行写。
更多信息见 [iSCSI 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/iscsi)
<!--
### local {#local}

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

A `local` volume represents a mounted local storage device such as a disk,
partition or directory.

Local volumes can only be used as a statically created PersistentVolume. Dynamic
provisioning is not supported yet.

Compared to `hostPath` volumes, local volumes can be used in a durable and
portable manner without manually scheduling Pods to nodes, as the system is aware
of the volume's node constraints by looking at the node affinity on the PersistentVolume.

However, local volumes are still subject to the availability of the underlying
node and are not suitable for all applications. If a node becomes unhealthy,
then the local volume will also become inaccessible, and a Pod using it will not
be able to run. Applications using local volumes must be able to tolerate this
reduced availability, as well as potential data loss, depending on the
durability characteristics of the underlying disk.

The following is an example of PersistentVolume spec using a `local` volume and
`nodeAffinity`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

PersistentVolume `nodeAffinity` is required when using local volumes. It enables
the Kubernetes scheduler to correctly schedule Pods using local volumes to the
correct node.

PersistentVolume `volumeMode` can be set to "Block" (instead of the default
value "Filesystem") to expose the local volume as a raw block device.

When using local volumes, it is recommended to create a StorageClass with
`volumeBindingMode` set to `WaitForFirstConsumer`. See the
[example](/docs/concepts/storage/storage-classes/#local). Delaying volume binding ensures
that the PersistentVolumeClaim binding decision will also be evaluated with any
other node constraints the Pod may have, such as node resource requirements, node
selectors, Pod affinity, and Pod anti-affinity.

An external static provisioner can be run separately for improved management of
the local volume lifecycle. Note that this provisioner does not support dynamic
provisioning yet. For an example on how to run an external local provisioner,
see the [local volume provisioner user
guide](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner).

{{< note >}}
The local PersistentVolume requires manual cleanup and deletion by the
user if the external static provisioner is not used to manage the volume
lifecycle.
{{< /note >}}
 -->

### local {#local}

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

`local` 卷代表一个挂载的本地存储，例如磁盘，分区或目录
`local` 卷只能以 静态创建 `PersistentVolume` 的方式使用。 目前还不支持动态管理

Compared to `hostPath` volumes, local volumes can be used in a durable and
portable manner without manually scheduling Pods to nodes, as the system is aware
of the volume's node constraints by looking at the node affinity on the PersistentVolume.
与 `hostPath` 相比， `local` 卷可以通过手动在节点上管理实现持久的和可移植的， 然后根据
PersistentVolume 的节点亲和性来打开对应节点上的卷。

但是， `local` 卷依然受底层节点的可用性影响，不是对所有应用都适用。 如果有一个节点应得不健康，
这个节点上的 `local` 也会变得不可用，使用这个卷的 Pod 也不能运行。 使用 `local` 卷的这些应用必要
要能忍受这样可用性降低的情况，同时还因为底层磁盘的持久性特性而产生的潜在的数据丢失的风险

以下示例中使用了 `local`卷和 `nodeAffinity` 的 `PersistentVolume`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

在使用 `local` 卷时 PersistentVolume 需要使用 `nodeAffinity`。 这让 k8s 可以正确地将
使用 `local` 卷的 Pod 调度到正确的节点上。

PersistentVolume 的 `volumeMode` 可以被设置为 "Block" (而不是默认值 "Filesystem")
将这个 `local` 卷作为块设备

当使用 `local` 卷时， 推荐在创建 StorageClass 时将 `volumeBindingMode` 设置为 `WaitForFirstConsumer`。
可以见
[示例](/k8sDocs/docs/concepts/storage/storage-classes/#local).
卷延迟绑定确定 PersistentVolumeClaim 在作绑定决策时检查 Pod 可能有的其它的节点约束，如果节点
的资源需求，节点的选择器，Pod 的亲和性和反亲和性。

可以在外部独立运行一个静态提供者来改善 `local` 卷生命周期管理。 要注意这个提供者目前还不支持动态管理。
运行外部提供者的示例见
[local 卷提供者用户指南](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner).
{{< note >}}
如果外部的静态提供都没有用来管理 `local` PersistentVolume 的生命周期， 需要手动清理和删除。
{{< /note >}}
<!--
### nfs {#nfs}

An `nfs` volume allows an existing NFS (Network File System) share to be
mounted into your Pod. Unlike `emptyDir`, which is erased when a Pod is
removed, the contents of an `nfs` volume are preserved and the volume is merely
unmounted.  This means that an NFS volume can be pre-populated with data, and
that data can be "handed off" between Pods.  NFS can be mounted by multiple
writers simultaneously.

{{< caution >}}
You must have your own NFS server running with the share exported before you can use it.
{{< /caution >}}

See the [NFS example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/nfs) for more details.
 -->
### nfs {#nfs}

`nfs` 卷可以将现有的 NFS (网络文件系统)挂载到 Pod 中。 与 `emptyDir` 在 Pod 删除时清除数据不同，
在 `iscsi` 卷中的内存会在卸载后依然会保存 其中的内容。也就是说一个 `iscsi` 卷可以预先存在数据，
其中的数据也可以在不同的 Pod 之间传递。 NFS 可以以读写方式同时挂载到多个 Pod 中。

{{< caution >}}
在使用之前必须要先搭建并运行好 NFS 服务器
{{< /caution >}}

更多详情见 [NFS 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/nfs)
<!--
### persistentVolumeClaim {#persistentvolumeclaim}

A `persistentVolumeClaim` volume is used to mount a
[PersistentVolume](/docs/concepts/storage/persistent-volumes/) into a Pod. PersistentVolumeClaims
are a way for users to "claim" durable storage (such as a GCE PersistentDisk or an
iSCSI volume) without knowing the details of the particular cloud environment.

See the [PersistentVolumes example](/docs/concepts/storage/persistent-volumes/) for more
details.
 -->
### persistentVolumeClaim {#persistentvolumeclaim}

`persistentVolumeClaim` 卷用于将
[PersistentVolume](/k8sDocs/docs/concepts/storage/persistent-volumes/) 挂载到 Pod 中。
`PersistentVolumeClaim` 是用户"声明"持久存储(如GCE PersistentDisk 或 iSCSI 卷)时不需要
知道具体云环境细节的一种方式
更多信息见 [PersistentVolumes 示例](/k8sDocs/docs/concepts/storage/persistent-volumes/)
<!--
### projected {#projected}

A `projected` volume maps several existing volume sources into the same directory.

Currently, the following types of volume sources can be projected:

- [`secret`](#secret)
- [`downwardAPI`](#downwardapi)
- [`configMap`](#configmap)
- `serviceAccountToken`

All sources are required to be in the same namespace as the Pod. For more details,
see the [all-in-one volume design document](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/node/all-in-one-volume.md).

The projection of service account tokens is a feature introduced in Kubernetes
1.11 and promoted to Beta in 1.12.
To enable this feature on 1.11, you need to explicitly set the `TokenRequestProjection`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/) to
True.
 -->

### projected {#projected}

`projected` 卷可以将几种现有的不同的卷作为源映射到同一个目录中

目前， 支持以下几种卷作为 `projected` 的源
- [`secret`](#secret)
- [`downwardAPI`](#downwardapi)
- [`configMap`](#configmap)
- `serviceAccountToken`

所有的源都需要要与 Pod 在同一个命令空间。 更多细节见
[all-in-one 卷设计文档](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/node/all-in-one-volume.md)

对服务账号凭据(Service Account Token)的投射支持是从 k8s 1.11 加入的， v1.12 提升为 beta 版本。
在 v1.11 中要启用该特性，需要显示地将
[功能阀](/k8sDocs/reference/command-line-tools-reference/feature-gates/)
`TokenRequestProjection` 设置为 `true`
<!--
#### Example Pod with a secret, a downward API, and a configmap.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: container-test
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
```
 -->

#### 示例： Pod 中包含 Secret, DownwardAPI, Configmap.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: container-test
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
```
<!--
#### Example Pod with multiple secrets with a non-default permission mode set.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - secret:
          name: mysecret2
          items:
            - key: password
              path: my-group/my-password
              mode: 511
```

Each projected volume source is listed in the spec under `sources`. The
parameters are nearly the same with two exceptions:

* For secrets, the `secretName` field has been changed to `name` to be consistent
  with ConfigMap naming.
* The `defaultMode` can only be specified at the projected level and not for each
  volume source. However, as illustrated above, you can explicitly set the `mode`
  for each individual projection.

When the `TokenRequestProjection` feature is enabled, you can inject the token
for the current [service account](/docs/reference/access-authn-authz/authentication/#service-account-tokens)
into a Pod at a specified path. Below is an example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-token-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: token-vol
      mountPath: "/service-account"
      readOnly: true
  volumes:
  - name: token-vol
    projected:
      sources:
      - serviceAccountToken:
          audience: api
          expirationSeconds: 3600
          path: token
```

The example Pod has a projected volume containing the injected service account
token. This token can be used by Pod containers to access the Kubernetes API
server, for example. The `audience` field contains the intended audience of the
token. A recipient of the token must identify itself with an identifier specified
in the audience of the token, and otherwise should reject the token. This field
is optional and it defaults to the identifier of the API server.

The `expirationSeconds` is the expected duration of validity of the service account
token. It defaults to 1 hour and must be at least 10 minutes (600 seconds). An administrator
can also limit its maximum value by specifying the `--service-account-max-token-expiration`
option for the API server. The `path` field specifies a relative path to the mount point
of the projected volume.

{{< note >}}
A Container using a projected volume source as a [subPath](#using-subpath) volume mount will not
receive updates for those volume sources.
{{< /note >}}
 -->

#### 示例： Pod 包含多有设置非默认权限模式的 Secret

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - secret:
          name: mysecret2
          items:
            - key: password
              path: my-group/my-password
              mode: 511
```

每一个投射(`projected`)卷的源都在配置 `sources` 下。 除了以下两个例外，其它的参数都基本相同:
- 对于 Secret, `secretName` 字段变为 `name` 以便与 ConfigMap 的命名一致
- `defaultMode` 只能在 projected 级别上设置不能在每个卷源上设置。 但是，就像上面例子中所示，
可以为每一个源设置各自独立的 `mode`

当 `TokenRequestProjection` 特性开启时，可能将当前[服务账号(ServiceAccount)]的凭据(token)
注入到 Pod 的指定路径，下面为示例:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-token-test
spec:
  containers:
  - name: container-test
    image: busybox
    volumeMounts:
    - name: token-vol
      mountPath: "/service-account"
      readOnly: true
  volumes:
  - name: token-vol
    projected:
      sources:
      - serviceAccountToken:
          audience: api
          expirationSeconds: 3600
          path: token
```

上面的例子中， `projected` 卷中包含了被注入的 服务账号凭据。 这个凭据可以被 Pod 的容器用来
访问 k8s API 服务. `audience` 字段中包含的是凭据的目标用户， 凭据的的接收者必须要验证凭据中
提供的 `audience` 与其本身的标识进行验证，否则就应该拒绝这个凭据。 这个字段为可选，默认的标识符为 API 服务

`expirationSeconds` 为服务账号凭据的有效时长。 默认为1小时，最少为 10 分钟(即 600 秒)。
管理员也可能通过API 服务的 `--service-account-max-token-expiration` 选项限制其最大值。
`path` 字段指定 `projected` 卷挂载点的相对路径。
{{< note >}}
如果一个容器以将一个 `projected` 卷以 [subPath](#using-subpath) 方式挂载，则不会接收到
卷源的更新
{{< /note >}}
<!--
### portworxVolume {#portworxvolume}

A `portworxVolume` is an elastic block storage layer that runs hyperconverged with
Kubernetes. [Portworx](https://portworx.com/use-case/kubernetes-storage/) fingerprints storage in a server, tiers based on capabilities,
and aggregates capacity across multiple servers. Portworx runs in-guest in virtual machines or on bare metal Linux nodes.

A `portworxVolume` can be dynamically created through Kubernetes or it can also
be pre-provisioned and referenced inside a Kubernetes Pod.
Here is an example Pod referencing a pre-provisioned PortworxVolume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-portworx-volume-pod
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /mnt
      name: pxvol
  volumes:
  - name: pxvol
    # This Portworx volume must already exist.
    portworxVolume:
      volumeID: "pxvol"
      fsType: "<fs-type>"
```

{{< caution >}}
Make sure you have an existing PortworxVolume with name `pxvol`
before using it in the Pod.
{{< /caution >}}

More details and examples can be found [here](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/portworx/README.md).
 -->

### portworxVolume {#portworxvolume}

`portworxVolume` 是一个弹性块存储层(elastic block storage layer), 它通过 k8s 运行
`hyperconverged`。
[Portworx](https://portworx.com/use-case/kubernetes-storage/) (这句有点绕)
Portworx 可以运行在虚拟机或裸金属的 Linux 节点中。

`portworxVolume` 可以通过 k8s 动态创建或预先创建然后在一个 k8s 的 Pod 中引用。
以下为在 Pod 中使用一个预先创建好的 `PortworxVolume` 的示例:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-portworx-volume-pod
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /mnt
      name: pxvol
  volumes:
  - name: pxvol
    # 这个 Portworx 卷必须要已经存在.
    portworxVolume:
      volumeID: "pxvol"
      fsType: "<fs-type>"
```

{{< caution >}}
在 Pod 中使用之前需要已经存在一个名叫 `pxvol` 的 `PortworxVolume`
{{< /caution >}}

更多详情及示例见 [这里](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/portworx/README.md).
<!--
### quobyte {#quobyte}

A `quobyte` volume allows an existing [Quobyte](https://www.quobyte.com) volume to
be mounted into your Pod.

{{< caution >}}
You must have your own Quobyte setup running with the volumes
created before you can use it.
{{< /caution >}}

Quobyte supports the {{< glossary_tooltip text="Container Storage Interface" term_id="csi" >}}.
CSI is the recommended plugin to use Quobyte volumes inside Kubernetes. Quobyte's
GitHub project has [instructions](https://github.com/quobyte/quobyte-csi#quobyte-csi) for deploying Quobyte using CSI, along with examples.
 -->

### quobyte {#quobyte}

`quobyte` 可以将已经存在的 [Quobyte](https://www.quobyte.com) 卷挂载到 Pod 中。
{{< caution >}}
在使用之前必须要配置好 Quobyte 并创建使用的卷
{{< /caution >}}

Quobyte 支持 {{< glossary_tooltip term_id="csi" >}}.
CSI 是在 k8s 中使用 Quobyte 卷的推荐插件。 Quobyte 的 GitHub 项目中有介绍使用 CSI 部署
Quobyte 的介绍[介绍](https://github.com/quobyte/quobyte-csi#quobyte-csi)
和示例。
<!--
### rbd {#rbd}

An `rbd` volume allows a
[Rados Block Device](https://ceph.com/docs/master/rbd/rbd/) volume to be mounted into your
Pod.  Unlike `emptyDir`, which is erased when a Pod is removed, the contents of
a `rbd` volume are preserved and the volume is merely unmounted.  This
means that a RBD volume can be pre-populated with data, and that data can
be "handed off" between Pods.

{{< caution >}}
You must have your own Ceph installation running before you can use RBD.
{{< /caution >}}

A feature of RBD is that it can be mounted as read-only by multiple consumers
simultaneously.  This means that you can pre-populate a volume with your dataset
and then serve it in parallel from as many Pods as you need.  Unfortunately,
RBD volumes can only be mounted by a single consumer in read-write mode - no
simultaneous writers allowed.

See the [RBD example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/rbd) for more details.
 -->

### rbd {#rbd}

`rbd` 卷可以让 [Rados 块设备](https://ceph.com/docs/master/rbd/rbd/)卷挂载到 Pod 中。
与 `emptyDir` 在 Pod 删除时清除数据不同， 在 `rbd` 卷中的内存会在卸载后依然会保存
其中的内容。也就是说一个 `rbd` 卷可以预先存在数据，其中的数据也可以在不同的 Pod 之间传递。
{{< caution >}}
在使用 RBD 之前需要先配置运行 Ceph
{{< /caution >}}

`RBD` 一个特性是可以以只读的方式同时挂载到多个消费者中。 这就代表着可以预先将数据集放入卷中
然后根据需要使用任意数量的 Pod 来提供访问服务。 不过 `RBD` 卷以读写方式挂载时只能有一个消费者，
不允许并行写。
更多信息见 [RBD 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/volumes/rbd)
<!--
### scaleIO {#scaleio}

ScaleIO is a software-based storage platform that can use existing hardware to
create clusters of scalable shared block networked storage. The `scaleIO` volume
plugin allows deployed Pods to access existing ScaleIO
volumes (or it can dynamically provision new volumes for persistent volume claims, see
[ScaleIO Persistent Volumes](/docs/concepts/storage/persistent-volumes/#scaleio)).

{{< caution >}}
You must have an existing ScaleIO cluster already setup and
running with the volumes created before you can use them.
{{< /caution >}}

The following is an example of Pod configuration with ScaleIO:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-0
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: pod-0
    volumeMounts:
    - mountPath: /test-pd
      name: vol-0
  volumes:
  - name: vol-0
    scaleIO:
      gateway: https://localhost:443/api
      system: scaleio
      protectionDomain: sd0
      storagePool: sp1
      volumeName: vol-0
      secretRef:
        name: sio-secret
      fsType: xfs
```

For further detail, please see the [ScaleIO examples](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/scaleio).
 -->

### scaleIO {#scaleio}

`ScaleIO` 是一个基于软件的存储平台，它可以使用已经存在的硬件来创建可扩展的共享块网络存储集群。
`scaleIO` 卷插件让 Pod 可以使用 已经存在的 ScaleIO 卷(也可以为 持久化卷声明(PersistentVolumeClaim)
动态地创建新的卷， 见 [ScaleIO 持久化卷](/k8sDocs/docs/concepts/storage/persistent-volumes/#scaleio))

{{< caution >}}
在使用 `ScaleIO` 卷之前，需要先部署配置运行 ScaleIO 集群。
{{< /caution >}}

以下为一个使用 ScaleIO 的 Pod 配置示例:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-0
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: pod-0
    volumeMounts:
    - mountPath: /test-pd
      name: vol-0
  volumes:
  - name: vol-0
    scaleIO:
      gateway: https://localhost:443/api
      system: scaleio
      protectionDomain: sd0
      storagePool: sp1
      volumeName: vol-0
      secretRef:
        name: sio-secret
      fsType: xfs
```

更多信息见 [ScaleIO 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/volumes/scaleio).
<!--
### secret {#secret}

A `secret` volume is used to pass sensitive information, such as passwords, to
Pods.  You can store secrets in the Kubernetes API and mount them as files for
use by Pods without coupling to Kubernetes directly.  `secret` volumes are
backed by tmpfs (a RAM-backed filesystem) so they are never written to
non-volatile storage.

{{< caution >}}
You must create a secret in the Kubernetes API before you can use it.
{{< /caution >}}

{{< note >}}
A Container using a Secret as a [subPath](#using-subpath) volume mount will not
receive Secret updates.
{{< /note >}}

Secrets are described in more detail [here](/docs/concepts/configuration/secret/).
 -->

### secret {#secret}

`secret` 卷用于将如密码这样的敏感数据传入 Pod 中。 用户可以将 Secret 存在 k8s API 中，然后
将它们以文件的方式挂载到 Pod 中而不需要直接与 k8s 耦合在一起。 `secret` 卷基于 tmpfs(
一个基于内存的文件系统)，所以这给永远不会写入持久性存储。
{{< caution >}}
在使用之前需要先在 k8s API 中先创建相应的 Secret
{{< /caution >}}

{{< note >}}
一个以 [subPath](#using-subpath) 挂载 Secret 的容器不会接收到 Secret 的更新。
{{< /note >}}

更多关于 Secret 的信息见 [这里](/k8sDocs/docs/concepts/configuration/secret/).
<!--
### storageOS {#storageos}

A `storageos` volume allows an existing [StorageOS](https://www.storageos.com)
volume to be mounted into your Pod.

StorageOS runs as a Container within your Kubernetes environment, making local
or attached storage accessible from any node within the Kubernetes cluster.
Data can be replicated to protect against node failure. Thin provisioning and
compression can improve utilization and reduce cost.

At its core, StorageOS provides block storage to Containers, accessible via a file system.

The StorageOS Container requires 64-bit Linux and has no additional dependencies.
A free developer license is available.

{{< caution >}}
You must run the StorageOS Container on each node that wants to
access StorageOS volumes or that will contribute storage capacity to the pool.
For installation instructions, consult the
[StorageOS documentation](https://docs.storageos.com).
{{< /caution >}}

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: redis
    role: master
  name: test-storageos-redis
spec:
  containers:
    - name: master
      image: kubernetes/redis:v1
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      volumeMounts:
        - mountPath: /redis-master-data
          name: redis-data
  volumes:
    - name: redis-data
      storageos:
        # The `redis-vol01` volume must already exist within StorageOS in the `default` namespace.
        volumeName: redis-vol01
        fsType: ext4
```

For more information including Dynamic Provisioning and Persistent Volume Claims, please see the
[StorageOS examples](https://github.com/kubernetes/examples/blob/master/volumes/storageos).
 -->

### storageOS {#storageos}

`storageos` 卷可以将现有的 [StorageOS](https://www.storageos.com) 卷挂载到 Pod 中。

StorageOS 以容器方式运行在 k8s 环境中，可以将集群中任意节点的本地存储或附加存储变为可访问。
数据可以被复制以提高可用性。瘦供给或压缩可以改善利用率并减少开销。
StorageOS 核心是向容器提供块存储，可以通过文件系统访问。
StorageOS 容器需要 64 位 Linux 外不需要其它额外的信赖。
有一个免费的开发者许可可用。
{{< caution >}}
需要在想让 StorageOS 访问的节点上运行 StorageOS 容器这样可以将它的存储容量加到池中。
更新安装指导见 [StorageOS 文档](https://docs.storageos.com).
{{< /caution >}}

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: redis
    role: master
  name: test-storageos-redis
spec:
  containers:
    - name: master
      image: kubernetes/redis:v1
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      volumeMounts:
        - mountPath: /redis-master-data
          name: redis-data
  volumes:
    - name: redis-data
      storageos:
        # The `redis-vol01` volume must already exist within StorageOS in the `default` namespace.
        volumeName: redis-vol01
        fsType: ext4
```

更多关于动态管理和 PersistentVolumeClaim 的信息见
[StorageOS 示例](https://github.com/kubernetes/examples/blob/master/volumes/storageos).
<!--
### vsphereVolume {#vspherevolume}

{{< note >}}
Prerequisite: Kubernetes with vSphere Cloud Provider configured. For cloudprovider
configuration please refer [vSphere getting started guide](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/).
{{< /note >}}

A `vsphereVolume` is used to mount a vSphere VMDK Volume into your Pod.  The contents
of a volume are preserved when it is unmounted. It supports both VMFS and VSAN datastore.

{{< caution >}}
You must create VMDK using one of the following methods before using with Pod.
{{< /caution >}}
 -->

### vsphereVolume {#vspherevolume}

{{< note >}}
前置条件: k8s 需要配置好 vSphere 云提供商。 cloudprovider 配置见
[vSphere 上手指导](https://vmware.github.io/vsphere-storage-for-kubernetes/documentation/).
{{< /note >}}

`vsphereVolume` 用于将  vSphere VMDK 卷挂载到 Pod 中。 卷中的数据在卸载后依然存在。
支持 VMFS 和 VSAN 数据存储

{{< caution >}}
需要使用以下方式中的其中一种创建 VMDK 后才能在 Pod 中使用。
{{< /caution >}}
<!--
#### Creating a VMDK volume

Choose one of the following methods to create a VMDK.

{{< tabs name="tabs_volumes" >}}
{{% tab name="Create using vmkfstools" %}}
First ssh into ESX, then use the following command to create a VMDK:

```shell
vmkfstools -c 2G /vmfs/volumes/DatastoreName/volumes/myDisk.vmdk
```
{{% /tab %}}
{{% tab name="Create using vmware-vdiskmanager" %}}
Use the following command to create a VMDK:

```shell
vmware-vdiskmanager -c -t 0 -s 40GB -a lsilogic myDisk.vmdk
```
{{% /tab %}}

{{< /tabs >}}
 -->

#### 创建 VMDK 卷

选择以下方式中的一种创建 VMDK

{{< tabs name="tabs_volumes" >}}
{{% tab name="Create using vmkfstools" %}}
先 ssh 到 ESX 中，然后使用用下面的命令创建一个 VMDK
```shell
vmkfstools -c 2G /vmfs/volumes/DatastoreName/volumes/myDisk.vmdk
```
{{% /tab %}}
{{% tab name="Create using vmware-vdiskmanager" %}}
使用下面的命令创建一个 VMDK
```shell
vmware-vdiskmanager -c -t 0 -s 40GB -a lsilogic myDisk.vmdk
```
{{% /tab %}}

{{< /tabs >}}

<!--
#### vSphere VMDK Example configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-vmdk
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-vmdk
      name: test-volume
  volumes:
  - name: test-volume
    # This VMDK volume must already exist.
    vsphereVolume:
      volumePath: "[DatastoreName] volumes/myDisk"
      fsType: ext4
```
 -->

#### vSphere VMDK 配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-vmdk
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-vmdk
      name: test-volume
  volumes:
  - name: test-volume
    # 这个 VMDK 必须要已经存在.
    vsphereVolume:
      volumePath: "[DatastoreName] volumes/myDisk"
      fsType: ext4
```

更多信息见 [这里](https://github.com/kubernetes/examples/tree/master/staging/volumes/vsphere).
<!--
#### CSI migration

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

The CSI Migration feature for vsphereVolume, when enabled, shims all plugin operations
from the existing in-tree plugin to the `csi.vsphere.vmware.com` {{< glossary_tooltip text="CSI" term_id="csi" >}} driver. In order to use this feature, the [vSphere CSI
Driver](https://github.com/kubernetes-sigs/vsphere-csi-driver)
must be installed on the cluster and the `CSIMigration` and `CSIMigrationvSphere`
[feature gates](/docs/reference/command-line-tools-reference/feature-gates/) must be enabled.

This also requires minimum vSphere vCenter/ESXi Version to be 7.0u1 and minimum HW Version to be VM version 15.

{{< note >}}
The following StorageClass parameters from the built-in vsphereVolume plugin are not supported by the vSphere CSI driver:

* `diskformat`
* `hostfailurestotolerate`
* `forceprovisioning`
* `cachereservation`
* `diskstripes`
* `objectspacereservation`
* `iopslimit`

Existing volumes created using these parameters will be migrated to the vSphere CSI driver, but new volumes created by the vSphere CSI driver will not be honoring these parameters.
{{< /note >}}
 -->

#### CSI 迁移

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

的 CSI 迁移特性，在启用时，会将来自已经存在的插件的插件操作转移到
`csi.vsphere.vmware.com` {{< glossary_tooltip text="CSI" term_id="csi" >}}(CSI)驱动。
为了使用该特性， 需要在集群中先安装
[vSphere CSI Driver](https://github.com/kubernetes-sigs/vsphere-csi-driver)
同时在
[功能阀](/k8sDocs/reference/command-line-tools-reference/feature-gates/)
启用 `CSIMigration` 和 `CSIMigrationAzureFile` 两个 `Alpha` 特性
This also requires minimum vSphere vCenter/ESXi Version to be 7.0u1 and minimum HW Version to be VM version 15.
同时还需要 vSphere vCenter/ESXi v7.0u1+ 和 VM v15+
{{< note >}}
以下来自 vsphereVolume 插件内置的 StorageClass 参数不受 vSphere CSI 驱动支持:
* `diskformat`
* `hostfailurestotolerate`
* `forceprovisioning`
* `cachereservation`
* `diskstripes`
* `objectspacereservation`
* `iopslimit`

已经存在使用这些参数创建的卷会被迁移到 vSphere CSI 驱动， 但是使用 vSphere CSI 驱动创建新
的卷时就不能再使用这些参数了
{{< /note >}}
<!--
#### CSI Migration Complete
{{< feature-state for_k8s_version="v1.19" state="beta" >}}

To turn off the vsphereVolume plugin from being loaded by controller manager and kubelet, you need to set this feature flag to true. This requires `csi.vsphere.vmware.com` {{< glossary_tooltip text="CSI" term_id="csi" >}} driver being installed on all worker nodes.
 -->

#### CSI Migration Complete
{{< feature-state for_k8s_version="v1.19" state="beta" >}}

为了关闭由控制器管理器和 kubelet 加载的 vsphereVolume 插件， 需要将这具功能特性标记为 true
同时还需要在每个节点上安装 `csi.vsphere.vmware.com`
{{< glossary_tooltip term_id="csi" >}}
驱动
<!--
## Using subPath

Sometimes, it is useful to share one volume for multiple uses in a single Pod. The `volumeMounts.subPath`
property can be used to specify a sub-path inside the referenced volume instead of its root.

Here is an example of a Pod with a LAMP stack (Linux Apache Mysql PHP) using a single, shared volume.
The HTML contents are mapped to its `html` folder, and the databases will be stored in its `mysql` folder:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```
 -->

## 使用 subPath

有时，在同一个 Pod 中的多个用户分享同一个卷是相当有用的。 `volumeMounts.subPath` 属性可以用来
在引用卷时不使用其根路径而是指定子路径

以下示例 Pod 为一个使用一个共享卷的 LAMP (Linux Apache Mysql PHP) 栈。
HTML 内容映射到 `html` 目录， 数据库会使用 `mysql` 目录。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```
<!--
### Using subPath with expanded environment variables

{{< feature-state for_k8s_version="v1.17" state="stable" >}}


Use the `subPathExpr` field to construct `subPath` directory names from Downward API environment variables.
The `subPath` and `subPathExpr` properties are mutually exclusive.

In this example, a Pod uses `subPathExpr` to create a directory `pod1` within the hostPath volume `/var/log/pods`, using the pod name from the Downward API.  The host directory `/var/log/pods/pod1` is mounted at `/logs` in the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```
 -->

### 在 subPath 配置上使用环境变量

{{< feature-state for_k8s_version="v1.17" state="stable" >}}

使用 `subPathExpr` 字段来通过 Downward API 的环境变量构建 `subPath` 的名称。
`subPath` 和 `subPathExpr` 相互独立。

在这个例子中， 一个 Pod 使用 `subPathExpr` 创建一个 `pod1` 目录在 hostPath 卷 `/var/log/pods` 中，
使用来自 Downward API 的 Pod 名称。 主机目录 `/var/log/pods/pod1` 挂载到容器中的 `/logs` 上。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```
<!--
## Resources

The storage media (Disk, SSD, etc.) of an `emptyDir` volume is determined by the
medium of the filesystem holding the kubelet root dir (typically
`/var/lib/kubelet`).  There is no limit on how much space an `emptyDir` or
`hostPath` volume can consume, and no isolation between Containers or between
Pods.

In the future, we expect that `emptyDir` and `hostPath` volumes will be able to
request a certain amount of space using a [resource](/docs/concepts/configuration/manage-resources-containers/)
specification, and to select the type of media to use, for clusters that have
several media types.
 -->

## 资源

`emptyDir` 卷的存储介质(Disk, SSD, etc.)是由 kubelet 根目录(通常为 `/var/lib/kubelet`)
所在的文件系统的存储介质决定的。 `emptyDir` 和 `hostPath` 卷没有使用容量限制，容器和 Pod
之间也没有限离。

在将来，我们预计将通过
[资源](/k8sDocs/docs/concepts/configuration/manage-resources-containers/)
来确定 `emptyDir` 和 `hostPath` 请求确定数量的空间，如果集群中有多种介质也 选择要使用的介质
<!--
## Out-of-Tree Volume Plugins

The Out-of-tree volume plugins include the Container Storage Interface (CSI)
and FlexVolume. They enable storage vendors to create custom storage plugins
without adding them to the Kubernetes repository.

Before the introduction of CSI and FlexVolume, all volume plugins (like
volume types listed above) were "in-tree" meaning they were built, linked,
compiled, and shipped with the core Kubernetes binaries and extend the core
Kubernetes API. This meant that adding a new storage system to Kubernetes (a
volume plugin) required checking code into the core Kubernetes code repository.

Both CSI and FlexVolume allow volume plugins to be developed independent of
the Kubernetes code base, and deployed (installed) on Kubernetes clusters as
extensions.

For storage vendors looking to create an out-of-tree volume plugin, please refer
to [this FAQ](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md).
 -->

## 外部卷插件


外部卷插件包括 {{<glossary_tooltip term_id="csi">}} 和 FlexVolume。它们使得存储提供商
可能创建自定义的存储插件而不需要将其添加到 k8s 代码库中。

在引入 CSI 和 FlexVolume 之前， 所有的卷插件(像所有前面列举的那些)都是内置的，它们在构建，连接
编译，发布都是与 k8s 核心二进制文件在一起的并且扩展了 k8s 核心 API。 这意味着如果我们要添加
一种新的存储系统(一个卷插件)到 k8s 中时，需要将其代码检入到 k8s 核心代码库中。

CSI 和 FlexVolume 都允许独立于 k8s 核心代码独立开发卷插件，并且独立以插件的方式部署(安装)
到 k8s 集群中。

要看看有哪些外部卷插件，请见
[这个 FAQ](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md).
<!--
### CSI

[Container Storage Interface](https://github.com/container-storage-interface/spec/blob/master/spec.md) (CSI)
defines a standard interface for container orchestration systems (like
Kubernetes) to expose arbitrary storage systems to their container workloads.

Please read the [CSI design proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md) for more information.

CSI support was introduced as alpha in Kubernetes v1.9, moved to beta in
Kubernetes v1.10, and is GA in Kubernetes v1.13.

{{< note >}}
Support for CSI spec versions 0.2 and 0.3 are deprecated in Kubernetes
v1.13 and will be removed in a future release.
{{< /note >}}

{{< note >}}
CSI drivers may not be compatible across all Kubernetes releases.
Please check the specific CSI driver's documentation for supported
deployments steps for each Kubernetes release and a compatibility matrix.
{{< /note >}}

Once a CSI compatible volume driver is deployed on a Kubernetes cluster, users
may use the `csi` volume type to attach, mount, etc. the volumes exposed by the
CSI driver.

A `csi` volume can be used in a pod in three different ways:
- through a reference to a [`persistentVolumeClaim`](#persistentvolumeclaim)
- with a [generic ephemeral volume](/docs/concepts/storage/ephemeral-volumes/#generic-ephemeral-volume) (alpha feature)
- with a [CSI ephemeral volume](/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volume) if the driver
  supports that (beta feature)

The following fields are available to storage administrators to configure a CSI
persistent volume:

- `driver`: A string value that specifies the name of the volume driver to use.
  This value must correspond to the value returned in the `GetPluginInfoResponse`
  by the CSI driver as defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#getplugininfo).
  It is used by Kubernetes to identify which CSI driver to call out to, and by
  CSI driver components to identify which PV objects belong to the CSI driver.
- `volumeHandle`: A string value that uniquely identifies the volume. This value
  must correspond to the value returned in the `volume.id` field of the
  `CreateVolumeResponse` by the CSI driver as defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume).
  The value is passed as `volume_id` on all calls to the CSI volume driver when
  referencing the volume.
- `readOnly`: An optional boolean value indicating whether the volume is to be
  "ControllerPublished" (attached) as read only. Default is false. This value is
  passed to the CSI driver via the `readonly` field in the
  `ControllerPublishVolumeRequest`.
- `fsType`: If the PV's `VolumeMode` is `Filesystem` then this field may be used
  to specify the filesystem that should be used to mount the volume. If the
  volume has not been formatted and formatting is supported, this value will be
  used to format the volume.
  This value is passed to the CSI driver via the `VolumeCapability` field of
  `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`, and
  `NodePublishVolumeRequest`.
- `volumeAttributes`: A map of string to string that specifies static properties
  of a volume. This map must correspond to the map returned in the
  `volume.attributes` field of the `CreateVolumeResponse` by the CSI driver as
  defined in the [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume).
  The map is passed to the CSI driver via the `volume_context` field in the
  `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`, and
  `NodePublishVolumeRequest`.
- `controllerPublishSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `ControllerPublishVolume` and `ControllerUnpublishVolume` calls. This field is
  optional, and may be empty if no secret is required. If the secret object
  contains more than one secret, all secrets are passed.
- `nodeStageSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `NodeStageVolume` call. This field is optional, and may be empty if no secret
  is required. If the secret object contains more than one secret, all secrets
  are passed.
- `nodePublishSecretRef`: A reference to the secret object containing
  sensitive information to pass to the CSI driver to complete the CSI
  `NodePublishVolume` call. This field is optional, and may be empty if no
  secret is required. If the secret object contains more than one secret, all
  secrets are passed.
-->
### CSI

[容器存储接口](https://github.com/container-storage-interface/spec/blob/master/spec.md) (CSI)
defines a standard interface for container orchestration systems (like
Kubernetes) to expose arbitrary storage systems to their container workloads.
定义一个标准接口用于容器编排系统(像 k8s)暴露任意存储系统到它们的容器工作负载中。
更多信息请见 [CSI 设计方案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md)

CSI 支持是在 k8s v1.9 中引入为 alpha， 在 k8s v1.10 进变为 beta 版本， k8s v1.13 变为 GA
{{< note >}}
对 CSI 0.2 和 0.3 标准的支持在 k8s v1.13 中被废弃并会在将来被移除
{{< /note >}}

{{< note >}}
CSI 驱动可能不能与所有的 k8s 发行版本兼容。请查阅对应 CSI 驱动 文档，了解其在每一个 k8s 版本
中的部署方式和对应的版本兼容表格
{{< /note >}}

当一个 CSI 兼容的卷驱动被部署到 k8s 集群中时， 用户可以使用 `csi` 卷类型为装载，挂载那些
通过 CSI 驱动暴露的卷。

一个 `csi` 卷可以以下三种方式在 Pod 中使用:
- 通过一个 [`persistentVolumeClaim`](#persistentvolumeclaim) 引用
- 通过[generic ephemeral volume](/docs/concepts/storage/ephemeral-volumes/#generic-ephemeral-volume)(alpha 特性)
- 如果驱动支持，通过 [CSI ephemeral volume](/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volume)  (beta 特性)

以下字段可以用于存储管理员对 CSI 持久卷的配置:

- `driver`: 一个字符串，用于指定卷所使用的驱动的名称。 取值范围是
  [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#getplugininfo)
  中定义的 CSI 驱动 `GetPluginInfoResponse` 的返回值。 它被 k8s 用来识别，应该调用哪个 CSI 驱动
  我大兵由 CSI 驱动组件来识别哪些 PV 对象是属于该 CSI 驱动的

- `volumeHandle`: 一个字符串， 用于唯一标识该卷。 这个值必须是在
  [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume).
  定义的 CSI 驱动 `CreateVolumeResponse` 返回值中的 `volume.id` 的值。 在所有引用这个
  卷时会该这个值 以 `volume_id` 调用参数发给 CSI 卷驱动

- `readOnly`: 一个可选布尔值，用于指定该卷是否以只读方式挂载(ControllerPublished)。
  默认值为 false. 这个值通过 `ControllerPublishVolumeRequest` 中 `readonly` 传递给
  CSI 驱动

- `fsType`: 如果 PV 的 `VolumeMode` 是 `Filesystem`，这个字段就可以用来指定挂载该卷所使用
  的文件系统。 如果这个卷还没有被格式化并支持格式化，则这个被被用来格式化这个卷。 这个值通过
  `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`, `NodePublishVolumeRequest`
  的 `VolumeCapability` 字段传递给 CSI 驱动。

- `volumeAttributes`: 一个键值都是字符串的字典，用于指定卷的静态属性。这个字段的键值必须是
  [CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md#createvolume)
  中定义 CSI 驱动 `CreateVolumeResponse` 返回值的 `volume.attributes` 字段的键值的子集
  这个字典通过 `ControllerPublishVolumeRequest`, `NodeStageVolumeRequest`,
  `NodePublishVolumeRequest` 的 `volume_context` 字段传递给 CSI 驱动

- `controllerPublishSecretRef`:  完成 `ControllerPublishVolume` 和
  `ControllerUnpublishVolume` 调用向 CSI 驱动传递敏感信息的 Secret 对象的引用。该字段为
  可选， 如果没使用 Secret 则留空。 如果 Secret 对象中有多个 Secret, 则所有的都会传递。

- `nodeStageSecretRef`: 为了完成 `NodeStageVolume` 调用， 向 CSI 驱动传递敏感信息的
  Secret 对象的引用。该字段为 可选， 如果没使用 Secret 则留空。
  如果 Secret 对象中有多个 Secret, 则所有的都会传递。

- `nodePublishSecretRef`:  为了完成 `NodePublishVolume` 调用， 向 CSI 驱动传递敏感信息的
  Secret 对象的引用。该字段为 可选， 如果没使用 Secret 则留空。
  如果 Secret 对象中有多个 Secret, 则所有的都会传递。
<!--
#### CSI raw block volume support

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

Vendors with external CSI drivers can implement raw block volumes support
in Kubernetes workloads.

You can [setup your PV/PVC with raw block volume support](/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)
as usual, without any CSI specific changes.
 -->

#### CSI 原生块卷支持

{{< feature-state for_k8s_version="v1.18" state="stable" >}}

有外部 CSI 驱动的供应商可以实现对 k8s 工作负载原生块卷的支持

用户可以在不作任何 CSI 配置修改的情况下
[设置支持原生块卷的 PV/PVC](/k8sDocs/docs/concepts/storage/persistent-volumes/#raw-block-volume-support)
<!--
#### CSI ephemeral volumes

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

You can directly configure CSI volumes within the Pod
specification. Volumes specified in this way are ephemeral and do not
persist across Pod restarts. See [Ephemeral
Volumes](/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volume)
for more information.
 -->
#### CSI 临时卷

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

You can directly configure CSI volumes within the Pod
specification. Volumes specified in this way are ephemeral and do not
persist across Pod restarts. See [Ephemeral
Volumes](/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volume)
for more information.
用户可以在 Pod 定义中直接配置 CSI 卷。 但是通过这种方式创建的卷是临时，Pod 重启后就不会存在。
更多信息请见
[临时卷](/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volume)
<!--
#### {{% heading "whatsnext" %}}

For more information on how to develop a CSI driver, refer to the [kubernetes-csi
documentation](https://kubernetes-csi.github.io/docs/)
 -->

#### {{% heading "whatsnext" %}}

更多关于如何开发 CSI 驱动的信息，请见
[kubernetes-csi 文档](https://kubernetes-csi.github.io/docs/)
<!--
#### Migrating to CSI drivers from in-tree plugins

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

The CSI Migration feature, when enabled, directs operations against existing in-tree
plugins to corresponding CSI plugins (which are expected to be installed and configured).
The feature implements the necessary translation logic and shims to re-route the
operations in a seamless fashion. As a result, operators do not have to make any
configuration changes to existing Storage Classes, PVs or PVCs (referring to
in-tree plugins) when transitioning to a CSI driver that supersedes an in-tree plugin.

In the alpha state, the operations and features that are supported include
provisioning/delete, attach/detach, mount/unmount and resizing of volumes.

In-tree plugins that support CSI Migration and have a corresponding CSI driver implemented
are listed in the "Types of Volumes" section above.
 -->

#### 从内部插件迁移到 CSI 驱动

{{< feature-state for_k8s_version="v1.14" state="alpha" >}}

当启用 CSI 迁移特性后，当内部插件有对应的 CSI 插件(已经完成安装配置)时。这个特性实现了必要的翻译
逻辑将操作无缝地转移。 最终，操作者不需要对已经存在的 StorageClass， PV， PVC(使用内部插件的)
配置做任何修改而实现 从内部插件迁移到 CSI 驱动上。

在 alpha 状态，支持的操作和特性包括 管理/删除 attach/detach, mount/unmount 和修改卷大小。

支持 CSI 迁移并且有对应 CSI 驱动实现的内部插件见上面的 [卷类型](#types-of-volumes) 章节
### FlexVolume {#flexVolume}

FlexVolume 是一个外部插件接口，它从 k8v v1.2 (在 CSI 之间)就存在了。 它使用一个基于 exec
的模式与驱动交互。 `FlexVolume` 驱动的二进制必须要预先安装在节点上一个预定义插件路径中(有些情况 master 也要装)

Pod 通过内部插件 `flexvolume` 与 FlexVolume 驱动交互，更多信息请见
[这里](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-storage/flexvolume.md).
<!--
## Mount propagation

Mount propagation allows for sharing volumes mounted by a Container to
other Containers in the same Pod, or even to other Pods on the same node.

Mount propagation of a volume is controlled by `mountPropagation` field in Container.volumeMounts.
Its values are:

 * `None` - This volume mount will not receive any subsequent mounts
   that are mounted to this volume or any of its subdirectories by the host.
   In similar fashion, no mounts created by the Container will be visible on
   the host. This is the default mode.

   This mode is equal to `private` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

 * `HostToContainer` - This volume mount will receive all subsequent mounts
   that are mounted to this volume or any of its subdirectories.

   In other words, if the host mounts anything inside the volume mount, the
   Container will see it mounted there.

   Similarly, if any Pod with `Bidirectional` mount propagation to the same
   volume mounts anything there, the Container with `HostToContainer` mount
   propagation will see it.

   This mode is equal to `rslave` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

 * `Bidirectional` - This volume mount behaves the same the `HostToContainer` mount.
   In addition, all volume mounts created by the Container will be propagated
   back to the host and to all Containers of all Pods that use the same volume.

   A typical use case for this mode is a Pod with a FlexVolume or CSI driver or
   a Pod that needs to mount something on the host using a `hostPath` volume.

   This mode is equal to `rshared` mount propagation as described in the
   [Linux kernel documentation](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)

{{< caution >}}
`Bidirectional` mount propagation can be dangerous. It can damage
the host operating system and therefore it is allowed only in privileged
Containers. Familiarity with Linux kernel behavior is strongly recommended.
In addition, any volume mounts created by Containers in Pods must be destroyed
(unmounted) by the Containers on termination.
{{< /caution >}}
 -->
## 挂载传播

挂载传播可以让同一个 Pod 中的一个容器上挂载的卷可以共享给另一个容器，甚至可以共享给同一个节点上
的其它 Pod

一个卷的挂载传播受容 `Container.volumeMounts` 的 `mountPropagation` 字段控制。可选值有:

- `None` 这个卷挂载不会接收到任何后续由主机挂载到这个卷或任意其子目录的挂载。
  类似地，由容器创建的挂载在主机上不可见。 这是默认模式。

  这个模式与
  [Linux 内核文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)
  中描述的 `private` 挂载传播等效。

- `HostToContainer` 这个卷挂载会接收所有后继到这个卷或其子目录的挂载。

  换种方式来说， 如果主机挂载了任意内容到这个卷挂载，容器中就会看到挂载的内容。

  相似地，如果任意使用 `Bidirectional` 挂载传播的 Pod 挂载了任意内容， 使用 `HostToContainer`
  挂载传播的容器也会看到这个挂载。

  这种模式与
  [Linux 内核文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)
  中描述的 `rslave` 挂载传播等效。

- `Bidirectional` 这种卷挂载的表现方式与 `HostToContainer` 挂载相同。 同时，所以由容器创建
  卷挂载也会反向传播给主机和所以使用这个卷的所有 Pod 的所有容器。

  使用这种模式的一个典型场景是一个使用 `FlexVolume` 或 CSI 驱动容器 或 一个需要使用 `hostPath`
  卷挂载一些主机内容的容器。
{{< caution >}}
`Bidirectional` 挂载传播可能会相当危险。 它可能危害主机操作系统并且只能在提权容器上使用。
强烈建议操作使用前要相当熟悉  Linux 内核行为。 另外，由 Pod 中容器创建的卷挂载必须要在容器被终结时销毁
(卸载)
{{< /caution >}}
<!--
### Configuration
Before mount propagation can work properly on some deployments (CoreOS,
RedHat/Centos, Ubuntu) mount share must be configured correctly in
Docker as shown below.

Edit your Docker's `systemd` service file.  Set `MountFlags` as follows:
```shell
MountFlags=shared
```
Or, remove `MountFlags=slave` if present.  Then restart the Docker daemon:
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```
 -->

### 配置
Before mount propagation can work properly on some deployments (CoreOS,
RedHat/Centos, Ubuntu) mount share must be configured correctly in
Docker as shown below.
想要让挂载传播在某些部署(CoreOS, RedHat/Centos, Ubuntu)上正常工作，需要在 Docker 上
进行正确的共享挂载配置，具体如下:
编辑 Docker `systemd` 服务配置文件，将 `MountFlags` 设置如下:

```
MountFlags=shared
```
或者，如果有配置为 `MountFlags=slave` 则将其移除。然后重启 Docker 守护进程:
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## {{% heading "whatsnext" %}}
<!--
* Follow an example of [deploying WordPress and MySQL with Persistent Volumes](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/).
-->

- 实践 [部署带有持久卷的 WordPress 和 MySQL](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/).
