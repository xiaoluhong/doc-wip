---
title: 使用Helm 2在kubernetes安装Rancher
---

> Helm 3 已经发布，Rancher安装说明已经更新成使用Helm 3来安装。

> 如果您使用的是Helm 2，我们建议您[迁移到Helm 3](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/)，因为Helm 3比Helm 2更安全，更容易使用。

> 本节提供了较早版本的使用Helm 2安装高可用Kubernetes Rancher的安装方法，如果无法升级到Helm 3，可以使用此方法。

对于生产环境，我们建议以高可用配置的方式来安装Rancher，以便用户可以一直访问Rancher服务。在kubernetes集群中安装Rancher时，Rancher会与集群的etcd数据库集成，并利用kubernetes调度来实现高可用。

以下步骤将指导您使用Rancher Kubernetes Engine(RKE)来部署三个节点的集群，并使用Helm安装Rancher。

> **重要:** Rancher的管理服务只能在RKE管理的kubernetes集群上运行。不支持将Rancher运行在托管的Kubernetes服务或者其他供应商的托管服务。

> **重要:** 为了最佳的性能，我们建议使用专用kubernetes集群来运行Rancher管理服务，不建议在此集群上运行用户的工作负载。部署Rancher之后，您可以通过[创建或导入集群](/docs/cluster-provisioning/#cluster-creation-in-rancher) 来运行您的工作负载。

### 推荐架构

- Rancher的DNS应为4层负载均衡器（TCP）
- 负载均衡器应允许kubernetes集群中全部三个节点的TCP/80和TCP/443端口访问。
- Ingress controller会将HTTP重定向到HTTPS，并在端口TCP/443终止SSL/TLS。
- Ingress controller会将流量转发到Rancher Pod 的TCP/80端口。

<figcaption>对于结合L4负载均衡部署的HA Rancher，对其访问的SSL会终止在ingress controller中。</figcaption>
![基于Kubernetes的高可用安装](/img/rancher/ha/rancher2ha.svg)
<sup>对于结合L4负载均衡部署的HA Rancher(TCP)，对其访问的SSL会终止在ingress controller中。</sup>

### 依赖工具

在安装过程中依赖以下CLI工具。请确认这些工具已经安装完成并且配置到环境变量 `$PATH` 中

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes 命令行工具.
- [rke]({{<baseurl>}}/rke/latest/en/installation/) - Rancher Kubernetes Engine, 构建kubernetes集群的cli工具
- [helm](https://docs.helm.sh/using_helm/#installing-helm) - Kubernetes包管理工具. 可参考 [Helm version requirements](/docs/installation/options/helm-version) 来选择合适的Helm版本安装Rancher。

### 安装大纲

- [创建Nodes与Load Balancer](/docs/installation/options/helm2/create-nodes-lb/)
- [使用rke安装Kubernetes](/docs/installation/options/helm2/kubernetes-rke/)
- [初始化Helm (tiller)](/docs/installation/options/helm2/helm-init/)
- [安装Rancher](/docs/installation/options/helm2/helm-rancher/)

### 其他安装选项

- [从RKE Add-on方式安装的Kubernetes迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)

### 较早的方法

[RKE add-on 安装](/docs/installation/options/helm2/rke-add-on/)

> **重要: RKE add-on 安装方式仅支持到 Rancher v2.0.8**
>
> 请在Kubernetes集群中使用Rancher helm chart来安装Rancher。更多内容，请参考[Kubernetes 安装 - 安装大纲](/docs/installation/options/helm2/#installation-outline).
>
> 如果您当前正在使用RKE add-on安装方法, 请参考 [从RKE Add-on方式安装的Kubernetes迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)
