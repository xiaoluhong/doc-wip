---
标题: RKE模板
---

_从Rancher v2.3.0开始提供_

RKE模板旨在允许DevOps和安全团队标准化和简化Kubernetes集群的创建。

RKE是[Rancher-Kubernetes引擎，]({{{<baseurl>}}/rke/latest/en/)它是Rancher用来提供Kubernetes集群的工具。

随着Kubernetes越来越受欢迎，有一种趋势是管理更多的小型集群。当您想要创建许多集群时，一致地管理它们更为重要。多集群管理面临着强制实施安全性和附加配置的挑战，在将集群移交给最终用户之前，这些配置需要标准化。

RKE模板有助于标准化这些配置。无论集群是使用Rancher UI、Rancher API还是自动化流程创建的，Rancher都将保证它从RKE模板提供的每个集群在生成方式上是统一和一致的。

管理员控制最终用户可以更改哪些群集选项。RKE模板也可以与特定的用户和组共享，这样管理员可以为不同的用户集创建不同的RKE模板。

如果使用RKE模板创建集群，则不能将其更改为其他RKE模板。只能将群集更新为同一模板的新版本。

从Rancher v2.3.3开始，您可以[将现有群集的配置另存为RKE模板](/docs/admin-settings/rke-templates/applying-templates/#converting-an-existing-cluster-to-use-an-rke-template)，则只有更新模板后，才能更改群集的设置。新模板还可以用于启动新的集群。

RKE模板的核心功能允许DevOps和安全团队:

- 标准化集群配置并确保按照最佳实践创建由Rancher提供的集群
- 防止技术水平较低的用户在配置群集时做出不知情的选择
- 与不同的用户和组集共享不同的模板
- 将模板的所有权委托给受信任的用户进行更改
- 控制哪些用户可以创建模板
- 要求用户从模板创建群集

## 可配置设置

RKE模板可以在Rancher用户界面中创建，也可以用YAML格式定义。它们可以定义在使用Rancher从基础结构提供程序提供自定义节点或节点时可以指定的所有相同参数:

- 云提供程序选项
- Pod安全选项
- 网络提供商
- 入口控制器
- 网络安全配置
- 网络插件
- 专用注册表URL和凭据
- 附加组件
- Kubernetes选项，包括Kubernetes组件的配置，如kube api、kube controller、kubelet和services

RKE模板的[附加组件部分](#add-ons)功能特别强大，因为它允许多种自定义选项。

## RKE模板范围

Rancher提供的集群支持RKE模板。模板可用于提供由基础结构提供商启动的自定义群集或群集。

RKE模板用于定义Kubernetes和Rancher设置。节点模板负责配置节点。有关如何将RKE模板与硬件结合使用的提示，请参阅[RKE模板和硬件](/docs/admin settings/RKE templates/RKE templates and hardware)。

可以从头开始创建RKE模板来预先定义集群配置。它们可以应用于启动新的集群，也可以从现有运行的集群导出模板。

从v2.3.3开始，现有群集的设置可以[保存为RKE模板](/docs/admin-settings/rke-templates/applying-templates/#converting-an-existing-cluster-to-use-an-rke-template)这将创建新模板并将群集设置绑定到模板，因此，只有更新[模板](/docs/admin settings/rke templates/creating and revising/#updateing-a-template)时才能升级集群，并且集群升级到[使用模板的新版本](/docs/admin settings/rke templates/creating and revising/#upgrading-a-cluster-to-use-a-new-template-revision)新模板也可以用于创建新集群。

## 示例场景

当组织同时拥有基本和高级Rancher用户时，管理员可能希望为高级用户提供更多创建群集的选项，同时限制基本用户的选项。

这些[示例场景](/docs/admin-settings/rke-templates/example-scenarios)描述了组织如何使用模板来标准化集群创建。

一些示例场景包括:

- **强制模板:** 如果管理员希望所有新的Rancher配置的群集都具有这些设置，则他们可能希望[强制每个人使用一个或多个模板设置](/docs/admin-settings/rke-templates/example-scenarios/#enforcing-a-template-setting-for-everyone)。
- **与不同的用户共享不同的模板:** 管理员可以给[基本用户和高级用户提供不同的模板，](/docs/admin-settings/rke-templates/example-scenarios/#templates-for-basic-and-advanced-users)，这样基本用户可以有更多受限的选项，高级用户在创建群集时可以更谨慎地使用。
- **更新模板设置:** 如果组织的安全和DevOps团队决定将最佳实践嵌入到新集群所需的设置中，则这些最佳实践可能会随时间而改变。如果最佳实践发生变化，[模板可以更新为新版本](/docs/admin-settings/rke-templates/example-scenarios/#updating-templates-and-clusters-created-with-them)并且从模板创建的集群可以[升级到新版本](/docs/admin settings/rke templates/creating and revising/#upgrading- a- cluster- to- use- a- new- template- revision)模板。
- **共享模板的所有权:** 当模板所有者不再希望维护模板或希望共享模板的所有权时，此场景描述如何[共享模板所有权](/docs/admin-settings/rke-templates/example-scenarios/#allowing-other-users-to-control-and-share-a-template)

## 模板管理

创建RKE模板时，可以在Rancher UI中的**工具 > RKE模板**下的**全局**视图中找到它。**创建模板时，您将成为模板所有者，这将授予您修改和共享模板的权限。您可以与特定用户或组共享RKE模板，也可以将其公开。

管理员可以启用模板强制，以要求用户在创建群集时始终使用RKE模板。这允许管理员保证Rancher总是为集群提供特定的设置。

RKE模板更新通过修订系统处理。如果要更改或更新模板，请创建模板的新修订版。然后，可以将使用旧版本模板创建的群集升级到新模板修订版。

在RKE模板中，可以将设置限制为模板所有者选择的内容，也可以打开设置以供最终用户选择值。创建模板时，Rancher UI中的每个设置上的**允许用户覆盖**切换指示了差异。

对于无法覆盖的设置，最终用户将无法直接编辑它们。为了让用户获得这些设置的不同选项，RKE模板所有者需要创建RKE模板的新版本，这将允许用户升级和更改该选项。

本节中的文件解释了RKE模板管理的细节:

- [获取创建模板的权限](/docs/admin-settings/rke-templates/creator-permissions/)
- [创建和修改模板](/docs/admin-settings/rke-templates/creating-and-revising/)
- [强制模板设置](/docs/admin-settings/rke-templates/enforcement/#requiring-new-clusters-to-use-a-cluster-template)
- [覆盖模板设置](/docs/admin-settings/rke-templates/overrides/)
- [与群集创建者共享模板](/docs/admin-settings/rke-templates/template-access-and-sharing/#sharing-templates-with-specific-users)
- [共享模板所有权](/docs/admin-settings/rke-templates/template-access-and-sharing/#sharing-ownership-of-templates)

提供[模板的示例YAML配置文件](/docs/admin-settings/rke-templates/example-yaml)以供参考。

## 应用模板

您可以[从模板创建群集](/docs/admin-settings/rke-templates/applying-templates/#creating-a-cluster-from-a-cluster-template)创建的群集，也可以从[与您共享的模板](/docs/admin settings/rke templates/template access and sharing)

如果RKE模板所有者创建了模板的新版本，则可以[将群集升级到该版本](/docs/admin-settings/rke-templates/applying-templates/#updating-a-cluster-created-with-an-rke-template)

可以从头开始创建RKE模板来预先定义集群配置。它们可以应用于启动新的集群，也可以从现有运行的集群导出模板。

从Rancher v2.3.3开始，您可以[将现有群集的配置另存为RKE模板](/docs/admin-settings/rke-templates/applying-templates/#converting-an-existing-cluster-to-use-an-rke-template)，则只有更新模板后，才能更改群集的设置。

## 标准化硬件

RKE模板设计用于标准化Kubernetes和Rancher设置。如果您还想标准化您的基础设施，可以使用RKE模板[与其他工具一起](/docs/admin-settings/rke-templates/rke-templates-and-hardware)。

## YAML定制

如果将RKE模板定义为YAML文件，则可以修改此[示例RKE模板YAML](/docs/admin-settings/rke-templates/example-yaml)。RKE模板中的YAML使用的自定义项与Rancher在创建RKE群集时使用的自定义项相同，但由于YAML位于Rancher提供的群集的上下文中，因此需要将RKE模板自定义项嵌套在YAML中的`Rancher_kubernetes_engine_config`指令下。

RKE文档还有[注释的]({{<baseurl>}}/rke/latest/en/example-yamls/)`cluster.yml`文件，您可以使用这些文件作为参考。
有关可用选项的指导，请参阅[群集配置]({{<baseurl>}}/rke/latest/en/config-options/)上的RKE文档。

#### 附加组件

RKE模板配置文件的加载项部分的工作方式与[群集配置文件的加载项部分]({{<baseurl>}}/rke/latest/en/config-options/add-ons/)相同。

用户定义的add-ons指令允许您调出并下拉Kubernetes清单，或者直接将它们内联。如果您将这些清单作为RKE模板的一部分，Rancher将在集群中提供这些清单。

您可以使用附加组件执行以下操作:

- 启动后在Kubernetes集群上安装应用程序
- 在使用Kubernetes守护程序集部署的节点上安装插件
- 自动设置命名空间、服务帐户或角色绑定

RKE模板配置必须嵌套在`rancher_kubernetes_engine_config`指令中。要设置加载项，在创建模板时，您将单击**Edit as YAML.*，然后使用`add ons`指令添加清单，或使用`addons_include`指令设置用于加载项的YAML文件。有关自定义加载项的详细信息，请参阅[用户定义的加载项文档]({{<baseurl>}}/rke/latest/en/config-options/add-ons/user-defined-add-ons/)