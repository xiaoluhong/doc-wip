---
标题：基于角色的访问控制(RBAC)
---

在Rancher中，每个人都以_user_身份进行身份验证，这是授予您访问Rancher权限的登录名。如[身份验证](/docs/admin-settings/authentication/)中所述，用户可以是本地用户，也可以是外部用户。

配置外部身份验证后，在`用户`页面上显示的用户会更改。

- 如果您以本地用户身份登录，则仅显示本地用户。

- 如果您以外部用户身份登录，则同时显示外部和本地用户。

### 用户和角色

一旦用户登录到Rancher，他们的_authorization_或他们在系统中的访问权限将由_global权限_以及_cluster和项目角色_决定。

- [全局权限](/docs/admin-settings/rbac/global-permissions/)：

  在任何特定群集范围之外定义用户授权。

- [集群和项目角色](/docs/admin-settings/rbac/cluster-project-roles/)：

  在分配了角色的特定集群或项目中定义用户授权。

全局权限以及群集和项目角色都在[Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)之上实现。因此，权限和角色的执行由Kubernetes执行。
