---
title: 创建 AKS 集群
---

您可以使用 Rancher 创建一个托管在 Microsoft Azure Kubernetes 服务 (AKS) 中的集群.

### Microsoft Azure 的预备条件

> **注意**
> 部署到 AKS 将会产生费用.

要与 Azure API 交互, AKS 集群需要 Azure 活动目录 （AD） 服务主体. 需要服务主体来动态创建和管理其他 Azure 资源, 它为您的集群提供了与AKS通信的凭证. 有关服务主体的详细信息, 请参阅 [AKS 文档](https：//docs.microsoft.com/en-us/azure/aks/kubernetes-服务主体).

在创建服务主体之前, 您需要从[Microsoft Azure网站](https://portal.azure.com)获取以下信息:

- 您的订阅 ID
- 您的租户 ID
- 应用 ID（也称为客户端 ID）
- 客户密钥
- 资源组

下面的部分描述了如何使用Azure命令行工具或Azure网站设置这些先决条件.

#### 使用 Azure 命令行工具设置服务主体

您可以通过运行以下命令来创建服务主体:

```
az ad sp create-for-rbac --skip-assignment
```

结果应显示有关新服务主体的信息：

```
{
  "appId": "xxxx--xxx",
  "displayName": "<SERVICE-PRINCIPAL-NAME>",
  "name": "http://<SERVICE-PRINCIPAL-NAME>",
  "password": "<SECRET>",
  "tenant": "<TENANT NAME>"
}
```

您还需要向服务主体添加角色, 以便它具有与 AKS API 通信的权限. 它还需要访问来创建和列出虚拟网络.

下面是将参与者角色分配给服务主体的示例命令. 参与者可以管理 AKS 上的任何内容, 但不能授予其他人访问权限：

```
az role assignment create \
  --assignee $appId \
  --scope /subscriptions/$<SUBSCRIPTION-ID>/resourceGroups/$<GROUP> \
  --role Contributor
```

您还可以创建服务主体, 并通过将两个命令合并为一个命令来授予其参与者权限. 在此命令中, 作用域需要提供 Azure 资源的完整路径：

```
az ad sp create-for-rbac \
  --scope /subscriptions/$<SUBSCRIPTION-ID>/resourceGroups/$<GROUP> \
  --role Contributor
```

#### 从 Azure 网站设置服务主体

还可以按照这些指示设置服务主体, 并从 Azure 网站授予其基于角色的访问权限.

1. 跳转到 Microsoft Azure 网站[主页](https://portal.azure.com).

1. 单击 **Azure Active Directory.**

1. 单击 **应用注册.**

1. 单击 **新注册.**

1. 输入名称. 这将是服务主体的名称.

1. 可选:选择哪些帐户可以使用服务主体.

1. 单击 **注册.**

1. 您现在应该看到服务主体的名称 **Azure Active Directory > 应用注册.**

1. 单击服务主体的名称. 请注意租户 ID 和应用程序 ID（也称为应用 ID 或客户端 ID）, 以便在配置 AKS 集群时使用它. 然后单击 **证书 & 密钥.**

1. 单击 **New client secret.**

1. 输入一个简短的描述，选择一个过期时间，然后单击**添加**. 注意客户端密钥, 以便在配置AKS集群时使用它.

**结果:** 您已经创建了一个服务主体, 您应该能够看到它在**应用注册**下的**Azure活动目录**中列出. 您仍然需要授予服务主体对 AKS 的访问权限.

要提供对服务主体的基于角色的访问权限，,

1. 点击左边导航栏中的 **All Services** . 然后点击 **订阅**.

1. 单击要与 Kubernetes 集群关联的订阅的名称。记下订阅 ID，以便在预配 AKS 集群时使用它。

1. 单击 **访问控制 (IAM).**

1. 在 **添加角色分配** 部分, 点击 **添加**.

1. 在 **角色** 字段中, 选择可以访问 AKS 的角色. 例如，您可以使用 **参与者** 角色，该角色具有管理所有内容的权限, 但授予其他用户访问权限除外.

1. 在 **分配访问权限** 字段中, 选择 **Azure AD 用户, 组, or 访问主体.**

1. 在 **选择** 字段中, 选择服务主体的名称, 然后单击 **保存**.

**结果:** 您的服务主体现在可以访问AKS.

### 创建 AKS 集群

使用 Rancher 来设置和配置您的 Kubernetes 集群.

1. 在 **集群** 页, 单击 **添加集群**.

1. 选择 **Azure Kubernetes Service**.

1. 输入 **集群名称**.

1. {{< step_create-cluster_member-roles >}}

1. 使用订阅 ID、租户 ID、应用 ID 和客户端密钥授予集群对 AKS 的访问权限. 如果您没有所有这些信息，则可以使用以下说明检索这些信息：

- **应用 ID and ID、租户 ID:** 要获取应用 ID 和租户 ID, 可以转到 Azure 网站, 然后单击**Azure Active Directory**, 然后单击 click **应用注册,** 然后单击服务主体的名称. 应用 ID 和租户 ID 都位于应用注册详细信息页上.
- **客户端密钥:** 如果在创建服务主体时未复制客户端密钥, 在应用注册详细信息页面, 则可以获取新密钥, 然后单击**证书 & 机密**，然后单击**新客户端密钥.**。
- **订阅 ID:** 您可以从**All services > Subscriptions.**获取门户中可用的订阅ID.

1.  {{< step_create-cluster_cluster-options >}}

1.  使用您的服务主体的输出完成**帐户访问**. 此信息用于 Azure 进行身份验证.

1.  使用**Nodes**来提供集群中的节点, 并选择一个地理区域.

      [Microsoft 文档:如何创建和使用SSH公钥和私钥对](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys)

    <br/>

1.  单击**创建**.
    <br/>
1.  检查您的选项来确认它们是正确的. 然后单击**创建**.

{{< result_create-cluster >}}
