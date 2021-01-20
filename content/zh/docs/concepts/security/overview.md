---
title: 云原生安全概述
content_type: concept
weight: 10
---

<!--
---
reviewers:
- zparnold
title: Overview of Cloud Native Security
content_type: concept
weight: 10
---
 -->
<!-- overview -->
<!--
This overview defines a model for thinking about Kubernetes security in the context of Cloud Native security.
 -->
本文定义了一个模式用于思考在云原生安全语境下的 k8s 安全

{{< warning >}}
<!--
This container security model provides suggestions, not proven information security policies.
 -->
这个容器安全模型只提供建议，并不提供安全策略信息。
{{< /warning >}}

<!-- body -->
<!--
## The 4C's of Cloud Native security

You can think about security in layers. The 4C's of Cloud Native security are Cloud,
Clusters, Containers, and Code.

{{< note >}}
This layered approach augments the [defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing))
computing approach to security, which is widely regarded as a best practice for securing
software systems.
{{< /note >}}

{{< figure src="/images/docs/4c.png" title="The 4C's of Cloud Native Security" >}}

Each layer of the Cloud Native security model builds upon the next outermost layer.
The Code layer benefits from strong base (Cloud, Cluster, Container) security layers.
You cannot safeguard against poor security standards in the base layers by addressing
security at the Code level.
 -->

## 云原生安全的 4C 概念 {#the-4cs-of-cloud-native-security}

可以将安全进行分层。 云原生安全的 4 个 C 就是云(Cloud), 集群(Clusters), 容器(Containers),
和 代码(Code).

{{< note >}}
这种计算机安全通过分层的方式实现在
[纵深防御](https://en.wikipedia.org/wiki/Defense_in_depth_(computing))
中有讨论，也是广泛认可作为软件系统安全的最佳实践。
{{< /note >}}

{{< figure src="/k8sDocs/images/docs/4c.png" title="云原生安全的 4C 概念" >}}

云原生安全模型的每一层都是构建于其外层的。 代码(Code)层受益于 (Cloud, Cluster, Container)
层的安全基础。 不能希望在基础层安全薄弱的情况下只通过代码层的安全策略就能提供良好的安全保障。

<!--
## Cloud

In many ways, the Cloud (or co-located servers, or the corporate datacenter) is the
[trusted computing base](https://en.wikipedia.org/wiki/Trusted_computing_base)
of a Kubernetes cluster. If the Cloud layer is vulnerable (or
configured in a vulnerable way) then there is no guarantee that the components built
on top of this base are secure. Each cloud provider makes security recommendations
for running workloads securely in their environment.
 -->

## 云环境 {#cloud}

在大多数情况下， 云环境(或同一个地方的服务器，或公司的数据中心) 在 k8s 集群中是
[受信任的计算基础](https://en.wikipedia.org/wiki/Trusted_computing_base)。 如果在云
环境层是不安全的(或者以一个不安全的方式配置的)，那么基于这个基础构建的组建也是没有安全保证的。
每个云提供商都对在其环境上安全运行工作负载的安全建议

<!--
### Cloud provider security

If you are running a Kubernetes cluster on your own hardware or a different cloud provider,
consult your documentation for security best practices.
Here are links to some of the popular cloud providers' security documentation:

{{< table caption="Cloud provider security" >}}

IaaS Provider        | Link |
-------------------- | ------------ |
Alibaba Cloud | https://www.alibabacloud.com/trust-center |
Amazon Web Services | https://aws.amazon.com/security/ |
Google Cloud Platform | https://cloud.google.com/security/ |
IBM Cloud | https://www.ibm.com/cloud/security |
Microsoft Azure | https://docs.microsoft.com/en-us/azure/security/azure-security |
VMWare VSphere | https://www.vmware.com/security/hardening-guides.html |

{{< /table >}}
 -->

### 云提供商安全 {#Cloud-provider-security}

如果 k8s 集群运行在自己的硬件上或不同的云提供商，请查询相应文档获取最佳安全实践。
下面是一些主流云提供商的安全文档的链接地址.

{{< table caption="云提供商安全" >}}

IaaS 提供商           | 链接 |
-------------------- | ------------ |
Alibaba Cloud | https://www.alibabacloud.com/trust-center |
Amazon Web Services | https://aws.amazon.com/security/ |
Google Cloud Platform | https://cloud.google.com/security/ |
IBM Cloud | https://www.ibm.com/cloud/security |
Microsoft Azure | https://docs.microsoft.com/en-us/azure/security/azure-security |
VMWare VSphere | https://www.vmware.com/security/hardening-guides.html |

{{< /table >}}

<!--
### Infrastructure security {#infrastructure-security}

Suggestions for securing your infrastructure in a Kubernetes cluster:

{{< table caption="Infrastructure security" >}}

Area of Concern for Kubernetes Infrastructure | Recommendation |
--------------------------------------------- | -------------- |
Network access to API Server (Control plane) | All access to the Kubernetes control plane is not allowed publicly on the internet and is controlled by network access control lists restricted to the set of IP addresses needed to administer the cluster.|
Network access to Nodes (nodes) | Nodes should be configured to _only_ accept connections (via network access control lists)from the control plane on the specified ports, and accept connections for services in Kubernetes of type NodePort and LoadBalancer. If possible, these nodes should not be exposed on the public internet entirely.
Kubernetes access to Cloud Provider API | Each cloud provider needs to grant a different set of permissions to the Kubernetes control plane and nodes. It is best to provide the cluster with cloud provider access that follows the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) for the resources it needs to administer. The [Kops documentation](https://github.com/kubernetes/kops/blob/master/docs/iam_roles.md#iam-roles) provides information about IAM policies and roles.
Access to etcd | Access to etcd (the datastore of Kubernetes) should be limited to the control plane only. Depending on your configuration, you should attempt to use etcd over TLS. More information can be found in the [etcd documentation](https://github.com/etcd-io/etcd/tree/master/Documentation).
etcd Encryption | Wherever possible it's a good practice to encrypt all drives at rest, but since etcd holds the state of the entire cluster (including Secrets) its disk should especially be encrypted at rest.

{{< /table >}}
 -->

<!--
### Infrastructure security {#infrastructure-security}

Suggestions for securing your infrastructure in a Kubernetes cluster:

{{< table caption="Infrastructure security" >}}

Area of Concern for Kubernetes Infrastructure | Recommendation |
--------------------------------------------- | -------------- |
Network access to API Server (Control plane) | All access to the Kubernetes control plane is not allowed publicly on the internet and is controlled by network access control lists restricted to the set of IP addresses needed to administer the cluster.|
Network access to Nodes (nodes) | Nodes should be configured to _only_ accept connections (via network access control lists)from the control plane on the specified ports, and accept connections for services in Kubernetes of type NodePort and LoadBalancer. If possible, these nodes should not be exposed on the public internet entirely.
Kubernetes access to Cloud Provider API | Each cloud provider needs to grant a different set of permissions to the Kubernetes control plane and nodes. It is best to provide the cluster with cloud provider access that follows the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) for the resources it needs to administer. The [Kops documentation](https://github.com/kubernetes/kops/blob/master/docs/iam_roles.md#iam-roles) provides information about IAM policies and roles.
Access to etcd | Access to etcd (the datastore of Kubernetes) should be limited to the control plane only. Depending on your configuration, you should attempt to use etcd over TLS. More information can be found in the [etcd documentation](https://github.com/etcd-io/etcd/tree/master/Documentation).
etcd Encryption | Wherever possible it's a good practice to encrypt all drives at rest, but since etcd holds the state of the entire cluster (including Secrets) its disk should especially be encrypted at rest.

{{< /table >}}
 -->

### 基础设施安全 {#infrastructure-security}

k8s 集群基础设计安全加固建议:

{{< table caption="基础设施安全" >}}

k8s 基础设计关注领域 | 推荐 |
--------------------------------------------- | -------------- |
访问 API 服务 (控制中心) 的网络 | k8s 控制中心是不允许任意来自公共互联网的访问，并且对控制中心的访问应该限制在那些需要管理集群的 IP 集合|
访问节点 (nodes) 的网络 |节点应该配置成 _只_ 允许来自控制中心访问指定端口的连接(通过网络访问控制列表)，和接收来自 k8s 中 NodePort 和 LoadBalancer 类型 Service 的连接。 如果可能，避免让所有的节点都暴露到互联网。
k8s 对云提供商 API 的访问 | 每个云提供商需要给 k8s 控制中心和节点授予不同个权限集。 在授予集群云提供商访问权限时，依照[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，只给予其所需要管理的资源。 [Kops documentation](https://github.com/kubernetes/kops/blob/master/docs/iam_roles.md#iam-roles) 文档提供了关于 IAM 和角色的信息。
访问 etcd | 对 etcd (k8s 的数据源)应该限制到只给控制中心。 基于配置，应该使用配置有 TLS 的 etcd。 更多信息可以在 [etcd 文档](https://github.com/etcd-io/etcd/tree/master/Documentation)中找到.
etcd 加密 | 如果可能，对所有驱动器在落盘时加密是一种好的实践，但因为 etcd 中存放了整个集群的状态(包括 Secret)它的数据最应该在落盘时加密。

{{< /table >}}

{{< todo-optimize >}}

<!--
## Cluster

There are two areas of concern for securing Kubernetes:

* Securing the cluster components that are configurable
* Securing the applications which run in the cluster
 -->

## 集群 {#cluster}

在对 k8s 安全加固时有两个值得关注的领域：

- 安全加固那些可配置的集群组件
- 安全加固那些运行在集群中的应用

<!--
### Components of the Cluster {#cluster-components}

If you want to protect your cluster from accidental or malicious access and adopt
good information practices, read and follow the advice about
[securing your cluster](/docs/tasks/administer-cluster/securing-a-cluster/).
 -->

### 集群的组件 {#cluster-components}

如果希望保护集群免于偶然或恶意的访问和彩良好的信息实践，请阅读下面的建议
[集群安全加固](/k8sDocs/docs/tasks/administer-cluster/securing-a-cluster/).

<!--
### Components in the cluster (your application) {#cluster-applications}

Depending on the attack surface of your application, you may want to focus on specific
aspects of security. For example: If you are running a service (Service A) that is critical
in a chain of other resources and a separate workload (Service B) which is
vulnerable to a resource exhaustion attack then the risk of compromising Service A
is high if you do not limit the resources of Service B. The following table lists
areas of security concerns and recommendations for securing workloads running in Kubernetes:

Area of Concern for Workload Security | Recommendation |
------------------------------ | --------------------- |
RBAC Authorization (Access to the Kubernetes API) | https://kubernetes.io/docs/reference/access-authn-authz/rbac/
Authentication | https://kubernetes.io/docs/concepts/security/controlling-access/
Application secrets management (and encrypting them in etcd at rest) | https://kubernetes.io/docs/concepts/configuration/secret/ <br> https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
Pod Security Policies | https://kubernetes.io/docs/concepts/policy/pod-security-policy/
Quality of Service (and Cluster resource management) | https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
Network Policies | https://kubernetes.io/docs/concepts/services-networking/network-policies/
TLS For Kubernetes Ingress | https://kubernetes.io/docs/concepts/services-networking/ingress/#tls
 -->

### 集群中的组件 (你的应用) {#cluster-applications}

基于应用的攻击面，可能希望专注于某个方面的安全。 例如: 如果运行了一个服务(Service A)它是与其它资源
组成的链中的关键，另一个工作负载 (Service B), 这应用面对资源枯竭攻击是很脆弱的，如果不对 Service B
进行资源限制则会让 Service A 被攻陷的风险变高。下面表格里面列举安全关注领域和加固集群中运行的工作
负载安全的建议:

工作负载安全关注领域              | 建议 |
------------------------------ | --------------------- |
RBAC 授权 (对 k8s API 的访问) | https://kubernetes.io/docs/reference/access-authn-authz/rbac/
认证方式 | https://kubernetes.io/docs/concepts/security/controlling-access/
应用 Secret 管理 (和其在 etcd 落盘时加密) | https://kubernetes.io/docs/concepts/configuration/secret/ <br> https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
Pod 安全策略 | https://kubernetes.io/docs/concepts/policy/pod-security-policy/
服务资源 (和集群资源管理) | https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
网络策略 | https://kubernetes.io/docs/concepts/services-networking/network-policies/
为 k8s Ingress 添加 TLS | https://kubernetes.io/docs/concepts/services-networking/ingress/#tls

<!--
## Container

Container security is outside the scope of this guide. Here are general recommendations and
links to explore this topic:

Area of Concern for Containers | Recommendation |
------------------------------ | -------------- |
Container Vulnerability Scanning and OS Dependency Security | As part of an image build step, you should scan your containers for known vulnerabilities.
Image Signing and Enforcement | Sign container images to maintain a system of trust for the content of your containers.
Disallow privileged users | When constructing containers, consult your documentation for how to create users inside of the containers that have the least level of operating system privilege necessary in order to carry out the goal of the container.
 -->

## 容器 {#container}


容器安全不在本文的讨论范围。 下面是一些通用的建议和关于本话题的延伸阅读链接:

容器安全关注领域                  | 建议 |
------------------------------ | -------------- |
容器漏洞扫描 和 操作系统信赖安全  | 作为一个镜像构建的步骤，应该对容器进行已知漏洞扫描。
镜像签名和验签 | 容器镜像的签名可以维护系统对容器内存的信任
禁用特权用户 | 在构建容器时，查询文档关于怎么创建可以实现容器目标的最小操作系统权限的用户

<!--
## Code

Application code is one of the primary attack surfaces over which you have the most control.
While securing application code is outside of the Kubernetes security topic, here
are recommendations to protect application code:
 -->

## 代码 {#code}

应用代码是一个最受开发者控制的主要攻击面之一。 因为对应用代码的安全加固不在 k8s 安全话题之内，下面
是一些保护应用代码的建议:
<!--
### Code security

{{< table caption="Code security" >}}

Area of Concern for Code | Recommendation |
-------------------------| -------------- |
Access over TLS only | If your code needs to communicate by TCP, perform a TLS handshake with the client ahead of time. With the exception of a few cases, encrypt everything in transit. Going one step further, it's a good idea to encrypt network traffic between services. This can be done through a process known as mutual or [mTLS](https://en.wikipedia.org/wiki/Mutual_authentication) which performs a two sided verification of communication between two certificate holding services. |
Limiting port ranges of communication | This recommendation may be a bit self-explanatory, but wherever possible you should only expose the ports on your service that are absolutely essential for communication or metric gathering. |
3rd Party Dependency Security | It is a good practice to regularly scan your application's third party libraries for known security vulnerabilities. Each programming language has a tool for performing this check automatically. |
Static Code Analysis | Most languages provide a way for a snippet of code to be analyzed for any potentially unsafe coding practices. Whenever possible you should perform checks using automated tooling that can scan codebases for common security errors. Some of the tools can be found at: https://owasp.org/www-community/Source_Code_Analysis_Tools |
Dynamic probing attacks | There are a few automated tools that you can run against your service to try some of the well known service attacks. These include SQL injection, CSRF, and XSS. One of the most popular dynamic analysis tools is the [OWASP Zed Attack proxy](https://owasp.org/www-project-zap/) tool. |

{{< /table >}}
 -->

### 代码安全 {#code-security}

{{< table caption="Code security" >}}

代码安全关注领域           | 建议 |
-------------------------| -------------- |
只允许通过 TLS 访问 | 如果你的代码需要通过 TCP 通信，提前与客户端执行 TLS 握手。除了少数情况下，对所有传输内容加密。更进一步，对服务之间的网络流量加密也是一个好主意。这可以通过相互(mutual)过程或 [mTLS](https://en.wikipedia.org/wiki/Mutual_authentication) 也就是两个服务都包含证书，对通信进行双向加密和验证。 |
限制通信端口范围 | 这个建议应该是不言自明的， 但无论啥时候都应该只暴露服务必不可少的通信或度量采集端口 |
第三方信赖安全 | 定期扫描应用的第三方信赖库是否有已知的安全漏洞是一个好的实践。 每种编辑语言都有自动执行这种检查的工具 |
静态代码分析 | 大多数语言都提供了对代码片断分析检查是否有潜在的不安全代码实践。 无论啥时候都应该使用自动化工作对代码库执行这种检查，以提早发现常见的安全错误。 一些工具可以在这里找: https://owasp.org/www-community/Source_Code_Analysis_Tools |
动态探测攻击 | 有几种自动化工具可以用来对着服务运行，尝试一些流行的服务攻击。 包含 SQL 注入， CSRF, 和 XSS. 最流行的动态分析工具之一就是 [OWASP Zed Attack proxy](https://owasp.org/www-project-zap/)  |

{{< /table >}}




## {{% heading "whatsnext" %}}
<!--
Learn about related Kubernetes security topics:

* [Pod security standards](/docs/concepts/security/pod-security-standards/)
* [Network policies for Pods](/docs/concepts/services-networking/network-policies/)
* [Controlling Access to the Kubernetes API](/docs/concepts/security/controlling-access)
* [Securing your cluster](/docs/tasks/administer-cluster/securing-a-cluster/)
* [Data encryption in transit](/docs/tasks/tls/managing-tls-in-a-cluster/) for the control plane
* [Data encryption at rest](/docs/tasks/administer-cluster/encrypt-data/)
* [Secrets in Kubernetes](/docs/concepts/configuration/secret/)
 -->
更多关于 k8s 安全的主题:

- [Pod 安全标准](/k8sDocs/docs/concepts/security/pod-security-standards/)
- [Pod 网络策略](/k8sDocs/docs/concepts/services-networking/network-policies/)
- [对 k8s API 的访问控制](/k8sDocs/docs/concepts/security/controlling-access)
- [集群网络加固](/k8sDocs/docs/tasks/administer-cluster/securing-a-cluster/)
- 对控制中心[传输中的数据加密](/k8sDocs/docs/tasks/tls/managing-tls-in-a-cluster/)
- [数据落盘加密](/k8sDocs/docs/tasks/administer-cluster/encrypt-data/)
- [k8s 中的 Secret](/k8sDocs/docs/concepts/configuration/secret/)
