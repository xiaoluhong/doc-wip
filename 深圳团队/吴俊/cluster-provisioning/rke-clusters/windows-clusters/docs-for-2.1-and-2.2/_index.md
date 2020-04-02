---
title: v2.1.x和v2.2.x Windows 文档（实验性）
---

_可用于 v2.1.0 到 v2.1.9 和 v2.2.0 到 v2.2.3_

本节介绍如何在 Rancher v2.1.x 和 v2.2.x版本中配置 Windows 集群. 如果您正在使用 Rancher v2.3.0 或更高版本, 请参阅新的文档 [v2.3.0 或更高版本](/docs/cluster-provisioning/rke-clusters/windows-clusters/).

当你创建一个 [自定义集群](/docs/cluster-provisioning/custom-clusters/), Rancher 使用 RKE (the Rancher Kubernetes Engine) 在现有的基础设施上配置 Kubernetes 集群.

您可以通过将 Linux 和 Windows 主机混合使用作为集群节点来使用 Rancher 设置自定义 Windows 集群.

> **重要:** 在v2.3之前的Rancher版本中, 对 Windows 节点的支持是实验性的. 因此, 如果您在v2.3之前使用 Rancher,则不建议在生产环境中使用 Windows 节点.

本指南将引导您完成创建包含三个节点的自定义集群:

- 一个 Linux 节点, 用作 Kubernetes controlplane 节点
- 另一个 Linux 节点, 用作 Kubernetes worker, 用于支持集群的入口
- 一个 Windows 节点, 它被分配给 Kubernetes worker 角色并运行您的 Windows 容器

有关 Windows 中支持的 Kubernetes 特性的摘要, 请参阅 [在 Kubernetes 中使用 Windows](https://kubernetes.io/docs/setup/windows/intro-windows-in-kubernetes/).

### 操作系统和容器要求

- 对于使用 Rancher v2.1.x 和 v2.2.x 配置的集群, 容器必须运行在 Windows Server 1809 或以上.
- 您必须在 Windows Server 核心版本1809或更高版本上构建容器, 才能在同一服务器版本上运行这些容器.

### 创建支持Windows的集群

在设置支持Windows节点和容器的自定义集群时，请完成下面的一系列任务.

<!-- TOC -->

- [1. 配置主机](#1-provision-hosts)
- [2. 云托管 VM 网络配置](#2-cloud-hosted-vm-networking-configuration)
- [3. 创建自定义集群](#3-create-the-custom-cluster)
- [4. 添加支持入口的Linux主机](#4-add-linux-host-for-ingress-support)
- [5. 添加 Windows Workers](#5-adding-windows-workers)
- [6. 云托管 VM 路由配置](#6-cloud-hosted-vm-routes-configuration)

<!-- /TOC -->

### 1. 配置主机

开始配置支持Windows的自定义集群, 请准备您的主机服务器. 根据我们的[需求](/docs/installation/requirements/)提供三个节点—两个Linux,一个Windows. 您的主机可以是:

- 云托管虚拟机
- 虚拟化集群中的虚拟机
- 裸机服务器

下表列出了您将分配给每个主机的[Kubernetes角色](/docs/cluster-provisioning/#kubernetes-cluster-node-components), 尽管在配置过程中, 您不会启用这些角色—但我们只是通知您每个节点的用途. 第一个节点是Linux主机, 主要负责管理Kubernetes控制面, 不过,在这个用例中, 我们将在这个节点上安装所有三个角色. 节点2还是一个Linux worker，负责入口支持. 最后, 第三个节点是Windows worker, 它将运行Windows应用程序.

| 节点   | 操作系统                                    | 集群未来的角色 |
| ------ | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 节点1 | Linux (推荐 Ubuntu Server 16.04)             | [Control Plane](/docs/cluster-provisioning/#control-plane-nodes), [etcd](/docs/cluster-provisioning/#etcd), [Worker](/docs/cluster-provisioning/#worker-nodes) |
| 节点2 | Linux (推荐 Ubuntu Server 16.04)             | [Worker](/docs/cluster-provisioning/#worker-nodes) (此节点用于入口支持)                                                                     |
| 节点3 | Windows (Windows Server core version 1809 or 更高版本) | [Worker](/docs/cluster-provisioning/#worker-nodes)                                                                                                             |

#### 要求

- 您可以在[安装部分](/docs/installation/requirements/)中查看Linux和Windows节点的节点要求.
- 虚拟化集群或裸机集群中的所有节点都必须使用第2层网络连接.
- 为了支持[入口](https://kubernetes.io/docs/concepts/services-networking/ingress/), 您的集群必须包含至少一个专门用于worker角色的Linux节点.
- 尽管我们推荐上表中列出的三种节点架构, 但是您可以添加额外的Linux和Windows worker来扩展您的集群以实现冗余.

### 2. 云托管VM网络配置

> **注意:** 此步骤仅适用于托管在云托管虚拟机上的节点. 如果您正在使用虚拟化集群或裸机服务器, 请跳到[创建自定义集群](#3-create-the-custom-cluster).

如果您将节点托管在下面列出的任何云服务上, 则必须在启动时禁用Linux或Windows主机的私有IP地址检查. 要禁用每个节点的此检查, 请按照下面每个服务提供的说明操作.

| 服务    | 禁用私有IP地址检查的说明                                                                                    |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Amazon EC2 | [禁用源/目标检查](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck)                      |
| Google GCE | [为实例启用 IP 转发](https://cloud.google.com/vpc/docs/using-routes#canipforward)                                                         |
| Azure VM   | [启用或禁用IP转发](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface#enable-or-disable-ip-forwarding) |

### 3. 创建自定义集群

要创建支持Windows节点的自定义集群，请按照[使用自定义节点创建集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/#2-create-the-custom-cluster)中的说明操作, 从[2. 创建自定义集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/#2-create-the-custom-cluster)开始. 完成链接说明时, 查找需要对 Windows 节点执行特殊操作的步骤, 这些节点带有注释标记. 这些说明将链接回此处, 通过特殊窗口在下面的子标题中列出.

#### 启用Windows支持选项

在选择**集群选项**时, 将**Windows Support** (**实验**)设置为启用.

选择此选项后, 从[步骤 6](/docs/cluster-provisioning/rke-clusters/custom-nodes/#step-6) 继续[使用自定义节点创建集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/#create-the-custom-cluster).

#### 网络选项

当为支持Windows的集群选择网络提供程序时时, 唯一可用的选项是 Flannel, 因为IP路由需要[host-gw](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw).

如果您的节点由云提供商托管, 并且您希望获得自动化支持, 例如负载平衡器或持久存储设备, 有关配置信息, 请参阅[选择云提供商](/docs/cluster-provisioning/rke-clusters/options/cloud-providers).

#### 节点配置

集群中的第一个节点应该是充当控制面角色的Linux主机. 在将Windows主机添加到集群之前, 必须完成此角色. 至少，节点必须启用此角色, 但我们建议同时启用这三个角色. 下表列出了我们的推荐设置 (稍后我们将为节点2和节点3提供推荐设置).

| 选项                | 设置                               |
| --------------------- | ------------------------------------- |
| 节点操作系统 | Linux                                 |
| 节点的角色            | etcd <br/> Control Plane <br/> Worker |

完成这些配置后, 继续[使用自定义节点创建集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/#create-the-custom-cluster)从[步骤8](/docs/cluster-provisioning/rke-clusters/custom-nodes/#step-8)开始.

### 4. 添加支持入口的Linux主机

在自定义集群的初始设置之后, 集群只有一个Linux主机. 添加另一个Linux主机, 该主机将用于支持集群的入口.

1. 使用内容菜单, 打开在[2. 创建自定义集群](#2-create-the-custom-cluster)中创建的自定义集群.

1. 从主菜单中, 选择**节点**.

1. 单击 **编辑集群**.

1. 向下滚动到**节点操作系统**. 选择**Linux**.

1. 选择 **Worker** 角色.

1. 将屏幕上显示的命令复制到剪贴板.

1. 使用远程终端连接登录到Linux主机. 运行复制到剪贴板的命令.

1. 在 **Rancher**中, 单击 **保存**.

**结果:** worker角色已安装在Linux主机上, 节点注册为Rancher.

### 5. 添加 Windows Workers

您可以通过编辑集群并选择**Windows**选项将Windows主机添加到自定义集群.

1. 从主菜单中选择**节点**.

1. 单击 **编辑集群**.

1. 向下滚动到**节点操作系统**. 选择**Windows**.

1. 选择 **Worker** 角色.

1. 将屏幕上显示的命令复制到剪贴板.

1. 使用您喜欢的工具登录到您的Windows主机, 例如 [Microsoft远程桌面](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients). 在**命令提示符(CMD)**中运行复制到剪贴板的命令.

1. 在 Rancher中, 单击 **保存**.

1. **可选:** 如果您希望向集群中添加更多的Windows节点, 请重复这些指令.

**结果:** worker角色已经安装在您的Windows主机上, 节点注册为Rancher.

### 6. 云托管 VM 路由配置

在 Windows 集群中, 容器使用 Flannel 的`host-gw`模式相互通信I. 在`host-gw`模式下, 同一节点上的所有容器都属于私有子网, 并且通信路由从一个节点上的子网到另一个节点上的子网通过主机网络.

- 当工作节点配置在AWS、虚拟化集群或裸机服务器上时, 确保它们属于相同的第2层子网. 如果节点不属于同一第2层子网，则`host-gw`网络将不起作用.

- 当工作节点配置在GCE或Azure上时, 它们不在相同的第2层子网中. GCE和Azure上的节点属于可路由的第3层网络. 按照下面的说明配置GCE和Azure, 以便云网络知道如何在每个节点上路由主机子网.

要在GCE或Azure上配置主机子网路由,首先运行以下命令来查找每个工作节点上的主机子网:

```bash
kubectl get nodes -o custom-columns=nodeName:.metadata.name,nodeIP:status.addresses[0].address,routeDestination:.spec.podCIDR
```

然后按照每个云提供商的说明为每个节点配置路由规则:

| 服务    | 说明                                                                                                                                                         |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Google GCE | 对于GCE, 为每个节点添加一个静态路由:[添加一个静态路由](https://cloud.google.com/vpc/docs/using-routes#addingroute).                                      |
| Azure VM   | 对于Azure,创建一个路由表:[自定义路由:用户定义](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined). |
