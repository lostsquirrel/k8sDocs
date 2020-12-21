---
title: Secret
id: secret
date: 2018-04-12
full_link: /k8sDocs/docs/concepts/configuration/secret/
short_description: >
  存放如密码, OAuth, ssh 密钥等敏感信息
aka:
tags:
- core-object
- security
---
 存放如密码, OAuth, ssh 密钥等敏感信息

<!--more-->
通过怎么使用敏感信息来控制意外暴露的风险，包括
[encryption](/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted)
{{< glossary_tooltip text="Pod" term_id="pod" >}} 可以以卷中的一个文件的方式引用 Secret
或通过 kubelet 拉取一个 Pod 的镜像。Secret 对敏感信息相当有用，
[ConfigMaps](/docs/tasks/configure-pod-container/configure-pod-configmap/)
用于非敏感信息
