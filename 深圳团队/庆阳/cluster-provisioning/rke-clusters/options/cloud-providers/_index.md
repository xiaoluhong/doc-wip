---
标题: 设置云提供商
---

_云提供商_ 是Kubernetes中的一个模块，它提供了一个用于管理节点、负载均衡和网络路由的接口。 有关更多信息，请参阅[关于云提供商的官方Kubernetes文档。](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)

当在Rancher中设置云提供商时，如果您使用的云提供商支持这种自动化，Rancher服务器可以在启动Kubernetes定义时自动配置新的节点，负载均衡器或持久存储设备。

- [云提供商选项](#cloud-provider-options)
- [设置Amazon云提供商](#setting-up-the-amazon-cloud-provider)
- [设置Azure云提供程序](#setting-up-the-azure-cloud-provider)

### 云提供商选项

默认情况下，**云提供商**选项设置为"无"。 支持的云提供商包括:

- [Amazon](#setting-up-the-amazon-cloud-provider)
- [Azure](#setting-up-the-azure-cloud-provider)

`自定义` 云提供商可以用来配置任意的[Kubernetes云提供商](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/).

对于自定义云提供商选项，您可以参考[RKE文档]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/)关于如何为您的特定云提供商编辑yaml文件。有一些特定的云提供商具有更详细的配置 :

- [vSphere]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/vsphere/)
- [Openstack]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/openstack/)

> **警告:** 如果您配置了不符合先决条件的节点的云提供商集群，则集群将无法正确配置。 下面列出了支持的云提供商的先决条件。

### 设置Amazon云提供商

使用`Amazon`云提供商时，您可以利用以下功能:

- **负载均衡:** 在**Port Mapping**中选择`Layer-4 Load Balancer`时或在启动带有`type: LoadBalancer`的`Service`时启动AWS Elastic Load Balancer(ELB)。
- **持久卷**: 允许您将AWS Elastic Block Store(EBS)用于持久卷。

请参阅[云提供商-aws自述文件](https://github.com/kubernetes/cloud-provider-aws/blob/master/README.md)有关Amazon云提供商的所有信息。

设置Amazon云提供商,

1. [创建IAM角色并附加到实例](#1-create-an-iam-role-and-attach-to-the-instances)
2. [配置ClusterID](#2-configure-the-clusterid)

#### 1. 创建IAM角色并附加到实例

添加到集群的所有节点必须能够与EC2交互，以便它们可以创建和删除资源。 您可以使用附加到实例的IAM角色来启用此交互。 请参阅[Amazon文档：创建IAM Role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#create-iam-role)如何创建IAM角色。 有两个示例策略:

- 第一个策略是针对具有`controlplane`角色的节点。 这些节点必须能够创建/删除EC2资源。 下面的IAM策略是一个例子，请删除您的用例的任何不需要的权限。
- 第二个策略是针对具有`etcd`或`worker`角色的节点。 这些节点只需要能够从EC2检索信息。

创建[Amazon EC2集群](/docs/cluster-provisioning/rke-clusters/node-pools/ec2/#create-the-amazon-ec2-cluster)时，必须在创建**节点模板**时填写创建的IAM角色的**Iam实例配置文件名称**（而不是ARN）。

创建[自定义集群](/docs/cluster-provisioning/custom-clusters/)时，必须手动将IAM角色附加到实例。

具有`controlplane`角色的节点的IAM策略:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeTags",
        "ec2:DescribeInstances",
        "ec2:DescribeRegions",
        "ec2:DescribeRouteTables",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeSubnets",
        "ec2:DescribeVolumes",
        "ec2:CreateSecurityGroup",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:ModifyInstanceAttribute",
        "ec2:ModifyVolume",
        "ec2:AttachVolume",
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:CreateRoute",
        "ec2:DeleteRoute",
        "ec2:DeleteSecurityGroup",
        "ec2:DeleteVolume",
        "ec2:DetachVolume",
        "ec2:RevokeSecurityGroupIngress",
        "ec2:DescribeVpcs",
        "elasticloadbalancing:AddTags",
        "elasticloadbalancing:AttachLoadBalancerToSubnets",
        "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
        "elasticloadbalancing:CreateLoadBalancer",
        "elasticloadbalancing:CreateLoadBalancerPolicy",
        "elasticloadbalancing:CreateLoadBalancerListeners",
        "elasticloadbalancing:ConfigureHealthCheck",
        "elasticloadbalancing:DeleteLoadBalancer",
        "elasticloadbalancing:DeleteLoadBalancerListeners",
        "elasticloadbalancing:DescribeLoadBalancers",
        "elasticloadbalancing:DescribeLoadBalancerAttributes",
        "elasticloadbalancing:DetachLoadBalancerFromSubnets",
        "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
        "elasticloadbalancing:ModifyLoadBalancerAttributes",
        "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
        "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
        "elasticloadbalancing:AddTags",
        "elasticloadbalancing:CreateListener",
        "elasticloadbalancing:CreateTargetGroup",
        "elasticloadbalancing:DeleteListener",
        "elasticloadbalancing:DeleteTargetGroup",
        "elasticloadbalancing:DescribeListeners",
        "elasticloadbalancing:DescribeLoadBalancerPolicies",
        "elasticloadbalancing:DescribeTargetGroups",
        "elasticloadbalancing:DescribeTargetHealth",
        "elasticloadbalancing:ModifyListener",
        "elasticloadbalancing:ModifyTargetGroup",
        "elasticloadbalancing:RegisterTargets",
        "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
        "iam:CreateServiceLinkedRole",
        "kms:DescribeKey"
      ],
      "Resource": ["*"]
    }
  ]
}
```

具有`etcd`或`worker`角色的节点的IAM策略:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeRegions",
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:GetRepositoryPolicy",
        "ecr:DescribeRepositories",
        "ecr:ListImages",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```

#### 2. 配置ClusterID

以下资源需要标记为`ClusterID`:

- **节点**: 在Rancher中添加的所有主机。
- **子网**: 用于集群的子网
- **安全组**: 用于集群的安全组。

> **注意**不要标记多个安全组。 创建弹性负载均衡器(ELB)时，标记多个组会产生错误。

当您创建[Amazon EC2 Cluster](/docs/cluster-provisioning/rke-clusters/node-pools/ec2/#create-the-amazon-ec2-cluster), `ClusterID`为创建的节点自动配置。 其他资源仍然需要手动标记。

使用以下标记:

**Key** = `kubernetes.io/cluster/CLUSTERID` **Value** = `owned`

`CLUSTERID`可以是您喜欢的任何字符串，只要它在所有标签集中相等。

将标记的值设置为`owned`会告诉集群，带有此标记的所有资源都由此集群拥有和管理。 如果在集群之间共享资源，则可以将标记更改为:

**Key** = `kubernetes.io/cluster/CLUSTERID` **Value** = `shared`.

#### 使用Amazon Elastic Container Registry (ECR)

当[创建IAM角色并附加到实例](#1-create-an-iam-role-and-attach-to-the-instances)中提到的IAM配置文件附加到实例时，kubelet组件能够自动获取ECR凭据。 当使用早于v1.15.0的Kubernetes版本时，需要在集群中配置Amazon云提供商。 从Kubernetes版本v1.15.0开始，kubelet可以获取ECR凭据，而无需在集群中配置Amazon云提供商。

### 设置Azure云提供程序

使用"Azure"云提供程序时，您可以利用以下功能:

- **负载均衡:** 在特定网络安全组中启动Azure负载平衡器。

- **持久卷:** 支持使用具有标准和高级存储帐户的Azure Blob磁盘和Azure托管磁盘。

- **网络存储:** 通过CIFS安装支持Azure文件。

Azure订阅不支持以下帐户类型:

- 单租户帐户（即没有订阅的帐户）。
- 多订阅帐户。

要设置Azure云提供程序需要配置以下凭据:

1. [设置Azure租户ID](#1-set-up-the-azure-tenant-id)
2. [设置Azure客户端ID和Azure客户端密钥](#2-set-up-the-azure-client-id-and-azure-client-secret)
3. [配置App注册许可](#3-configure-app-registration-permissions)
4. [设置Azure网络安全组名称](#4-set-up-azure-network-security-group-name)

#### 1. 设置Azure租户ID

浏览[Azure控制台](https://portal.azure.com), 登录并跳转**Azure Active Directory** 然后选择 **属性**. 您的**目录 ID** 是您的 **租户ID** (tenantID).

如果您想使用Azure CLI，您可以运行命令`az account show`来获取信息。

#### 2. 设置Azure客户端ID和Azure客户端密钥

浏览[Azure portal](https://portal.azure.com), 登录并按照以下步骤创建**应用注册**中的**Azure客户端ID** (aadClientId)和**Azure客户端密钥** (aadClientSecret).

1. 选择**Azure Active Directory**.
1. 选择**应用注册**.
1. 选择**注册应用程序**.
1. 输入**名称**, 在**Application类型**中选择`Web app / API`和在这种情况下，可以是任何东西的**Sign-on URL**。
1. 选择**创建**.

在**应用注册**视图, 您应该看到您创建的应用程序注册。 列中显示的值**应用ID**便是您需要的**Azure客户端ID**.

下一步是生成**Azure客户端密钥**:

1. 打开您创建的应用程序注册。
1. 在**设置**视图中，打开**键**。
1. 输入**密钥说明**，选择到期时间并选择**保存**。
1. **Value**列中显示的生成值是您需要用作**Azure客户端密钥**的值。 此值将只显示一次。

#### 3. 配置应用程序注册权限

您需要做的最后一件事是为您的应用程序注册分配适当的权限

1. 转到**更多服务**，搜索**订阅**并打开它。
1. 打开**访问控制(IAM)**。
1. 选择**添加**。
1. 对于**角色**，选择`参与者`。
1. 对于**选择**，选择您创建的应用程序注册名称。
1. 选择**保存**。

#### 4. 设置Azure网络安全组名称

需要自定义Azure网络安全组（securityGroupName）才能允许Azure负载平衡器工作。

如果使用Rancher Machine Azure驱动程序设置主机，则需要手动编辑它们以将它们分配给此网络安全组。

在设置过程中，您应该已将自定义主机分配给此网络安全组。

只有预期为负载均衡后台的主机需要在此组中。
