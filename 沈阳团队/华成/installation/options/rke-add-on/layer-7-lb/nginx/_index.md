---
标题: NGINX配置
---

> #### **重要说明: Rancher v2.0.8之前仅支持RKE附加安装**
>
> 请使用Rancher Helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE附加组件安装方法，请参阅[使用RKE附加组件从Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)以获取有关如何使用Helm chart的详细信息。

### 安装 NGINX

首先在负载均衡器主机上安装NGINX。NGINX具有可用于所有已知操作系统的软件包。

有关安装NGINX的帮助，请参阅其 [安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)。

### 创建NGINX配置

请参见 [示例NGINX config](/docs/installation/options/chart-options/#example-nginx-config)。

### 运行NGINX

- 重新加载或重启NGINX

  ```
  # Reload NGINX
  nginx -s reload

  # Restart NGINX
  # Depending on your Linux distribution
  service nginx restart
  systemctl restart nginx
  ```

### 浏览Rancher UI

您现在应该能够浏览到 `https://FQDN`。
