---
标题: 配置PingIdentity (SAML)
---

_v2.0.7版本可用_

如果您的组织使用Ping Identity Provider (IdP)进行用户身份验证，您可以配置Rancher以允许您的用户使用他们的IdP凭据登录。

> **先决条件:**
> 
> - 您必须配置一个[Ping IdP服务器](https://www.pingidentity.com/)。
以下是Rancher服务提供商配置所需的url: Metadata URL: `https://<rancher-server>/v1-saml/ping/saml/metadata` Assertion Consumer Service(ACS) URL: `https://<rancher-server>/v1-saml/ping/saml/acs`
> - 从IdP服务器导出元数据`metadata.xml`文件。有关更多信息，请参见[PingIdentity文档](https://document.pingidentity.com/pingfederate/pf83/index.shtml#concept_exportingMetadata.html)。

1. 在**全局**视图中，从主菜单中选择**安全 > 认证**。

1. 选择**PingIdentity**。

1. 完成**配置Ping帐户**表单。Ping IdP允许您指定要使用的数据存储。您可以添加数据库，也可以使用现有的ldap服务器。例如，如果您选择Active Directory (AD)服务器，下面的示例将描述如何将AD属性映射到Rancher中的字段。

   1. **显示名称字段**: 输入包含用户显示名称的AD属性(例如:`displayName`)。

   1. **用户名字段**: 输入包含用户名/给定名的AD属性(例如:`givenName`)。

   1. **UID字段**: 输入每个用户唯一的AD属性(例如:`sAMAccountName`，`ishedname`)。

   1. **Groups字段**: 创建用于管理组成员关系的条目(例如:`memberOf`)。

   1. **Rancher API主机**: 输入Rancher 服务器的URL。

   1. **私钥**和**证书**:这是一个密钥-证书对，用于在Rancher和您的IdP之间创建一个安全shell。

      您可以使用openssl命令生成一个密钥-证书对。示例如下:

      ```
      openssl req -x509 -newkey rsa:2048 -keyout myservice.key -out myservice.cert -days 365 -nodes -subj "/CN=myservice.example.com"
      ```

   1. **IDP-metadata**: [从你的IdP服务器导出](https://documentation.pingidentity.com/pingfederate/pf83/index.shtml#concept_exportingMetadata.html)的`metadata.xml`文件。


1) 完成**配置Ping帐号**表单后，点击页面底部的**启用Ping认证**。

   Rancher会将您重定向到IdP登录页面。输入使用Ping IdP进行身份验证的凭据，以验证您的Rancher PingIdentity配置。

   **注:** 您可能需要禁用弹出窗口拦截器才能看到IdP登录页面。

**结果:** Rancher被配置为使用PingIdentity认证。您的用户现在可以使用他们的PingIdentity账号登录到Rancher。

{{< saml_caveats >}}
