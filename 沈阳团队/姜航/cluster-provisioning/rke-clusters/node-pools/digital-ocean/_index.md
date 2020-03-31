---
title: 创建DigitalOcean集群
---

使用 {{< product >}} 在DigitalOcean中创建Kubernetes集群。

1.  在`集群列表`界面中，点击`添加集群`。

2.  选择**DigitalOcean**。

3.  输入**集群名称**。

4.  {{< step_create-cluster_member-roles >}}

5.  {{< step_create-cluster_cluster-options >}}

6.  {{< step_create-cluster_node-pools >}}

        1.	点击 **添加主机模板**。 注意：从v2.2.0开始，账户访问信息存储为云凭证。 云凭证会存储为Kubernetes的secret。 多个主机模板可以使用相同的云凭证。 您可以使用现有的云凭证或创建新的云凭证。 要创建新的云凭证，请输入**名称**和**账户访问**数据，然后单击**创建**。

        2.  完成 ***Digital Ocean选项** 表单的填写。

        	- **访问令牌** 会存储您的Digital Ocean个人访问令牌. 请参照[DigitalOcean 说明：如何生成个人令牌](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2#how-to-generate-a-personal-access-token)。

        	- **Droplet Options** 设置集群的地理区域和规格。

        4. {{< step_rancher-template >}}

        5. 点击**创建**。

        6. **可选:** 添加其他主机池。

    <br/>

7.  检查您填写的信息以确保填写正确，然后点击 **创建**。

{{< result_create-cluster >}}

## 可选步骤

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下替代方法来访问集群：

- **通过kubectl CLI访问集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)来通过kubectl访问您的集群. 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher会将您连接到下游集群。 此方法使您无需Rancher UI即可管理集群。

- **通过kubectl CLI和授权的集群地址访问您的集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-directly-with-a-downstream-cluster)来通过kubectl直接访问您的集群,而不需要通过Rancher进行认证. 我们建议您设定此方法访问集群，这样在您无法连接Rancher时您仍然能够访问集群。

