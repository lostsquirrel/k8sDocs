---
title: 配置最佳实践
content_type: concept
weight: 10
---
<!--
---
reviewers:
- mikedanese
title: Configuration Best Practices
content_type: concept
weight: 10
---
 -->
<!-- overview -->

本文主要加强和巩固从用户指引，入门文档，和示例中介绍过的配置的最佳实践。

这是一个活动文档，如果用户觉得有些内容文档上没有但可以对其他人有帮助，不要犹豫，赶紧提交问题单或
提供 PR

<!-- body -->
<!--
## General Configuration Tips

- When defining configurations, specify the latest stable API version.

- Configuration files should be stored in version control before being pushed to the cluster. This allows you to quickly roll back a configuration change if necessary. It also aids cluster re-creation and restoration.

- Write your configuration files using YAML rather than JSON. Though these formats can be used interchangeably in almost all scenarios, YAML tends to be more user-friendly.

- Group related objects into a single file whenever it makes sense. One file is often easier to manage than several. See the [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/all-in-one/guestbook-all-in-one.yaml) file as an example of this syntax.

- Note also that many `kubectl` commands can be called on a directory. For example, you can call `kubectl apply` on a directory of config files.

- Don't specify default values unnecessarily: simple, minimal configuration will make errors less likely.

- Put object descriptions in annotations, to allow better introspection.
 -->

## 通用配置提示


- 在定义配置时，指定最新的稳定 API 版本。

- 配置文件在推送到集群之前先保住到版本控制系统。 这样如果需要可以快速回滚配置变更。 也可以帮助集群重建或恢复。

- 使用 YAML 格式编写配置文件而不是 JSON。 虽然这两个格式在几乎所有场景下都可以互换，但 YAML 对用户更友好。

- 当几个对象有关系时将其放在一个文件中。 一个文件会比多个文件更容易管理。相关语法见示例
   [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/all-in-one/guestbook-all-in-one.yaml)
- 要知道 `kubectl` 命令可以向一个目录调用，例如，可以对一个目录执行 `kubectl apply`，表示
  执行其中的所有配置文件
- 请不要指定不必要的默认值: 简单，最小化配置，通常可以减少错误的产生。
- 将对象描述加到注解中，让其可以对用户或自动化工具更友好。
<!--
## "Naked" Pods versus ReplicaSets, Deployments, and Jobs {#naked-pods-vs-replicasets-deployments-and-jobs}

- Don't use naked Pods (that is, Pods not bound to a [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/) or [Deployment](/docs/concepts/workloads/controllers/deployment/)) if you can avoid it. Naked Pods will not be rescheduled in the event of a node failure.

  A Deployment, which both creates a ReplicaSet to ensure that the desired number of Pods is always available, and specifies a strategy to replace Pods (such as [RollingUpdate](/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)), is almost always preferable to creating Pods directly, except for some explicit [`restartPolicy: Never`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) scenarios. A [Job](/docs/concepts/workloads/controllers/job/) may also be appropriate.
-->


## "裸奔"的 Pod VS ReplicaSet, Deployment, Job {#naked-pods-vs-replicasets-deployments-and-jobs}

- 如果能避免，尽量不要让 Pod 裸奔(也就是没有与
  [ReplicaSet](/k8sDocs/docs/concepts/workloads/controllers/replicaset/)
  或
  [Deployment](/k8sDocs/docs/concepts/workloads/controllers/deployment/)
  绑定的 Pod
  )。
  裸奔的 Pod 在节点挂掉之后不会重新调度。

  面 Deployment， 在创建一个 ReplicaSet 来保证期望个数的 Pod 始终可用外，还可以指定替换策略
  (如
    [RollingUpdate](/k8sDocs/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
  )，
  除了一个具体场景
  ([`restartPolicy: Never`](/k8sDocs/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy))
  都推荐使用 Deployment 而不是直接创建 Pod。
  [Job](/k8sDocs/docs/concepts/workloads/controllers/job/) 也是另一种选择。
<!--
## Services

- Create a [Service](/docs/concepts/services-networking/service/) before its corresponding backend workloads (Deployments or ReplicaSets), and before any workloads that need to access it. When Kubernetes starts a container, it provides environment variables pointing to all the Services which were running when the container was started. For example, if a Service named `foo` exists, all containers will get the following variables in their initial environment:

  ```shell
  FOO_SERVICE_HOST=<the host the Service is running on>
  FOO_SERVICE_PORT=<the port the Service is running on>
  ```

  *This does imply an ordering requirement* - any `Service` that a `Pod` wants to access must be created before the `Pod` itself, or else the environment variables will not be populated.  DNS does not have this restriction.

- An optional (though strongly recommended) [cluster add-on](/docs/concepts/cluster-administration/addons/) is a DNS server.  The
DNS server watches the Kubernetes API for new `Services` and creates a set of DNS records for each.  If DNS has been enabled throughout the cluster then all `Pods` should be able to do name resolution of `Services` automatically.

- Don't specify a `hostPort` for a Pod unless it is absolutely necessary. When you bind a Pod to a `hostPort`, it limits the number of places the Pod can be scheduled, because each <`hostIP`, `hostPort`, `protocol`> combination must be unique. If you don't specify the `hostIP` and `protocol` explicitly, Kubernetes will use `0.0.0.0` as the default `hostIP` and `TCP` as the default `protocol`.

  If you only need access to the port for debugging purposes, you can use the [apiserver proxy](/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls) or [`kubectl port-forward`](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

  If you explicitly need to expose a Pod's port on the node, consider using a [NodePort](/docs/concepts/services-networking/service/#nodeport) Service before resorting to `hostPort`.

- Avoid using `hostNetwork`, for the same reasons as `hostPort`.

- Use [headless Services](/docs/concepts/services-networking/service/#headless-services) (which have a `ClusterIP` of `None`) for easy service discovery when you don't need `kube-proxy` load balancing.
 -->

## Service

- 在对应的后端工作负载(Deployments 或 ReplicaSets) 和任意访问它的工作负载之前创建一个
  [Service](/docs/concepts/services-networking/service/)。
  当 k8s 启动一个容器时， 它会在容器启动时生成指向所有正在运行的 Service 的环境变量。
  例如， 如果存在一个叫 `foo` 的 Service，所有都会有如下初始化环境变量:
  ```shell
  FOO_SERVICE_HOST=<the host the Service is running on>
  FOO_SERVICE_PORT=<the port the Service is running on>
  ```

  *这就存在一个隐式的启动顺序要求* - 一个 Pod 想要访问的任意 Service 都需要在这个 Pod 创建
  之前就要存在，否则对应的环境变量就不会生成。 DNS 就没有这个限制。

- 一个可选(但强烈推荐)的
  [集群插件](/docs/concepts/cluster-administration/addons/)
  就是 DNS 服务。 DNS 服务会监听 k8s API 中新增的 `Services` 并对应创建一系列 DNS 记录。
  如果集群中启动了 DNS 则 所有的 Pod 都可以自动通过 DNS 解析访问 `Services`

- 如果不必要，请不要为一个 Pod 指定 `hostPort`。 当将一个 Pod 绑定到一个 `hostPort` 时，
  就会限制这个 Pod 能调度的范围， 因为每个 <`hostIP`, `hostPort`, `protocol`> 的组合在
  全集群内是唯一的。 如果没有显示地指定 `hostIP` 和 `protocol`，k8s 全使用 `0.0.0.0`
  作为默认的 `hostIP`， `TCP` 作为默认的 `protocol`.

  如果只为调试目的需要访问端口，可以使用
  [apiserver proxy](/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls)
  或
  [`kubectl port-forward`](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).
  如果确实需要将 Pod 端口在节点上显露，在使用 `hostPort` 之前考虑使用 `hostPort`的
  [NodePort](/docs/concepts/services-networking/service/#nodeport)

- 与 `hostPort` 的理由一样，避免使用 `hostNetwork`

- 在不需要 `kube-proxy` 负载均衡的情况下，使用
  [headless Services](/docs/concepts/services-networking/service/#headless-services)
  (就是将 `ClusterIP` 设置为 `None`)
  来实现简单的服务发现
<!--
## Using Labels

- Define and use [labels](/docs/concepts/overview/working-with-objects/labels/) that identify __semantic attributes__ of your application or Deployment, such as `{ app: myapp, tier: frontend, phase: test, deployment: v3 }`. You can use these labels to select the appropriate Pods for other resources; for example, a Service that selects all `tier: frontend` Pods, or all `phase: test` components of `app: myapp`. See the [guestbook](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/) app for examples of this approach.

A Service can be made to span multiple Deployments by omitting release-specific labels from its selector. [Deployments](/docs/concepts/workloads/controllers/deployment/) make it easy to update a running service without downtime.

A desired state of an object is described by a Deployment, and if changes to that spec are _applied_, the deployment controller changes the actual state to the desired state at a controlled rate.

- You can manipulate labels for debugging. Because Kubernetes controllers (such as ReplicaSet) and Services match to Pods using selector labels, removing the relevant labels from a Pod will stop it from being considered by a controller or from being served traffic by a Service. If you remove the labels of an existing Pod, its controller will create a new Pod to take its place. This is a useful way to debug a previously "live" Pod in a "quarantine" environment. To interactively remove or add labels, use [`kubectl label`](/docs/reference/generated/kubectl/kubectl-commands#label).
 -->

## 使用标签

- 为应用或 Deployment 定义和使用
  [标签](/k8sDocs/docs/concepts/overview/working-with-objects/labels/)
  来设置 __有语义的属性__, 如
  `{ app: myapp, tier: frontend, phase: test, deployment: v3 }`，
  这样其它的资源就可以通过这些标签来选择合适的 Pod; 例如， 一个 Service 可以选择所有
  `tier: frontend` 的 Pod，或者 `app: myapp` 的所有 `phase: test` 组件。
  更多使用方式见示例
  [guestbook](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/)

  一个 Service 可以在选择器中省略与发布版本相关的标签，这样就可以横跨多个版本。这样
  [Deployments](/k8sDocs/docs/concepts/workloads/controllers/deployment/)
  就可以很容易地在不停止提供服务的情况下实现服务的更新

  一个对象的期望状态是以一个 Deployment 来描述的， 如果它的配置的变更被_执行_，则 Deployment
  控制器会以一个控制频率将实际状态变更到与期望状态相同。

- 用户可以可能操作标签来方便调试。 因为 k8s 控制器(如 ReplicaSet) 和 Service 使用选择器标签
  来匹配 Pod， 移除相应的标签就可以认为是将其从一个控制器中移出或从一个 Service 服务后端移出。
  当移除一个现有 Pod 的标签后，它的控制器会创建一个新的 Pod 来代替它的位置。 这是一种在隔离环境中调试之前
  存活 Pod 的有效方式。 而要移除或添加标签，使用
  [`kubectl label`](/k8sDocs/docs/reference/generated/kubectl/kubectl-commands#label).

<!--
## Container Images

The [imagePullPolicy](/docs/concepts/containers/images/#updating-images) and the tag of the image affect when the [kubelet](/docs/reference/command-line-tools-reference/kubelet/) attempts to pull the specified image.

- `imagePullPolicy: IfNotPresent`: the image is pulled only if it is not already present locally.

- `imagePullPolicy: Always`: every time the kubelet launches a container, the kubelet queries the container image registry to resolve the name to an image digest. If the kubelet has a container image with that exact digest cached locally, the kubelet uses its cached image; otherwise, the kubelet downloads (pulls) the image with the resolved digest, and uses that image to launch the container.

- `imagePullPolicy` is omitted and either the image tag is `:latest` or it is omitted: `Always` is applied.

- `imagePullPolicy` is omitted and the image tag is present but not `:latest`: `IfNotPresent` is applied.

- `imagePullPolicy: Never`: the image is assumed to exist locally. No attempt is made to pull the image.

{{< note >}}
To make sure the container always uses the same version of the image, you can specify its [digest](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier); replace `<image-name>:<tag>` with `<image-name>@<digest>` (for example, `image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`). The digest uniquely identifies a specific version of the image, so it is never updated by Kubernetes unless you change the digest value.
{{< /note >}}

{{< note >}}
You should avoid using the `:latest` tag when deploying containers in production as it is harder to track which version of the image is running and more difficult to roll back properly.
{{< /note >}}

{{< note >}}
The caching semantics of the underlying image provider make even `imagePullPolicy: Always` efficient. With Docker, for example, if the image already exists, the pull attempt is fast because all image layers are cached and no image download is needed.
{{< /note >}}
-->

## 容器镜像

[imagePullPolicy](/docs/concepts/containers/images/#updating-images) 和镜像和标签会影响
[kubelet](/docs/reference/command-line-tools-reference/kubelet/)
拉取指定的镜像。

- `imagePullPolicy: IfNotPresent`: 镜像只在本地不存在时才会拉取。
- `imagePullPolicy: Always`: 每次 kubelet 启动一个容器时， kubelet 查询容器镜像仓库将名称解析为镜像摘要。
  如果 kubelet 发现本地缓存中有摘要完全相同的容器镜像时， kubelet 会使用缓存的镜像。否则，
  kubelet 使用解析的摘要下载(拉取)镜像，并使用这个镜像来启动容器。
- `imagePullPolicy` 如果该字段未设置而 镜像标签为 `:latest`，或没有标签 则会被忽略或者如果未设置值，则默认执行 `Always`
- `imagePullPolicy` 如果该字段未设置而镜像设置了标签并且不是 `:latest`，则该字段会应用为 `IfNotPresent`
- `imagePullPolicy: Never`: 假定镜像是本地存在。 不会尝试拉取镜像

{{< note >}}
要保证容器即便使用同一个版本的镜像，可以使用
[摘要](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier);
用 `<image-name>@<digest>`(例如，`image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`)
代替 `<image-name>:<tag>`。 摘要可以唯一标识指定版本的镜像， 所以在不修改摘要的情况，k8s
不会改掉的这个镜像。
{{< /note >}}

{{< note >}}
在生产环境中应当避免使用 `:latest` 标签，因为它会使得很难追踪正在运行的是哪个镜像，更难以实现
回滚。
{{< /note >}}

{{< note >}}
底层镜像供应者的缓存主义使得即便 `imagePullPolicy: Always` 也是高效的。 例如，Docker
如果镜像已经存在， 摘取尝试会很快，因为镜像的所有层都已经缓存并不需要下载。
{{< /note >}}
<!--
## Using kubectl

- Use `kubectl apply -f <directory>`. This looks for Kubernetes configuration in all `.yaml`, `.yml`, and `.json` files in `<directory>` and passes it to `apply`.

- Use label selectors for `get` and `delete` operations instead of specific object names. See the sections on [label selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors) and [using labels effectively](/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively).

- Use `kubectl create deployment` and `kubectl expose` to quickly create single-container Deployments and Services. See [Use a Service to Access an Application in a Cluster](/docs/tasks/access-application-cluster/service-access-application-cluster/) for an example.
 -->
## 使用 kubectl

- 使用`kubectl apply -f <directory>`. 这会查找 k8s 配置，包含 `<directory>` 中的
  `.yaml`, `.yml`, `.json` 文件，并作为 `apply` 的参数

- 使用标签选择器作为 `get` 和 `delete` 操作的选择条件而不指定对象名称。 见
  [标签选择器](/k8sDocs/docs/concepts/overview/working-with-objects/labels/#label-selectors)
  和
  [高效使用标签](/k8sDocs/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively).
- 使用 `kubectl create deployment` 和 `kubectl expose` 快速创建单容器的 Deployment 和 Service
  示例见
  [使用 Service 访问集群中的应用](/k8sDocs/docs/tasks/access-application-cluster/service-access-application-cluster/)
