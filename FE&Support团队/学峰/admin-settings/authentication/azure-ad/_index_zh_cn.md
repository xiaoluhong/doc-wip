---
标题: 配置Azure AD
---

_从v2.0.3开始提供_

如果你在Azure中托管了一个Active Directory(AD)实例，你可以将Rancher配置为允许你的用户使用他们的AD帐户登录。Azure AD外部身份验证的配置要求您在Azure和Rancher中进行配置。

> **注意:** Azure AD集成仅支持服务提供商发起的登录。

> **先决条件:** 已配置Azure AD实例。

> **注意:** 此过程的大部分是从[Microsoft Azure门户网站](https://Portal.Azure.com/)执行的。

### Azure活动目录配置大纲

将Rancher配置为允许用户使用其Azure AD帐户进行身份验证涉及多个过程。开始之前，请查看下面的大纲。

<a id=“提示”></a>

> **提示:** 在开始之前，我们建议您创建一个空文本文件。您可以使用此文件从Azure复制值，稍后将其粘贴到Rancher中。

<!-- TOC -->

- [1. 用Azure注册Rancher](#1-register-rancher-with-azure)
- [2. 创建Azure API密钥](#2-create-an-azure-api-key)
- [3. 为Rancher设置所需权限](#3-set-required-permissions-for-rancher)
- [4. 添加回复URL](#4-add-a-reply-url)
- [5. 复制Azure应用程序数据](#5-copy-azure-application-data)
- [6. 在Rancher中配置Azure AD](#6-configure-azure-ad-in-rancher)

<!-- /TOC -->

#### 1. 在Azure注册Rancher

在Rancher中启用Azure AD之前，必须向Azure注册Rancher。

1. 以管理用户身份登录到[Microsoft Azure](https://portal.Azure.com/)。以后步骤中的配置需要管理访问权限。

1. 使用“搜索”打开**应用程序注册**服务。

   ![Open App Registrations](/img/rancher/search-app-registrations.png)

1. 单击**新建申请注册**，完成**创建**表单。

   ![New App Registration](/img/rancher/new-app-registration.png)

   1. 输入**名称**(类似于`Rancher`)。

   1. 在**应用程序类型**中，确保选择了**Web应用程序/API**。

   1. 在**登录URL**字段中，输入Rancher服务器的URL。

   1. 单击**创建**。

#### 2. 创建Azure API密钥

从Azure门户创建API密钥。Rancher将使用此密钥向Azure AD进行身份验证。

1. 使用搜索打开**应用注册**服务。然后打开您在上一个过程中创建的Rancher条目。

   ![Open Rancher Registration](/img/rancher/open-rancher-app.png)

   **步骤结果:** 一个新的刀片为Rancher打开。

1. 单击**设置**。

1. 从**设置**刀片中，选择**键**。

1. 从**密码**，创建一个API密钥。

   1. 输入**密钥描述**(类似于`Rancher`)。
   
   1. 为键选择**持续时间**。此下拉列表设置密钥的到期日期。较短的持续时间更安全，但要求在过期后创建新密钥。
   
   1. 单击**保存**(不需要输入保存后将自动填充的值)。
   
   <a id=“秘密”></a>

1. 复制键值并将其保存到[空文本文件](#提示)。

   稍后，您将此密钥作为**应用程序机密**输入Rancher UI。

   您将无法在Azure UI中再次访问密钥值。

#### 3. 为Rancher设置所需权限

接下来，在Azure中为Rancher设置API权限。

1. 从**设置**刀片中，选择**所需权限**。

   ![Open Required Permissions](/img/rancher/select-required-permissions.png)

1. 单击**Windows Azure活动目录**。

1. 从**启用访问**刀片中，选择以下**委派权限**:
   <br/>
   <br/>
   
   - **作为登录用户访问目录**
   - **读取目录数据**
   - **读取所有组**
   - **阅读所有用户的完整配置文件**
   - **阅读所有用户的基本配置文件**
   - **登录并阅读用户配置文件**

1. 单击**保存**。

1. 在**所需权限**中，单击**授予权限**。然后单击**是**。

> **注意:** 必须以Azure管理员身份登录才能成功保存权限设置。

#### 4. 添加回复URL

要将Azure广告与Rancher一起使用，必须将Rancher与Azure一起白名单。您可以通过向Azure提供Rancher的回复URL来完成此白名单，该URL是您的Rancher服务器URL，后跟验证路径。

1. 从**设置**刀片中，选择**回复URL**。

   ![Azure: Enter Reply URL](/img/rancher/enter-azure-reply-url.png)

1. 从**回复URL**blade中，输入您的Rancher服务器的URL，并附加验证路径:`<MY_Rancher_URL>/verify auth azure`。

   > **提示:**您可以在Rancher的Azure AD身份验证页面(全局视图> 安全身份验证> Azure AD)中找到您的个性化Azure回复URL。

1. 单击**保存**。

   **结果:** 您的回复URL已保存。

> **注意:** 此更改最长可能需要5分钟才能生效，因此如果在Azure AD配置之后无法立即进行身份验证，请不要惊慌。

#### 5. 复制Azure应用程序数据

作为在Azure中的最后一步，复制将用于为Azure AD身份验证配置Rancher的数据，并将其粘贴到空文本文件中。
1. 获取您的Rancher**租户ID**。

   1. 使用搜索打开**Azure Active Directory**服务。

      ![Open Azure Active Directory](/img/rancher/search-azure-ad.png)

   1. 从**Azure活动目录**菜单中，打开**属性**。

   1. 复制**目录ID**并粘贴到您的[文本文件](#提示)。

您将把这个值作为**租户ID**粘贴到Rancher中。

1. 获取您的Rancher**申请ID**。

   1 使用搜索打开**应用程序注册**。

      ![Open App Registrations](/img/rancher/search-app-registrations.png)

   1. 找到你为Rancher创建的条目。

   1. 复制**应用程序ID**并粘贴到[文本文件](#提示)。

1. 获取Rancher**图形端点**、**令牌端点**和**身份验证端点**。

   1. 在**应用注册**中，单击**端点**。

      ![Click Endpoints](/img/rancher/click-endpoints.png)

   2. 将下列终结点复制到剪贴板并粘贴到[文本文件](#提示)(这些值将是您的Rancher终结点值)。

      - **Microsoft Graph API端点**(图形端点)
      - **OAuth 2.0令牌终结点(v1)**(令牌终结点)
      - **OAuth 2.0授权终结点(v1)**(身份验证终结点)

> **注意:** 复制终结点的v1版本

#### 6. 在Rancher中配置Azure AD

从Rancher UI中，输入有关托管在Azure中的广告实例的信息以完成配置。

输入复制到[文本文件]的值(提示)。

1. 登录到Rancher。在**全局**视图中，选择**安全性>身份验证**。

1. 选择**Azure AD**。

1. 使用完成[复制Azure应用程序数据](#5-Copy-Azure-Application-Data)时复制的信息完成**配置Azure AD帐户**表单。

   > **重要提示:** 输入图形终结点时，从URL中删除租户ID，如下所示。
   >
   > <code> http<span> s://g</span> raph.windows.net/<del> abb5adde-bee8-4821-8b03-e63efdc7701c</del> </code>
   
   下表将您在Azure门户中复制的值映射到Rancher中的字段。
   
   | Rancher | 蔚蓝价值 |
   | ------------------ | ------------------------------------- |
   | 租户ID  | 目录ID |
   | 应用程序ID  | 应用程序ID |
   | 应用程序机密 | 密钥值 |
   | 端点 |  https://login.microsoftonline.com/ |
   | 图形终结点 |  Microsoft Azure AD Graph API终结点 |
   | 令牌端点 |  OAuth 2.0令牌端点 |
   | 认证端点 |  OAuth 2.0授权端点 |

1. 单击 **使用Azure进行身份验证**。

**结果:** 已配置Azure Active Directory身份验证。