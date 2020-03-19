---
title: 推荐的集群架构
---

有三个角色可以分配给节点: `etcd`, `controlplane` and `worker`.

## 将 worker 节点与其他角色的节点分离

在设计集群时, 您有两个选择:

- 为每个角色使用专用节点. 这可确保指定角色所需的组件的资源可用性。它还根据 [端口要求](/docs/cluster-provisioning/node-requirements/#networking-requirements/) 严格隔离每个角色之间的网络流量.
- 将`etcd`和`controlplane`角色分配给相同的节点. 这些节点必须满足这两个角色的硬件需求.

无论在哪种情况下, 都不应该使用`worker`角色, 也不应该将其添加到具有`etcd`或`controlplane`角色的节点中.

因此, 每个节点应该具有以下角色配置之一:

- `etcd`
- `controlplane`
- `etcd` 和 `controlplane`
- `worker`

## 每个角色的推荐节点数

集群应该具有:

- 至少有三个角色为`etcd`的节点可以在丢失一个节点时存活下来。增加这个计数以获得更高的节点容错率, 并将它们分散到(可用性)区域以提供更好的容错能力.
- 为了主组件高可用性，至少有两个角色为`controlplane`的节点。.
- 分配至少两个`worker`节点, 以在节点出现故障时, 为工作负载重新调度.

有关每个角色的用途的更多信息, 请参阅 [Kubernetes中关于节点的角色一节.](/docs/cluster-provisioning/production/nodes-and-roles)

#### Controlplane 节点数

使用`controlplane`角色添加多个节点可以使每个主组件高度可用.

#### etcd 节点数

在维护集群可用性时, 可以一次性丢失的节点数量由分配给`etcd`角色的节点数量决定. 对于一个有 n 个成员的集群, 最小值是(n/2)+1. 因此, 我们建议在一个区域内的3个不同的可用性区域中各创建一`etcd`节点, 以在失去一个可用性区域时生存下来. 如果您只使用两个区域, 那么您只能在没有丢失大部分节点的区域中存活.

| 具有`etcd`角色的节点 | Majority | 容错能力 |
| ---------------------- | -------- | ----------------- |
| 1                      | 1        | 0                 |
| 2                      | 2        | 0                 |
| 3                      | 2        | **1**             |
| 4                      | 3        | 1                 |
| 5                      | 3        | **2**             |
| 6                      | 4        | 2                 |
| 7                      | 4        | **3**             |
| 8                      | 5        | 3                 |
| 9                      | 5        | **4**             |

参考:

- [关于最佳etcd集群大小的官方etcd文档](https://etcd.io/docs/v3.4.0/faq/#what-is-failure-tolerance)
- [Kubernetes官方文档关于为Kubernetes运行etcd集群](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

#### Worker 节点数

使用`worker`角色添加多个节点将确保在节点失败时可以重新调度工作负载.

#### 为什么对 Rancher 群集和运行应用程序的群集的生产要求不同

您可能已经注意到, 我们的 [Kubernetes 安装](/docs/installation/k8s-install/)指令不符合我们对生产就绪群集的定义, 因为`Worker`角色没有专用节点. 但是,对于 Rancher 的安装, 这三个节点群集是有效的, 因为:
- 它允许一个`etcd`节点失败.
- 它通过拥有多个`controlplane`节点来维护主组件的多个实例.
- 除了Rancher本身之外, 不应该在此集群上创建其他工作负载.

## 参考

- [Kubernetes: 主组件](https://kubernetes.io/docs/concepts/overview/components/#master-components)
