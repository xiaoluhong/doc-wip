---
标题: 配置Microsoft Active Directory联合身份验证服务（SAML）
---

v2.0.7版本起可用_

如果您的公司使用Microsoft Active Directory联合身份验证服务（AD FS）进行用户身份验证，则可以将Rancher配置为允许用户使用其AD FS凭据登录。
### 先决添加

- 安装好Rancher。

  - 获取您的Rancher Server URL。在AD FS配置期间，请将该URL替换为 `<RANCHER_SERVER>` 占位符。

  - 您的Rancher必须具有全局管理员帐户。

- 您必须配置一个[Microsoft AD FS 服务器](https://docs.microsoft.com/en-us/windows-server/identity/active-directory-federation-services) .

      	- 获取您的AD FS服务器IP / DNS名称。在AD FS配置期间，将此IP / DNS名称替换为`<AD_SERVER>`.
      	- 您必须有权在您的帐户上添加[Relying Party Trusts]（https://docs.microsoft.com/zh-cn/windows-server/identity/ad-fs/operations/create-a-relying-party-trust） AD FS服务器。

### 设置方法

使用Rancher Server设置Microsoft AD FS要求在Active Directory服务器上配置AD FS，并配置Rancher来利用AD FS服务器。以下页面作为在Rancher安装上设置Microsoft AD FS身份验证的指南。

- [1 — 为Rancher配置Microsoft AD FS](/docs/admin-settings/authentication/microsoft-adfs/microsoft-adfs-setup)
- [2 — 为Microsoft AD FS配置Rancher](/docs/admin-settings/authentication/microsoft-adfs/rancher-adfs-setup)


#### [下一步: 为Rancher配置Microsoft AD FS](/docs/admin-settings/authentication/microsoft-adfs/microsoft-adfs-setup)
