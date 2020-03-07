---
title: '1. 准备您的节点'
---

本节介绍如何为离线安装Rancher准备您的节点。Rancher服务器可能会离线安装在防火墙或代理之后的封闭环境。这里有两个选项卡，用于高可用性（推荐）或Docker安装。

## 先决条件

 tabs 
 tab "Kubernetes 安装 (推荐)" 

#### 操作系统，Docker，硬件和网络

确保您的节点满足常规的[安装要求](/docs/installation/requirements/)。

#### 私有仓库

Rancher支持使用私有仓库进行离线安装。您必须具有自己的私有仓库或其他将Docker镜像分发到计算机的方法。

如果需要有关创建私有仓库的帮助，请参考[Docker文档](https://docs.docker.com/registry/)。

#### CLI 工具

Kubernetes安装需要以下CLI工具。确保这些工具已安装在您的工作站上，并且在您的计算机中可用`$PATH`。

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes命令行工具。
- [rke]({{<baseurl>}}/rke/latest/en/installation/) - Rancher Kubernetes引擎，用于构建Kubernetes集群的cli。
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes的软件包管理。请参阅[Helm版本要求](/docs/installation/options/helm-version)以选择Helm的版本来安装Rancher。

 /tab 
 tab "Docker 安装" 

#### 操作系统，Docker，硬件和网络

确保您的节点满足常规的[安装要求](/docs/installation/requirements/)。

#### 私有仓库

Rancher支持使用私有仓库进行离线安装。您必须具有自己的私有仓库或其他将Docker镜像分发到计算机的方法。

如果需要有关创建私有仓库的帮助，请参考[Docker文档](https://docs.docker.com/registry/)。
 /tab 
 /tabs 

## 设置基础架构

 tabs 
 tab "Kubernetes 安装 (推荐)" 

Rancher建议在Kubernetes集群上安装Rancher。一个高可用的Kubernetes安装由三个节点组成，这些节点在Kubernetes集群上运行Rancher服务器组件。持久性层(etcd)也复制到这三个节点上，在其中一个节点发生故障时提供冗余和数据复制。

#### 推荐架构

- Rancher的DNS应该解析为4层负载均衡器
- 负载均衡器应将端口 TCP/80 和 TCP/443 转发到Kubernetes集群中的所有3个节点。
- Ingress控制器会将HTTP重定向到HTTPS，并在端口 TCP/443 上终止 SSL/TLS。
- Ingress控制器会将流量转发到Rancher部署中Pod上的端口TCP/80。

<figcaption>Rancher安装在具有4层负载均衡器的Kubernetes集群上，描绘了Ingress控制器处的SSL终止</figcaption>

![Rancher HA](/img/rancher/ha/rancher2ha.svg)

#### A. 根据我们的要求配置三台离线的Linux主机

这些主机将与Internet断开连接，但需要能够与您的私有仓库连接。

在[需求](/docs/installation/requirements)中查看每个集群节点的硬件和软件需求。

#### B. 设置您的负载均衡器

在设置运行Rancher服务器组件的Kubernetes集群时，将在每个节点上部署一个ingress控制器pod。Ingress控制器pod绑定到主机网络上的TCP/80和TCP/443端口，并且是到Rancher服务器的HTTPS流量的入口点。

您需要将负载均衡器配置为基本的4层TCP转发器，以将流量定向到这些ingress控制器pod。确切的配置将取决于您的环境。

> **重要：**
> 仅使用此负载均衡器（即`local` 集群Ingress）对Rancher服务器进行负载均衡。与其他应用程序共享此Ingress可能会在其他应用的Ingress配置重新加载后导致Rancher出现websocket错误。

**负载均衡器配置示例：**

- 有关如何设置NGINX负载均衡器的示例，请参考[此页](/docs/installation/k8s-install/create-nodes-lb/nginx)。
- 有关如何设置Amazon NLB负载均衡器的示例，请参考[此页](/docs/installation/k8s-install/create-nodes-lb/nlb)。

 /tab 
 tab "Docker 安装" 

Docker安装适用于想要测试Rancher的Rancher用户。您可以使用`docker run`命令在单个节点上安装Rancher服务器组件，而不是在Kubernetes集群上运行。由于只有一个节点和一个Docker容器，因此，如果该节点发生故障，则其他节点上没有可用的etcd数据副本，您将丢失Rancher服务器的所有数据。

> **重要：** 如果您按照Docker安装指南安装Rancher，则没有升级路径可将Docker安装过渡到Kubernetes安装。

仅使用一个节点来安装Rancher，您可以选择遵循Kubernetes安装指南，而不必运行Docker安装。之后，您可以扩展Kubernetes集群中的etcd节点，使其成为Kubernetes安装。

#### A. 根据我们的要求配置单个离线的Linux主机

这些主机将与Internet断开连接，但需要能够与您的私有仓库连接。

在[需求](/docs/installation/requirements)中查看每个集群节点的硬件和软件需求。

 /tab 
 /tabs 

#### [下一步: 收集并将镜像发布到您的私有仓库](/docs/installation/other-installation-methods/air-gap/populate-private-registry/)

