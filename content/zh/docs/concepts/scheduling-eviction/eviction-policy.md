---
title: Eviction Policy
content_type: concept
weight: 60
---
<!--
---
title: Eviction Policy
content_type: concept
weight: 60
---
 -->
<!-- overview -->
<!--
This page is an overview of Kubernetes' policy for eviction.
 -->
本文简单介绍 k8s 驱逐策略

<!-- body -->
<!--
## Eviction Policy

The {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} proactively monitors for
and prevents total starvation of a compute resource. In those cases, the `kubelet` can reclaim
the starved resource by failing one or more Pods. When the `kubelet` fails
a Pod, it terminates all of its containers and transitions its `PodPhase` to `Failed`.
If the evicted Pod is managed by a Deployment, the Deployment creates another Pod
to be scheduled by Kubernetes.
 -->

## 驱逐策略 {#eviction-policy}

{{< glossary_tooltip term_id="kubelet" >}}
会主要监控并防止出现一个计算资源总体不足。 如果出现这种情况， `kubelet` 可以通过让一个
或多个 Pod 失效的方式回收不足的资源。 当 `kubelet` 使 Pod 失效时， 它就是将 Pod 内所有
的容器终止并将其 `PodPhase` 修改为  `Failed`. 如果被驱逐的 Pod 是被 Deployment 管理的，
则 Deployment 分创建另一个 Pod 并由 k8s 调度。

## {{% heading "whatsnext" %}}
<!--
- Learn how to [configure out of resource handling](/docs/tasks/administer-cluster/out-of-resource/) with eviction signals and thresholds.
 -->

- 从 [对资源不足处理的配置](/k8sDocs/docs/tasks/administer-cluster/out-of-resource/)
  中学习驱逐信号和阈值
