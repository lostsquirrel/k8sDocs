---
title: 容器生命周期钩子
date: 2020-07-27
publishdate: 2020-07-28
weight: 20203
---
<!-- overview -->
<!--
This page describes how kubelet managed Containers can use the Container lifecycle hook framework
to run code triggered by events during their management lifecycle.
-->
本文主要介绍如何使用容器生命周期钩子框架来实现由 kubelet 管理的容器通过在管理过程中的生命周期事件来触发代理运行。


<!-- body -->

## 概述
<!--
Analogous to many programming language frameworks that have component lifecycle hooks, such as Angular,
Kubernetes provides Containers with lifecycle hooks.
The hooks enable Containers to be aware of events in their management lifecycle
and run code implemented in a handler when the corresponding lifecycle hook is executed.
-->
与许多编程语言的框架的组件有生命周期钩子(如: `Angular`)一样， k8s 也为容器提供了生命周期钩子。
生命周期钩子可以是容器收到自身生命周期管理时发生的事件，当这些事件发生时就会触发对应的钩子调用执行处理器中的代码实现


## 容器钩子

<!--
There are two hooks that are exposed to Containers:

`PostStart`

This hook executes immediately after a container is created.
However, there is no guarantee that the hook will execute before the container ENTRYPOINT.
No parameters are passed to the handler.

`PreStop`

This hook is called immediately before a container is terminated due to an API request or management event such as liveness probe failure, preemption, resource contention and others. A call to the preStop hook fails if the container is already in terminated or completed state.
It is blocking, meaning it is synchronous,
so it must complete before the call to delete the container can be sent.
No parameters are passed to the handler.

A more detailed description of the termination behavior can be found in
[Termination of Pods](/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).

-->
容器钩子有以下两个：

- `PostStart`
  这个钩子在容器创建后马上执行。
  但是不能保证鑫子会在容器执行 `ENTRYPOINT` 之前执行。

  这个钩子不会向处理器传递任何参数

- `PreStop`
  在容器被终结，比如因为一个 API 请求或诸如存活探针失败，优先级较低，资源竞争等原因，会立即执行这个钩子。 如果容器已经为终止或完成状态则调用 `preStop` 钩子会失败。
  该钩子会造成阻塞，也就是说它是同步的，所以必须在钩子调用处理器执行完成后，才能发送删除容器指令。

  这个钩子不会向处理器传递任何参数

### 钩子处理器的实现方式

<!--
Containers can access a hook by implementing and registering a handler for that hook.
There are two types of hook handlers that can be implemented for Containers:

* Exec - Executes a specific command, such as `pre-stop.sh`, inside the cgroups and namespaces of the Container.
Resources consumed by the command are counted against the Container.
* HTTP - Executes an HTTP request against a specific endpoint on the Container.
-->

容器可以用过为钩子实现并注册处理器来达成钩子的使用，以下为容器可实现的两种类型的钩子处理器
* Exec - 在容器的 `cgroups` 和 `namespaces` 权限内执行指定的命令，例如 `pre-stop.sh`， 所需要的资源包含在容器申请的资源中
* HTTP - 向容器上的指定接口发送一个请求

### 钩子处理器的执行

<!--
When a Container lifecycle management hook is called,
the Kubernetes management system executes the handler in the Container registered for that hook. 

Hook handler calls are synchronous within the context of the Pod containing the Container.
This means that for a `PostStart` hook,
the Container ENTRYPOINT and hook fire asynchronously.
However, if the hook takes too long to run or hangs,
the Container cannot reach a `running` state.

The behavior is similar for a `PreStop` hook.
If the hook hangs during execution,
the Pod phase stays in a `Terminating` state and is killed after `terminationGracePeriodSeconds` of pod ends.
If a `PostStart` or `PreStop` hook fails,
it kills the Container.

Users should make their hook handlers as lightweight as possible.
There are cases, however, when long running commands make sense,
such as when saving state prior to stopping a Container.
-->
当一个容器的生命周期管理钩子被调用时， k8s 管理系统就会执行注册到那个钩子上的处理器。

钩子处理器调用在容器所在的 Pod 的上下文中是同步的。 也就是说对于一个 `PostStart` 钩子， 它与容器的 `ENTRYPOINT` 是异步执行的。
但是，如果钩子执行时间过长或挂起，容器便不会进入到 `running` 的状态。

`PreStop` 钩子的行为模式也是相似的。 如果钩子在执行过程中挂起，则 Pod 在终结前的 `terminationGracePeriodSeconds` 时间内都是 `Terminating` 状态。
如果 `PostStart` 或 `PreStop` 执行失败，会导致对应容器被杀死

用户应该尽可能地让钩子处理器越轻量越好。 但是也有些场景，运行长时间的钩子命令是有意义的，比如当保存状态比停止容器更重要时

### 钩子的送达可靠性

<!--
Hook delivery is intended to be *at least once*,
which means that a hook may be called multiple times for any given event,
such as for `PostStart` or `PreStop`.
It is up to the hook implementation to handle this correctly.

Generally, only single deliveries are made.
If, for example, an HTTP hook receiver is down and is unable to take traffic,
there is no attempt to resend.
In some rare cases, however, double delivery may occur.
For instance, if a kubelet restarts in the middle of sending a hook,
the hook might be resent after the kubelet comes back up.
-->

对于 `PostStart` 或 `PreStop` 钩子投递被定为 *至少一次*， 也就是说对于任意一个事件可能多次调用钩子。 这就需要实现的钩子处理器需要正确的应对这种情况。

通常情况下，只能投递一次。如果钩子所调用的 HTTP 目标接口不可用，这种情况下不会尝试重发。 在有些不常见的场景中，也可能发生两次投递的情况。到目前为止，如果 kubelet 在钩子发送过程中重启，这个钩子可能在 kubelet 启动后再次发送

### 如何调试钩子处理器

<!--
The logs for a Hook handler are not exposed in Pod events.
If a handler fails for some reason, it broadcasts an event.
For `PostStart`, this is the `FailedPostStartHook` event,
and for `PreStop`, this is the `FailedPreStopHook` event.
You can see these events by running `kubectl describe pod <pod_name>`.
Here is some example output of events from running this command:
-->
钩子处理器的日志不会输出到 Pod 的事件中。 如果一个处理器因为某些原因失败，会发送一个广播事件。 `PostStart` 是 `FailedPostStartHook`事件， `PreStop`是 `FailedPreStopHook`事件。 可以通过执行命令 `kubectl describe pod <pod_name>` 查看这些事件， 命令输出结果类似如下:
```
Events:
  FirstSeen  LastSeen  Count  From                                                   SubObjectPath          Type      Reason               Message
  ---------  --------  -----  ----                                                   -------------          --------  ------               -------
  1m         1m        1      {default-scheduler }                                                          Normal    Scheduled            Successfully assigned test-1730497541-cq1d2 to gke-test-cluster-default-pool-a07e5d30-siqd
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulling              pulling image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Created              Created container with docker id 5c6a256a2567; Security:[seccomp=unconfined]
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Pulled               Successfully pulled image "test:1.0"
  1m         1m        1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Started              Started container with docker id 5c6a256a2567
  38s        38s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 5c6a256a2567: PostStart handler: Error executing in Docker Container: 1
  37s        37s       1      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Normal    Killing              Killing container with docker id 8df9fdfd7054: PostStart handler: Error executing in Docker Container: 1
  38s        37s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}                         Warning   FailedSync           Error syncing pod, skipping: failed to "StartContainer" for "main" with RunContainerError: "PostStart handler: Error executing in Docker Container: 1"
  1m         22s       2      {kubelet gke-test-cluster-default-pool-a07e5d30-siqd}  spec.containers{main}  Warning   FailedPostStartHook
```



## {{% heading "whatsnext" %}}


* 实践指导
  [挂载处理器到容器生命周期事件](../../../3-tasks/02-configure-pod-container/16-attach-handler-lifecycle-event/).
