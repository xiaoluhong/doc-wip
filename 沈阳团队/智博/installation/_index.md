---
title: Installing Rancher
---
安装Rancher

This section provides an overview of the architecture options of installing Rancher, describing advantages of each option.

本节整体概述了Rancher各种安装方式，并介绍了每个选项的优点。

#### Terminology

In this section,

**The Rancher server** manages and provisions Kubernetes clusters. You can interact with downstream Kubernetes clusters through the Rancher server's user interface.

**RKE (Rancher Kubernetes Engine)** is a certified Kubernetes distribution and CLI/library which creates and manages a Kubernetes cluster. When you create a cluster in the Rancher UI, it calls RKE as a library to provision Rancher-launched Kubernetes clusters.

在这个部分的主要涉及的名词如下：

** Rancher server**用于管理和配置Kubernetes集群。 您可以通过Rancher server的UI与下游Kubernetes集群进行交互。

** RKE（Rancher Kubernetes Engine）**是经过认证的Kubernetes发行版，它拥有对应的CLI工具可用于创建和管理Kubernetes集群。 在Rancher UI中创建集群时，它将调用RKE来配置Rancher启动的Kubernetes集群。

#### Overview of Installation Options
#### 安装选项概述

If you use Rancher to deploy Kubernetes clusters, it is important to ensure that the Rancher server doesn't fail, because if it goes down, you could lose access to the Kubernetes clusters that are managed by Rancher. For that reason, we recommend that for a production-grade architecture, you should set up a Kubernetes cluster with RKE, then install Rancher on it. After Rancher is installed, you can use Rancher to deploy and manage Kubernetes clusters.

如果您使用Rancher部署Kubernetes集群，需要确保Rancher server不会出现，因为如果它发生故障，您可能会失去对由Rancher管理的Kubernetes集群的访问权限。 因此，我们建议对于生产级架构，应使用RKE设置Kubernetes集群，然后在其上安装Rancher。 安装Rancher后，您可以使用Rancher部署和管理Kubernetes集群。

For testing or demonstration purposes, you can install Rancher in single Docker container. In this installation, you can use Rancher to set up Kubernetes clusters out-of-the-box.

为了测试或演示目的，您可以在单个Docker容器中安装Rancher，这个过程非常简洁，Rancher是基本是开箱即用。

Our [instructions for installing Rancher on Kubernetes](/docs/installation/k8s-install) describe how to first use RKE to create and manage a cluster, then install Rancher onto that cluster. For this type of architecture, you will need to deploy three nodes - typically virtual machines - in the infrastructure provider of your choice. You will also need to configure a load balancer to direct front-end traffic to the three nodes. When the nodes are running and fulfill the [node requirements,](/docs/installation/requirements) you can use RKE to deploy Kubernetes onto them, then use Helm to deploy Rancher onto Kubernetes.

在[Kubernetes上安装Rancher的说明](/docs/installation/k8s-install)文档中描述了如何首先使用RKE创建和管理集群，然后将Rancher安装到该集群上。 对于这种类型的部署方式，您将需要三个节点作为基础架构（通常是虚拟机）。 您还需要配置负载均衡器，以将前端流量定向到这三个节点。 当节点运行并满足[节点要求](/docs/installation/requirements)时，可以使用RKE将Kubernetes部署到它们上，然后使用Helm将Rancher部署到Kubernetes上。

For a longer discussion of Rancher architecture, refer to the [architecture overview,](/docs/overview/architecture) [recommendations for production-grade architecture,](/docs/overview/architecture-recommendations) or our [best practices guide.](/docs/best-practices/deployment-types)

有关Rancher部署架构的详细讨论，请参考[架构概述](/docs/overview/architecture)[生产级部署架构的建议，](/docs/overview/architecture-recommendations)或我们的[最佳实践指南]。(/docs/best-practices/deployment-types)

Rancher can be installed on these main architectures:

- **High-availability Kubernetes Install:** We recommend using [Helm,](/docs/overview/concepts/#about-helm) a Kubernetes package manager, to install Rancher on a dedicated Kubernetes cluster. We recommend using three nodes in the cluster because increased availability is achieved by running Rancher on multiple nodes.
- **Single-node Kubernetes Install:** Another option is to install Rancher with Helm on a Kubernetes cluster, but to only use a single node in the cluster. In this case, the Rancher server doesn't have high availability, which is important for running Rancher in production. However, this option is useful if you want to save resources by using a single node in the short term, while preserving a high-availability migration path. In the future, you can add nodes to the cluster to get a high-availability Rancher server.
- **Docker Install:** For test and demonstration purposes, Rancher can be installed with Docker on a single node. This installation works out-of-the-box, but there is no migration path from a Docker installation to a high-availability installation on a Kubernetes cluster. Therefore, you may want to use a Kubernetes installation from the start.

Rancher的部署可以有三种架构：

- **高可用Kubernetes安装：**我们建议使用Kubernetes程序包管理器[Helm，](/docs/overview/concepts/#about-helm)在专用的Kubernetes集群上安装Rancher。我们建议在集群中使用三个节点，因为通过在多个节点上运行Rancher可以提高可用性。
- **单节点Kubernetes安装：**另一个选择是在Kubernetes群集上使用Helm安装Rancher，但仅在群集中使用单个节点。在这种情况下，Rancher服务器不具有高可用性，这对于在生产环境中运行Rancher并不友好。但是，如果您想在短期内通过使用单个节点来节省资源，同时又保留高可用性迁移路径，则此选项很有用。将来，您可以将节点添加到群集中以获得高可用的Rancher server。
-** Docker安装：**为了进行测试和演示，可以将Rancher与Docker一起安装在单个节点上。此安装是开箱即用的，但是这种在后续会很难迁移到Kubernetes集群上。因此，您可能希望一开始就使用Kubernetes安装。

The single-node Kubernetes install is achieved by describing only one node in the `cluster.yml` when provisioning the Kubernetes cluster with RKE. The single node should have all three roles: `etcd`, `controlplane`, and `worker`. Then Rancher can be installed with Helm on the cluster in the same way that it would be installed on any other cluster.

通过RKE配置Kubernetes集群时，仅在`cluster.yml`中描述一个节点即可实现单节点Kubernetes的安装。 单个节点应具有所有三个角色："etcd"，"controlplane""和"worker"。 然后，可以使用Helm将Rancher安装在群集上，就像在其他任何群集上安装一样。

There are also separate instructions for installing Rancher in an air gap environment or behind an HTTP proxy:

| Level of Internet Access           | Kubernetes Installation - Strongly Recommended                                                                                 | Docker Installation                                                                                                                                                  |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| With direct access to the Internet | [Docs](/docs/installation/k8s-install/)                                                                                        | [Docs](/docs/installation/other-installation-methods/single-node-docker)                                                                                             |
| Behind an HTTP proxy               | These [docs,](/docs/installation/k8s-install/) plus this [configuration](/docs/installation/options/chart-options/#http-proxy) | These [docs,](/docs/installation/other-installation-methods/single-node) plus this [configuration](/docs/installation/other-installation-methods/single-node/proxy/) |
| In an air gap environment          | [Docs](/docs/installation/other-installation-methods/air-gap)                                                                  | [Docs](/docs/installation/other-installation-methods/air-gap)                                                                                                        |

关于在私有环境中或HTTP代理后面安装Rancher的单独说明：

| 网络访问水平                       | 基于Kubernetes安装（推荐）                                                                                                     | 基于Docker安装                                                                                                                                                       |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可直接访问Internet                 | [文档](/docs/installation/k8s-install/)                                                                                        | [文档](/docs/installation/other-installation-methods/single-node-docker)                                                                                             |
| 依托Http代理                       | 基于[此文档,](/docs/installation/k8s-install/) 并附加 [相关配置](/docs/installation/options/chart-options/#http-proxy)         | 基于 [此文档,](/docs/installation/other-installation-methods/single-node) 并附加 [相关配置](/docs/installation/other-installation-methods/single-node/proxy/)        |
| 私有网络环境                       | [文档](/docs/installation/other-installation-methods/air-gap)                                                                  | [文档](/docs/installation/other-installation-methods/air-gap)                                                                                                        |

#### Prerequisites
安装要求

Before installing Rancher, make sure that your nodes fulfill all of the [installation requirements.](/docs/installation/requirements/)

在安装Rancher之前，请确保您的节点满足所有[安装要求。](/docs/installation/requirements/)

#### Architecture Tip
架构技巧

For the best performance and greater security, we recommend a separate, dedicated Kubernetes cluster for the Rancher management server. Running user workloads on this cluster is not advised. After deploying Rancher, you can [create or import clusters](/docs/cluster-provisioning/#cluster-creation-in-rancher) for running your workloads.

为了获得最佳性能和更高的安全性，我们建议为Rancher管理服务器使用单独的专用Kubernetes集群。 不建议在此群集上运行用户工作负载。 部署Rancher后，您可以[创建或导入群集](/docs/cluster-provisioning/#cluster-creation-in-rancher)运行您的工作负载。

For more architecture recommendations, refer to [this page.](/docs/overview/architecture-recommendations)

有关更多架构建议，请参阅[本页。](/docs/overview/architecture-recommendations)

#### More Options for Installations on a Kubernetes Cluster
在Kubernetes上安装Rancher的更多选项

Refer to the [Helm chart options](/docs/installation/options/chart-options/) for details on installing Rancher on a Kubernetes cluster with other configurations, including:

- With [API auditing to record all transactions](/docs/installation/options/chart-options/#api-audit-log)
- With [TLS termination on a load balancer](/docs/installation/options/chart-options/#external-tls-termination)
- With a [custom Ingress](/docs/installation/options/chart-options/#customizing-your-ingress)

关于在Kubernetes上安装Rancher的详细配置，请参考[Helm chart 选项](/docs/installation/options/chart-options/)：

- [开启API审计](/docs/installation/options/chart-options/#api-audit-log)
- [在负载均衡器上做TLS termination](/docs/installation/options/chart-options/#external-tls-termination)
- [自定义Ingress](/docs/installation/options/chart-options/#customizing-your-ingress)

In the Rancher installation instructions, we recommend using RKE (Rancher Kubernetes Engine) to set up a Kubernetes cluster before installing Rancher on the cluster. RKE has many configuration options for customizing the Kubernetes cluster to suit your specific environment. Please see the [RKE Documentation]({{<baseurl>}}/rke/latest/en/config-options/) for the full list of options and capabilities.

在Rancher安装说明中，推荐使用RKE设置Kubernetes并基于它部署Rancher。RKE有许多配置选项可用于自定义Kubernetes以适合您的特定环境。 有关选项和功能的完整列表，请参见[RKE文档]({{<baseurl>}}/rke/latest/en/config-options/）。

#### More Options for Installations with Docker
在Docker上安装Rancher的更多选项

Refer to the [Docker installation docs](/docs/installation/other-installation-methods/single-node-docker) for details other configurations including:

- With [API auditing to record all transactions](/docs/installation/other-installation-methods/single-node-docker/#api-audit-log)
- With an [external load balancer](/docs/installation/other-installation-methods/single-node-docker/single-node-install-external-lb/)
- With a [persistent data store](/docs/installation/other-installation-methods/single-node-docker/#persistent-data)

有关其详细配置，请参考[Docker安装方式](/docs/installation/other-installation-methods/single-node-docker)：

- [开启API审计](/docs/installation/other-installation-methods/single-node-docker/#api-audit-log)
- [外部负载均衡](/docs/installation/other-installation-methods/single-node-docker/single-node-install-external-lb/)
- [持久化存储](/docs/installation/other-installation-methods/single-node-docker/#persistent-data)

