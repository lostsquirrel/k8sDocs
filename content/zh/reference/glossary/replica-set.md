---
title: ReplicaSet
id: replica-set
date: 2018-04-12
full_link: /k8sDocs/concepts/workloads/controllers/replicaset/
short_description: >
 ReplicaSet 保证在任意时间点上某个 Pod 有指定数量副本在运行
aka:
tags:
- fundamental
- core-object
- workload
---
 ReplicaSet 旨在在任意时间点上维持一组 Pod 副本的正常运行

 <!--
 ---
 title: ReplicaSet
 id: replica-set
 date: 2018-04-12
 full_link: /docs/concepts/workloads/controllers/replicaset/
 short_description: >
  ReplicaSet ensures that a specified number of Pod replicas are running at one time

 aka:
 tags:
 - fundamental
 - core-object
 - workload
 ---
  A ReplicaSet (aims to) maintain a set of replica Pods running at any given time.
  -->
<!--more-->

Workload objects such as {{< glossary_tooltip term_id="deployment" >}} make use of ReplicaSets
to ensure that the configured number of {{< glossary_tooltip term_id="pod" text="Pods" >}} are
running in your cluster, based on the spec of that ReplicaSet.
