---
title: Job
id: job
date: 2018-04-12
full_link: /k8sDocs/concepts/workloads/controllers/job/
short_description: >
  一个运行后即结束的有限或批量任务
aka:
tags:
- fundamental
- core-object
- workload
---
 一个运行后即结束的有限或批量任务

<!--more-->

创建一个或多个 {{< glossary_tooltip term_id="pod" >}} 对象， 并保证指定数量的 Pod 最终成功完成并终止。 当这些 Pod 成功完成，Job 会跟踪这此 Pod 的成功完成的状态。
