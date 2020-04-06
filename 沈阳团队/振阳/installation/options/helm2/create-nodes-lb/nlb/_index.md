---
title: Amazon NLB
---

### 目标

配置Amazon NLB是一个多阶段过程。 我们将其细分为多个任务以便于理解。

1. [创建目标群组](#create-target-groups)

   首先为 **TCP** 协议创建两个目标群组，一个是TCP端口443，一个是TCP端口80（提供到TCP端口443的重定向）。您将添加您的Linux节点到这些组中。

2. [注册目标](#register-targets)

   添加您的Linux节点到这些目标群组中。

3. [创建NLB](#create-your-nlb)

   使用亚马逊的向导创建网络负载平衡器。 作为此过程的一部分，您将添加在 **1. 创建的目标群组** 中创建的目标群组。

> **注意事项:** 当以端口443的`tcp`模式而非`tls`模式终止流量时，Rancher仅支持使用Amazon NLB。 这是由于以下原因：当在NLB终止时，NLB不会将正确的标头注入请求中。 这意味着，如果希望使用由Amazon证书管理器(ACM)管理的证书时，应该使用ELB或ALB。

### 创建目标群组

第一个NLB配置步骤是创建两个目标群组。 从技术上讲，只需要端口443即可访问Rancher，但为了提升易用性通常会为端口80添加侦听器，该端口将自动重定向到端口443。 节点上的NGINX ingress controller将确保将端口80重定向到端口443。

登陆 [亚马逊 AWS 控制台](https://console.aws.amazon.com/ec2/) , 确保选择指定的 **Region** 来创建您的EC2实例（Linux主机）。

目标群组配置位于**EC2** **服务** **负载平衡** 部分中。选择 **服务** 然后选择 **EC2**, 找到 **负载平衡** 然后选择 **目标群组**。

![EC2负载平衡部分](/img/rancher/ha/nlb/ec2-loadbalancing.png)

点击 **创建目标群组** 来创建第一个关于TCP端口443的目标组。

#### 目标群组 (TCP 端口 443)

根据下表配置第一个目标组。 表格下方显示了配置的屏幕截图。

| Option                              | Setting           |
| ----------------------------------- | ----------------- |
| Target Group Name                   | `rancher-tcp-443` |
| Protocol                            | `TCP`             |
| Port                                | `443`             |
| Target type                         | `instance`        |
| VPC                                 | Choose your VPC   |
| Protocol<br/>(Health Check)         | `HTTP`            |
| Path<br/>(Health Check)             | `/healthz`        |
| Port (Advanced health check)        | `override`,`80`   |
| Healthy threshold (Advanced health) | `3`               |
| Unhealthy threshold (Advanced)      | `3`               |
| Timeout (Advanced)                  | `6 seconds`       |
| Interval (Advanced)                 | `10 second`       |
| Success codes                       | `200-399`         |

<hr />
**屏幕快照: 目标群组TCP端口443设置**<br/>

![目标组 443](/img/rancher/ha/nlb/create-targetgroup-443.png)

<hr />
**屏幕快照: 目标群组TCP端口443高级设置**<br/>

![目标组 443 高级](/img/rancher/ha/nlb/create-targetgroup-443-advanced.png)

<hr />

点击 **创建目标组** 来创建第二个关于TCP端口80的目标组。

#### 目标群组 (TCP 端口 80)

根据下表配置第二个目标组。 表格下方显示了配置的屏幕截图。

| Option                              | Setting          |
| ----------------------------------- | ---------------- |
| Target Group Name                   | `rancher-tcp-80` |
| Protocol                            | `TCP`            |
| Port                                | `80`             |
| Target type                         | `instance`       |
| VPC                                 | Choose your VPC  |
| Protocol<br/>(Health Check)         | `HTTP`           |
| Path<br/>(Health Check)             | `/healthz`       |
| Port (Advanced health check)        | `traffic port`   |
| Healthy threshold (Advanced health) | `3`              |
| Unhealthy threshold (Advanced)      | `3`              |
| Timeout (Advanced)                  | `6 seconds`      |
| Interval (Advanced)                 | `10 second`      |
| Success codes                       | `200-399`        |

<hr />
**屏幕快照: 目标群组TCP端口80设置****<br/>

![目标群组 80](/img/rancher/ha/nlb/create-targetgroup-80.png)

<hr />
**屏幕快照: 目标群组TCP端口443高级设置**<br/>

![目标群组 80 高级](/img/rancher/ha/nlb/create-targetgroup-80-advanced.png)

<hr />

### 注册目标

下一步，为目标群组添加Linux节点。

选择 **rancher-tcp-443** 的目标群组，点击 **目标** 选项卡然后选择 **编辑**。

![编辑目标群组 443](/img/rancher/ha/nlb/edit-targetgroup-443.png)

选择您想添加的实例（Linux节点）, 点击 **添加到已注册**。

<hr />

**屏幕快照: 添加目标到TCP端口443目标群组**<br/>

![添加目标到TCP端口443目标群组](/img/rancher/ha/nlb/add-targets-targetgroup-443.png)

<hr />

**屏幕快照: 向目标组TCP端口443添加了目标**<br/>

![向目标组TCP端口443添加了目标](/img/rancher/ha/nlb/added-targets-targetgroup-443.png)

添加实例后，单击屏幕右下角的`保存`。

重复这些步骤， 将 **rancher-tcp-443** 替换成 **rancher-tcp-80**。 需要将相同的实例作为目标添加到该目标组。

### 创建您的NLB

使用Amazon的向导创建网络负载均衡器。 作为此过程的一部分，您将添加在[创建目标组](#create-target-groups)中已创建的目标组。 

1.  在Web浏览器中，导航至 [亚马逊 EC2 控制台](https://console.aws.amazon.com/ec2/)。

2.  在导航窗格中，选择 **负载平衡** > **负载均衡器**。

3.  点击 **创建负载均衡器**。

4.  选择 **网络负载均衡器** 点击 **创建**。

5.  完成 **步骤1: 配置负载均衡器** 表单。

    - **基本配置**

      - 名称: `rancher`
      - 模式: `内部` or `面向internet`

        您为NLB选择的模式取决于实例/VPC的配置。 如果您的实例没有与之关联的公共IP，或者您将仅在内部访问Rancher，则应将NLB方案设置为`内部`，而不是`面向internet`。

    - **侦听器**

          	添加如下的 **负载均衡器协议** 和 **负载均衡器端口**。
          	- `TCP`: `443`

    - **可用区**

      - 选择您的 **VPC** 和 **可用区**。

6.  完成 **步骤2: 配置路由** 表单。

    - 从 **目标组** 下拉列表中选择 **已经存在的目标组**。

    - 从 **名称** 下拉框中选择 `rancher-tcp-443`。

    - 打开 **高级运行状况检查设置**， 配置 **间隔** 为 `10 秒`。

7.  完成 **步骤3: 注册目标**。由于您较早注册了目标，因此只需点击 **下一步: 审核**。

8.  完成 **步骤4: 审核**。 查看并检查负载均衡器的详细信息，确认无误后单击`创建`。

9.  完成创建NLB步骤后，点击 **关闭**。

### 为TCP端口80添加到NLB的侦听器

1. 选择您刚创建的NLB然后点击 **侦听器** 选项卡。

2. 点击 **添加侦听器**。

3. 使用 `TCP`:`80` 作为 **协议** : **端口**。

4. 点击 **添加操作** 并选择 **转发至...**。

5. 根据 **转发至** 下拉框, 选择 `rancher-tcp-80`。

6. 点击屏幕右上角的 **保存**。
