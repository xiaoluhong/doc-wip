---
title: Rancher 代理
---

有两个不同的代理资源方式部署在 Rancher 管理的集群:

- [cattle-cluster-agent](#cattle-cluster-agent)
- [cattle-node-agent](#cattle-node-agent)

有关 Rancher 服务器如何配置群集并与之通信的概念概述，请参阅 [架构](/docs/overview/architecture/)

#### cattle-cluster-agent

`cattle-cluster-agent` 用于连接集群的 Kubernetes API [Rancher 推出的 Kubernetes](/docs/cluster-provisioning/rke-clusters/). 通过部署资源部署 `cattle-cluster-agent`.

#### cattle-node-agent

在执行集群操作时, `cattle-node-agent` 用于和集群[Rancher 推出 Kubernetes](/docs/cluster-provisioning/rke-clusters/)中的节点进行交互. 集群操作的示例包括升级 Kubernetes 版本和创建/恢复 etcd 快照. 通过守护进程资源部署 `cattle-node-agent`, 以确保其在每个节点上运行. 当 `cattle-node-agent` 不可用时, `cattle-node-agent` 作为回退选项连接到 Kubernetes API of [Rancher 推出的 Kubernetes](/docs/cluster-provisioning/rke-clusters/).

> **注意:** 在Rancher v2.2.4及以下版本中，`cattle-node-agent` pod 无法忍受所有污染，导致Kubernetes升级失败。Rancher v2.2.5及更高版本中包含了对此的修复.

#### 调度规则

_适用于 v2.3.0 及更高版本_

| 组件              | 节点亲和性和节点选择器        | 节点选择器 | 容忍       |
| ---------------------- | ------------------------------------- | ------------ | ----------------- |
| `cattle-cluster-agent` | `beta.kubernetes.io/os:NotIn:windows` | none         | `operator:Exists` |
| `cattle-node-agent`    | `beta.kubernetes.io/os:NotIn:windows` | none         | `operator:Exists` |

`cattle-cluster-agent` 部署首选 `requiredDuringSchedulingIgnoredDuringExecution` 作为调度规则, 倾向于在具有   `controlplane` 角色的节点上调度. 参考 [Kubernetes: 将 pod 分配给节点](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) 可以找到更多关于调度规则的信息.

`requiredDuringSchedulingIgnoredDuringExecution` 配置如下表所示:

| Weight | Expression                                       |
| ------ | ------------------------------------------------ |
| 100    | `node-role.kubernetes.io/controlplane:In:"true"` |
| 1      | `node-role.kubernetes.io/etcd:In:"true"`         |
