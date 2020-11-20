---
title: 用于已完成资源的 TTL 控制器
date: 2020-08-31
publishdate: 2020-08-31
weight: 2030106
---
<!--  
---
reviewers:
- janetkuo
title: TTL Controller for Finished Resources
content_type: concept
weight: 70
---
-->
<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

The TTL controller provides a TTL (time to live) mechanism to limit the lifetime of resource
objects that have finished execution. TTL controller only handles
{{< glossary_tooltip text="Jobs" term_id="job" >}} for now,
and may be expanded to handle other resources that will finish execution,
such as Pods and custom resources.

Alpha Disclaimer: this feature is currently alpha, and can be enabled with both kube-apiserver and kube-controller-manager
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
`TTLAfterFinished`.
 -->
{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

TTL 控制器提供了一个限制那些已经执行完成的资源对象生存期的 TTL (存活时间)机制。
目前 TTL 控制器只能控制 {{< glossary_tooltip  term_id="job" >}}， 可能会扩展到通解处理其它完成执行的资源
如 Pod 和 自定义资源。
Alpha Disclaimer: this feature is currently alpha, and can be enabled with both kube-apiserver and kube-controller-manager
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
`TTLAfterFinished`.
Alpha 版本的免责声明: 这个特性现在还是 alpha 版本， 可以在 kube-apiserver 中 kube-controller-manager
通过 `TTLAfterFinished` [功能阀](/docs/reference/command-line-tools-reference/feature-gates/) 开启
<!-- body -->
<!--
## TTL Controller

The TTL controller only supports Jobs for now. A cluster operator can use this feature to clean
up finished Jobs (either `Complete` or `Failed`) automatically by specifying the
`.spec.ttlSecondsAfterFinished` field of a Job, as in this
[example](/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically).
The TTL controller will assume that a resource is eligible to be cleaned up
TTL seconds after the resource has finished, in other words, when the TTL has expired. When the
TTL controller cleans up a resource, it will delete it cascadingly, that is to say it will delete
its dependent objects together with it. Note that when the resource is deleted,
its lifecycle guarantees, such as finalizers, will be honored.

The TTL seconds can be set at any time. Here are some examples for setting the
`.spec.ttlSecondsAfterFinished` field of a Job:

* Specify this field in the resource manifest, so that a Job can be cleaned up
  automatically some time after it finishes.
* Set this field of existing, already finished resources, to adopt this new
  feature.
* Use a
  [mutating admission webhook](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  to set this field dynamically at resource creation time. Cluster administrators can
  use this to enforce a TTL policy for finished resources.
* Use a
  [mutating admission webhook](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  to set this field dynamically after the resource has finished, and choose
  different TTL values based on resource status, labels, etc.
 -->
## TTL 控制器

目前 TTL 控制器只支持 Job。 集群管理员可以通过该特性来自动清理已完成的 Job(无论 `Complete` 或 `Failed`)，
只需要在 Job 对象上设置 `.spec.ttlSecondsAfterFinished`， 见 [示例](/k8sDocs/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically).
TTL 控制器假定一个资源在完成后 TTL 秒之后就是能够被回收的，换句话来说，就是当 TTL 过期的时候。
当 TTL 控制器清理一个资源时，会级联地删除，也就是说会连同它的从属对象一起删除。 要注意当一个资源被删除时，
它的生命周期保证，如析构器，就会被触发。
TTL 时间可以在任意时刻设置。 以下为在 Job 上设置 `.spec.ttlSecondsAfterFinished` 的一些示例:
- 在资源的配置清单中设置该字段，因而 Job 可以在完成后的某个时间点被自动清理。  
- 在已经存在，已经完成的资源上设置该字段，然后享受该特性。
- 使用
  [mutating admission webhook](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  来在资源创建时动态添加该字段。集群管理员可以使用它来给已经完成的资源加持一个 TTL 策略

- 使用
  [mutating admission webhook](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  在资源完成时动态添加该字段。 通过不同的资源状态，标签，等来设置不同的 TTL 值
<!--
## Caveat

### Updating TTL Seconds

Note that the TTL period, e.g. `.spec.ttlSecondsAfterFinished` field of Jobs,
can be modified after the resource is created or has finished. However, once the
Job becomes eligible to be deleted (when the TTL has expired), the system won't
guarantee that the Jobs will be kept, even if an update to extend the TTL
returns a successful API response.

### Time Skew

Because TTL controller uses timestamps stored in the Kubernetes resources to
determine whether the TTL has expired or not, this feature is sensitive to time
skew in the cluster, which may cause TTL controller to clean up resource objects
at the wrong time.

In Kubernetes, it's required to run NTP on all nodes
(see [#6159](https://github.com/kubernetes/kubernetes/issues/6159#issuecomment-93844058))
to avoid time skew. Clocks aren't always correct, but the difference should be
very small. Please be aware of this risk when setting a non-zero TTL.
 -->
## 附加说明

### 更新 TTL 时间

要注意 TTL 的时间， 例如 Job 的 `.spec.ttlSecondsAfterFinished` 字段可以在资源创建或完成后进行修改。
但是，当 Job 变得可以被删除时(当 TTL 过期后)， 系统就不保证 Job 对象会继续保留，即便增加 TTL 的请求 API 请求响应成功的。

### 时间偏差

因为 TTL 控制器使用存放于 k8s 资源中的时间戳来决定 TTL 是否已经过期， 这个特性对集群中的时间偏差很敏感。
可能会导致 TTL 控制器在错误的时间清理资源对象。

在 k8s 系统中，需要在所有的节点上运行 NTP (见问题单 [#6159](https://github.com/kubernetes/kubernetes/issues/6159#issuecomment-93844058))
来避免引志时间偏差。 时钟不是始终正确的，但差异必须要特别小。 在设置非零 TTL 时一定要注意这个风险。

## {{% heading "whatsnext" %}}

* [自动清理 Job](/k8sDocs/docs/concepts/workloads/controllers/job/#clean-up-finished-jobs-automatically)

* [设置文稿](https://github.com/kubernetes/enhancements/blob/master/keps/sig-apps/0026-ttl-after-finish.md)
