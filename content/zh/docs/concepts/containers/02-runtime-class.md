---
title: Runtime Class
date: 2020-07-27
publishdate: 2020-07-28
weight: 20202
---
<!-- overview -->
{{< feature-state for_k8s_version="v1.14" state="beta" >}}
本文介绍 `RuntimeClass` 资源和运行时选择机制。
`RuntimeClass` 是一个选择容器运行时配置的特性。而容器运行时配置则用于运行 Pod 中的容器的。
<!-- body -->

## Motivation
用户可以在不同的 Pod 上配置不同的 `RuntimeClass` 以达成性能与安全之间的平衡。 例如，有一部分工作负载需要一个高级别的信息安全保降。这时就需要选择装 Pod 调度到运行在物理虚拟化的容器运行时。 这样才有相对其它运行时更高的隔离级别同时也会有额外的资源开销。
`RuntimeClass` 也可以用于在同一个运行时上，对不同的 Pod 使用不同的配置。

## 设置
确保 `RuntimeClass` 功能特性是开启的(默认开启)。关于如何开启或关闭功能特性见[这里](../../../reference/command-line-tools-reference/feature-gates/)。 apiserver 和 kubelet 的`RuntimeClass` 功能特性必须要开启。
  1. 配置节点上的 CRI 实现(各种运行时配置方式不同)
  2. 创建相应的 `RuntimeClass` 资源

1. 配置节点上的 CRI 实现

基于 RuntimeClass 的配置因 容器运行时接口(CRI)具体实现的不同而不同。 具体配置文档见[下面章节](#cri-configuration)
{{<note>}}
默认情况下 `RuntimeClass` 假定整个集群中所有节点的配置是相同的(也就是说所有节点针对容器运行时的配置是一样的)，为了支持这样的配置请见[下面章节-调度](#scheduling)
{{</note>}}
配置上有一个想对应的 handler 的名称， 被 `RuntimeClass` 所引用。 这个字段的命名必须符合 DNS-1123 标签的标准(字母，数字，中划线(`-`)组成)

2. 创建相应的 `RuntimeClass` 资源

上一步提到的每个配置都有对应的 `handler` 名称，用于区分不同的配置。 每一个 `handler` 都会创建一个对就的 `RuntimeClass` 对象。
`RuntimeClass` 资源目前只有两个有意义的字段： 名称 (`metadata.name`)和处理器 (`handler`), 对象定义类似如下:
```yaml
apiVersion: node.k8s.io/v1beta1  # RuntimeClass 定义在 node.k8s.io API 组中
kind: RuntimeClass
metadata:
  name: myclass  # The name the RuntimeClass will be referenced by
  # RuntimeClass is a non-namespaced resource
handler: myconfiguration  # The name of the corresponding CRI configuration
```
`RuntimeClass` 对象命名必须是一个有效的 [DNS 子域名格式](../../00-overview/03-working-with-objects/01-names/#dns-subdomain-names)
{{<note>}}
建议只有集群管理员对 `RuntimeClass` 对象有写权限(create/update/patch/delete). 一般情况这是默认配置。 更多信息见 [授权概览](../../../reference/03-access-authn-authz/07-authorization/)
{{</note>}}

## 使用说明

当在集群中配置好了 `RuntimeClasses`， 使用就是一件简单的事情，只需要在 Pod 定义中添加 `runtimeClassName`就行， 例：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  runtimeClassName: myclass
  # ...
```
这个配置会让 kublet 使用对应名称的 `RuntimeClass` 来运行这个 Pod， 如果没有叫这个名称的 `RuntimeClass`， 或者 CRI 不能运行这个名称对应的处理器，则这个进入 `Failed` [状态](../../03-workloads/00-pods/00-pod-lifecycle/#pod-phase) 。 错误信息在对应的[事件](../../../3-tasks/08-debug-application-cluster/00-debug-application-introspection/)对象中
如果没有 runtimeClassName 字段则使用默认的运行时处理器，与 `RuntimeClass` 功能特性关闭等同

### CRI 配置

更多关于 CRI 运行时配置见[这里](/k8sDocs/docs/setup/production-environment/)

#### dockershim

k8s 内置的 dockershim CRI 不支持运行时处理器

#### containerd

运行时处理器通过 containerd 的 `/etc/containerd/config.toml` 配置文件时行配置， 有效的处理器配置在运行时配置区:
```toml
[plugins.cri.containerd.runtimes.${HANDLER_NAME}]
```

更多 containerd 的配置见文档:  https://github.com/containerd/cri/blob/master/docs/config.md
#### CRI-O

运行时处理器通过 CRI-O 的 `/etc/crio/crio.conf` 配置文件时行配置，有效的处理器配置在 [crio.runtime table](https://github.com/cri-o/cri-o/blob/master/docs/crio.conf.5.md#crioruntime-table):
```
[crio.runtime.runtimes.${HANDLER_NAME}]
  runtime_path = "${PATH_TO_BINARY}"
```
更多 CRI-O 的配置见文档: https://raw.githubusercontent.com/cri-o/cri-o/9f11d1d/docs/crio.conf.5.md

## 调度

{{< feature-state for_k8s_version="v1.16" state="beta" >}}

k8s v1.16， RuntimeClass 通过 scheduling 字段实现了对异构系统集群的支持. 通过对这些字段的配置实现让带有对应 RuntimeClass Pod 被调度到受到支持的节点上。而要实现支持这个调度，需要开启 [RuntimeClass 准入控制器](../../../reference/03-access-authn-authz/04-admission-controllers/#runtimeclass)(k8s v1.16 默认开启)

为了保证 Pod 会落到 RuntimeClass 指定的节点上，需要要在节点上设置通用标签，让 `runtimeclass.scheduling.nodeSelector` 可以选择到对应的节点。RuntimeClass 的节点选择器与 Pod 的节点选择器的交集会作为最终的节点选择条件。 如果两都有冲突，则 Pod 会被拒绝。
{{< todo-optimize >}}
如果支持的节点上有防止其它 RuntimeClass 的 Pod 在其上运行的一毒点(Taint), 则需要在 RuntimeClass 上添加 {{< glossary_tooltip text="耐受(tolerations)" term_id="toleration" >}} 与 nodeSelector 相似， RuntimeClass 的耐受(tolerations) 会与 Pod 的 耐受(tolerations)的并集组成最终的节点{{< glossary_tooltip text="耐受(tolerations)" term_id="toleration" >}}

要了解更多关于节点选择与{{< glossary_tooltip text="耐受(tolerations)" term_id="toleration" >}}信息，见 [分配 Pod 到节点](/k8sDocs/docs/concepts/scheduling-eviction/assign-pod-node/)

## Pod Overhead

{{< feature-state for_k8s_version="v1.18" state="beta" >}}
用户可以在 Pod 上配置 `overhead` 进行资源限制。 声明 `overhead` 可以让集群(包括调度器)在调度 Pod 时计算合适的资源。
要使用 Pod `overhead` 需要打开功[能特性开关](../../../reference/command-line-tools-reference/feature-gates/)(默认开启) PodOverhead
Pod `overhead` 是通过 RuntimeClass 的 overhead 字段进行定义的， 用户可以利用 RuntimeClass 来设置正在运行的 Pod的 `overhead`， 并保证k8s 能够计算到这些 `overhead`
{{< todo-optimize >}}
## {{% heading "whatsnext" %}}

- [RuntimeClass Design](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/runtime-class.md)
- [RuntimeClass Scheduling Design](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/runtime-class-scheduling.md)
- Read about the [Pod Overhead](/docs/concepts/configuration/pod-overhead/) concept
- [PodOverhead Feature Design](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/20190226-pod-overhead.md)
