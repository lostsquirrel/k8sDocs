---
title: 容器存储接口 (CSI)
id: csi
date: 2018-06-25
full_link: /k8sDocs/docs/concepts/storage/volumes/#csi
short_description: >
     容器存储接口 (CSI)定义一个标准接口用于暴露存储系统到容器中

aka:
tags:
- storage
---
<!--
 The Container Storage Interface (CSI) defines a standard interface to expose storage systems to containers.
 -->
 容器存储接口 (CSI)定义一个标准接口用于暴露存储系统到容器中
<!--more-->
<!--
CSI allows vendors to create custom storage plugins for Kubernetes without adding them to the Kubernetes repository (out-of-tree plugins). To use a CSI driver from a storage provider, you must first [deploy it to your cluster](https://kubernetes-csi.github.io/docs/deploying.html). You will then be able to create a {{< glossary_tooltip text="Storage Class" term_id="storage-class" >}} that uses that CSI driver.
 -->
CSI 允许厂商在不将源代码添加到 k8s 代码库的情况下创建自定义的存储插件(out-of-tree 插件)。
要使用存储提供者的 CSI 驱动， 需要先将其 [部署到集群中](https://kubernetes-csi.github.io/docs/deploying.html)
这样就可以创建一个使用这个 CSI 驱动的
{{< glossary_tooltip text="Storage Class" term_id="storage-class" >}}

* [CSI 官方文档](/k8sDocs/docs/concepts/storage/volumes/#csi)
* [可用 CSI 驱动列表](https://kubernetes-csi.github.io/docs/drivers.html)
