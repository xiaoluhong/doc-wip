---
xtitle: Amazon NLB配置
---

> #### **重要提示：Rancher v2.0.8之前仅支持RKE add-on安装**
>
> 请使用Rancher helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE add-on安装方法，参见[从带有RKE add-on组件的Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)获取有关如何使用Helm chart的详细信息。

### 目标

配置Amazon NLB是一个多阶段过程，我们将其分解为多个任务，使其容易理解。

1. [创建目标组](#create-target-groups)

   首先为**TCP**协议创建两个目标组，一个与TCP端口443有关，另一个与TCP端口80有关 (提供重定向到TCP端口443)，您将把Linux节点添加到这些组中。

2. [注册目标](#register-targets)

   将Linux节点添加到目标组

3. [创建NLB](#create-your-nlb)

   使用Amazon的向导创建网络负载均衡器。 作为此过程的一部分，您将添加在**1. 创建目标组**中创建的目标组。

### 创建目标组

您的第一个NLB配置步骤是创建两个目标组。从技术上讲，只需要端口443即可访问Rancher，但为端口80添加一个侦听器很方便，它将自动重定向到端口443。节点上的NGINX控制器将确保将端口80重定向到端口443。

登录[Amazon AWS控制台](https://console.aws.amazon.com/ec2/)开始，确保选择创建EC2实例(Linux节点)的**地区**。

目标组配置位于**EC2**服务的**负载平衡**部分。选择**服务**，然后选择**EC2**，找到**负载平衡**部分并打开**目标组**。

![EC2 Load Balancing section](/img/rancher/ha/nlb/ec2-loadbalancing.png")

单击**创建目标组**，创建关于TCP端口443的第一个目标组。

#### 目标组 (TCP端口443)

根据下表配置第一个目标组。该配置的屏幕截图显示在表格正下方。

| 选项                                 | 设置              |
| ----------------------------------- | ----------------- |
| 目标组名称                            | `rancher-tcp-443` |
| 协议                                 | `TCP`             |
| 端口                                 | `443`             |
| 目标类型                             | `实例`             |
| VPC                                 | 选择您的VPC        |
| 协议<br/>(运行状况检查)               | `HTTP`            |
| 路径<br/>(运行状况检查)               | `/healthz`        |
| 端口 (高级运行状况检查)                | `覆盖`,`80`       |
| 正常阈值 (高级运行状况检查)            | `3`               |
| 不正常阈值 (高级)                     | `3`               |
| 超时 (高级)                          | `6 秒`            |
| 间隔 (高级)                          | `10 秒`           |
| 成功代码                             | `200-399`         |

<hr />
**屏幕快照目标组TCP端口443设置**<br/>

![Target group 443](/img/rancher/ha/nlb/create-targetgroup-443.png")

<hr />
**屏幕快照目标组TCP端口443高级设置**<br/>

![Target group 443 Advanced](/img/rancher/ha/nlb/create-targetgroup-443-advanced.png")

<hr />

单击**创建目标组**，创建有关TCP端口80的第二个目标组。

#### 目标组 (TCP端口80)

根据下表配置第二个目标组。该配置的屏幕截图显示在表格正下方。

| 选项                                 | 设置             |
| ----------------------------------- | ---------------- |
| 目标组名称                            | `rancher-tcp-80` |
| 协议                                 | `TCP`            |
| 端口                                 | `80`             |
| 目标类型                             | `实例`            |
| VPC                                 | 选择您的VPC       |
| 协议<br/>(运行状况检查)               | `HTTP`           |
| 路径<br/>(运行状况检查)               | `/healthz`       |
| 端口 (高级运行状况检查)               | `流量端口`         |
| 正常阈值 (高级运行状况检查)            | `3`              |
| 不正常阈值 (高级)                    | `3`              |
| 超时 (高级)                         | `6 秒`            |
| 间隔 (高级)                         | `10 秒`           |
| 成功代码                            | `200-399`        |

<hr />
**屏幕快照目标组TCP端口80设置**<br/>

![Target group 80](/img/rancher/ha/nlb/create-targetgroup-80.png")

<hr />
**屏幕快照目标组TCP端口80高级设置**<br/>

![Target group 80 Advanced](/img/rancher/ha/nlb/create-targetgroup-80-advanced.png")

<hr />

### 注册目标

接下来，将Linux节点添加到两个目标组中。

选择名为**rancher-tcp-443**的目标组，单击选项卡**目标**并选择**编辑**。

![Edit target group 443](/img/rancher/ha/nlb/edit-targetgroup-443.png")

选择要添加的实例(Linux节点)，然后单击**添加到已注册**。

<hr />
**屏幕快照将目标添加到目标组TCP端口443**<br/>

![Add targets to target group 443](/img/rancher/ha/nlb/add-targets-targetgroup-443.png")

<hr />
**屏幕快照已将目标添加到目标组TCP端口443**<br/>

![Added targets to target group 443](/img/rancher/ha/nlb/added-targets-targetgroup-443.png")

实例添加后，单击屏幕右下方的**保存**。

重复这些步骤，将**rancher-tcp-443**替换为**rancher-tcp-80**，需要将相同的实例作为目标添加到该目标组。

### 创建NLB

使用Amazon的向导创建网络负载均衡器。作为此过程的一部分，您将添加在[创建目标组](#create-target-groups)中创建的目标组。

1.  在您的Web浏览器中，导航到[Amazon EC2控制台](https://console.aws.amazon.com/ec2/)。

2.  在导航栏中, 选择**负载平衡**>**负载均衡器**。

3.  单击**创建负载均衡器**。

4.  选择**网络负载均衡器**，然后单击**创建**。

5.  完成**步骤 1: 配置负载均衡器**表单。

    - **基本配置**

      - 名称: `rancher`
      - 模式: `internet-facing`

    - **侦听器**

          	在下面添加**负载均衡器协议**和**负载均衡器端口** 。
          	- `TCP`: `443`

    - **可用区**

      - 选择您的**VPC**和**可用区**。

6.  完成**步骤 2: 配置路由**表单。

    - 从**目标组**下拉列表中，选择**现有目标组**。

    - 从**名称**下拉列表中，选择`rancher-tcp-443`。

    - 打开**高级运行状况检查设置**，然后将**时间间隔**配置为`10 秒`。

7.  完成**步骤 3: 注册目标**。由于您之前注册了目标，因此您只需单击**下一步: 审核**。

8.  完成**步骤 4: 审核**。查看负载均衡器的详细信息，并在满意时单击**创建**。

9.  AWS创建NLB之后，单击**关闭**。

### 为NLB添加TCP端口80的侦听器

1. 选择新创建的NLB，然后选择我**侦听器**选项卡。

2. 单击**添加侦听器**。

3. 使用`TCP`:`80`作为**协议**:**端口**。

4. 单机**添加操作**，然后选择**转发至...**。

5. 从**转发至**的下拉列表中选择`rancher-tcp-80`。

6. 单击屏幕右上方的**保存**。
