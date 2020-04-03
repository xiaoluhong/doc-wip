---
title: '1. 准备您的节点'
---

本节介绍如何为离线安装Rancher准备您的节点。Rancher服务器可能会离线安装在防火墙或代理之后的封闭环境。这里有一些选项卡，用于高可用性（推荐）或Docker安装。

## 先决条件

 tabs 
 tab "Kubernetes 安装（推荐）" 

#### OS, Docker, Hardware, and Networking

确保您的节点满足常规[安装要求](/docs/installation/requirements/)。

#### 私有镜像仓库

Rancher 支持使用私有镜像仓库离线安装。您必须有自己的私有镜像仓库或其他可以可以在计算机上拉取 Docker 镜像的方法。

如果您需要有关创建私有镜像仓库的帮助，请参阅 [Docker documentation](https://docs.docker.com/registry/)。

#### CLI Tools

Kubernetes 安装需要如下 CLI 工具。确保这些工具已安装在您的工作站上，并且在您的计算机中可用 `$PATH`。

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes command-line tool.
- [rke]({{<baseurl>}}/rke/latest/en/installation/) - Rancher Kubernetes Engine, cli for building Kubernetes clusters.
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Package management for Kubernetes. Refer to the [Helm version requirements](/docs/installation/options/helm-version) to choose a version of Helm to install Rancher.

 /tab 
 tab "Docker安装" 

#### OS, Docker, Hardware, and Networking

确保您的节点满足常规[安装要求](/docs/installation/requirements/)。

#### 私有镜像仓库

Rancher 支持使用私有镜像仓库离线安装。您必须有自己的私有镜像仓库或其他可以在计算机上拉取 Docker 镜像的方法。

如果您需要有关创建私有镜像仓库的帮助，请参阅[Docker documentation](https://docs.docker.com/registry/)。
 /tab 
 /tabs 

## 设置基础架构

 tabs 
 tab "Kubernetes安装（推荐）" 

Rancher 建议在 Kubernetes 集群上安装 Rancher server。Rancher server运行在由三个节点组成的高可用性的 Kubernetes 集群上。持久层(etcd)也安装在这三个节点上，以保证在其中一个节点出现错误时提供重复数据删除和数据备份。

#### 推荐架构

- Rancher 的 DNS 应该解析为4层负载均衡器。
- 负载均衡器应将端口 TCP/80 和 TCP/443 转发到 Kubernetes 集群中的所有3个节点。
- Ingress控制器会将HTTP重定向到 HTTPS,并在端口 TCP/443 上终止 SSL/TLS。
- Ingress控制器会将流量转发到 Rancher deployment 中 Pod 上的端口TCP/80。

<figcaption>Kubernetes Rancher安装了4层负载均衡器，描述了ingress控制器的SSL终止</figcaption>

![Rancher HA](/img/rancher/ha/rancher2ha.svg)

#### A. 根据我们的要求配置三台离线的 linux 主机

这些主机将与Internet断开连接，但是必须可以连接到私有镜像仓库。

在[需求](/docs/installation/requirements)中查看每个集群节点的硬件和软件需求。

#### B. 设置 Load Balancer

设置将运行 Rancher 服务器组件的 Kubernetes 集群时, 将在每个节点上部署一个 Ingress 控制器容器。Ingress 控制器 Pod 绑定到主机网络上的端口TCP/ 80和TCP/ 443, 并且是Rancher server https的入口。

您将需要配置一个负载均衡器作为一个基本的4层 TCP 转发器，以将流量定向到这些入口控制器Pod。 更加准确的配置将根据您的环境而有所不同。

> **重要:**
> 仅使用此load balancer（即local群集Ingress）对 Rancher server进行负载平衡。与其他应用程序共享此 Ingress 可能会在其他应用的 Ingress 配置重新加载后导致 Rancher 出现 websocket 错误。

**负载均衡配置示例:**

- 有关如何设置 NGINX load balancer的示例，请参阅[this page](/docs/installation/k8s-install/create-nodes-lb/nginx)。
- 有关如何设置 Amazon NLB load balancer的示例，请参阅[this page](/docs/installation/k8s-install/create-nodes-lb/nlb)。

 /tab 
 tab "Docker 安装" 

Docker 安装是为想要测试 Rancher 的 Rancher 用户准备的。 您不需要在 Kubernetes 集群上运行，而是使用 docker run 命令在单个节点上安装 Rancher server 组件。 由于只有一个节点和一个 Docker 容器，如果该节点宕机，其他节点上就没有可用的 etcd 数据副本，您将丢失 Rancher 服务器的所有数据。与运行单节点安装不同，您可以选择遵循 Kubernetes 安装指南，但只使用一个节点来安装 Rancher。 然后，您可以扩展您的 Kubernetes 集群中的 etcd 节点，使其成为 Kubernetes 安装。

> **重要提示:** 如果您按照 Docker 安装指南安装 Rancher，则没有升级路径将 Docker 安装方式转换为 Kubernetes 安装方式。

您可以选择使用 Kubernetes 安装指南，而不必用 Docker 安装，如果仅使用一个节点来安装 Rancher 之后，您也可以扩展 Kubernetes 集群中的etcd节点，使其成为 Kubernetes 安装。

#### A. 根据我们的要求配置一个单节点离线 Linux 主机

这些主机将与Internet断开连接， 但是必须可以连接到私有镜像仓库。

在[需求](/docs/installation/requirements)中查看每个集群节点的硬件和软件需求。

 /tab 
 /tabs 

#### [Next: Collect and Publish Images to your Private Registry](/docs/installation/other-installation-methods/air-gap/populate-private-registry/)
