---
title: Kubernetes 回滚
---

如果升级Rancher且升级未成功完成，则可能需要将Rancher Server回滚到最后的正常状态。

要还原Rancher，请遵循此处详述的过程：[还原备份-Kubernetes安装](/docs/backups/restorations/ha-restoration)

还原Rancher Server集群的快照会将Rancher还原到快照时的版本和状态。

> **注意:** 托管集群对其状态具有权威性。这意味着在还原快照后，还原rancher server将不会还原工作负载部署或在托管集群上所做的更改。
