---
title: 创建 GKE 集群
---

### 谷歌云平台的先决条件

> **注意**
> 部署到 GKE 将会产生费用.

使用[谷歌云平台](https://console.cloud.google.com/projectselector/iam-admin/serviceaccounts)创建服务账户. GKE使用这个帐户来操作您的集群.创建此帐户还将生成用于身份验证的私钥.

服务帐户需要以下角色:

- **Compute Viewer:** `roles/compute.viewer`
- **Project Viewer:** `roles/viewer`
- **Kubernetes Engine Admin:** `roles/container.admin`
- **Service Account User:** `roles/iam.serviceAccountUser`

[谷歌文档:创建和启用服务帐户](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances)

### 创建 GKE 集群

使用 {{< product >}} 来设置和配置您的Kubernetes集群.

1. 从 **集群** 页面, 单击 **添加集群**.

2. 选择 **Google Container Engine**.

3. 输入 **集群名称**.

4. {{< step_create-cluster_member-roles >}}

5. 将您的服务帐户私钥粘贴在`服务帐户`文本框中,或者`从文件中读取` **. 然后单击`下一步：配置节点`.

   > **注意:** 在提交您的私钥之后, 您可能必须启用谷歌Kubernetes引擎API. 如果出现提示, 浏览到Rancher UI中显示的URL以启用API.

6. 选择您的**集群选项**, 自定义您的**节点**, 并自定义GKE集群的**安全性**. 检查你的选项并确认它们是正确的. 然后单击**创建**

{{< result_create-cluster >}}
