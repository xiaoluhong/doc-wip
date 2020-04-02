---
title: Rancher与日志收集目标服务的集成
---

日志服务很有用，因为它使您能够：

- 捕获并分析集群的状态
- 在您的环境中发现趋势
- 将日志保存到群集之外的安全位置
- 随时了解容器崩溃，Pod驱逐或节点死亡等事件
- 更轻松地调试和排除故障

Rancher支持与以下日志收集目标服务集成：

- Elasticsearch
- Splunk
- Kafka
- Syslog
- Fluentd

本节涵盖以下主题：

- [Rancher日志的工作方式](#how-logging-integrations-work)
- [前提](#requirements)
- [日志范围](#logging-scope)
- [启用集群日志](#enabling-cluster-logging)

## Rancher日志服务的工作方式

Rancher使用Fluentd收集日志，并与外部日志服务集成。

将Rancher配置为与这些外部日志服务集成时，必须将Rancher指向服务的访问地址端点并提供身份验证信息。

Fluentd收集保存在节点上 `/var/log/containers` 目录下的容器日志，其中包含了容器的标准输出日志和错误日志并发送到外部日志服务，您可以进入接收日志的外部日志服务查看日志。

> **注意:** 您只能为每个群集或每个项目配置一个外部日志服务。

## 前提

群集中每个节点上的Docker守护程序应配置 [configured](https://docs.docker.com/config/containers/logging/configure/) （默认）日志驱动为: `json-file`. 您可以通过运行以下命令来检查日志驱动配置：

```
$ docker info | grep 'Logging Driver'
Logging Driver: json-file
```

## 日志收集范围

您可以在集群级别或项目级别配置日志收集。

- 集群日志采集集群中所有Pod的日志，即包含所有项目。对于 [RKE 集群](/docs/cluster-provisioning/rke-clusters)，它还会收集所有Kubernetes系统组件的日志。
- [项目级别的日志](/docs/project-admin/tools/logging/) 收集所有在这个项目下的Pod的日志。

发送到外部日志服务的日志来源于:

- Pod日志地址 `/var/log/containers`。
- Kubernetes系统组件日志地址 `/var/lib/rancher/rke/log/`。

## 开启集群级别的日志

作为[管理员](/docs/admin-settings/rbac/global-permissions/) 或者 [集群所有者](/docs/admin-settings/rbac/cluster-project-roles/#cluster-roles)，您可以进行配置Rancher将Kubernetes集群日志发送到日志服务。

1. 从**全局**视图中，进入要收集日志的群集。

1. 在导航栏中选择**工具>日志**。

1. 选择一个日志服务，然后输入配置。有关详细的配置，请参考特定的服务。 Rancher支持与以下日志服务集成：

   - [Elasticsearch](/docs/cluster-admin/tools/logging/elasticsearch/)
   - [Splunk](/docs/cluster-admin/tools/logging/splunk/)
   - [Kafka](/docs/cluster-admin/tools/logging/kafka/)
   - [Syslog](/docs/cluster-admin/tools/logging/syslog/)
   - [Fluentd](/docs/cluster-admin/tools/logging/fluentd/)

1. （可选）您可以通过单击位于日志记录目标上方的**以文件形式编辑**来输入自定义高级配置，而不必使用UI来配置日志服务。仅在选择日志服务后，此链接才可见。

   - 使用文件编辑器，为任何日志服务输入原始的Fluentd配置。有关如何设置输出配置，请参阅每个日志服务的文档。

     - [Elasticsearch Documentation](https://github.com/uken/fluent-plugin-elasticsearch)
     - [Splunk Documentation](https://github.com/fluent/fluent-plugin-splunk)
     - [Kafka Documentation](https://github.com/fluent/fluent-plugin-kafka)
     - [Syslog Documentation](https://github.com/dlackty/fluent-plugin-remote_syslog)
     - [Fluentd Documentation](https://docs.fluentd.org/v1.0/articles/out_forward)

   - 如果日志记录服务使用的是TLS，则还需要填写 **SSL 配置**。

     1. 提供**客户端私钥**和**客户端证书**。您可以复制和粘贴它们，也可以使用**从文件读取**按钮上传它们。

        - 您可以使用自签名证书，也可以使用证书颁发机构提供的证书。

        - 您可以使用openssl命令生成自签名证书。例如：

          ```
          openssl req -x509 -newkey rsa:2048 -keyout myservice.key -out myservice.cert -days 365 -nodes -subj "/CN=myservice.example.com"
          ```

     2. 如果使用的是自签名证书，请选择**启用-输入受信任的服务器证书**，并提供 **PEM格式的CA证书**。

1. (可选) 填写 **其他日志配置**。

   1. **可选：**使用**添加字段**按钮将自定义日志字段添加到您的日志中。这些字段是键值对（例如“ foo = bar”），可用于区别来自另一个系统的日志。

   1.输入**刷新间隔**。 定义 [Fluentd](https://www.fluentd.org/) 将日志刷新到日志服务的频率。间隔以秒为单位。

   1. **包含系统日志**。来自系统项目和RKE组件中的pod的日志将发送到日志服务。取消选中它以排除系统日志。

1. 点击 **测试**。Rancher将测试日志发送到日志服务。

   > **注意:** 如果使用以文件形式自定义配置，则此按钮将替换为 _Dry Run_ 。在这种情况下，Rancher调用Fluentd dry run命令以验证配置。

1. 点击 **保存**.

**结果:** 现在，Rancher已配置为将日志发送到所选日志服务。登录到外部日志服务，以便您可以开始查看日志。

### 相关链接

[日志架构](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
