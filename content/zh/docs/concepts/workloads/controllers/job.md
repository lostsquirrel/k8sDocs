---
title: Job
date: 2020-08-26
publishdate: 2020-08-28
weight: 2030104
---
<!--
---
reviewers:
- erictune
- soltysh
title: Jobs
content_type: concept
feature:
  title: Batch execution
  description: >
    In addition to services, Kubernetes can manage your batch and CI workloads, replacing containers that fail, if desired.
weight: 50
---
 -->
<!-- overview -->

一个 Job 会创建一个或多个 Pod 并保证这些 Pod 中指定数量的最终执行完成并终止。 当一个 Pod 成功完成时， Job 跟踪这些成功完成的状态。
当成功完成的数量达到指定的数量时， 这个任务(Job)就完成了。 删除一个 Job 会清除它所创建的 Pod。

使用 Pod 的一个简单场景为创建一个 Job 对象以保证一个 Pod 可靠地完成。 Job 对象会在第一个 Pod 挂掉或被删除(如因为节点硬件故障或节点重启)后创建一个新的 Pod。

用户也可以使用 Job 并行地运行多个 Pod。

<!-- body -->
<!--  
## Running an example Job

Here is an example Job config.  It computes π to 2000 places and prints it out.
It takes around 10s to complete.

{{< codenew file="controllers/job.yaml" >}}

You can run the example with this command:

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/job.yaml
```
```
job.batch/pi created
```

Check on the status of the Job with `kubectl`:

```shell
kubectl describe jobs/pi
```
```
Name:           pi
Namespace:      default
Selector:       controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
Labels:         controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
                job-name=pi
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"pi","namespace":"default"},"spec":{"backoffLimit":4,"template":...
Parallelism:    1
Completions:    1
Start Time:     Mon, 02 Dec 2019 15:20:11 +0200
Completed At:   Mon, 02 Dec 2019 15:21:16 +0200
Duration:       65s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
           job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  14m   job-controller  Created pod: pi-5rwd7
```

To view completed Pods of a Job, use `kubectl get pods`.

To list all the Pods that belong to a Job in a machine readable form, you can use a command like this:

```shell
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
echo $pods
```
```
pi-5rwd7
```

Here, the selector is the same as the selector for the Job.  The `--output=jsonpath` option specifies an expression
that just gets the name from each Pod in the returned list.

View the standard output of one of the pods:

```shell
kubectl logs $pods
```
The output is similar to this:
```shell
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```
-->
## 运行一个示例 Job

以下为一个示例 Job 的配置。 它会计算圆周率(π)后 2000 位并打印出来. 大概用时为 10 秒。

{{< codenew file="controllers/job.yaml" >}}

通过以下命令运行该示例:

```shell
kubectl apply -f job.yaml
```
```
job.batch/pi created
```

通过 `kubectl` 命令查看 Job 状态:

```shell
kubectl describe jobs/pi
```
```
Name:           pi
Namespace:      default
Selector:       controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
Labels:         controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
                job-name=pi
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"pi","namespace":"default"},"spec":{"backoffLimit":4,"template":...
Parallelism:    1
Completions:    1
Start Time:     Mon, 02 Dec 2019 15:20:11 +0200
Completed At:   Mon, 02 Dec 2019 15:21:16 +0200
Duration:       65s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=c9948307-e56d-4b5d-8302-ae2d7b7da67c
           job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  14m   job-controller  Created pod: pi-5rwd7
```

用过命令 `kubectl get pods` 查看 Job 中执行完成的 Pod。
以机器可读的模式列举属于 Job 的所有 Pod， 执行以下命令:
```shell
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')
echo $pods
```
```
pi-5rwd7
```

命令里面的选择器与 Job 的选择器是一样的。 `--output=jsonpath` 选项中的表达式用于指定返回列表中只有 Pod 的名称

查看 Pod 的标准输出:

```shell
kubectl logs $pods
```
输出内容类似如下:
```shell
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```
<!--
## Writing a Job spec

As with all other Kubernetes config, a Job needs `apiVersion`, `kind`, and `metadata` fields.
Its name must be a valid [DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A Job also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
 -->
## 编写 Job 配置

与其它所有 k8s 配置一样， Job 的必要字段有 `apiVersion`, `kind`, `metadata`。
它的名字必须是一个有效的 [DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

Job 还是需要有一个 [`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) 配置区域

<!--
### Pod Template

The `.spec.template` is the only required field of the `.spec`.


The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a {{< glossary_tooltip text="Pod" term_id="pod" >}}, except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a pod template in a Job must specify appropriate
labels (see [pod selector](#pod-selector)) and an appropriate restart policy.

Only a [`RestartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Never` or `OnFailure` is allowed.
 -->

### Pod Template

`.spec.template` 是 `.spec` 唯一必要字段。
`.spec.template` 是一个 [Pod 模板](/k8sDocs/docs/concepts/workloads/pods/#pod-templates)
除了因为嵌套没有 `apiVersion` 或 `kind` 字段外， 与 {{< glossary_tooltip text="Pod" term_id="pod" >}} 的配置格式完全一样。

相对于 Pod 新增的字段是 Job 中的 Pod 模板需要有恰当的标签 (见 [Pod 选择器](#pod-selector)) 和重启策略。

[`RestartPolicy`](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 的值只能是 `Never` 或 `OnFailure`
<!--  
### Pod selector

The `.spec.selector` field is optional.  In almost all cases you should not specify it.
See section [specifying your own pod selector](#specifying-your-own-pod-selector).
-->

### Pod 选择器 {#pod-selector}

`.spec.selector` 字段为可选，几乎所有的情景中用户都不应该指定。见 [设置自己的 Pod 选择器](#specifying-your-own-pod-selector)
<!--
### Parallel execution for Jobs

There are three main types of task suitable to run as a Job:

1. Non-parallel Jobs
   - normally, only one Pod is started, unless the Pod fails.
   - the Job is complete as soon as its Pod terminates successfully.
1. Parallel Jobs with a *fixed completion count*:
   - specify a non-zero positive value for `.spec.completions`.
   - the Job represents the overall task, and is complete when there is one successful Pod for each value in the range 1 to `.spec.completions`.
   - **not implemented yet:** Each Pod is passed a different index in the range 1 to `.spec.completions`.
1. Parallel Jobs with a *work queue*:
   - do not specify `.spec.completions`, default to `.spec.parallelism`.
   - the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
   - each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
   - when _any_ Pod from the Job terminates with success, no new Pods are created.
   - once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
   - once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output.  They should all be in the process of exiting.

For a _non-parallel_ Job, you can leave both `.spec.completions` and `.spec.parallelism` unset.  When both are
unset, both are defaulted to 1.

For a _fixed completion count_ Job, you should set `.spec.completions` to the number of completions needed.
You can set `.spec.parallelism`, or leave it unset and it will default to 1.

For a _work queue_ Job, you must leave `.spec.completions` unset, and set `.spec.parallelism` to
a non-negative integer.

For more information about how to make use of the different types of job, see the [job patterns](#job-patterns) section.
 -->
### 并行执行的 {#parallel-jobs}

以下为三种适合用 Job 跑的任务:

1. 非并行 Job
  - 通常除非挂掉否则只启动一个 Pod
  - 当 Pod 以成功状态终结则 Job 也同时完成

2. 带 *固定完成数* 的并行 Job
  - `.spec.completions` 值是一个非零正数
  - 这个 Job 代表任务的总包，每个 Pod 执行成功则在完成则相当于领到 1 到 `.spec.completions` 之间的一个号
  - **还没有实现的功能:** 每个 Pod 完成可以传递邮不是 1 到 `.spec.completions` 之间的索引
3. 带 *工作队列* 的并行 Job
  - 不要设置 `.spec.completions`， 默认使用 `.spec.parallelism`
  - 这些 Pod 需要相互协作或由外部服务来决定每个应该做什么。 比如一个 Pod 可以从工作队列中获取一批 N 个条目。
  - 每个 Pod 都能独立决定它的协作者是否干完，如果都干完了整个 Job 就完成。
  - 当 Job 中的 _任意_ Pod 成功完成并终止，就不会再创建新的 Pod。
  - 当至少有一个 Pod 成功完成并终止且所以 Pod 被终止，则 Job 成功完成
  - 当任意 Pod 成功完成并退出， 其它的 Pod 就不能再干任何工作或写任何输出。它们都应该在退出的过程中。

对于 _非并行_ Job， 可以不设置 `.spec.completions` 和 `.spec.parallelism`， 它们默认都是 1

对于 *固定完成数* 的并行 Job， 用户应该通过设置 `.spec.completions` 来指定需要完成的数量。
可以设置 `.spec.parallelism`，也可以不设置，让它使用默认的 1

对于 *工作队列* 的并行 Job， 用户必须不要设置 `.spec.completions`， 并将  `.spec.parallelism` 设置为非负数(0怎么说？)

For more information about how to make use of the different types of job, see the [job patterns](#job-patterns) section.

更多关于怎么用不同类型的 Job 的信息见 [Job 模式](#job-patterns) 小节
<!--
#### Controlling parallelism

The requested parallelism (`.spec.parallelism`) can be set to any non-negative value.
If it is unspecified, it defaults to 1.
If it is specified as 0, then the Job is effectively paused until it is increased.

Actual parallelism (number of pods running at any instant) may be more or less than requested
parallelism, for a variety of reasons:

- For _fixed completion count_ Jobs, the actual number of pods running in parallel will not exceed the number of
  remaining completions.   Higher values of `.spec.parallelism` are effectively ignored.
- For _work queue_ Jobs, no new Pods are started after any Pod has succeeded -- remaining Pods are allowed to complete, however.
- If the Job {{< glossary_tooltip term_id="controller" >}} has not had time to react.
- If the Job controller failed to create Pods for any reason (lack of `ResourceQuota`, lack of permission, etc.),
  then there may be fewer pods than requested.
- The Job controller may throttle new Pod creation due to excessive previous pod failures in the same Job.
- When a Pod is gracefully shut down, it takes time to stop.
 -->

#### 并行控制

申请并行数量 (`.spec.parallelism`) 可以被设置为任意非零的数。
如果没有设置则默认为 1.
如果设置为 0， 则这个 Job 实际上是被暂停，想要恢复需要改大这个值。

实际运行数量(任意时刻运行的 Pod 数量)可能会比请求的数量或多或少差一些， 可能的原因有:

- 对于 *固定完成数* Job， 实际并行运行的 Pod 数量不会超出未完成的数量。 设置更大的 `.spec.parallelism` 值实际是被忽略的。
- 对于 *工作队列* Job， 在任意 Pod 成功后就不会再创建新的 Pod -- 但是，剩下的 Pod 还是允许继续完成
- 如果 Job {{< glossary_tooltip term_id="controller" >}} 没有反应过来。
- 如果 Job 控制器因为某些原因(`ResourceQuota` 资源配额不足，权限不足，等)，可以会比请求数量要少。
- Job 控制器可能会因为同一个 Job 之前的 Pod 的故障情况来限制新 Pod 的创建。
- 当一个 Pod 是平滑关闭的时候，需要时间来停止。
<!--
## Handling Pod and container failures

A container in a Pod may fail for a number of reasons, such as because the process in it exited with
a non-zero exit code, or the container was killed for exceeding a memory limit, etc.  If this
happens, and the `.spec.template.spec.restartPolicy = "OnFailure"`, then the Pod stays
on the node, but the container is re-run.  Therefore, your program needs to handle the case when it is
restarted locally, or else specify `.spec.template.spec.restartPolicy = "Never"`.
See [pod lifecycle](/docs/concepts/workloads/pods/pod-lifecycle/#example-states) for more information on `restartPolicy`.

An entire Pod can also fail, for a number of reasons, such as when the pod is kicked off the node
(node is upgraded, rebooted, deleted, etc.), or if a container of the Pod fails and the
`.spec.template.spec.restartPolicy = "Never"`.  When a Pod fails, then the Job controller
starts a new Pod.  This means that your application needs to handle the case when it is restarted in a new
pod.  In particular, it needs to handle temporary files, locks, incomplete output and the like
caused by previous runs.

Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and
`.spec.template.spec.restartPolicy = "Never"`, the same program may
sometimes be started twice.

If you do specify `.spec.parallelism` and `.spec.completions` both greater than 1, then there may be
multiple pods running at once.  Therefore, your pods must also be tolerant of concurrency.
 -->
## 如何应对 Pod 和容器的故障

Pod 中的容器可能因为几种原因失败， 例如因为容器内的进程以非 0 的返回码退出， 或容器因为超出内存限制而退出，等。
如果发生了这些情况，并且 `.spec.template.spec.restartPolicy = "OnFailure"`， 这个 Pod 会留在原有的节点上，但容器会重启。
因此，用户的应用程序需要通过应对这种在本地的重新启动，或都设置 `.spec.template.spec.restartPolicy = "Never"`
更多关于重启策略(`restartPolicy`)的信息见 [Pod 生命周期](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#example-states)

整个 Pod 也可能整个挂掉，也可能有几种原因， 例如当 Pod 被从节点上踢出(节点在更新，重启，删除，等)， 或者 Pod 中的容器挂了而
`.spec.template.spec.restartPolicy = "Never"`。 当一个 Pod 挂掉，Job 控制器会重启一个新的 Pod。 也就是说用户的应用能够
处理这种在新 Pod 重启的情况。特别是要能够处理临时文件，锁，未完成的输出和与前一次运行相似的原因(这意思不太明白?)

还要注意即便设置了  `.spec.parallelism = 1` 和 `.spec.completions = 1` 且
`.spec.template.spec.restartPolicy = "Never"`， 同一个程序也有时候可能会启动两次。

如果用户设置 `.spec.parallelism` 和 `.spec.completions` 的值都大于 1，那么就可能同时启动并运行多个 Pod，
因此要保证这些 Pod 是能够处理并发的。
<!--
### Pod backoff failure policy

There are situations where you want to fail a Job after some amount of retries
due to a logical error in configuration etc.
To do so, set `.spec.backoffLimit` to specify the number of retries before
considering a Job as failed. The back-off limit is set by default to 6. Failed
Pods associated with the Job are recreated by the Job controller with an
exponential back-off delay (10s, 20s, 40s ...) capped at six minutes. The
back-off count is reset when a Job's Pod is deleted or successful without any
other Pods for the Job failing around that time.

{{< note >}}
If your job has `restartPolicy = "OnFailure"`, keep in mind that your container running the Job
will be terminated once the job backoff limit has been reached. This can make debugging the Job's executable more difficult. We suggest setting
`restartPolicy = "Never"` when debugging the Job or using a logging system to ensure output
from failed Jobs is not lost inadvertently.
{{< /note >}}
 -->
### Pod 失效补尝策略

在有些情况下，用户期望因为配置中的一些逻辑错误在进行一定次数的重试后，让一个 Job 失败。
要达到这个目的，将 `.spec.backoffLimit` 设置为认为失败前重试的次数。 这个补尝数默认为 6.
Job 相关的 Pod 在挂掉以后， Job 控制器会以指数延迟(10s, 20s, 40s ...)最长至六分钟来尝试重建 Pod.
补尝计数在 Job 的 Pod 被删除或该 Job 的其它 Pod 都没有出毛病时被重置

{{< note >}}
如果 Job 配置有 `restartPolicy = "OnFailure"`， 就要注意到这个 Job 中运行的容器会在达到补尝上限后终止。
这会使得调度 Job 内部的程序更麻烦。 我们建议在调度 Job 使用配置 `restartPolicy = "Never"` 或者使用日志系统
以保证失败的 Job 的日志不会无意间丢失。

{{< /note >}}
<!--
## Job termination and cleanup

When a Job completes, no more Pods are created, but the Pods are not deleted either.  Keeping them around
allows you to still view the logs of completed pods to check for errors, warnings, or other diagnostic output.
The job object also remains after it is completed so that you can view its status.  It is up to the user to delete
old jobs after noting their status.  Delete the job with `kubectl` (e.g. `kubectl delete jobs/pi` or `kubectl delete -f ./job.yaml`). When you delete the job using `kubectl`, all the pods it created are deleted too.

By default, a Job will run uninterrupted unless a Pod fails (`restartPolicy=Never`) or a Container exits in error (`restartPolicy=OnFailure`), at which point the Job defers to the
`.spec.backoffLimit` described above. Once `.spec.backoffLimit` has been reached the Job will be marked as failed and any running Pods will be terminated.

Another way to terminate a Job is by setting an active deadline.
Do this by setting the `.spec.activeDeadlineSeconds` field of the Job to a number of seconds.
The `activeDeadlineSeconds` applies to the duration of the job, no matter how many Pods are created.
Once a Job reaches `activeDeadlineSeconds`, all of its running Pods are terminated and the Job status will become `type: Failed` with `reason: DeadlineExceeded`.

Note that a Job's `.spec.activeDeadlineSeconds` takes precedence over its `.spec.backoffLimit`. Therefore, a Job that is retrying one or more failed Pods will not deploy additional Pods once it reaches the time limit specified by `activeDeadlineSeconds`, even if the `backoffLimit` is not yet reached.

Example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Note that both the Job spec and the [Pod template spec](/docs/concepts/workloads/pods/init-containers/#detailed-behavior) within the Job have an `activeDeadlineSeconds` field. Ensure that you set this field at the proper level.

Keep in mind that the `restartPolicy` applies to the Pod, and not to the Job itself: there is no automatic Job restart once the Job status is `type: Failed`.
That is, the Job termination mechanisms activated with `.spec.activeDeadlineSeconds` and `.spec.backoffLimit` result in a permanent Job failure that requires manual intervention to resolve.
 -->
## Job 终止与清理

当一个 Job 完成后，就不会再创建新的 Pod， 但现有的 Pod 也不会删除。 保留这个 Pod 的目的是为了让用户能够
在 Job 过成后还能够查看 Pod 的日志，以方便检查错误，警告或其它诊断输出。 Job 对象本身也会在完成后继续保底以方便用户能够查看它的状态。
由用户决定在查看状态后是否删除这些 Job。 可以通过 `kubectl` (如. `kubectl delete jobs/pi` 或 `kubectl delete -f ./job.yaml`) 删除 Job。
当用户使用 `kubectl` 删除 Job 后， 它所创建的所有的 Pod 也一起被删除了。

默认情况下， 一个 Job 是连续运行的， 但当一个 Pod 挂了(`restartPolicy=Never`) 或一个容器存在错误(`restartPolicy=OnFailure`)
在这些情况下 Job 就会进行上面介经的补尝 `.spec.backoffLimit` 。 当 `.spec.backoffLimit` 上限达到后， Job 就会被标记为失败，其它仍在运行的 Pod 也会被终止。

另一个终止 Job 的办法是活跃死线(active deadline), 通过 Job 的 `.spec.activeDeadlineSeconds` 设置秒数。
`activeDeadlineSeconds` 表示 Job 的持续时间， 无论创建了多少个 Pod。当 Job 达到 `activeDeadlineSeconds` 时，
它所以正在运行的 Pod都会被终止，Job 的状态会进入 `type: Failed`， `reason: DeadlineExceeded`。

注意 Job 的 `.spec.activeDeadlineSeconds` 优先级比它的 `.spec.backoffLimit` 高。 因此，如果 Job 在 Pod 失败后尝试一两次后
如果达到 `activeDeadlineSeconds` 设置的时间就会再部署额外的 Pod 了， 即便 `backoffLimit` 还没有达到。

示例:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

注意 Job 本身的配置和 Job 内的 [Pod 模板配置](/k8sDocs/docs/concepts/workloads/pods/init-containers/#detailed-behavior)
都有 `activeDeadlineSeconds` 字段， 要注意设置的是哪个。

还要注意重启策略(`restartPolicy`) 是应用在 Pod 上的，而不是在 Job 上面: 当 Job 状态是 `type: Failed` 时是没有怎么重启的。
也就是当 Job 被 `.spec.activeDeadlineSeconds` 和 `.spec.backoffLimit` 机制触发了之后， Job 就会进行永久的失败状态，需要人工介入才能解决。
<!--
## Clean up finished jobs automatically

Finished Jobs are usually no longer needed in the system. Keeping them around in
the system will put pressure on the API server. If the Jobs are managed directly
by a higher level controller, such as
[CronJobs](/docs/concepts/workloads/controllers/cron-jobs/), the Jobs can be
cleaned up by CronJobs based on the specified capacity-based cleanup policy.
 -->
## 自动清理已经完成的 Job

完成的 Job 通常不需要再留在系统中。 把它们留在系统中会增加 api-server 的压力。 如果 Job 是由
更高级的控制器，如 [CronJobs](/k8sDocs/docs/concepts/workloads/controllers/cron-jobs/),
Job 可以被 CronJob 基于指定容量的清理策略来清除。
<!--
### TTL mechanism for finished Jobs

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

Another way to clean up finished Jobs (either `Complete` or `Failed`)
automatically is to use a TTL mechanism provided by a
[TTL controller](/docs/concepts/workloads/controllers/ttlafterfinished/) for
finished resources, by specifying the `.spec.ttlSecondsAfterFinished` field of
the Job.

When the TTL controller cleans up the Job, it will delete the Job cascadingly,
i.e. delete its dependent objects, such as Pods, together with the Job. Note
that when the Job is deleted, its lifecycle guarantees, such as finalizers, will
be honored.

For example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

The Job `pi-with-ttl` will be eligible to be automatically deleted, `100`
seconds after it finishes.

If the field is set to `0`, the Job will be eligible to be automatically deleted
immediately after it finishes. If the field is unset, this Job won't be cleaned
up by the TTL controller after it finishes.

Note that this TTL mechanism is alpha, with feature gate `TTLAfterFinished`. For
more information, see the documentation for
[TTL controller](/docs/concepts/workloads/controllers/ttlafterfinished/) for
finished resources.
 -->
### 对于已完成的 Job 的 TTL 机制

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

另一种自动清理完成的 Job (无论是 完成(`Complete`) 或失败(`Failed`) 的)方式是使用 TTL 机制，
它由 [TTL 控制器](/docs/concepts/workloads/controllers/ttlafterfinished/)提供用于
清理完成的资源，通过在 Job 上配置 `.spec.ttlSecondsAfterFinished` 字段实现。

当 TTL 控制器清理 Job 时，它会级联地删除 Job, 例如， 它依赖的对象，如 Pod 会与 Job 一起删除
注意当 Job 被删除后，它的生命周期保证对象如解构器也会被触发。(猜的)
示例:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

这个叫 `pi-with-ttl` Job 当它结束 `100` 秒后被自动删除。

这个这个字段的值设置为 `0`， 则这个 Job 会在执行完成之后马上就被自动删除了。 如果没有设置这个字段
则这个 Job 在结束后不会被 TTL 控制器清除。

要注意 这个 TTL 机器还在 `alpha` 状态， 需要使用 `TTLAfterFinished` 功能阀启用。
更多相关信息见 [TTL 控制器](/k8sDocs/docs/concepts/workloads/controllers/ttlafterfinished/)
<!--
## Job patterns

The Job object can be used to support reliable parallel execution of Pods.  The Job object is not
designed to support closely-communicating parallel processes, as commonly found in scientific
computing.  It does support parallel processing of a set of independent but related *work items*.
These might be emails to be sent, frames to be rendered, files to be transcoded, ranges of keys in a
NoSQL database to scan, and so on.

In a complex system, there may be multiple different sets of work items.  Here we are just
considering one set of work items that the user wants to manage together &mdash; a *batch job*.

There are several different patterns for parallel computation, each with strengths and weaknesses.
The tradeoffs are:

- One Job object for each work item, vs. a single Job object for all work items.  The latter is
  better for large numbers of work items.  The former creates some overhead for the user and for the
  system to manage large numbers of Job objects.
- Number of pods created equals number of work items, vs. each Pod can process multiple work items.
  The former typically requires less modification to existing code and containers.  The latter
  is better for large numbers of work items, for similar reasons to the previous bullet.
- Several approaches use a work queue.  This requires running a queue service,
  and modifications to the existing program or container to make it use the work queue.
  Other approaches are easier to adapt to an existing containerised application.


The tradeoffs are summarized here, with columns 2 to 4 corresponding to the above tradeoffs.
The pattern names are also links to examples and more detailed description.

|                            Pattern                                   | Single Job object | Fewer pods than work items? | Use app unmodified? |  Works in Kube 1.1? |
| -------------------------------------------------------------------- |:-----------------:|:---------------------------:|:-------------------:|:-------------------:|
| [Job Template Expansion](/docs/tasks/job/parallel-processing-expansion/)            |                   |                             |          ✓          |          ✓          |
| [Queue with Pod Per Work Item](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |         ✓         |                             |      sometimes      |          ✓          |
| [Queue with Variable Pod Count](/docs/tasks/job/fine-parallel-processing-work-queue/)  |         ✓         |             ✓               |                     |          ✓          |
| Single Job with Static Work Assignment                               |         ✓         |                             |          ✓          |                     |

When you specify completions with `.spec.completions`, each Pod created by the Job controller
has an identical [`spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).  This means that
all pods for a task will have the same command line and the same
image, the same volumes, and (almost) the same environment variables.  These patterns
are different ways to arrange for pods to work on different things.

This table shows the required settings for `.spec.parallelism` and `.spec.completions` for each of the patterns.
Here, `W` is the number of work items.

|                             Pattern                                  | `.spec.completions` |  `.spec.parallelism` |
| -------------------------------------------------------------------- |:-------------------:|:--------------------:|
| [Job Template Expansion](/docs/tasks/job/parallel-processing-expansion/)           |          1          |     should be 1      |
| [Queue with Pod Per Work Item](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |          W          |        any           |
| [Queue with Variable Pod Count](/docs/tasks/job/fine-parallel-processing-work-queue/)  |          1          |        any           |
| Single Job with Static Work Assignment                               |          W          |        any           |
 -->
## Job 的形式

Job 对象可用于支持可靠的并行 Pod。 Job 对象在设计上不是用来支持紧密通信的并行处理，这种场景常见于科学计算。
Job 支持的是并行处理一组相互独立但有关系的 *工作内容*。 它们可能是发送电子邮件，需要渲染的帧，文件转码，NoSQL 数据库中需要被扫描的键，等等。

在一个复杂的系统中，可能会有多个不同的工作项集合。 这里我们只考虑一个用户想要一起管理的工作项集群 &mdash; 一个 *批量任务*，
对于并行计算有几种不同的模式，每一种有其优势和劣势。需要做出以下权衡:


- 一个 Job 对象对一个工作项， vs. 一个 Job 对象对所有的工作项。 后一种更适合于工作项比较多的情况。
  前一种对于用户来说有更多额外的创建工作和对系统来说要管理大量的 Job 对象。

- 创建 Pod 的数量与工作项的数量相同， vs. 每个 Pod 可以处理多个工作项。
  前者通常需要对现有代码和容器做出少量更改。后台适用于数量很大的工作项，与上一个条目的原因类似

- 以不同的方式来使用队列。 需要运行一个队列服务，还需要修改现有代码或容器使其能够使用工作队列。
  前面介绍的方式更容易用在已经存在的容器化应用上。

以上因素权衡总结如下， 包含上面提到的 4 种情况，分 2 列。
以下模式的名称有对象实例的连接地址，其中有更多详细说明。

|                            Pattern                                   | Single Job object | Fewer pods than work items? | Use app unmodified? |  Works in Kube 1.1? |
| -------------------------------------------------------------------- |:-----------------:|:---------------------------:|:-------------------:|:-------------------:|
| [Job Template Expansion](/k8sDocs/tasks/job/parallel-processing-expansion/)            |                   |                             |          ✓          |          ✓          |
| [有队列，每个工作项一个 Pod](/k8sDocs/tasks/job/coarse-parallel-processing-work-queue/)   |         ✓         |                             |      有时候      |          ✓          |
| [Queue with Variable Pod Count](/k8sDocs/tasks/job/fine-parallel-processing-work-queue/)  |         ✓         |             ✓               |                     |          ✓          |
| 带静态任务分配的单个 Job          |         ✓         |                             |          ✓          |                     |

当用户设置完成数(`.spec.completions`)时， Job 控制器创建的每一个 Pod 都拥有完全相同的 [`spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)。 也就是说这个任务所有的 Pod 都拥有相同的命令，相同的镜像，
相同的数据卷， (几乎)相同的环境变量。 这些模式是以不同的方式组织 Pod 以应对不同的工作内容。

以下表格展示每种模式所需要设置的 `.spec.parallelism` 和 `.spec.completions` 字段的值。
其中， `W` 是工作项的数量

|                             Pattern                                  | `.spec.completions` |  `.spec.parallelism` |
| -------------------------------------------------------------------- |:-------------------:|:--------------------:|
| [Job Template Expansion](/docs/tasks/job/parallel-processing-expansion/)           |          1          |     should be 1      |
| [有队列，每个工作项一个 Pod](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |          W          |        any           |
| [Queue with Variable Pod Count](/docs/tasks/job/fine-parallel-processing-work-queue/)  |          1          |        any           |
| 带静态任务分配的单个 Job                              |          W          |        any           |

<!--
## Advanced usage

### Specifying your own Pod selector

Normally, when you create a Job object, you do not specify `.spec.selector`.
The system defaulting logic adds this field when the Job is created.
It picks a selector value that will not overlap with any other jobs.

However, in some cases, you might need to override this automatically set selector.
To do this, you can specify the `.spec.selector` of the Job.

Be very careful when doing this.  If you specify a label selector which is not
unique to the pods of that Job, and which matches unrelated Pods, then pods of the unrelated
job may be deleted, or this Job may count other Pods as completing it, or one or both
Jobs may refuse to create Pods or run to completion.  If a non-unique selector is
chosen, then other controllers (e.g. ReplicationController) and their Pods may behave
in unpredictable ways too.  Kubernetes will not stop you from making a mistake when
specifying `.spec.selector`.

Here is an example of a case when you might want to use this feature.

Say Job `old` is already running.  You want existing Pods
to keep running, but you want the rest of the Pods it creates
to use a different pod template and for the Job to have a new name.
You cannot update the Job because these fields are not updatable.
Therefore, you delete Job `old` but _leave its pods
running_, using `kubectl delete jobs/old --cascade=false`.
Before deleting it, you make a note of what selector it uses:

```
kubectl get job old -o yaml
```
```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

Then you create a new Job with name `new` and you explicitly specify the same selector.
Since the existing Pods have label `controller-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`,
they are controlled by Job `new` as well.

You need to specify `manualSelector: true` in the new Job since you are not using
the selector that the system normally generates for you automatically.

```
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

The new Job itself will have a different uid from `a8f3d00d-c6d2-11e5-9f87-42010af00002`.  Setting
`manualSelector: true` tells the system to that you know what you are doing and to allow this
mismatch.
 -->
## 高级用法

### 设置自定义 Pod 选择器 {#specifying-your-own-pod-selector}

通常，当用户创始 Job 对象时，不需要配置 `.spec.selector`. 系统默认逻辑会在 Job 创建时添加该字段。
它会选择一个与其它 Job 对象不重叠的值设置在上面。

但是，有些时候，可能需要用户手动指定 `.spec.selector` 来覆盖掉这种默认设置的选择器。

这么做的时候要千万小心。 如果用户设置的选择器不止匹配到当前 Job 的 Pod，这些不相关的 Pod 可能就会被删除,
或者这个 Job 就会批别个 Job 完成的 Pod 认为是自己的，甚至其中一个或两个 Job 都不能创建 Pod 或 成功运行完成。
如果使用了一个非唯一的选择器，其它的控制器(如 ReplicationController) 和它们的 Pod 也可以出现不可预知的行为。
在设置 `.spec.selector` 时， k8s 不会阻止你去犯错误。

以下是一个用到该性场景的示例。

假设有一个已经在运行的 Job 名字叫 `old`。 用户想要让现有的 Pod 继续运行， 但余下的 Pod 需要以新的 Job 名称和新的 Pod 模板创建。
但是 Job 不能更新，因为这些字段不能更新。因此用户需要删除叫 `old` 的 Job, 但是要 _保留它在运行的 Pod_.
使用 `kubectl delete jobs/old --cascade=false` 命令删除 Job前，需要先看看它使用的选择器:

```
kubectl get job old -o yaml
```
```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

这时候创建一个叫 `new` 的新 Job 并显示的设置相同的选择器。
因为已经存在的 Pod 都有标签 `controller-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`，
所以它们会被 `new` Job 管理

要使用自定义的选择器而不是系统自动创建的，需要在新 Job 上配置 `manualSelector: true`
```
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

这个新的 Job 会拥有一个与 `a8f3d00d-c6d2-11e5-9f87-42010af00002` 不同的 UID。
设置 `manualSelector: true` 就是告诉系统你知道自己在做什么，这个不匹配是允许的。
<!--
## Alternatives

### Bare Pods

When the node that a Pod is running on reboots or fails, the pod is terminated
and will not be restarted.  However, a Job will create new Pods to replace terminated ones.
For this reason, we recommend that you use a Job rather than a bare Pod, even if your application
requires only a single Pod.

### Replication Controller

Jobs are complementary to [Replication Controllers](/docs/concepts/workloads/controllers/replicationcontroller/).
A Replication Controller manages Pods which are not expected to terminate (e.g. web servers), and a Job
manages Pods that are expected to terminate (e.g. batch tasks).

As discussed in [Pod Lifecycle](/docs/concepts/workloads/pods/pod-lifecycle/), `Job` is *only* appropriate
for pods with `RestartPolicy` equal to `OnFailure` or `Never`.
(Note: If `RestartPolicy` is not set, the default value is `Always`.)

### Single Job starts controller Pod

Another pattern is for a single Job to create a Pod which then creates other Pods, acting as a sort
of custom controller for those Pods.  This allows the most flexibility, but may be somewhat
complicated to get started with and offers less integration with Kubernetes.

One example of this pattern would be a Job which starts a Pod which runs a script that in turn
starts a Spark master controller (see [spark example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/spark/README.md)), runs a spark
driver, and then cleans up.

An advantage of this approach is that the overall process gets the completion guarantee of a Job
object, but maintains complete control over what Pods are created and how work is assigned to them.
 -->
## 替代方案

### 裸 Pod

当 Pod 所在的节点重启或挂掉时， Pod 就会终止并不会被重启。 但 Job 会创建新的 Job 来代替被终止掉的。
因为这个原因， 我们推荐用户使用 Job 而不是裸 Pod， 即使该应用只需要一个 Pod。

### `ReplicationController`

Job 与 [ReplicationController](/k8sDocs/docs/concepts/workloads/controllers/replicationcontroller/)是互补的.
ReplicationController 管理那些不期望被终止的 Pod (如 web 服务)，
Job 管理那些预期要终止的 Pod (如 批量任务)

就如在 [Pod 生命周期](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/)中讨论的，
`Job` 是唯一适合将 Pod 重启策略(`RestartPolicy`) 设置为 在失败时(`OnFailure`) 或 永不(`Never`) (重启)的。
{{<note>}}
如果 `RestartPolicy` 没有设置，则默认值为 总是(`Always`)重启
{{</note>}}

### 单个 Job 启动作为控制器的 Pod

另一种模式为单个 Job 创建一个 Pod， 这个 Pod 再创建其它的 Pod，这个 Pod 对于其它的 Pod 来说就表现为一个自定义控制器。
这种方式允许最大的灵活性， 但可能入门有点复杂并且与k8s 集成比较少。

使用这个模式的一个示例为一个 Job 创建了一个 Pod， 这个 Pod 内运行了一个脚本启动一个 Spark 主控制器
(见 [spark 示例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/spark/README.md))
运行一个 spark driver， 然后清理现场。

这种方式是的一个好处是由一个 Job 对象来保证所以进程的完成， 只需要维护完成控制不震怒近 哪些 Pod 要创建，怎么给它们分配工作
## `CronJob` {#cron-jobs}

用户可以使用 [`CronJob`](/docs/concepts/workloads/controllers/cron-jobs/) 来创建一个在指定 时间/日期 运行的 Job,
与 Unix 的 `cron` 类似。
