---
标题：配置全局默认专用镜像仓库
---

您可能想要使用私有Docker镜像仓库在组织内共享您的自定义基础镜像。借助私有镜像仓库，您可以为集群中使用的Docker镜像保留私有，一致且集中的真实来源。

在Rancher中设置私有镜像仓库的主要方法有两种：通过全局视图中的`设置`选项卡设置全局默认镜像仓库，以及在群集级别设置的高级选项中设置私有镜像仓库。全局默认镜像仓库旨在用于离线环境设置，不需要证书的镜像仓库。群集级专用镜像仓库旨在用于所有需要专用凭据的设置。

本部分是关于配置全局默认私有镜像仓库的，并且重点介绍在安装Rancher之后如何从Rancher UI配置镜像仓库。

有关在Rancher的安装过程中使用命令行选项设置私有镜像仓库的说明，请参阅[离线Docker安装](/docs/installation/air-gap-single-node)或[离线Kubernetes安装](/docs/installation/air-gap-high-availability)说明。

如果您的私人镜像仓库需要凭据，则不能将其用作默认镜像仓库。没有全局的方法来为每个Rancher所配置的群集设置具有授权的私有镜像仓库。因此，如果您希望由Rancher配置的群集从具有凭据的私有镜像仓库中提取图像，则必须[通过高级群集选项传递镜像仓库凭据](＃provisioning-clusters-with-private-registries-that-每次创建新集群时都需要(require-credentials)。

## 将没有凭据的专用镜像仓库设置为默认镜像仓库

1. 登录到Rancher并配置默认的管理员密码。

2. 进入`设置`视图。

   ！[设置](/img/rancher/airgap/settings.png`)

3. 查找名为`system-default-registry`的设置，然后选择`编辑`。

   ！[编辑](/img/rancher/airgap/edit-system-default-registry.png`)

4. 将该值更改为您的镜像仓库(例如`registry.yourdomain.com：port`)。不要在镜像仓库前面加上`http://`或`https://`。

   ！[保存](/img/rancher/airgap/enter-system-default-registry.png`)

**结果：** Rancher将使用您的私有镜像仓库提取系统镜像。

## 在部署群集时使用凭据设置私有镜像仓库

使用Rancher设置群集时，可以按照以下步骤配置私有镜像仓库：

1. 通过Rancher UI创建集群时，请转到`集群选项`部分，然后单击`显示高级选项`。
2. 在<b>启用专用注册中心</b>部分中，单击 **启用。**
3. 输入镜像仓库URL和凭据。
4. 点击 **保存。**

**结果：** 新群集将能够从专用镜像仓库中提取图像。
