---
title: TLS 设置
---

_自v2.1.7起可用_

在Rancher v2.1.7中，默认TLS配置已更改为仅接受TLS 1.2和安全TLS密码套件。不支持TLS 1.3和TLS 1.3专用密码套件。

### 配置 TLS 设置

通过将环境变量传递到Rancher服务器容器来启用和配置审计日志。请参阅以下内容以启用安装。

- [使用Docker在单个节点上安装Rancher](/docs/installation/other-installation-methods/single-node-docker/#tls-settings)

- [在Kubernetes上安装Rancher](/docs/installation/options/chart-options/#tls-settings)

### TLS 设置

| 参数                | 描述               | 默认值                                                                                                                                                                                                                                                                      | 可用选项                                                            |
| ------------------------ | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `CATTLE_TLS_MIN_VERSION` | 最低TLS版本       | `1.2`                                                                                                                                                                                                                                                                        | `1.0`, `1.1`, `1.2`                                                          |
| `CATTLE_TLS_CIPHERS`     | 允许的TLS密码套件 | `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,`<br/>`TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,`<br/>`TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,`<br/>`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,`<br/>`TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,`<br/>`TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305` | 请参阅 [Golang tls constants](https://golang.org/pkg/crypto/tls/#pkg-constants) |

### 旧版配置

如果您需要按照Rancher v2.1.7之前的方式配置TLS，请使用以下设置：

| 参数                | 旧版的值                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CATTLE_TLS_MIN_VERSION` | `1.0`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `CATTLE_TLS_CIPHERS`     | `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,`<br/>`TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,`<br/>`TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,`<br/>`TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,`<br/>`TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,`<br/>`TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,`<br/>`TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,`<br/>`TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,`<br/>`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,`<br/>`TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,`<br/>`TLS_RSA_WITH_AES_128_GCM_SHA256,`<br/>`TLS_RSA_WITH_AES_256_GCM_SHA384,`<br/>`TLS_RSA_WITH_AES_128_CBC_SHA,`<br/>`TLS_RSA_WITH_AES_256_CBC_SHA,`<br/>`TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,`<br/>`TLS_RSA_WITH_3DES_EDE_CBC_SHA` |

