---
title: 使用 kubeconfig 文件组织集群访问
content_type: concept
weight: 60
---

<!--
---
title: Organizing Cluster Access Using kubeconfig Files
content_type: concept
weight: 60
---
 -->
<!-- overview -->

kubeconfig 文件用来组织关于集群，用户，命名空间，和认证机制信息。 `kubectl` 命令行工具使用
kubeconfig 文件来查找其用以选择集群和与集群 API 服务通信的信息。

{{< note >}}
一个用于配置集群访问的文件就叫做 *kubeconfig 文件*。 这是引用配置文件的常用方式。 但并不意味
着有一个叫 `kubeconfig` 的文件。
{{< /note >}}

默认情况下， `kubectl` 会在 `$HOME/.kube` 目录中寻找一个叫 `config` 的文件。 用户可以通过
`KUBECONFIG` 环境变量来设置其它的 kubeconfig 文件。

创建和设置 kubeconfig 文件的详细指南，见
[配置多集群访问](/k8sDocs/docs/tasks/access-application-cluster/configure-access-multiple-clusters).


<!-- body -->
<!--
## Supporting multiple clusters, users, and authentication mechanisms

Suppose you have several clusters, and your users and components authenticate
in a variety of ways. For example:

- A running kubelet might authenticate using certificates.
- A user might authenticate using tokens.
- Administrators might have sets of certificates that they provide to individual users.

With kubeconfig files, you can organize your clusters, users, and namespaces.
You can also define contexts to quickly and easily switch between
clusters and namespaces.
 -->

## 支持多集群，用户，认证机制 {#supporting-multiple-clusters-users-and-authentication-mechanisms}

假设你有几个集群，并且你的用户和组件有不同的认证方式。 例如:

- 一个运行的 kubelet 可能使用证书来认证。
- 一个用户可能使用使用令牌来认证
- 管理员可能为每个用户提供认证证书

通过 kubeconfig 文件，就可以组织你的集群，用户， 和命名空间。 你也可以通过定义上下文的方式实现
快速在不同集群和命名空间之间切换。

<!--
## Context

A *context* element in a kubeconfig file is used to group access parameters
under a convenient name. Each context has three parameters: cluster, namespace, and user.
By default, the `kubectl` command-line tool uses parameters from
the *current context* to communicate with the cluster.

To choose the current context:
```
kubectl config use-context
```
 -->

## 上下文 {#context}

在一个 kubeconfig 文件中的一个 *上下文* 元素以一个方便的名称的方式来组织访问参数。每个上下文
有三个参数: 集群，命名空间，和用户。 默认情况下， `kubectl` 命令行工具使用来自 *当前上下文*
中的参数来与集群通信。

要选择当前上下文,使用下面的命令:

```shell
kubectl config use-context
```

<!--
## The KUBECONFIG environment variable

The `KUBECONFIG` environment variable holds a list of kubeconfig files.
For Linux and Mac, the list is colon-delimited. For Windows, the list
is semicolon-delimited. The `KUBECONFIG` environment variable is not
required. If the `KUBECONFIG` environment variable doesn't exist,
`kubectl` uses the default kubeconfig file, `$HOME/.kube/config`.

If the `KUBECONFIG` environment variable does exist, `kubectl` uses
an effective configuration that is the result of merging the files
listed in the `KUBECONFIG` environment variable.
 -->


## `KUBECONFIG` 环境变量 {#the-kubeconfig-environment-variable}

`KUBECONFIG` 环境变量中包含的是一个 kubeconfig 文件的列表。 对于 Linux 和 Mac，列表是用
冒号(`:`)分隔。 对于 Windows， 列表分隔符是分号(`;`).  `KUBECONFIG` 环境变量不是必要的。
如果 `KUBECONFIG` 环境变量不存在， `kubectl` 会使用默认的 kubeconfig 文件 `$HOME/.kube/config`

如果 `KUBECONFIG` 环境变量是存在的，`kubectl` 会使用一个由 `KUBECONFIG` 环境变量列举的
文件合并结果作为有效配置。

<!--
## Merging kubeconfig files

To see your configuration, enter this command:

```shell
kubectl config view
```

As described previously, the output might be from a single kubeconfig file,
or it might be the result of merging several kubeconfig files.

Here are the rules that `kubectl` uses when it merges kubeconfig files:

1. If the `--kubeconfig` flag is set, use only the specified file. Do not merge.
   Only one instance of this flag is allowed.

   Otherwise, if the `KUBECONFIG` environment variable is set, use it as a
   list of files that should be merged.
   Merge the files listed in the `KUBECONFIG` environment variable
   according to these rules:

   * Ignore empty filenames.
   * Produce errors for files with content that cannot be deserialized.
   * The first file to set a particular value or map key wins.
   * Never change the value or map key.
     Example: Preserve the context of the first file to set `current-context`.
     Example: If two files specify a `red-user`, use only values from the first file's `red-user`.
     Even if the second file has non-conflicting entries under `red-user`, discard them.

   For an example of setting the `KUBECONFIG` environment variable, see
   [Setting the KUBECONFIG environment variable](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable).

   Otherwise, use the default kubeconfig file, `$HOME/.kube/config`, with no merging.

1. Determine the context to use based on the first hit in this chain:

    1. Use the `--context` command-line flag if it exists.
    1. Use the `current-context` from the merged kubeconfig files.

   An empty context is allowed at this point.

1. Determine the cluster and user. At this point, there might or might not be a context.
   Determine the cluster and user based on the first hit in this chain,
   which is run twice: once for user and once for cluster:

   1. Use a command-line flag if it exists: `--user` or `--cluster`.
   1. If the context is non-empty, take the user or cluster from the context.

   The user and cluster can be empty at this point.

1. Determine the actual cluster information to use. At this point, there might or
   might not be cluster information.
   Build each piece of the cluster information based on this chain; the first hit wins:

   1. Use command line flags if they exist: `--server`, `--certificate-authority`, `--insecure-skip-tls-verify`.
   1. If any cluster information attributes exist from the merged kubeconfig files, use them.
   1. If there is no server location, fail.

1. Determine the actual user information to use. Build user information using the same
   rules as cluster information, except allow only one authentication
   technique per user:

   1. Use command line flags if they exist: `--client-certificate`, `--client-key`, `--username`, `--password`, `--token`.
   1. Use the `user` fields from the merged kubeconfig files.
   1. If there are two conflicting techniques, fail.

1. For any information still missing, use default values and potentially
   prompt for authentication information.
 -->

## 合并 `kubeconfig` 文件 {#merging-kubeconfig-files}

要看现在的配置，输入下面的命令:

```shell
kubectl config view
```

就如之间所描述的， 输出结果可能来自单个 kubeconfig 文件，也可能是几个 kubeconfig 文件合并的
结果。

以下为 `kubectl` 在合并 kubeconfig 文件使用的规则:

1. 如果设置 `--kubeconfig` 标志，就只使用这个指定的文件。不合并配置文件。这个标志只允许有一个实例。

   否则，如果设置了 `KUBECONFIG` 环境变量，就会使用其中列举文件合并的结果。 合并 `KUBECONFIG`
   环境变量中列举的文件合并会依照如下规则:
   - 忽略空文件名。
   - 文件内容不能被反序列化则会生成错误
   - 排在前面的文件中的键(以及其值)会作为最终结果中的键值
   - 永远不修改值或字典键
    示例: 保留第一个设置 `current-context` 文件中的上下文。
    示例: 如果有两个文件配置了 `red-user`, 使用排在前面的那个文件 `red-user` 的值。
    即便第二个文件 `red-user` 的实体与第一个不冲突，也会把它们整体都丢弃了

   这里有一个 `KUBECONFIG` 环境变量的示例，详见
   [设置 `KUBECONFIG` 环境变量](/k8sDocs/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable).

   再没有就使用默认的 `kubeconfig` 文件，`$HOME/.kube/config`， 没得其它合并。

2. 基于这个链的第一个命中来决定上下文:
    1. 如果有则使用 `--context` 命令行标志
    2. 使用合并 `kubeconfig` 文件中的 `current-context`

   在这个点允许空的上下文

3. 确定集群和用户，在这个点上，可能有也可能没有上下文。 基于下面这个链中的第一个命中的集群和用户，
   这个会被运行两次: 一次用于用户和一次用于集群

   1. 如果有命令标志则使用该标志: `--user` 或 `--cluster`.
   2. 如果上下文不是空的，则使用这个上下文中的用户或集群

   在这一级用户和集群可以是空的

4. 决定实际使用的集群信息， 在这一级， 可能有也可能没有集群信息。 会依据下面这个链中的片段来构建集群信息；
   第一个命中会作为最终结果:

   1. 即便存在，也使用的命令标志: `--server`, `--certificate-authority`, `--insecure-skip-tls-verify`.
   2. 如果合并后的 kubeconfig 文件中有任意集群信息的属性存在，这使用它们。
   3. 如果其中没有服务地址信息，则会失败

5. 决定实际使用的用户信息。 构建用户信息使用与集群信息相同的规则，只是限制一个用户只能有一个认证机制:

   1. 如果存在则使用的标记: `--client-certificate`, `--client-key`, `--username`, `--password`, `--token`.
   2. 使用合并后的 kubeconfig 文件中的 `user` 字段。
   3. 如果有两种冲突的认证机制，则会失败

6. 对于到这一步还缺失的信息，使用默认值和潜在的输入提示认证信息。

<!--

## File references

File and path references in a kubeconfig file are relative to the location of the kubeconfig file.
File references on the command line are relative to the current working directory.
In `$HOME/.kube/config`, relative paths are stored relatively, and absolute paths
are stored absolutely.
 -->

## 文件引用 {#file-references}

文件和 kubeconfig 文件中的路径引用是相同对 kubeconfig 文件的路径的。 命令行的文件引用是
相对于目前的工作目录的。 在 `$HOME/.kube/config` 中, 相对路径以相路径式存储，绝对路径以绝对路径存储。


## {{% heading "whatsnext" %}}

<!--
* [Configure Access to Multiple Clusters](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
* [`kubectl config`](/docs/reference/generated/kubectl/kubectl-commands#config)
-->

* [配置多集群访问](/k8sDocs/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
* [`kubectl config`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)
