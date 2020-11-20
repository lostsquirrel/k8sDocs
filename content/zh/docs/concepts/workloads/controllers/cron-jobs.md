---
title: CronJob
date: 2020-09-01
publishdate: 2020-09-01
weight: 2030107
---
<!--
---
reviewers:
- erictune
- soltysh
- janetkuo
title: CronJob
content_type: concept
weight: 80
---
 -->
<!-- overview -->
<!--
{{< feature-state for_k8s_version="v1.8" state="beta" >}}

A _CronJob_ creates {{< glossary_tooltip term_id="job" text="Jobs" >}} on a repeating schedule.

One CronJob object is like one line of a _crontab_ (cron table) file. It runs a job periodically
on a given schedule, written in [Cron](https://en.wikipedia.org/wiki/Cron) format.

{{< caution >}}
All **CronJob** `schedule:` times are based on the timezone of the
{{< glossary_tooltip term_id="kube-controller-manager" text="kube-controller-manager" >}}.

If your control plane runs the kube-controller-manager in Pods or bare
containers, the timezone set for the kube-controller-manager container determines the timezone
that the cron job controller uses.
{{< /caution >}}

When creating the manifest for a CronJob resource, make sure the name you provide
is a valid [DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
The name must be no longer than 52 characters. This is because the CronJob controller will automatically
append 11 characters to the job name provided and there is a constraint that the
maximum length of a Job name is no more than 63 characters.
 -->
{{< feature-state for_k8s_version="v1.8" state="beta" >}}

_CronJob_ 可以定时重复地创建 {{< glossary_tooltip term_id="job" >}}

一个 CronJob 对象看起来像是 _crontab_ (cron table) 文件的一行。
它会按照以 [Cron](https://en.wikipedia.org/wiki/Cron) 编写的计划定期的运行 Job

{{< caution >}}
所有 **CronJob** `时间表:`的时间都是基于
{{< glossary_tooltip term_id="kube-controller-manager" text="kube-controller-manager" >}}
的时间的。

如果控制中心把 kube-controller-manager 运行在 Pod 中或裸容器中， kube-controller-manager 所在
容器的时区就决定了 CronJob 控制器所使用的时区
{{< /caution >}}

在创建 CronJob 资源的配置定义时，要确保其名称是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
名称不能多于 52 个字符。 因为 CronJob 会自动在它创建的 Job 的名称是自身的名称再加 11 个字符，
这样 Job 的名称就不会超过 63 个字符的限制。

<!-- body -->
<!--
## CronJob

CronJobs are useful for creating periodic and recurring tasks, like running backups or
sending emails. CronJobs can also schedule individual tasks for a specific time, such as
scheduling a Job for when your cluster is likely to be idle.
-->

## CronJob

CronJob 对于创建定期重复的任务是很有用的，例如运行备份任务或发送邮件。
CronJob 也可以在指定时间调度单个应用，例如当集群变得空闲时调度 Job 任务
<!--
### Example

This example CronJob manifest prints the current time and a hello message every minute:

{{< codenew file="application/job/cronjob.yaml" >}}

([Running Automated Tasks with a CronJob](/docs/tasks/job/automated-tasks-with-cron-jobs/)
takes you through this example in more detail).
 -->
### 示例

This example CronJob manifest prints the current time and a hello message every minute:
这个示例中 CronJob 定义的任务是每分钟打印当前时间和一个问候信息:

{{< codenew file="application/job/cronjob.yaml" >}}

([使用 CronJob 运行自动化任务](/k8sDocs/tasks/job/automated-tasks-with-cron-jobs/)
中的示例有更多详细的说明).
<!--
## CronJob limitations {#cron-job-limitations}

A cron job creates a job object _about_ once per execution time of its schedule. We say "about" because there
are certain circumstances where two jobs might be created, or no job might be created. We attempt to make these rare,
but do not completely prevent them. Therefore, jobs should be _idempotent_.

If `startingDeadlineSeconds` is set to a large value or left unset (the default)
and if `concurrencyPolicy` is set to `Allow`, the jobs will always run
at least once.

For every CronJob, the CronJob {{< glossary_tooltip term_id="controller" >}} checks how many schedules it missed in the duration from its last scheduled time until now. If there are more than 100 missed schedules, then it does not start the job and logs the error

````
Cannot determine if job needs to be started. Too many missed start time (> 100). Set or decrease .spec.startingDeadlineSeconds or check clock skew.
````

It is important to note that if the `startingDeadlineSeconds` field is set (not `nil`), the controller counts how many missed jobs occurred from the value of `startingDeadlineSeconds` until now rather than from the last scheduled time until now. For example, if `startingDeadlineSeconds` is `200`, the controller counts how many missed jobs occurred in the last 200 seconds.

A CronJob is counted as missed if it has failed to be created at its scheduled time. For example, If `concurrencyPolicy` is set to `Forbid` and a CronJob was attempted to be scheduled when there was a previous schedule still running, then it would count as missed.

For example, suppose a CronJob is set to schedule a new Job every one minute beginning at `08:30:00`, and its
`startingDeadlineSeconds` field is not set. If the CronJob controller happens to
be down from `08:29:00` to `10:21:00`, the job will not start as the number of missed jobs which missed their schedule is greater than 100.

To illustrate this concept further, suppose a CronJob is set to schedule a new Job every one minute beginning at `08:30:00`, and its
`startingDeadlineSeconds` is set to 200 seconds. If the CronJob controller happens to
be down for the same period as the previous example (`08:29:00` to `10:21:00`,) the Job will still start at 10:22:00. This happens as the controller now checks how many missed schedules happened in the last 200 seconds (ie, 3 missed schedules), rather than from the last scheduled time until now.

The CronJob is only responsible for creating Jobs that match its schedule, and
the Job in turn is responsible for the management of the Pods it represents.
 -->
## CronJob 局限 {#cron-job-limitations}

CronJob 按照计划在每个执行时刻创建 _大约_ 一个 Job 对象。 这里说 "大约" 是因为在某些特定的情况下可能会创建两个，或者一个都没有创建。
我们尽量避免这些情况的发生，但是不能实现完全不发生。 因此 Job 应该是 _幂等_ 的。

如果 `startingDeadlineSeconds` 设置得足够大或不设置(默认)并且 如果 `concurrencyPolicy`  设置为 `Allow`
这时候 Job 始终至少能运行一次

如果错过的调度次数大于 100， 则不再启动这个 Job 并向日志输出如下错误信息。
````
Cannot determine if job needs to be started. Too many missed start time (> 100). Set or decrease .spec.startingDeadlineSeconds or check clock skew.
````

一个重要的信息是如果 `startingDeadlineSeconds` 字段被设置(不是 `nil`)，控制器在计数错过的 Job 的时间是距离当前 `startingDeadlineSeconds`和时间而不是上次调度到当前的时间。
例如， 如果 `startingDeadlineSeconds` 的值为 `200`， 控制器会计数过去 200 秒内错过的次数。

CronJob 在调度时间创建失败被认为是一个错过计数。 例如， 如果 `concurrencyPolicy` 设置为 `Forbid`
并且 CronJob 在之前的调度还没有执行完的时间尝试新的调度，就会被认为是错过。

例如， 假定，一个 CronJob 设置为 从 `08:30:00` 开始，每分钟调度一个新的 Job，并且它的
`startingDeadlineSeconds` 字段没有设置。 如果 CronJob 控制器恰好在 `08:29:00` 到 `10:21:00` 挂了，
Job 就会因为没有成功启动而引起没有成功调度的次数就会大于 100.
再假设， 如果一个 CronJob 设置为 从 `08:30:00` 开始，每分钟调度一个新的 Job，
`startingDeadlineSeconds` 设置为 200 秒。 CronJob 控制器挂掉的时间也是一样 (`08:29:00` 到 `10:21:00`)，
Job 仍然会在 `10:22:00` 启动， 这是因为控制器计算的是过去 200 秒(也就是 3 个错过的调度)发生错过的调度数，而不是从最后一个调度时间到当前的时间。

CronJob 只负责创建符合它时间表的 Job，然后 Job 负责管理它代表的 Pod。

## {{% heading "whatsnext" %}}

[Cron 表达式格式](https://en.wikipedia.org/wiki/Cron)
CronJob `schedule` 字段的详细文档.

更多 CronJob 创建和管理，以及示例，见  [使用 CronJob 运行自动化任务](/k8sDocs/tasks/job/automated-tasks-with-cron-jobs).
