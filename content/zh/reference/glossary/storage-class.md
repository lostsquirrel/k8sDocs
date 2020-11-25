---
title: StorageClass
id: storageclass
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/storage/storage-classes
short_description: >
  StorageClass 为管理员提供了一种描述不同存储类别的方式

aka:
tags:
- core-object
- storage
---
<!--
 A StorageClass provides a way for administrators to describe different available storage types.
  -->
 `StorageClass` 为管理员提供了一种描述不同存储类别的方式
<!--more-->
<!--
StorageClasses can map to quality-of-service levels, backup policies, or to arbitrary policies determined by cluster administrators. Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a {{< glossary_tooltip text="Persistent Volume" term_id="persistent-volume" >}} belonging to the class needs to be dynamically provisioned. Users can request a particular class using the name of a StorageClass object.
 -->

`StorageClasse` 可以与 服务质量(QoS)级别，备份策略， 或其它由管理决定的任意策略。 每个
`StorageClasse` 都包含 `provisioner`, `parameters`, `reclaimPolicy` 字段，它们将在
有属于这个 `StorageClasse` 的
{{< glossary_tooltip text="Persistent Volume" term_id="persistent-volume" >}}
在动态管理时用到。 用户可以通过使用 `StorageClass` 名称指定特定类别。
