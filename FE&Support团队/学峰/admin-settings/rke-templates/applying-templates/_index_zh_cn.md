---
标题: 应用模板
---

您可以从创建的RKE模板或已[与您共享的模板](/docs/admin-settings/rke-templates/template-access-and-sharing)创建群集。

RKE模板可以应用于新的集群。

从Rancher v2.3.3开始，您可以[将现有群集的配置保存为RKE模板](#converting-an-existing-cluster-to-use-an-rke-template)，然后只有更新模板时才能更改群集的设置。

不能将群集更改为使用其他RKE模板。只能将群集更新为同一模板的新版本。

本节包括以下主题:

- [从RKE模板创建集群](#converting-an-existing-cluster-to-use-an-rke-template)
- [更新用RKE模板创建的群集](#updating-a-cluster-created-with-an-rke-template)
- [将现有群集转换为使用RKE模板](#converting-an-existing-cluster-to-use-an-rke-template)

#### 从RKE模板创建群集

要使用rke模板添加群集[由基础结构提供商托管](/docs/cluster-provisioning/rke-clusters)，请使用以下步骤:

1. 从**全局**视图，转到**群集**选项卡。
1. 单击**添加群集**，然后选择基础设施提供商。
1. 像往常一样提供集群名称和节点模板的详细信息。
1. 若要使用RKE模板，请在**群集选项**下选中**使用现有RKE模板和修订版复选框**
1. 从下拉菜单中选择现有模板和修订。
1. 可选项:可以编辑创建模板时RKE模板所有者标记为**允许用户覆盖**的任何设置。如果有要更改的设置，但没有此选项，则需要与模板所有者联系以获取模板的新版本。然后您需要编辑集群以将其升级到新版本。
1. 单击**保存**启动群集。

#### 更新使用RKE模板创建的群集

当模板所有者创建模板时，每个设置在Rancher UI中都有一个开关，指示用户是否可以覆盖该设置。

- 如果设置允许用户覆盖，则可以通过[编辑群集](/docs/cluster-admin/editing-clusters/)更新群集中的这些设置
- 如果该开关处于关闭状态，则除非群集所有者创建了允许您覆盖这些设置的模板修订版，否则无法更改这些设置。如果有要更改的设置，但没有此选项，则需要与模板所有者联系以获取模板的新版本。

如果集群是从RKE模板创建的，则可以编辑集群以将集群更新为模板的新版本。

从Rancher v2.3.3开始，现有群集的设置可以[保存为RKE模板](##converting-an-existing-cluster-to-use-an-rke-template)在这种情况下，还可以编辑群集以将群集更新为模板的新版本。

> **注意:** 不能将集群更改为使用其他RKE模板。只能将群集更新为同一模板的新版本。

#### 将现有群集转换为使用RKE模板

_从v2.3.3开始提供_

本节介绍如何从现有集群创建RKE模板。

RKE模板不能应用于现有群集，除非将现有群集的设置保存为RKE模板。这会将群集的设置导出为新的RKE模板，并将群集绑定到该模板。结果是，只有更新[模板](/docs/admin settings/rke templates/creating and revising/#updateing-a-template)并将群集升级到[使用较新版本的模板](/docs/admin settings/rke templates/creating and revising/#upgrading-a-cluster to-use-a-new-template-revision)时，才能更改群集

要将现有群集转换为使用RKE模板，

1. 在Rancher中的**全局**视图中，单击**群集**选项卡。
1. 转到将转换为使用RKE模板的群集。单击**省略号(…)** > **另存为RKE模板**
1. 在出现的表单中输入模板的名称，然后单击**创建**

**结果:**

- 将创建一个新的RKE模板。
- 将转换群集以使用新模板。
- 可以[从新模板创建新群集](/docs/admin-settings/rke-templates/applying-templates/#creating-a-cluster-from-an-rke-template)