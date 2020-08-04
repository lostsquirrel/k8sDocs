---
title: 初始化容器
weight: 2030001
date: 2020-07-29
publishdate: 2020-08-04
---
<!--
---
reviewers:
- erictune
title: Init Containers
content_type: concept
weight: 40
---
-->

<!-- overview -->
<!--
This page provides an overview of init containers: specialized containers that run
before app containers in a {{< glossary_tooltip text="Pod" term_id="pod" >}}.
Init containers can contain utilities or setup scripts not present in an app image.

You can specify init containers in the Pod specification alongside the `containers`
array (which describes app containers).
  -->
本文主要介绍初始化容器: 一种在 Pod 中在应用容器启动之前运行的专用容器。
初始化容器用于提供应用镜像没有的工具或初始化脚本。
初始化容器定义与应用容器定义是相邻关系

<!-- body -->
<!--
## Understanding init containers

A {{< glossary_tooltip text="Pod" term_id="pod" >}} can have multiple containers
running apps within it, but it can also have one or more init containers, which are run
before the app containers are started.

Init containers are exactly like regular containers, except:

* Init containers always run to completion.
* Each init container must complete successfully before the next one starts.

If a Pod's init container fails, Kubernetes repeatedly restarts the Pod until the init container
succeeds. However, if the Pod has a `restartPolicy` of Never, Kubernetes does not restart the Pod.

To specify an init container for a Pod, add the `initContainers` field into
the Pod specification, as an array of objects of type
[Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core),
alongside the app `containers` array.
The status of the init containers is returned in `.status.initContainerStatuses`
field as an array of the container statuses (similar to the `.status.containerStatuses`
field).
 -->
## 理解初始化容器是做什么的

一个 {{< glossary_tooltip text="Pod" term_id="pod" >}} 中可以饮上多个应用容器，同时还可以
有一个或多个初始化容器，这些初始化容器会在应用容器之前运行并结束。

初始化容器与普通容器并无太多不同，除了以下：

- 初始化化容器运行在有限时间结束
- 只有在上一初始化容器运行结束后，下一个才能开始运行。

如果 Pod 的初始化容器挂了， k8s 会不停重启该 Pod 直至初始化容器成功运行完成。
但如果 Pod 的 `restartPolicy` 值为 `Never`， 则 k8s 不会重启该 Pod。

要为一个 Pod 配置初始化容器， 需要在配置中添加 `initContainers` 字段， 字段值为一个类型为
[Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
的对象数组， 与 `containers` 字段相邻。
初始化容器返回的状态存放于 `.status.initContainerStatuses` 字段，是一个与容器状态字段(`.status.containerStatuses`)类似的数组
<!--
### Differences from regular containers

Init containers support all the fields and features of app containers,
including resource limits, volumes, and security settings. However, the
resource requests and limits for an init container are handled differently,
as documented in [Resources](#resources).

Also, init containers do not support `lifecycle`, `livenessProbe`, `readinessProbe`, or
`startupProbe` because they must run to completion before the Pod can be ready.

If you specify multiple init containers for a Pod, Kubelet runs each init
container sequentially. Each init container must succeed before the next can run.
When all of the init containers have run to completion, Kubelet initializes
the application containers for the Pod and runs them as usual.
 -->
###  初始化容器和普通容器有啥区别

初始化容器动脚应用容器的所有字段和特性， 包括资源限制， 数据卷， 安全设置。
但是对初始化化容器对资源的限制处理方式有所区别，具体见 [资源](#resources).

还有，初始化容器不支持 `lifecycle`, `livenessProbe`, `readinessProbe`, `startupProbe`，
因为只有初始化容器执行完 Pod 才可能进入就绪状态。

如果一个 Pod 中配置了多个初始化容器，则 kubelet 会顺序依次运行。
只有在上一初始化容器运行结束后，下一个才能开始运行。
当所有初始化容器运行完成后， kubelet 才会初始化 Pod 中的应用容器并以常规方式运行
<!--
## Using init containers

Because init containers have separate images from app containers, they
have some advantages for start-up related code:

* Init containers can contain utilities or custom code for setup that are not present in an app
  image. For example, there is no need to make an image `FROM` another image just to use a tool like
  `sed`, `awk`, `python`, or `dig` during setup.
* The application image builder and deployer roles can work independently without
  the need to jointly build a single app image.
* Init containers can run with a different view of the filesystem than app containers in the
  same Pod. Consequently, they can be given access to
  {{< glossary_tooltip text="Secrets" term_id="secret" >}} that app containers cannot access.
* Because init containers run to completion before any app containers start, init containers offer
  a mechanism to block or delay app container startup until a set of preconditions are met. Once
  preconditions are met, all of the app containers in a Pod can start in parallel.
* Init containers can securely run utilities or custom code that would otherwise make an app
  container image less secure. By keeping unnecessary tools separate you can limit the attack
  surface of your app container image.
 -->
## 怎么使用初始化容器

因为初始化容器与应用容器是使用不同的镜像，所以在执行初始化相关代码有些优势:

- 初始化容器可以包含应用容器中没有的工作或自定义代码。
  例如， 不需要为了用像 `sed`, `awk`, `python`, `dig` 来做初始化而要在应用镜像中加入
  `FROM` 其它的镜像。
- 应用镜像的构建和部署角色可以独立工作，而不需要打在一个应用镜像里
- 同一个 Pod 的初始化化容器可以与应用容器运行在不同文件系统视角下， 因而可以让初始化容器可以读取
  应用容器不能读取的 {{< glossary_tooltip text="Secrets" term_id="secret" >}}。
- 因为只能在初始化容器运行完成后应用容器才能启动， 初始化容器就提供了一种机制，可以让容器在
  达成一定前置条件之前应用容器会被阻塞或延迟。 当前置条件达成， Pod 中所有的应用容器可以并行启动。
- 可以在初始化容器可以安全地运行放在应用容器中可以不那么安全的工具或自定义代码。
  通过将不必要的工具从应用镜像中移出(到初始化容器中)可以减少应用容器的攻击面

<!--
### Examples
Here are some ideas for how to use init containers:

* Wait for a {{< glossary_tooltip text="Service" term_id="service">}} to
  be created, using a shell one-line command like:
  ```shell
  for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
  ```

* Register this Pod with a remote server from the downward API with a command like:
  ```shell
  curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```

* Wait for some time before starting the app container with a command like
  ```shell
  sleep 60
  ```

* Clone a Git repository into a {{< glossary_tooltip text="Volume" term_id="volume" >}}

* Place values into a configuration file and run a template tool to dynamically
  generate a configuration file for the main app container. For example,
  place the `POD_IP` value in a configuration and generate the main app
  configuration file using Jinja.
 -->
### 示例

以下为几个怎么用初始化容器的点子:

- 等待一个 {{< glossary_tooltip text="Service" term_id="service">}} 创建完成，
  使用以下 shell 命令:
  ```shell
  for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
  ```
- 通过以下命令将该 Pod 注册到一个远程服务的 WEB API
  ```shell
  curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```
- 等待一定时间后再启动 Pod中的应用容器  
  ```shell
  sleep 60
  ```
- 从 Git 仓库中克隆一个库到 {{< glossary_tooltip text="Volume" term_id="volume" >}}
- 将配置值放入配置文件，通过一个模板工具为应用容器动态生成配置文件， 例， 将 `POD_IP` 放丰配置文件中
  通过如 `Jinja` 这样的工具生成主应用的配置文件
<!--
#### Init containers in use

This example defines a simple Pod that has two init containers.
The first waits for `myservice`, and the second waits for `mydb`. Once both
init containers complete, the Pod runs the app container from its `spec` section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

You can start this Pod by running:

```shell
kubectl apply -f myapp.yaml
```
```
pod/myapp-pod created
```

And check on its status with:
```shell
kubectl get -f myapp.yaml
```
```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```

or for more details:
```shell
kubectl describe -f myapp.yaml
```
```
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
```

To see logs for the init containers in this Pod, run:
```shell
kubectl logs myapp-pod -c init-myservice # Inspect the first init container
kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
```

At this point, those init containers will be waiting to discover Services named
`mydb` and `myservice`.

Here's a configuration you can use to make those Services appear:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

To create the `mydb` and `myservice` services:

```shell
kubectl apply -f services.yaml
```
```
service/myservice created
service/mydb created
```

You'll then see that those init containers complete, and that the `myapp-pod`
Pod moves into the Running state:

```shell
kubectl get -f myapp.yaml
```
```
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

This simple example should provide some inspiration for you to create your own
init containers. [What's next](#whats-next) contains a link to a more detailed example.
 -->
#### 初始化容器实践

本示例定义一个简单的Pod， 其中包含两个初始化容器。 第一个等待 `myservice` 的创建，
第二个等待 `mydb` 的创建。 当这两个初始化容器都运行完成，则运行 `spec` 配置的应用容器。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

通过以下命令启动该 Pod
```sh
kubectl apply -f myapp.yaml
```
```
pod/myapp-pod created
```

通过以下命令查看 Pod 状态
```sh
kubectl get -f myapp.yaml
```
结果类似如下
```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```
通过以下命令可以查看更详情的信息
```sh
kubectl describe -f myapp.yaml
```
结果类似如下
```
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
```

通过以下命令查看初始化容器的日志
```sh
kubectl logs myapp-pod -c init-myservice # 查看第一个初始化容器
kubectl logs myapp-pod -c init-mydb      # 查看第二个初始化容器
```

此时，这两个初始化容器都分别在等待 `myservice` 和 `mydb` 两个
{{< glossary_tooltip term_id="service">}} 被创建
以下为创建所需要 {{< glossary_tooltip term_id="service">}} 的定义文件
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```
通过以下命令创建 `myservice` 和 `mydb` 两个
{{< glossary_tooltip term_id="service">}}
```sh
kubectl apply -f services.yaml
```
输出
```
service/myservice created
service/mydb created
```
这时候再使用以下命令就会发现初始化容器已经完成， 名叫 myapp-pod 的 Pod 进入运行状态
```sh
kubectl get -f myapp.yaml
```
```
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```
这个简单示例可以对用户创建自己的初始化容器有所启发。 更多关于初始化容器的示例见 [相关资料](#whats-next)
<!--
## Detailed behavior

During Pod startup, the kubelet delays running init containers until the networking
and storage are ready. Then the kubelet runs the Pod's init containers in the order
they appear in the Pod's spec.

Each init container must exit successfully before
the next container starts. If a container fails to start due to the runtime or
exits with failure, it is retried according to the Pod `restartPolicy`. However,
if the Pod `restartPolicy` is set to Always, the init containers use
`restartPolicy` OnFailure.

A Pod cannot be `Ready` until all init containers have succeeded. The ports on an
init container are not aggregated under a Service. A Pod that is initializing
is in the `Pending` state but should have a condition `Initialized` set to true.

If the Pod [restarts](#pod-restart-reasons), or is restarted, all init containers
must execute again.

Changes to the init container spec are limited to the container image field.
Altering an init container image field is equivalent to restarting the Pod.

Because init containers can be restarted, retried, or re-executed, init container
code should be idempotent. In particular, code that writes to files on `EmptyDirs`
should be prepared for the possibility that an output file already exists.

Init containers have all of the fields of an app container. However, Kubernetes
prohibits `readinessProbe` from being used because init containers cannot
define readiness distinct from completion. This is enforced during validation.

Use `activeDeadlineSeconds` on the Pod and `livenessProbe` on the container to
prevent init containers from failing forever. The active deadline includes init
containers.

The name of each app and init container in a Pod must be unique; a
validation error is thrown for any container sharing a name with another.
 -->
## 一些细节行为

在 Pod 的启动过程中， kubelet 会等待网络和存储就绪后才会运行初始化容器。
kubelet 会按照定义配置中的顺序运行初始化容器。

后一个初始化容器已经在前一个运行且成功退出后才启动。 如果一个容器因为运行环境而挂掉或错误退出，
会根据 Pod 上配置 `restartPolicy` 进行重试。 但如果 Pod `restartPolicy` 值为 `Always`，
初始化容器对应 `restartPolicy` 的值为 `OnFailure`

如果 Pod [重启](#pod-restart-reasons)，或已经重启，所有的初始化容器都会重新执行。

初始化容器的定义配置中能修改的字段只有容器的 `image` 字段，
如果修改初始化容器的 `image` 字段，则表示要重启该 Pod。

因为初始化容器可以重启，重试，或重新执行， 所以初始化容器的代码必须是幂等的。 特别是，如果代码
需要向 `EmptyDirs` 写入文件，需要考虑到输出文件已经存在的情况。

初始化容器包含应用容器拥有的所有字段。但 kubelet 禁止在初始化容器上使用 `readinessProbe`，
因为定义的就绪探针与一个执行完就退出的任务是不适用的。 这会在验证是检查，如果出现则报错。

在 Pod 上使用 `activeDeadlineSeconds` 和容器上使用 `livenessProbe` 可以防止初始化容器
一直失败的情况出现。 活跃死线包含初始化容器。

在 Pod 中的应用容器和初始化容器的名称全局(Pod 作用域内)唯一。 如果有相同的名称在验证时会报错。
<!--
### Resources

Given the ordering and execution for init containers, the following rules
for resource usage apply:

* The highest of any particular resource request or limit defined on all init
  containers is the *effective init request/limit*
* The Pod's *effective request/limit* for a resource is the higher of:
  * the sum of all app containers request/limit for a resource
  * the effective init request/limit for a resource
* Scheduling is done based on effective requests/limits, which means
  init containers can reserve resources for initialization that are not used
  during the life of the Pod.
* The QoS (quality of service) tier of the Pod's *effective QoS tier* is the
  QoS tier for init containers and app containers alike.

Quota and limits are applied based on the effective Pod request and
limit.

Pod level control groups (cgroups) are based on the effective Pod request and
limit, the same as the scheduler.
 -->
### 资源 {#resources}

为了让初始化容器能获取到执行所需要的资源，以下为资源申请的规则:

- 在所有初始化容器中的资源 下限/上限 的单项资源最高值为 *有效初始化下限/上限*
- Pod 的每项资源的 *有效 下限/上限* 是以下中较大的一个:
  - 所有应用容器该资源的 下限/上限 的总和
  - 该资源的 有效初始化下限/上限
- 调度是基于 资源 有效 下限/上限来进行的。也就是初始化容器可以申请用于初始化的资源可能在 Pod
  的余生中都用不到了。
- Pod 的 有效 Qos 层中的 QoS (服务质量)层就是初始化容器和应用容器通用的

配额和限制都是基于有效 Pod 下限/上限执行。

Pod 级别的控制组(cgroups)基于 有效 Pod 下限/上限， 与调度器一致

{{<todo-optimize>}}
<!--
### Pod restart reasons {#pod-restart-reasons}

A Pod can restart, causing re-execution of init containers, for the following
reasons:

* A user updates the Pod specification, causing the init container image to change.
  Any changes to the init container image restarts the Pod. App container image
  changes only restart the app container.
* The Pod infrastructure container is restarted. This is uncommon and would
  have to be done by someone with root access to nodes.
* All containers in a Pod are terminated while `restartPolicy` is set to Always,
  forcing a restart, and the init container completion record has been lost due
  to garbage collection.
 -->
### 引起 Pod 重启的原因 {#pod-restart-reasons}

可能引起 一个 Pod 重启并导致初始化容器的重新执行的原因如下：

- 某个用户更新的 Pod 定义配置， 导致初始化容器的镜像变更。 任意对初始化容器镜像的修改都会导致 Pod 重启。
  应用容器镜像变更只能重启该应用容器
- Pod 基础设施容器被重启，这种情况不常见， 只能由拥有节点 root 权限的用户进行。
- Pod 中所有的容器都被终止，因为`restartPolicy` 的值为 `Always`，所以强制重启， 此时初始化容器
  的完成记录因为垃圾清楚而丢失。

## {{% heading "whatsnext" %}} {#whats-next}

<!--
* Read about [creating a Pod that has an init container](/docs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container)
* Learn how to [debug init containers](/docs/tasks/debug-application-cluster/debug-init-containers/)
 -->
* 实践 [创建带有初始化容器的 Pod](/k8sDocs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container)
* 实践 [调试初始化容器](/k8sDocs/tasks/debug-application-cluster/debug-init-containers/)
