---
标题: Amazon ALB配置
---

> #### **重要说明: Rancher v2.0.8之前仅支持RKE附加安装**
>
> 请使用Rancher Helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE附加组件安装方法, 请参阅[使用RKE附加组件从Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)以获取有关如何使用Helm chart的详细信息。

### 目标

配置Amazon ALB是一个多阶段过程。我们将其分解为多个任务，因此更易于理解。

1. [创建目标组](#create-target-group)

   首先为http协议创建一个目标组。您将Linux节点添加到该组中。

2. [注册目标](#register-targets)

   将Linux节点添加到目标组。

3. [创建您的ALB](#create-your-alb)

   使用Amazon的Wizard创建应用程序负载平衡器。作为此过程的一部分，您将添加在**1. 创建目标组**中创建的目标组。

### 创建目标组

您的第一个ALB配置步骤是为HTTP创建一个目标组。

登录到[Amazon AWS Console](https://console.aws.amazon.com/ec2/)控制台开始使用。

以下文档将指导您完成此过程。使用下表中的数据来完成该过程。

[Amazon文档：创建目标组](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-target-group.html)

#### 目标组 (HTTP)

| 选项               | 设置              |
| ----------------- | ----------------- |
| 目标组名称          | `rancher-http-80` |
| 协议               | `HTTP`            |
| 端口               | `80`              |
| 目标类型            | `instance`        |
| VPC                | 选择您的VPC        |
| 协议<br/>(健康检查)  | `HTTP`            |
| 路径<br/>(健康检查)  | `/healthz`        |

### 注册目标

接下来，将Linux节点添加到目标组。

[Amazon文档：向您的目标组注册目标](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-register-targets.html)

#### 创建您的ALB

使用Amazon的Wizard创建应用程序负载平衡器。作为此过程的一部分，您将添加在[创建目标组](#create-target-group)中创建的目标组。

1.  在您的Web浏览器中，访问[Amazon EC2 Console](https://console.aws.amazon.com/ec2/)。

2.  在页面窗口中, 选择 **LOAD BALANCING** > **负载均衡器**.

3.  点击 **创建负载均衡器**.

4.  选择 **应用程序负载均衡器**.

5.  完成 **步骤1：配置负载均衡器** 表单。

    - **基本配置**

      - 名称: `rancher-http`
      - 模式: `面向 internet`
      - IP地址类型: `ipv4`

    - **侦听器**

          	在下面添加 **负载均衡器协议** 和 **负载均衡器端口** 。
          	- `HTTP`: `80`
          	- `HTTPS`: `443`

    - **可用区**

      - 选择您的 **VPC** 和 **可用区**。

6.  完成 **步骤2：配置安全设置** 表单。

    配置要用于SSL termination的证书。

7.  完成 **步骤3：配置安全组** 表单。

8.  完成 **步骤4：配置路由** 表单。

    - 从 **目标组** 下拉列表中，选择 **现有目标组**。

    - 添加目标组 `rancher-http-80`.

9.  完成 **步骤5：注册目标**。 由于您早先注册了目标，因此您只需单击 **下一步: 审核**。

10. 完成 **步骤6：检查**。 查看负载均衡器的详细信息，并在满意时单击 **创建**。

11. AWS创建ALB之后，单击 **关闭**。
