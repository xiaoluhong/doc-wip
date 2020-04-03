---
标题: 配置 GitHub
---

在使用GitHub的环境中，您可以配置Rancher允许使用GitHub凭据登录。

> **先决条件:** 阅读 [外部认证配置和主用户](/docs/admin-settings/authentication/#external-authentication-configuration-and-principal-users).

1.  使用分配了 `administrator` 角色的本地用户（即本地主体）登录Rancher 。

2.  从 **全局** 视图中, 从主菜单中选择 **安全 > 认证**。

3.  选择 **GitHub**.

4.  按照显示的说明**安装GitHub应用程序**. Rancher将您重定向到GitHub以完成注册。

    > **什么是授权回调URL？**
    >
    > 授权回调URL是用户开始使用您的应用程序的URL（即初始屏幕显示）。

    > 当您使用外部身份验证时，身份验证实际上不会在您的应用程序中进行。而是在外部进行身份验证（在本例中为GitHub）。在此外部身份验证成功完成之后，授权回调URL是用户重新输入应用程序的位置。



5.  从github中拷贝 **Client ID** 和 **Client Secret**将它们粘贴在Rancher中.

    > **从哪里获取 Client ID 和 Client Secret?**
    >
    > 从 GitHub, 选择 Settings > Developer Settings > OAuth Apps. 此处有 Client ID 和 Client Secret.

6.  点击 **Authenticate with GitHub**.

7.   使用 **Site Access** 选项来配置用户授权范围.

    - **允许任何有效用户**

      不建议配置任何有效用户都可以访问Rancher!

    - **允许集群，项目以及授权用户和组织的成员**

     添加为**集群成员** 或**项目成员** 任何GitHub用户或组均可登录Rancher 另外,可以添加**授权用户和组织**列表中的任何GitHub用户或组都可以登录Rancher。

    - **限制仅访问授权用户和组织**

        只有添加到“授权用户和组织”中的GitHub用户或组才能登录到Rancher。

      <br/>

8.  点击 **保存**.

**结果:**

- GitHub身份验证已配置。
- 您已可以使用GitHub帐户（即外部主体）登录到Rancher 。
