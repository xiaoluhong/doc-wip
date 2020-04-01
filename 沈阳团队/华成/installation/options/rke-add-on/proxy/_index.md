---
标题: HTTP代理配置
---

> #### **重要说明: Rancher v2.0.8之前仅支持RKE附加安装**
>
> 请使用Rancher helm chart将Rancher安装在Kubernetes集群上。 有关详细信息，请参见 [Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline).
>
> 如果您当前正在使用RKE附加组件安装方法, 请参阅 [使用RKE附加组件从Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/) 以获取有关如何使用Helm Chart的详细信息。

如果您在代理后操作Rancher，并且想要通过代理访问服务（例如检索目录），则必须提供有关代理的Rancher信息。由于Rancher是用Go编写的，因此它使用了常见的代理环境变量，如下所示。

确保 `NO_PROXY` 包含不应使用代理的网络地址，网络地址范围和域。

| 环境变量 | 目的                                                                                                                 |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| HTTP_PROXY           | 启动HTTP连接时要使用的代理地址                 |
| HTTPS_PROXY          | 启动HTTP连接时要使用的代理地址                 |
| NO_PROXY             | 在启动连接时不使用代理的网络地址，网络地址范围和域 |

> **注意** NO_PROXY 必须为大写才能使用网络范围（CIDR）表示法。

### 在Kubernetes集群上安装Rancher

使用Kubernetes安装时，需要将环境变量添加到RKE Con​​fig File模板中。

- [使用外部负载均衡器（TCP / Layer 4）RKE配置文件模板进行Kubernetes安装](/docs/installation/ha-server-install/#5-download-rke-config-file-template)
- [使用外部负载均衡器（HTTPS / Layer 7）RKE配置文件模板进行Kubernetes安装](/docs/installation/ha-server-install-external-lb/#5-download-rke-config-file-template)

环境变量应在RKE配置文件模板内部的 `Deployment` 定义。您只需将以 `env:` 开头的部分添加到(但不包括) `ports:`。确保缩进与前面的 `name:` 相同。需要配置 `NO_PROXY` 的值为:

- `localhost`
- `127.0.0.1`
- `0.0.0.0`
- 配置 `service_cluster_ip_range` (默认值: `10.43.0.0/16`)

下面的示例基于一个可访问的代理服务器 `http://192.168.0.1:3128`, 并且不包括访问网络范围 `192.168.10.0/24`, 配置 `service_cluster_ip_range` (`10.43.0.0/16`) 以及域 `example.com` 下的所有主机名。 如果更改了 `service_cluster_ip_range`, 则必须相应地更新下面的值。

```yaml

---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  namespace: cattle-system
  name: cattle
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: cattle
    spec:
      serviceAccountName: cattle-admin
      containers:
        - image: rancher/rancher:latest
          imagePullPolicy: Always
          name: cattle-server
          env:
            - name: HTTP_PROXY
              value: 'http://192.168.10.1:3128'
            - name: HTTPS_PROXY
              value: 'http://192.168.10.1:3128'
            - name: NO_PROXY
              value: 'localhost,127.0.0.1,0.0.0.0,10.43.0.0/16,192.168.10.0/24,example.com'
          ports:
```
