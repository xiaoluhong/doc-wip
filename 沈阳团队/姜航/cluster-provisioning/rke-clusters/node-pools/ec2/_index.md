---
title: 创建Amazon EC2集群
---

使用Rancher在Amazon EC2中创建Kubernetes集群。

#### 前提条件

- 创建实例时会使用**AWS EC2 访问密钥**。 请参照[Amazon 文档: 创建访问密钥](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)。

- 为用户的访问密钥**创建IAM策略**。 请参照[Amazon 文档: 创建IAM策略](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-start)。 下面是三个JSON策略的实例:
  - [IAM策略示例](#IAM策略示例)
  - [包含PassRole的IAM策略示例](#包含PassRole的IAM策略示例) (如果您想要使用[Kubernetes Cloud Provider](/docs/cluster-provisioning/rke-clusters/options/cloud-providers) 或IAM实例配置文件)
  - [允许加密EBS卷的IAM策略示例](#允许加密EBS卷的IAM策略示例)
- 为用户添加**IAM 策略许可**。 请参照[Amazon 文档: 为用户添加许可](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html#users_change_permissions-add-console)。

## 创建EC2集群

创建群集的步骤因您的Rancher版本而异。

 tabs 
 tab "Rancher v2.2.0+" 

1. [创建云凭证](#1-创建云凭证)
2. [使用云凭证和EC2信息创建主机模板](#2-使用云凭证和EC2信息创建主机模板)
3. [使用主机模板创建带主机池的集群](#3-使用主机模板创建带主机池的集群)


#### 1. 创建云凭证

1. 在Rancher用户界面中，点击右上角的用户个人资料按钮，然后点击**云凭证**。
1. 点击**添加云凭证**。
1. 为云凭证填写名称。
1. 在**云凭证类型**选项中,选择**Amazon**。
1. 在**区域**选项中, 选择您将要创建主机的AWS区域。
1. 填写您的AWS EC2 访问密钥：**Access Key** 和**Secret Key**。
1. 点击**创建**。

**结果：** 您已经创建了用于配置群集中节点的云凭证。 您可以将这些云凭证用于其他主机模板或其他集群。

#### 2. 使用云凭证和EC2信息创建主机模板

使用[EC2管理控制台](https://aws.amazon.com/ec2)中可用的信息来完成以下内容。

1. 在Rancher用户界面中，点击右上角的用户个人资料按钮，然后点击**主机模板**。
1. 点击**添加模板**。
1. 在**区域**选项中, 选择您创建云凭证相同的区域。
1. 在**云凭证**选项中, 选择您创建的云凭证。
1. 点击**下一步：认证 & 设置节点**
1. 选择您的集群的可用区和网络设置。 点击 **下一步: 选择安全组**。
1. 选择默认的安全组或配置一个新的安全组。 请根据文档[Amazon EC2使用主机驱动时的安全组](/docs/cluster-provisioning/node-requirements/#security-group-for-nodes-on-aws-ec2)来查看`rancher-nodes`安全组创建的规则。 然后点击**下一步: 设置实例选项**。
1. 配置将要创建的实例。确保为选择的AMI配置正确的**SSH用户**。

> 如果您需要设定 <b>IAM 实例配置名称</b> (not ARN), 例如当您需要使用[Kubernetes Cloud Provider](/docs/cluster-provisioning/rke-clusters/options/cloud-providers), 您需要额外的许可在您的策略中。请参照[包含PassRole的IAM策略示例](#包含PassRole的IAM策略示例)。

可选：在主机模板的**引擎选项**部分中，您可以配置Docker守护程序。 您可能需要指定Docker版本或Docker镜像仓库地址。

#### 3. 使用主机模板创建带主机池的集群

{{< step_create-cluster_node-pools >}}

1. 在**集群列表**界面中，点击**添加集群**。

1. 选择**Amazon EC2**。

1. 填写**集群名称**。

1. 为每个Kubernetes角色创建一个主机池。 对于每个主机池，选择您创建的主机模板。

1. 点击**添加成员**来添加能够访问集群的用户。

1. 使用**角色**下拉菜单来为每个用户设定权限。

1. 通过**集群选项**来选择Kubernetes版本, 网络插件及是否开启网络隔离。 请参照[选择Cloud Providers](/docs/cluster-provisioning/rke-clusters/options/cloud-providers/) 来设置Kubernetes Cloud Provider。

1. 点击**创建**。

{{< result_create-cluster >}}
 /tab 
 tab "Rancher v2.2.0+之前的版本" 

1. 在**集群列表**界面中，点击**添加集群**。

1. 选择**Amazon EC2**。

1. 填写**集群名称**。

1. {{< step_create-cluster_member-roles >}}

1. {{< step_create-cluster_cluster-options >}} 请参照[选择Cloud Providers](/docs/cluster-provisioning/rke-clusters/options/cloud-providers/) 来设置Kubernetes Cloud Provider。

1. {{< step_create-cluster_node-pools >}}

1. 点击**添加主机模板**。

1. 使用[EC2管理控制台](https://aws.amazon.com/ec2)中可用的信息来完成以下内容。


    - **账户许可** 选项用于配置创建主机的区域及云凭证。关于如何创建访问密钥和权限，请参照[前提条件](#prerequisites)。
    - **区域和网络** 选项用于配置您的集群可用区和网络设置。
    - **安全组** 选项用于配置您的主机的安全组。 请参照 [Amazon EC2使用主机驱动的安全组](/docs/cluster-provisioning/node-requirements/#security-group-for-nodes-on-aws-ec2) 来查看`rancher-nodes`安全组中创建了哪些规则。
    - **实例** 选项用于配置将要创建的实例。确保为选择的AMI配置正确的**SSH用户**。

<br /><br />
如果您需要设定 **IAM 实例配置名称** (not ARN), 例如当您需要使用[Kubernetes Cloud Provider](/docs/cluster-provisioning/rke-clusters/options/cloud-providers), 您需要额外的许可在您的策略中。请参照[包含PassRole的IAM策略示例](#包含PassRole的IAM策略示例)。

1. {{< step_rancher-template >}}
1. 点击**创建**。
1. **可选：** 添加其他主机池。
1. 检查您填写的信息以确保填写正确，然后点击 **创建**。

{{< result_create-cluster >}}
 /tab 
 /tabs 

#### 可选步骤

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下替代方法来访问集群：

- **通过kubectl CLI访问集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)来通过kubectl访问您的集群. 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher会将您连接到下游集群。 此方法使您无需Rancher UI即可管理集群。

- **通过kubectl CLI和授权的集群地址访问您的集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-directly-with-a-downstream-cluster)来通过kubectl直接访问您的集群,而不需要通过Rancher进行认证. 我们建议您设定此方法访问集群，这样在您无法连接Rancher时您仍然能够访问集群。

#### IAM策略示例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:Describe*",
        "ec2:ImportKeyPair",
        "ec2:CreateKeyPair",
        "ec2:CreateSecurityGroup",
        "ec2:CreateTags",
        "ec2:DeleteKeyPair"
      ],
      "Resource": "*"
    },
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": ["ec2:RunInstances"],
      "Resource": [
        "arn:aws:ec2:REGION::image/ami-*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:instance/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:placement-group/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:volume/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:subnet/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:key-pair/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:network-interface/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:security-group/*"
      ]
    },
    {
      "Sid": "VisualEditor2",
      "Effect": "Allow",
      "Action": [
        "ec2:RebootInstances",
        "ec2:TerminateInstances",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:instance/*"
    }
  ]
}
```

#### 包含PassRole的IAM策略示例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "ec2:AuthorizeSecurityGroupIngress",
        "ec2:Describe*",
        "ec2:ImportKeyPair",
        "ec2:CreateKeyPair",
        "ec2:CreateSecurityGroup",
        "ec2:CreateTags",
        "ec2:DeleteKeyPair"
      ],
      "Resource": "*"
    },
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": ["iam:PassRole", "ec2:RunInstances"],
      "Resource": [
        "arn:aws:ec2:REGION::image/ami-*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:instance/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:placement-group/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:volume/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:subnet/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:key-pair/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:network-interface/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:security-group/*",
        "arn:aws:iam::AWS_ACCOUNT_ID:role/YOUR_ROLE_NAME"
      ]
    },
    {
      "Sid": "VisualEditor2",
      "Effect": "Allow",
      "Action": [
        "ec2:RebootInstances",
        "ec2:TerminateInstances",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:instance/*"
    }
  ]
}
```

#### 允许加密EBS卷的IAM策略示例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKeyWithoutPlaintext",
        "kms:Encrypt",
        "kms:DescribeKey",
        "kms:CreateGrant",
        "ec2:DetachVolume",
        "ec2:AttachVolume",
        "ec2:DeleteSnapshot",
        "ec2:DeleteTags",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:DeleteVolume",
        "ec2:CreateSnapshot"
      ],
      "Resource": [
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:volume/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:instance/*",
        "arn:aws:ec2:REGION:AWS_ACCOUNT_ID:snapshot/*",
        "arn:aws:kms:REGION:AWS_ACCOUNT_ID:key/KMS_KEY_ID"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeTags",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots"
      ],
      "Resource": "*"
    }
  ]
}
```
