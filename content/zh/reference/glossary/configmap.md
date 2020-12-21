---
title: ConfigMap
id: configmap
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/configuration/configmap/
short_description: >
  一个以键值对形式存储非敏感数据的 API 对象。 可以被用到环境变量，命令行参数或卷(volume)中的配置文件
aka:
tags:
- core-object
---

一个以键值对形式存储非敏感数据的 API 对象。
Pod 可以在环境变量，命令行参数或
{{< glossary_tooltip text="volume" term_id="volume" >}}
中的配置文件中使用它。
<!--more-->

ConfigMap 允许用户将环境相关的配置从
{{< glossary_tooltip term_id="image" >}}
中解耦出来，这样可以很容易地实现应用的移植。
