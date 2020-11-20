---
title: Pod 预设信息
date: 2020-07-28
publishdate: 2020-08-07
weight: 2030003
---
<!--
---
reviewers:
- jessfraz
title: Pod Presets
content_type: concept
weight: 50
---
 -->
<!-- overview -->
{{< feature-state for_k8s_version="v1.6" state="alpha" >}}

This page provides an overview of PodPresets, which are objects for injecting
certain information into pods at creation time. The information can include
secrets, volumes, volume mounts, and environment variables.

{{< feature-state for_k8s_version="v1.6" state="alpha" >}}

本文简单介绍 `PodPreset`， 一个用于在特定时间向 Pod 中注入特定信息的对象。
可注入的信息包括
{{<glossary_tooltip term_id="secret">}},
{{<glossary_tooltip term_id="volume">}},
{{<glossary_tooltip term_id="volume">}} 挂载,
和环境变量

<!-- body -->
<!--
## Understanding Pod presets

A PodPreset is an API resource for injecting additional runtime requirements
into a Pod at creation time.
You use [label selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors)
to specify the Pods to which a given PodPreset applies.

Using a PodPreset allows pod template authors to not have to explicitly provide
all information for every pod. This way, authors of pod templates consuming a
specific service do not need to know all the details about that service.
-->

## 理解 Pod 预设信息

PodPreset 是在Pod 创建时向其中注入的运行环境需要的额外信息的 API 对象资源。
用户可以通过 [label selectors](/k8sDocs/docs/concepts/overview/working-with-objects/labels/#label-selectors)
来指定哪些 PodPreset 要用到该 Pod 上面。

PodPreset 可以让 Pod 模板的创建者不必要为每个 Pod 提供都提供所有信息。
通过这种方式，Pod 模板的创建者在消费特定服务时，不需要知道该服务的所有细节

{{<todo-optimize>}}
<!--
## Enable PodPreset in your cluster {#enable-pod-preset}

In order to use Pod presets in your cluster you must ensure the following:

1. You have enabled the API type `settings.k8s.io/v1alpha1/podpreset`. For
   example, this can be done by including `settings.k8s.io/v1alpha1=true` in
   the `--runtime-config` option for the API server. In minikube add this flag
   `--extra-config=apiserver.runtime-config=settings.k8s.io/v1alpha1=true` while
   starting the cluster.
1. You have enabled the admission controller named `PodPreset`. One way to doing this
   is to include `PodPreset` in the `--enable-admission-plugins` option value specified
   for the API server. For example, if you use Minikube, add this flag:

   ```shell
   --extra-config=apiserver.enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodPreset
   ```

   while starting your cluster.
 -->
## 打开集群中的 `PodPreset`

为了能使用 Pod 预设信息，集群需要保证达到以下条件:

1. 需要启用 API 类型 `settings.k8s.io/v1alpha1/podpreset`. 具体操作是:
  在 api-server 中的 `--runtime-config` 选项中添加 `settings.k8s.io/v1alpha1=true`;
  对于 minikube， 需要在集群启动时添加
  `--extra-config=apiserver.runtime-config=settings.k8s.io/v1alpha1=true`

1. 需要启用一个叫 `PodPreset` 的准入控制器。
  一种方式是在 api-server `--enable-admission-plugins` 选择值中添加 `PodPreset`
  对于 minikube, 则在集群启动时添加以下参数:
  ```shell
   --extra-config=apiserver.enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodPreset
  ```
<!--
## How it works

Kubernetes provides an admission controller (`PodPreset`) which, when enabled,
applies Pod Presets to incoming pod creation requests.
When a pod creation request occurs, the system does the following:

1. Retrieve all `PodPresets` available for use.
1. Check if the label selectors of any `PodPreset` matches the labels on the
   pod being created.
1. Attempt to merge the various resources defined by the `PodPreset` into the
   Pod being created.
1. On error, throw an event documenting the merge error on the pod, and create
   the pod _without_ any injected resources from the `PodPreset`.
1. Annotate the resulting modified Pod spec to indicate that it has been
   modified by a `PodPreset`. The annotation is of the form
   `podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"`.

Each Pod can be matched by zero or more PodPresets; and each PodPreset can be
applied to zero or more Pods. When a PodPreset is applied to one or more
Pods, Kubernetes modifies the Pod Spec. For changes to `env`, `envFrom`, and
`volumeMounts`, Kubernetes modifies the container spec for all containers in
the Pod; for changes to `volumes`, Kubernetes modifies the Pod Spec.

{{< note >}}
A Pod Preset is capable of modifying the following fields in a Pod spec when appropriate:
- The `.spec.containers` field
- The `.spec.initContainers` field
{{< /note >}}
 -->
## 工作原理

k8s 提供了一个准入控制器(`PodPreset`), 当这个控制器打开时，就会向进入的 Pod 创建请求执行。
当一个 Pod 的创建请求发生时， 系统会做以下操作:

1. 取得所有可用的 `PodPresets`
1. 检查 `PodPresets` 标签选择器是否与新创建的 Pod 上的标签匹配
1. 尝试将 `PodPreset` 中定义的各种资源合并到新创建的 Pod
1. 如果出错， 抛出一个一个事件描述合并出错到 Pod 上， 并在 _不_ 注意任意 `PodPreset` 资源的情况下创建 Pod
1. 将由 `PodPreset` 修改的结果加入到注解备查。注解格式为 `podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"`
<!--
### Disable Pod Preset for a specific pod

There may be instances where you wish for a Pod to not be altered by any Pod
preset mutations. In these cases, you can add an annotation in the Pod's `.spec`
of the form: `podpreset.admission.kubernetes.io/exclude: "true"`.
 -->
### 在指定 Pod 禁用 `PodPreset`

针对某些实例，用户可能不希望其被 `PodPreset` 修改， 这种情况下， 用户可以在 Pod 定义上添加注解，
格式为 `podpreset.admission.kubernetes.io/exclude: "true"`.

## {{% heading "whatsnext" %}}
<!--
See [Injecting data into a Pod using PodPreset](/docs/tasks/inject-data-application/podpreset/)

For more information about the background, see the [design proposal for PodPreset](https://git.k8s.io/community/contributors/design-proposals/service-catalog/pod-preset.md).
 -->
实践 [使用 PodPreset 向 Pod 注入数据](/k8sDocs/tasks/inject-data-application/podpreset/)
更多背景信息， 见[design proposal for PodPreset](https://git.k8s.io/community/contributors/design-proposals/service-catalog/pod-preset.md)
