---
title: Kafka
---

如果您的组织使用[Kafka](https://kafka.apache.org/)则可以配置Rancher向其发送Kubernetes日志。之后，您可以到Kafka服务器以查看日志。

> **前提:** 配置Kafka服务器。

### Kafka服务器配置

1. 选择您的Kafka服务器使用的**访问端点**类型：

- **Zookeeper**: 输入IP地址和端口。默认情况下，Zookeeper使用端口`2181`。注意，Zookeeper类型无法启用TLS。
- **Broker**: 点击**添加访问地址**。对于每个Kafka Broker，输入IP地址和端口。默认情况下，Kafka Broker使用端口`9092`。

1. 在**主题**字段中，输入Kubernetes集群提交日志的Kafka [topic](https://kafka.apache.org/documentation/#basic_ops_add_topic) 名称。

### **Broker**访问地址类型

#### SSL配置

如果您的Kafka的**Broker**使用SSL则需要填写**SSL配置**.

1. 提供**客户端私钥**和**客户端证书**。您可以复制和粘贴它们，也可以使用**从文件读取**按钮上载它们。

1. 提供**PEM格式的CA证书**。您可以复制和粘贴证书，也可以使用**从文件读取**按钮上载证书。

> **注意:** 启用客户端身份验证时，Kafka不支持自签名证书。

#### SASL配置

如果您的Kafka集群的Broker使用[SASL身份验证](https://kafka.apache.org/documentation/#security_sasl) ，则需要填写**SASL配置**

1. 输入SASL的**用户名**和**密码**。

1. 选择您的Kafka集群使用的**SASL类型**。

   - 如果您的Kafka使用的是**Plain**，请确保您的Kafka集群配置了SSL。

   - 如果您的Kafka使用**Scram**，则需要选择使用的**Scram机制**
