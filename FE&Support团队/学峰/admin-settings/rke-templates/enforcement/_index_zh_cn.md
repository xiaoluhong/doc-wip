---
标题: 模板强制
---

本节介绍模板管理员如何在Rancher中强制使用模板，从而限制用户在没有模板的情况下创建集群的能力。

默认情况下，Rancher中的任何标准用户都可以创建集群。但当开启RKE模板强制时，

- 只有管理员才能在没有模板的情况下创建群集。
- 所有标准用户必须使用RKE模板创建新群集。
- 标准用户不能在不使用模板的情况下创建群集。

用户只能在管理员[授予其权限](/docs/admin-settings/rke-templates/creator-permissions/#allowing-a-user-to-create-templates)的情况下创建新模板

使用RKE模板创建集群后，集群创建者无法编辑模板中定义的设置。创建集群后更改这些设置的唯一方法是[将集群升级到新版本](/docs/admin-settings/rke-templates/applying-templates/#updating-a-cluster-created-with-an-rke-template)相同模板。如果集群创建者想要更改模板定义的设置，他们需要联系模板所有者以获取模板的新版本。有关模板修订工作方式的详细信息，请参阅[修订模板文档](/docs/admin settings/rke templates/creating and revising/#update-a-template)

## 要求新集群使用RKE模板

您可能希望要求新群集使用模板，以确保[标准用户](/docs/admin-settings/rbac/global-permissions/)启动的任何群集都将使用经过管理员审核的Kubernetes和/或Rancher设置。

要要求新群集使用RKE模板，管理员可以通过以下步骤启用RKE模板强制:

1. 在**全局**视图中，单击**设置**选项卡。
1. 转到`cluster template enforcement`设置。单击垂直**省略号(…)**并单击**编辑**
1. 将该值设置为**真**，然后单击**保存**

**结果:** Rancher设置的所有群集都必须使用模板，除非创建者是管理员。

## 禁用RKE模板强制

要允许在没有RKE模板的情况下创建新群集，管理员可以通过以下步骤关闭RKE模板强制:

1. 在**全局**视图中，单击**设置**选项卡。
1. 转到`cluster template enforcement`设置。单击垂直**省略号(…)**并单击**编辑**
1. 将该值设置为**False**并单击**保存**

**结果:** 当Rancher提供集群时，它们不需要使用模板。