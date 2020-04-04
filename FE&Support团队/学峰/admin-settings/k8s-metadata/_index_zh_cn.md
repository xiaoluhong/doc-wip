---
标题: 升级Kubernetes而不升级Rancher
---

_从2.3.0版起提供_

RKE元数据特性允许您在发布新版本的Kubernetes后立即为集群提供它们，而无需升级Rancher。此功能对于利用Kubernetes的补丁版本非常有用，例如，如果您希望在Rancher服务器最初支持v1.14.6时升级到Kubernetes v1.14.7。

> **注意:** Kubernetes API可以在次要版本之间更改。因此，我们不支持引入次要的Kubernetes版本，例如在Rancher当前支持v1.14时引入v1.15。您需要升级Rancher以添加对次要Kubernetes版本的支持。

Rancher的Kubernetes元数据包含特定于Rancher用于提供[RKE clusters](/docs/cluster-provisioning/rke-clusters/)。Rancher定期同步数据，并为**系统映像** **服务选项**和**加载项模板**创建自定义资源定义(CRD)因此，当新的Kubernetes版本与Rancher服务器版本兼容时，Kubernetes元数据使Rancher可以使用新版本来配置群集。元数据概述了[Rancher Kubernetes Engine]({{<baseurl>}/rke/latest/en/)(rke)用于部署各种Kubernetes版本的信息。

下表描述了受定期数据同步影响的CRD。

> **注意:** 只有管理员可以编辑元数据CRD。除非明确建议，否则建议不要更新现有对象。

| 资源 | 描述 | RancherAPI URL |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| 系统映像 | 用于通过RKE部署Kubernetes的系统映像列表。 | `<RANCHER_SERVER_URL>/v3/rkek8ssystemimages` |
| 服务选项 | 传递给Kubernetes组件的默认选项，如`kube api`、`scheduler`、`kubelet`、`kube proxy`和`kube controller manager`等 |
| 加载项模板 | 用于部署加载项组件的YAML定义，如Canal、Calico、Flannel、Weave、Kube dns、CoreDNS、`metrics server`、`nginx ingres`  | `<RANCHER_server_URL>/v3/rkeaddons` |

管理员可以将RKE元数据设置配置为执行以下操作:

- 刷新Kubernetes元数据，如果一个新的Kubernetes补丁版本出来了，他们希望Rancher在不升级Rancher的情况下为集群提供最新版本的Kubernetes
- 更改Rancher用于同步元数据的元数据URL，如果需要本地同步Rancher而不是与GitHub同步，这对于气隙设置非常有用
- 防止Rancher自动同步元数据，这是防止Rancher中提供新的和不受支持的Kubernetes版本的一种方法

#### 刷新Kubernetes元数据

默认情况下，刷新Kubernetes元数据的选项可供管理员使用，也可供具有**管理群集驱动程序**[全局角色]的任何用户使用(/docs/admin settings/rbac/global permissions/)

要强制Rancher刷新Kubernetes元数据，可以在右侧的**工具>驱动程序>刷新Kubernetes元数据**下执行手动刷新操作。

#### 配置元数据同步

> 只有管理员可以更改这些设置。

RKE元数据配置控制Rancher同步元数据的频率以及从何处下载数据。您可以从Rancher UI中的设置或通过端点`v3/settings/rke metadata config`中的Rancher API配置元数据。

要在Rancher中编辑元数据配置，

一。转到**全局**视图并单击**设置**选项卡。
一。转到**rke元数据配置**部分。单击**省略号(…)**并单击**编辑**
一。您可以选择填写以下参数:

- `refresh interval minutes`:这是Rancher等待同步元数据的时间量。若要禁用定期刷新，请将`刷新间隔分钟数`设置为0。
- `url`:这是Rancher从中获取数据的HTTP路径。
- `branch`:如果URL是Git URL，则指Git分支名称。

如果没有气隙设置，则不需要指定Rancher获取元数据的URL或Git分支，因为默认设置是从[Rancher的元数据Git存储库]中提取(https://github.com/Rancher/kontaner-driver-metadata.Git)

但是，如果您有[离线设置需求](#air-gap-setups)，则需要将Kubernetes元数据存储库镜像到Rancher可用的位置。然后需要在`rke metadata config`设置中更改URL和Git分支，以指向存储库的新位置。
#### 离线设置

如果Rancher服务器的当前版本支持新的Kubernetes版本元数据，则Rancher依赖于`rke metadata config`的定期刷新来下载该元数据。有关兼容的Kubernetes和Rancher版本的表，请参阅[服务条款部分](https://Rancher.com/support-maintenance-terms/all-supported-versions/Rancher-v2.2.8/)

如果设置了air gap，则可能无法从Rancher的Git存储库中自动定期刷新Kubernetes元数据。在这种情况下，应该禁用定期刷新以防止日志显示错误。或者，您可以配置元数据设置，以便Rancher可以与RKE元数据的本地副本同步。

若要将Rancher与RKE元数据的本地镜像同步，管理员将通过更新`url`和`branch`以指向镜像来配置`RKE元数据配置`设置。

在将新的Kubernetes版本加载到Rancher设置中之后，需要执行其他步骤才能使用它们启动集群。Rancher需要访问更新的系统图像。虽然元数据设置只能由管理员更改，但任何用户都可以下载Rancher系统映像并为它们准备一个私有Docker注册表。

1. 要下载私有注册表的系统映像，请单击Rancher用户界面左下角的Rancher服务器版本。
1. 下载Linux或Windows操作系统特定的映像列表。
1. 下载`rancher images.txt`。
1. 在[air gap install](/docs/installation/other-installation-methods/air-gap/populate-private-registry)过程中使用相同的步骤准备私有注册表，但不要使用releases页面中的`rancher images.txt`，而是使用从前面步骤获得的步骤。

**结果:** Rancher的气隙安装现在可以同步Kubernetes元数据。如果在新版本的Kubernetes发布时更新私有注册表，则无需升级Rancher就可以使用新版本提供集群。