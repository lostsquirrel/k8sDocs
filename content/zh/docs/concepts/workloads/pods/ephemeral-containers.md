---
title: 临时容器
date: 2020-07-29
publishdate: 2020-08-12
weight: 2030005
---
<!--
---
reviewers:
- verb
- yujuhong
title: Ephemeral Containers
content_type: concept
weight: 80
---
 -->
<!-- overview -->
<!--
{{< feature-state state="alpha" for_k8s_version="v1.16" >}}

This page provides an overview of ephemeral containers: a special type of container
that runs temporarily in an existing {{< glossary_tooltip term_id="pod" >}} to
accomplish user-initiated actions such as troubleshooting. You use ephemeral
containers to inspect services rather than to build applications.

{{< warning >}}
Ephemeral containers are in early alpha state and are not suitable for production
clusters. In accordance with the [Kubernetes Deprecation Policy](
/docs/reference/using-api/deprecation-policy/), this alpha feature could change
significantly in the future or be removed entirely.
{{< /warning >}}

 -->
{{< feature-state state="alpha" for_k8s_version="v1.16" >}}
本文主要简述临时容器: 一个特殊类型的容器，用来实现如应用调试而临时运行在一个已经存在的 {{< glossary_tooltip term_id="pod" >}}
中的容器。用户应该仅使用它来调试服务而不是用来构建应用

{{< warning >}}
临时容器目前还是 alpha 状态， 不适合用于生产一并。 并且根据 [k8s 废弃策略](
/k8sDocs/reference/using-api/deprecation-policy/), 这些处于 alpha 版本特性在未来可能被
动大刀子或直接被干掉
{{< /warning >}}
<!-- body -->
<!--  
## Understanding ephemeral containers

{{< glossary_tooltip text="Pods" term_id="pod" >}} are the fundamental building
block of Kubernetes applications. Since Pods are intended to be disposable and
replaceable, you cannot add a container to a Pod once it has been created.
Instead, you usually delete and replace Pods in a controlled fashion using
{{< glossary_tooltip text="deployments" term_id="deployment" >}}.

Sometimes it's necessary to inspect the state of an existing Pod, however, for
example to troubleshoot a hard-to-reproduce bug. In these cases you can run
an ephemeral container in an existing Pod to inspect its state and run
arbitrary commands.
-->
## 临时容器是啥东西

{{< glossary_tooltip term_id="pod" >}} 是 k8s 应用构建的基石. 但在 Pod 的设计理念上它就是一个
一次性的可替换的组件，所以就不能在 Pod 创建后再往里面塞容器了。 如果要做变更通常就是通过
{{< glossary_tooltip term_id="deployment" >}} 这种删除替换的套路。

但是有时候又需要查看已经存在 Pod 内的某些状太，比如找一个很难重现的虫。 在这种情况下就可以在已经
存在的 Pod 里面塞一个临时容器然后在里面使劲折腾了。
<!--
### What is an ephemeral container?

Ephemeral containers differ from other containers in that they lack guarantees
for resources or execution, and they will never be automatically restarted, so
they are not appropriate for building applications.  Ephemeral containers are
described using the same `ContainerSpec` as regular containers, but many fields
are incompatible and disallowed for ephemeral containers.

- Ephemeral containers may not have ports, so fields such as `ports`,
  `livenessProbe`, `readinessProbe` are disallowed.
- Pod resource allocations are immutable, so setting `resources` is disallowed.
- For a complete list of allowed fields, see the [EphemeralContainer reference
  documentation](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ephemeralcontainer-v1-core).

Ephemeral containers are created using a special `ephemeralcontainers` handler
in the API rather than by adding them directly to `pod.spec`, so it's not
possible to add an ephemeral container using `kubectl edit`.

Like regular containers, you may not change or remove an ephemeral container
after you have added it to a Pod.
 -->
### 临时容器是什么样一个东西

临时容器与其它容器有点不同，它缺乏对资源与执行的保证，它永远不会被重启，所以它也不适合用来构建应用。
临时容器与其它容器一样也是通过 `ContainerSpec` 来定义， 但有许多字段在临时容器上是不可用的。

- 临时容器不能有端口，所以与端口相关的 `ports`, `livenessProbe`, `readinessProbe` 都是不能用的。
- Pod 分配的资源是不可变的，所以也不能设置 `resources`。
- 所以可用字段列表见 [EphemeralContainer 参考文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#ephemeralcontainer-v1-core)

临时容器是用一个在 API 叫 `ephemeralcontainers` 的特殊处理器创建的， 并不能通过直接添加 `pod.spec` 来创建。
所以也不能通过 `kubectl edit` 来向 Pod 中添加一个临时容器。

与普通容器一样，临时容器被加到 Pod 中之后就能再修改或删除。
<!--
## Uses for ephemeral containers

Ephemeral containers are useful for interactive troubleshooting when `kubectl
exec` is insufficient because a container has crashed or a container image
doesn't include debugging utilities.

In particular, [distroless images](https://github.com/GoogleContainerTools/distroless)
enable you to deploy minimal container images that reduce attack surface
and exposure to bugs and vulnerabilities. Since distroless images do not include a
shell or any debugging utilities, it's difficult to troubleshoot distroless
images using `kubectl exec` alone.

When using ephemeral containers, it's helpful to enable [process namespace sharing](/docs/tasks/configure-pod-container/share-process-namespace/) so
you can view processes in other containers.

See [Debugging with Ephemeral Debug Container](/docs/tasks/debug-application-cluster/debug-running-pod/#debugging-with-ephemeral-debug-container)
for examples of troubleshooting using ephemeral containers.
 -->
## 临时容器是怎么用的

临时容器在遇到需要交互式地找虫， 而 `kubectl exec` 又因为容器已经挂了或都容器镜像中没有调试工具时很有用的。

特别是 [distroless images](https://github.com/GoogleContainerTools/distroless)
允许用户能够部署最小化的容器镜像而达到减少攻击面，减少缺陷和漏洞的暴露。 但 `distroless` 镜像中是没
shell 或任何其它的调试工具的，所以对 `distroless` 镜像要只通过 `kubectl exec` 来调试就变得无比困难了。

在使用临时容器时，建议打开 [进程命名空间共享](/k8sDocs/tasks/configure-pod-container/share-process-namespace/)
这样就可以看到其它容器中的进程了。

更多使用临时容器调试的例子见 [使用临时容器找虫](/k8sDocs/tasks/debug-application-cluster/debug-running-pod/#debugging-with-ephemeral-debug-container)
<!--
## Ephemeral containers API

{{< note >}}
The examples in this section require the `EphemeralContainers` [feature
gate](/docs/reference/command-line-tools-reference/feature-gates/) to be
enabled, and Kubernetes client and server version v1.16 or later.
{{< /note >}}

The examples in this section demonstrate how ephemeral containers appear in
the API. You would normally use `kubectl alpha debug` or another `kubectl`
[plugin](/docs/tasks/extend-kubectl/kubectl-plugins/) to automate these steps
rather than invoking the API directly.

Ephemeral containers are created using the `ephemeralcontainers` subresource
of Pod, which can be demonstrated using `kubectl --raw`. First describe
the ephemeral container to add as an `EphemeralContainers` list:

```json
{
    "apiVersion": "v1",
    "kind": "EphemeralContainers",
    "metadata": {
            "name": "example-pod"
    },
    "ephemeralContainers": [{
        "command": [
            "sh"
        ],
        "image": "busybox",
        "imagePullPolicy": "IfNotPresent",
        "name": "debugger",
        "stdin": true,
        "tty": true,
        "terminationMessagePolicy": "File"
    }]
}
```

To update the ephemeral containers of the already running `example-pod`:

```shell
kubectl replace --raw /api/v1/namespaces/default/pods/example-pod/ephemeralcontainers  -f ec.json
```

This will return the new list of ephemeral containers:

```json
{
   "kind":"EphemeralContainers",
   "apiVersion":"v1",
   "metadata":{
      "name":"example-pod",
      "namespace":"default",
      "selfLink":"/api/v1/namespaces/default/pods/example-pod/ephemeralcontainers",
      "uid":"a14a6d9b-62f2-4119-9d8e-e2ed6bc3a47c",
      "resourceVersion":"15886",
      "creationTimestamp":"2019-08-29T06:41:42Z"
   },
   "ephemeralContainers":[
      {
         "name":"debugger",
         "image":"busybox",
         "command":[
            "sh"
         ],
         "resources":{

         },
         "terminationMessagePolicy":"File",
         "imagePullPolicy":"IfNotPresent",
         "stdin":true,
         "tty":true
      }
   ]
}
```

You can view the state of the newly created ephemeral container using `kubectl describe`:

```shell
kubectl describe pod example-pod
```

```
...
Ephemeral Containers:
  debugger:
    Container ID:  docker://cf81908f149e7e9213d3c3644eda55c72efaff67652a2685c1146f0ce151e80f
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:9f1003c480699be56815db0f8146ad2e22efea85129b5b5983d0e0fb52d9ab70
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
    State:          Running
      Started:      Thu, 29 Aug 2019 06:42:21 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
...
```

You can interact with the new ephemeral container in the same way as other
containers using `kubectl attach`, `kubectl exec`, and `kubectl logs`, for
example:

```shell
kubectl attach -it example-pod -c debugger
```
 -->
## 临时容器 API

{{< note >}}
以下示例需要打开 `EphemeralContainers` [功能阀](/k8sDocs/reference/command-line-tools-reference/feature-gates/)
k8s 客户端和服务端版本大于等于 `v1.16`
{{< /note >}}

本节示例通过演示彼把临时容器塞进 API 中的， 用户通常可以使用 `kubectl alpha debug`
或另一个 `kubectl` [插件](/k8sDocs/tasks/extend-kubectl/kubectl-plugins/) 来自动实现这些步骤而不需要直接调用 API

临时容器是使用 Pod 的子资源 `ephemeralcontainers` 来创建的。 使用 `kubectl --raw`
第一步将临时容器作为 `EphemeralContainers` 列表添加:

```json
{
    "apiVersion": "v1",
    "kind": "EphemeralContainers",
    "metadata": {
            "name": "example-pod"
    },
    "ephemeralContainers": [{
        "command": [
            "sh"
        ],
        "image": "busybox",
        "imagePullPolicy": "IfNotPresent",
        "name": "debugger",
        "stdin": true,
        "tty": true,
        "terminationMessagePolicy": "File"
    }]
}
```

而要更新已经在运行的 `example-pod` 中的临时容器：

```shell
kubectl replace --raw /api/v1/namespaces/default/pods/example-pod/ephemeralcontainers  -f ec.json
```

返回一个新的临时容器列表：

```json
{
   "kind":"EphemeralContainers",
   "apiVersion":"v1",
   "metadata":{
      "name":"example-pod",
      "namespace":"default",
      "selfLink":"/api/v1/namespaces/default/pods/example-pod/ephemeralcontainers",
      "uid":"a14a6d9b-62f2-4119-9d8e-e2ed6bc3a47c",
      "resourceVersion":"15886",
      "creationTimestamp":"2019-08-29T06:41:42Z"
   },
   "ephemeralContainers":[
      {
         "name":"debugger",
         "image":"busybox",
         "command":[
            "sh"
         ],
         "resources":{

         },
         "terminationMessagePolicy":"File",
         "imagePullPolicy":"IfNotPresent",
         "stdin":true,
         "tty":true
      }
   ]
}
```
可以通过 `kubectl describe` 查看新创建临时容器的状态:

```shell
kubectl describe pod example-pod
```

返回类似如下

```
...
Ephemeral Containers:
  debugger:
    Container ID:  docker://cf81908f149e7e9213d3c3644eda55c72efaff67652a2685c1146f0ce151e80f
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:9f1003c480699be56815db0f8146ad2e22efea85129b5b5983d0e0fb52d9ab70
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
    State:          Running
      Started:      Thu, 29 Aug 2019 06:42:21 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
...
```

用户也可以与普通容器一样用 `kubectl attach`, `kubectl exec`, `kubectl logs` 与临时容器交互。
例如：

```shell
kubectl attach -it example-pod -c debugger
```
