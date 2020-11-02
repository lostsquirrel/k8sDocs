---
title: Deployment
date: 2020-08-14
publishdate: 2020-08-20
weight: 2030100
---
<!--
---
reviewers:
- janetkuo
title: Deployments
feature:
  title: Automated rollouts and rollbacks
  description: >
    Kubernetes progressively rolls out changes to your application or its configuration, while monitoring application health to ensure it doesn't kill all your instances at the same time. If something goes wrong, Kubernetes will rollback the change for you. Take advantage of a growing ecosystem of deployment solutions.

content_type: concept
weight: 10
---
 -->
<!-- overview -->
<!--
A _Deployment_ provides declarative updates for {{< glossary_tooltip text="Pods" term_id="pod" >}}
{{< glossary_tooltip term_id="replica-set" text="ReplicaSets" >}}.

You describe a _desired state_ in a Deployment, and the Deployment {{< glossary_tooltip term_id="controller" >}} changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

{{< note >}}
Do not manage ReplicaSets owned by a Deployment. Consider opening an issue in the main Kubernetes repository if your use case is not covered below.
{{< /note >}}
 -->
_Deployment_ 为 {{< glossary_tooltip term_id="replica-set" text="ReplicaSets" >}} 管理的 {{< glossary_tooltip text="Pods" term_id="pod" >}}
提供了声明式的更新。

用户在 Deployment 中描述了一个 _期望状态_， 然后这个 Deployment {{< glossary_tooltip term_id="controller" >}}
以控制器的方式变更实际状态为期望状态。 用户可以定义 Deployment 来创建一个新的 `ReplicaSet`，
或者删除一个已经存在的 Deployment， 再用一个新的 Deployment 来接管它的所以资源。

{{< note >}}
不要管属于 `Deployment` 的 `ReplicaSet`。 如果以下提到的应用场景都没办法满足你的需求请考虑到 k8s 主仓库提一个问题单
{{< /note >}}

<!-- body -->
<!--
## Use Case

The following are typical use cases for Deployments:

* [Create a Deployment to rollout a ReplicaSet](#creating-a-deployment). The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
* [Declare the new state of the Pods](#updating-a-deployment) by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
* [Rollback to an earlier Deployment revision](#rolling-back-a-deployment) if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
* [Scale up the Deployment to facilitate more load](#scaling-a-deployment).
* [Pause the Deployment](#pausing-and-resuming-a-deployment) to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
* [Use the status of the Deployment](#deployment-status) as an indicator that a rollout has stuck.
* [Clean up older ReplicaSets](#clean-up-policy) that you don't need anymore.
 -->
## 应用场景

以下为 `Deployment` 常见应用场景:

* [使用 Deployment 来管理 ReplicaSet](#creating-a-deployment). ReplicaSet 会在后台创建 Pod， 查看它发布的这些发布是否成功
* [声明 Pod 的新状态](#updating-a-deployment) 通过更新 Deployment 的 Pod 模板定义(PodTemplateSpec). 创建一个新的 ReplicaSet 并以一个受控的频率将 Pod 从旧的 ReplicaSet 移到新的 ReplicaSet。每创建一个新的 ReplicaSet 都会更新一次 Deployment 的版本号
* [Deployment 回滚到之前一个版本](#rolling-back-a-deployment)， 如果 Deployment 当前部署的应用状态不稳定，每次回滚也会更新 Deployment 版本
* [让 Deployment 扩容以承载更多的负载](#scaling-a-deployment).
* [暂停 Deployment](#pausing-and-resuming-a-deployment) 后一次修改 PodTemplateSpec 的多个地方，然后恢复这个 Deployment 让它开始一个新的发布
* [通过 Deployment 的状态](#deployment-status) 来标示一个发布应用的状态
* [清除不需要的 ReplicaSet](#clean-up-policy).
<!--
## Creating a Deployment

The following is an example of a Deployment. It creates a ReplicaSet to bring up three `nginx` Pods:

{{< codenew file="controllers/nginx-deployment.yaml" >}}

In this example:

* A Deployment named `nginx-deployment` is created, indicated by the `.metadata.name` field.
* The Deployment creates three replicated Pods, indicated by the `.spec.replicas` field.
* The `.spec.selector` field defines how the Deployment finds which Pods to manage.
  In this case, you simply select a label that is defined in the Pod template (`app: nginx`).
  However, more sophisticated selection rules are possible,
  as long as the Pod template itself satisfies the rule.

  {{< note >}}
  The `.spec.selector.matchLabels` field is a map of {key,value} pairs.
  A single {key,value} in the `matchLabels` map is equivalent to an element of `matchExpressions`,
  whose key field is "key" the operator is "In", and the values array contains only "value".
  All of the requirements, from both `matchLabels` and `matchExpressions`, must be satisfied in order to match.
  {{< /note >}}

* The `template` field contains the following sub-fields:
  * The Pods are labeled `app: nginx`using the `.metadata.labels` field.
  * The Pod template's specification, or `.template.spec` field, indicates that
  the Pods run one container, `nginx`, which runs the `nginx`
  [Docker Hub](https://hub.docker.com/) image at version 1.14.2.
  * Create one container and name it `nginx` using the `.spec.template.spec.containers[0].name` field.

Before you begin, make sure your Kubernetes cluster is up and running.
Follow the steps given below to create the above Deployment:


1. Create the Deployment by running the following command:

   ```shell
   kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
   ```

  {{< note >}}
  You can specify the `--record` flag to write the command executed in the resource annotation `kubernetes.io/change-cause`.
  The recorded change is useful for future introspection. For example, to see the commands executed in each Deployment revision.
  {{< /note >}}


2. Run `kubectl get deployments` to check if the Deployment was created.

   If the Deployment is still being created, the output is similar to the following:
   ```shell
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-deployment   0/3     0            0           1s
   ```
   When you inspect the Deployments in your cluster, the following fields are displayed:
   * `NAME` lists the names of the Deployments in the namespace.
   * `READY` displays how many replicas of the application are available to your users. It follows the pattern ready/desired.
   * `UP-TO-DATE` displays the number of replicas that have been updated to achieve the desired state.
   * `AVAILABLE` displays how many replicas of the application are available to your users.
   * `AGE` displays the amount of time that the application has been running.

   Notice how the number of desired replicas is 3 according to `.spec.replicas` field.

3. To see the Deployment rollout status, run `kubectl rollout status deployment.v1.apps/nginx-deployment`.

   The output is similar to:
   ```shell
   Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
   deployment.apps/nginx-deployment successfully rolled out
   ```

4. Run the `kubectl get deployments` again a few seconds later.
   The output is similar to this:
   ```shell
   NAME               READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-deployment   3/3     3            3           18s
   ```
   Notice that the Deployment has created all three replicas, and all replicas are up-to-date (they contain the latest Pod template) and available.

5. To see the ReplicaSet (`rs`) created by the Deployment, run `kubectl get rs`. The output is similar to this:
   ```shell
   NAME                          DESIRED   CURRENT   READY   AGE
   nginx-deployment-75675f5897   3         3         3       18s
   ```
   ReplicaSet output shows the following fields:

   * `NAME` lists the names of the ReplicaSets in the namespace.
   * `DESIRED` displays the desired number of _replicas_ of the application, which you define when you create the Deployment. This is the _desired state_.
   * `CURRENT` displays how many replicas are currently running.
   * `READY` displays how many replicas of the application are available to your users.
   * `AGE` displays the amount of time that the application has been running.

   Notice that the name of the ReplicaSet is always formatted as `[DEPLOYMENT-NAME]-[RANDOM-STRING]`.
   The random string is randomly generated and uses the `pod-template-hash` as a seed.

6. To see the labels automatically generated for each Pod, run `kubectl get pods --show-labels`.
   The output is similar to:
   ```shell
   NAME                                READY     STATUS    RESTARTS   AGE       LABELS
   nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
   nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
   nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
   ```
   The created ReplicaSet ensures that there are three `nginx` Pods.

{{< note >}}
You must specify an appropriate selector and Pod template labels in a Deployment
(in this case, `app: nginx`).

Do not overlap labels or selectors with other controllers (including other Deployments and StatefulSets).
Kubernetes doesn't stop you from overlapping, and if multiple controllers have overlapping selectors those controllers might conflict and behave unexpectedly.
{{< /note >}}
 -->
## 创建一个 Deployment

以下为一个 Deployment 的示例。创建一个 ReplicaSet 来启动三个 `nginx` 的 Pod：

{{< codenew file="controllers/nginx-deployment.yaml" >}}

在这个示例中:

- 创建了一个叫 `nginx-deployment` Deployment， 这个名字定义在 `.metadata.name` 字段
- 这个 Deployment 会创建三个 Pod 为副本， 由 `.spec.replicas` 字段定义
- `.spec.selector` 字段定义 Deployment 怎么去找需要它管理的 Pod. 在本例中，只是定义在 Pod 模板中的
  标签(`app: nginx`), 更复杂的选择规则也是支持的，只要 Pod 模板满足这些规则
  {{< note >}}
  `.spec.selector.matchLabels` 是一个键值对 ({key,value}) 字典。
  `matchLabels` 中的一个键值对相当于 `matchExpressions` 中的一个元素
  这个元素的键就是值对的键，操作符为 `In`， 值为包含该值的数组。
  需要同时满足 `matchLabels` 和 `matchExpressions` 中的条件，才是符合该选择器的目标
  {{< /note >}}

- `template` 字段又包含以下子字段:
  - `.metadata.labels` 字段中的 `app: nginx` 是 Pod 的标签
  - `.template.spec` 字段为 Pod 的模板定义， 表示这个 Pod 运行一个容器， 应用程序为 `nginx`  
    运行的是 [Docker Hub](https://hub.docker.com/) 上面的 `1.14.2` 版本的 nginx 镜像
  - `.spec.template.spec.containers[0].name` 字段表示，创建的第一个也只有一个容器，它的名字为 `nginx`

在开始之前，需要保证 k8s 集群已经启动并运行。

以下为创建这个 Deployment 的具体步骤:

1. 通过以下命令创建该 Deployment：

  ```shell
  kubectl apply -f nginx-deployment.yaml
  ```

  {{< note >}}
  用户可以通过在命令中添加 `--record` 来把执行的命令写入到资源的 `kubernetes.io/change-cause` 注解 中。
  这个变更记录对于将来的回溯是很有用的。比如，查看 Deployment 每一个版本执行了什么命令
  {{< /note >}}

2. 运行 `kubectl get deployments` 命令，查看 Deployment 是否已经创建

  如果 Deployment 还在创建中，则输出结果如下:
  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   0/3     0            0           1s
  ```

  其中每个字段的说明如下:
  - `NAME` 显示 Deployment 的名称
  - `READY` 展示应用的副本可用的数量， 格式为 就绪数/期望数
  - `UP-TO-DATE` 展示已经被更新达到预期状态的副本的数量
  - `AVAILABLE` 展示已经能为用户提供服务的副本数量
  - `AGE` 应用运行的时长

  可以看到，根据 `.spec.replicas` 字段， 期望的副本数量是 `3`

3. 要查看 Deployment 发布状态，可以通过命令 `kubectl rollout status deployment.v1.apps/nginx-deployment`
  输出类似如下:

  ```
  Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
  deployment.apps/nginx-deployment successfully rolled out
  ```  

4. 等几秒再执行命令 `kubectl get deployments`
  输出类似如下:

  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3/3     3            3           18s
  ```  

  这时可以看到 Deployment 已经创建全部的三个副本， 所有副本的达成更新(是通过最新的 Pod 模板创建的)并且可用的

5. 通过命令 `kubectl get rs` 查看由 Deployment 创建的 ReplicaSet (`rs`)
  输出类似如下:

  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-75675f5897   3         3         3       18s
  ```  

  字段说明如下：
  - `NAME` ReplicaSet 的名称
  - `DESIRED` 应用的 _期望_ 副本数，创建 Deployment 时指定。 这是期望状态
  - `CURRENT` 展示当前正在运行的副本数
  - `READY` 展示当前可以提供服务的副本数
  - `AGE` 展示应用的运行时长

  可以看到 ReplicaSet 的名称格式总是为 `[DEPLOYMENT-NAME]-[RANDOM-STRING]`
  为个随机字符串是以 `pod-template-hash` 为种子生成的

6. 要查看每个 Pod 自动创建的标签，可能执行命令 `kubectl get pods --show-labels`:
  输出类似如下:

  ```
  NAME                                READY     STATUS    RESTARTS   AGE       LABELS
  nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
  ```
  ReplicaSet 会确保有三个 `nginx` Pod

{{< note >}}
用户应该在 Deployment 中定义恰当的选择器和 Pod 模板标签(本例为 `app: nginx`)
一定不要与其它的控制器(包括其它的 Deployment 和 StatefulSet)使用一样的标签或选择器
{{< /note >}}
<!--
### Pod-template-hash label

{{< caution >}}
Do not change this label.
{{< /caution >}}

The `pod-template-hash` label is added by the Deployment controller to every ReplicaSet that a Deployment creates or adopts.

This label ensures that child ReplicaSets of a Deployment do not overlap. It is generated by hashing the `PodTemplate` of the ReplicaSet and using the resulting hash as the label value that is added to the ReplicaSet selector, Pod template labels,
and in any existing Pods that the ReplicaSet might have.
 -->
### 标签 `pod-template-hash`

{{< caution >}}
这个标签一定不要改
{{< /caution >}}

`pod-template-hash` 是由 Deployment 的控制器添加到每个由 Deployment 创建或收编的 ReplicaSet 的。
这个标签确保 Deployment 所属的 ReplicaSet 的标签不会重叠。 这个标签的值是对 ReplicaSet 的 `PodTemplate` 进行哈希的结果，
并将这个标签添加到 ReplicaSet 的选择器和 Pod 模板的标签中，然后也会添加到已经由 ReplicaSet 管理或将来收编的所以 Pod 中。
<!--
## Updating a Deployment

{{< note >}}
A Deployment's rollout is triggered if and only if the Deployment's Pod template (that is, `.spec.template`)
is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout.
{{< /note >}}

Follow the steps given below to update your Deployment:

1. Let's update the nginx Pods to use the `nginx:1.16.1` image instead of the `nginx:1.14.2` image.

    ```shell
    kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```
    or simply use the following command:

    ```shell
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment image updated
    ```

    Alternatively, you can `edit` the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.14.2` to `nginx:1.16.1`:

    ```shell
    kubectl edit deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment edited
    ```

2. To see the rollout status, run:

    ```shell
    kubectl rollout status deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    ```
    or
    ```
    deployment.apps/nginx-deployment successfully rolled out
    ```

Get more details on your updated Deployment:

* After the rollout succeeds, you can view the Deployment by running `kubectl get deployments`.
    The output is similar to this:
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           36s
    ```

* Run `kubectl get rs` to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it
up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.

    ```shell
    kubectl get rs
    ```

    The output is similar to this:
    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       6s
    nginx-deployment-2035384211   0         0         0       36s
    ```

* Running `get pods` should now show only the new Pods:

    ```shell
    kubectl get pods
    ```

    The output is similar to this:
    ```
    NAME                                READY     STATUS    RESTARTS   AGE
    nginx-deployment-1564180365-khku8   1/1       Running   0          14s
    nginx-deployment-1564180365-nacti   1/1       Running   0          14s
    nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
    ```

    Next time you want to update these Pods, you only need to update the Deployment's Pod template again.

    Deployment ensures that only a certain number of Pods are down while they are being updated. By default,
    it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).

    Deployment also ensures that only a certain number of Pods are created above the desired number of Pods.
    By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

    For example, if you look at the above Deployment closely, you will see that it first created a new Pod,
    then deleted some old Pods, and created new ones. It does not kill old Pods until a sufficient number of
    new Pods have come up, and does not create new Pods until a sufficient number of old Pods have been killed.
    It makes sure that at least 2 Pods are available and that at max 4 Pods in total are available.

* Get details of your Deployment:
  ```shell
  kubectl describe deployments
  ```
  The output is similar to this:
  ```
  Name:                   nginx-deployment
  Namespace:              default
  CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
  Labels:                 app=nginx
  Annotations:            deployment.kubernetes.io/revision=2
  Selector:               app=nginx
  Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
     Containers:
      nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
      Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
    ```
    Here you see that when you first created the Deployment, it created a ReplicaSet (nginx-deployment-2035384211)
    and scaled it up to 3 replicas directly. When you updated the Deployment, it created a new ReplicaSet
    (nginx-deployment-1564180365) and scaled it up to 1 and then scaled down the old ReplicaSet to 2, so that at
    least 2 Pods were available and at most 4 Pods were created at all times. It then continued scaling up and down
    the new and the old ReplicaSet, with the same rolling update strategy. Finally, you'll have 3 available replicas
    in the new ReplicaSet, and the old ReplicaSet is scaled down to 0.
 -->
## 更新 Deployment

{{< note >}}
Deployment 有且仅有在对 Deployment Pod 模板(也就是 `.spec.template`)发生变更后触发发布，
例如，如果模板标签或容器被更新。 其它如扩充 Deployment 副本数不会触发发布。
{{< /note >}}

以下步骤更新 Deployment：

1. 将 Pod 使用的 `nginx` 镜像版本从 `nginx:1.14.2` 升到 `nginx:1.16.1`

  ```shell
  kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
  ```
  或者简单使用如下命令:

  ```shell
  kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
  ```

  输出结果类似如下:
  ```
  deployment.apps/nginx-deployment image updated
  ```

  另外还有一个方式，就是修改 Deployment 中 `.spec.template.spec.containers[0].image` 的值将其从原来的 `nginx:1.14.2` 改成 `nginx:1.16.1`:

  ```shell
  kubectl edit deployment.v1.apps/nginx-deployment
  ```

  输出结果类似如下:
  ```
  deployment.apps/nginx-deployment edited
  ```

2. 查看发布状态，执行如下命令:

  ```shell
  kubectl rollout status deployment.v1.apps/nginx-deployment
  ```

  输出结果类似如下:
  ```
  Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
  ```
  或
  ```
  deployment.apps/nginx-deployment successfully rolled out
  ```

查看更新后 Deployment 的更多细节:

- 当发布执行完成后，查看 Deployment 执行命令  `kubectl get deployments`：
 输出结果类似如下:
  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3/3     3            3           36s
  ```

- 执行 `kubectl get rs` 命令可以看到 Deployment 是通过创建一个新的 ReplicaSet 并将其副本数扩充到 3
同时将旧的 ReplicaSet 的副本数收缩至 0

  ```shell
  kubectl get rs
  ```

  输出结果类似如下:
  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-1564180365   3         3         3       6s
  nginx-deployment-2035384211   0         0         0       36s
  ```

- 这时再执行命令 `get pods`， 应该就只会看到新创建的 Pod:

  ```shell
  kubectl get pods
  ```

  输出结果类似如下:
  ```
  NAME                                READY     STATUS    RESTARTS   AGE
  nginx-deployment-1564180365-khku8   1/1       Running   0          14s
  nginx-deployment-1564180365-nacti   1/1       Running   0          14s
  nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
  ```

  当用户下次想更新这些 Pod 时， 只需要再次更新 Deployment 的 Pod 模板即可。

  Deployment 会确保在更新过程中挂掉的 Pod 数不会大于指定的值。 默认情况下，会保证至少有期望数量 75% 数量的 Pod 是正常运行的(最多只有 25% 是不可用的)

  Deployment 还会保证多于期望数量的 Pod 数不会大于指定的值。 默认情况下同时正常运行的 Pod 数不会大于期望数量的 125%(上限为 25%)

  如果仔细观察上面的例子。 就会发现更新过程中，首先会创建一个新的 Pod， 然后删除一些旧的，再创建新的。
  在新创建并就绪的 Pod 的数量没有达到特定数量之前不会干掉旧的 Pod， 同时在没干掉特定数量旧的 Pod 之前不会创建新的 Pod。
  这个过程中会保证在少的时候至少有 2 个 Pod 可用，在多的时候至多有 4 个 Pod 可用。

- 继续来看 Deployment 的更多细节：

  ```shell
  kubectl describe deployments
  ```
  输出结果类似如下:
  ```
  Name:                   nginx-deployment
  Namespace:              default
  CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
  Labels:                 app=nginx
  Annotations:            deployment.kubernetes.io/revision=2
  Selector:               app=nginx
  Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
     Containers:
      nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
      Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
  ```
  这里可以看到， 在第一次创建 Deployment 的时候， 创建了一个 ReplicaSet (nginx-deployment-2035384211) 并直接将副本数扩充到 3.
  当 更新 Deployment 的时候， 又创建了一个新的 ReplicaSet (nginx-deployment-1564180365) 并将副本数扩充到 1 ， 紧接着将旧的 ReplicaSet
  的副本数收缩至 2， 因此至始终少有 2 Pod 可用， 至多有 4 个 Pod 可用。 接下来继续使用相同的滚动更新策略扩充新的，收缩旧的 ReplicaSet。到最后就变成新的
  ReplicaSet 有 3 个副本可用， 旧的  ReplicaSet 副本数收缩为 0 。
  <!--
### Rollover (aka multiple updates in-flight)

Each time a new Deployment is observed by the Deployment controller, a ReplicaSet is created to bring up
the desired Pods. If the Deployment is updated, the existing ReplicaSet that controls Pods whose labels
match `.spec.selector` but whose template does not match `.spec.template` are scaled down. Eventually, the new
ReplicaSet is scaled to `.spec.replicas` and all old ReplicaSets is scaled to 0.

If you update a Deployment while an existing rollout is in progress, the Deployment creates a new ReplicaSet
as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously
 -- it will add it to its list of old ReplicaSets and start scaling it down.

For example, suppose you create a Deployment to create 5 replicas of `nginx:1.14.2`,
but then update the Deployment to create 5 replicas of `nginx:1.16.1`, when only 3
replicas of `nginx:1.14.2` had been created. In that case, the Deployment immediately starts
killing the 3 `nginx:1.14.2` Pods that it had created, and starts creating
`nginx:1.16.1` Pods. It does not wait for the 5 replicas of `nginx:1.14.2` to be created
before changing course.
 -->
### Rollover (aka multiple updates in-flight) 没想到比较恰当的

每次 Deployment 控制器发现新创建了一个 Deployment， 就会创建一个新的 ReplicaSet 来创建期望数量的 Pod.
如 Deployment 被更新后， 对于之前存在的 ReplicaSet， 它控制的 Pod 的标签能够匹配 `.spec.selector` 但是模板不能匹配 `.spec.template`，副本数量就会被收缩。
最后， 新创建的 ReplicaSet 的副本数扩充到 `.spec.replicas` 配置的数量。 所以旧的 ReplicaSet 副本收缩为 0。

如果在前一个发布还在进行时又更新的 Deployment， Deployment 会为被次更新创建一个对应的 ReplicaSet，就开始扩容。
前一个正在扩容的 ReplicaSet 就会被跳过。 它会添加到旧 ReplicaSet 的列表，然后开始收缩容量。

例如， 假设创建一个包含 5 个 `nginx:1.14.2`  副本的 Deployment 这时将 Deployment 改为包含 `nginx:1.16.1` 副本， 此时只创建了 3 个 `nginx:1.14.2` 副本。
在这种情况下， Deployment 会马上开始干掉 已经被创建的 3 个 `nginx:1.14.2` Pod， 并开始创建 `nginx:1.16.1` Pod。 并不会等 5 个 `nginx:1.14.2` 创建完成后才应用变更引发的任务
<!--
### Label selector updates

It is generally discouraged to make label selector updates and it is suggested to plan your selectors up front.
In any case, if you need to perform a label selector update, exercise great caution and make sure you have grasped
all of the implications.

{{< note >}}
In API version `apps/v1`, a Deployment's label selector is immutable after it gets created.
{{< /note >}}

* Selector additions require the Pod template labels in the Deployment spec to be updated with the new label too,
otherwise a validation error is returned. This change is a non-overlapping one, meaning that the new selector does
not select ReplicaSets and Pods created with the old selector, resulting in orphaning all old ReplicaSets and
creating a new ReplicaSet.
* Selector updates changes the existing value in a selector key -- result in the same behavior as additions.
* Selector removals removes an existing key from the Deployment selector -- do not require any changes in the
Pod template labels. Existing ReplicaSets are not orphaned, and a new ReplicaSet is not created, but note that the
removed label still exists in any existing Pods and ReplicaSets.
 -->
### 标签选择器的更新

通常不建议在标签选择器上做修改，而是预先就想好标签选择器。 在任何你想要修改标签器的情况下，一定要小心小心再小心， 确保已经理清楚所有的头绪。

{{< note >}}
对于版本为 `apps/v1` API， Deployment 在创建后不可变更。
{{< /note >}}

- 标签选择器添加条目必须要 Deployment 的 Pod 模板的标签也要添加该条目， 否则会返回一个校验错误。 也不能让这个变更与原来的不重叠，不重叠的意思是新的选择器
  不能匹配到旧的选择创建的 ReplicaSet 主 Pod， 导致旧的 ReplicaSet 脱钩 然后创建新的ReplicaSet

- 标签选择器更新的是其中的已经存在的一个键的值， 与添加条目的行为一至。
- 标签选择器更新的删除一个存在的键， 不会对 Pod 模板做任何修改。 存在的 ReplicaSet 不会脱钩， 也不会创建新的 ReplicaSet， 但要注意删除的标签键，依然存在于这些
  ReplicaSet 和 Pod 中

<!--
## Rolling Back a Deployment

Sometimes, you may want to rollback a Deployment; for example, when the Deployment is not stable, such as crash looping.
By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want
(you can change that by modifying revision history limit).

{{< note >}}
A Deployment's revision is created when a Deployment's rollout is triggered. This means that the
new revision is created if and only if the Deployment's Pod template (`.spec.template`) is changed,
for example if you update the labels or container images of the template. Other updates, such as scaling the Deployment,
do not create a Deployment revision, so that you can facilitate simultaneous manual- or auto-scaling.
This means that when you roll back to an earlier revision, only the Deployment's Pod template part is
rolled back.
{{< /note >}}

* Suppose that you made a typo while updating the Deployment, by putting the image name as `nginx:1.161` instead of `nginx:1.16.1`:

    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment image updated
    ```

* The rollout gets stuck. You can verify it by checking the rollout status:

    ```shell
    kubectl rollout status deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    ```

* Press Ctrl-C to stop the above rollout status watch. For more information on stuck rollouts,
[read more here](#deployment-status).

* You see that the number of old replicas (`nginx-deployment-1564180365` and `nginx-deployment-2035384211`) is 2, and new replicas (nginx-deployment-3066724191) is 1.

    ```shell
    kubectl get rs
    ```

    The output is similar to this:
    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       25s
    nginx-deployment-2035384211   0         0         0       36s
    nginx-deployment-3066724191   1         1         0       6s
    ```

* Looking at the Pods created, you see that 1 Pod created by new ReplicaSet is stuck in an image pull loop.

    ```shell
    kubectl get pods
    ```

    The output is similar to this:
    ```
    NAME                                READY     STATUS             RESTARTS   AGE
    nginx-deployment-1564180365-70iae   1/1       Running            0          25s
    nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
    nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
    nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
    ```

    {{< note >}}
    The Deployment controller stops the bad rollout automatically, and stops scaling up the new ReplicaSet. This depends on the rollingUpdate parameters (`maxUnavailable` specifically) that you have specified. Kubernetes by default sets the value to 25%.
    {{< /note >}}

* Get the description of the Deployment:
    ```shell
    kubectl describe deployment
    ```

    The output is similar to this:
    ```
    Name:           nginx-deployment
    Namespace:      default
    CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
    Labels:         app=nginx
    Selector:       app=nginx
    Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
    StrategyType:       RollingUpdate
    MinReadySeconds:    0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.161
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    ReplicaSetUpdated
    OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
    NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
    Events:
      FirstSeen LastSeen    Count   From                    SubObjectPath   Type        Reason              Message
      --------- --------    -----   ----                    -------------   --------    ------              -------
      1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
    ```

  To fix this, you need to rollback to a previous revision of Deployment that is stable.
 -->
## Deployment 回滚示例

有时候会需要对 Deployment 进行回滚，例如当一个 Deployment 进入像无限崩溃等不稳定情况时。
默认情况下系统会保存 Deployment 所有的发布历史以便用户通过回滚到之前的任意版本(用户可以修改限制保留历史数)

{{< note >}}
Deployment 版本在 Deployment 发布(rollout)时触发。 也就是说一个新的版本在且仅在 Deployment 的 Pod 模板 (`.spec.template`) 发生变更，
如更新了模板中的标签或容器的镜像。 其它的如扩充 Deployment 副本数是不会创建版本的，这样在手动或自动伸缩容量时不会引起版本的增加。
也就是说当用户回滚到一个之前的版本时，只有 Deployment 的 Pod 模板部署回到了这个版本的状态。
{{< /note >}}

- 假设在将 Deployment 的镜像名改为 `nginx:1.16.1` 时，手抖写成了 `nginx:1.161`

  ```shell
  kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
  ```

  输出结果类似如下:
  ```
  deployment.apps/nginx-deployment image updated
  ```

- 这时这个发布就卡住了， 可以通过以下命令查看发布状态

  ```shell
  kubectl rollout status deployment.v1.apps/nginx-deployment
  ```

  输出结果类似如下:
  ```
  Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
  ```

- `Ctrl-C` 退出发布状态监控， 要查看卡住的 Deployment, 见[这里](#deployment-status)

- 这时候可以看到旧的副本数 (`nginx-deployment-1564180365` 和 `nginx-deployment-2035384211`) 是 2, 新的副本数 (`nginx-deployment-3066724191`) 是 1.

  ```shell
  kubectl get rs
  ```

  输出结果类似如下:
  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-1564180365   3         3         3       25s
  nginx-deployment-2035384211   0         0         0       36s
  nginx-deployment-3066724191   1         1         0       6s
  ```

- 再来看创建的 Pod， 这时候看到由新的  ReplicaSet 因为拉取镜像出问题卡在那了

  ```shell
  kubectl get pods
  ```

  输出结果类似如下:
  ```
  NAME                                READY     STATUS             RESTARTS   AGE
  nginx-deployment-1564180365-70iae   1/1       Running            0          25s
  nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
  nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
  nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
  ```

  {{< note >}}
  Deployment 控制器会自动停止错误的发布， 也会停止扩充新 ReplicaSet 的副本数。 这个行为信赖于配置的更新参数 (也就是`maxUnavailable`)，
  k8s 中默认设置为
  {{< /note >}}25%。

- 查看 Deployment 的描述信息

  ```shell
  kubectl describe deployment
  ```

  输出结果类似如下:
  ```
  Name:           nginx-deployment
  Namespace:      default
  CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
  Labels:         app=nginx
  Selector:       app=nginx
  Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
  StrategyType:       RollingUpdate
  MinReadySeconds:    0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
    Containers:
     nginx:
      Image:        nginx:1.161
      Port:         80/TCP
      Host Port:    0/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    ReplicaSetUpdated
  OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
  NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
  Events:
    FirstSeen LastSeen    Count   From                    SubObjectPath   Type        Reason              Message
    --------- --------    -----   ----                    -------------   --------    ------              -------
    1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
    22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
    21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
    21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
    13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
    13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
  ```

  而要修复这个问题，就需要将 Deployment 回滚到上一个稳定版本
<!--
### Checking Rollout History of a Deployment

Follow the steps given below to check the rollout history:

1. First, check the revisions of this Deployment:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```
    The output is similar to this:
    ```
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
    2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
    ```

    `CHANGE-CAUSE` is copied from the Deployment annotation `kubernetes.io/change-cause` to its revisions upon creation. You can specify the`CHANGE-CAUSE` message by:

    * Annotating the Deployment with `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"`
    * Append the `--record` flag to save the `kubectl` command that is making changes to the resource.
    * Manually editing the manifest of the resource.

2. To see the details of each revision, run:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
    ```

    The output is similar to this:
    ```
    deployments "nginx-deployment" revision 2
      Labels:       app=nginx
              pod-template-hash=1159050644
      Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
      Containers:
       nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
         QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
      No volumes.
    ```
 -->
### 查看 Deployment 的历史版本

以下步骤可以查看布历史:

1. 第一步， 查看 Deployment 历史版本列表:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```
    输出结果类似如下:
    ```
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
    2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
    ```

    `CHANGE-CAUSE` 是从 Deployment 中 `kubernetes.io/change-cause` 注解中记录的信息。 可能通过以下方式指定 `CHANGE-CAUSE` 中的消息
    - 在 Deployment 上打注解 `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"`
    - 在修改该资源的 `kubectl` 命令中添加 `--record` 标志
    - 人工修改资源配置文件

2. 要查看每个版本的详细信息，执行如下命令:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
    ```

    输出结果类似如下:
    ```
    deployments "nginx-deployment" revision 2
      Labels:       app=nginx
              pod-template-hash=1159050644
      Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
      Containers:
       nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
         QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
      No volumes.
    ```

<!--
### Rolling Back to a Previous Revision
Follow the steps given below to rollback the Deployment from the current version to the previous version, which is version 2.

1. Now you've decided to undo the current rollout and rollback to the previous revision:
    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment rolled back
    ```
    Alternatively, you can rollback to a specific revision by specifying it with `--to-revision`:

    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment rolled back
    ```

    For more details about rollout related commands, read [`kubectl rollout`](/docs/reference/generated/kubectl/kubectl-commands#rollout).

    The Deployment is now rolled back to a previous stable revision. As you can see, a `DeploymentRollback` event
    for rolling back to revision 2 is generated from Deployment controller.

2. Check if the rollback was successful and the Deployment is running as expected, run:
    ```shell
    kubectl get deployment nginx-deployment
    ```

    The output is similar to this:
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           30m
    ```
3. Get the description of the Deployment:
    ```shell
    kubectl describe deployment nginx-deployment
    ```
    The output is similar to this:
    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=4
                            kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
    Events:
      Type    Reason              Age   From                   Message
      ----    ------              ----  ----                   -------
      Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
      Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
      Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
    ```

 -->
### 回滚到之前一个版本
以下步骤给出把 Deployment 版本从当前版本回滚到之前的版本， 这里是编号为2的版本

1. 将当前版本回滚到之前一个版本:
    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment
    ```

    输出结果类似如下：
    ```
    deployment.apps/nginx-deployment rolled back
    ```
    另外还有一个方式， 可以通过 `--to-revision` 参数指定回滚到指定版本:

    ```shell
    kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
    ```

    输出结果类似如下：
    ```
    deployment.apps/nginx-deployment rolled back
    ```

    更多发布相关的命令，见[`kubectl rollout`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#rollout).
    这时 Deployment 就已经回滚到之前的稳定版本。如你所见， 一个用来将 Deployment 回滚到版本 2 的 `DeploymentRollback` 事件由 Deployment 控制器创建

2. 检查回滚是否成功， Deployment 是否达到预期， 执行发中下命令:
    ```shell
    kubectl get deployment nginx-deployment
    ```

    输出结果类似如下：
    ```
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           30m
    ```
3. 查看 Deployment 描述信息:
    ```shell
    kubectl describe deployment nginx-deployment
    ```
    输出结果类似如下：
    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
    Labels:                 app=nginx
    Annotations:            deployment.kubernetes.io/revision=4
                            kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
    Selector:               app=nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
    Events:
      Type    Reason              Age   From                   Message
      ----    ------              ----  ----                   -------
      Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
      Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
      Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
    ```
<!--
## Scaling a Deployment

You can scale a Deployment by using the following command:

```shell
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```
The output is similar to this:
```
deployment.apps/nginx-deployment scaled
```

Assuming [horizontal Pod autoscaling](/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) is enabled
in your cluster, you can setup an autoscaler for your Deployment and choose the minimum and maximum number of
Pods you want to run based on the CPU utilization of your existing Pods.

```shell
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```
The output is similar to this:
```
deployment.apps/nginx-deployment scaled
```
 -->
## Deployment 副本数管理

可以通过以下命令修改 Deployment 副本数:

```shell
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```
输出结果类似如下：
```
deployment.apps/nginx-deployment scaled
```

假设集群启用了 [Pod 水平扩展](/k8sDocs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)， 用户可以为 Deployment 设置
一个自动扩展器，可以根据 Pod 的 CPU 使用率选择 Pod 数量的下限和上限。

```shell
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```
输出结果类似如下：
```
deployment.apps/nginx-deployment scaled
```
<!--
### Proportional scaling

RollingUpdate Deployments support running multiple versions of an application at the same time. When you
or an autoscaler scales a RollingUpdate Deployment that is in the middle of a rollout (either in progress
or paused), the Deployment controller balances the additional replicas in the existing active
ReplicaSets (ReplicaSets with Pods) in order to mitigate risk. This is called *proportional scaling*.

For example, you are running a Deployment with 10 replicas, [maxSurge](#max-surge)=3, and [maxUnavailable](#max-unavailable)=2.

* Ensure that the 10 replicas in your Deployment are running.
  ```shell
  kubectl get deploy
  ```
  The output is similar to this:

  ```
  NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment     10        10        10           10          50s
  ```

* You update to a new image which happens to be unresolvable from inside the cluster.
    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment image updated
    ```

* The image update starts a new rollout with ReplicaSet nginx-deployment-1989198191, but it's blocked due to the
`maxUnavailable` requirement that you mentioned above. Check out the rollout status:
    ```shell
    kubectl get rs
    ```
      The output is similar to this:
    ```
    NAME                          DESIRED   CURRENT   READY     AGE
    nginx-deployment-1989198191   5         5         0         9s
    nginx-deployment-618515232    8         8         8         1m
    ```

* Then a new scaling request for the Deployment comes along. The autoscaler increments the Deployment replicas
to 15. The Deployment controller needs to decide where to add these new 5 replicas. If you weren't using
proportional scaling, all 5 of them would be added in the new ReplicaSet. With proportional scaling, you
spread the additional replicas across all ReplicaSets. Bigger proportions go to the ReplicaSets with the
most replicas and lower proportions go to ReplicaSets with less replicas. Any leftovers are added to the
ReplicaSet with the most replicas. ReplicaSets with zero replicas are not scaled up.

In our example above, 3 replicas are added to the old ReplicaSet and 2 replicas are added to the
new ReplicaSet. The rollout process should eventually move all replicas to the new ReplicaSet, assuming
the new replicas become healthy. To confirm this, run:

```shell
kubectl get deploy
```

The output is similar to this:
```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```
The rollout status confirms how the replicas were added to each ReplicaSet.
```shell
kubectl get rs
```

The output is similar to this:
```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```
 -->

### 同比例扩容

更新策略为 RollingUpdate 的 Deployment 支持一个应用同时运行多个版本。 当用户或自动容量管量器在发布过程中(正在进行或暂停)进行容量伸缩，
Deployment 控制器会新增的副本按已经存在的活跃 ReplicaSet (包含 Pod 的 ReplicaSet)中副本数的比例分散到各个ReplicaSet中与以降低风险。
这就被称为 *同比例扩容*.

例如， 用户运行一个有 10 个副本的 Deployment， 它的 [maxSurge](#max-surge)=3, [maxUnavailable](#max-unavailable)=2.
- 确保 Deployment 的 10 个副本正常运行
  ```shell
  kubectl get deploy
  ```
  输出结果类似如下:

  ```
  NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment     10        10        10           10          50s
  ```

- 更新了一个集群中不可用的镜像
    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
    ```

    输出结果类似如下:
    ```
    deployment.apps/nginx-deployment image updated
    ```

- 更新镜像创建， 开始创建 ReplicaSet `nginx-deployment-1989198191`， 但因为上面提到的 `maxUnavailable`， 所以会阻塞，通过以下命令查看发布状态：
    ```shell
    kubectl get rs
    ```
    输出结果类似如下:
    ```
    NAME                          DESIRED   CURRENT   READY     AGE
    nginx-deployment-1989198191   5         5         0         9s
    nginx-deployment-618515232    8         8         8         1m
    ```

- 这时候 Deployment 又来了一个扩容请求。 自动伸缩管理器将 Deployment 的副本数加到 15.
Deployment 控制器需要决定这增加的5个副本应该加到哪里。 根据 proportional scaling ， 会将新增的副本分散到所有的 ReplicaSet。
副本占比多的 ReplicaSet 新增的副本也多，副本占比少的新增副本就少。 余数再分给副本最多的 ReplicaSet。 副本数为 0 ReplicaSet 的则不会扩容。

在上面的例子中， 3 个副本被添加到旧的 ReplicaSet 中， 两个副本被添加到新的 ReplicaSet。 发布的继续推进最终会将所有的副本都移到新的 ReplicaSet
保证所以新的副本都是正常的。 执行以下命令验证:

```shell
kubectl get deploy
```

输出结果类似如下:
```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```
发布状态也确认了副本是怎么加到每个 ReplicaSet 上的
```shell
kubectl get rs
```

输出结果类似如下:
```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```
<!--
## Pausing and Resuming a Deployment

You can pause a Deployment before triggering one or more updates and then resume it. This allows you to
apply multiple fixes in between pausing and resuming without triggering unnecessary rollouts.

* For example, with a Deployment that was just created:
  Get the Deployment details:
  ```shell
  kubectl get deploy
  ```
  The output is similar to this:
  ```
  NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx     3         3         3            3           1m
  ```
  Get the rollout status:
  ```shell
  kubectl get rs
  ```
  The output is similar to this:
  ```
  NAME               DESIRED   CURRENT   READY     AGE
  nginx-2142116321   3         3         3         1m
  ```

* Pause by running the following command:
    ```shell
    kubectl rollout pause deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment paused
    ```

* Then update the image of the Deployment:
    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment image updated
    ```

* Notice that no new rollout started:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
* Get the rollout status to ensure that the Deployment is updates successfully:
    ```shell
    kubectl get rs
    ```

    The output is similar to this:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```

* You can make as many updates as you wish, for example, update the resources that will be used:
    ```shell
    kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    The initial state of the Deployment prior to pausing it will continue its function, but new updates to
    the Deployment will not have any effect as long as the Deployment is paused.

* Eventually, resume the Deployment and observe a new ReplicaSet coming up with all the new updates:
    ```shell
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    The output is similar to this:
    ```
    deployment.apps/nginx-deployment resumed
    ```
* Watch the status of the rollout until it's done.
    ```shell
    kubectl get rs -w
    ```

    The output is similar to this:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```
* Get the status of the latest rollout:
    ```shell
    kubectl get rs
    ```

    The output is similar to this:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```
{{< note >}}
You cannot rollback a paused Deployment until you resume it.
{{< /note >}}
 -->
## Deployment 的暂停与恢复


用户可以一个或多个更新触发之前暂停 Deployment 然后再恢复它。 通过这种方式用户可以在暂停后进行多次修改然后恢复，这样在多次
修改的过程中就不会触发不必要的发布。

* 例如以下为一个刚创建的 Deployment:
  查看 Deployment 信息:
  ```shell
  kubectl get deploy
  ```
  输出结果类似如下:
  ```
  NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  nginx     3         3         3            3           1m
  ```
  查看发布状态:
  ```shell
  kubectl get rs
  ```
  输出结果类似如下:
  ```
  NAME               DESIRED   CURRENT   READY     AGE
  nginx-2142116321   3         3         3         1m
  ```

* 执行以下命令暂停 Deployment:
    ```shell
    kubectl rollout pause deployment.v1.apps/nginx-deployment
    ```

    输出结果类似如下:
    ```
    deployment.apps/nginx-deployment paused
    ```

* 这时先修改 Deployment 的镜像:
    ```shell
    kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
    ```

    输出结果类似如下:
    ```
    deployment.apps/nginx-deployment image updated
    ```

* 可以看到现在没有触发新的发布:
    ```shell
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出结果类似如下:
    ```
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```
- 查看发布状态，确保 Deployment 的更新的成功的：
    ```shell
    kubectl get rs
    ```

    输出结果类似如下:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```

- 这时候用户就可以进行任意多次修改，例如，以下为修改资源占用:
    ```shell
    kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
    ```

    输出结果类似如下:
    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    Deployment 会继续以暂停之前的状态提供功能， 但在恢复的在的所有修改都不会生效
- 最后，恢复 Deployment 观察新建的 ReplicaSet 会包含暂停过程中的所以的更新
    ```shell
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    输出结果类似如下:
    ```
    deployment.apps/nginx-deployment resumed
    ```
* Watch the status of the rollout until it's done.
- 观察发布状态直至任务完成
    ```shell
    kubectl get rs -w
    ```

    输出结果类似如下:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```
* 查看最终的发布状态:
    ```shell
    kubectl get rs
    ```

    输出结果类似如下:
    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```
{{< note >}}
Deployment 在暂停后直至恢复，中间不能回滚
{{< /note >}}
<!--
## Deployment status

A Deployment enters various states during its lifecycle. It can be [progressing](#progressing-deployment) while
rolling out a new ReplicaSet, it can be [complete](#complete-deployment), or it can [fail to progress](#failed-deployment).
 -->

## Deployment 的状态 {#deployment-status}

一个 Deployment 在整个生命周期中会进入许多种状态。更新到一个新的 ReplicaSet 时， 状态可以是 [进行中](#progressing-deployment)
也可能是 [完成](#complete-deployment)，还可以是 [失败](#failed-deployment)
<!--
### Progressing Deployment

Kubernetes marks a Deployment as _progressing_ when one of the following tasks is performed:

* The Deployment creates a new ReplicaSet.
* The Deployment is scaling up its newest ReplicaSet.
* The Deployment is scaling down its older ReplicaSet(s).
* New Pods become ready or available (ready for at least [MinReadySeconds](#min-ready-seconds)).
 -->

### Deployment 进行中 {#progressing-deployment}

当 Deployment 在进行以下任一任务时，标记为 _进行中_

- Deployment 创建一个新的 ReplicaSet
- Deployment 在对最新的 ReplicaSet 扩容
- Deployment 在对旧的 ReplicaSet 缩减容量
- 新创建的 Pod 正在进入就绪或可用状态(就绪最小时间[MinReadySeconds](#min-ready-seconds))
### Complete Deployment

Kubernetes marks a Deployment as _complete_ when it has the following characteristics:

* All of the replicas associated with the Deployment have been updated to the latest version you've specified, meaning any
updates you've requested have been completed.
* All of the replicas associated with the Deployment are available.
* No old replicas for the Deployment are running.

You can check if a Deployment has completed by using `kubectl rollout status`. If the rollout completed
successfully, `kubectl rollout status` returns a zero exit code.

```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```
The output is similar to this:
```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment.apps/nginx-deployment successfully rolled out
```
and the exit status from `kubectl rollout` is 0 (success):
```shell
echo $?
```
```
0
```

### Deployment 完成

当 Deployment 拥有以下特征时被 k8s 标记为 完成
- 所有与 Deployment 关联的副本都已经使用了最新的配置，也就是说所以的更新请求都已经完成。
- 所有与 Deployment 关联的副本都已经可用
- Deployment 没有旧的副本在运行

用户可以通过命令 `kubectl rollout status` 检测 Deployment 是否完成。 如果发布已经完成，
`kubectl rollout status` 命令的退出码为 0

```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```
输出结果类似如下:
```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment.apps/nginx-deployment successfully rolled out
```
`kubectl rollout` 的退出码为 0 (成功):
```shell
echo $?
```
```
0
```
<!--
### Failed Deployment

Your Deployment may get stuck trying to deploy its newest ReplicaSet without ever completing. This can occur
due to some of the following factors:

* Insufficient quota
* Readiness probe failures
* Image pull errors
* Insufficient permissions
* Limit ranges
* Application runtime misconfiguration

One way you can detect this condition is to specify a deadline parameter in your Deployment spec:
([`.spec.progressDeadlineSeconds`](#progress-deadline-seconds)). `.spec.progressDeadlineSeconds` denotes the
number of seconds the Deployment controller waits before indicating (in the Deployment status) that the
Deployment progress has stalled.

The following `kubectl` command sets the spec with `progressDeadlineSeconds` to make the controller report
lack of progress for a Deployment after 10 minutes:

```shell
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```
The output is similar to this:
```
deployment.apps/nginx-deployment patched
```
Once the deadline has been exceeded, the Deployment controller adds a DeploymentCondition with the following
attributes to the Deployment's `.status.conditions`:

* Type=Progressing
* Status=False
* Reason=ProgressDeadlineExceeded

See the [Kubernetes API conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties) for more information on status conditions.

{{< note >}}
Kubernetes takes no action on a stalled Deployment other than to report a status condition with
`Reason=ProgressDeadlineExceeded`. Higher level orchestrators can take advantage of it and act accordingly, for
example, rollback the Deployment to its previous version.
{{< /note >}}

{{< note >}}
If you pause a Deployment, Kubernetes does not check progress against your specified deadline.
You can safely pause a Deployment in the middle of a rollout and resume without triggering
the condition for exceeding the deadline.
{{< /note >}}

You may experience transient errors with your Deployments, either due to a low timeout that you have set or
due to any other kind of error that can be treated as transient. For example, let's suppose you have
insufficient quota. If you describe the Deployment you will notice the following section:

```shell
kubectl describe deployment nginx-deployment
```
The output is similar to this:
```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

If you run `kubectl get deployment nginx-deployment -o yaml`, the Deployment status is similar to this:

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

Eventually, once the Deployment progress deadline is exceeded, Kubernetes updates the status and the
reason for the Progressing condition:

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

You can address an issue of insufficient quota by scaling down your Deployment, by scaling down other
controllers you may be running, or by increasing quota in your namespace. If you satisfy the quota
conditions and the Deployment controller then completes the Deployment rollout, you'll see the
Deployment's status update with a successful condition (`Status=True` and `Reason=NewReplicaSetAvailable`).

```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

`Type=Available` with `Status=True` means that your Deployment has minimum availability. Minimum availability is dictated
by the parameters specified in the deployment strategy. `Type=Progressing` with `Status=True` means that your Deployment
is either in the middle of a rollout and it is progressing or that it has successfully completed its progress and the minimum
required new replicas are available (see the Reason of the condition for the particulars - in our case
`Reason=NewReplicaSetAvailable` means that the Deployment is complete).

You can check if a Deployment has failed to progress by using `kubectl rollout status`. `kubectl rollout status`
returns a non-zero exit code if the Deployment has exceeded the progression deadline.

```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```
The output is similar to this:
```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
```
and the exit status from `kubectl rollout` is 1 (indicating an error):
```shell
echo $?
```
```
1
```
 -->
### Deployment 失败

Deployment 可能在尝试部署最新的 ReplicaSet 的时候被卡住然后再也完不成。造成这种情况的因素可能有:

- 配额不足
- 就绪探针检测结果为失败
- 镜像拉取出错
- 权限不足
- 应用运行环境配置错误

检测到这种情况的一种方法为在 Deployment 的配置中加入一个死线参数: ([`.spec.progressDeadlineSeconds`](#progress-deadline-seconds)).
`.spec.progressDeadlineSeconds` 表示 Deployment 控制器在(Deployment 状态上)显示 Deployment 停止之前等待的时间(单位秒)
以下 `kubectl` 添加/修改 `progressDeadlineSeconds`， 让控制器在 10 分钟还在进行中的发布标记为失败
```shell
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
```
输出结果类似如下:
```
deployment.apps/nginx-deployment patched
```
当超过设定时限后， Deployment 控制器会向 Deployment 添加一个拥有如下属性的 DeploymentCondition 到 `.status.conditions` 。
- Type=Progressing
- Status=False
- Reason=ProgressDeadlineExceeded

更新关于状态条件的信息见 [Kubernetes API conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties)
{{< note >}}
对于停止的 Deployment k8s 除了添加一个 `Reason=ProgressDeadlineExceeded` 条件之前不会做任何其它操作。
层级更高的编排者会根据这个条件进行相应的操作，例如，将 Deployment 回滚到前一个稳定版本
{{< /note >}}

{{< note >}}
如果 Deployment 被暂停，则 k8s 不会受该时限影响。 用户可以发布乾中安全的暂停并恢复 Deployment 而不需要担心触发这个
时限条件。
{{< /note >}}

用户可能会遇到 Deployment 短时的错误，可能是因为用户设置一个较低的超时时间或其它任意可以被认作瞬时错误的。
例如， 假设配额不足。 这时时候获取 Deployment 描述信息：

```shell
kubectl describe deployment nginx-deployment
```
输出结果类似如下:
```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

执行命令 `kubectl get deployment nginx-deployment -o yaml`, Deployment 状态类似如下:

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

最终， 当超出 Deployment 的处理时限时， k8s 更新状态和添加处理条件的原因
```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

在遇到配额不足的情况，用户可以通过缩减 Deployment 副本数来解决， 也可以缩减用户其它控制器的副本数，还可以增加用户命名空间的配额。
如果达到配额条件， Deployment 这时候就会过成 Deployment 的发布， 这时候就会看到 Deployment 状态被更新为一个成功条件
(`Status=True` 和 `Reason=NewReplicaSetAvailable`)
```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

当 `Type=Available` 的 `Status=True` 时意味着 Deployment 达到了最小可用状。 最小可用状态受 deployment 策略中的参数控制。
`Type=Progressing` 的 `Status=True` 意味着 Deployment 的发布还在进行或已经处理成功完成，最小需求的副本数已经可用(见具体条件的原因 - 就本例来说
  `Reason=NewReplicaSetAvailable` 表示 Deployment 已经完成)

用户可以通过执行命令 `kubectl rollout status` 来检测 Deployment 是处理失败。如果 Deployment 已经超出处理时限
`kubectl rollout status` 返回码非 0


```shell
kubectl rollout status deployment.v1.apps/nginx-deployment
```
输出结果类似如下:
```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
```
`kubectl rollout` 返回码为 1 (表示出错):
```shell
echo $?
```
```
1
```
<!--
### Operating on a failed deployment

All actions that apply to a complete Deployment also apply to a failed Deployment. You can scale it up/down, roll back
to a previous revision, or even pause it if you need to apply multiple tweaks in the Deployment Pod template.
 -->
### 在失败状态的 Deployment 上可以执行的操作

所有在完成的 Deployment 上可以执行的操作都可以在失败的 Deployment 执行. 用户可以扩充/缩减容量, 回滚到之前的一个版本, 甚至如果用户需要
多次修改 Deployment 的 Pod 模板可以将其暂停.
<!--
## Clean up Policy

You can set `.spec.revisionHistoryLimit` field in a Deployment to specify how many old ReplicaSets for
this Deployment you want to retain. The rest will be garbage-collected in the background. By default,
it is 10.

{{< note >}}
Explicitly setting this field to 0, will result in cleaning up all the history of your Deployment
thus that Deployment will not be able to roll back.
{{< /note >}}
 -->
## 清理策略

用户可以通过设置 Deployment 中的 `.spec.revisionHistoryLimit` 字段, 指定保留多少份旧的 ReplicaSets 的记录.
超出的部分会被后台垃圾回收. 默认是 10

{{< note >}}
如果将该字段设置为 0, 则会导致所以的历史都会被删除, Deployment 不能被回滚.
{{< /note >}}
<!--
## Canary Deployment

If you want to roll out releases to a subset of users or servers using the Deployment, you
can create multiple Deployments, one for each release, following the canary pattern described in
[managing resources](/docs/concepts/cluster-administration/manage-deployment/#canary-deployments).
 -->
## 金丝雀 Deployment

如查用户想让一部分用户或服务使用这个 Deployment 发布的版本, 可以创建多个 Deployment, 每一个发布一个版本,
依据[资源管理](/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)中描述的金丝雀规则。

<!--
## Writing a Deployment Spec

As with all other Kubernetes configs, a Deployment needs `.apiVersion`, `.kind`, and `.metadata` fields.
For general information about working with config files, see
[deploying applications](/docs/tasks/run-application/run-stateless-application-deployment/),
configuring containers, and [using kubectl to manage resources](/docs/concepts/overview/working-with-objects/object-management/) documents.
The name of a Deployment object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A Deployment also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
 -->
## 编写 Deployment Spec

As with all other Kubernetes configs, a Deployment needs `.apiVersion`, `.kind`, and `.metadata` fields.
For general information about working with config files, see
[deploying applications](/docs/tasks/run-application/run-stateless-application-deployment/),
configuring containers, and [using kubectl to manage resources](/docs/concepts/overview/working-with-objects/object-management/) documents.
The name of a Deployment object must be a valid
[DNS subdomain name](/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

A Deployment also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
与所以其它的 k8s 配置一样，Deployment 必须要有  `.apiVersion`, `.kind`, `.metadata` 字段。
配置文件的通用信息见 [部署应用](/k8sDocs/tasks/run-application/run-stateless-application-deployment/),
配置容器， [使用 kubect 管理资源](/k8sDocs/concepts/overview/working-with-objects/object-management/).
Deployment 对象的名称必须是一个有效的 [DNS 子域名](/k8sDocs/concepts/overview/working-with-objects/names#dns-subdomain-names)

Deployment 还需要有一个 [`.spec` 定义区](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).
<!--
### Pod Template

The `.spec.template` and `.spec.selector` are the only required field of the `.spec`.

The `.spec.template` is a [Pod template](/docs/concepts/workloads/pods/#pod-templates). It has exactly the same schema as a {{< glossary_tooltip text="Pod" term_id="pod" >}}, except it is nested and does not have an `apiVersion` or `kind`.

In addition to required fields for a Pod, a Pod template in a Deployment must specify appropriate
labels and an appropriate restart policy. For labels, make sure not to overlap with other controllers. See [selector](#selector)).

Only a [`.spec.template.spec.restartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always` is
allowed, which is the default if not specified.
 -->
### Pod 模板

`.spec` 的必要字段仅有 `.spec.template` 和 `.spec.selector`两个。

`.spec.template` 字段就是一个 [Pod 模板](/k8sDocs/concepts/workloads/pods/#pod-templates)对象。定义规范与 {{< glossary_tooltip text="Pod" term_id="pod" >}}
完全一样， 除了因为它嵌套的 Deployment 中所以没 `apiVersion` 或 `kind` 字段。

相对裸 Pod 所以需要的字段， Deployment 中的 Pod 模板需要指定适当的标签和重启策略。 对于标签定义，需要注意不要与其它的控制器重叠了。 见 [selector](#selector)

[`.spec.template.spec.restartPolicy`](/k8sDocs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 的值只能是 `Always`， 这也是默认值。
<!--
### Replicas

`.spec.replicas` is an optional field that specifies the number of desired Pods. It defaults to 1.
 -->
### 副本数

`.spec.replicas` 是一个可选字段，表示期望的 Pod 的数量。 默认为 1
<!--
### Selector

`.spec.selector` is a required field that specifies a [label selector](/docs/concepts/overview/working-with-objects/labels/)
for the Pods targeted by this Deployment.

`.spec.selector` must match `.spec.template.metadata.labels`, or it will be rejected by the API.

In API version `apps/v1`, `.spec.selector` and `.metadata.labels` do not default to `.spec.template.metadata.labels` if not set. So they must be set explicitly. Also note that `.spec.selector` is immutable after creation of the Deployment in `apps/v1`.

A Deployment may terminate Pods whose labels match the selector if their template is different
from `.spec.template` or if the total number of such Pods exceeds `.spec.replicas`. It brings up new
Pods with `.spec.template` if the number of Pods is less than the desired number.

{{< note >}}
You should not create other Pods whose labels match this selector, either directly, by creating
another Deployment, or by creating another controller such as a ReplicaSet or a ReplicationController. If you
do so, the first Deployment thinks that it created these other Pods. Kubernetes does not stop you from doing this.
{{< /note >}}

If you have multiple controllers that have overlapping selectors, the controllers will fight with each
other and won't behave correctly.
 -->
### 选择器

`.spec.selector` 是一个必要字段，用于定义 [标签选择器](/k8sDocs/concepts/overview/working-with-objects/labels/), 标签选择器用于选择 Deployment 下属的 Pod

`.spec.selector` 必须与 `.spec.template.metadata.labels`配置, 否则 API 会拒绝.

API 版本 `apps/v1` 中， `.spec.selector` 和 `.metadata.labels` 如果没有设置不会默认设置为 `.spec.template.metadata.labels`。 所以这两个字段必须要显示的设置。

Deployment 终结标签匹配选择器，但模板与  `.spec.template` 不同的 Pod。
如果标签选择器匹配的 Pod 数量大于 `.spec.replicas` 的数量，则会终结多于的 Pod。
如果标签选择器匹配的 Pod 数量小于 `.spec.replicas` 的数量， 会使用`.spec.template`创建新的 Pod， 使总数与期望数相同。

{{< note >}}
用户需要注意不能创建标签匹配该选择的的 Pod， 不管理是直接创建还是通过 Deployment 创建，或者通过其它的如 ReplicaSet 或 ReplicationController
控制器来创建。 如果这样做了，第一个 Deployment 这些 Pod 也是它创建的。k8s 不会阻止用户这样做
{{< /note >}}

如果有多个控制器使用了一样的选择器，这些选择器就会打架，然后不能正常工作。
<!--
### Strategy

`.spec.strategy` specifies the strategy used to replace old Pods by new ones.
`.spec.strategy.type` can be "Recreate" or "RollingUpdate". "RollingUpdate" is
the default value.
 -->
### 策略

`.spec.strategy` 指定新 Pod 代替旧 Pod 使用的策略
`.spec.strategy.type` 的值可以是 `Recreate` 或 `RollingUpdate`。 默认为 `RollingUpdate`
<!--
#### Recreate Deployment

All existing Pods are killed before new ones are created when `.spec.strategy.type==Recreate`.

{{< note >}}
This will only guarantee Pod termination previous to creation for upgrades. If you upgrade a Deployment, all Pods
of the old revision will be terminated immediately. Successful removal is awaited before any Pod of the new
revision is created. If you manually delete a Pod, the lifecycle is controlled by the ReplicaSet and the
replacement will be created immediately (even if the old Pod is still in a Terminating state). If you need an
"at most" guarantee for your Pods, you should consider using a
[StatefulSet](/docs/concepts/workloads/controllers/statefulset/).
{{< /note >}}
 -->
#### Deployment 重建策略

当设置 `.spec.strategy.type==Recreate` 时，会先将所以旧的 Pod 删除再创建新的

{{< note >}}
这个策略只能保证终止旧 Pod 的操作在创建新 Pod 的操作之前。 如果用户更新的一个 Deployment， 所以旧的 Pod 就会马上被标记为终止产状。
成功删除则要等到有任意一个新版本的 Pod 创建成功之后。 如果在这之前，用户手动删除了一个 Pod， 这时它的生命周期还受 ReplicaSet 控制，
所以会立马又创建一个旧的(尽管旧的 Pod 依然处于终止中的状态)。 如果用户需要 “最大的” 保证， 则建议考虑使用
[StatefulSet](/k8sDocs/concepts/workloads/controllers/statefulset/)
{{< /note >}}
<!--
#### Rolling Update Deployment

The Deployment updates Pods in a rolling update
fashion when `.spec.strategy.type==RollingUpdate`. You can specify `maxUnavailable` and `maxSurge` to control
the rolling update process.
 -->
#### Deployment 滚动更新策略

当 `.spec.strategy.type==RollingUpdate` 时 Deployment 以滚动方式更新 Pod。
用户可以通过 `maxUnavailable` 和 `maxSurge` 来控制更新进程
<!--
##### Max Unavailable

`.spec.strategy.rollingUpdate.maxUnavailable` is an optional field that specifies the maximum number
of Pods that can be unavailable during the update process. The value can be an absolute number (for example, 5)
or a percentage of desired Pods (for example, 10%). The absolute number is calculated from percentage by
rounding down. The value cannot be 0 if `.spec.strategy.rollingUpdate.maxSurge` is 0. The default value is 25%.

For example, when this value is set to 30%, the old ReplicaSet can be scaled down to 70% of desired
Pods immediately when the rolling update starts. Once new Pods are ready, old ReplicaSet can be scaled
down further, followed by scaling up the new ReplicaSet, ensuring that the total number of Pods available
at all times during the update is at least 70% of the desired Pods.
 -->
##### 最大不可用数

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段， 用于设置在更新过程最大不可用 Pod 的数量。 字段值可以是一个数字(如， 5)
也可以是期望副本数的百分比(如，10%)。 数字为百分比向下取整。 如果 `.spec.strategy.rollingUpdate.maxSurge` 的值是 0 则该字段值不能是 0.
默认值为 `25%`

例如， 当这个值设置为 30%, 旧的 ReplicaSet 在滚动更新开始时就能缩减容量是期望数量的 70%， 旧的 ReplicaSet 会在 新的 ReplicaSet 扩容后
相应的收缩自己的容量，以确保在整个更新过程中可用的 Pod 数不低于期望数量的 70%。
<!--
##### Max Surge

`.spec.strategy.rollingUpdate.maxSurge` is an optional field that specifies the maximum number of Pods
that can be created over the desired number of Pods. The value can be an absolute number (for example, 5) or a
percentage of desired Pods (for example, 10%). The value cannot be 0 if `MaxUnavailable` is 0. The absolute number
is calculated from the percentage by rounding up. The default value is 25%.

For example, when this value is set to 30%, the new ReplicaSet can be scaled up immediately when the
rolling update starts, such that the total number of old and new Pods does not exceed 130% of desired
Pods. Once old Pods have been killed, the new ReplicaSet can be scaled up further, ensuring that the
total number of Pods running at any time during the update is at most 130% of desired Pods.
 -->
##### 最大可超预期数

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段， 用于设置在更新过程最大可超过期望数的数量。 这个值可以是一个数量(如， 5)
也可以是期望副本数的百分比(如，10%)。如果 `MaxUnavailable` 值为 0， 则该字段值不能是 0. 数字为百分比向上取整。默认值为 25%。

例如， 当这个值设置为 30%, 新的 ReplicaSet 会在滚动更新开始后马上扩容。 只要新旧 Pod 总数不超过期望数量的 130%。
当旧的 Pod 被干掉， 新的 ReplicaSet 就能继续扩容， 只要始终确保总的 Pod 不超过期望数量的 130%。
<!--
### Progress Deadline Seconds

`.spec.progressDeadlineSeconds` is an optional field that specifies the number of seconds you want
to wait for your Deployment to progress before the system reports back that the Deployment has
[failed progressing](#failed-deployment) - surfaced as a condition with `Type=Progressing`, `Status=False`.
and `Reason=ProgressDeadlineExceeded` in the status of the resource. The Deployment controller will keep
retrying the Deployment. This defaults to 600. In the future, once automatic rollback will be implemented, the Deployment
controller will roll back a Deployment as soon as it observes such a condition.

If specified, this field needs to be greater than `.spec.minReadySeconds`.
 -->
### 更新处理时限

`.spec.progressDeadlineSeconds` 是一个可选字段， 用于设置 Deployment 进行更新处理的时限(单位秒)， 如果超过这个时限
这次更新还没有完成则系统就会报告 Deployment [处理失败](#failed-deployment)，报告为向状态资源添加一个包含以下字段的条件
`Type=Progressing`, `Status=False`, `Reason=ProgressDeadlineExceeded`. Deployment 控制器会持续重试这个 Deployment.
这个字段默认值为 600. 在将来，当自动回滚实现以后，当发现这个超时条件就会自动回滚这个 Deployment。

如果为设置该字段，该字段的值需要大于 `.spec.minReadySeconds`
<!--
### Min Ready Seconds

`.spec.minReadySeconds` is an optional field that specifies the minimum number of seconds for which a newly
created Pod should be ready without any of its containers crashing, for it to be considered available.
This defaults to 0 (the Pod will be considered available as soon as it is ready). To learn more about when
a Pod is considered ready, see [Container Probes](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes).
 -->
### 最小就绪时间

`.spec.minReadySeconds` 是一个可选字段， 用于设置一个新创建的 Pod 在没有任何容器崩掉的情况下达成就绪到被认为可用的最小时限(单位秒)
默认是 0 (Pod 在就绪后就认为可用)。 了解更多关于 Pod 被认为就绪的信息，见[容器探针](/k8sDocs/concepts/workloads/pods/pod-lifecycle/#container-probes).
<!--
### Revision History Limit

A Deployment's revision history is stored in the ReplicaSets it controls.

`.spec.revisionHistoryLimit` is an optional field that specifies the number of old ReplicaSets to retain
to allow rollback. These old ReplicaSets consume resources in `etcd` and crowd the output of `kubectl get rs`. The configuration of each Deployment revision is stored in its ReplicaSets; therefore, once an old ReplicaSet is deleted, you lose the ability to rollback to that revision of Deployment. By default, 10 old ReplicaSets will be kept, however its ideal value depends on the frequency and stability of new Deployments.

More specifically, setting this field to zero means that all old ReplicaSets with 0 replicas will be cleaned up.
In this case, a new Deployment rollout cannot be undone, since its revision history is cleaned up.
 -->
### 历史版本记录限制

Deployment 的版本历史存在受它控制的 ReplicaSet 中。

`.spec.revisionHistoryLimit` 是一个可选字段。 用户设置允许回滚而保留的旧的 ReplicaSet 的数量。
这个旧的 ReplicaSet 会被存储在 `etcd` 中， 可以通过命令 `kubectl get rs` 查看。 Deployment 每个版本的配置被保存在它的 ReplicaSet 中，
因此，当一个旧的 ReplicaSet 被删除，就会的会回滚到该版本的能力。 默认会保留 10 个旧的 ReplicaSet， 但是合理的值基于新发 Deployment 的频次和稳定性。

更准确的说， 如果把该字段值设置为0， 意味着所以副本数为 0 的旧 ReplicaSet 都会被删除， 在这种情况下， 新的 Deployment 就没办法回滚了，因为历史版本都清空了。
<!--
### Paused

`.spec.paused` is an optional boolean field for pausing and resuming a Deployment. The only difference between
a paused Deployment and one that is not paused, is that any changes into the PodTemplateSpec of the paused
Deployment will not trigger new rollouts as long as it is paused. A Deployment is not paused by default when
it is created.
 -->

### 暂停
`.spec.paused` 是一个可选布尔字段，用于暂停和恢复 Deployment。 Deployment 有没有暂停的区别是对 PodTemplateSpec 变更， 暂停的 Deployment
在被恢复之前不会触发更新。 Deployment 创建时默认没有暂停。
