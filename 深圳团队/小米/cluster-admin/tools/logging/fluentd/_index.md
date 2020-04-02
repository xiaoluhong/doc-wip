---
title: Fluentd
---

如果您的组织使用 [Fluentd](https://www.fluentd.org/)， 则可以配置Rancher向其发送Kubernetes日志。然后，您可以在Fluentd服务器中查看日志。

> **前提:** 配置Fluentd接收日志
>
> 有关详细信息，请参见 [Fluentd 文档](https://docs.fluentd.org/v1.0/articles/in_forward)。

### Fluentd 配置

您可以添加多个Fluentd服务器。如果要添加其他Fluentd服务器，请单击**添加Fluentd服务器**。对于每个Fluentd服务器，请完成配置信息：

1. 在**访问地址**字段中，输入Fluentd实例的地址和端口，例如 `http://Fluentd-server:24224`.

1. 如果您的Fluentd Server使用共享密钥进行身份验证，请输入**共享密钥**。

1. 如果您的Fluentd Server使用用户名和密码进行身份验证，请输入**用户名**和**密码**。

1. **可选：** 输入Fluentd服务器的**主机名**。

1. 输入Fluentd服务器的负载均衡**权重**。如果一台服务器的权重为20，另一台服务器的权重为30，则事件将以2：3的比率发送。如果不输入权重，则默认权重为60。

1. 如果该服务器是备用服务器，勾选**仅用作备用服务器**。当所有其他服务器都不可用时，将使用备用服务器。

添加所有Fluentd服务器之后，您可以选择**启用Gzip压缩**。默认启用以减少传输数据。

### SSL配置

如果您的Fluentd服务器正在使用TLS，勾选**启用TLS**。如果您使用的是自签名证书，请提供**PEM格式的CA证书**。您可以复制和粘贴证书，也可以使用**从文件读取**按钮上传证书。

> **注意:** 启用客户端身份验证后，Fluentd不支持自签名证书。
