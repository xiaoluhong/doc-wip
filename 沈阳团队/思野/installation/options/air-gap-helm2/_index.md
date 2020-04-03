---
title: 使用Helm 2 在离线环境下安装 Rancher
---

> 在发布Helm 3之后，Rancher安装说明已更新为使用Helm 3。
>
> 如果您使用的是Helm 2，建议您[迁移到Helm 3](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/)，因为它比Helm 2更易于使用且更安全。
>
> 本节提供了使用Helm 2 在离线环境下安装 Rancher 比较旧的说明版本，适用于无法升级到Helm 3的情况。

本节介绍如何为离线安装Rancher准备您的节点。Rancher服务器可能会离线安装在防火墙或代理之后的封闭环境。这里有两个选项卡，用于高可用性（推荐）或Docker安装。

#### 离线安装 Kubernetes

本节介绍如何在离线环境中的 Kubernetes 集群上安装 Rancher。

Rancher server运行在由三个节点组成的高可用性的 Kubernetes 集群上。持久层(etcd)也安装在这三个节点上，以保证在其中一个节点出现错误时提供重复数据删除和数据备份。

#### 离线安装 Docker

本机还介绍了如何在离线环境中的单个节点上安装 Rancher。

Docker 安装是为想要测试 Rancher 的 Rancher 用户准备的。 您不需要在 Kubernetes 集群上运行，而是使用 docker run 命令在单个节点上安装 Rancher server 组件。 由于只有一个节点和一个 Docker 容器，如果该节点宕机，其他节点上就没有可用的 etcd 数据副本，您将丢失 Rancher 服务器的所有数据。与运行单节点安装不同，您可以选择遵循 Kubernetes 安装指南，但只使用一个节点来安装 Rancher。 然后，您可以扩展您的 Kubernetes 集群中的 etcd 节点，使其成为 Kubernetes 安装。

> **重要提示:** 如果您按照 Docker 安装指南安装 Rancher，则没有升级路径将 Docker 安装方式转换为 Kubernetes 安装方式。

您可以选择使用 Kubernetes 安装指南，而不必用 Docker 安装， 如果仅使用一个节点来安装Rancher。之后，您也可以扩展 Kubernetes 集群中的etcd节点，使其成为 Kubernetes 安装。

## 安装概要

- [1. 准备您的节点](/docs/installation/other-installation-methods/air-gap/prepare-nodes/)
- [2. 搜集并且推送镜像到私有镜像仓库](/docs/installation/other-installation-methods/air-gap/populate-private-registry/)
- [3. 使用 RKE 启动 Kubernetes 集群](/docs/installation/other-installation-methods/air-gap/launch-kubernetes/)
- [4. 安装 Rancher](/docs/installation/other-installation-methods/air-gap/install-rancher/)

#### [Next: Prepare your Node(s)](/docs/installation/other-installation-methods/air-gap/prepare-nodes/)
