---
title: Secret
content_type: concept
weight: 30
---
<!--
---
reviewers:
- mikedanese
title: Secrets
content_type: concept
feature:
  title: Secret and configuration management
  description: >
    Deploy and update secrets and application configuration without rebuilding your image and without exposing secrets in your stack configuration.
weight: 30
---
 -->
<!-- overview -->
<!--
Kubernetes Secrets let you store and manage sensitive information, such
as passwords, OAuth tokens, and ssh keys. Storing confidential information in a Secret
is safer and more flexible than putting it verbatim in a
{{< glossary_tooltip term_id="pod" >}} definition or in a
{{< glossary_tooltip text="container image" term_id="image" >}}.
See [Secrets design document](https://git.k8s.io/community/contributors/design-proposals/auth/secrets.md) for more information.

A Secret is an object that contains a small amount of sensitive data such as
a password, a token, or a key. Such information might otherwise be put in a
Pod specification or in an image. Users can create Secrets and the system
also creates some Secrets.
 -->

k8s Secret 可以让用户存储和管理敏感信息，如密码, OAuth token, ssh 密钥。将私密信息放在
Secret 中更安全，并且比直接放在
{{< glossary_tooltip term_id="pod" >}}
的配置定义中或
{{< glossary_tooltip term_id="image" >}}
中更加灵活。
更多信息见
[Secrets 设计文稿](https://git.k8s.io/community/contributors/design-proposals/auth/secrets.md)

Secret 一个包含少量敏感信息如 如密码, OAuth token, ssh 密钥的对象。 这些信息也能被放在
Pod 的配置定义或镜像中。 用户可以创建 Secret，系统也会创建一些 Secret。

<!-- body -->
<!--
## Overview of Secrets

To use a Secret, a Pod needs to reference the Secret.
A Secret can be used with a Pod in three ways:

- As [files](#using-secrets-as-files-from-a-pod) in a
  {{< glossary_tooltip text="volume" term_id="volume" >}} mounted on one or more of
  its containers.
- As [container environment variable](#using-secrets-as-environment-variables).
- By the [kubelet when pulling images](#using-imagepullsecrets) for the Pod.

The name of a Secret object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
You can specify the `data` and/or the `stringData` field when creating a
configuration file for a Secret.  The `data` and the `stringData` fields are optional.
The values for all keys in the `data` field have to be base64-encoded strings.
If the conversion to base64 string is not desirable, you can choose to specify
the `stringData` field instead, which accepts arbitrary strings as values.

The keys of `data` and `stringData` must consist of alphanumeric characters,
`-`, `_` or `.`. All key-value pairs in the `stringData` field are internally
merged into the `data` field. If a key appears in both the `data` and the
`stringData` field, the value specified in the `stringData` field takes
precedence.
 -->

## Secret 概览 {#overview-of-secrets}

要使用 Secret 需要在 Pod 中引用这个 Secret。 Pod 可以以下三种方式 Secret:

- 作为
  {{< glossary_tooltip text="volume" term_id="volume" >}}
  中的
  [文件](#using-secrets-as-files-from-a-pod)
  挂载到一个或多个容器中。
- 作为 [容器中的环境变量](#using-secrets-as-environment-variables).
- 通过 [kubelet 拉取镜像](#using-imagepullsecrets)

Secret 对象的名称必须是一个有效的
[DNS 子域名](/k8sDocs/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
在为 Secret 创建配置文件时可以指定 `data` 和/或 `stringData` 字段。 `data` 和 `stringData`
字段都是可选的。 `data` 字段下所有键的值都得是 base64 编码的字符串。如果不想转化为 base64
的编码字符串，则可以选择 `stringData` 字段代替，它可以接受任意字符串作为值。

`data` 和 `stringData` 下面的键必须由 字母，数字， `-`, `_` 或 `.` 组成。 `stringData`
字段下面的所有键值对都会内部地合并到 `data` 字段。 如果一个键同时出现在 `data` 和 `stringData`
字段下面， 则 `stringData` 字段下面指定的值有更高的优先级。
<!--
## Types of Secret {#secret-types}

When creating a Secret, you can specify its type using the `type` field of
the [`Secret`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#secret-v1-core)
resource, or certain equivalent `kubectl` command line flags (if available).
The Secret type is used to facilitate programmatic handling of the Secret data.

Kubernetes provides several builtin types for some common usage scenarios.
These types vary in terms of the validations performed and the constraints
Kubernetes imposes on them.

| Builtin Type | Usage |
|--------------|-------|
| `Opaque`     |  arbitrary user-defined data |
| `kubernetes.io/service-account-token` | service account token |
| `kubernetes.io/dockercfg` | serialized `~/.dockercfg` file |
| `kubernetes.io/dockerconfigjson` | serialized `~/.docker/config.json` file |
| `kubernetes.io/basic-auth` | credentials for basic authentication |
| `kubernetes.io/ssh-auth` | credentials for SSH authentication |
| `kubernetes.io/tls` | data for a TLS client or server |
| `bootstrap.kubernetes.io/token` | bootstrap token data |

You can define and use your own Secret type by assigning a non-empty string as the
`type` value for a Secret object. An empty string is treated as an `Opaque` type.
Kubernetes doesn't impose any constraints on the type name. However, if you
are using one of the builtin types, you must meet all the requirements defined
for that type.
 -->

## Secret 的类别 {#secret-types}

在创建 Secret 时，用户通过设置
[`Secret`](https://kubernetes.io/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#secret-v1-core)
资源的 `type` 字段指定它的类型， 或使用等同的 `kubectl` 命令参数(如果存在)。 Secret 类型用于
帮助程式化地处理 Secret 数据。

k8s 提供了几种内置的类型用于一些常见的使用场景。 这些类型不同对应不同的验证操作和对其施加不同的
约束。
{{<todo-optimize >}}

| 内置类型       | Usage |
|--------------|-------|
| `Opaque`     |  任意用户定义数据 |
| `kubernetes.io/service-account-token` | service account token |
| `kubernetes.io/dockercfg` | 序列化的 `~/.dockercfg` 文件 |
| `kubernetes.io/dockerconfigjson` | 序列化的 `~/.docker/config.json` 文件 |
| `kubernetes.io/basic-auth` | 基础认证的凭据 |
| `kubernetes.io/ssh-auth` | SSH 认证的凭据 |
| `kubernetes.io/tls` | 用户 TLS 客户端或服务端的数据  |
| `bootstrap.kubernetes.io/token` | 引导 token 数据 |

用户可以通过为 Secret 对象的 `type` 值设置为一个非空字符串的方式定义并使用自己的 Secret 类型。
一个空字符串被认作是 `Opaque` 类型。k8s 没有对类型的名称作任何限制。 但如果使用的是内置类型，
必须要满足这个类型的定义需求。

<!--

### Opaque secrets

`Opaque` is the default Secret type if omitted from a Secret configuration file.
When you create a Secret using `kubectl`, you will use the `generic`
subcommand to indicate an `Opaque` Secret type. For example, the following
command creates an empty Secret of type `Opaque`.

```shell
kubectl create secret generic empty-secret
kubectl get secret empty-secret
```

The output looks like:

```
NAME           TYPE     DATA   AGE
empty-secret   Opaque   0      2m6s
```

The `DATA` column shows the number of data items stored in the Secret.
In this case, `0` means we have just created an empty Secret.
 -->
### Opaque 类别 Secret {#opaque-secrets}

如果 Secret 配置文件中没有定义 Secret 类型则 `Opaque` 就是默认类型。 当使用 `kubectl` 创建
Secret, 可以使用 `generic` 子命令来表示 `Opaque` Secret 类型。 例如，以下命令会创建一个
类型为 `Opaque` 空 Secret 。

```shell
kubectl create secret generic empty-secret
kubectl get secret empty-secret
```

输出类似如下:
```
NAME           TYPE     DATA   AGE
empty-secret   Opaque   0      2m6s
```

`DATA` 列显示的是 Secret 中存储的数据条目数。 在这种情况下， `0` 表示这里创建的是一个空的 Secret
<!--
###  Service account token Secrets

A `kubernetes.io/service-account-token` type of Secret is used to store a
token that identifies a service account. When using this Secret type, you need
to ensure that the `kubernetes.io/service-account.name` annotation is set to an
existing service account name. An Kubernetes controller fills in some other
fields such as the `kubernetes.io/service-account.uid` annotation and the
`token` key in the `data` field set to actual token content.

The following example configuration declares a service account token Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  # You can include additional key value pairs as you do with Opaque Secrets
  extra: YmFyCg==
```

When creating a `Pod`, Kubernetes automatically creates a service account Secret
and automatically modifies your Pod to use this Secret. The service account token
Secret contains credentials for accessing the API.

The automatic creation and use of API credentials can be disabled or
overridden if desired. However, if all you need to do is securely access the
API server, this is the recommended workflow.

See the [ServiceAccount](/docs/tasks/configure-pod-container/configure-service-account/)
documentation for more information on how service accounts work.
You can also check the `automountServiceAccountToken` field and the
`serviceAccountName` field of the
[`Pod`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)
for information on referencing service account from Pods.
 -->

###  服务账号(Service account) token 类别 Secret {#service-account-token-secrets}

`kubernetes.io/service-account-token` 类型的 Secret 用于存储一个鉴定服务账号(service account)的 token.
当使用这个 Secret 类别时， 需要保证 `kubernetes.io/service-account.name` 注解设置为一
个现有的服务账号(service account)名称. 一个 k8s 控制会填充其它字段，如
`kubernetes.io/service-account.uid` 注解， `data` 字段中的 `token` 键设置实际 token
的内容。

The following example configuration declares a service account token Secret:
下面的例子中定义了一个 服务账号(service account) token 类别的 Secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  # 可以与 Opaque 类别的 Secret 一样添加更多的键值对
  extra: YmFyCg==
```

当创建一个 `Pod` 时， k8s 会自动地创建一个服务账号 Secret 再自动修改 Pod 来使用这个 Secret.
这个 服务账号(service account) token 中包含了访问 API 的凭据。

如果需要可以禁用或覆盖这种自动创建和使用 API 凭据的行为。 但是，如果仅需要安全地访问 API 服务，
这种是推荐的工作方式。

更多关于服务账号是工件的信息见
[ServiceAccount](/docs/tasks/configure-pod-container/configure-service-account/)。
也可以查看
[`Pod`](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core)
中的 `automountServiceAccountToken` 和 `serviceAccountName` 字段了解 Pod 中引用的
服务账号的信息。
<!--
### Docker config Secrets

You can use one of the following `type` values to create a Secret to
store the credentials for accessing a Docker registry for images.

- `kubernetes.io/dockercfg`
- `kubernetes.io/dockerconfigjson`

The `kubernetes.io/dockercfg` type is reserved to store a serialized
`~/.dockercfg` which is the legacy format for configuring Docker command line.
When using this Secret type, you have to ensure the Secret `data` field
contains a `.dockercfg` key whose value is content of a `~/.dockercfg` file
encoded in the base64 format.

The `kubernetes.io/dockerconfigjson` type is designed for storing a serialized
JSON that follows the same format rules as the `~/.docker/config.json` file
which is a new format for `~/.dockercfg`.
When using this Secret type, the `data` field of the Secret object must
contain a `.dockerconfigjson` key, in which the content for the
`~/.docker/config.json` file is provided as a base64 encoded string.

Below is an example for a `kubernetes.io/dockercfg` type of Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
    "<base64 encoded ~/.dockercfg file>"
```

{{< note >}}
If you do not want to perform the base64 encoding, you can choose to use the
`stringData` field instead.
{{< /note >}}

When you create these types of Secrets using a manifest, the API
server checks whether the expected key does exists in the `data` field, and
it verifies if the value provided can be parsed as a valid JSON. The API
server doesn't validate if the JSON actually is a Docker config file.

When you do not have a Docker config file, or you want to use `kubectl`
to create a Docker registry Secret, you can do:

```shell
kubectl create secret docker-registry secret-tiger-docker \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com
```

This command creates a Secret of type `kubernetes.io/dockerconfigjson`.
If you dump the `.dockerconfigjson` content from the `data` field, you will
get the following JSON content which is a valid Docker configuration created
on the fly:

```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "tiger",
      "password": "pass113",
      "email": "tiger@acme.com",
      "auth": "dGlnZXI6cGFzczExMw=="
    }
  }
}
```
 -->

### Docker 配置 Secret {#docker-config-Secrets}

用户可以使用以下 `type` 值来创建存储访问 Docker 镜像仓库的凭据。

- `kubernetes.io/dockercfg`
- `kubernetes.io/dockerconfigjson`

`kubernetes.io/dockercfg` 类别是存储序列化 `~/.dockercfg` 文件的保留类别， 其中存放的
是配置 Docker 命令行的经典格式。 当使用这个 Secret 类别时， 需要确保 `data` 字段中包含
一个 `.dockercfg` 键，并且它的值就是一个 `~/.dockercfg` 内容的 base64 编码格式。

`kubernetes.io/dockerconfigjson`  类别被设计用来存储序列化 JSON, 其格式规则与
`~/.docker/config.json` 文件一样, 也就是 `~/.dockercfg` 的新格式。
当使用这个 Secret 类别时， Secret 对象的 `data` 字段必须包含一个 `.dockerconfigjson` 键
，其中的内容是 `~/.docker/config.json` 文件内容 base64 编码字符串

Below is an example for a `kubernetes.io/dockercfg` type of Secret:
下面是一个 `kubernetes.io/dockercfg` 类别 Secret 的示例:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
    "<base64 encoded ~/.dockercfg file>"
```

{{< note >}}
如果不想要执行 base64 编码，则可以使用 `stringData` 字段，而不是 `data` 字段
{{< /note >}}

在创建这些类别的 Secret 时，API 服务会检查其中的键是否是 `data` 字段中存在。并且会验证提供
的值是否能解析为有效的 JSON. 但 API 服务不会验证这个 JSON 实际上是不是一个 Docker 配置文件。

当在没有 Docker 配置文件时，可以使用 `kubectl` 创建一个 Docker 镜像仓库凭据 Secret， 例如:

```shell
kubectl create secret docker-registry secret-tiger-docker \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com
```

这个命令会创建一个 `kubernetes.io/dockerconfigjson` 类别的 Secret。如果将 `.dockerconfigjson`
`data` 字段转存，就会得到新创建的一个 JSON 格式的有效的 Docker 配置:

```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "username": "tiger",
      "password": "pass113",
      "email": "tiger@acme.com",
      "auth": "dGlnZXI6cGFzczExMw=="
    }
  }
}
```
<!--
### Basic authentication Secret

The `kubernetes.io/basic-auth` type is provided for storing credentials needed
for basic authentication. When using this Secret type, the `data` field of the
Secret must contain the following two keys:

- `username`: the user name for authentication;
- `password`: the password or token for authentication.

Both values for the above two keys are base64 encoded strings. You can, of
course, provide the clear text content using the `stringData` for Secret
creation.

The following YAML is an example config for a basic authentication Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: t0p-Secret
```

The basic authentication Secret type is provided only for user's convenience.
You can create an `Opaque` for credentials used for basic authentication.
However, using the builtin Secret type helps unify the formats of your credentials
and the API server does verify if the required keys are provided in a Secret
configuration.
 -->

### 基础认证 Secret

`kubernetes.io/basic-auth` 类别的 Secret 是用来存储基础认证所需的凭据的。 当使用这个
Secret 类别时， Secret 的 `data` 字段必须要包含以下两个键:

- `username`: 认证的用户名;
- `password`: 认证的密码或 token.

以下两个键的值都需要进行 base64 编码。 也可以在创建 Secret 使用 `stringData` 这样就可以在
直接使用明文的键值。

以下 YAML 就是一个基础认证 Secret 的示例:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: t0p-Secret
```

提供基础认证类型的 Secret 只是为了用户方便。 也可以创建一个 `Opaque` 类别的 Secret 来存储
基础认证的凭据。 但使用内置的 Secret 类别有助于统一凭据格式并且 API 服务也会验证 Secret 配置
中是否提供了需要的键。
<!--
### SSH authentication secrets

The builtin type `kubernetes.io/ssh-auth` is provided for storing data used in
SSH authentication. When using this Secret type, you will have to specify a
`ssh-privatekey` key-value pair in the `data` (or `stringData`) field
as the SSH credential to use.

The following YAML is an example config for a SSH authentication Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # the data is abbreviated in this example
  ssh-privatekey: |
     MIIEpQIBAAKCAQEAulqb/Y ...
```

The SSH authentication Secret type is provided only for user's convenience.
You can create an `Opaque` for credentials used for SSH authentication.
However, using the builtin Secret type helps unify the formats of your credentials
and the API server does verify if the required keys are provided in a Secret
configuration.

{{< caution >}}
SSH private keys do not establish trusted communication between an SSH client and
host server on their own. A secondary means of establishing trust is needed to
mitigate "man in the middle" attacks, such as a `known_hosts` file added to a
ConfigMap.
{{< /caution >}}
 -->
<!--
### SSH authentication secrets

The builtin type `kubernetes.io/ssh-auth` is provided for storing data used in
SSH authentication. When using this Secret type, you will have to specify a
`ssh-privatekey` key-value pair in the `data` (or `stringData`) field
as the SSH credential to use.

The following YAML is an example config for a SSH authentication Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # the data is abbreviated in this example
  ssh-privatekey: |
     MIIEpQIBAAKCAQEAulqb/Y ...
```

The SSH authentication Secret type is provided only for user's convenience.
You can create an `Opaque` for credentials used for SSH authentication.
However, using the builtin Secret type helps unify the formats of your credentials
and the API server does verify if the required keys are provided in a Secret
configuration.

{{< caution >}}
SSH private keys do not establish trusted communication between an SSH client and
host server on their own. A secondary means of establishing trust is needed to
mitigate "man in the middle" attacks, such as a `known_hosts` file added to a
ConfigMap.
{{< /caution >}}
 -->

### SSH 认证 Secret {#ssh-authentication-secrets}

内置的 `kubernetes.io/ssh-auth` 类别的 Secret 是用于存储 SSH 认证数据的。 当使用该类别
Secret 时， 需要在 `data` (或 `stringData`)指定用于 SSH 认证的键值对，其中键为 `ssh-privatekey`
值为私钥的内容

下面的 YAML 就是一个 SSH 认证 Secret 的配置示例:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  # 示例中省略了部分数据
  ssh-privatekey: |
     MIIEpQIBAAKCAQEAulqb/Y ...
```

提供 SSH 认证 Secret 只是为了用户方便。 也可以创建一个 `Opaque` 类别的 Secret 来存储
SSH 认证的凭据。 但使用内置的 Secret 类别有助于统一凭据格式并且 API 服务也会验证 Secret 配置
中是否提供了需要的键。

{{< caution >}}
SSH 私钥并不能独立在 SSH 客户端和服务端之间建议安全连接。 一种建立安全连接辅助方式能降低"中间人"攻击的可能，
例如将 `known_hosts` 文件添加到 ConfigMap
{{< /caution >}}
<!--
### TLS secrets

Kubernetes provides a builtin Secret type `kubernetes.io/tls` for to storing
a certificate and its associated key that are typically used for TLS . This
data is primarily used with TLS termination of the Ingress resource, but may
be used with other resources or directly by a workload.
When using this type of Secret, the `tls.key` and the `tls.crt` key must be provided
in the `data` (or `stringData`) field of the Secret configuration, although the API
server doesn't actually validate the values for each key.

The following YAML contains an example config for a TLS Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
    MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
    MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```

The TLS Secret type is provided for user's convenience. You can create an `Opaque`
for credentials used for TLS server and/or client. However, using the builtin Secret
type helps ensure the consistency of Secret format in your project; the API server
does verify if the required keys are provided in a Secret configuration.

When creating a TLS Secret using `kubectl`, you can use the `tls` subcommand
as shown in the following example:

```shell
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```

The public/private key pair must exist before hand. The public key certificate
for `--cert` must be .PEM encoded (Base64-encoded DER format), and match the
given private key for `--key`.
The private key must be in what is commonly called PEM private key format,
unencrypted. In both cases, the initial and the last lines from PEM (for
example, `--------BEGIN CERTIFICATE-----` and `-------END CERTIFICATE----` for
a cetificate) are *not* included.
 -->

### TLS Secret {#tls-secrets}

k8s 提供了内置的 `kubernetes.io/tls` Secret 类别用在存储 TLS 通常使用的证书和对应的 key.
这些数据主要用于 Ingress 资源的 TLS 终结，但可能被其它资源使用或直接被工作负载使用。 当使用
该类另的 Secret 时，在 Secret 配置的 `data` (或 `stringData`) 字段中必须要提供 `tls.key`
和 `tls.crt` 这两个键，尽管 API 服务实际上是不验证这些键的。

The following YAML contains an example config for a TLS Secret:
下面的 YAML 中包含了一个 TLS Secret 的示例配置:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # 下面的示例数据部分省略
  tls.crt: |
    MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
    MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```

提供 TLS Secret 只是为了用户方便。 也可以创建一个 `Opaque` 类别的 Secret 来存储
TLS 服务端和/或客户端的凭据。 但使用内置的 Secret 类别有助于统一凭据格式并且 API 服务也会
验证 Secret 配置中是否提供了需要的键。

在使用 `kubectl` 创建 TLS Secret，可以使用 `tls` 子命令，见下面的例子:

```shell
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```

公钥/私钥对必须要先准备好。 `--cert` 使用的公钥证书必要是 .PEM 编码(Base64-encoded DER 格式)，
并且与 `--key` 中提供的私钥匹配。 私钥也必须以常见的 PEM 私钥格式提供， 不加密。
对于两个键， PEM 的首尾行(就如证书的 `--------BEGIN CERTIFICATE-----` 和
`-------END CERTIFICATE----`)是 *不* 包含在内的。
<!--
### Bootstrap token Secrets

A bootstrap token Secret can be created by explicitly specifying the Secret
`type` to `bootstrap.kubernetes.io/token`. This type of Secret is designed for
tokens used during the node bootstrap process. It stores tokens used to sign
well known ConfigMaps.

A bootstrap token Secret is usually created in the `kube-system` namespace and
named in the form `bootstrap-token-<token-id>` where `<token-id>` is a 6 character
string of the token ID.

As a Kubernetes manifest, a bootstrap token Secret might look like the
following:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-5emitj
  namespace: kube-system
type: bootstrap.kubernetes.io/token
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wOS0xM1QwNDozOToxMFo=
  token-id: NWVtaXRq
  token-secret: a3E0Z2lodnN6emduMXAwcg==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
```

A bootstrap type Secret has the following keys specified under `data`:

- `token_id`: A random 6 character string as the token identifier. Required.
- `token-secret`: A random 16 character string as the actual token secret. Required.
- `description`: A human-readable string that describes what the token is
  used for. Optional.
- `expiration`: An absolute UTC time using RFC3339 specifying when the token
  should be expired. Optional.
- `usage-bootstrap-<usage>`: A boolean flag indicating additional usage for
  the bootstrap token.
- `auth-extra-groups`: A comma-separated list of group names that will be
  authenticated as in addition to the `system:bootstrappers` group.

The above YAML may look confusing because the values are all in base64 encoded
strings. In fact, you can create an identical Secret using the following YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  # Note how the Secret is named
  name: bootstrap-token-5emitj
  # A bootstrap token Secret usually resides in the kube-system namespace
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  auth-extra-groups: "system:bootstrappers:kubeadm:default-node-token"
  expiration: "2020-09-13T04:39:10Z"
  # This token ID is used in the name
  token-id: "5emitj"
  token-secret: "kq4gihvszzgn1p0r"
  # This token can be used for authentication
  usage-bootstrap-authentication: "true"
  # and it can be used for signing
  usage-bootstrap-signing: "true"
```
 -->

### 引导令牌 Secret {#bootstrap-token-secrets}

引导令牌 Secret 可以通过显示地将 Secret 的 `type` 设置为 `bootstrap.kubernetes.io/token`
来创建. 这个 Secret 类别是设计来存储节点引导进程所使用的令牌的。 它存储的令牌是用来签发认可
的 ConfigMap 的。

引导令牌 Secret 通常是创建在 `kube-system` 命名空间中， 命名格式为 `bootstrap-token-<token-id>`
其中 `<token-id>` 是一个 6 字符的令牌 ID。

一个引导令牌 Secret 可能就长成下面的这个梯子:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-5emitj
  namespace: kube-system
type: bootstrap.kubernetes.io/token
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wOS0xM1QwNDozOToxMFo=
  token-id: NWVtaXRq
  token-secret: a3E0Z2lodnN6emduMXAwcg==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
```

一个引导令牌 Secret 的 `data`字段下有如下字段:

- `token-id`: 一个 6 字符随机字符串作为令牌 ID. 必要.

- `token-secret`: 一个 16 字符随机字符串，作为真正的令牌秘文. 必要.

- `description`: 一个人类可读的字符串，描述令牌是用来做啥的。 可选

- `expiration`: 一个绝对 UTF 时间，使用 RFC3339， 指定令牌啥时候过期。可选

- `usage-bootstrap-<usage>`: 一个布尔标示，用来指示这个引导令牌附加使用信息

- `auth-extra-groups`: 一个逗号分隔的组名称列表，用来认证 `system:bootstrappers` 打头的组

上面的 YAML 因为值都是 base64 编码的，看起来有点晕。实际上可以使用以下 YAML 创建一个与它一样
的 Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  # 注意 Secret 是怎么命名的
  name: bootstrap-token-5emitj
  # 引导令牌 Secret 通常都是在 kube-system 命名空间中
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  auth-extra-groups: "system:bootstrappers:kubeadm:default-node-token"
  expiration: "2020-09-13T04:39:10Z"
  # 这个 令牌 ID 被用在名称上
  token-id: "5emitj"
  token-secret: "kq4gihvszzgn1p0r"
  # 这个令牌可以被用作认证
  usage-bootstrap-authentication: "true"
  # 也可以被用来签名
  usage-bootstrap-signing: "true"
```
<!--
## Creating a Secret

There are several options to create a Secret:

- [create Secret using `kubectl` command](/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- [create Secret from config file](/docs/tasks/configmap-secret/managing-secret-using-config-file/)
- [create Secret using kustomize](/docs/tasks/configmap-secret/managing-secret-using-kustomize/)
 -->

## 创建 Secret {#creating-a-secret}

There are several options to create a Secret:
有几种创建 Secret 方式:

- [使用 `kubectl` 命令创建 Secret](/k8sDocs/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- [通过配置文件创建 Secret ](/k8sDocs/docs/tasks/configmap-secret/managing-secret-using-config-file/)
- [使用 kustomize 创建 Secret](/k8sDocs/docs/tasks/configmap-secret/managing-secret-using-kustomize/)
<!--
## Editing a Secret

An existing Secret may be edited with the following command:

```shell
kubectl edit secrets mysecret
```

This will open the default configured editor and allow for updating the base64 encoded Secret values in the `data` field:

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: { ... }
  creationTimestamp: 2016-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
```
 -->

## 编辑 Secret {#editing-a-secret}

An existing Secret may be edited with the following command:
可以通过以下命令修改一个现有的 Secret:

```shell
kubectl edit secrets mysecret
```

This will open the default configured editor and allow for updating the base64 encoded Secret values in the `data` field:
这会打开默认编辑器，并允许对 `data` 字段下面  base64 编辑的 Secret 进行修改:

```yaml
# 请绑架下面的对象，以 '#' 开头的行会被忽略，
# 一个空文件会中止编辑。 如果保存时发生错误就会响应相应的失败信息
#
apiVersion: v1
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: { ... }
  creationTimestamp: 2016-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
```
<!--
## Using Secrets

Secrets can be mounted as data volumes or exposed as
{{< glossary_tooltip text="environment variables" term_id="container-env-variables" >}}
to be used by a container in a Pod. Secrets can also be used by other parts of the
system, without being directly exposed to the Pod. For example, Secrets can hold
credentials that other parts of the system should use to interact with external
systems on your behalf.
 -->

## 使用 Secret {#using-secrets}

Secret 可以挂载为数据卷或暴露为
{{< glossary_tooltip term_id="container-env-variables" >}}
以供 Pod 中的容器使用。 Secret 也可以被系统的其它部分使用，而不需要直接暴露给 Pod。 例如，
Secret 可以存储系统其它部分与外部系统交互时需要用到的凭据。
<!--
### Using Secrets as files from a Pod

To consume a Secret in a volume in a Pod:

1. Create a secret or use an existing one. Multiple Pods can reference the same secret.
1. Modify your Pod definition to add a volume under `.spec.volumes[]`. Name the volume anything, and have a `.spec.volumes[].secret.secretName` field equal to the name of the Secret object.
1. Add a `.spec.containers[].volumeMounts[]` to each container that needs the secret. Specify `.spec.containers[].volumeMounts[].readOnly = true` and `.spec.containers[].volumeMounts[].mountPath` to an unused directory name where you would like the secrets to appear.
1. Modify your image or command line so that the program looks for files in that directory. Each key in the secret `data` map becomes the filename under `mountPath`.

This is an example of a Pod that mounts a Secret in a volume:

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
    secret:
      secretName: mysecret
```

Each Secret you want to use needs to be referred to in `.spec.volumes`.

If there are multiple containers in the Pod, then each container needs its
own `volumeMounts` block, but only one `.spec.volumes` is needed per Secret.

You can package many files into one secret, or use many secrets, whichever is convenient.
 -->

### 将 Secret 用作 Pod 中的文件 {#using-secrets-as-files-from-a-pod}

在 Pod 中以卷的方式使用 Secret:

1. 创建或使用一个现有的 Secret. 多个 Pod 可以引用同一个 Secret.
2. 修改 Pod 定义， 在 `.spec.volumes[]` 下面添加一个卷。 卷名随便起，但其中的
`.spec.volumes[].secret.secretName` 字段的值需要是 Secret 对象的名称。
3. 在每个需要 Secret 的容器中添加一个 `.spec.containers[].volumeMounts[]`。 指定
  `.spec.containers[].volumeMounts[].readOnly = true`
  和
  `.spec.containers[].volumeMounts[].mountPath` 指向一个期望的未使用的位置
4. 修改镜像或命令行，让程序查找目录中的文件。 Secret `data` 字段下的每一个键都会投射为
  `mountPath` 目录中的一个文件名。

下面是一个将 Secret 挂载为卷的 Pod 的示例:
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
    secret:
      secretName: mysecret
```

每一个想要使用的 Secret 都需要在 `.spec.volumes` 中引用。

如果 Pod 中有多个容器， 则每个容器需要各自的 `volumeMounts` 块， 但每个 Secret 只需要一个
`.spec.volumes`。

用户可以将多个文件放在一个 Secret 中，或使用多个 Secret, 哪个方便用哪个。
<!--
#### Projection of Secret keys to specific paths

You can also control the paths within the volume where Secret keys are projected.
You can use the `.spec.volumes[].secret.items` field to change the target path of each key:

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
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

What will happen:

* `username` secret is stored under `/etc/foo/my-group/my-username` file instead of `/etc/foo/username`.
* `password` secret is not projected.

If `.spec.volumes[].secret.items` is used, only keys specified in `items` are projected.
To consume all keys from the secret, all of them must be listed in the `items` field.
All listed keys must exist in the corresponding secret. Otherwise, the volume is not created.
 -->

#### 将 Secret 的键投射到指定目录 {#projection-of-secret-keys-to-specific-paths}

用户可以控制 Secret 键投射到卷中的哪个目录。
通过 `.spec.volumes[].secret.items` 字段就可以修改每个键的目标路径:

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
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

结果表现为:

* `username` 秘文就是存储在 `/etc/foo/my-group/my-username` 而不是 `/etc/foo/username`.
* `password` 秘文则不会投射


如果使用了 `.spec.volumes[].secret.items`，则只有在 `items` 中指定的键才会被投射。
要使用 Secret 中所有键，则需要在  `items` 字段把它们全部列举。
所有列表的键都必须要在对应的 Secret 存在，否则，卷不能被创建。

#### Secret files permissions

You can set the file access permission bits for a single Secret key.
If you don't specify any permissions, `0644` is used by default.
You can also set a default mode for the entire Secret volume and override per key if needed.

For example, you can specify a default mode like this:

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
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 0400
```

Then, the secret will be mounted on `/etc/foo` and all the files created by the
secret volume mount will have permission `0400`.

Note that the JSON spec doesn't support octal notation, so use the value 256 for
0400 permissions. If you use YAML instead of JSON for the Pod, you can use octal
notation to specify permissions in a more natural way.

Note if you `kubectl exec` into the Pod, you need to follow the symlink to find
the expected file mode. For example,

Check the secrets file mode on the pod.
```
kubectl exec mypod -it sh

cd /etc/foo
ls -l
```

The output is similar to this:
```
total 0
lrwxrwxrwx 1 root root 15 May 18 00:18 password -> ..data/password
lrwxrwxrwx 1 root root 15 May 18 00:18 username -> ..data/username
```

Follow the symlink to find the correct file mode.

```
cd /etc/foo/..data
ls -l
```

The output is similar to this:
```
total 8
-r-------- 1 root root 12 May 18 00:18 password
-r-------- 1 root root  5 May 18 00:18 username
```

You can also use mapping, as in the previous example, and specify different
permissions for different files like this:

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
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 0777
```

In this case, the file resulting in `/etc/foo/my-group/my-username` will have
permission value of `0777`. If you use JSON, owing to JSON limitations, you
must specify the mode in decimal notation, `511`.

Note that this permission value might be displayed in decimal notation if you
read it later.

#### Consuming Secret values from volumes

Inside the container that mounts a secret volume, the secret keys appear as
files and the secret values are base64 decoded and stored inside these files.
This is the result of commands executed inside the container from the example above:

```shell
ls /etc/foo/
```

The output is similar to:

```
username
password
```

```shell
cat /etc/foo/username
```

The output is similar to:

```
admin
```

```shell
cat /etc/foo/password
```

The output is similar to:

```
1f2d1e2e67df
```

The program in a container is responsible for reading the secrets from the
files.

#### Mounted Secrets are updated automatically

When a secret currently consumed in a volume is updated, projected keys are eventually updated as well.
The kubelet checks whether the mounted secret is fresh on every periodic sync.
However, the kubelet uses its local cache for getting the current value of the Secret.
The type of the cache is configurable using the `ConfigMapAndSecretChangeDetectionStrategy` field in
the [KubeletConfiguration struct](https://github.com/kubernetes/kubernetes/blob/{{< param "docsbranch" >}}/staging/src/k8s.io/kubelet/config/v1beta1/types.go).
A Secret can be either propagated by watch (default), ttl-based, or simply redirecting
all requests directly to the API server.
As a result, the total delay from the moment when the Secret is updated to the moment
when new keys are projected to the Pod can be as long as the kubelet sync period + cache
propagation delay, where the cache propagation delay depends on the chosen cache type
(it equals to watch propagation delay, ttl of cache, or zero correspondingly).

{{< note >}}
A container using a Secret as a
[subPath](/docs/concepts/storage/volumes#using-subpath) volume mount will not receive
Secret updates.
{{< /note >}}

### Using Secrets as environment variables

To use a secret in an {{< glossary_tooltip text="environment variable" term_id="container-env-variables" >}}
in a Pod:

1. Create a secret or use an existing one.  Multiple Pods can reference the same secret.
1. Modify your Pod definition in each container that you wish to consume the value of a secret key to add an environment variable for each secret key you wish to consume. The environment variable that consumes the secret key should populate the secret's name and key in `env[].valueFrom.secretKeyRef`.
1. Modify your image and/or command line so that the program looks for values in the specified environment variables.

This is an example of a Pod that uses secrets from environment variables:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```

#### Consuming Secret Values from environment variables

Inside a container that consumes a secret in an environment variables, the secret keys appear as
normal environment variables containing the base64 decoded values of the secret data.
This is the result of commands executed inside the container from the example above:

```shell
echo $SECRET_USERNAME
```

The output is similar to:

```
admin
```

```shell
echo $SECRET_PASSWORD
```

The output is similar to:

```
1f2d1e2e67df
```

#### Environment variables are not updated after a secret update

If a container already consumes a Secret in an environment variable, a Secret update will not be seen by the container unless it is restarted.
There are third party solutions for triggering restarts when secrets change.

## Immutable Secrets {#secret-immutable}

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

The Kubernetes beta feature _Immutable Secrets and ConfigMaps_ provides an option to set
individual Secrets and ConfigMaps as immutable. For clusters that extensively use Secrets
(at least tens of thousands of unique Secret to Pod mounts), preventing changes to their
data has the following advantages:

- protects you from accidental (or unwanted) updates that could cause applications outages
- improves performance of your cluster by significantly reducing load on kube-apiserver, by
closing watches for secrets marked as immutable.

This feature is controlled by the `ImmutableEphemeralVolumes` [feature
gate](/docs/reference/command-line-tools-reference/feature-gates/),
which is enabled by default since v1.19. You can create an immutable
Secret by setting the `immutable` field to `true`. For example,
```yaml
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

{{< note >}}
Once a Secret or ConfigMap is marked as immutable, it is _not_ possible to revert this change
nor to mutate the contents of the `data` field. You can only delete and recreate the Secret.
Existing Pods maintain a mount point to the deleted Secret - it is recommended to recreate
these pods.
{{< /note >}}

### Using imagePullSecrets

The `imagePullSecrets` field is a list of references to secrets in the same namespace.
You can use an `imagePullSecrets` to pass a secret that contains a Docker (or other) image registry
password to the kubelet. The kubelet uses this information to pull a private image on behalf of your Pod.
See the [PodSpec API](/docs/reference/generated/kubernetes-api/{{< latest-version >}}/#podspec-v1-core) for more information about the `imagePullSecrets` field.

#### Manually specifying an imagePullSecret

You can learn how to specify `ImagePullSecrets` from the [container images documentation](/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod).

### Arranging for imagePullSecrets to be automatically attached

You can manually create `imagePullSecrets`, and reference it from
a ServiceAccount. Any Pods created with that ServiceAccount
or created with that ServiceAccount by default, will get their `imagePullSecrets`
field set to that of the service account.
See [Add ImagePullSecrets to a service account](/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account)
 for a detailed explanation of that process.

### Automatic mounting of manually created Secrets

Manually created secrets (for example, one containing a token for accessing a GitHub account)
can be automatically attached to pods based on their service account.
See [Injecting Information into Pods Using a PodPreset](/docs/tasks/inject-data-application/podpreset/) for a detailed explanation of that process.

## Details

### Restrictions

Secret volume sources are validated to ensure that the specified object
reference actually points to an object of type Secret. Therefore, a secret
needs to be created before any Pods that depend on it.

Secret resources reside in a {{< glossary_tooltip text="namespace" term_id="namespace" >}}.
Secrets can only be referenced by Pods in that same namespace.

Individual secrets are limited to 1MiB in size. This is to discourage creation
of very large secrets which would exhaust the API server and kubelet memory.
However, creation of many smaller secrets could also exhaust memory. More
comprehensive limits on memory usage due to secrets is a planned feature.

The kubelet only supports the use of secrets for Pods where the secrets
are obtained from the API server.
This includes any Pods created using `kubectl`, or indirectly via a replication
controller. It does not include Pods created as a result of the kubelet
`--manifest-url` flag, its `--config` flag, or its REST API (these are
not common ways to create Pods.)

Secrets must be created before they are consumed in Pods as environment
variables unless they are marked as optional. References to secrets that do
not exist will prevent the Pod from starting.

References (`secretKeyRef` field) to keys that do not exist in a named Secret
will prevent the Pod from starting.

Secrets used to populate environment variables by the `envFrom` field that have keys
that are considered invalid environment variable names will have those keys
skipped. The Pod will be allowed to start. There will be an event whose
reason is `InvalidVariableNames` and the message will contain the list of
invalid keys that were skipped. The example shows a pod which refers to the
default/mysecret that contains 2 invalid keys: `1badkey` and `2alsobad`.

```shell
kubectl get events
```

The output is similar to:

```
LASTSEEN   FIRSTSEEN   COUNT     NAME            KIND      SUBOBJECT                         TYPE      REASON
0s         0s          1         dapi-test-pod   Pod                                         Warning   InvalidEnvironmentVariableNames   kubelet, 127.0.0.1      Keys [1badkey, 2alsobad] from the EnvFrom secret default/mysecret were skipped since they are considered invalid environment variable names.
```

### Secret and Pod lifetime interaction

When a Pod is created by calling the Kubernetes API, there is no check if a referenced
secret exists. Once a Pod is scheduled, the kubelet will try to fetch the
secret value. If the secret cannot be fetched because it does not exist or
because of a temporary lack of connection to the API server, the kubelet will
periodically retry. It will report an event about the Pod explaining the
reason it is not started yet. Once the secret is fetched, the kubelet will
create and mount a volume containing it. None of the Pod's containers will
start until all the Pod's volumes are mounted.

## Use cases

### Use-Case: As container environment variables

Create a secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

Create the Secret:
```shell
kubectl apply -f mysecret.yaml
```

Use `envFrom` to define all of the Secret's data as container environment variables. The key from the Secret becomes the environment variable name in the Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
  restartPolicy: Never
```

### Use-Case: Pod with ssh keys

Create a secret containing some ssh keys:

```shell
kubectl create secret generic ssh-key-secret --from-file=ssh-privatekey=/path/to/.ssh/id_rsa --from-file=ssh-publickey=/path/to/.ssh/id_rsa.pub
```

The output is similar to:

```
secret "ssh-key-secret" created
```

You can also create a `kustomization.yaml` with a `secretGenerator` field containing ssh keys.

{{< caution >}}
Think carefully before sending your own ssh keys: other users of the cluster may have access to the secret. Use a service account which you want to be accessible to all the users with whom you share the Kubernetes cluster, and can revoke this account if the users are compromised.
{{< /caution >}}

Now you can create a Pod which references the secret with the ssh key and
consumes it in a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret
  containers:
  - name: ssh-test-container
    image: mySshImage
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

When the container's command runs, the pieces of the key will be available in:

```
/etc/secret-volume/ssh-publickey
/etc/secret-volume/ssh-privatekey
```

The container is then free to use the secret data to establish an ssh connection.

### Use-Case: Pods with prod / test credentials

This example illustrates a Pod which consumes a secret containing production
credentials and another Pod which consumes a secret with test environment
credentials.

You can create a `kustomization.yaml` with a `secretGenerator` field or run
`kubectl create secret`.

```shell
kubectl create secret generic prod-db-secret --from-literal=username=produser --from-literal=password=Y4nys7f11
```

The output is similar to:

```
secret "prod-db-secret" created
```

You can also create a secret for test environment credentials.

```shell
kubectl create secret generic test-db-secret --from-literal=username=testuser --from-literal=password=iluvtests
```

The output is similar to:

```
secret "test-db-secret" created
```

{{< note >}}
Special characters such as `$`, `\`, `*`, `=`, and `!` will be interpreted by your [shell](https://en.wikipedia.org/wiki/Shell_(computing)) and require escaping.
In most shells, the easiest way to escape the password is to surround it with single quotes (`'`).
For example, if your actual password is `S!B\*d$zDsb=`, you should execute the command this way:

```shell
kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
```

 You do not need to escape special characters in passwords from files (`--from-file`).
{{< /note >}}

Now make the Pods:

```shell
cat <<EOF > pod.yaml
apiVersion: v1
kind: List
items:
- kind: Pod
  apiVersion: v1
  metadata:
    name: prod-db-client-pod
    labels:
      name: prod-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: prod-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
- kind: Pod
  apiVersion: v1
  metadata:
    name: test-db-client-pod
    labels:
      name: test-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: test-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
EOF
```

Add the pods to the same kustomization.yaml:

```shell
cat <<EOF >> kustomization.yaml
resources:
- pod.yaml
EOF
```

Apply all those objects on the API server by running:

```shell
kubectl apply -k .
```

Both containers will have the following files present on their filesystems with the values for each container's environment:

```
/etc/secret-volume/username
/etc/secret-volume/password
```

Note how the specs for the two Pods differ only in one field; this facilitates
creating Pods with different capabilities from a common Pod template.

You could further simplify the base Pod specification by using two service accounts:

1. `prod-user` with the `prod-db-secret`
1. `test-user` with the `test-db-secret`

The Pod specification is shortened to:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-db-client-pod
  labels:
    name: prod-db-client
spec:
  serviceAccount: prod-db-client
  containers:
  - name: db-client-container
    image: myClientImage
```

### Use-case: dotfiles in a secret volume

You can make your data "hidden" by defining a key that begins with a dot.
This key represents a dotfile or "hidden" file. For example, when the following secret
is mounted into a volume, `secret-volume`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - name: dotfile-test-container
    image: k8s.gcr.io/busybox
    command:
    - ls
    - "-l"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

The volume will contain a single file, called `.secret-file`, and
the `dotfile-test-container` will have this file present at the path
`/etc/secret-volume/.secret-file`.

{{< note >}}
Files beginning with dot characters are hidden from the output of  `ls -l`;
you must use `ls -la` to see them when listing directory contents.
{{< /note >}}

### Use-case: Secret visible to one container in a Pod

Consider a program that needs to handle HTTP requests, do some complex business
logic, and then sign some messages with an HMAC. Because it has complex
application logic, there might be an unnoticed remote file reading exploit in
the server, which could expose the private key to an attacker.

This could be divided into two processes in two containers: a frontend container
which handles user interaction and business logic, but which cannot see the
private key; and a signer container that can see the private key, and responds
to simple signing requests from the frontend (for example, over localhost networking).

With this partitioned approach, an attacker now has to trick the application
server into doing something rather arbitrary, which may be harder than getting
it to read a file.

<!-- TODO: explain how to do this while still using automation. -->

## Best practices

### Clients that use the Secret API

When deploying applications that interact with the Secret API, you should
limit access using [authorization policies](
/docs/reference/access-authn-authz/authorization/) such as [RBAC](
/docs/reference/access-authn-authz/rbac/).

Secrets often hold values that span a spectrum of importance, many of which can
cause escalations within Kubernetes (e.g. service account tokens) and to
external systems. Even if an individual app can reason about the power of the
secrets it expects to interact with, other apps within the same namespace can
render those assumptions invalid.

For these reasons `watch` and `list` requests for secrets within a namespace are
extremely powerful capabilities and should be avoided, since listing secrets allows
the clients to inspect the values of all secrets that are in that namespace. The ability to
`watch` and `list` all secrets in a cluster should be reserved for only the most
privileged, system-level components.

Applications that need to access the Secret API should perform `get` requests on
the secrets they need. This lets administrators restrict access to all secrets
while [white-listing access to individual instances](
/docs/reference/access-authn-authz/rbac/#referring-to-resources) that
the app needs.

For improved performance over a looping `get`, clients can design resources that
reference a secret then `watch` the resource, re-requesting the secret when the
reference changes. Additionally, a ["bulk watch" API](
https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/bulk_watch.md)
to let clients `watch` individual resources has also been proposed, and will likely
be available in future releases of Kubernetes.

## Security properties

### Protections

Because secrets can be created independently of the Pods that use
them, there is less risk of the secret being exposed during the workflow of
creating, viewing, and editing Pods. The system can also take additional
precautions with Secrets, such as avoiding writing them to disk where
possible.

A secret is only sent to a node if a Pod on that node requires it.
The kubelet stores the secret into a `tmpfs` so that the secret is not written
to disk storage. Once the Pod that depends on the secret is deleted, the kubelet
will delete its local copy of the secret data as well.

There may be secrets for several Pods on the same node. However, only the
secrets that a Pod requests are potentially visible within its containers.
Therefore, one Pod does not have access to the secrets of another Pod.

There may be several containers in a Pod. However, each container in a Pod has
to request the secret volume in its `volumeMounts` for it to be visible within
the container. This can be used to construct useful [security partitions at the
Pod level](#use-case-secret-visible-to-one-container-in-a-pod).

On most Kubernetes distributions, communication between users
and the API server, and from the API server to the kubelets, is protected by SSL/TLS.
Secrets are protected when transmitted over these channels.

{{< feature-state for_k8s_version="v1.13" state="beta" >}}

You can enable [encryption at rest](/docs/tasks/administer-cluster/encrypt-data/)
for secret data, so that the secrets are not stored in the clear into {{< glossary_tooltip term_id="etcd" >}}.

### Risks

 - In the API server, secret data is stored in {{< glossary_tooltip term_id="etcd" >}};
   therefore:
   - Administrators should enable encryption at rest for cluster data (requires v1.13 or later).
   - Administrators should limit access to etcd to admin users.
   - Administrators may want to wipe/shred disks used by etcd when no longer in use.
   - If running etcd in a cluster, administrators should make sure to use SSL/TLS
     for etcd peer-to-peer communication.
 - If you configure the secret through a manifest (JSON or YAML) file which has
   the secret data encoded as base64, sharing this file or checking it in to a
   source repository means the secret is compromised. Base64 encoding is _not_ an
   encryption method and is considered the same as plain text.
 - Applications still need to protect the value of secret after reading it from the volume,
   such as not accidentally logging it or transmitting it to an untrusted party.
 - A user who can create a Pod that uses a secret can also see the value of that secret. Even
   if the API server policy does not allow that user to read the Secret, the user could
   run a Pod which exposes the secret.
 - Currently, anyone with root permission on any node can read _any_ secret from the API server,
   by impersonating the kubelet. It is a planned feature to only send secrets to
   nodes that actually require them, to restrict the impact of a root exploit on a
   single node.


## {{% heading "whatsnext" %}}

- Learn how to [manage Secret using `kubectl`](/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
- Learn how to [manage Secret using config file](/docs/tasks/configmap-secret/managing-secret-using-config-file/)
- Learn how to [manage Secret using kustomize](/docs/tasks/configmap-secret/managing-secret-using-kustomize/)
