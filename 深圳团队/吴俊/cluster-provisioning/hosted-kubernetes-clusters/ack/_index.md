---
title: 创建阿里云 ACK 集群
---

_从v2.2.0_开始可用_

您可以使用 Rancher 创建一个托管于 Alibaba Cloud Kubernetes (ACK) 中的集群. Rancher 已经为 ACK 实现并打包了[集群驱动](/docs/admin-settings/drivers/cluster-drivers/), 但是默认情况下, 这个集群驱动是`非活动的`. 为了启动 ACK 集群, 您需要[启用ACK集群驱动程序](/docs/admin-settings/drivers/cluster-drivers/#activating-deactivating-cluster-drivers). 启用集群驱动后，可以开始配置 ACK 集群.

### 预备知识

> **注意**
> 部署到 ACK 将会产生费用.

1. 在阿里云中, 通过控制台激活以下服务.

   - [Container Service](https://cs.console.aliyun.com)
   - [Resource Orchestration Service](https://ros.console.aliyun.com)
   - [RAM](https://ram.console.aliyun.com)

2. 确保您将用于创建 ACK 集群的帐户具有适当的权限. 从阿里巴巴云的官方文档[角色授权](https://www.alibabacloud.com/help/doc-detail/86483.htm)和[使用容器服务控制台作为 RAM 用户](https://www.alibabacloud.com/help/doc-detail/86484.htm)获得详细信息.

3. 在阿里云里，创建一个[访问密钥](https://www.alibabacloud.com/help/doc-detail/53045.html).

4. 在阿里云里, 创建一个[SSH 密钥对](https://www.alibabacloud.com/help/doc-detail/51793.html). 这个密钥用于访问 Kubernetes 集群中的节点.

### 创建一个 ACK 集群

1. 在 **集群** 页, 点击 **添加**.

1. 选择 **Alibaba ACK**.

1. 输入 **集群名称**.

1. {{< step_create-cluster_member-roles >}}

1. 为 ACK 集群配置 **访问账户**. 选择构建集群的地理区域, 并输入之前创建的访问密钥.

1. 单击 **下一步: 配置集群**, 然后选择集群类型、Kubernetes 版本和可用的区域.

1. 如果你选择 **Kubernetes** 作为集群类型, 单击 **下一步: 配置主节点**, 然后完成 **主节点** 表单的配置.

1. 点击 **下一步: 配置 Worker 节点**, 完成 **Worker 节点** 表单.

1. 检查你的选项是否正确. 然后点击 **创建**.

{{< result_create-cluster >}}
