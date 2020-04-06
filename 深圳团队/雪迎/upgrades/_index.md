---
title: 升级和回滚
---

#### 升级 Rancher

- [升级](/docs/upgrades/upgrades/)

#### 回滚不成功的升级

如果您的Rancher Server没有成功升级, 您可以回滚到升级之前的版本：

- [回滚使用Docker安装的Rancher](/docs/upgrades/single-node-rollbacks)
- [回滚安装在Kubernetes集群上的Rancher](/docs/upgrades/ha-server-rollbacks)


> **注意:** 如果要在以下两种情况下回滚到版本，则必须遵循一些额外的 [说明](/docs/upgrades/rollbacks/) 才能使集群正常工作。
>
> - 从v2.1.6 +回滚到v2.1.0-v2.1.5或v2.0.0-v2.0.10之间的任何版本。
> - 从v2.0.11 +回滚到v2.0.0-v2.0.10之间的任何版本。
