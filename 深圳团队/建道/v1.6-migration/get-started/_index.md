---
标题: '1. 开始使用'
---

开始迁移到Rancherv2.x通过安装Rancher并配置新的Rancher环境。

### 大纲

<!-- TOC -->

- [A. 安装Rancher v2.x](#a-install-rancher-v2-x)
- [B. 配置身份验证](#b-configure-authentication)
- [C. 提供一个集群和项目](#c-provision-a-cluster-and-project)
- [D. 创建堆栈](#d-create-stacks)

<!-- /TOC -->

### A. 安装 Rancher v2.x

从v1.6迁移到v2.x的第一步是将Rancher v2.x Server与v1.6 Server并排安装，因在迁移过程中需要使用旧的安装。 由于v1.6和v2.x之间的体系结构更改，因此没有直接的升级途径。 您必须独立安装v2.x，然后将v1.6服务迁移到v2.x。

v2.x的新功能，与Rancher Server的所有通信均已加密。 以下过程不仅指导你安装Rancher，而且还将指导你创建和安装这些证书。

在安装v2.x之前，请提供一台或多台主机以用作Rancher Server(s)。 你可以在以下位置找到这些主机的要求 [Server要求](/docs/installation/requirements/).

提供节点后, 开始安装Rancher:

- [Docker安装](/docs/installation/single-node)

  对于开发环境，可以使用Docker将Rancher安装在单个节点上。 此安装过程将单个Rancher容器部署到你的主机。

- [Kubernetes安装](/docs/installation/k8s-install/)

  对于用户需要不断访问集群的生产环境，我们推荐采用高可用Kubernetes集群安装的方式安装Rancher。 此安装过程将设置一个三节点集群，并使用Helm chart在每个节点上安装Rancher。

  > **重要区别:** 尽管你可以在每个节点上使用外部数据库和Docker命令以高可用Kubernetes配置来安装Rancher v1.6，但Kubernetes安装中的Rancher v2.x需要现有的Kubernetes集群。 查阅 [Kubernetes安装](/docs/installation/k8s-install/) 以满足所有要求。

### B. 配置身份验证

安装Rancher v2.x Server之后，我们建议配置外部身份验证（例如Active Directory或GitHub），以便用户可以使用其单点登录Rancher。 有关已支持的身份验证提供程序的完整列表以及有关如何配置它们的说明, 查看 [认证方式](/docs/admin-settings/authentication).

<figcaption>Rancher v2.x Authentication</figcaption>

![Rancher v2.x 认证方式](/img/rancher/auth-providers.svg)

#### 本地用户

尽管我们建议使用外部身份验证提供程序，但Rancher v1.6和v2.x都为Rancher本地用户提供支持。 但是，这些用户无法从Rancher v1.6迁移到v2.x。 如果你在Rancher v1.6中使用了本地用户，并希望在v2.x中继续进行使用， 你会需要去 [手动重新创建这些用户账号](/docs/admin-settings/authentication/) 并为其分配访问权限。

最佳实践，你应该使用外部_和_本地身份验证的混合体。 这种做法可在你的外部身份验证遇到中断时提供对Rancher的访问，因为你仍然可以使用本地用户帐户登录。 设置一些本地帐户作为Rancher的管理用户。

#### SAML 身份验证提供者

在Rancher v1.6中，我们鼓励SAML用户使用Shibboleth，因为它是我们提供的唯一SAML身份验证选项。 但是，为了更好地对它们的微小差异进行支持，我们为v2.x添加了经过充分测试的SAML提供程序：Ping Identity，Microsoft ADFS和FreeIPA。

### C.提供一个集群和项目

开始在Rancher v2.x的工作通过配置新的Kubernetes集群，类似于v1.6中的环境。 该集群将托管你的应用程序部署。

在Rancher v2.x中结合在一起的集群和项目等效于v1.6环境。 _集群_是计算边界（即你的主机），而_项目_是管理边界（即用于为用户分配访问权限的命名空间组）。

以下标题中提供了有关配置集群的更多基本信息，但有关完整信息， 查看 [配置Kubernetes集群](/docs/cluster-provisioning/).

#### 集群

在Rancher v1.6中，计算节点已添加到_环境_中。 Rancher v2.x避免使用_环境_来表示_集群_，因为Kubernetes将此术语用于一组计算机而不是_环境_。

Rancher v2.x允许你在任何地方启动Kubernetes集群。 使用以下方式托管集群：

- 一个 [托管的Kubernetes提供商](/docs/cluster-provisioning/hosted-kubernetes-clusters/).
- 一个 [来自基础设施提供商的节点池](/docs/cluster-provisioning/rke-clusters/node-pools/). Rancher在节点上启动Kubernetes。
- 任何 [自定义节点](/docs/cluster-provisioning/rke-clusters/custom-nodes/). Rancher可以在节点上启动Kubernetes，无论是裸机服务器，虚拟机还是不太流行的基础架构提供商上的云主机。

#### 项目

此外，Rancher v2.x引入了 [项目](/docs/k8s-in-rancher/projects-and-namespaces/), 该对象将集群分为不同的应用程序组，这对于应用用户权限很有用。 集群和项目的这种模型允许多租户，因为主机归集群所有，并且可以将集群进一步划分为多个项目，用户可以在其中管理应用程序，而其他用户则不能。

当你创建一个集群时，将自动创建两个项目：

- `System` 项目，其中包含运行重要的Kubernetes资源的系统命名空间（例如ingress控制器和集群dns服务）
- `Default` 项目。

但是，对于生产环境， 我们推荐 [创建自己的项目](/docs/project-admin/namespaces/#creating-projects) 并为其指定一个描述性名称。

在配置新的集群和项目之后，你可以授权用户访问和使用项目资源。 与Rancher v1.6环境类似， Rancher v2.x 允许你 [将用户分配给项目](/docs/k8s-in-rancher/projects-and-namespaces/editing-projects/). 通过将用户分配给项目，你可以限制用户可以访问哪些应用程序和资源。

### D.创建堆栈

在Rancher v1.6中，使用_堆栈_将属于你应用程序的服务分组在一起。 在v2.x中，你需要去 [创建命名空间](/docs/k8s-in-rancher/projects-and-namespaces/#creating-namespaces), 出于相同的目的，它们相当于v2.x的堆栈。

在Rancher v2.x中，名称空间是项目的子对象。 创建项目时，会向该项目添加一个`default`命名空间，但你可以创建自己的名称空间以与v1.6的堆栈并行。

在迁移过程中，如果你未明确定义服务应部署到的名称空间，则会将其部署到`default`命名空间。

与v1.6一样，Rancher v2.x支持命名空间内和跨命名空间的服务发现 (我们很快将会了解到[服务发现](/docs/v1.6-migration/discover-services)).

#### [下一步]: 迁移你的服务](/docs/v1.6-migration/run-migration-tool)
