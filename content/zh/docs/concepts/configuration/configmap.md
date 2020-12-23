---
title: ConfigMap
content_type: concept
weight: 20
---
<!--
---
title: ConfigMaps
content_type: concept
weight: 20
---
 -->
<!-- overview -->

<!--

{{< glossary_definition term_id="configmap" prepend="A ConfigMap is" length="all" >}}

{{< caution >}}
ConfigMap does not provide secrecy or encryption.
If the data you want to store are confidential, use a
{{< glossary_tooltip text="Secret" term_id="secret" >}} rather than a ConfigMap,
or use additional (third party) tools to keep your data private.
{{< /caution >}}
 -->

{{< glossary_definition term_id="configmap" prepend="ConfigMap 就是" length="all" >}}

{{< caution >}}
ConfigMap 不提供保密性或对其加密。 如果想要存放的保留信息，请使用
{{< glossary_tooltip text="Secret" term_id="secret" >}}
而不是 ConfigMap， 或使用外部(第三方)工具来保证数据的保密性。
{{< /caution >}}

<!-- body -->
<!--
## Motivation

Use a ConfigMap for setting configuration data separately from application code.

For example, imagine that you are developing an application that you can run on your
own computer (for development) and in the cloud (to handle real traffic).
You write the code to look in an environment variable named `DATABASE_HOST`.
Locally, you set that variable to `localhost`. In the cloud, you set it to
refer to a Kubernetes {{< glossary_tooltip text="Service" term_id="service" >}}
that exposes the database component to your cluster.
This lets you fetch a container image running in the cloud and
debug the exact same code locally if needed.

A ConfigMap is not designed to hold large chunks of data. The data stored in a
ConfigMap cannot exceed 1 MiB. If you need to store settings that are
larger than this limit, you may want to consider mounting a volume or use a
separate database or file service.
 -->

## 动机

使用 ConfigMap 将配置数据与应用代码分离。

例如，假设开发了一个应用，可以在本地机器上(开发)运行和云环境(处理生产流量)。 可以在代码中读取
一个叫 `DATABASE_HOST` 环境变量。 在本地，可以将这个变量设置为 `localhost`。 在云环境中
可以将它指定 k8s 集群中用于暴露数据库组件的
{{< glossary_tooltip text="Service" term_id="service" >}}
名称。 如果需要，这会使得用户可以在云环境中拉取容器镜像并运行的与本地调试使用的是同一套代码。

ConfigMap 并不是设计来存放大块数据。 存放于 ConfigMap 的数据不能超过 1 MiB。 如果需要存储
超过这个限制的配置数据， 可以考虑挂载一个卷，或使用独立的数据库或文件服务。

<!--
## ConfigMap object

A ConfigMap is an API [object](/docs/concepts/overview/working-with-objects/kubernetes-objects/)
that lets you store configuration for other objects to use. Unlike most
Kubernetes objects that have a `spec`, a ConfigMap has `data` and `binaryData`
fields. These fields accepts key-value pairs as their values.  Both the `data`
field and the `binaryData` are optional. The `data` field is designed to
contain UTF-8 byte sequences while the `binaryData` field is designed to
contain binary data.

The name of a ConfigMap must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

Each key under the `data` or the `binaryData` field must consist of
alphanumeric characters, `-`, `_` or `.`. The keys stored in `data` must not
overlap with the keys in the `binaryData` field.

Starting from v1.19, you can add an `immutable` field to a ConfigMap
definition to create an [immutable ConfigMap](#configmap-immutable).
 -->

## ConfigMap 对象

ConfigMap 是一个 API
[对象](/k8sDocs/docs/concepts/overview/working-with-objects/kubernetes-objects/)
, 它可以让用户存放供其它对象使用的配置信息。 与其它大多数据 k8s 对象有 `spec` 不同， ConfigMap
包含的字段有 `data` 和 `binaryData`。 这些字段接受键值对作为他们的值。 `data` 字段和
`binaryData` 字段都是可选的。 `data` 字段设计上是用来存储 UTF-8 字节顺序的而 `binaryData`
字段设计上是用来存储二进制数据的。

ConfigMap 的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

`data` 或 `binaryData` 字段下的键必须由字母数字，`-`, `_` 或 `.` 组成。 `data` 存放的
键与 `binaryData` 字段下面的键不能重复。

从 k8s  v1.19 开始，可以在 ConfigMap 中添加 `immutable` 字段，定义一个
[不可变的 ConfigMap](#configmap-immutable).
<!--
## ConfigMaps and Pods

You can write a Pod `spec` that refers to a ConfigMap and configures the container(s)
in that Pod based on the data in the ConfigMap. The Pod and the ConfigMap must be in
the same {{< glossary_tooltip text="namespace" term_id="namespace" >}}.

Here's an example ConfigMap that has some keys with single values,
and other keys where the value looks like a fragment of a configuration
format.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

There are four different ways that you can use a ConfigMap to configure
a container inside a Pod:

1. Inside a container command and args
1. Environment variables for a container
1. Add a file in read-only volume, for the application to read
1. Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

These different methods lend themselves to different ways of modeling
the data being consumed.
For the first three methods, the
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} uses the data from
the ConfigMap when it launches container(s) for a Pod.

The fourth method means you have to write code to read the ConfigMap and its data.
However, because you're using the Kubernetes API directly, your application can
subscribe to get updates whenever the ConfigMap changes, and react
when that happens. By accessing the Kubernetes API directly, this
technique also lets you access a ConfigMap in a different namespace.

Here's an example Pod that uses values from `game-demo` to configure a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # Define the environment variable
        - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                     # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    # You set volumes at the Pod level, then mount them into containers inside that Pod
    - name: config
      configMap:
        # Provide the name of the ConfigMap you want to mount.
        name: game-demo
        # An array of keys from the ConfigMap to create as files
        items:
        - key: "game.properties"
          path: "game.properties"
        - key: "user-interface.properties"
          path: "user-interface.properties"
```

A ConfigMap doesn't differentiate between single line property values and
multi-line file-like values.
What matters is how Pods and other objects consume those values.

For this example, defining a volume and mounting it inside the `demo`
container as `/config` creates two files,
`/config/game.properties` and `/config/user-interface.properties`,
even though there are four keys in the ConfigMap. This is because the Pod
definition specifies an `items` array in the `volumes` section.
If you omit the `items` array entirely, every key  in the ConfigMap becomes
a file with the same name as the key, and you get 4 files.
-->

## ConfigMap 和 Pod

用户可以在编写一个 Pod `spec` 中引用一个 ConfigMap 并基于这个 ConfigMap 中的数据来配置这个
Pod 中的容器。 Pod 和 ConfigMap 必须在同一个
{{< glossary_tooltip term_id="namespace" >}}.

以下为一个 ConfigMap 的示例，其中包含几个有一个值的键和其它值看起来像是配置格式的片断的键

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # 属性格式的键; 每个键与一个值相映射
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # 文件格式的键
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

有以下四种方式可以使用 ConfigMap 配置 Pod 中的容器:

1. 在一个容器中的命令和参数
2. 容器中的环境变量
3. 添加一个文件到一个只读卷，用于应用读取
4. 在 Pod 中写代码使用 k8s API 来读取 ConfigMap

这些不同的方式使得它们可以不用模式供给数据。
对于前三种方式
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} 在为 Pod 启动容器时
使用 ConfigMap 中的数据。

第四种方式意味着需要用户写代理来读取 ConfigMap 和其中的数据。 但因为直接使用的是 k8s API,
应用可以通过订阅来获得 ConfigMap 变更时的更新， 并在其发生时做出反映。 通过直接访问 k8s API,
这种方式也让用户能够访问其它命名空间的 ConfigMap。

Here's an example Pod that uses values from `game-demo` to configure a Pod:
以下示例中的 Pod 使用 `game-demo`  中的值来配置一个这个 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # 定义环境变量
        - name: PLAYER_INITIAL_LIVES # 注意这个键与 ConfigMap 键名称是不一样的
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 引用 ConfigMap 的名称
              key: player_initial_lives # 使用其中的键
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    # 可以在 Pod 级别设置卷， 然后将它们挂载到容器
    - name: config
      configMap:
        # 想要挂载的 ConfigMap 的名称
        name: game-demo
        # ConfigMap 中用来创建文件的键的列表
        items:
        - key: "game.properties"
          path: "game.properties"
        - key: "user-interface.properties"
          path: "user-interface.properties"
```

ConfigMap 是的单行属性值和多行的类似文件的值是没有不同的。重要的是 Pod 和其它对象是怎么使用这些值的。

对于这个示例， 定义一个卷并将其挂载到 `demo` 容器中的 `/config` 目录，并创建两个文件，
`/config/game.properties` 和 `/config/user-interface.properties`, 即便 ConfigMap
中包含了四个键。 这是因为 Pod 中的 `volumes` 区域定义一个 `items` 数组。 如果省略了
`items` 数组实体， 每个 ConfigMap 中的键就会变成以这个键名称一样的文件，这样就会有四个文件。
<!--
## Using ConfigMaps

ConfigMaps can be mounted as data volumes. ConfigMaps can also be used by other
parts of the system, without being directly exposed to the Pod. For example,
ConfigMaps can hold data that other parts of the system should use for configuration.

The most common way to use ConfigMaps is to configure settings for
containers running in a Pod in the same namespace. You can also use a
ConfigMap separately.

For example, you
might encounter {{< glossary_tooltip text="addons" term_id="addons" >}}
or {{< glossary_tooltip text="operators" term_id="operator-pattern" >}} that
adjust their behavior based on a ConfigMap.
 -->

## 使用 ConfigMap

ConfigMap 可以挂载为数据卷。 ConfigMap 也可以在不直接暴露给 Pod 的情况下被系统的其它部分使用。
例如， ConfigMap 可以包含系统其它部分用于檲的数据。

ConfigMap 最常用的一种方式就为在同一个命名空间中的 Pod 中运行的容器提供配置。 也可以单独使用
ConfigMap

例如，也可能遇到
{{< glossary_tooltip text="addons" term_id="addons" >}}
和
{{< glossary_tooltip text="operators" term_id="operator-pattern" >}}
基于 ConfigMap 来调整他们的行为。
<!--
### Using ConfigMaps as files from a Pod

To consume a ConfigMap in a volume in a Pod:

1. Create a ConfigMap or use an existing one. Multiple Pods can reference the
   same ConfigMap.
1. Modify your Pod definition to add a volume under `.spec.volumes[]`. Name
   the volume anything, and have a `.spec.volumes[].configMap.name` field set
   to reference your ConfigMap object.
1. Add a `.spec.containers[].volumeMounts[]` to each container that needs the
   ConfigMap. Specify `.spec.containers[].volumeMounts[].readOnly = true` and
   `.spec.containers[].volumeMounts[].mountPath` to an unused directory name
   where you would like the ConfigMap to appear.
1. Modify your image or command line so that the program looks for files in
   that directory. Each key in the ConfigMap `data` map becomes the filename
   under `mountPath`.

This is an example of a Pod that mounts a ConfigMap in a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

Each ConfigMap you want to use needs to be referred to in `.spec.volumes`.

If there are multiple containers in the Pod, then each container needs its
own `volumeMounts` block, but only one `.spec.volumes` is needed per ConfigMap.
-->

### 将 ConfigMap 以文件的方式用到 Pod 中

在 Pod 中以卷的方式使用一个 ConfigMap:

1. 创建一个新的 ConfigMap 或使用一个现有的。 多个 Pod 可以引用同一个 ConfigMap
2. 修改 Pod 定义，在 `.spec.volumes[]` 下面添加一个卷。这卷的名称随便起， 但其中
  `.spec.volumes[].configMap.name` 字段需要设置引用上一步提到的 ConfigMap 对象
3. 在每一个需要访问这个 ConfigMap 的容器中添加 `.spec.containers[].volumeMounts[]`。
  设置 `.spec.containers[].volumeMounts[].readOnly = true` 和
  将 `.spec.containers[].volumeMounts[].mountPath` 指向一个期望的未使用的目录名
4. 修改镜像或命令让容器中的程序查看目录中的文件。 ConfigMap 中 `data` 字典下的每一个键就会
  对应 `mountPath` 目录中的一个文件名

下面这个示例中就一个将一个 ConfigMap 挂载为卷的 Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

每个想要使用的 ConfigMap 都需要在 `.spec.volumes` 中被引用。

如果 Pod 中有多个容器， 每个容器都需要有自己的 `volumeMounts` 块， 但每个 ConfigMap 只需要
一个 `.spec.volumes`
<!--
#### Mounted ConfigMaps are updated automatically

When a ConfigMap currently consumed in a volume is updated, projected keys are eventually updated as well.
The kubelet checks whether the mounted ConfigMap is fresh on every periodic sync.
However, the kubelet uses its local cache for getting the current value of the ConfigMap.
The type of the cache is configurable using the `ConfigMapAndSecretChangeDetectionStrategy` field in
the [KubeletConfiguration struct](https://github.com/kubernetes/kubernetes/blob/{{< param "docsbranch" >}}/staging/src/k8s.io/kubelet/config/v1beta1/types.go).
A ConfigMap can be either propagated by watch (default), ttl-based, or simply redirecting
all requests directly to the API server.
As a result, the total delay from the moment when the ConfigMap is updated to the moment
when new keys are projected to the Pod can be as long as the kubelet sync period + cache
propagation delay, where the cache propagation delay depends on the chosen cache type
(it equals to watch propagation delay, ttl of cache, or zero correspondingly).

ConfigMaps consumed as environment variables are not updated automatically and require a pod restart.
 -->
#### 让挂载的 ConfigMap 自动更新

当一个正在被以卷方式使用的 ConfigMap 更新时，与其相映射的键最终也会更新。 kubelet 会在每个
同步周期检查挂载的 ConfigMap 是否更新。 但是， kubelet 会使用本地缓存来获取 ConfigMap 的
当前值。 缓存的类型可以通过
[KubeletConfiguration struct](https://github.com/kubernetes/kubernetes/blob/{{< param "docsbranch" >}}/staging/src/k8s.io/kubelet/config/v1beta1/types.go).
中的 `ConfigMapAndSecretChangeDetectionStrategy` 字段来配置。 ConfigMap 的传播方式有
监视(默认)，基于 ttl, 或简单地将所有请求直接重定向给 API server. 最终， 从 ConfigMap 更新
到新的键被投射到 Pod 中的总延时就是 kubelet 同时间隔时长 + 缓存传播延时， 而其中缓存传播延时
又基于缓存的类型(相应地它可能等于 监视传播延时，缓存的 TTL, 或零)。

通过环境变量引用的 ConfigMap 是不能自动更新的，需要重启 Pod 才行。
<!--
## Immutable ConfigMaps {#configmap-immutable}

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

The Kubernetes beta feature _Immutable Secrets and ConfigMaps_ provides an option to set
individual Secrets and ConfigMaps as immutable. For clusters that extensively use ConfigMaps
(at least tens of thousands of unique ConfigMap to Pod mounts), preventing changes to their
data has the following advantages:

- protects you from accidental (or unwanted) updates that could cause applications outages
- improves performance of your cluster by significantly reducing load on kube-apiserver, by
  closing watches for ConfigMaps marked as immutable.

This feature is controlled by the `ImmutableEphemeralVolumes`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/).
You can create an immutable ConfigMap by setting the `immutable` field to `true`.
For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

Once a ConfigMap is marked as immutable, it is _not_ possible to revert this change
nor to mutate the contents of the `data` or the `binaryData` field. You can
only delete and recreate the ConfigMap. Because existing Pods maintain a mount point
to the deleted ConfigMap, it is recommended to recreate these pods.
 -->

## 不可变 ConfigMap {#configmap-immutable}

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

这个 k8s 的 bata 特性 _不可变 Secret 和 ConfigMap_ 提供了一个可选项，可以让一个 Secret
和 ConfigMap 变为不可变。 对于那些广泛使用 ConfigMap (一个 ConfigMap 至少被 10k Pod 挂载)，
防止修改它们中的数据有以下好处:

- 防止误操作(或不想要)的更新可能引发的应用事故
- 当 ConfigMap 标记为不可变时会关闭监视，这样能极大地减少 kube-apiserver 的负载，从而改善
  集群性能。

这个特性通过设置 `ImmutableEphemeralVolumes`
[功能阀](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)
控制。 用户可以在 ConfigMap 中通过设置 `immutable` 字段为 `true` 让其成功不可变 ConfigMap

示例：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

当一个 ConfigMap 被标记为不可变后，它就 _不_ 可能再被变会普通(可修改)的了，也不可能再修改其
中 `data` 或 `binaryData` 字段的值。只能删除或重建 ConfigMap。 因为现在的 Pod 会维持
对已经删除的 ConfigMap 挂载指向， 推荐对这些 Pod 也进行重建。

## {{% heading "whatsnext" %}}

* 概念 [Secrets](/k8sDocs/docs/concepts/configuration/secret/).
* 实践 [配置一个 Pod 使用 ConfigMap](/k8sDocs/docs/tasks/configure-pod-container/configure-pod-configmap/).
* 阅读 [The Twelve-Factor App](https://12factor.net/) 以便理解分离代码和配置的动机
