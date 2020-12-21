---
title: 命名空间(namespace)
id: namespace
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/overview/working-with-objects/namespaces
short_description: >
  一个用于在同一个物理集群中支持多个虚拟集群的抽象概念
aka:
tags:
- fundamental
---
 一个用于在同一个物理
{{< glossary_tooltip text="cluster" term_id="cluster" >}}
中支持多个虚拟集群的抽象概念
<!--more-->

命名空间(namespace) 被用来组织集群中的对象并提供分隔集群资源的一种方式。 在一个命名空间中的
资源名称必须唯一，但在不同命名空间不受限制。
