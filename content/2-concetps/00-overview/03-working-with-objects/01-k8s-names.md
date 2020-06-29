---
title:
weight:
date: 2020-06-29
draft: true
---

# k8s 对象命名
所有 k8s API 中的对象，可以通过 名字+UID 进行唯一标识
用户可以通过 标签(labels) 和 注释(annotations) 向对象添加非唯一的属性
关于名称和UID,详见(https://git.k8s.io/community/contributors/design-proposals/architecture/identifiers.md)

## 对象名称
    由客户端提供关联到资源URL上的一个对象，比如：/api/v1/pods/some-name.
    同一类型的对象名称不能重复，如果对象已经删除则可以再次使用这个名称创建新的对象
    对象名称最多可以有 253个字符，包含 小写字母和数字，中划线(-),点(.), 具体到每类对象还有更严格的限制

## UIDs
    由系统创建的对象的唯一标识，类型为字符串
    k8s 集群整个生命周期内，创建的每一个对象的UID都是唯一的，用于区分可能存在或曾今过的相似对象
