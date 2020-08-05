---
title: 控制器
id: controller
date: 2018-04-12
full_link: /k8sDocs/2-concepts/01-architecture/02-controller/
short_description: >
  一个控制回路，负责通过 apiserver 监控集群的共享状态，并尝试通过变更(某些对象)的方式实现集群从当前状态向期望状态迁移
aka:
tags:
- architecture
- fundamental
---
<!--
title: Controller
id: controller
date: 2018-04-12
full_link: /docs/concepts/architecture/controller/
short_description: >
  A control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state.
-->
<!--

In Kubernetes, controllers are control loops that watch the state of your
{{< glossary_tooltip term_id="cluster" text="cluster">}}, then make or request
changes where needed.
Each controller tries to move the current cluster state closer to the desired
state.
-->
在 k8s 中， 控制器就是监控{{< glossary_tooltip term_id="cluster">}}状态，并按需求实施或发送变量请求的控制回路。
每个控制器都在尝试将集群的当前状态迁移到期望状态
<!--more-->

Controllers watch the shared state of your cluster through the
{{< glossary_tooltip text="apiserver" term_id="kube-apiserver" >}} (part of the
{{< glossary_tooltip term_id="control-plane" >}}).

Some controllers also run inside the control plane, providing control loops that
are core to Kubernetes' operations. For example: the deployment controller, the
daemonset controller, the namespace controller, and the persistent volume
controller (and others) all run within the
{{< glossary_tooltip term_id="kube-controller-manager" >}}.
