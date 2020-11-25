---
title: PersistentVolumeClaim
id: persistent-volume-claim
date: 2018-04-12
full_link: /docs/concepts/storage/persistent-volumes/
short_description: >
  索要定义在 PersistentVolume 中的存储资源，然后就可以将其以卷的方式挂载到容器中。
aka:
tags:
- core-object
- storage
---
<!--
 Claims storage resources defined in a {{< glossary_tooltip text="PersistentVolume" term_id="persistent-volume" >}} so that it can be mounted as a volume in a {{< glossary_tooltip text="container" term_id="container" >}}.
 -->

 索要定义在
 {{< glossary_tooltip text="PersistentVolume" term_id="persistent-volume" >}}
中的存储资源，然后就可以将其以卷的方式挂载到
{{< glossary_tooltip text="container" term_id="container" >}}
中。
<!--more-->
<!--
Specifies the amount of storage, how the storage will be accessed (read-only, read-write and/or exclusive) and how it is reclaimed (retained, recycled or deleted). Details of the storage itself are described in the PersistentVolume object.
 -->

指定存储的容量，存储提供什么访问权限(只读，读写)， 存储怎么回收 (retained, recycled or deleted)
存储本身的细节在 PersistentVolume 对象中。
