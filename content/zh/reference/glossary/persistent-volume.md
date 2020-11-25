---
title: 持久化卷(PV)
id: persistent-volume
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/storage/persistent-volumes/
short_description: >
  一个代表集群中一个存储资源的 API 对象。 存在形式为一个通用，可插拔的资源， 这个资源可以在
  任意 Pod 的生命周期外持久存在

aka:
tags:
- core-object
- storage
---
<!--
 An API object that represents a piece of storage in the cluster. Available as a general, pluggable resource that persists beyond the lifecycle of any individual {{< glossary_tooltip text="Pod" term_id="pod" >}}.
  -->
 一个代表集群中一个存储资源的 API 对象。 存在形式为一个通用，可插拔的资源， 这个资源可以在
 任意 {{< glossary_tooltip text="Pod" term_id="pod" >}} 的生命周期外持久存在
<!--more-->
<!--
PersistentVolumes (PVs) provide an API that abstracts details of how storage is provided from how it is consumed.
PVs are used directly in scenarios where storage can be created ahead of time (static provisioning).
For scenarios that require on-demand storage (dynamic provisioning), PersistentVolumeClaims (PVCs) are used instead.
 -->
 
持久化卷(PV) 提供了一个描述存储是怎么提供和消费细节的 API。PV 可以用于存储预先创建的场景(静态管理)
对于按需请求的存储(动态管理)，则要使用 PersistentVolumeClaim (PVC)
