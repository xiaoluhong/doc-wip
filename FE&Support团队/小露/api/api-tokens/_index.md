---
title: API Tokens
---

默认情况下，一些集群级别的API Tokens是永久有效(`ttl=0`)，除非删除它们，否则用不失效，并且API Tokens不会因更改用户密码而失效。

您可以通过删除API Tokens或禁用用户来禁用它们。

删除 API Tokens:

1. 访问Rancher API视图中的所有Tokens列表：`https://<Rancher-Server-IP>/v3/tokens`.

1. 通过要删除的Tokens ID访问，例如：`https://<Rancher-Server-IP>/v3/tokens/kubectl-shell-user-vqkqt`

1. 点击**Delete.**

以下是用`ttl=0`生成的Tokens完整列表:

| Token             | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `kubeconfig-*`    | Kubeconfig Token                                             |
| `kubectl-shell-*` | web shell执行`kubectl`命令Token                          |
| `agent-*`         | agent部署Token                                               |
| `compose-token-*` | Token for compose                                            |
| `helm-token-*`    | Helm chart部署Token                                          |
| `*-pipeline*`     | 项目Pipeline Token                                           |
| `telemetry-*`     | Telemetry Token                                              |
| `drain-node-*`    | Token for drain (we use `kubectl` for drain because there is no native Kubernetes API) |
