---
title: Authentication, Permissions and Global Configuration
---

---
标题:身份验证、权限和全局配置
---

After installation, the [system administrator](/docs/admin-settings/rbac/global-permissions/) should configure Rancher to configure authentication, authorization, security, default settings, security policies, drivers and global DNS entries.

安装完成后，[系统管理员](/docs/admin-settings/rbac/global-permissions/)应当对Rancher平台配置身份验证、授权、安全、默认设置、安全策略、驱动程序和全局DNS等项目。

### First Log In
### 首次登录

After you log into Rancher for the first time, Rancher will prompt you for a **Rancher Server URL**.You should set the URL to the main entry point to the Rancher Server. When a load balancer sits in front a Rancher Server cluster, the URL should resolve to the load balancer. The system will automatically try to infer the Rancher Server URL from the IP address or host name of the host running the Rancher Server. This is only correct if you are running a single node Rancher Server installation. In most cases, therefore, you need to set the Rancher Server URL to the correct value yourself.

第一次登录Rancher后，Rancher将提示您输入一个**Rancher服务器URL**。您应该将URL设置为Rancher服务器的主入口点。当负载均衡器位于Rancher服务器集群前面时，URL应该设置为负载均衡地址。系统将自动尝试从运行Rancher服务器的主机的IP地址或主机名推断Rancher服务器的URL。但只有在运行单个节点Rancher服务器安装时才上述推断才正确。因此，在大多数情况下，您需要自己将Rancher服务器URL设置为正确的值。

> **Important!** After you set the Rancher Server URL, we do not support updating it. Set the URL with extreme care.

> **重要!**Rancher服务器URL在设置后不支持更新修改操作。务必确保设置正确的URL。

### Authentication
### 身份验证

One of the key features that Rancher adds to Kubernetes is centralized user authentication. This feature allows to set up local users and/or connect to an external authentication provider. By connecting to an external authentication provider, you can leverage that provider's user and groups.

Rancher向Kubernetes添加的关键特性之一是集中式用户身份验证。该特性允许设置本地用户和/或连接到外部身份验证系统。通过连接到外部身份验证系统，您可以利用该系统的用户和组。

For more information how authentication works and how to configure each provider, see [Authentication](/docs/admin-settings/authentication/).

有关身份验证如何工作以及如何配置不同身份验证系统的更多信息，请参见[身份验证](/docs/admin-settings/authentication/)。

### Authorization
### 授权


Within Rancher, each person authenticates as a _user_, which is a login that grants you access to Rancher. Once the user logs in to Rancher, their _authorization_, or their access rights within the system, is determined by the user's role. Rancher provides built-in roles to allow you to easily configure a user's permissions to resources, but Rancher also provides the ability to customize the roles for each Kubernetes resource.

在Rancher中，每个人都认证为一个*用户*，这是一个允许您访问Rancher的登录名。一旦用户登录到Rancher，他们在系统中的*授权*或访问权限就由用户的角色决定。Rancher提供了内置的角色，允许您轻松地配置用户对资源的权限，同时Rancher还提供了为每个Kubernetes资源定制角色的能力。

For more information how authorization works and how to customize roles, see [Roles Based Access Control (RBAC)](/docs/admin-settings/rbac/).

有关授权如何工作以及如何自定义角色的更多信息，请参见[基于角色的访问控制(RBAC)](/docs/admin-settings/rbac/)。

### Pod Security Policies
### Pod安全策略

_Pod Security Policies_ (or PSPs) are objects that control security-sensitive aspects of pod specification, e.g. root privileges. If a pod does not meet the conditions specified in the PSP, Kubernetes will not allow it to start, and Rancher will display an error message.

_Pod安全策略_(或PSPs)是控制安全敏感相关对象的pod规范，例如root特权。如果一个pod不符合PSP中指定的条件，Kubernetes将不允许它启动，Rancher将显示一个错误消息。

For more information how to create and use PSPs, see [Pod Security Policies](/docs/admin-settings/pod-security-policies/).

有关如何创建和使用psp的更多信息，请参见[Pod安全策略](/docs/admin-settings/pod-security-policies/)。

### Provisioning Drivers
### 配置驱动程序

Drivers in Rancher allow you to manage which providers can be used to provision [hosted Kubernetes clusters](/docs/cluster-provisioning/hosted-kubernetes-clusters/) or [nodes in an infrastructure provider](/docs/cluster-provisioning/rke-clusters/node-pools/) to allow Rancher to deploy and manage Kubernetes.

Rancher中的驱动程序允许您管理使用何种供应商创建[托管Kubernetes集群](/docs/cluster-provisioning/hosted-kubernetes-clusters/) 或者 [基础设施供应商提供的主机节点](/docs/cluster-provisioning/rke-clusters/node-pools/) 以允许Rancher部署和管理Kubernetes。


For more information, see [Provisioning Drivers](/docs/admin-settings/drivers/).

相关更多信息，请参考[配置驱动程序](/docs/admin-settings/drivers/)。

### Adding Kubernetes Versions into Rancher

### 添加Kubernetes版本到Rancher


_Available as of v2.3.0_

_v2.3.0版本可用_

With this feature, you can upgrade to the latest version of Kubernetes as soon as it is released, without upgrading Rancher. This feature allows you to easily upgrade Kubernetes patch versions (i.e. `v1.15.X`), but not intended to upgrade Kubernetes minor versions (i.e. `v1.X.0`) as Kubernetes tends to deprecate or add APIs between minor versions.

基于此特性，您可以在Kubernetes发布后立即升级到最新版本，而无需升级Rancher。这个功能可以让您轻松升级Kubernetes的补丁版本（例如：`v1.15.X`)，但不升级Kubernetes的小版本(例如：`v1.X.0`)。因为Kubernetes倾向于在较小的版本之间不支持或添加api。

The information that Rancher uses to provision [RKE clusters](/docs/cluster-provisioning/rke-clusters/) is now located in the Rancher Kubernetes Metadata. For details on metadata configuration and how to change the Kubernetes version used for provisioning RKE clusters, see [Rancher Kubernetes Metadata.](/docs/admin-settings/k8s-metadata/)

Rancher用于提供[RKE集群](/docs/cluster-provisioning/rke-clusters/)的信息现在位于Rancher Kubernetes元数据中。有关元数据配置和如何更改用于供应RKE集群的Kubernetes版本的详细信息，请参见[Rancher Kubernetes元数据 ](/docs/admin-settings/k8s-metadata/)。

Rancher Kubernetes Metadata contains Kubernetes version information which Rancher uses to provision [RKE clusters](/docs/cluster-provisioning/rke-clusters/).

Rancher Kubernetes元数据包含Rancher用于提供[RKE集群](/docs/cluster-provisioning/rke-clusters/)的Kubernetes版本信息。

For more information on how metadata works and how to configure metadata config, see [Rancher Kubernetes Metadata](/docs/admin-settings/k8s-metadata/).

有关元数据如何工作以及如何配置元数据配置的更多信息，请参见[Rancher Kubernetes元数据](/docs/admin-settings/k8s-metadata/)。

### Enabling Experimental Features

### 启用试验性功能

_Available as of v2.3.0_

_v2.3.0版本可用_


Rancher includes some features that are experimental and disabled by default. Feature flags were introduced to allow you to try these features. For more information, refer to the section about [feature flags.](/docs/admin-settings/feature-flags/)

Rancher包含一些在默认情况下禁用的试验性特性。引入特性标志能够让您能够尝试这些特性。有关更多信息，请参考关于[特性标志](/docs/admin-settings/features-flags/)的章节。
