---
title: Rancher 部署策略
---

有两种推荐的部署策略，每种方法都有其优缺点。哪种方法最适合你的场景，请阅读更多信息:

- [Hub and Spoke](#hub-and-spoke)
- [Regional](#regional)

## Hub & Spoke Strategy

---

在这个部署场景中，有一个Rancher控制平面来管理遍布全球的Kubernetes集群。控制平面将在一个高可用性的Kubernetes集群上运行，由于延迟将会产生影响。

![Hub and Spoke Deployment](/img/rancher/bpg/hub-and-spoke.png)

### 优点

- 环境可以具有跨区域的节点和网络连接。
- 单一控制平面界面，查看/查看所有区域和环境。
- Kubernetes不需要Rancher操作，并且可以容忍失去与牧场主控制平面的连接。

### 缺点

- 受制于网络延迟。
- 如果控制平面失效，新服务的全局供应在恢复之前不可用。但是，每个Kubernetes集群可以继续单独管理。

## Regional Strategy

---

在区域部署模型中，控制平面被部署在靠近计算节点的地方。

![Regional Deployment](/img/rancher/bpg/regional.png)

### 优点

- 如果另一个区域的控制平面发生故障，则区域内的Rancher功能将保持运行状态。
- 网络延迟大大降低，提高Rancher的性能。
- Rancher控制平面的升级可以在每个区域独立完成。

### 缺点

- 管理多个Rancher安装的开销。
- Visibility across global Kubernetes clusters requires multiple interfaces/panes of glass.（待讨论）
- 在Rancher中部署多集群应用程序需要为每个Rancher Server重复这个过程。
