---
title: 卷(Volume)
id: volume
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/storage/volumes/
short_description: >
  一个可以被 Pod 中的容器访问的包含数据的目录
aka:
tags:
- core-object
- fundamental
---
 一个可以被
 {{< glossary_tooltip term_id="pod" >}}
 中的
 {{< glossary_tooltip term_id="container" >}}
 访问的包含数据的目录
<!--more-->

一个 k8s 卷(volume)与包含它的 Pod 存储一样久。 因此，一个卷通过比 Pod 中运行的所有容器都
活得久，并且卷(volume)中的数据也可以在容器重启间保留

更多信息见 [存储(Storage)](/docs/concepts/storage/)
