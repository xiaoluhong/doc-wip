---
title: Authentication
---

---
标题：身份验证
---

One of the key features that Rancher adds to Kubernetes is centralized user authentication. This feature allows your users to use one set of credentials to authenticate with any of your Kubernetes clusters.

Rancher向Kubernetes添加的关键特性之一是集中式用户身份验证。该特性允许您的用户使用一组凭证对任何Rancher管理的Kubernetes集群进行身份验证。

This centralized user authentication is accomplished using the Rancher authentication proxy, which is installed along with the rest of Rancher. This proxy authenticates your users and forwards their requests to your Kubernetes clusters using a service account.

这种集中式的用户身份验证是使用Rancher身份验证代理完成的，该代理与Rancher的其他组件一起安装。这个代理验证您的用户，并使用一个服务帐户将用户请求转发到您的Kubernetes集群。

<!-- todomark add diagram -->

### External vs. Local Authentication

### 外部验证与本地验证


The Rancher authentication proxy integrates with the following external authentication services. The following table lists the first version of Rancher each service debuted.

Rancher身份验证代理支持与以下外部身份验证服务集成。下表列出了每个身份验证服务Rancher支持的最初版本。

| 外部身份验证服务                                                       | 以下版本后可用  |
| ---------------------------------------------------------------------- | --------------- |
| [Microsoft Active Directory](/docs/admin-settings/authentication/ad/)  | v2.0.0          |
| [GitHub](/docs/admin-settings/authentication/github/)                  | v2.0.0          |
| [Microsoft Azure AD](/docs/admin-settings/authentication/azure-ad/)    | v2.0.3          |
| [FreeIPA](/docs/admin-settings/authentication/freeipa/)                | v2.0.5          |
| [OpenLDAP](/docs/admin-settings/authentication/openldap/)              | v2.0.5          |
| [Microsoft AD FS](/docs/admin-settings/authentication/microsoft-adfs/) | v2.0.7          |
| [PingIdentity](/docs/admin-settings/authentication/ping-federate/)     | v2.0.7          |
| [Keycloak](/docs/admin-settings/authentication/keycloak/)              | v2.1.0          |
| [Okta](/docs/admin-settings/authentication/okta/)                      | v2.2.0          |
| [Google OAuth](/docs/admin-settings/authentication/google/)            | v2.3.0          |

<br/>
However, Rancher also provides [local authentication](/docs/admin-settings/authentication/local/).

In most cases, you should use an external authentication service over local authentication, as external authentication allows user management from a central location. However, you may want a few local authentication users for managing Rancher under rare circumstances, such as if your external authentication provider is unavailable or undergoing maintenance.

同时，Rancher也提供了[本地认证](/docs/admin-settings/authentication/local/)。

在大多数情况下，应该使用外部身份验证服务，而不是本地身份验证，因为外部身份验证允许对用户进行集中管理。但是您可能需要一些本地身份验证用户，以便在特定的情况下管理Rancher，例如在外部身份验证系统不可用或正在进行维护的情况下。

### Users and Groups

### 用户和组

Rancher relies on users and groups to determine who is allowed to log in to Rancher and which resources they can access. When authenticating with an external provider, groups are provided from the external provider based on the user. These users and groups are given specific roles to resources like clusters, projects, multi-cluster apps, and global DNS providers and entries. When you give access to a group, all users who are a member of that group in the authentication provider will be able to access the resource with the permissions that you've specified. For more information on roles and permissions, see [Role Based Access Control](/docs/admin-settings/rbac/).

Rancher依赖于用户和组以确定谁被允许登录到Rancher，以及他们可以访问哪些资源。当使用外部系统进行身份验证时，将由外部系统提供用户和组。这些用户和组被赋予集群、项目、多集群应用程序、全局DNS提供商等资源的特定角色。当您授予对某个组的访问权时，身份验证提供程序中属于该组的所有用户都将能够使用您指定的权限访问该资源。有关角色和权限的更多信息，请参见[基于角色的访问控制](/docs/admin-settings/rbac/)。

> **Note:** Local authentication does not support creating or managing groups.

> **注:** 本地认证不支持创建或管理用户组。

For more information, see [Users and Groups](/docs/admin-settings/authentication/user-groups/)

有关更多信息, 请参考[用户和组](/docs/admin-settings/authentication/user-groups/)


### Scope of Rancher Authorization

### Rancher授权范围

After you configure Rancher to allow sign on using an external authentication service, you should configure who should be allowed to log in and use Rancher. The following options are available:

在您配置Rancher以允许使用外部身份验证服务登录之后，您应该配置应该允许谁登录并使用Rancher。以下是可供选择的方案:

| Access Level                                                                 | Description                                                                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Allow any valid Users                                                        | _Any_ user in the authorization service can access Rancher. We generally discourage use of this setting!                                                                                                                                                                       |
| Allow members of Clusters, Projects, plus Authorized Users and Organizations | Any user in the authorization service and any group added as a **Cluster Member** or **Project Member** can log in to Rancher. Additionally, any user in the authentication service or group you add to the **Authorized Users and Organizations** list may log in to Rancher. |
| Restrict access to only Authorized Users and Organizations                   | Only users in the authentication service or groups added to the Authorized Users and Organizations can log in to Rancher.                                                                                                                                                      |


| 访问级别                                                                     | 说明                                                                                                                                                                                                                                                                  |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 允许任何有效用户                                                        | 认证服务中的*任何*用户都可以访问Rancher。 通常情况下不建议使用该设置!                                                                                                                                                                       |
| 允许集群和项目成员，以及授权的用户和组织 | 认证服务中属于**集群成员**或**项目成员**的用户或组成员能够访问Rancher。此外，添加在**授权用户和组织**列表中的用户和组成员也能够访问Rancher。|
| 仅限于授权用户和组织                   | 仅有在**授权用户和组织**列表中的用户和组成员能够访问Rancher。                                                                                                                                                      |


To set the Rancher access level for users in the authorization service, follow these steps:

1. From the **Global** view, click **Security > Authentication.**

1. Use the **Site Access** options to configure the scope of user authorization. The table above explains the access level for each option.

1. Optional: If you choose an option other than **Allow any valid Users,** you can add users to the list of authorized users and organizations by searching for them in the text field that appears.

1. Click **Save.**

**Result:** The Rancher access configuration settings are applied.

{{< saml_caveats >}}

要在授权服务中为用户设置Rancher访问级别，请执行以下步骤:

1. 在**全局**视图中，单击**安全 > 认证**。

1. 使用**站点访问**选项来配置用户授权范围。上表解释了每个选项的访问级别。

1. 可选:如果您选择**允许任何有效用户**以外的选项，您可以通过在出现的文本字段中搜索用户，将用户添加到授权用户和组织的列表中。

1. 点击* *保存**。

**结果**：Rancher访问配置设置被应用。


### External Authentication Configuration and Principal Users

Configuration of external authentication requires:

- A local user assigned the administrator role, called hereafter the _local principal_.
- An external user that can authenticate with your external authentication service, called hereafter the _external principal_.

Configuration of external authentication affects how principal users are managed within Rancher. Follow the list below to better understand these effects.

1. Sign into Rancher as the local principal and complete configuration of external authentication.

   ![Sign In](/img/rancher/sign-in.png)

2. Rancher associates the external principal with the local principal. These two users share the local principal's user ID.

   ![Principal ID Sharing](/img/rancher/principal-ID.png)

3. After you complete configuration, Rancher automatically signs out the local principal.

   ![Sign Out Local Principal](/img/rancher/sign-out-local.png)

4. Then, Rancher automatically signs you back in as the external principal.

   ![Sign In External Principal](/img/rancher/sign-in-external.png)

5. Because the external principal and the local principal share an ID, no unique object for the external principal displays on the Users page.

   ![Sign In External Principal](/img/rancher/users-page.png)

6. The external principal and the local principal share the same access rights.

### 外部身份验证配置和用户主体

配置外部认证需要:

- 分配管理员角色的本地用户，以下称为*本地主体*。
- 可以使用外部认证服务进行认证的外部用户，以下简称为*外部主体*。

外部身份验证的配置将影响Rancher中主要用户的管理方式。按照下面的列表来更好地理解这些影响。

1. 作为本地主体登录到Rancher并完成外部身份验证的配置。

![登录](/img/rancher/sign-in.png)

2. Rancher将外部主体与本地主体相关联。这两个用户共享本地主体的用户ID。

![主体ID共享] (/img/rancher/principal-ID.png)

3. 完成配置后，Rancher将自动退出本地主体。

(/img/rancher/sign-out-local.png)

4. 然后，Rancher会自动将您登录回外部主体。

(/img/rancher/sign-in-external.png)

5. 因为外部主体和本地主体共享一个ID，所以用户列表页面上不会显示外部主体的惟一对象。

(/img/rancher/users-page.png)

6. 外部主体和本地主体共享相同的访问权限。
