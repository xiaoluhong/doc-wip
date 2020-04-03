---
title: 生产就绪集群的检查列表
---

在本节中, 我们推荐了创建可用于生产的Kubernetes集群的最佳实践，这些集群将运行您的应用程序和服务.

对于创建一个集群的需求列表, 包括操作系统/Docker、硬件和网络的需求, 请参考 [节点需求.](/docs/cluster-provisioning/node-requirements)

这是我们强烈建议用于所有生产集群的最佳实践清单.

有关我们建议的所有最佳实践的完整列表, 请参阅 [最佳实践部分.](/docs/best-practices)

#### 节点需求

- 确保您的节点满足所有的 [节点需求,](/docs/cluster-provisioning/node-requirements/) 包括端口需求.

#### 备份 etcd

- 启用 etcd 快照. 验证是否正在创建快照, 并运行灾难恢复场景来验证快照是否有效. etcd是存储集群状态的位置, 丢失 etcd 数据意味着丢失集群. 请确保为你的集群配置 [etcd循环快照](/docs/backups/backups/ha-backups/#option-a-recurring-snapshots), 并确保快照也存储在外部(节点之外).

#### 集群架构

- 节点应具有以下角色配置之一：
  - `etcd`
  - `controlplane`
  - `etcd` and `controlplane`
  - `worker` (不应在具有`etcd`或`controlplane`角色的节点上使用或添加`worker`角色)
- 至少有三个角色为`etcd`的节点, 当失去一个节点时仍能存活. 增加此计数以提高节点容错率, 并将其分散到（可用性）区域, 以提供更好的容错能力.
- 分配两个或多个节点`controlplane`角色, 以实现主组件的高可用性.
- 分配两个或多个节点`worker`角色, 以在节点出现故障时, 为工作负载重新调度.

有关每个角色的用途的更多信息, 请参考 [Kubernetes中关于节点的角色一节.](/docs/cluster-provisioning/production/nodes-and-roles)

有关每个 Kubernets 角色的节点数的详细信息, 请参阅有关 [推荐的体系结构.](/docs/overview/architecture-recommendations/)

#### 日志记录和监控

- 为 Kubernetes 组件配置告警/通知程序(系统服务).
- 配置日志记录以进行集群分析和事后分析.

#### 可靠性

- 在集群上执行负载测试，以验证其硬件能够支持您的工作负载.

#### 网络

- 最小化网络延迟. Rancher 建议最小化 etcd 节点之间的延迟. `心跳间隔`的默认设置为`500`, 而`选举超时`的默认设置为`5000`. 这些[etcd调优设置](https://coreos.com/etcd/docs/latest/tuning.html)允许etcd在大多数网络（真正的高延迟网络除外）中运行.
- 集群节点应该位于单个区域内. 大多数云提供商在一个区域内提供多个可用性区域, 可用于为您的集群创建更高的可用性. 对于任何角色的节点, 使用多个可用性区域都是可以的. 如果您正在使用[Kubernetes云提供商](/docs/cluster-provisioning/rke-clusters/options/Cloud-providers/)资源, 请参考文档了解任何限制(例如区域存储限制).
