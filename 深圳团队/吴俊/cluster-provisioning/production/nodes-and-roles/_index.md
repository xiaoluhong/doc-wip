---
title: Kubernetes 中节点的角色
---

本节描述 Kubernetes 中的 `etcd` 节点、`controlplane` 节点和`worker` 节点的角色, 以及这些角色如何在集群中协同工作

这个图适用于 Kubernetes 集群 [使用 RKE 和 Rancher 一起启动.](/docs/cluster-provisioning/rke-clusters/).

![集群图片](/img/rancher/clusterdiagram.svg)<br/>
<sup>线条显示组件之间的通信流. 颜色纯粹用于视觉辅助</sup>

## etcd

使用`etcd`角色运行etcd节点, 这是一个一致的、高度可用的键值存储方式, 用于 Kubernetes 对所有集群数据的备份存储. etcd 将数据复制到每个节点.

> **注意:** 在UI中如果具有`etcd`角色的节点显示为`Unschedulable`, 这意味着在默认情况下不会将pod调度到这些节点.

## controlplane

在具有`controlplane`角色的节点上运行 Kubernetes 主组件(不包括 `etcd`, 因为它是一个单独的角色). 有关主组件的详细列表, 请参阅 [Kubernetes: 主组件](https://kubernetes.io/docs/concepts/overview/components/#master-components).

> **注意:** 在UI中如果具有`controlplane`角色的节点显示为`Unschedulable`, 这意味着在默认情况下不会将pod调度到这些节点.

#### kube-apiserver

Kubernetes API服务器（`kube-apiserver`）是水平扩展的. 每个具有`controlplane`角色的节点将被添加到具有需要访问Kubernetes API服务器的组件的节点上的NGINX代理中. 这意味着如果一个节点变得不可调度, 该节点上的本地 NGINX 代理将把请求转发到列表中的另一个 Kubernetes API 服务器.

#### kube-controller-manager

The Kubernetes controller manager uses leader election using an endpoint in Kubernetes. One instance of the `kube-controller-manager` will create an entry in the Kubernetes endpoints and updates that entry in a configured interval. Other instances will see an active leader and wait for that entry to expire (for example, when a node is unresponsive).

#### kube-scheduler

The Kubernetes scheduler uses leader election using an endpoint in Kubernetes. One instance of the `kube-scheduler` will create an entry in the Kubernetes endpoints and updates that entry in a configured interval. Other instances will see an active leader and wait for that entry to expire (for example, when a node is unresponsive).

## worker

具有`worker`角色的节点运行 Kubernetes 节点组件. 参见 [Kubernetes: 节点组件](https://kubernetes.io/docs/concepts/overview/components/#node-components) 的详细列表.

## 参考

- [Kubernetes: 节点组件](https://kubernetes.io/docs/concepts/overview/components/#node-components)
