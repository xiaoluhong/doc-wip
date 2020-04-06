---
title: 初始化Windows初始化
---

_从v2.3.0开始支持_

当使用Rancher初始化一个[自定义集群](/docs/cluster-provisioning/custom-clusters/)时，Rancher会在您的基础架构资源上，使用RKE(the Rancher Kubernetes Engine)进行Kubernetes集群初始化。

您可以同时使用Linux以及Windows的主机组成您的集群。Windows节点只能作为工作负载节点使用，Linux则需要作为管理节点。

您只能在集群是启用Windows支持的情况下才可以将一个Windows节点添加到集群当中。Windows支持只能适用于自定义集群并且Kubernetes版本1.15+以及Flannel网络插件。Windows不能在已创建的集群中打开。

> Windows集群比Linux集群有需要有更多的前提田间。例如，Windows节点必须有50GB的磁盘空间，并且需要保证满足所有一下的节点[需求](#requirements-for-windows-clusters)。

Kubernetes的Windows节点支持特性汇总如[文档 supported functionality and limitations for using Kubernetes with Windows](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)或者参考[指引 guide for scheduling Windows containers in Kubernetes](https://kubernetes.io/docs/setup/production-environment/windows/user-guide-windows-containers/)。

本手册包含以下的主题

<!-- TOC -->

- [先决条件](#先决条件)
- [要求](#Windows集群要求)
  - [OS和Docker](#OS和Docker)
  - [节点](#节点)
  - [网络](#网络)
  - [架构](#架构)
  - [容器](#容器)
- [教程：如何在Windows支持下创建集群](#教程：如何在Windows支持下创建集群)
- [Azure中存储类的配置](#Azure中存储类的配置)
<!-- /TOC -->

## 先决条件

在配置新集群之前，请确保已在接受入站网络流量的设备上安装了Rancher。 为了使集群节点与Rancher通信，这是必需的。 如果尚未安装Rancher，请在继续本指南之前参考[安装文档](/docs/installation/)。

> **Cloud Providers注意事项:** 如果您在集群中设置Kubernetes cloud provider，则需要执行一些其他步骤。 如果要利用云提供商的功能（例如，为集群自动配置存储，负载平衡器或其他基础结构），则可能需要设置云提供商。 请参阅[本页](/docs/cluster-provisioning/rke-clusters/options/cloud-providers/)，以获取有关如何配置满足前提条件的节点的云提供商集群的详细信息。

## Windows集群要求

对于自定义集群，网络，操作系统和Docker的一般节点要求与[Rancher安装](/docs/installation/requirements/)的节点要求相同。

#### OS和Docker

为了将Windows工作程序节点添加到集群，该节点必须运行以下Windows Server版本之一和相应版本的Docker Engine-Enterprise Edition（EE）：

- 具有Windows Server核心版本1809的节点应使用Docker EE-basic 18.09或Docker EE-basic 19.03。
- 具有Windows Server核心版本1903的节点应使用Docker EE-basic 19.03。

> **注意事项：**
>
> - 如果您使用的是AWS，Rancher建议将 _Microsoft Windows Server 2019 Base with Containers_ 作为Amazon Machine Image（AMI）。
> - 如果您使用的是GCE，Rancher建议将 _Windows Server 2019 Datacenter for Containers_ 作为OS镜像。

#### 节点

集群中的主机至少必须具有：

- 2 core CPUs
- 5 GB memory
- 50 GB 磁盘空间

如果节点不满足这些要求，Rancher将不会纳管该节点。

#### 网络

Rancher仅支持使用Flannel作为网络提供程序的Windows。

有两个网络选项：[**Host Gateway (L2bridge)**](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw)和[**VXLAN (Overlay)**](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan)。默认选项是 **VXLAN (Overlay)** 模式。

对于 **Host Gateway (L2bridge)** 联网，最好对所有节点使用相同的第2层网络。否则，您需要为其配置路由规则。有关详细信息，请参阅[有关配置云托管的VM路由的文档。](/docs/cluster-provisioning/rke-clusters/windows-clusters/host-gateway-requirements/#cloud-hosted-vm-routes-configuration)，如果您使用的是亚马逊EC2，Google GCE或Azure VM，还需要[禁用私有IP地址检查](/docs/cluster-provisioning/rke-clusters/windows-clusters/host-gateway-requirements/#disabling-private-ip-address-checks)。

对于 **VXLAN (Overlay)** 网络，必须安装[KB4489899](https://support.microsoft.com/en-us/help/4489899)修补程序。大多数云托管的VM已经具有此修补程序。

#### 架构

Kubernetes集群管理节点(`etcd`和`controlplane`)必须在Linux节点上运行。

工作负载将部署在其中的`worker`节点通常是Windows节点，但是必须至少有一个在Linux上运行的`worker`节点才能运行Rancher cluster agent，Metrics Server，DNS和与Ingress相关的容器。

我们建议您在下表中列出最低的三节点体系结构，但是您始终可以添加其他Linux和Windows工作者以扩展集群以实现冗余：

<a id="guide-architecture"></a>

| 节点 | 操作系统 | 集群角色 | 目的 |
| --- | ------- | ------- | --- |
| Node 1 | Linux (推荐Ubuntu Server 18.04) | [Control Plane](/docs/cluster-provisioning/#control-plane-nodes), [etcd](/docs/cluster-provisioning/#etcd-nodes), [Worker](/docs/cluster-provisioning/#worker-nodes) | 管理Kubernetes集群 |
| Node 2 | Linux (推荐Ubuntu Server 18.04) | [Worker](/docs/cluster-provisioning/#worker-nodes) | Support the Rancher 集群的Cluster agent, Metrics server, DNS, and Ingress |
| Node 3 | Windows (Windows Server core version 1809 或以上版本) | [Worker](/docs/cluster-provisioning/#worker-nodes) | 运行Windows容器 |

#### 容器

Windows要求容器必须建立在与容器相同的Windows Server版本上。 因此，必须在Windows Server核心版本1809或更高版本上构建容器。 如果您已经为早期的Windows Server核心版本构建了现有容器，则必须在Windows Server核心版本1809或更高版本上重新构建它们。

## 教程：如何在Windows支持下创建集群

本教程介绍了如何使用[推荐的体系结构](#guide-architecture)中的三个节点创建Rancher配置的集群。

使用Rancher设置自定义集群时，将通过在每个集群上安装[Rancher agent](/docs/cluster-provisioning/custom-clusters/agent-options/)将节点添加到集群中。 从Rancher UI创建或编辑集群时，您将看到一个**自定义Node启动命令**，您可以在每个服务器上运行该命令以将其添加到自定义集群中。

要设置支持Windows节点和容器的自定义集群，您需要完成以下任务。

<!-- TOC -->

1. [初始化主机](#1-初始化主机)
1. [创建自定义集群](#2-创建自定义集群)
1. [在集群上添加节点](#3-在集群上添加节点)
1. [可选：配置Azure文件](#5-可选：配置Azure文件)
   <!-- /TOC -->

## 1. 初始化主机

要开始配置具有Windows支持的自定义集群，请准备主机。

您的主机可以是：

- 云托管的虚拟机
- 虚拟化集群中的VM
- 裸机服务器

您将置备三个节点：

- 一个Linux节点，用于管理Kubernetes控制平面并存储您的`etcd`
- 第二个Linux节点，它将是另一个工作节点
- Windows节点，它将Windows容器作为工作节点运行

| 节点    | 操作系统                                                      |
| ------ | ------------------------------------------------------------ |
| Node 1 | Linux (推荐Ubuntu Server 18.04)                      |
| Node 2 | Linux (推荐Ubuntu Server 18.04)                      |
| Node 3 | Windows (Windows Server core version 1809或以上)     |

如果您的节点由 **Cloud Provider** 托管，并且您需要自动化支持（例如负载平衡器或永久性存储设备），则您的节点还有其他配置要求。 有关详细信息，请参阅[选择云提供商。](/docs/cluster-provisioning/rke-clusters/options/cloud-providers)

## 2. 创建自定义集群

创建支持Windows节点的自定义集群的说明与一般[创建自定义集群的说明](/docs/cluster-provisioning/rke-clusters/custom-nodes/#2-create-the-custom-cluster)具有一些特定于Windows的要求。

仅当集群使用Kubernetes v1.15 +和Flannel网络提供程序时才启用Windows支持。

1.从**Global**视图中，单击**Clusters**选项卡，然后单击**添加集群**。

1.单击**自定义**。

1.在**集群名称**文本框中输入集群的名称。

1.在“**Kubernetes版本**下拉菜单中，选择v1.15或更高版本。

1.在**网络驱动**字段中，选择**Flannel**。

1.在**Windows支持**部分中，单击**启用**。

1.可选：启用Windows支持后，您将能够选择Flannel后端。有两个网络选项：[**Host Gateway (L2bridge)**](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw)和[**VXLAN (Overlay)**](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan)。默认选项是**VXLAN (Overlay)**模式。

1. 点击 **下一步**.

> **重要提示：** 对于 <b>Host Gateway (L2bridge)</b>网络，最好对所有节点使用相同的第2层网络。 否则，您需要为其配置路由规则。 有关详细信息，请参阅[有关配置云托管的VM路由的文档。](/docs/cluster-provisioning/rke-clusters/windows-clusters/host-gateway-requirements/#cloud-hosted-vm-routes-configuration)。如果您使用的是亚马逊EC2，Google GCE或Azure VM，还需要[禁用私有IP地址检查](/docs/cluster-provisioning/rke-clusters/windows-clusters/host-gateway-requirements/#disabling-private-ip-address-checks)。

## 3. 在集群上添加节点

本节介绍如何将Linux和Worker节点注册到自定义集群。

#### 添加Linux Master节点

集群中的第一个节点应该是同时具有 **Control Plane** 和 **etcd** 角色的Linux主机。至少必须为此节点启用这两个角色，并且必须先将此节点添加到集群中，然后才能添加Windows主机。

在本节中，我们在Rancher UI上填写表格，以获取自定义命令，以在Linux主节点上安装Rancher代理。然后，我们将复制命令并在Linux主节点上运行该命令，以在集群中注册该节点。

1. 在**节点操作系统**部分中，单击**Linux**。

1. 在**节点角色**部分中，至少选择**etcd**和**控制平面**。我们建议选择所有三个。

1. 可选：如果您单击**显示高级选项**，则可以自定义[Rancher agent](/docs/admin-settings/agent-options/)和[node labels。](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)）

1. 将屏幕上显示的命令复制到剪贴板。

1. SSH进入Linux主机，然后运行复制到剪贴板的命令。

1. 完成配置Linux节点后，选择**完成**。

{{<result_create-cluster>}}

该节点可能需要几分钟才能在您的集群中注册。

#### 添加Linux Worker节点

初始配置自定义集群后，集群仅具有单个Linux主机。 接下来，我们添加另一个Linux`worker`主机，它将用于为您的集群支持 _Rancher集群agent_，_Metrics服务器_，_DNS_和_Ingress_。

1.在**全局**视图中，单击**集群**。

1.转到您创建的自定义集群，然后单击**省略号（...）> 编辑。**

1.向下滚动到**节点操作系统**。 选择 **Linux**。

1.在**自定义节点运行命令**部分中，转到**节点选项**并选择**Worker**角色。

1.将屏幕上显示的命令复制到剪贴板。

1.使用远程终端连接登录到Linux主机。 运行复制到剪贴板的命令。

1.在 **Rancher** 中，单击**保存**。

**结果：** **Worker**角色节点已安装在您的Linux主机上，并且该节点向Rancher注册。 该节点可能需要几分钟才能在您的集群中注册。

> **Note:** Linux节点上的污点(Taint)设置
>
> 对于添加到集群中的每个Linux工作程序节点，以下污点将添加到Linux工作程序节点。 通过将此污点添加到Linux Worker节点，添加到Windows集群的所有工作负载都将自动调度到Windows Worker节点。 如果要将工作负载专门安排到Linux工作节点上，则需要为这些工作负载添加容差。

> | Taint Key      | Taint Value | Taint Effect |
> | -------------- | ----------- | ------------ |
> | `cattle.io/os` | `linux`     | `NoSchedule` |

#### 添加Windows节点

您可以通过编辑集群并选择**Windows**选项将Windows主机添加到自定义集群。

1.在**全局**视图中，单击**集群**。

1.转到您创建的自定义集群，然后单击**省略号（...）>编辑。**

1.向下滚动到**节点操作系统**。选择 **Windows**。注意：您将看到 **worker** 角色是唯一可用的角色。

1.将屏幕上显示的命令复制到剪贴板。

1.使用首选工具，例如[Microsoft远程桌面](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients)。在**命令提示符（CMD）**中运行复制到剪贴板的命令。

1.在**Rancher**中，单击**保存**。

1.可选：如果要向集群添加更多Windows节点，请重复这些说明。

**结果：** Worker **角色已安装在Windows主机上，并且该节点向Rancher注册。该节点可能需要几分钟才能在您的集群中注册。您现在拥有Windows Kubernetes集群。

#### 可选的下一步

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下替代方法来访问集群：

- **使用kubectl CLI访问您的集群：**请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)访问集群 在工作站上使用kubectl。 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher会将您连接到下游集群。 此方法使您可以在没有Rancher UI的情况下管理集群。
- **通过kubectl CLI使用授权的集群端点访问集群：**遵循[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-directly-with-a-downstream-cluster)，而无需通过Rancher服务器进行身份验证即可直接使用kubectl访问您的集群。 我们建议设置此替代方法来访问您的集群，以便在无法连接到Rancher的情况下仍然可以访问该集群。

## Azure中存储类的配置

如果您为节点使用Azure VM，则可以将[Azure文件](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv)用作[存储类](/docs/cluster-admin/volumes-and-storage/#adding-storage-classes)。

为了使Azure平台创建所需的存储资源，请按照下列步骤操作：

1. [配置Azure cloud provider。](/docs/cluster-provisioning/rke-clusters/options/cloud-providers/#azure)

1. 配置`kubectl`连接到您的集群。

1. 复制service account的`ClusterRole`和`ClusterRoleBinding`配置：

```
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: system:azure-cloud-provider
        rules:
        - apiGroups: ['']
          resources: ['secrets']
          verbs:     ['get','create']
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: system:azure-cloud-provider
        roleRef:
          kind: ClusterRole
          apiGroup: rbac.authorization.k8s.io
          name: system:azure-cloud-provider
        subjects:
        - kind: ServiceAccount
          name: persistent-volume-binder
          namespace: kube-system
```

1.  使用以下命令创建相关资源

    ```
    # kubectl create -f <MANIFEST>
    ```
