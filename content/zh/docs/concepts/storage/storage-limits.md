---
title: 节点级别的卷限制
content_type: concept
weight: 100
---
<!--
---
reviewers:
- jsafrane
- saad-ali
- thockin
- msau42
title: Node-specific Volume Limits
content_type: concept
---
 -->
<!-- overview -->
<!--
This page describes the maximum number of volumes that can be attached
to a Node for various cloud providers.

Cloud providers like Google, Amazon, and Microsoft typically have a limit on
how many volumes can be attached to a Node. It is important for Kubernetes to
respect those limits. Otherwise, Pods scheduled on a Node could get stuck
waiting for volumes to attach.
-->

本文主要介绍在各家云提供商上每个节点能够挂载的卷的最大数量。
像 Google, Amazon, 和 Microsoft 这样典型的云提供上对于每个节点上能挂载的卷数量是有限制的。
让 k8s 遵守这个限制是比较重要的。 否则，Pod 在调度到一个节点后就会卡在等待卷挂载上。

<!-- body -->
<!--
## Kubernetes default limits

The Kubernetes scheduler has default limits on the number of volumes
that can be attached to a Node:

<table>
  <tr><th>Cloud service</th><th>Maximum volumes per Node</th></tr>
  <tr><td><a href="https://aws.amazon.com/ebs/">Amazon Elastic Block Store (EBS)</a></td><td>39</td></tr>
  <tr><td><a href="https://cloud.google.com/persistent-disk/">Google Persistent Disk</a></td><td>16</td></tr>
  <tr><td><a href="https://azure.microsoft.com/en-us/services/storage/main-disks/">Microsoft Azure Disk Storage</a></td><td>16</td></tr>
</table>
 -->

## k8s 默认的限制

k8s 调度器对每个节点上能够挂载的卷的数量是有限制的:

|云服务|每个节点挂载卷的最大值|
|-----|------------------|
|[Amazon Elastic Block Store (EBS)](https://aws.amazon.com/ebs/)|39|
|[Google Persistent Disk](https://cloud.google.com/persistent-disk/)|16|
|[Microsoft Azure Disk Storage](https://azure.microsoft.com/en-us/services/storage/main-disks/)|16|

<!--
## Custom limits

You can change these limits by setting the value of the
`KUBE_MAX_PD_VOLS` environment variable, and then starting the scheduler.
CSI drivers might have a different procedure, see their documentation
on how to customize their limits.

Use caution if you set a limit that is higher than the default limit. Consult
the cloud provider's documentation to make sure that Nodes can actually support
the limit you set.

The limit applies to the entire cluster, so it affects all Nodes.
-->
<!--
## Custom limits

You can change these limits by setting the value of the
`KUBE_MAX_PD_VOLS` environment variable, and then starting the scheduler.
CSI drivers might have a different procedure, see their documentation
on how to customize their limits.

Use caution if you set a limit that is higher than the default limit. Consult
the cloud provider's documentation to make sure that Nodes can actually support
the limit you set.

The limit applies to the entire cluster, so it affects all Nodes.
 -->

## 自定义限制


可以通过设置 `KUBE_MAX_PD_VOLS` 环境变量的值来修改这个限制， 修改后重新启动调度器。
CSI 驱动可能需要不同的步骤，具体怎么修改这个自定义配置见他们的文档。

要小心设置了一个比默认值大的限制。 参照云提供商的文档来决定节点实际支持的限制再设置这个值。

这个限制会应用到整个集群，所以它会影响所有的节点。

<!--
## Dynamic volume limits

{{< feature-state state="stable" for_k8s_version="v1.17" >}}

Dynamic volume limits are supported for following volume types.

- Amazon EBS
- Google Persistent Disk
- Azure Disk
- CSI

For volumes managed by in-tree volume plugins, Kubernetes automatically determines the Node
type and enforces the appropriate maximum number of volumes for the node. For example:

* On
<a href="https://cloud.google.com/compute/">Google Compute Engine</a>,
up to 127 volumes can be attached to a node, [depending on the node
type](https://cloud.google.com/compute/docs/disks/#pdnumberlimits).

* For Amazon EBS disks on M5,C5,R5,T3 and Z1D instance types, Kubernetes allows only 25
volumes to be attached to a Node. For other instance types on
<a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (EC2)</a>,
Kubernetes allows 39 volumes to be attached to a Node.

* On Azure, up to 64 disks can be attached to a node, depending on the node type. For more details, refer to [Sizes for virtual machines in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes).

* If a CSI storage driver advertises a maximum number of volumes for a Node (using `NodeGetInfo`), the {{< glossary_tooltip text="kube-scheduler" term_id="kube-scheduler" >}} honors that limit.
Refer to the [CSI specifications](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodegetinfo) for details.

* For volumes managed by in-tree plugins that have been migrated to a CSI driver, the maximum number of volumes will be the one reported by the CSI driver.
-->

## 动态卷限制

{{< feature-state state="stable" for_k8s_version="v1.17" >}}

以下类型的卷支持动态卷限制：
- Amazon EBS
- Google Persistent Disk
- Azure Disk
- CSI

对于由内部卷插件管理的卷， k8s 自动决定节点的类型并为节点设置合适的最大卷数量限制。 例如:

- 在
  [Google Compute Engine](https://cloud.google.com/compute/)
  上
  [基于节点的类型](https://cloud.google.com/compute/docs/disks/#pdnumberlimits).
  每个节点最多可以挂载 127 个卷。

- 对于 Amazon EBS 磁盘， 在 M5,C5,R5,T3 和 Z1D 类型的实例上，k8s 允许挂载到每个节点上的
  卷的最大值是 25. 对于
  [Amazon Elastic Compute Cloud (EC2)](https://aws.amazon.com/ec2/)
  上其它类型的实例类型， k8s 允许挂载到每个节点上能挂载卷的最大值是 39

- 在 Azure, 基于节点类型的不同，每个节点上能挂载磁盘数量是 64， 更多信息见
  [Azure 中虚拟机的大小](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes)

- 如果一个 CSI 存储驱动指定了每个节点的最大卷数量(通过 `NodeGetInfo`)，
  {{< glossary_tooltip text="kube-scheduler" term_id="kube-scheduler" >}}， 也会
  接受这个限制。
  详见
  [CSI specifications](https://github.com/container-storage-interface/spec/blob/master/spec.md#nodegetinfo)

- 对于由内部插件管理的卷，但已经迁移到了 CSI 驱动的，节点能够挂载的卷的最大数量会由 CSI 驱动
  来报告这个值。
