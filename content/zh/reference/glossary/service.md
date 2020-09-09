---
title: Service
id: service
date: 2018-04-12
full_link: /k8sDocs/concepts/services-networking/service/
short_description: >
  以网络服务的方式让一个由一组 Pod 构成的应用可以对外提供服务的一种方式
aka:
tags:
- fundamental
- core-object
---
以网络服务的方式让一个由一组
{{< glossary_tooltip text="Pod" term_id="pod" >}}
组成的应用能够对外提供服务的一种抽象方式
<!--more-->

属于 Service 的 Pod (通常)是由
{{< glossary_tooltip term_id="selector" >}}
决定的。 如果有 Pod 加入或删除，匹配到选择器的 Pod 就会发生变化。 Service 保证流量只会流向
当前工作负载的 Pod 集合。
