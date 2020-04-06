---
标题: 示例场景
---

这些示例场景描述了组织如何使用模板来标准化集群创建。

- **强制模板:** 如果管理员希望所有新的Rancher配置的群集都具有这些设置，则可能希望[强制每个人使用一个或多个模板设置](#enforcing-a-template-setting-for-everyone)。
- **与不同的用户共享不同的模板:** 管理员可能会给[基本用户和高级用户提供不同的模板，](#templates-for-basic-and-advanced-users)，以便基本用户有更多受限制的选项，高级用户在创建群集时有更多的自主权。
- **更新模板设置:** 如果组织的安全和DevOps团队决定将最佳实践嵌入到新集群所需的设置中，则这些最佳实践可能会随时间而改变。如果最佳实践发生更改，[模板可以更新为新版本](#updating-templates-and-clusters-created-with-them)，并且从模板创建的群集可以升级到模板的新版本。
- **共享模板的所有权:** 当模板所有者不再希望维护模板或希望委托模板的所有权时，此场景描述如何[共享模板所有权](#allowing-other-users-to-control-and-share-a-template)

## 为每个人强制执行模板设置

假设有一个组织，管理员决定用Kubernetes版本1.14创建所有新集群。

1. 首先，管理员创建一个模板，将Kubernetes版本指定为1.14，并将所有其他设置标记为**允许用户覆盖**。
1. 管理员将模板公开。
1. 管理员打开模板强制。

**结果:**

- 组织中的所有Rancher用户都可以访问该模板。
- [标准用户](/docs/admin-settings/rbac/global-permissions/)使用此模板创建的所有新群集都将使用Kubernetes 1.14，它们无法使用其他Kubernetes版本。默认情况下，标准用户没有创建模板的权限，因此除非与他们共享更多模板，否则此模板将是他们唯一可以使用的模板。
- 所有标准用户必须使用群集模板创建新群集。如果不使用模板，则无法创建群集。

通过这种方式，管理员在整个组织中强制使用Kubernetes版本，同时仍然允许最终用户配置其他所有内容。

## 基本和高级用户模板

假设一个组织有基本用户和高级用户。管理员希望基本用户必须使用模板，而高级用户和管理员可以根据自己的需要创建集群。

1. 首先，管理员打开[RKE template enforcement.](/docs/admin-settings/rke-templates/enforcement/#requiring-new-clusters-to-use-a-cluster-template)这意味着Rancher中的每个[标准用户](/docs/admin settings/rbac/global permissions/)在创建群集时都需要使用RKE模板。
1. 然后，管理员创建两个模板:

- 一个基本用户模板，除了访问密钥之外，几乎所有选项都指定了
- 一个用于高级用户的模板(具有大多数或所有选项)已启用**允许用户覆盖**

1. 管理员只与高级用户共享高级模板。
1. 管理员将基本用户的模板设置为公共模板，因此对于创建由Rancher提供的群集的每个人来说，更严格的模板是一个选项。

**结果:** 创建群集时，除管理员外，所有Rancher用户都必须使用模板。每个人都有权访问限制性模板，但只有高级用户才有权使用更为允许的模板。基本用户受到更多限制，而高级用户在配置其Kubernetes集群时有更多的自由。

## 更新用它们创建的模板和群集

假设一个组织有一个模板，它要求集群使用Kubernetes v1.14。然而，随着时间的推移，管理人员会改变主意。他们决定希望用户能够升级他们的集群以使用更新版本的Kubernetes。

在这个组织中，许多集群是用一个需要Kubernetes v1.14的模板创建的。由于模板不允许重写该设置，因此创建群集的用户无法直接编辑该设置。

模板所有者有几个选项允许集群创建者在其集群上升级Kubernetes:

- **在模板上指定Kubernetes v1.15:** 模板所有者可以创建指定Kubernetes v1.15的新模板修订版。然后使用该模板的每个集群的所有者可以将其集群升级到模板的新版本。此模板升级允许集群创建者将其集群上的Kubernetes升级到v1.15。
- **允许在模板上使用任何Kubernetes版本:** 创建模板修订时，模板所有者还可以使用Rancher UI上该设置附近的开关将Kubernetes版本标记为 **允许用户覆盖**。这将允许升级到此模板修订版的集群使用Kubernetes的任何版本。
- **允许在模板上使用最新的次要Kubernetes版本:** 模板所有者还可以创建一个模板版本，其中Kubernetes版本定义为 **最新的v1.14(允许修补程序版本升级)。** 这意味着使用该版本的群集将能够获得修补程序版本升级，但不允许主要版本升级。

## 允许其他用户控制和共享模板

假设爱丽丝是个牧场管理员。她拥有一个RKE模板，该模板反映了她的组织为创建集群而商定的最佳实践。

Bob是一个高级用户，可以对集群配置做出明智的决策。随着最佳实践的不断更新，Alice相信Bob会创建新的模板修订版。因此，她决定让Bob成为模板的所有者。

要与Bob共享模板的所有权，Alice[将Bob添加为模板的所有者](/docs/admin-settings/rke-templates/template-access-and-sharing/#sharing-ownership-of-templates)

结果是，作为模板所有者，Bob负责该模板的版本控制。Bob现在可以执行以下所有操作:

- [修改模板](/docs/admin-settings/rke-templates/creating-and-revising/#updating-a-template)最佳实践更改时
- [禁用过时的修订](/docs/admin-settings/rke-templates/creating-and-revising/#disabling-a-template-revision)模板，以便不能用它创建新的集群
- [删除整个模板](/docs/admin-settings/rke-templates/creating-and-revising/#deleting-a-template)如果组织希望朝不同的方向发展
- [将某个修订设置为默认值](/docs/admin-settings/rke-templates/creating-and-revising/#setting-a-template-revision-as-default)当用户使用它创建集群时。模板的最终用户仍可以选择要使用哪个修订版创建群集。
- [共享模板](/docs/admin-settings/rke-templates/template-access-and-sharing)与特定用户共享模板，使所有Rancher用户都可以使用该模板，或与其他用户共享该模板的所有权。