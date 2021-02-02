---
title: 调度与驱逐
weight: 1000
description: >
  在 k8s 中， 调度指的是搞清楚 Pod 是不是与节点匹配，当匹配了 kubelet 才能在这个节点运行这个 Pod。
  驱逐就是在资源不足的节点上主动挂掉一个或多个 Pod 的过程。
---
<!--
---
title: "Scheduling and Eviction"
weight: 90
description: >
  In Kubernetes, scheduling refers to making sure that Pods are matched to Nodes so that the kubelet can run them.
  Eviction is the process of proactively failing one or more Pods on resource-starved Nodes.
---
-->
