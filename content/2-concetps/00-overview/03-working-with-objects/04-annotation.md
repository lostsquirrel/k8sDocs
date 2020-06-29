---
title:
weight:
date: 2020-06-29
draft: true
---
# 注解 (Annotations)
为对象添加非标识性元数据(non-identifying)
常见使用注解的示例：
- Fields managed by a declarative configuration layer. Attaching these fields as annotations distinguishes them from default values set by clients or servers, and from auto-generated fields and fields set by auto-sizing or auto-scaling systems.
- 构建，发布，镜像相关信息如 时间戳，发布编号，git 分支，PR 编号，镜像 hash, 镜像库地址
- Pointers to logging, monitoring, analytics, or audit repositories.
- 客户端库或工具用于调试目的的信息 例如：名称, 版本, 构建信息.
- 来自用户或 工具/系统 信息, 例如 对象相关的外部系统URLs
- Lightweight rollout tool metadata: for example, config or checkpoints.
- 负责人的联系信息
