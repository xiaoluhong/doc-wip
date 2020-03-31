---
title: Splunk
---

如果您的组织使用 [Splunk](https://www.splunk.com/)，则可以配置Rancher向其发送Kubernetes日志。之后，您可以登录到Splunk服务器以查看日志。

> **前提:**
>
> - 为您的Splunk服务器（Splunk Enterprise或Splunk Cloud）配置HTTP事件收集。
> - 创建新Token或复制现有Token。
>
> 有关更多信息，请参阅 [Splunk 文档](http://docs.splunk.com/Documentation/Splunk/7.1.2/Data/UsetheHTTPEventCollector#About_Event_Collector_tokens).

### Splunk配置

1. 在**访问地址**字段中，输入您的Splunk实例输入IP地址和端口(例如： `http://splunk-server:8088`)

   - Splunk通常使用端口 `8088`。 如果您使用的是Splunk Cloud，可以通过 [Splunk 支持](https://www.splunk.com/en_us/support-and-services.html) 来获取访问地址。

1. 输入您在Splunk中获得的**Token**。

1. 在**日志源**字段中定义Rancher日志源的名字。

1. **可选:** 输入您Token有权限访问的 [index](http://docs.splunk.com/Documentation/Splunk/7.1.2/Indexer/Aboutindexesandindexers)。

### SSL配置

如果您的Splunk实例使用SSL，则您的**访问地址**必须以`https://` 开头。输入正确的访问地址，将展开**SSL配置**表单。

1. 提供**客户端私钥**和**客户端证书**。您可以复制和粘贴它们，也可以使用**从文件读取**按钮上传它们。

   - 您可以使用自签名证书，也可以使用证书颁发机构提供的证书。

   - 您可以使用openssl命令生成自签名证书。例如：

     ```
     openssl req -x509 -newkey rsa:2048 -keyout myservice.key -out myservice.cert -days 365 -nodes -subj "/CN=myservice.example.com"
     ```

1. 输入您的**客户密钥密码**。

1. 选择是否要验证SSL证书

   - 如果使用的是自签名证书，请选择**启用-输入受信任的服务器证书**，并提供 **PEM格式的CA证书**。您可以复制和粘贴证书，也可以使用**从文件读取**按钮上传证书。
   - 如果您使用的是来自证书颁发机构的证书，请选择**启用-输入受信任的服务器证书**。您无需提供**PEM格式的CA证书**。


### 查看日志

1. 登录到Splunk服务器。

1. 点击**Search & Reporting**。**Indexed Events**的数量应在增加。

1. 点击 Data Summary 选择 Sources 选项卡.
   ![View Logs](/img/rancher/splunk/splunk4.jpg)

1. 要查看实际的日志，请选择您先前声明的数据源名字。
   ![View Logs](/img/rancher/splunk/splunk5.jpg)

### 故障排除

您可以使用curl来查看 **HEC** 是否开启

```
$ curl http://splunk-server:8088/services/collector/event \
    -H 'Authorization: Splunk 8da70994-b1b0-4a79-b154-bfaae8f93432' \
    -d '{"event": "hello world"}'
```

如果正确配置了Splunk，则应该收到 **json** 返回 `success code 0`。您应该能够
将记录数据发送到HEC。

如果收到错误，请检查Splunk和Rancher中的配置。
