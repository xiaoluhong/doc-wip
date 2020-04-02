---
title: 创建Azure集群
---

使用 {{< product >}} 在Azure中创建Kubernetes集群。

1.  在`集群列表`界面中，点击`添加集群`。

2.  选择**Azure**。

3.  输入**集群名称**。

4.  {{< step_create-cluster_member-roles >}}

5.  {{< step_create-cluster_cluster-options >}}

6.  {{< step_create-cluster_node-pools >}}

        1.	点击 **添加主机模板**。

        2.	完成 **Azure选项** 表单的填写。

        	- **账户访问**用于存储您的账户信息，以便进行Azure的身份验证。 注意：从v2.2.0开始，账户访问信息存储为云凭证。 云凭证会存储为Kubernetes的secret。 多个主机模板可以使用相同的云凭证。 您可以使用现有的云凭证或创建新的云凭证。 要创建新的云凭证，请输入**名称**和**账户访问**数据，然后单击**创建**。

        	- **区域**用于设置您的托管集群的地理区域以及其他位置元数据。

        	- **网络**用于设置您的集群的网络配置。

        	- **实例**用于自定义您的虚拟机设定。

        3. {{< step_rancher-template >}}

        4. 点击**创建**。

        5. **可选:** 添加其他主机池。

    <br />

7.  检查您填写的信息以确保填写正确，然后点击 **创建**.

{{< result_create-cluster >}}

## 可选步骤

创建集群后，您可以通过Rancher UI访问它。 作为最佳实践，我们建议设置以下替代方法来访问集群：

- **通过kubectl CLI访问集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#accessing-clusters-with-kubectl-on-your-workstation)来通过kubectl访问您的集群. 在这种情况下，您将通过Rancher服务器的身份验证代理进行身份验证，然后Rancher会将您连接到下游集群。 此方法使您无需Rancher UI即可管理集群。

- **通过kubectl CLI和授权的集群地址访问您的集群:** 请按照[这些步骤](/docs/cluster-admin/cluster-access/kubectl/#authenticating-directly-with-a-downstream-cluster)来通过kubectl直接访问您的集群,而不需要通过Rancher进行认证. 我们建议您设定此方法访问集群，这样在您无法连接Rancher时您仍然能够访问集群。
