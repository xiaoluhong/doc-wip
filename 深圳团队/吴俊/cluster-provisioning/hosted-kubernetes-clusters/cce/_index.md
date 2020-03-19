---
title: 创建华为CCE集群
---

_自 v2.2.0 起可以_

您可以使用 Rancher 创建一个托管于 Huawei Cloud Container Engine (CCE) 中的集群. Rancher 已经为 CCE 实现并打包了[集群驱动](/docs/admin-settings/drivers/cluster-drivers/), 但是默认情况下, 这个集群驱动是`非活动的`. 为了启动 CCE 集群, 您需要[启用CCE集群驱动程序](/docs/admin-settings/drivers/cluster-drivers/#activating-deactivating-cluster-drivers). 启用集群驱动后，可以开始配置 CCE 集群.

### 华为的预备条件

>**注意**
>部署到 CCE 将会产生费用.

1. 在华为 CCE 门户中查找您的项目ID. 请参阅关于如何[管理项目](https://support.huaweicloud.com/en-us/usermanual-iam/en-us_topic_0066738518.html)的 CCE 文档.

2. 创建一个 [Access Key ID and Secret Access Key](https://support.huaweicloud.com/en-us/usermanual-iam/en-us_topic_0079477318.html).

### 局限性

华为 CCE 服务不支持通过 API 创建具有公共访问权限的集群. 您需要在与要配置的 CCE 集群相同的 VPC 中运行 Rancher.

### 创建一个 CCE 集群

1. 从 **集群** 页面, 单击 **添加集群**.

2. 选择 **华为 CCE**.

3. 输入 **集群名称**.

4. {{< step_create-cluster_member-roles >}}

5. 输入 **项目 ID**, 访问密钥 ID 为 **Access Key** 和 私密访问密钥为 **Secret Key**. 然后单击**下一步: 配置集群**.

6. 填写以下群集配置:

    |设置|描述|
    |---|---|
    | 集群类型 | 您要包含在集群中的类型或节点, `虚拟机` or `裸机`. |
	| 描述 | 集群的描述. |
	| 主版本 | Kubernetes 版本. |
	| 管理规模计数 | 群集的最大节点计数. 选项有50、200和1000. 规模越大，成本就越高. |
	| 高可用性 | 启用主节点高可用性. 启用高可用性的群集将具有更高的成本. |
	| 容器网络模式 | 群集中使用的网络模式。`虚拟机`类型支持`overlay-l2`和`vpc-router`, `裸机`类型支持 `underlay-ipvlan` |
	| 容器网络 CIDR | 群集的网络CIDR. |
	| VPC 名称 | 要部署群集的VPC名称. 如果为空, Rancher 将创建一个. |
	| 子网名称 | 群集将部署到的子网名称. 如果空白, Rancher将创建一个. |
	| 外部服务器 | 该选项是为将来保留的, 我们可以通过 API 启用 CCE 集群公共访问. 目前，它始终处于禁用状态. |
	| 群集标签 | 集群的标签. |
	| 高速子网 | 此选项仅在`BareMetal`类型中受支持. 它要求您为裸机选择一个高速网络的 VPC. |

	**注意:** 如果您要在`cluster.yml`中而不是  Rancher UI中编辑集群, 请注意, 从 Rancher v2.3.0 开始, 群集配置指令必须嵌套在`cluster.yml`中的`rancher_kubernetes_engine_config`指令下. 有关更多信息, 请参阅有关 [Rancher v2.3.0+中的配置文件结构.](/docs/cluster-provisioning/rke-clusters/options/#config-file-structure-in-rancher-v2-3-0)

7. 填写集群的以下节点配置:

    |设置|描述|
	|---|---|
	| 区域 | 集群节点所在的可用区域. |
	| 计费方式 | 集群节点的计费方式. 在`虚拟机`类型中，仅支持`按使用量付费`. 在`裸机`中, 您可以选择`按使用量付费`或`按年/按月`. |
	| 有效期 | 此选项仅在`年/月`计费模式中显示. 这意味着您要为集群节点支付多长时间. |
	| 自动续定 | 此选项仅在`年/月`计费模式中显示. 这意味着集群节点将自动或不自动续定`年度/月度`付款. |
	| 数据卷类型 | 群集节点的数据卷类型. `SATA`, `SSD` or `SAS` for this option. |
	| 数据卷大小 | 群集节点的数据卷大小 |
	| 根卷类型 | 群集节点的根卷类型. `SATA`, `SSD` or `SAS` for this option. |
	| 根卷大小 | 群集节点的根卷大小 |
	| 节点规格 | 群集节点的节点规格. Rancher UI 中的规格列表来自华为云. 它包括所有支持的节点类型. |
	| 节点数 | 群集的节点数 |
	| 节点操作系统 | 群集节点的操作系统. 现在只支持`EulerOS 2.2`和`CentOS 7.4`. |
	| SSH密钥名 | 群集节点的 ssh 密钥 |
	| EIP | 群集节点的公用IP选项。`Disabled`表示集群节点不会绑定公共IP。`Create EIP`意味着集群节点将在配置后绑定一个或多个新创建的 EIP, 并在UI中显示更多选项以设置创建EIP参数.`Select Existed EIP`意味着节点将绑定到您选择的 EIP.  |
	| EIP 数量 | 仅当选择`创建EIP`时才会显示此选项. 这意味着要为节点创建多少 EIP. |
	| EIP 类型 | 仅当选择`创建EIP`时才会显示此选项. 选项是 `5_bgp` 和 `5_sbgp`. |
	| EIP 共享类型 | 仅当选择`创建EIP`时才会显示此选项. The only option is `PER`. |
	| EIP 收费模式 | 仅当选择`创建EIP`时才会显示此选项. 选项是按`带宽`付费和按`流量`付费`. |
	| EIP 带宽大小 | 仅当选择`创建EIP`时才会显示此选项. EIP 的带宽. |
	| 身份验证模式 | 这意味着启用`RBAC`或同时启用`身份验证代理`. 如果选择`authenticing Proxy`, 则还需要用于验证代理的证书. |
	| 节点标签 | 集群节点的标签. |

8. 单击 **创建** 去创建 CCE 集群.

{{< result_create-cluster >}}
