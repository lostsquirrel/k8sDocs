---
title: 容器环境变量
id: container-env-variables
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/containers/container-environment/
short_description: >
  容器环境变量是 name=value 对，用来给 Pod 中运行的容器提供有用的信息
aka:
tags:
- fundamental
---
 容器环境变量是 name=value 对，用来给
 {{< glossary_tooltip term_id="pod" >}}
 中运行的容器提供有用的信息

<!--more-->

容器环境变量提供的信息，这些信息是运行容器化应用所需要的，并且还有其它对
{{< glossary_tooltip  term_id="container" >}}
重要的资源的信息。 例如， 文件系统明细，关于容器本身的信息，还其它的集群资源，如 Service Endpoint.
