---
title: 为离线安装设置本地系统图表
---

该[系统图表](https://github.com/rancher/system-charts) 存储库包含监控、日志、告警和全局DNS等功能所需的所有应用项。

在Rancher的离线安装中，您将需要配置Rancher以使用系统图表的本地副本。本节介绍如何在Rancher v2.3.0中使用CLI标志以及在v2.3.0之前的Rancher版本中使用Git镜像来使用本地系统图表。

## 在Rancher v2.3.0中使用本地系统图表

在Rancher v2.3.0中，`system-charts`的本地副本已打包到`rancher/rancher`容器中。为了能够在离线安装中使用这些功能，您将需要运行带有额外环境变量`CATTLE_SYSTEM_CATALOG=bundled`的Rancher安装命令，该变量将告诉Rancher使用图表的本地副本，而不是尝试从GitHub上获取它们。

[Docker离线安装说明](/docs/installation/air-gap-single-node/install-rancher)和[Kubernetes离线安装说明](/docs/installation/air-gap-high-availability/install-rancher/#c-install-rancher)中包含用于带有捆绑的`system-charts`的Rancher安装的示例命令。

## 在v2.3.0之前为Rancher设置系统图表

#### A. 准备系统图表

该[系统图表](https://github.com/rancher/system-charts) 存储库包含监控、日志、告警和全局DNS等功能所需的所有应用项。为了能够在离线安装中使用这些功能，您将需要将`system-charts`存储库镜像到网络中Rancher可以到达的位置，并配置Rancher来使用该存储库。

请参阅`system-charts`存储库中的发行说明，以查看哪个分支对应于您的Rancher版本。

#### B. 配置系统图表

需要将Rancher配置为使用system-charts存储库的Git镜像。您可以从Rancher UI或Rancher的API视图配置系统图表存储库。

 tabs 
 tab "Rancher UI" 

在Rancher UI的商店设置页面中，请按照下列步骤操作：

1. 进入**全局**视图。

1. 单击**工具>商店设置**。

1. 系统图表以名称`system-library`显示。要编辑系统图表的配置，请点击**省略号 (...) > 升级**。

1. 在**商店URL地址**字段中，输入`system-charts`存储库的Git镜像的位置。

1. 单击**保存**。

**结果：** Rancher配置为从您的`system-charts`存储库下载所有必需的应用项。

 /tab 
 tab "Rancher API" 

1. 登录到Rancher。

1. 在浏览器中打开`https://<your-rancher-server>/v3/catalogs/system-library`。

   ![Open](/img/rancher/airgap/system-charts-setting.png)

1. 单击右上角的**Edit**，然后将**url**值的位置更新为`system-charts`存储库的Git镜像。

   ![Update](/img/rancher/airgap/system-charts-update.png)

1. 单击**Show Request**

1. 单击**Send Request**

**结果：** Rancher配置为从您的`system-charts`存储库下载所有必需的应用项。

 /tab 
 /tabs 

