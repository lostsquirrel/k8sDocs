---
title: 容器运行环境
id: container-runtime
date: 2019-06-05
full_link: /docs/setup/production-environment/container-runtimes
short_description: 容器运行环境就是负责运行容器的软件
aka:
tags:
- fundamental
- workload
---
 容器运行环境就是负责运行容器的软件

<!--more-->

Kubernetes supports several container runtimes: {{< glossary_tooltip term_id="docker">}},
{{< glossary_tooltip term_id="containerd" >}}, {{< glossary_tooltip term_id="cri-o" >}},
and any implementation of the [Kubernetes CRI (Container Runtime
Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).
