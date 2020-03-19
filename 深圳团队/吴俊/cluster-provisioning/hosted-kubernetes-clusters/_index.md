---
title: 从托管的Kubernetes提供商设置集群
---
在这种情况下, rancher 不预设 kubernetes, 因为它是通过 Google Kubernetes Engine (GKE), Amazon Elastic Container Service for Kubernetes, or Azure Kubernetes Service 等提供商来安装的.

如果您使用了一个 Kubernetes 提供商, 如谷歌 GKE, Rancher 集成了它的云api，允许您通过 Rancher UI 创建和管理托管集群的基于角色的访问控制。

在这个用例中, Rancher使用供应商的API向托管的供应商发送请求. 然后，提供商为您提供并托管集群. 当集群完成构建时, 您可以通过 Rancher UI 管理它, 同时还可以管理您位于本地或云服务商提供的的集群.

Rancher支持以下Kubernetes供应商:

| Kubernetes 提供商                                                                                           | 可用于 |
| --------------------------------------------------------------------------------------------------------------- | --------------- |
| [Google GKE (Google Kubernetes Engine)](https://cloud.google.com/kubernetes-engine/)                            | v2.0.0          |
| [Amazon EKS (Amazon Elastic Container Service for Kubernetes)](https://aws.amazon.com/eks/)                     | v2.0.0          |
| [Microsoft AKS (Azure Kubernetes Service)](https://azure.microsoft.com/en-us/services/kubernetes-service/)      | v2.0.0          |
| [Alibaba ACK (Alibaba Cloud Container Service for Kubernetes)](https://www.alibabacloud.com/product/kubernetes) | v2.2.0          |
| [Tencent TKE (Tencent Kubernetes Engine)](https://intl.cloud.tencent.com/product/tke)                           | v2.2.0          |
| [Huawei CCE (Huawei Cloud Container Engine)](https://www.huaweicloud.com/en-us/product/cce.html)                | v2.2.0          |

### 托管与于Kubernetes提供商的身份验证

当使用Rancher创建由提供商托管的集群时, 会提示您输入身份验证信息。访问提供商的API需要这些信息。有关如何获取此信息的详细信息，请参阅以下步骤:

- [创建 GKE 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/gke)
- [创建 EKS 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/eks)
- [创建 AKS 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/aks)
- [创建 ACK 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/ack)
- [创建 TKE 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/tke)
- [创建 CCE 集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/cce)
