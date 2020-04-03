---
title: 使用Rancher的项目和Kubernetes命名空间
---

命名空间是Kubernetes的概念，它允许集群中的虚拟集群，这对于将集群划分为单独的“虚拟集群”非常有用，每个虚拟集群都有自己的访问控制和资源配额。

项目是一组命名空间，是Rancher引入的一个概念。 项目使您可以将多个名称空间作为一个组进行管理，并在其中执行Kubernetes操作。 您可以使用项目来支持多租户，以便团队可以访问集群中的项目而无需访问同一集群中的其他项目。

本节描述项目和名称空间如何与Rancher一起使用。 它涵盖以下主题：

- [Namespace命名空间描述](#Namespace命名空间描述)
- [Project项目描述](#Project项目描述)
  - [集群默认（Default）项目](#集群默认（Default）项目)
  - [集群系统（System）项目](#集群系统（System）项目)
- [项目授权](#项目授权)
- [Pod安全策略](#Pod安全策略)
- [创建项目](#创建项目)
- [集群与项目间切换](#集群与项目间切换)

## Namespace命名空间描述

命名空间是Kubernetes引入的概念。根据[关于名称空间的Kubernetes官方文档](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

> Kubernetes支持由同一物理集群支持的多个虚拟集群。这些虚拟集群称为名称空间。 [...]命名空间旨在用于具有多个用户的环境，这些用户分布在多个团队或项目中。对于拥有几到几十个用户的集群，您根本不需要创建或考虑名称空间。

命名空间提供以下功能：

- **提供名称范围：**资源名称在名称空间内必须是唯一的，但在名称空间之间则必须唯一。命名空间不能彼此嵌套，并且每个Kubernetes资源只能位于一个命名空间中。
- **资源配额：**命名空间提供了一种在多个用户之间划分集群资源的方法。

您可以在项目级别分配资源，以便项目中的每个名称空间都可以使用它们。您还可以通过将资源显式分配给名称空间来绕过此继承。

您可以将以下资源直接分配给名称空间：

- [工作负载](/docs/k8s-in-rancher/workloads/)
- [负载均衡/Ingress](/docs/k8s-in-rancher/load-balancers-and-ingress/)
- [服务发现](/docs/k8s-in-rancher/service-discovery/)
- [PVC](/docs/k8s-in-rancher/volumes-and-storage/persistent-volume-claims/)
- [证书](/docs/k8s-in-rancher/certificates/)
- [配置映射](/docs/k8s-in-rancher/configmaps/)
- [镜像仓库凭证](/docs/k8s-in-rancher/registries/)
- [密文](/docs/k8s-in-rancher/secrets/)

为了管理普通Kubernetes集群中的权限，集群管理员为每个名称空间配置基于角色的访问策略。 使用Rancher，用户权限将改为在项目级别分配，并且权限由特定项目拥有的任何名称空间自动继承。

> **Note:** 如果您使用`kubectl`创建命名空间，则可能无法使用，因为`kubectl`不需要将新命名空间限制在您有权访问的项目中。 如果您的权限仅限于项目级别，则最好[通过Rancher创建名称空间](/docs/project-admin/namespaces/#creating-namespaces)以确保您有权访问该名称空间。

有关创建和移动名称空间的更多信息，请参见[名称空间](/docs/project-admin/namespaces/)。

## Project项目描述

在层次结构方面：

- 集群包含项目
- 项目包含名称空间

您可以使用项目来支持多租户，以便团队可以访问集群中的项目而无需访问同一集群中的其他项目。

在基本版本的Kubernetes中，诸如基于角色的访问权限或集群资源之类的功能已分配给各个名称空间。 通过使个人或团队同时访问多个名称空间，项目可以使您节省时间。

您可以使用项目执行以下操作：

- 将用户分配到一组名称空间（即[项目成员资格](/docs/k8s-in-rancher/projects-and-namespaces/project-members)）。
- 在项目中为用户分配特定角色。 角色可以是所有者，成员，只读或[自定义](/docs/admin-settings/rbac/default-custom-roles/)。
- 为项目分配资源。
- 分配Pod安全策略。

创建集群时，将在其中自动创建两个项目：

- [集群默认（Default）项目](#集群默认（Default）项目)
- [集群系统（System）项目](#集群系统（System）项目)

#### 集群默认（Default）项目

当您使用Rancher设置集群时，它将自动为该集群创建一个 `默认` 项目。 您可以使用该项目来开始使用集群，但是您始终可以将其删除，并用具有更多描述性名称的项目替换它。

如果您不需要的仅是默认名称空间，那么您也不需要仅需要Rancher中的 **Default** 项目。

如果您需要除 **Default** 项目以外的其他组织级别，则可以在Rancher中创建更多项目以隔离名称空间，应用程序和资源。

#### 集群系统（System）项目

_从v2.0.7版本开始支持_

故障排除时，您可以查看`system`项目以检查Kubernetes系统中的重要名称空间是否正常运行。 这个易于访问的项目使您不必对单个系统名称空间容器进行故障排除。

要打开它，请打开**全局**菜单，然后为您的集群选择 `System` 项目。

系统项目：

- 在配置集群时自动创建。
- 列出存在于 `v3/settings/system-namespaces` 中的所有命名空间。
- 允许您添加更多名称空间或将其名称空间移至其他项目。
- 无法删除，因为它是集群操作所必需的。

> **Note:** 在两个集群中：
>
> - [运河网络插件](/docs/cluster-provisioning/rke-clusters/options/#canal)正在使用。
> - 启用了项目网络隔离选项。
>
> `System` 项目将覆盖项目网络隔离选项，以便它可以与其他项目通信，收集日志并检查运行状况。

## 项目授权

仅在两种情况下，标准用户才有权访问项目：

- 管理员，集群所有者或集群成员将标准用户明确添加到项目的**成员**选项卡中。
- 标准用户可以访问自己创建的项目。

## Pod安全策略

Rancher扩展了Kubernetes以允许在[项目级别](/docs/project-admin/pod-security-policies)上应用[Pod安全策略](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)，作为[集群级别](../pod-security-policy)之外的安全性策略。但是，作为最佳实践，我们建议在集群级别应用Pod安全策略。

## 创建项目

本节介绍如何使用名称以及可选的pod安全策略，成员和资源配额创建新项目。

1. [命名新项目](#1-命名新项目)
2. [可选：选择一个Pod安全策略](#2-可选：选择一个Pod安全策略)
3. [推荐：添加项目成员](#3-推荐：添加项目成员)
4. [可选：添加资源配额](#4-可选：添加资源配额)

#### 1. 命名新项目

1. 从**全局**视图中，从主菜单中选择**集群**。 从**集群**页面中，打开要从中创建项目的集群。

1. 从主菜单中，选择**项目/命名空间**。 然后点击**添加项目**。

1. 输入一个**项目名称**。

#### 2. 可选：选择一个Pod安全策略

仅当您已经创建了Pod安全策略时，此选项才可用。 有关说明，请参阅[创建Pod安全策略](/docs/admin-settings/pod-security-policies/)。

将PSP分配给项目将：

- 覆盖集群的默认PSP。
- 将PSP应用于项目。
- 将PSP应用于以后添加到项目中的任何名称空间。

#### 3. 推荐：添加项目成员

使用**成员**部分为其他用户提供项目访问权限和角色。

默认情况下，您的用户被添加为项目`Owner`。

> **关于权限的说明：**
>
> - 为项目分配了`所有者`或`成员`角色的用户会自动继承`命名空间创建`角色。但是，此角色是[Kubernetes ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)，这意味着它的作用域扩展到了集群中的所有项目。因此，为项目明确分配了`所有者`或`成员`角色的用户，即使仅分配了`只读`角色，也可以在分配给他们的其他项目中创建名称空间。
> - 选择`自定义`以动态创建自定义角色：[自定义项目角色](/docs/admin-settings/rbac/cluster-project-roles/#custom-project-roles)。

要添加成员：

1. 点击`添加成员`。
1. 在`名称`组合框中，搜索要分配项目访问权限的用户或组。注意：只有启用了外部身份验证，您才可以搜索组。
1. 从`角色`下拉列表中，选择一个角色。有关更多信息，请参阅[关于项目角色的文档。](/docs/admin-settings/rbac/cluster-project-roles/#custom-project-roles)

#### 4. 可选：添加资源配额

_从v2.1.0版本开始支持_

资源配额限制了项目（及其名称空间）可以消耗的资源。有关更多信息，请参见[资源配额](/docs/k8s-in-rancher/projects-and-namespaces/resource-quotas)。

要添加资源配额，

1. 单击**添加配额**。
1. 选择[资源类型](/docs/k8s-in-rancher/projects-and-namespaces/resource-quotas/#resource-quota-types)。
1. 输入`项目限制`和`名称空间默认限制`的值。
1. **可选：**指定**容器默认资源限制**，它将应用于项目中启动的每个容器。如果您有由资源配额设置的CPU或内存限制，则建议使用此参数。可以在单个命名空间或容器级别上覆盖它。有关更多信息，请参见[容器默认资源限制](/docs/project-admin/resource-quotas/#setting-container-default-resource-limit)，注：此选项自v2.2.0起可用。
1. 单击**创建**。

**结果：**您的项目已创建。您可以从集群的**项目/名称空间**视图中查看它。

| 字段                   | 描述             |
| ----------------------- | ----------------------------- |
| 项目配额限制           | 项目所有的配额限制。                                    |
| 命名空间默认配额限制 | 项目默认的配额限制，项目下所有项目的配额限制之和不得大于项目配额。 |

## 集群与项目间切换

要在集群和项目之间切换，请使用主菜单中的**Global**下拉菜单。

![Global Menu](/img/rancher/global-menu.png)

或者，您可以使用主菜单在项目和集群之间切换。

- 要在集群之间切换，请打开**全局**视图，然后从主菜单中选择**集群**。 然后打开一个集群。
- 要在项目之间切换，请打开一个集群，然后从主菜单中选择**项目/命名空间**。 选择要打开的项目的链接。
