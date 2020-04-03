---
title: 创建腾讯 TKE 集群
---

_从v2.2.0_开始可用_

您可以使用Rancher创建一个托管在Tencent Kubernetes Engine（TKE）中的集群. Rancher已经为TKE实现并打包了[集群驱动](/docs/admin-settings/drivers/cluster-drivers/), 但是默认情况下，这个集群驱动程序是`非活动的`. 为了启动TKE集群, 您需要[启用TKE集群驱动程序](/docs/admin-settings/drivers/cluster-drivers/#activating-deactivating-cluster-drivers). 启用集群驱动程序后. 您可以开始配置TKE集群.

### 前提条件

> **注意**
> 部署到 TKE 将产生费用.

1. 请确保您将用于创建TKE集群的帐户具有适当的权限,详细信息请参考[云访问管理](https://intl.cloud.tencent.com/document/product/598/10600)文档.

2. 创建 [云API密钥ID和密钥](https://console.cloud.tencent.com/capi).

3. 创建 [专用网络和子网](https://intl.cloud.tencent.com/document/product/215/4927) 在您要部署Kubernetes集群的区域中.

4. 创建 [SSH密钥对](https://intl.cloud.tencent.com/document/product/213/6092). 此密钥用于访问Kubernetes集群中的节点.

### 创建 TKE 集群

1. 从 **集群** 页面, 单击 **添加集群**.

2. 选择 **Tencent TKE**.

3. 输入 **集群名称**.

4. {{< step_create-cluster_member-roles >}}

5. 为 TKE 集群配置**帐户访问**. 使用[前提条件](#prerequisites)中获得的信息完成每个下拉列表和字段.

   | 选项     | 描述                                                                        |
   | ---------- | ---------------------------------------------------------------------------------- |
   | 区域     | 从下拉列表中选择要在其中构建集群的地理区域. |
   | Secret ID  | 输入从腾讯云控制台获取的 Secret ID.              |
   | Secret Key | 输入从腾讯云控制台获取的 Secret key.                 |

6. 单击 `下一步: 配置集群` 来设置您的TKE集群配置.

   | 选项                 | 描述                                                                                                                                                               |
   | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | Kubernetes 版本     | TKE 现在只支持Kubernetes版本1.10.5.                                                                                                                      |
   | 节点数量             | 输入您希望为 Kubernetes 集群购买的工作节点数量,最多100个.                                                                              |
   | VPC                    | 选择您在腾讯云控制台中创建的VPC名称.                                                                                                   |
   | 容器 CIDR 网络 | 输入 Kubernetes 集群的CIDR范围, 您可以在腾讯云控制台的VPC服务中查看CIDR的可用范围。默认172.16.0.0/16. |

   **注意:** 如果要在`cluster.yml`而不是Rancher UI中编辑集群,请注意,从Rancher v2.3.0开始,集群配置指令必须嵌套在`cluster.yml`中的`Rancher_kubernetes_engine_config`指令下. 有关更多信息,请参阅[Rancher v2.3.0+中的配置文件结构](/docs/cluster-provisioning/rke-clusters/options/#config-file-structure-in-rancher-v2-3-0)一节

7. 单击 `下一步: 选择实例类型` 选择将用于 TKE 集群的实例类型.

   | 选项            | 描述                                                                                                                           |
   | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
   | 可用性区域 | 选择 VPC 的可用性区域.                                                                                       |
   | 子网            | 选择在VPC中创建的子网, 如果在选择的可用性区域中没有子网, 则添加一个新的子网.       |
   | 实例类型     | 从下拉菜单中选择要用于TKE集群的VM实例类型,默认为 S2.MEDIUM4 (CPU 2 Memory 4 GiB). |

8. 单击 `下一步: 配置实例` 配置将用于TKE集群的VM实例.

   | 选项           | 描述                                                                                                                                             |
   | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | 操作系统 | 操作系统的名称，当前支持Centos7.2x86_64 或 ubuntu16.04.1 LTSx86_64.                                                        |
   | 安全组   | 安全组ID，默认不绑定任何安全组.                                                                                           |
   | 根磁盘类型   | 系统磁盘类型. 系统磁盘类型限制详见[CVM实例配置](https://cloud.tencent.com/document/product/213/11518). |
   | 根磁盘大小   | 系统磁盘大小. Linux系统调整范围为20-50G, 步长为1.                                                                            |
   | 数据磁盘类型   | 数据磁盘类型, SSD云驱动器的默认值                                                                                                  |
   | 数据磁盘大小   | 数据磁盘大小（GB）,步长为10.                                                                                                        |
   | 带宽类型  | 带宽类型、按流量计费或按小时计费                                                                                                    |
   | 带宽       | 公网带宽（Mbps）                                                                                                                         |
   | 密钥对         | 密钥id,在关联密钥后可以用来记录到VM节点                                                                              |

9. 单击 **创建**.

{{< result_create-cluster >}}
