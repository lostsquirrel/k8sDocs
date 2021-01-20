---
title: Pod 安全标准
content_type: concept
weight: 20
---
<!--
---
reviewers:
- tallclair
title: Pod Security Standards
content_type: concept
weight: 10
---
 -->
<!-- overview -->
<!--
Security settings for Pods are typically applied by using [security
contexts](/docs/tasks/configure-pod-container/security-context/). Security Contexts allow for the
definition of privilege and access controls on a per-Pod basis.

The enforcement and policy-based definition of cluster requirements of security contexts has
previously been achieved using [Pod Security Policy](/docs/concepts/policy/pod-security-policy/). A
_Pod Security Policy_ is a cluster-level resource that controls security sensitive aspects of the
Pod specification.

However, numerous means of policy enforcement have arisen that augment or replace the use of
PodSecurityPolicy. The intent of this page is to detail recommended Pod security profiles, decoupled
from any specific instantiation.
 -->

对于 Pod 的安全设置通常是使用
[安全性上下文](/k8sDocs/docs/tasks/configure-pod-container/security-context/).
安全性上下文允许对 Pod 级的权限定义和访问控制。

实施和基于策略定义的集群安全性上下文要求是通过使用
[Pod 安全性策略](/docs/concepts/policy/pod-security-policy/)
实现的。 _Pod 安全性策略_ 是集群级别的资源，用来控制安全性敏感方面的 Pod 定义。

但是， 多种方式的执行策略会引发争论或替换 PodSecurityPolicy 的使用。 本文的目的是详细推荐
安全策略方案， 与任意指定实例解耦。
<!-- body -->

<!--
## Policy Types

There is an immediate need for base policy definitions to broadly cover the security spectrum. These
should range from highly restricted to highly flexible:

- **_Privileged_** - Unrestricted policy, providing the widest possible level of permissions. This
  policy allows for known privilege escalations.
- **_Baseline/Default_** - Minimally restrictive policy while preventing known privilege
  escalations. Allows the default (minimally specified) Pod configuration.
- **_Restricted_** - Heavily restricted policy, following current Pod hardening best practices.
 -->

## 策略类别 {#policy-types}

安全策略定义来大体上能覆盖安全相关问题。 下面这些从高限制到高灵活性的范围:

- **_Privileged_** - 不受限策略， 提供最大可能级别的权限。 这个策略允许所有已知权限升级。
- **_Baseline/Default_** - 在防止已知权限升级的情况下相对最小限制策略。 允许默认(最少配置)的 Pod 配置
- **_Restricted_** - 严重限制策略， 依照目前 Pod 加固最佳实践
<!--
## Policies

### Privileged

The Privileged policy is purposely-open, and entirely unrestricted. This type of policy is typically
aimed at system- and infrastructure-level workloads managed by privileged, trusted users.

The privileged policy is defined by an absence of restrictions. For allow-by-default enforcement
mechanisms (such as gatekeeper), the privileged profile may be an absence of applied constraints
rather than an instantiated policy. In contrast, for a deny-by-default mechanism (such as Pod
Security Policy) the privileged policy should enable all controls (disable all restrictions).
 -->
## 策略 {#policies}

### 特权 {#privileged}

特权策略是以开放为目的的，并且完全没有限制。这个策略类别通常是用于系统级和基础设施级别的工作负载，
它们由特权，受信的用户管理。

特权策略的定义方式就是没有限制。 对于 默认允许执行机制(如守门人(`gatekeeper`)), 特权方案是没有
执行的约束条件而不是一个实际的策略。相反的，在一个 默认拒绝的机制中(如 Pod 安全策略)，特权策略
就应该启用所有控制(关闭所有约束)。

<!--
### Baseline/Default

The Baseline/Default policy is aimed at ease of adoption for common containerized workloads while
preventing known privilege escalations. This policy is targeted at application operators and
developers of non-critical applications. The following listed controls should be
enforced/disallowed:

<table>
	<caption style="display:none">Baseline policy specification</caption>
	<tbody>
		<tr>
			<td><strong>Control</strong></td>
			<td><strong>Policy</strong></td>
		</tr>
		<tr>
			<td>Host Namespaces</td>
			<td>
				Sharing the host namespaces must be disallowed.<br>
				<br><b>Restricted Fields:</b><br>
				spec.hostNetwork<br>
				spec.hostPID<br>
				spec.hostIPC<br>
				<br><b>Allowed Values:</b> false<br>
			</td>
		</tr>
		<tr>
			<td>Privileged Containers</td>
			<td>
				Privileged Pods disable most security mechanisms and must be disallowed.<br>
				<br><b>Restricted Fields:</b><br>
				spec.containers[*].securityContext.privileged<br>
				spec.initContainers[*].securityContext.privileged<br>
				<br><b>Allowed Values:</b> false, undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>Capabilities</td>
			<td>
				Adding additional capabilities beyond the <a href="https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities">default set</a> must be disallowed.<br>
				<br><b>Restricted Fields:</b><br>
				spec.containers[*].securityContext.capabilities.add<br>
				spec.initContainers[*].securityContext.capabilities.add<br>
				<br><b>Allowed Values:</b> empty (or restricted to a known list)<br>
			</td>
		</tr>
		<tr>
			<td>HostPath Volumes</td>
			<td>
				HostPath volumes must be forbidden.<br>
				<br><b>Restricted Fields:</b><br>
				spec.volumes[*].hostPath<br>
				<br><b>Allowed Values:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>Host Ports</td>
			<td>
				HostPorts should be disallowed, or at minimum restricted to a known list.<br>
				<br><b>Restricted Fields:</b><br>
				spec.containers[*].ports[*].hostPort<br>
				spec.initContainers[*].ports[*].hostPort<br>
				<br><b>Allowed Values:</b> 0, undefined (or restricted to a known list)<br>
			</td>
		</tr>
		<tr>
			<td>AppArmor <em>(optional)</em></td>
			<td>
				On supported hosts, the 'runtime/default' AppArmor profile is applied by default. The default policy should prevent overriding or disabling the policy, or restrict overrides to an allowed set of profiles.<br>
				<br><b>Restricted Fields:</b><br>
				metadata.annotations['container.apparmor.security.beta.kubernetes.io/*']<br>
				<br><b>Allowed Values:</b> 'runtime/default', undefined<br>
			</td>
		</tr>
		<tr>
			<td>SELinux <em>(optional)</em></td>
			<td>
				Setting custom SELinux options should be disallowed.<br>
				<br><b>Restricted Fields:</b><br>
				spec.securityContext.seLinuxOptions<br>
				spec.containers[*].securityContext.seLinuxOptions<br>
				spec.initContainers[*].securityContext.seLinuxOptions<br>
				<br><b>Allowed Values:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>/proc Mount Type</td>
			<td>
				The default /proc masks are set up to reduce attack surface, and should be required.<br>
				<br><b>Restricted Fields:</b><br>
				spec.containers[*].securityContext.procMount<br>
				spec.initContainers[*].securityContext.procMount<br>
				<br><b>Allowed Values:</b> undefined/nil, 'Default'<br>
			</td>
		</tr>
		<tr>
			<td>Sysctls</td>
			<td>
				Sysctls can disable security mechanisms or affect all containers on a host, and should be disallowed except for an allowed "safe" subset.
				A sysctl is considered safe if it is namespaced in the container or the Pod, and it is isolated from other Pods or processes on the same Node.<br>
				<br><b>Restricted Fields:</b><br>
				spec.securityContext.sysctls<br>
				<br><b>Allowed Values:</b><br>
				kernel.shm_rmid_forced<br>
				net.ipv4.ip_local_port_range<br>
				net.ipv4.tcp_syncookies<br>
				net.ipv4.ping_group_range<br>
				undefined/empty<br>
			</td>
		</tr>
	</tbody>
</table>
 -->

### 基线/默认 {#baseline}

基线/默认策略的目的是在防止已知的特权升级情况下，让得普通的容器化工作负载来使用。 这个策略的目标
用户是应用运维人员和非关键应用的开发者。 下面列举这这些控制应该受限/不被允许:

<table>
	<caption style="display:none">基线策略规范</caption>
	<tbody>
		<tr>
			<td><strong>控制</strong></td>
			<td><strong>策略</strong></td>
		</tr>
		<tr>
			<td>宿主机命令空间</td>
			<td>
				Sharing the host namespaces must be disallowed.<br>
        主机命名空间分离必须禁止。
				<br><b>受限字段:</b><br>
				spec.hostNetwork<br>
				spec.hostPID<br>
				spec.hostIPC<br>
				<br><b>可用值:</b> false<br>
			</td>
		</tr>
		<tr>
			<td>特权容器</td>
			<td>
        特权 Pod 禁用了多数安全机制，必须禁止。
				<br><b>受限字段:</b><br>
				spec.containers[*].securityContext.privileged<br>
				spec.initContainers[*].securityContext.privileged<br>
				<br><b>可用值:</b> false, undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>Capabilities</td>
			<td>
        [默认集](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)
        外的额外能力是不允许的
				<br><b>受限字段:</b><br>
				spec.containers[*].securityContext.capabilities.add<br>
				spec.initContainers[*].securityContext.capabilities.add<br>
				<br><b>可用值:</b> empty (or restricted to a known list)<br>
			</td>
		</tr>
		<tr>
			<td>HostPath 卷</td>
			<td>
				HostPath 卷必须禁止<br>
				<br><b>受限字段:</b><br>
				spec.volumes[*].hostPath<br>
				<br><b>可用值:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>宿主机端口</td>
			<td>
				宿主机端口应该禁用, 或最少限制在一个已知列表.<br>
				<br><b>受限字段:</b><br>
				spec.containers[*].ports[*].hostPort<br>
				spec.initContainers[*].ports[*].hostPort<br>
				<br><b>可用值:</b> 0, undefined (或限制在一个已知列表)<br>
			</td>
		</tr>
		<tr>
			<td>AppArmor <em>(可选)</em></td>
			<td>
        在受支持的主机上，默认执行 'runtime/default' AppArmor 方案。 默认策略应该防止被覆盖或
        禁用策略，或限制对允许方案集的覆盖.<br>
				<br><b>受限字段:</b><br>
				metadata.annotations['container.apparmor.security.beta.kubernetes.io/*']<br>
				<br><b>可用值:</b> 'runtime/default', undefined<br>
			</td>
		</tr>
		<tr>
			<td>SELinux <em>(可选)</em></td>
			<td>
        不允许设置自定义SELinux选项.<br>
				<br><b>受限字段:</b><br>
				spec.securityContext.seLinuxOptions<br>
				spec.containers[*].securityContext.seLinuxOptions<br>
				spec.initContainers[*].securityContext.seLinuxOptions<br>
				<br><b>可用值:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>/proc 挂载类别</td>
			<td>
        默认的 /proc masks 被设置来减少攻击面， 是必要的.<br>
				<br><b>受限字段:</b><br>
				spec.containers[*].securityContext.procMount<br>
				spec.initContainers[*].securityContext.procMount<br>
				<br><b>可用值:</b> undefined/nil, 'Default'<br>
			</td>
		</tr>
		<tr>
			<td>Sysctls</td>
			<td>
        Sysctls 可以禁用安全机制或影响主机上的所有容器，在允许的安全子集外都应该被禁止。
        如果 sysctl 命名空间为容器内或 Pod 内被认为是安全的， 它会被与同一个节点上其它 Pod 或进程隔离.<br>
				<br><b>受限字段:</b><br>
				spec.securityContext.sysctls<br>
				<br><b>可用值:</b><br>
				kernel.shm_rmid_forced<br>
				net.ipv4.ip_local_port_range<br>
				net.ipv4.tcp_syncookies<br>
				net.ipv4.ping_group_range<br>
				undefined/empty<br>
			</td>
		</tr>
	</tbody>
</table>

<!--
### Restricted

The Restricted policy is aimed at enforcing current Pod hardening best practices, at the expense of
some compatibility. It is targeted at operators and developers of security-critical applications, as
well as lower-trust users.The following listed controls should be enforced/disallowed:


<table>
	<caption style="display:none">Restricted policy specification</caption>
	<tbody>
		<tr>
			<td><strong>Control</strong></td>
			<td><strong>Policy</strong></td>
		</tr>
		<tr>
			<td colspan="2"><em>Everything from the default profile.</em></td>
		</tr>
		<tr>
			<td>Volume Types</td>
			<td>
				In addition to restricting HostPath volumes, the restricted profile limits usage of non-core volume types to those defined through PersistentVolumes.<br>
				<br><b>Restricted Fields:</b><br>
				spec.volumes[*].hostPath<br>
				spec.volumes[*].gcePersistentDisk<br>
				spec.volumes[*].awsElasticBlockStore<br>
				spec.volumes[*].gitRepo<br>
				spec.volumes[*].nfs<br>
				spec.volumes[*].iscsi<br>
				spec.volumes[*].glusterfs<br>
				spec.volumes[*].rbd<br>
				spec.volumes[*].flexVolume<br>
				spec.volumes[*].cinder<br>
				spec.volumes[*].cephFS<br>
				spec.volumes[*].flocker<br>
				spec.volumes[*].fc<br>
				spec.volumes[*].azureFile<br>
				spec.volumes[*].vsphereVolume<br>
				spec.volumes[*].quobyte<br>
				spec.volumes[*].azureDisk<br>
				spec.volumes[*].portworxVolume<br>
				spec.volumes[*].scaleIO<br>
				spec.volumes[*].storageos<br>
				spec.volumes[*].csi<br>
				<br><b>Allowed Values:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>Privilege Escalation</td>
			<td>
				Privilege escalation (such as via set-user-ID or set-group-ID file mode) should not be allowed.<br>
				<br><b>Restricted Fields:</b><br>
				spec.containers[*].securityContext.allowPrivilegeEscalation<br>
				spec.initContainers[*].securityContext.allowPrivilegeEscalation<br>
				<br><b>Allowed Values:</b> false<br>
			</td>
		</tr>
		<tr>
			<td>Running as Non-root</td>
			<td>
				Containers must be required to run as non-root users.<br>
				<br><b>Restricted Fields:</b><br>
				spec.securityContext.runAsNonRoot<br>
				spec.containers[*].securityContext.runAsNonRoot<br>
				spec.initContainers[*].securityContext.runAsNonRoot<br>
				<br><b>Allowed Values:</b> true<br>
			</td>
		</tr>
		<tr>
			<td>Non-root groups <em>(optional)</em></td>
			<td>
				Containers should be forbidden from running with a root primary or supplementary GID.<br>
				<br><b>Restricted Fields:</b><br>
				spec.securityContext.runAsGroup<br>
				spec.securityContext.supplementalGroups[*]<br>
				spec.securityContext.fsGroup<br>
				spec.containers[*].securityContext.runAsGroup<br>
				spec.initContainers[*].securityContext.runAsGroup<br>
				<br><b>Allowed Values:</b><br>
				non-zero<br>
				undefined / nil (except for `*.runAsGroup`)<br>
			</td>
		</tr>
		<tr>
			<td>Seccomp</td>
			<td>
				The RuntimeDefault seccomp profile must be required, or allow specific additional profiles.<br>
				<br><b>Restricted Fields:</b><br>
				spec.securityContext.seccompProfile.type<br>
				spec.containers[*].securityContext.seccompProfile<br>
				spec.initContainers[*].securityContext.seccompProfile<br>
				<br><b>Allowed Values:</b><br>
				'runtime/default'<br>
				undefined / nil<br>
			</td>
		</tr>
	</tbody>
</table>
 -->

### 受限的 {#restricted}

受限策略旨在实施目前的 Pod 加固最佳实践，代价就是牺牲一些兼容性。目标用户为高安全性应用的运维和
开发人员，也可以是低信任度的用户。 下面列举的控制项应该受限/不被允许:

<table>
	<caption style="display:none">受限策略规范</caption>
	<tbody>
		<tr>
			<td><strong>Control</strong></td>
			<td><strong>Policy</strong></td>
		</tr>
		<tr>
			<td colspan="2"><em>所有都来自默认方案</em></td>
		</tr>
		<tr>
			<td>卷类型</td>
			<td>
        为了进一步限制 HostPath 卷， 受限方案在定义 PV 时限制了这些非核心卷类型的使用.<br>
				<br><b>受限字段:</b><br>
				spec.volumes[*].hostPath<br>
				spec.volumes[*].gcePersistentDisk<br>
				spec.volumes[*].awsElasticBlockStore<br>
				spec.volumes[*].gitRepo<br>
				spec.volumes[*].nfs<br>
				spec.volumes[*].iscsi<br>
				spec.volumes[*].glusterfs<br>
				spec.volumes[*].rbd<br>
				spec.volumes[*].flexVolume<br>
				spec.volumes[*].cinder<br>
				spec.volumes[*].cephFS<br>
				spec.volumes[*].flocker<br>
				spec.volumes[*].fc<br>
				spec.volumes[*].azureFile<br>
				spec.volumes[*].vsphereVolume<br>
				spec.volumes[*].quobyte<br>
				spec.volumes[*].azureDisk<br>
				spec.volumes[*].portworxVolume<br>
				spec.volumes[*].scaleIO<br>
				spec.volumes[*].storageos<br>
				spec.volumes[*].csi<br>
				<br><b>可用值:</b> undefined/nil<br>
			</td>
		</tr>
		<tr>
			<td>权限提升</td>
			<td>
				权限提升 (如通过设置 set-user-ID 或 set-group-ID 文件模式) 应该被禁止.<br>
				<br><b>受限字段:</b><br>
				spec.containers[*].securityContext.allowPrivilegeEscalation<br>
				spec.initContainers[*].securityContext.allowPrivilegeEscalation<br>
				<br><b>可用值:</b> false<br>
			</td>
		</tr>
		<tr>
			<td>以 非root用户运行</td>
			<td>
				要求容器必须以非 root 用户运行 .<br>
				<br><b>受限字段:</b><br>
				spec.securityContext.runAsNonRoot<br>
				spec.containers[*].securityContext.runAsNonRoot<br>
				spec.initContainers[*].securityContext.runAsNonRoot<br>
				<br><b>可用值:</b> true<br>
			</td>
		</tr>
		<tr>
			<td>非root 组 <em>(可选)</em></td>
			<td>
				容器应该避免以 root 主要或辅助的 GID 来运行 .<br>
				<br><b>受限字段:</b><br>
				spec.securityContext.runAsGroup<br>
				spec.securityContext.supplementalGroups[*]<br>
				spec.securityContext.fsGroup<br>
				spec.containers[*].securityContext.runAsGroup<br>
				spec.initContainers[*].securityContext.runAsGroup<br>
				<br><b>可用值:</b><br>
				non-zero<br>
				undefined / nil (`*.runAsGroup` 除外)<br>
			</td>
		</tr>
		<tr>
			<td>Seccomp</td>
			<td>
				必须要使用 RuntimeDefault 安全计算模式(seccomp) 方案，或允许指定额外方案.<br>
				<br><b>受限字段:</b><br>
				spec.securityContext.seccompProfile.type<br>
				spec.containers[*].securityContext.seccompProfile<br>
				spec.initContainers[*].securityContext.seccompProfile<br>
				<br><b>可用值:</b><br>
				'runtime/default'<br>
				undefined / nil<br>
			</td>
		</tr>
	</tbody>
</table>

<!--
## Policy Instantiation

Decoupling policy definition from policy instantiation allows for a common understanding and
consistent language of policies across clusters, independent of the underlying enforcement
mechanism.

As mechanisms mature, they will be defined below on a per-policy basis. The methods of enforcement
of individual policies are not defined here.

[**PodSecurityPolicy**](/docs/concepts/policy/pod-security-policy/)

- [Privileged](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/privileged-psp.yaml)
- [Baseline](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/baseline-psp.yaml)
- [Restricted](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/restricted-psp.yaml)
 -->

## 策略安装 {#policy-instantiation}

将策略定义从策略安装中解耦出来可以使用其成功一个共识和跨集群策略的统一语言，独立于底层的执行机制。

作为成熟机制， 它们会定义在单个策略基础之下。 每个策略的执行方法不是在这里定义的。
{{<todo-optimize>}}

[**PodSecurityPolicy**](/k8sDocs/docs/concepts/policy/pod-security-policy/)

- [Privileged](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/privileged-psp.yaml)
- [Baseline](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/baseline-psp.yaml)
- [Restricted](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/restricted-psp.yaml)
<!--
## FAQ

### Why isn't there a profile between privileged and default?

The three profiles defined here have a clear linear progression from most secure (restricted) to least
secure (privileged), and cover a broad set of workloads. Privileges required above the baseline
policy are typically very application specific, so we do not offer a standard profile in this
niche. This is not to say that the privileged profile should always be used in this case, but that
policies in this space need to be defined on a case-by-case basis.

SIG Auth may reconsider this position in the future, should a clear need for other profiles arise.
 -->

## FAQ

### 为啥在 特权(privileged)和基线(default) 之间没有一个中间方案  {#why-isnt-there-a-profile-between-privileged-and-default}

这里定义的三个方案有一个明显的线性变化， 从最安全(受限)到最不安全(特权)，覆盖了广泛的工作负载集。
在基线策略之上的权限需求通常是特别具体到应用的， 所以在这个层面我会不需要提供一个标准的方案。
这也不是说特权方案就会始终用于这个应用场景，但是在这个地方用到的策略就需要单个场景级别地来定义。

如果有明确的其它方案的需求提出来，未来 SIG Auth 可以重新考虑加到这里。

{{<todo-optimize>}}

### 安全策略与安全上下文之间的区别 {#whats-the-difference-between-a-security-policy-and-a-security-context}

[安全上下文](/k8sDocs/docs/tasks/configure-pod-container/security-context/) 是在
Pod 和 容器的运行时上配置。 安全上下文是在Pod 配置中作为 Pod 和容器定义的一部分存在的，代表
传递给容器运行时的参数。

安全策略是在安全上下文中执行指定设置的控制面板机制，连同安全上下文之外的其它参数。 从 2020 年 2 月
形如，当前原生执行这些安全策略的方案就是
[Pod 安全策略](/k8sDocs/docs/concepts/policy/pod-security-policy/) - 一个在集群中
以中心化方式在 Pod 上执行安全策略的机制。 其它执行安全策略的方式也已经在 k8s 生态中开发了，就
如
[OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper).

### Windows Pod 应该执行哪个方案 {#what-profiles-should-i-apply-to-my-windows-pods}

windows 在 k8s 中是有些限制且与基于 Linux 的标准工作负载有明显差异的。 特别是  Pod 的 SecurityContext
字段在
[Windows Pod 上是没有效果的](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#v1-podsecuritycontext). 因此，目前是没有标准的 Pod 安全方案存在的。

### 沙盒 Pod 是个啥情况 {#what-about-sandboxed-pods}

目前没有一个标准的 API 可以控制一个 Pod 是否被认为是沙盒。沙盒 Pod 可能可以通过使用一个沙盒
运行时(如 gVisor 或 Kata容器)来识别， 但没有对沙盒运行时有一个标准的定义。

对沙盒工作负载的保护需求可能与其它的不同。 例如，在工作负载从底层内核隔离后对特殊权限的限制需求就减少了。
这使得工作负载在隔离的情况下提升指定的权限。

另外，对沙盒工作负载的保护调度信赖沙盒的实现方式。 例如，没有一个推荐策略可以推荐给所有的沙盒工作负载。
