---
title: Control Plane
id: control-plane
date: 2019-05-12
full_link:
short_description: >
  暴露那些定义，部署，管理容器生命周期的 API 和接口的容器编排层。

aka:
tags:
- fundamental
---
 暴露那些定义，部署，管理容器生命周期的 API 和接口的容器编排层。

 <!--more-->

 这一层由许多不同的组件构成，诸如(但不限于)

 * {{< glossary_tooltip term_id="etcd" >}}
 * {{< glossary_tooltip term_id="kube-apiserver" >}}
 * {{< glossary_tooltip term_id="kube-scheduler" >}}
 * {{< glossary_tooltip term_id="kube-controller-manager" >}}
 * {{< glossary_tooltip term_id="cloud-controller-manager" >}}

 这些组件可以以传统的操作系统服务(守护进程)或以容器的方式运行。 这些组件所运行的节点，按传统叫做
 {{< glossary_tooltip term_id="master" >}}.
