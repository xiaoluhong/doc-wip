---
title: 2 — 在Rancher中配置对接Microsoft AD FS
---

自v2.0.7版本开始可用  

完成 [在Microsoft AD FS中配置Rancher对接](/docs/admin-settings/authentication/microsoft-adfs/microsoft-adfs-setup/), 将您的AD FS信息输入Rancher，以允许AD FS用户通过Rancher进行身份验证。

> **有关配置AD FS服务器的重要说明:**
>
> - SAML 2.0 WebSSO协议服务URL为: `https://<RANCHER_SERVER>/v1-saml/adfs/saml/acs`
> - 依赖方信任标识符URL为: `https://<RANCHER_SERVER>/v1-saml/adfs/saml/metadata`
> - 您必须 `federationmetadata.xml` 从AD FS服务器导出文件。可以在以下位置找到: `https://<AD_SERVER>/federationmetadata/2007-06/federationmetadata.xml`

1.  在**全局** 视图中, 选择 **安全 > 认证** 菜单.

2.  选择 **ADFS**.

3.  完成 **配置 AD FS帐户** 表单. Microsoft AD FS允许您指定现有的Active Directory（AD）服务器。以下示例描述了如何将AD属性映射到Rancher中的字段。

    | 字段名                     | 描述                                                                                                                                                                                                    |
    | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | 显示名称       | 包含用户显示名称的AD属性. <br/><br/>例: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`                                                                 |
    | 用户名          | 包含用户名/给定名称的AD属性。 <br/><br/>例: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`                                                                       |
    | UID               | 每个用户唯一的AD属性。 <br/><br/>例: `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`                                                                                   |
    | 组            | 进行输入以管理组成员身份。 <br/><br/>例: `http://schemas.xmlsoap.org/claims/Group`                                                                                                      |
    | Rancher API 地址         | Rancher服务器的URL。
                                                                                                                                                                               |
    | 私人钥 / 证书 | 这是用于在Rancher和您的AD FS之间创建安全外壳的密钥证书对。确保将通用名称（CN）设置为Rancher Server URL。<br/><br/>[证书创建命令](#cert-command) |
    | 元数据 XML              | 从AD FS服务器导出的文件。`federationmetadata.xml` . <br/><br/>您可以在找到此文件 `https://<AD_SERVER>/federationmetadata/2007-06/federationmetadata.xml`.                                |

    <a id="cert-command"></a>

    > **Tip:** 您可以使用openssl命令生成证书。例如：:
    >
    >        openssl req -x509 -newkey rsa:2048 -keyout myservice.key -out myservice.cert -days 365 -nodes -subj "/CN=myservice.example.com"

1) 完成 **配置AD FS帐户** 表单后, 点击 **使用AD FS进行身份验证**按钮.

Rancher将您重定向到AD FS登录页面。输入通过Microsoft AD FS进行身份验证的凭据，以验证Rancher AD FS配置。
   > **注意:** 您可能必须禁用弹出窗口阻止程序才能查看AD FS登录页面。

**结果:**  Rancher被配置为与MS FS一起使用。您的用户现在可以使用其MS FS登录名登录Rancher.
