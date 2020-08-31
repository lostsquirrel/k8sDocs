---
title: 垃圾回收
date: 2020-08-31
publishdate: 2020-08-31
weight: 2030105
---
<!--  
---
title: Garbage Collection
content_type: concept
weight: 60
---
-->
<!-- overview -->

在 k8s 中垃圾回收的作用就是删除那些曾经有所有者，现在没有的对象。

<!-- body -->
<!--
## Owners and dependents

Some Kubernetes objects are owners of other objects. For example, a ReplicaSet
is the owner of a set of Pods. The owned objects are called *dependents* of the
owner object. Every dependent object has a `metadata.ownerReferences` field that
points to the owning object.

Sometimes, Kubernetes sets the value of `ownerReference` automatically. For
example, when you create a ReplicaSet, Kubernetes automatically sets the
`ownerReference` field of each Pod in the ReplicaSet. In 1.8, Kubernetes
automatically sets the value of `ownerReference` for objects created or adopted
by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job
and CronJob.

You can also specify relationships between owners and dependents by manually
setting the `ownerReference` field.

Here's a configuration file for a ReplicaSet that has three Pods:

{{< codenew file="controllers/replicaset.yaml" >}}

If you create the ReplicaSet and then view the Pod metadata, you can see
OwnerReferences field:

```shell
kubectl apply -f https://k8s.io/examples/controllers/replicaset.yaml
kubectl get pods --output=yaml
```

The output shows that the Pod owner is a ReplicaSet named `my-repset`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

{{< note >}}
Cross-namespace owner references are disallowed by design. This means:
1) Namespace-scoped dependents can only specify owners in the same namespace,
and owners that are cluster-scoped.
2) Cluster-scoped dependents can only specify cluster-scoped owners, but not
namespace-scoped owners.
{{< /note >}}
 -->
## 所有者和从属者

有些 k8s 对象是其它对象的所有者。 如，一个 ReplicaSet 就是一个 Pod 集合的所有者。
那些被拥有的对象就叫所有者的 *从属者(dependents)*， 每个从属者都有一个 `metadata.ownerReferences`
字段指向它的所有者。

有时候， k8s 会自动添加 `ownerReference` 的值。 如，当创建一个 ReplicaSet 时， k8s 自动
为这个 ReplicaSet 所属的 Pod 设置 `ownerReference`。 在 `1.8` 中， k8s 为创建或捕获对象
自动设置 `ownerReference`值的对象有 ReplicationController, ReplicaSet, StatefulSet,
DaemonSet, Deployment, Job, CronJob.

用户可以通过设置 `ownerReference` 字段来手动指定所有者和从属者之间的关系
以下为一个包含 3 个 Pod 的 ReplicaSet 的配置文件:

{{< codenew file="controllers/replicaset.yaml" >}}

如果用户创建了 ReplicaSet 然后查看所属 Pod 的元数据(metadata), 就可以看到 `OwnerReferences` 字段

```shell
kubectl apply -f replicaset.yaml
kubectl get pods --output=yaml
```

从输出结果可以看到 Pod 的所有者是一个叫 `my-repset` 的 ReplicaSet

```yaml
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

{{< note >}}
在设计上跨名字空间的所属关系是不允许的。 这就是说:
1) 名字空间作用域内从属者只能指定同一个名字空间的对象为其所有者和集群作用域的所有者
2) 集群作用哉的从属者只能指定集群作用域的所有者，不能指定名字空间作用域的所有者
{{< /note >}}
<!--
## Controlling how the garbage collector deletes dependents

When you delete an object, you can specify whether the object's dependents are
also deleted automatically. Deleting dependents automatically is called *cascading
deletion*.  There are two modes of *cascading deletion*: *background* and *foreground*.

If you delete an object without deleting its dependents
automatically, the dependents are said to be *orphaned*.
 -->
## 控制垃圾回收器怎么删除从属者

当用户删除一个对象时，可能指定是否同时自动删除它的从属者。 自动删除从属都的行为叫做 *级联删除*。
级联删除又有两种模式 *后台* 和 *前台*。

如果用户在删除一个对象是没有自动删除它的所属者，这些从属者就认为是 *孤儿*
<!--
### Foreground cascading deletion

In *foreground cascading deletion*, the root object first
enters a "deletion in progress" state. In the "deletion in progress" state,
the following things are true:

 * The object is still visible via the REST API
 * The object's `deletionTimestamp` is set
 * The object's `metadata.finalizers` contains the value "foregroundDeletion".

Once the "deletion in progress" state is set, the garbage
collector deletes the object's dependents. Once the garbage collector has deleted all
"blocking" dependents (objects with `ownerReference.blockOwnerDeletion=true`), it deletes
the owner object.

Note that in the "foregroundDeletion", only dependents with
`ownerReference.blockOwnerDeletion=true` block the deletion of the owner object.
Kubernetes version 1.7 added an [admission controller](/docs/reference/access-authn-authz/admission-controllers/#ownerreferencespermissionenforcement) that controls user access to set
`blockOwnerDeletion` to true based on delete permissions on the owner object, so that
unauthorized dependents cannot delay deletion of an owner object.

If an object's `ownerReferences` field is set by a controller (such as Deployment or ReplicaSet),
blockOwnerDeletion is set automatically and you do not need to manually modify this field.
 -->
### 前台级联删除

In *foreground cascading deletion*, the root object first
enters a "deletion in progress" state. In the "deletion in progress" state,
the following things are true:

在使用 *前台级联删除* 时， 根对象先进入 "删除中" 状态。 在 "删除中" 状态时, 以下状态为真:

- 依然可以通过 REST API 访问该对象
- 该对象上已经设置了 `deletionTimestamp`
- 该对象上的 `metadata.finalizers` 包含值 "foregroundDeletion"

当 "删除中" 被设置后， 垃圾回收器就会删除它的从属对象。 当垃圾回收器删除所有 "阻塞" 的对象(包含 `ownerReference.blockOwnerDeletion=true` 的对象)后，
就会删除对象本身。

要注意到 *前台级联删除* 只会被那些包含 `ownerReference.blockOwnerDeletion=true` 对象的删除所阻塞。
在 k8s `v1.7` 添加了 [admission controller](/k8sDocs/reference/access-authn-authz/admission-controllers/#ownerreferencespermissionenforcement)
其上添加了基于所有者对象上的删除权限设置用户对 `blockOwnerDeletion` 设置为 true 的权限。
因此对于未授权的从属对象不会延迟其所有者对象的删除。

如果一个对象的 `ownerReferences` 字段是由控制器(如 Deployment 或 ReplicaSet) 设置的，
`blockOwnerDeletion` 也会自动被设置，用户不需要手动修改该字段。
<!--
### Background cascading deletion

In *background cascading deletion*, Kubernetes deletes the owner object
immediately and the garbage collector then deletes the dependents in
the background.
 -->
### 后台级联删除

在使用 *后台级联删除*, k8s 立马删除所有者对象，然后垃圾回器后台删除其从属对象
<!--
### Setting the cascading deletion policy

To control the cascading deletion policy, set the `propagationPolicy`
field on the `deleteOptions` argument when deleting an Object. Possible values include "Orphan",
"Foreground", or "Background".

Here's an example that deletes dependents in background:

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
  -H "Content-Type: application/json"
```

Here's an example that deletes dependents in foreground:

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```

Here's an example that orphans dependents:

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

kubectl also supports cascading deletion.
To delete dependents automatically using kubectl, set `--cascade` to true.  To
orphan dependents, set `--cascade` to false. The default value for `--cascade`
is true.

Here's an example that orphans the dependents of a ReplicaSet:

```shell
kubectl delete replicaset my-repset --cascade=false
```
 -->
### 设置级联删除策略

想要控制级联删除策略, 可以在删除对象时设置 `deleteOptions` 参数的 `propagationPolicy` 字段，
可选的值有 "Orphan", "Foreground", "Background".
以下为一个后台删除从属对象的示例:

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
  -H "Content-Type: application/json"
```

以下为一个前台删除从属对象的示例:

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```

以下为一个将从属对象设置为 孤儿的示例:
```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

kubectl 也支持级联删除。要使用 kubectl 自动删除从属对象， 设置 `--cascade` 为 `true`
将从属对象设置为 孤儿 设置 `--cascade` 为 `false`. `--cascade` 默认值为 `true`

以下为一个将 ReplicaSet 从属对象设置为 孤儿的示例
```shell
kubectl delete replicaset my-repset --cascade=false
```
<!--
### Additional note on Deployments

Prior to 1.7, When using cascading deletes with Deployments you *must* use `propagationPolicy: Foreground`
to delete not only the ReplicaSets created, but also their Pods. If this type of _propagationPolicy_
is not used, only the ReplicaSets will be deleted, and the Pods will be orphaned.
See [kubeadm/#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613) for more information.
 -->
### Deployment 额外需要注意的地方

Prior to 1.7, When using cascading deletes with Deployments you *must* use `propagationPolicy: Foreground`
to delete not only the ReplicaSets created, but also their Pods. If this type of _propagationPolicy_
is not used, only the ReplicaSets will be deleted, and the Pods will be orphaned.
See [kubeadm/#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613) for more information.

在 `1.7` 版本之前， 当使用级联删除 Deployment 时，*必须* 要使用 `propagationPolicy: Foreground`
来确定不止删除创建的  ReplicaSet，还要删除对应的 Pod。如果没有使用该类型的 _propagationPolicy_，
只有 ReplicaSet 会被删除，Pod 会被设置人孤儿。
更多信息见 [kubeadm/#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613)

## 已知问题

见问题单 [#26120](https://github.com/kubernetes/kubernetes/issues/26120)



## {{% heading "whatsnext" %}}


[设计文稿 1](https://git.k8s.io/community/contributors/design-proposals/api-machinery/garbage-collection.md)

[设计文稿 2](https://git.k8s.io/community/contributors/design-proposals/api-machinery/synchronous-garbage-collection.md)
