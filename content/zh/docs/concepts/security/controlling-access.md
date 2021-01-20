---
title: 对 k8s API 的访问控制
content_type: concept
---
<!--
---
reviewers:
- erictune
- lavalamp
title: Controlling Access to the Kubernetes API
content_type: concept
---
 -->
<!-- overview -->
<!--
This page provides an overview of controlling access to the Kubernetes API.
 -->
本文概述对 k8s API 的访问控制
<!-- body -->
<!--
Users access the [Kubernetes API](/docs/concepts/overview/kubernetes-api/) using `kubectl`,
client libraries, or by making REST requests.  Both human users and
[Kubernetes service accounts](/docs/tasks/configure-pod-container/configure-service-account/) can be
authorized for API access.
When a request reaches the API, it goes through several stages, illustrated in the
following diagram:
 -->
用户可以使用 `kubectl`, 客户端库，或发送 REST 请求来访问
[Kubernetes API](/k8sDocs/docs/concepts/overview/kubernetes-api/). 用户和
[k8s 服务账号](/k8sDocs/docs/tasks/configure-pod-container/configure-service-account/)
都可以授权访问 API。 当一个请求到达 API 时，会经过几个阶段，如下图所示

![k8s API 请求的处理步骤图](/k8sDocs/images/docs/admin/access-control-overview.svg)

<!--
## Transport security

In a typical Kubernetes cluster, the API serves on port 443, protected by TLS.
The API server presents a certificate. This certificate may be signed using
a private certificate authority (CA), or based on a public key infrastructure linked
to a generally recognized CA.

If your cluster uses a private certificate authority, you need a copy of that CA
certifcate configured into your `~/.kube/config` on the client, so that you can
trust the connection and be confident it was not intercepted.

Your client can present a TLS client certificate at this stage.
 -->

## 传输安全 {#transport-security}

在一个典型的 k8s 集群中，API 使用 443 端口提供服务，由 TLS 保护。 API 服务提供一个证书。这个
证书可能使用一个私人证书颁发机构(CA), 或基于一个与公认 CA 连接的公钥签发的。

如果集群使用了一个 私人证书颁发机构， 需要将 CA 证书拷贝到客户机的  `~/.kube/config` 中，这样
才能信任这个连接并相邻这个连接不会被拦截。

在这一步，客户端也可以提供一个 TLS 客户端证书。

<!--
## Authentication

Once TLS is established, the HTTP request moves to the Authentication step.
This is shown as step **1** in the diagram.
The cluster creation script or cluster admin configures the API server to run
one or more Authenticator modules.
Authenticators are described in more detail in
[Authentication](/docs/reference/access-authn-authz/authentication/).

The input to the authentication step is the entire HTTP request; however, it typically
just examines the headers and/or client certificate.

Authentication modules include client certificates, password, and plain tokens,
bootstrap tokens, and JSON Web Tokens (used for service accounts).

Multiple authentication modules can be specified, in which case each one is tried in sequence,
until one of them succeeds.

If the request cannot be authenticated, it is rejected with HTTP status code 401.
Otherwise, the user is authenticated as a specific `username`, and the user name
is available to subsequent steps to use in their decisions.  Some authenticators
also provide the group memberships of the user, while other authenticators
do not.

While Kubernetes uses usernames for access control decisions and in request logging,
it does not have a `User` object nor does it store usernames or other information about
users in its API.
 -->

## 认证 {#authentication}

当 TLS 建立后， HTTP 请求就进行到认证这一步。也就是前面图片中的步骤 **1** 。
集群创建脚本或集群管理会让 API 服务运行一个或多个认证器模块。更多关于认证器的信息见
[Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

认证步骤的输入就整个 HTTP 请求； 但是，它通常只会检查头和/或 客户端证书。

认证模块包括和客户端证书，密码，和普通令牌， 引导令牌，JSON Web令牌(用于服务账号)。

可以指定多个认证模块，在这种情况下他们会按顺序逐个尝试直到有一个成功(也可能会者失败)

如果请求认证失败，则会使用 `401` HTTP 状态码以示拒绝。 否则用户以指定 `username` 来认证，这个
用户名就会被用到接下来的步骤中的决策中。 有些认证器也会提供用户的组信息，有些认证器则不会。

当 k8s 在使用用户名作为访问控制决策和在请求日志中，是不会有一个 `User` 对象，也不会在 API 存储用户名
或其它关于用户的信息的。

<!--
## Authorization

After the request is authenticated as coming from a specific user, the request must be authorized. This is shown as step **2** in the diagram.

A request must include the username of the requester, the requested action, and the object affected by the action. The request is authorized if an existing policy declares that the user has permissions to complete the requested action.

For example, if Bob has the policy below, then he can read pods only in the namespace `projectCaribou`:

```json
{
    "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
    "kind": "Policy",
    "spec": {
        "user": "bob",
        "namespace": "projectCaribou",
        "resource": "pods",
        "readonly": true
    }
}
```
If Bob makes the following request, the request is authorized because he is allowed to read objects in the `projectCaribou` namespace:

```json
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "spec": {
    "resourceAttributes": {
      "namespace": "projectCaribou",
      "verb": "get",
      "group": "unicorn.example.org",
      "resource": "pods"
    }
  }
}
```
If Bob makes a request to write (`create` or `update`) to the objects in the `projectCaribou` namespace, his authorization is denied. If Bob makes a request to read (`get`) objects in a different namespace such as `projectFish`, then his authorization is denied.

Kubernetes authorization requires that you use common REST attributes to interact with existing organization-wide or cloud-provider-wide access control systems. It is important to use REST formatting because these control systems might interact with other APIs besides the Kubernetes API.

Kubernetes supports multiple authorization modules, such as ABAC mode, RBAC Mode, and Webhook mode. When an administrator creates a cluster, they configure the authorization modules that should be used in the API server. If more than one authorization modules are configured, Kubernetes checks each module, and if any module authorizes the request, then the request can proceed. If all of the modules deny the request, then the request is denied (HTTP status code 403).

To learn more about Kubernetes authorization, including details about creating policies using the supported authorization modules, see [Authorization](/docs/reference/access-authn-authz/authorization/).
 -->


## 授权 {#authorization}

当来自指定用户的请求认证后，这个请求必须要被授权。 这就是图片中的步骤 **2**。

请求必须包含请求者的用户名，请求行为，和受行为影响的对象。 如果有存在的策略定义这个用户有完成这个
请求行为的权限就表示请求被授权了。

例如， 如果 Bob 拥有以下策略，则他只能读取命名空间 `projectCaribou` 中的 Pod :


```json
{
    "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
    "kind": "Policy",
    "spec": {
        "user": "bob",
        "namespace": "projectCaribou",
        "resource": "pods",
        "readonly": true
    }
}
```
如果 Bob 发起以下请求，这个请求会被授权，因为他被允许读取 命名空间 `projectCaribou` 中的 Pod :

```json
{
  "apiVersion": "authorization.k8s.io/v1beta1",
  "kind": "SubjectAccessReview",
  "spec": {
    "resourceAttributes": {
      "namespace": "projectCaribou",
      "verb": "get",
      "group": "unicorn.example.org",
      "resource": "pods"
    }
  }
}
```

如果 Bob 发起一个写请求(`create` 或 `update`)到命名空间 `projectCaribou` 中的对象，他的
授权就会被拒绝的。 如果 Bob 发起一个读请求(`get`)到另个名称空间如 `projectFish` 中的对象，
这时他的授权也会被拒绝。

k8s 授权要求用户使用通用 REST 属性与存在的组织内或云提供商内的访问控制系统交互。使用 REST 格式
很重要是因为这些控制系统可能与 k8s API 外的其它 API 进行交互。

k8s 支持多种授权模块，如 ABAC 模式，RBAC 模式，和 Webhook 模式。 当管理员在创建集群时，他们
每配置 API 服务中使用的授权模块。 如果配置了多个授权模块， k8s 会检查每个模块， 如果有任意一个
模块对这个模块授权，则这个请求就能继续。 如果所有的请求都被拒绝，则这个请求就会被拒绝(HTTP 状态码 403)。

要学习更多 k8s 授权， 包含使用支持的授权模块创建策略， 见
[Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/).

<!--
## Admission control

Admission Control modules are software modules that can modify or reject requests.
In addition to the attributes available to Authorization modules, Admission
Control modules can access the contents of the object that is being created or modified.

Admission controllers act on requests that create, modify, delete, or connect to (proxy) an object.
Admission controllers do not act on requests that merely read objects.
When multiple admission controllers are configured, they are called in order.

This is shown as step **3** in the diagram.

Unlike Authentication and Authorization modules, if any admission controller module
rejects, then the request is immediately rejected.

In addition to rejecting objects, admission controllers can also set complex defaults for
fields.

The available Admission Control modules are described in [Admission Controllers](/docs/reference/access-authn-authz/admission-controllers/).

Once a request passes all admission controllers, it is validated using the validation routines
for the corresponding API object, and then written to the object store (shown as step **4**).
 -->

## 准入控制 {#admission-control}

准入控制模块是软件模块，它可以修改或拒绝请求。 相对于授权模块能够访问的属性， 准入控制模块可以
访问请求创建或修改的对象的内容。

准入控制器对对创建，修改，删除或连接(代理)的对象执行操作。 准入控制器不会对只读对象的请求执行操作。
当配置了多个准入控制器时，他们会以顺序调用。

这就是图片中的步骤 **3**

与认证与授权模块不同，如果有任意准入控制器模块拒绝，则这个请求立马就会被拒绝。

在拒绝对象个，准入控制器也可以设置复杂的默认字段。

可以使用的准入控制模块描述见
[准入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).

当请求通过了所有准入控制器，会对对应的 API 对象使用验证程序来对其验证，然后将这个对象写入存储(如步骤 **4** 所示).

{{<todo-optimize>}}

<!--
## API server ports and IPs

The previous discussion applies to requests sent to the secure port of the API server
(the typical case).  The API server can actually serve on 2 ports:

By default the Kubernetes API server serves HTTP on 2 ports:

  1. `localhost` port:

      - is intended for testing and bootstrap, and for other components of the master node
        (scheduler, controller-manager) to talk to the API
      - no TLS
      - default is port 8080, change with `--insecure-port` flag.
      - default IP is localhost, change with `--insecure-bind-address` flag.
      - request **bypasses** authentication and authorization modules.
      - request handled by admission control module(s).
      - protected by need to have host access

  2. “Secure port”:

      - use whenever possible
      - uses TLS.  Set cert with `--tls-cert-file` and key with `--tls-private-key-file` flag.
      - default is port 6443, change with `--secure-port` flag.
      - default IP is first non-localhost network interface, change with `--bind-address` flag.
      - request handled by authentication and authorization modules.
      - request handled by admission control module(s).
      - authentication and authorization modules run.
 -->

## API 服务的端口与 IP {#api-server-ports-and-ips}

The previous discussion applies to requests sent to the secure port of the API server
(the typical case).  The API server can actually serve on 2 ports:
上面讨论中执行的请求是发送到 API 服务的安全端口的(典型场景). API 服务实际上是可以使用 2 个端口提供服务:

By default the Kubernetes API server serves HTTP on 2 ports:
默认情况下 k8s API 服务使用 2 个端口提供 HTTP 服务:

1. `localhost` 端口:
  - 主要用于测试和引导，也可以用于主节点上的其它模块(scheduler, controller-manager) 与 API 通信
  - 没有 TLS
  - 默认端口是 8080，使用 `--insecure-port` 标志修改
  - 默认 IP 是 `localhost`， 使用 `--insecure-bind-address` 标志修改
  - 请求 **绕过** 认证和授权模块。
  - 请求会被准入控制模块处理
  - 保护方式是需要主机访问

2. "安全端口":
  - 尽量无论何时都使用
  - 使用 TLS 使用 `--tls-cert-file` 使用证书，使用 `--tls-private-key-file` 使用密钥
  - 默认端口是 6443， 使用 `--secure-port` 标志修改
  - 默认 IP 首先是一个非 localhost 网络接口， 使用 `--bind-address` 标志修改
  - 请求会被认证和授权模块处理
  - 请求会被准入控制模块处理
  - 认证和授权模块会运行


## {{% heading "whatsnext" %}}

<!--

Read more documentation on authentication, authorization and API access control:

- [Authenticating](/docs/reference/access-authn-authz/authentication/)
   - [Authenticating with Bootstrap Tokens](/docs/reference/access-authn-authz/bootstrap-tokens/)
- [Admission Controllers](/docs/reference/access-authn-authz/admission-controllers/)
   - [Dynamic Admission Control](/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Authorization](/docs/reference/access-authn-authz/authorization/)
   - [Role Based Access Control](/docs/reference/access-authn-authz/rbac/)
   - [Attribute Based Access Control](/docs/reference/access-authn-authz/abac/)
   - [Node Authorization](/docs/reference/access-authn-authz/node/)
   - [Webhook Authorization](/docs/reference/access-authn-authz/webhook/)
- [Certificate Signing Requests](/docs/reference/access-authn-authz/certificate-signing-requests/)
   - including [CSR approval](/docs/reference/access-authn-authz/certificate-signing-requests/#approval-rejection)
     and [certificate signing](/docs/reference/access-authn-authz/certificate-signing-requests/#signing)
- Service accounts
  - [Developer guide](/docs/tasks/configure-pod-container/configure-service-account/)
  - [Administration](/docs/reference/access-authn-authz/service-accounts-admin/)

You can learn about:
- how Pods can use
  [Secrets](/docs/concepts/configuration/secret/#service-accounts-automatically-create-and-attach-secrets-with-api-credentials)
  to obtain API credentials.
 -->

更多关于 认证，授权， API 准入控制的文档:

- [Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
   - [Authenticating with Bootstrap Tokens](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/)
- [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
   - [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)
   - [Role Based Access Control](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
   - [Attribute Based Access Control](https://kubernetes.io/docs/reference/access-authn-authz/abac/)
   - [Node Authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
   - [Webhook Authorization](https://kubernetes.io/docs/reference/access-authn-authz/webhook/)
- [Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
   - including [CSR approval](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#approval-rejection)
     and [certificate signing](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#signing)
- 服务账号
  - [开发者引导](/k8sDocs/docs/tasks/configure-pod-container/configure-service-account/)
  - [Administration](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)

还可以学习:
- Pod 怎么使用
  [Secret](/k8sDocs/docs/concepts/configuration/secret/#service-accounts-automatically-create-and-attach-secrets-with-api-credentials)
  获取 API 凭据
