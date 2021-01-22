---
title: Pod 安全策略
content_type: concept
weight: 30
---
<!--
---
reviewers:
- pweil-
- tallclair
title: Pod Security Policies
content_type: concept
weight: 30
---
 -->
<!-- overview -->

{{< feature-state state="beta" >}}
<!--
Pod Security Policies enable fine-grained authorization of pod creation and
updates.
 -->
Pod 安全策略给予 Pod 的创建和更新更细粒度的授权。
<!-- body -->

<!--
## What is a Pod Security Policy?

A _Pod Security Policy_ is a cluster-level resource that controls security
sensitive aspects of the pod specification. The [PodSecurityPolicy](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podsecuritypolicy-v1beta1-policy) objects
define a set of conditions that a pod must run with in order to be accepted into
the system, as well as defaults for the related fields. They allow an
administrator to control the following:

| Control Aspect                                      | Field Names                                 |
| ----------------------------------------------------| ------------------------------------------- |
| Running of privileged containers                    | [`privileged`](#privileged)                                |
| Usage of host namespaces                            | [`hostPID`, `hostIPC`](#host-namespaces)    |
| Usage of host networking and ports                  | [`hostNetwork`, `hostPorts`](#host-namespaces) |
| Usage of volume types                               | [`volumes`](#volumes-and-file-systems)      |
| Usage of the host filesystem                        | [`allowedHostPaths`](#volumes-and-file-systems) |
| Allow specific FlexVolume drivers                   | [`allowedFlexVolumes`](#flexvolume-drivers) |
| Allocating an FSGroup that owns the pod's volumes   | [`fsGroup`](#volumes-and-file-systems)      |
| Requiring the use of a read only root file system   | [`readOnlyRootFilesystem`](#volumes-and-file-systems) |
| The user and group IDs of the container             | [`runAsUser`, `runAsGroup`, `supplementalGroups`](#users-and-groups) |
| Restricting escalation to root privileges           | [`allowPrivilegeEscalation`, `defaultAllowPrivilegeEscalation`](#privilege-escalation) |
| Linux capabilities                                  | [`defaultAddCapabilities`, `requiredDropCapabilities`, `allowedCapabilities`](#capabilities) |
| The SELinux context of the container                | [`seLinux`](#selinux)                       |
| The Allowed Proc Mount types for the container      | [`allowedProcMountTypes`](#allowedprocmounttypes) |
| The AppArmor profile used by containers             | [annotations](#apparmor)                    |
| The seccomp profile used by containers              | [annotations](#seccomp)                     |
| The sysctl profile used by containers               | [`forbiddenSysctls`,`allowedUnsafeSysctls`](#sysctl)                      |
 -->

## Pod 安全策略是啥 {#what-is-a-pod-security-policy}

_Pod 安全策略_ 是一个集群层的资源，它被用来控制 Pod 定义中的安全敏感方面的内容。
[PodSecurityPolicy](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podsecuritypolicy-v1beta1-policy) 对象是一个条件集合， 一个 Pod 只有以符合这些条件和与这些条件相关的默认设置的方式运行才能被系统接受。
它允许系统管理进行以下控制:

| 控制方面                      | 字段名称                                |
| ----------------------------| ------------------------------------------- |
| 运行特权容器                  | [`privileged`](#privileged)                                |
| 使用主机命名空间               | [`hostPID`, `hostIPC`](#host-namespaces)    |
| 使用主机网络和端口             | [`hostNetwork`, `hostPorts`](#host-namespaces) |
| 能够使用的卷类型               | [`volumes`](#volumes-and-file-systems)      |
| 能够使用主机文件系统           | [`allowedHostPaths`](#volumes-and-file-systems) |
| 允许设置 FlexVolume 驱动      | [`allowedFlexVolumes`](#flexvolume-drivers) |
| 分配一个包含 Pod 卷的 FSGroup  | [`fsGroup`](#volumes-and-file-systems)      |
| 要求使用一个只读根文件系统       | [`readOnlyRootFilesystem`](#volumes-and-file-systems) |
| 用户的 UID 和 GID             | [`runAsUser`, `runAsGroup`, `supplementalGroups`](#users-and-groups) |
| 限制升级到 root 权限           | [`allowPrivilegeEscalation`, `defaultAllowPrivilegeEscalation`](#privilege-escalation) |
| Linux 功能                    | [`defaultAddCapabilities`, `requiredDropCapabilities`, `allowedCapabilities`](#capabilities) |
| 容器的 SELinux 上下文           | [`seLinux`](#selinux)                       |
| 允许容器 Proc 挂载类型          | [`allowedProcMountTypes`](#allowedprocmounttypes) |
| 容器使用的 AppArmor 方案        | [annotations](#apparmor)                    |
| 容器使用的 seccomp 方案         | [annotations](#seccomp)                     |
| 容器使用的 sysctl 方案          | [`forbiddenSysctls`,`allowedUnsafeSysctls`](#sysctl)                      |

<!--
## Enabling Pod Security Policies

Pod security policy control is implemented as an optional (but recommended)
[admission
controller](/docs/reference/access-authn-authz/admission-controllers/#podsecuritypolicy). PodSecurityPolicies
are enforced by [enabling the admission
controller](/docs/reference/access-authn-authz/admission-controllers/#how-do-i-turn-on-an-admission-control-plug-in),
but doing so without authorizing any policies **will prevent any pods from being
created** in the cluster.

Since the pod security policy API (`policy/v1beta1/podsecuritypolicy`) is
enabled independently of the admission controller, for existing clusters it is
recommended that policies are added and authorized before enabling the admission
controller.
 -->

## 启用 Pod 安全策略 {#enabling-pod-security-policies}

Pod 安全策略控制是以一个可选(但推荐)的
[准入控制器](/docs/reference/access-authn-authz/admission-controllers/#podsecuritypolicy)
来实现的。 `PodSecurityPolicy` 是通过
[开启准入控制器](/docs/reference/access-authn-authz/admission-controllers/#how-do-i-turn-on-an-admission-control-plug-in)
来执行的，但如果只用 Pod 安全策略，而没有授权任意策略会使得集群 **阻止所有 Pod 的创建**。

因为 Pod 安全策略 API (`policy/v1beta1/podsecuritypolicy`) 的启用是独立于准入控制器的，
对象已经存在的集群建议在启用准入策略之间先添加策略和授权。

<!--
## Authorizing Policies

When a PodSecurityPolicy resource is created, it does nothing. In order to use
it, the requesting user or target pod's [service
account](/docs/tasks/configure-pod-container/configure-service-account/) must be
authorized to use the policy, by allowing the `use` verb on the policy.

Most Kubernetes pods are not created directly by users. Instead, they are
typically created indirectly as part of a
[Deployment](/docs/concepts/workloads/controllers/deployment/),
[ReplicaSet](/docs/concepts/workloads/controllers/replicaset/), or other
templated controller via the controller manager. Granting the controller access
to the policy would grant access for *all* pods created by that controller,
so the preferred method for authorizing policies is to grant access to the
pod's service account (see [example](#run-another-pod)).
 -->

## 对策略授权 {#authorizing-policies}

当一个 `PodSecurityPolicy` 资源被创建后，它是不会干事情的。要使用它， 请求用户或目标 Pod 的
[服务账号](/docs/tasks/configure-pod-container/configure-service-account/) 必须有
使用这个策略的授权，这个授权是能会在策略上使用 `use` 动词实现的。

大多数 k8s 中的 Pod 都不是由用户直接创建的。 而通常是通过控制器管理器作为
[Deployment](/docs/concepts/workloads/controllers/deployment/),
[ReplicaSet](/docs/concepts/workloads/controllers/replicaset/), 或其它模板控制器
的一部分间接创建的。 授予策略该控制器的访问权限也就是授予对该控制创建的 *所有* Pod 的访问权限，
所以授予策略权限的和冼方式是授予 Pod 的账号访问权限(见 [示例](#run-another-pod))

<!--
### Via RBAC

[RBAC](/docs/reference/access-authn-authz/rbac/) is a standard Kubernetes
authorization mode, and can easily be used to authorize use of policies.

First, a `Role` or `ClusterRole` needs to grant access to `use` the desired
policies. The rules to grant access look like this:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <role name>
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - <list of policies to authorize>
```

Then the `(Cluster)Role` is bound to the authorized user(s):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding name>
roleRef:
  kind: ClusterRole
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
subjects:
# Authorize specific service accounts:
- kind: ServiceAccount
  name: <authorized service account name>
  namespace: <authorized pod namespace>
# Authorize specific users (not recommended):
- kind: User
  apiGroup: rbac.authorization.k8s.io
  name: <authorized user name>
```

If a `RoleBinding` (not a `ClusterRoleBinding`) is used, it will only grant
usage for pods being run in the same namespace as the binding. This can be
paired with system groups to grant access to all pods run in the namespace:
```yaml
# Authorize all service accounts in a namespace:
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:serviceaccounts
# Or equivalently, all authenticated users in a namespace:
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:authenticated
```

For more examples of RBAC bindings, see [Role Binding
Examples](/docs/reference/access-authn-authz/rbac#role-binding-examples).
For a complete example of authorizing a PodSecurityPolicy, see
[below](#example).

 -->

### 通过 RBAC {#via-rbac}

[RBAC](/docs/reference/access-authn-authz/rbac/) is a standard Kubernetes
authorization mode, and can easily be used to authorize use of policies.

[RBAC](/docs/reference/access-authn-authz/rbac/)
是标准的 k8s 授权模式， 它可以轻松地用于授权策略的使用。

First, a `Role` or `ClusterRole` needs to grant access to `use` the desired
policies. The rules to grant access look like this:

首先， 一个 `Role` 或 `ClusterRole` 需要授权使用(`use`) 期望的策略。 授权的规则如下:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <role name>
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - <list of policies to authorize>
```

Then the `(Cluster)Role` is bound to the authorized user(s):
然后将 `(Cluster)Role` 绑定到授权的用户:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding name>
roleRef:
  kind: ClusterRole
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
subjects:
# 授权给指定服务账号
- kind: ServiceAccount
  name: <authorized service account name>
  namespace: <authorized pod namespace>
# 授权给指定用户(不推荐)
- kind: User
  apiGroup: rbac.authorization.k8s.io
  name: <authorized user name>
```

如果使用的是 `RoleBinding` (不是 `ClusterRoleBinding`)，它就只能给予运行在同一个命名空间
的 Pod 的使用作为绑定。 这也可以搭上系统组的方式授予对命名空间中所有 Pod 的访问权限:

```yaml
# 授权给命名空间中所有的服务账号
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:serviceaccounts
# 等效的，授权给命名空间中的所有授权用户
- kind: Group
  apiGroup: rbac.authorization.k8s.io
  name: system:authenticated
```

更多 RBAC 绑定的示例， 见
[角色绑定示例](/docs/reference/access-authn-authz/rbac#role-binding-examples).
对一个 `PodSecurityPolicy` 授权的完整示例， 见
[下面](#example).

<!--
### Troubleshooting

- The [controller manager](/docs/reference/command-line-tools-reference/kube-controller-manager/)
  must be run against the secured API port and must not have superuser permissions. See
  [Controlling Access to the Kubernetes API](/docs/concepts/security/controlling-access)
  to learn about API server access controls.  
  If the controller manager connected through the trusted API port (also known as the
  `localhost` listener), requests would bypass authentication and authorization modules;
  all PodSecurityPolicy objects would be allowed, and users would be able to create grant
  themselves the ability to create privileged containers.

  For more details on configuring controller manager authorization, see
  [Controller Roles](/docs/reference/access-authn-authz/rbac/#controller-roles).
 -->

### 故障排除 {#troubleshooting}

- [控制器管理器](/docs/reference/command-line-tools-reference/kube-controller-manager/)
  必须要对接一个安全的 API 端口，并且必须不能有超级用户权限。 学习 API 服务访问控制见
  [k8s API 访问控制](/docs/concepts/security/controlling-access)
  如果控制器管理器是通过受信的 API 端口(也就是我们知道的 `localhost` 监听器)连接的，请求就会
  绕过认证和授权模块； 所以的 PodSecurityPolicy 对象都是被允许的，用户也可以给自己授予创建
  特权容器的能力。

  更多关于配置控制器管理器授权的信息，见
  [控制器角色](/docs/reference/access-authn-authz/rbac/#controller-roles).

<!--
## Policy Order

In addition to restricting pod creation and update, pod security policies can
also be used to provide default values for many of the fields that it
controls. When multiple policies are available, the pod security policy
controller selects policies according to the following criteria:

1. PodSecurityPolicies which allow the pod as-is, without changing defaults or
mutating the pod, are preferred.  The order of these non-mutating
PodSecurityPolicies doesn't matter.
2. If the pod must be defaulted or mutated, the first PodSecurityPolicy
(ordered by name) to allow the pod is selected.

{{< note >}}
During update operations (during which mutations to pod specs are disallowed)
only non-mutating PodSecurityPolicies are used to validate the pod.
{{< /note >}}
 -->

## 策略顺序 {#policy-order}

在限制 Pod 的创建和修改外， Pod 安全策略也可以且用于为它控制的许多字段提供默认舉。 当有多个策略
存在时， Pod 安全策略控制权限下面的条件选择策略:

1. PodSecurityPolicy 首选是允许 Pod 保持原样，不用修改默认或修改 Pod。 这些不修改 Pod 的
   PodSecurityPolicy 的顺序是没有啥影响的。

2. 如果 Pod 必须要设置默认值或进行修改，允许 Pod 的第一个(以名称排序) PodSecurityPolicy
   会被选中。

{{< note >}}
在更新操作中(在对 Pod 定义不允许修改时) 只有不修改 Pod 的 PodSecurityPolicy 会被用于验证
Pod
{{< /note >}}

<!--
## Example

_This example assumes you have a running cluster with the PodSecurityPolicy
admission controller enabled and you have cluster admin privileges._
 -->

## 示例 {#example}

_这个示例假设运行的集群中启用了 PodSecurityPolicy 准入控制器并且用户有集群管理员权限_

<!--
### Set up

Set up a namespace and a service account to act as for this example. We'll use
this service account to mock a non-admin user.

```shell
kubectl create namespace psp-example
kubectl create serviceaccount -n psp-example fake-user
kubectl create rolebinding -n psp-example fake-editor --clusterrole=edit --serviceaccount=psp-example:fake-user
```

To make it clear which user we're acting as and save some typing, create 2
aliases:

```shell
alias kubectl-admin='kubectl -n psp-example'
alias kubectl-user='kubectl --as=system:serviceaccount:psp-example:fake-user -n psp-example'
```
 -->

### 开搞

设置一个命名空间和一个服务账号为在这个例子中使用。 我们会用这个服务账号扮演非管理员用户。

```shell
kubectl create namespace psp-example
kubectl create serviceaccount -n psp-example fake-user
kubectl create rolebinding -n psp-example fake-editor --clusterrole=edit --serviceaccount=psp-example:fake-user
```

为了清楚的显示我们是以哪个用户进行的操作，并省些敲字的工夫，创建 2 个别名:

```shell
alias kubectl-admin='kubectl -n psp-example'
alias kubectl-user='kubectl --as=system:serviceaccount:psp-example:fake-user -n psp-example'
```

<!--
### Create a policy and a pod

Define the example PodSecurityPolicy object in a file. This is a policy that
simply prevents the creation of privileged pods.
The name of a PodSecurityPolicy object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

{{< codenew file="policy/example-psp.yaml" >}}

And create it with kubectl:

```shell
kubectl-admin create -f example-psp.yaml
```

Now, as the unprivileged user, try to create a simple pod:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
EOF
```

The output is similar to this:

```
Error from server (Forbidden): error when creating "STDIN": pods "pause" is forbidden: unable to validate against any pod security policy: []
```

**What happened?** Although the PodSecurityPolicy was created, neither the
pod's service account nor `fake-user` have permission to use the new policy:

```shell
kubectl-user auth can-i use podsecuritypolicy/example
no
```

Create the rolebinding to grant `fake-user` the `use` verb on the example
policy:

{{< note >}}
This is not the recommended way! See the [next section](#run-another-pod)
for the preferred approach.
{{< /note >}}

```shell
kubectl-admin create role psp:unprivileged \
    --verb=use \
    --resource=podsecuritypolicy \
    --resource-name=example
role "psp:unprivileged" created

kubectl-admin create rolebinding fake-user:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:fake-user
rolebinding "fake-user:psp:unprivileged" created

kubectl-user auth can-i use podsecuritypolicy/example
yes
```

Now retry creating the pod:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
EOF
```

The output is similar to this

```
pod "pause" created
```

It works as expected! But any attempts to create a privileged pod should still
be denied:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: privileged
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
      securityContext:
        privileged: true
EOF
```

The output is similar to this:

```
Error from server (Forbidden): error when creating "STDIN": pods "privileged" is forbidden: unable to validate against any pod security policy: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

Delete the pod before moving on:

```shell
kubectl-user delete pod pause
```
 -->

### 创建一个策略和一个 Pod {#create-a-policy-and-a-pod}

在一个文件中定义这个示例的 PodSecurityPolicy 对象。 这个策略的作用就是防止特权容器的创建。
PodSecurityPolicy 对象的名称必须是一个有效的
[DNS 子域名](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

{{< codenew file="policy/example-psp.yaml" >}}

使用 kubectl 创建:

```shell
kubectl-admin create -f example-psp.yaml
```

此时，作为一个非特权用户，尝试创建一个简单的 Pod:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
EOF
```

输出结果类似如下:

```
Error from server (Forbidden): error when creating "STDIN": pods "pause" is forbidden: unable to validate against any pod security policy: []
```

**发生了啥?** 尽管创建了 PodSecurityPolicy， 但不管是 Pod 的服务账号或 `fake-user` 拥有
使用这个新策略的权限:

```shell
kubectl-user auth can-i use podsecuritypolicy/example
no
```

创建一个角色绑定来授予 `fake-user` 使用(`use`) 这个示例策略:

{{< note >}}
不推荐使用这种方式！首选方式见 [下一节](#run-another-pod)
{{< /note >}}

```shell
kubectl-admin create role psp:unprivileged \
    --verb=use \
    --resource=podsecuritypolicy \
    --resource-name=example

# 输出
role "psp:unprivileged" created

kubectl-admin create rolebinding fake-user:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:fake-user

# 输出
rolebinding "fake-user:psp:unprivileged" created

kubectl-user auth can-i use podsecuritypolicy/example
yes
```

现在再次尝试创建 Pod:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
EOF
```

输出结果类似如下:

```
pod "pause" created
```

结果与预期相符！但任何创建特权 Pod 的尝试都会被拒绝:

```shell
kubectl-user create -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: privileged
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
      securityContext:
        privileged: true
EOF
```

输出结果类似如下:

```
Error from server (Forbidden): error when creating "STDIN": pods "privileged" is forbidden: unable to validate against any pod security policy: [spec.containers[0].securityContext.privileged: Invalid value: true: Privileged containers are not allowed]
```

在往下之前先删除这个 Pod:

```shell
kubectl-user delete pod pause
```

<!--
### Run another pod

Let's try that again, slightly differently:

```shell
kubectl-user create deployment pause --image=k8s.gcr.io/pause
deployment "pause" created

kubectl-user get pods
No resources found.

kubectl-user get events | head -n 2
LASTSEEN   FIRSTSEEN   COUNT     NAME              KIND         SUBOBJECT                TYPE      REASON                  SOURCE                                  MESSAGE
1m         2m          15        pause-7774d79b5   ReplicaSet                            Warning   FailedCreate            replicaset-controller                   Error creating: pods "pause-7774d79b5-" is forbidden: no providers available to validate pod request
```

**What happened?** We already bound the `psp:unprivileged` role for our `fake-user`,
why are we getting the error `Error creating: pods "pause-7774d79b5-" is
forbidden: no providers available to validate pod request`? The answer lies in
the source - `replicaset-controller`. Fake-user successfully created the
deployment (which successfully created a replicaset), but when the replicaset
went to create the pod it was not authorized to use the example
podsecuritypolicy.

In order to fix this, bind the `psp:unprivileged` role to the pod's service
account instead. In this case (since we didn't specify it) the service account
is `default`:

```shell
kubectl-admin create rolebinding default:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:default
rolebinding "default:psp:unprivileged" created
```

Now if you give it a minute to retry, the replicaset-controller should
eventually succeed in creating the pod:

```shell
kubectl-user get pods --watch
NAME                    READY     STATUS    RESTARTS   AGE
pause-7774d79b5-qrgcb   0/1       Pending   0         1s
pause-7774d79b5-qrgcb   0/1       Pending   0         1s
pause-7774d79b5-qrgcb   0/1       ContainerCreating   0         1s
pause-7774d79b5-qrgcb   1/1       Running   0         2s
```
 -->

### 运行另一个 Pod {#Run-another-pod}

我们再来一次稍微不同的尝试:

```shell
kubectl-user create deployment pause --image=k8s.gcr.io/pause
# 输出
deployment "pause" created

kubectl-user get pods
# 输出
No resources found.

kubectl-user get events | head -n 2
# 输出
LASTSEEN   FIRSTSEEN   COUNT     NAME              KIND         SUBOBJECT                TYPE      REASON                  SOURCE                                  MESSAGE
1m         2m          15        pause-7774d79b5   ReplicaSet                            Warning   FailedCreate            replicaset-controller                   Error creating: pods "pause-7774d79b5-" is forbidden: no providers available to validate pod request
```

**发生了啥?**  我们已经将角色 `psp:unprivileged` 绑定到用户 `fake-user`， 为啥还得到错误
信息 `Error creating: pods "pause-7774d79b5-" is forbidden:
no providers available to validate pod request`? 原因在来源 - `replicaset-controller`.
测试用户成功地创建了 Deployment(也就是成功创建了一个 ReplcatSet)，但当 ReplcatSet 开始
创建 Pod 时， 它没有被授权使用示例的 Pod 安全策略

而要修复这个问题， 将 `psp:unprivileged` 绑定到 Pod 的服务账号上。 在这种情况下(因为我们没有指定)
那服务账号就是 `default`:

```shell
kubectl-admin create rolebinding default:psp:unprivileged \
    --role=psp:unprivileged \
    --serviceaccount=psp-example:default
# 输出
rolebinding "default:psp:unprivileged" created
```

这时再重试， `replicaset-controller` 应该最终可以成功创建这个 Pod:

```shell
kubectl-user get pods --watch
NAME                    READY     STATUS    RESTARTS   AGE
pause-7774d79b5-qrgcb   0/1       Pending   0         1s
pause-7774d79b5-qrgcb   0/1       Pending   0         1s
pause-7774d79b5-qrgcb   0/1       ContainerCreating   0         1s
pause-7774d79b5-qrgcb   1/1       Running   0         2s
```

### 收尾 {#clean-up}

删除命名空间，清楚示例中的大多数资源：

```shell
kubectl-admin delete ns psp-example
# 输出
namespace "psp-example" deleted
```

要注意 `PodSecurityPolicy` 资源是没有命名空间的，需要单独清除:

```shell
kubectl-admin delete psp example
# 输出
podsecuritypolicy "example" deleted
```

<!--
### Example Policies

This is the least restrictive policy you can create, equivalent to not using the
pod security policy admission controller:

{{< codenew file="policy/privileged-psp.yaml" >}}

This is an example of a restrictive policy that requires users to run as an
unprivileged user, blocks possible escalations to root, and requires use of
several security mechanisms.

{{< codenew file="policy/restricted-psp.yaml" >}}

See [Pod Security Standards](/docs/concepts/security/pod-security-standards/#policy-instantiation) for more examples.
 -->

### 示例策略 {#example-policies}

这是一个用户可以创建最没有约束性的策略， 等效于不使用 Pod 安全策略准入控制:

{{< codenew file="policy/privileged-psp.yaml" >}}

这是一个有约束的策略，它要求用户以非特别用户运行，阻止可能提升为 root, 并要求使用几个安全机制。
{{< codenew file="policy/restricted-psp.yaml" >}}

更多示例见 [Pod 安全标准](/docs/concepts/security/pod-security-standards/#policy-instantiation).

<!--
## Policy Reference

### Privileged

**Privileged** - determines if any container in a pod can enable privileged mode.
By default a container is not allowed to access any devices on the host, but a
"privileged" container is given access to all devices on the host. This allows
the container nearly all the same access as processes running on the host.
This is useful for containers that want to use linux capabilities like
manipulating the network stack and accessing devices.
 -->
## 策略使用参数 {#policy-reference}

### Privileged

**Privileged** - 决定容器中的任意 Pod 是否可以启用特权模式。默认情况下容器是不允许访问主机上
的任意设备的，但 "特权(privileged)" 是可以访问宿主机上的所有设备的。 这就允许容器可以与运行在
主机上的进程拥有几乎相同的访问权限。 这对于想要使用 Linux 功能，如操作网络栈和访问设备是很有用的。

<!--
### Host namespaces

**HostPID** - Controls whether the pod containers can share the host process ID
namespace. Note that when paired with ptrace this can be used to escalate
privileges outside of the container (ptrace is forbidden by default).

**HostIPC** - Controls whether the pod containers can share the host IPC
namespace.

**HostNetwork** - Controls whether the pod may use the node network
namespace. Doing so gives the pod access to the loopback device, services
listening on localhost, and could be used to snoop on network activity of other
pods on the same node.

**HostPorts** - Provides a list of ranges of allowable ports in the host
network namespace. Defined as a list of `HostPortRange`, with `min`(inclusive)
and `max`(inclusive). Defaults to no allowed host ports.
 -->

### 宿主机命名空间 {#host-namespaces}

**HostPID** - Controls whether the pod containers can share the host process ID
namespace. Note that when paired with ptrace this can be used to escalate
privileges outside of the container (ptrace is forbidden by default).

**HostPID** - 控制
**HostIPC** - Controls whether the pod containers can share the host IPC
namespace.

**HostNetwork** - Controls whether the pod may use the node network
namespace. Doing so gives the pod access to the loopback device, services
listening on localhost, and could be used to snoop on network activity of other
pods on the same node.

**HostPorts** - Provides a list of ranges of allowable ports in the host
network namespace. Defined as a list of `HostPortRange`, with `min`(inclusive)
and `max`(inclusive). Defaults to no allowed host ports.

### Volumes and file systems

**Volumes** - Provides a list of allowed volume types. The allowable values
correspond to the volume sources that are defined when creating a volume. For
the complete list of volume types, see [Types of
Volumes](/docs/concepts/storage/volumes/#types-of-volumes). Additionally, `*`
may be used to allow all volume types.

The **recommended minimum set** of allowed volumes for new PSPs are:

- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- secret
- projected

{{< warning >}}
PodSecurityPolicy does not limit the types of `PersistentVolume` objects that
may be referenced by a `PersistentVolumeClaim`, and hostPath type
`PersistentVolumes` do not support read-only access mode. Only trusted users
should be granted permission to create `PersistentVolume` objects.
{{< /warning >}}

**FSGroup** - Controls the supplemental group applied to some volumes.

- *MustRunAs* - Requires at least one `range` to be specified. Uses the
minimum value of the first range as the default. Validates against all ranges.
- *MayRunAs* - Requires at least one `range` to be specified. Allows
`FSGroups` to be left unset without providing a default. Validates against
all ranges if `FSGroups` is set.
- *RunAsAny* - No default provided. Allows any `fsGroup` ID to be specified.

**AllowedHostPaths** - This specifies a list of host paths that are allowed
to be used by hostPath volumes. An empty list means there is no restriction on
host paths used. This is defined as a list of objects with a single `pathPrefix`
field, which allows hostPath volumes to mount a path that begins with an
allowed prefix, and a `readOnly` field indicating it must be mounted read-only.
For example:

```yaml
allowedHostPaths:
  # This allows "/foo", "/foo/", "/foo/bar" etc., but
  # disallows "/fool", "/etc/foo" etc.
  # "/foo/../" is never valid.
  - pathPrefix: "/foo"
    readOnly: true # only allow read-only mounts
```

{{< warning >}}There are many ways a container with unrestricted access to the host
filesystem can escalate privileges, including reading data from other
containers, and abusing the credentials of system services, such as Kubelet.

Writeable hostPath directory volumes allow containers to write
to the filesystem in ways that let them traverse the host filesystem outside the `pathPrefix`.
`readOnly: true`, available in Kubernetes 1.11+, must be used on **all** `allowedHostPaths`
to effectively limit access to the specified `pathPrefix`.
{{< /warning >}}

**ReadOnlyRootFilesystem** - Requires that containers must run with a read-only
root filesystem (i.e. no writable layer).

### FlexVolume drivers

This specifies a list of FlexVolume drivers that are allowed to be used
by flexvolume. An empty list or nil means there is no restriction on the drivers.
Please make sure [`volumes`](#volumes-and-file-systems) field contains the
`flexVolume` volume type; no FlexVolume driver is allowed otherwise.

For example:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: allow-flex-volumes
spec:
  # ... other spec fields
  volumes:
    - flexVolume
  allowedFlexVolumes:
    - driver: example/lvm
    - driver: example/cifs
```

### Users and groups

**RunAsUser** - Controls which user ID the containers are run with.

- *MustRunAs* - Requires at least one `range` to be specified. Uses the
minimum value of the first range as the default. Validates against all ranges.
- *MustRunAsNonRoot* - Requires that the pod be submitted with a non-zero
`runAsUser` or have the `USER` directive defined (using a numeric UID) in the
image. Pods which have specified neither `runAsNonRoot` nor `runAsUser` settings
will be mutated to set `runAsNonRoot=true`, thus requiring a defined non-zero
numeric `USER` directive in the container. No default provided. Setting
`allowPrivilegeEscalation=false` is strongly recommended with this strategy.
- *RunAsAny* - No default provided. Allows any `runAsUser` to be specified.

**RunAsGroup** - Controls which primary group ID the containers are run with.

- *MustRunAs* - Requires at least one `range` to be specified. Uses the
minimum value of the first range as the default. Validates against all ranges.
- *MayRunAs* - Does not require that RunAsGroup be specified. However, when RunAsGroup
is specified, they have to fall in the defined range.
- *RunAsAny* - No default provided. Allows any `runAsGroup` to be specified.


**SupplementalGroups** - Controls which group IDs containers add.

- *MustRunAs* - Requires at least one `range` to be specified. Uses the
minimum value of the first range as the default. Validates against all ranges.
- *MayRunAs* - Requires at least one `range` to be specified. Allows
`supplementalGroups` to be left unset without providing a default.
Validates against all ranges if `supplementalGroups` is set.
- *RunAsAny* - No default provided. Allows any `supplementalGroups` to be
specified.

### Privilege Escalation

These options control the `allowPrivilegeEscalation` container option. This bool
directly controls whether the
[`no_new_privs`](https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt)
flag gets set on the container process. This flag will prevent `setuid` binaries
from changing the effective user ID, and prevent files from enabling extra
capabilities (e.g. it will prevent the use of the `ping` tool). This behavior is
required to effectively enforce `MustRunAsNonRoot`.

**AllowPrivilegeEscalation** - Gates whether or not a user is allowed to set the
security context of a container to `allowPrivilegeEscalation=true`. This
defaults to allowed so as to not break setuid binaries. Setting it to `false`
ensures that no child process of a container can gain more privileges than its parent.

**DefaultAllowPrivilegeEscalation** - Sets the default for the
`allowPrivilegeEscalation` option. The default behavior without this is to allow
privilege escalation so as to not break setuid binaries. If that behavior is not
desired, this field can be used to default to disallow, while still permitting
pods to request `allowPrivilegeEscalation` explicitly.

### Capabilities

Linux capabilities provide a finer grained breakdown of the privileges
traditionally associated with the superuser. Some of these capabilities can be
used to escalate privileges or for container breakout, and may be restricted by
the PodSecurityPolicy. For more details on Linux capabilities, see
[capabilities(7)](http://man7.org/linux/man-pages/man7/capabilities.7.html).

The following fields take a list of capabilities, specified as the capability
name in ALL_CAPS without the `CAP_` prefix.

**AllowedCapabilities** - Provides a list of capabilities that are allowed to be added
to a container. The default set of capabilities are implicitly allowed. The
empty set means that no additional capabilities may be added beyond the default
set. `*` can be used to allow all capabilities.

**RequiredDropCapabilities** - The capabilities which must be dropped from
containers. These capabilities are removed from the default set, and must not be
added. Capabilities listed in `RequiredDropCapabilities` must not be included in
`AllowedCapabilities` or `DefaultAddCapabilities`.

**DefaultAddCapabilities** - The capabilities which are added to containers by
default, in addition to the runtime defaults. See the [Docker
documentation](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)
for the default list of capabilities when using the Docker runtime.

### SELinux

- *MustRunAs* - Requires `seLinuxOptions` to be configured. Uses
`seLinuxOptions` as the default. Validates against `seLinuxOptions`.
- *RunAsAny* - No default provided. Allows any `seLinuxOptions` to be
specified.

### AllowedProcMountTypes

`allowedProcMountTypes` is a list of allowed ProcMountTypes.
Empty or nil indicates that only the `DefaultProcMountType` may be used.

`DefaultProcMount` uses the container runtime defaults for readonly and masked
paths for /proc.  Most container runtimes mask certain paths in /proc to avoid
accidental security exposure of special devices or information. This is denoted
as the string `Default`.

The only other ProcMountType is `UnmaskedProcMount`, which bypasses the
default masking behavior of the container runtime and ensures the newly
created /proc the container stays intact with no modifications. This is
denoted as the string `Unmasked`.

### AppArmor

Controlled via annotations on the PodSecurityPolicy. Refer to the [AppArmor
documentation](/docs/tutorials/clusters/apparmor/#podsecuritypolicy-annotations).

### Seccomp

As of Kubernetes v1.19, you can use the `seccompProfile` field in the
`securityContext` of Pods or containers to [control use of seccomp
profiles](/docs/tutorials/clusters/seccomp). In prior versions, seccomp was
controlled by adding annotations to a Pod. The same PodSecurityPolicies can be
used with either version to enforce how these fields or annotations are applied.

**seccomp.security.alpha.kubernetes.io/defaultProfileName** - Annotation that
specifies the default seccomp profile to apply to containers. Possible values
are:

- `unconfined` - Seccomp is not applied to the container processes (this is the
  default in Kubernetes), if no alternative is provided.
- `runtime/default` - The default container runtime profile is used.
- `docker/default` - The Docker default seccomp profile is used. Deprecated as
  of Kubernetes 1.11. Use `runtime/default` instead.
- `localhost/<path>` - Specify a profile as a file on the node located at
  `<seccomp_root>/<path>`, where `<seccomp_root>` is defined via the
  `--seccomp-profile-root` flag on the Kubelet. If the `--seccomp-profile-root`
  flag is not defined, the default path will be used, which is
  `<root-dir>/seccomp` where `<root-dir>` is specified by the `--root-dir` flag.

{{< note >}}
  The `--seccomp-profile-root` flag is deprecated since Kubernetes
  v1.19. Users are encouraged to use the default path.
{{< /note >}}

**seccomp.security.alpha.kubernetes.io/allowedProfileNames** - Annotation that
specifies which values are allowed for the pod seccomp annotations. Specified as
a comma-delimited list of allowed values. Possible values are those listed
above, plus `*` to allow all profiles. Absence of this annotation means that the
default cannot be changed.

### Sysctl

By default, all safe sysctls are allowed.

- `forbiddenSysctls` - excludes specific sysctls. You can forbid a combination of safe and unsafe sysctls in the list. To forbid setting any sysctls, use `*` on its own.
- `allowedUnsafeSysctls` - allows specific sysctls that had been disallowed by the default list, so long as these are not listed in `forbiddenSysctls`.

Refer to the [Sysctl documentation](
/docs/tasks/administer-cluster/sysctl-cluster/#podsecuritypolicy).

## {{% heading "whatsnext" %}}

- See [Pod Security Standards](/docs/concepts/security/pod-security-standards/) for policy recommendations.

- Refer to [Pod Security Policy Reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podsecuritypolicy-v1beta1-policy) for the api details.
