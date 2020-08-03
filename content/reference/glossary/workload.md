---
title: 工作负载
id: workloads
date: 2019-02-13
full_link: /k8sDocs/2-concepts/03-workloads/
short_description: >
  一个工作负载就是一个运行在k8s上的应用

aka:
tags:
- fundamental
---
   一个工作负载就是一个运行在k8s上的应用

<!--more-->

Various core objects that represent different types or parts of a workload
include the DaemonSet, Deployment, Job, ReplicaSet, and StatefulSet objects.

For example, a workload that has a web server and a database might run the
database in one {{< glossary_tooltip term_id="StatefulSet" >}} and the web server
in a {{< glossary_tooltip term_id="Deployment" >}}.
