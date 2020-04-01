---
title: 在Kubernetes集群上安装Rancher
---

对于生产环境，我们建议以高可用配置安装Rancher，以便您的用户始终可以访问Rancher Server。当安装在Kubernetes集群中时，Rancher将与集群的etcd数据库集成，并利用Kubernetes调度来实现高可用性。

本节介绍如何使用RKE创建和管理集群，然后将Rancher安装到该集群上。对于这种类型的架构，您将需要在基础设施提供商中部署三个VM。您还需要配置负载均衡器，将前端流量定向到这三个VM。当VM运行并满足[节点要求](/docs/installation/requirements)时，可以使用RKE将Kubernetes部署到这些VM上，然后使用Helm软件包管理器将Rancher部署到Kubernetes上。

#### 可选：在单节点Kubernetes集群上安装Rancher

如果您只有一个节点，但您想在将来的生产中使用Rancher server，则最好将Rancher安装在单节点Kubernetes集群上，而不是使用Docker安装它。

一种选择是在Kubernetes集群上使用Helm安装Rancher，但仅使用集群中的单个节点。在这种情况下，Rancher server不具有高可用性，这对于在生产环境中运行Rancher至关重要。但是，如果您想在短期内通过使用单个节点来节省资源，同时又保留高可用性迁移路径，则此选项很有用。将来，您可以将节点添加到群集中以获得高可用的Rancher server。

通过为RKE配置Kubernetes集群时，可以通过在`cluster.yml`中只描述一个节点就可以实现单节点Kubernetes的安装。单个节点将具有所有三个角色：`etcd`，`controlplane`和`worker`。然后，使用Helm将Rancher安装在群集上，就像在其他任何群集上安装一样。

#### 关于架构的重要说明

Rancher管理服务器只能在RKE管理的Kubernetes集群上运行。不支持在托管的Kubernetes或其他提供商上使用Rancher。

为了获得最佳性能和安全性，我们建议为Rancher管理服务器使用专用的Kubernetes群集。不建议在此群集上运行用户工作负载。部署Rancher之后，您可以[创建或导入](/docs/cluster-provisioning/#cluster-creation-in-rancher)用于运行工作负载的群集。

我们建议负载均衡器和Ingress控制器使用以下架构和配置：

- Rancher的DNS应该解析为4层负载均衡器
- 负载均衡器应将端口TCP/80和TCP/443转发到Kubernetes集群中的所有3个节点。
- Ingress控制器会将HTTP重定向到HTTPS，并在端口TCP/443上终止SSL/TLS。
- Ingress控制器会将流量转发到Rancher deployment中Pod上的端口TCP/80。

有关Kubernetes安装如何工作的更多信息，请参阅[此页面](/docs/installation/how-ha-works)。

有关Rancher如何工作的信息（与安装方法无关），请参阅[架构部分。](/docs/overview/architecture)

### 需要的CLI工具

此安装需要以下CLI工具。请确保这些工具已经安装并在`$PATH`中可用

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes命令行工具.
- [rke]({{<baseurl>}}/rke/latest/en/installation/) - Rancher Kubernetes Engine, 用于构建Kubernetes集群的cli。
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes的软件包管理工具。请参阅[Helm版本要求](/docs/installation/options/helm-version)选择Helm的版本来安装Rancher。

### 安装摘要

- [创建节点和负载均衡器](/docs/installation/k8s-install/create-nodes-lb/)
- [使用RKE安装Kubernetes](/docs/installation/k8s-install/kubernetes-rke/)
- [安装Rancher](/docs/installation/k8s-install/helm-rancher/)

### 其他安装选项

- [从RKE Add-on安装的高可用kubernetes集群迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)
- [使用Helm 2安装Rancher:](/docs/installation/options/helm2) 本节提供了使用Helm 2安装高可用Rancher的说明，如果无法升级到Helm 3，则可以使用该说明。

### 以前的方法

[RKE add-on 安装](/docs/installation/options/rke-add-on/)

> **重要说明：RKE ADD-ON安装仅在RANCHER V2.0.8以上支持**
>
> 请使用Rancher Helm chart在Kubernetes集群上安装Rancher。更多信息请参阅[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)
>
> 如果您当前正在使用RKE add-on安装方式，请参阅[从RKE Add-on安装的Kubernetes集群迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)以获取有关如何使用Helm chart的详细信息。
