---
title: Operator pattern
id: operator-pattern
date: 2019-05-21
full_link: /docs/concepts/extend-kubernetes/operator/
short_description: >
  A specialized controller used to manage a custom resource
  一个用于管理自定义资源的专用控制器
aka:
tags:
- architecture
---

[operator pattern](/docs/concepts/extend-kubernetes/operator/) 是用来连接一个
{{< glossary_tooltip term_id="controller" >}}
到一个或多个自定义资源的系统设计。
<!--more-->

You can extend Kubernetes by adding controllers to your cluster, beyond the built-in
controllers that come as part of Kubernetes itself.
用户可以在 k8s 内置的的控制器外，可以通过添加控制器到集群中来扩展 k8s.
If a running application acts as a controller and has API access to carry out tasks
against a custom resource that's defined in the control plane, that's an example of
the Operator pattern.

如果一个运行的应用表现得像一个控制器，并对定义在控制面板中的自定义资源执行任务，这就是一个
操作模式(Operator pattern)的示例
