---
title: 创建 EKS 集群
---

Amazon EKS 为 Kubernetes 集群提供了一个托管控制平面. Amazon EKS 跨多个可用性区域运行 Kubernetes 控制平面实例,以确保高可用性. Rancher 为管理和部署在 Amazon EKS 中运行的 Kubernetes 集群提供了直观的用户界面. 通过本指南, 您将使用 Rancher 在您的 AWS 帐户中快速轻松地启动 Amazon EKS Kubernetes 集群. 有关 Amazon EKS 的更多信息, 参考[文档](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html).

### Amazon Web 服务的预备条件

> **注意**
> 部署Amazon AWS将产生费用. 了解更多信息, 请参阅[EKS定价页](https://aws.amazon.com/eks/pricing/).

要在EKS上建立集群,需要建立Amazon VPC（虚拟私有云. 您还需要确保用于创建EKS集群的帐户具有适当的权限. 有关详细信息，请参阅官方文档 [Amazon EKS 预备条件](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#eks-prereqs).

#### Amazon VPC

您需要建立一个Amazon VPC来启动EKS集群. VPC使您能够将AWS资源启动到您定义的虚拟网络中. 了解更多信息, 参考 [教程: 为Amazon EKS集群创建一个包含公共和私有子网的VPC](https://docs.aws.amazon.com/eks/latest/userguide/create-public-private-vpc.html).

#### IAM 策略

Rancher需要访问您的AWS帐户, 以便在Amazon EKS中提供和管理您的Kubernetes集群.您需要在AWS帐户中为Rancher创建一个用户,并定义该用户可以访问的内容.

1. 请按照以下步骤创建具有程序访问权限的用户:[此处](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html).

2. 下一步, 创建 IAM 策略,定义该用户在 AWS 账户中有权访问的内容. 请务必仅授予此用户在帐户中的最小访问权限. 按照 [此处](https://docs.aws.amazon.com/eks/latest/userguide/EKS_IAM_user_policies.html) 的步骤创建 IAM 策略并将其附加到用户.

3. 最后,按照 [此处](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)的步骤为此用户创建访问密钥和密钥.

> **注意:** 定期轮换访问权限和密钥非常重要. 参考[文档](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#rotating_access_keys_console)了解更多信息.

有关EKS的IAM策略的更多详细信息，请参阅官方[有关 Amazon EKS IAM 策略、角色和权限的文档](https://docs.aws.amazon.com/eks/latest/userguide/IAM_policies.html).

### 结构

下图展示了Rancher 2.x的高级架构. 该图描述了一个Rancher服务器安装, 它管理两个Kubernetes集群:一个由RKE创建, 另一个由EKS创建.

![具有EKS托管集群的Rancher体系结构](/img/rancher/rancher-architecture.svg)

### 创建 EKS 集群

使用Rancher设置和配置Kubernetes集群.

1.  从 **集群** 页面, 单击 **添加集群**.

1.  选择 **Amazon EKS**.

1.  输入 **集群名称**.

1.  {{< step_create-cluster_member-roles >}}

1.  为 EKS 集群配置**账户访问**. 使用[2. 创建 Access Key 和 Secret Key](#prerequisites-in-amazon-web-services)中获取的信息填写每个下拉列表和字段.

    | 设置    | 描述                                                                                                          |
    | ---------- | -------------------------------------------------------------------------------------------------------------------- |
    | 区域     | 从下拉列表中选择要在其中创建集群的地理区域。.                                    |
    | Access Key | 输入在[2. 创建 Access Key 和 Secret Key](#2-create-access-key-and-secret-key)中创建的访问密钥. |
    | Secret Key | 输入在[2. 创建 Access Key 和 Secret Key](#2-create-access-key-and-secret-key)中创建的密钥. |

1.  单击 **下一步: 选择服务角色**. 然后选择[服务角色](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html).

    | 服务角色                                    | 描述                                                                                                                                                                                                                                                                                                             |
    | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | Standard: Rancher 生成的服务角色        | 如果选择此角色, Rancher 会自动添加一个服务角色以用于集群.                                                                                                                                                                                                                            |
    | Custom: 从现有服务角色中选择 | 如果您选择此角色, Rancher 允许您从已经在 AWS 中创建的服务角色中进行选择. 有关在 AWS 中创建自定义服务角色的详细信息, 参考 [Amazon documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#create-service-linked-role). |

1.  单击 **下一步: 选择 VPC 和 Subnet**.

1.  为**工作节点的公用IP**选择一个选项. 您对此选项的选择决定了**VPC 和 Subnet**可用的选项.

    | 选择               | 描述                                                                                                                                                                                                                                                                                                       |
    | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | Yes                  | 当您的集群节点被配置时，它们被分配了一个私有和公共IP地址.                                                                                                                                                                                                                 |
    | No: 仅限私有 IP | 当您的集群节点被配置时，它们只被分配了一个私有IP地址.<br/><br/>如果您选择此选项, 您还必须选择一个**VPC & Subnet**来允许您的实例访问 internet. 此访问是必需的,以便您的工作节点可以连接到Kubernetes控制平面. |

1.  现在选择 **VPC & Subnet**. 有关更多信息, 请参阅AWS文档中的[Cluster VPC Considerations](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html). 根据上一步的选择，按照下面的一组说明进行操作.

    - [ Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
    - [VPCs and Subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)

    accordion id="yes" label="Public IP for Worker Nodes—Yes" 

    如果您选择将一个公共IP地址分配给集群的工作节点，您可以在Rancher自动生成的VPC之间进行选择 (例如,**Standard: Rancher生成的VPC and Subnets**), 或者您已经使用AWS创建的VPC (例如, **Custom: 选择您现有的VPC and Subnets**). 选择最适合您的用例的选项.

1.  选择 **VPC and Subnet** 选项.

    | 操作                                           | 描述                                                                                                                                                                                                                                                             |
    | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | Standard: Rancher生成的VPC and Subnets       | 在配置集群时,Rancher会生成一个新的VPC和子网.                                                                                                                                                                                                |
    | Custom: 选择您现有的VPC and Subnets | 设置集群时, Rancher将使用您已经在[AWS中创建](https://docs.aws.amazon.com/vpc/latest/userguide/getting-started-ipv4.html)的VPC和子网来配置您的节点. 如果选择此选项,请完成以下剩余步骤. |

1.  如果您使用 **Custom:  选择您现有的VPC and Subnets**:

    (如果您选择 **Standard**, 跳到 [步骤 11](#select-instance-options))

    1. 确保 **Custom: 选择您现有的VPC and Subnets** 已经选择.

    1. 在显示的下拉列表中,选择一个VPC.

    1. 单击 **下一步: 选择 Subnets**. 然后选择其中一个显示的**Subnets**.

    1. 单击 **下一步: 选择安全组**.
        /accordion 
        accordion id="no" label="Public IP for Worker Nodes—No: Private IPs only" 

    如果您选择此选项, 您还必须选择允许您的实例访问internet的**VPC & Subnet**. 这个访问是必需的,这样您的工作节点就可以连接到Kubernetes控制平面. 遵循以下步骤.

> **提示:** 仅使用专用IP地址时, 可以通过创建由两个子网（一个专用集和一个公用集）构成的VPC来为节点提供internet访问. 私有集应该将其路由表配置为指向公共集中的NAT. 有关从专用子网路由流量的详细信息. 有关私有子网路由流量的更多信息, 请查看 [官方AWS文档](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html).

1. 从显示的下拉列表中,选择专有网络 VPC.

1. 单击 **下一步: 选择 Subnets**. 然后选择其中一个显示的**Subnets**.

1. 单击 **下一步: 选择安全组**.
     /accordion 

1. <a id="security-group"></a>选择**安全组**. 请参阅下面的文档,了解如何创建一个.

   Amazon 文档:

   - [集群安全组注意事项](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html)
   - [VPC的安全组](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
   - [创建安全组](https://docs.aws.amazon.com/vpc/latest/userguide/getting-started-ipv4.html#getting-started-create-security-group)

1. <a id="select-instance-options"></a>单击 **选择实例选项**, 然后编辑可用的节点选项. 工作节点的实例类型和大小影响每个工作节点将有多少IP地址可用. 参考这个 [文档](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI) 了解更多信息.

   | 选项              | 描述                                                                                                                                                                                                                                                                                                                 |
   | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | 实例类型       | 为正在配置的实例选择[硬件规格](https://aws.amazon.com/ec2/instance-types/).                                                                                                                                                                                                               |
   | Custom AMI Override | 如果您想使用自定义的 [Amazon Machine Image](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html#creating-an-ami) (AMI), 请在这里指定它, Rancher 将会为您选择的 EKS 版本使用 [EKS-optimized AMI](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html). |
   | Minimum ASG Size    | 通过[Amazon Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)控制,在低流量期间集群将扩展到的最小的实例数.                                                                                                     |
   | Maximum ASG Size    | 通过[Amazon Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)控制,在低流量期间集群将扩展到的最大的实例数.                                                                                                    |
   | 用户数据           | 可以传递自定义命令来执行自动配置任务 **警告：修改此命令可能会导致节点无法加入集群** _注意: 从v2.2.0起提供_                                                                                                                                 |

1. 单击 **创建**.

{{< result_create-cluster >}}

### 故障排除

对于您的Amazon EKS Kubernetes集群的任何问题或故障排除细节, 请参考 [文档](https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html).

### AWS服务事件

查找任何AWS服务事件的信息, 请参考 [此页](https://status.aws.amazon.com/).

### 安全性和合规性

对于您的Amazon EKS Kubernetes集群的安全性和遵从性的更多信息, 请参考 [文档](https://docs.aws.amazon.com/eks/latest/userguide/shared-responsibilty.html).

### 教程

AWS开源博客上的这篇[教程](https://aws.amazon.com/blogs/opensource/managing-eks-clusters-rancher/) 将指导您如何使用Rancher设置一个EKS集群, 部署一个可公开访问的应用程序来测试集群, 并部署一个示例项目, 使用其他开源软件如Grafana和infloxdb的组合来跟踪实时地理空间数据.
