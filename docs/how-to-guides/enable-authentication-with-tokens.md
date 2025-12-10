# Azure Active Directory 身份验证

本操作指南向您展示如何通过 Azure 配置 Medical Imaging Server for DICOM 的身份验证设置。要完成此配置，您将：

1. **在 Azure AD 中创建资源应用程序**：此资源应用程序将是 Medical Imaging Server for DICOM 的表示，可用于身份验证和获取令牌。应用程序要与 Azure AD 交互，需要注册。
2. **向 Medical Imaging Server for DICOM 提供应用注册详细信息**：注册资源应用程序后，您将设置 Medical Imaging Server for DICOM App Service 的身份验证。
3. **在 Azure AD 中创建服务客户端应用程序**：客户端应用程序注册是可用于身份验证和获取令牌的应用程序的 Azure AD 表示。服务客户端旨在供应用程序使用，以在没有用户交互式身份验证的情况下获取访问令牌。它将具有某些应用程序权限，并在获取访问令牌时使用应用程序机密（密码）。
4. **通过 Postman 或 Azure CLI 检索访问令牌**：启用服务客户端应用程序后，您可以获取访问令牌以对应用程序进行身份验证。

## 先决条件

1. 将 [Medical Imaging Server for DICOM 部署到 Azure](../quickstarts/deploy-via-azure.md)。
2. 本教程需要 Azure AD 租户。如果您尚未创建租户，请参阅[在 Azure Active Directory 中创建新租户](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)。

## 身份验证设置概述

配置中公开的当前身份验证设置如下：

```json

{
  "DicomServer" : {
    "Security": {
      "Enabled":  true,
      "Authentication": {
        "Audience": "",
        "Authority": ""
      }
    }
  }
}
```

| 元素                    | 描述 |
| -------------------------- | --- |
| Enabled                    | 服务器是否启用了任何安全功能。 |
| Authentication:Audience    | 标识令牌的目标接收者。这由 `DevelopmentIdentityProvider` 自动设置。 |
| Authentication:Authority   | jwt 令牌的颁发者。这由 `DevelopmentIdentityProvider` 自动设置。 |

## 使用 Azure AD 进行身份验证

### 为 Medical Imaging Server for DICOM 在 Azure AD 中创建资源应用程序

1. 登录 [Azure 门户](https://ms.portal.azure.com/)。搜索 **App Services** 并选择您的 Medical Imaging Server for DICOM App Service。复制 Dicom App Service 的 **URL**。
2. 选择 **Azure Active Directory** > **应用注册** > **新注册**：
    1. 输入应用注册的**名称**。
    2. 在**重定向 URI** 中，选择 **Web** 并输入 Medical Imaging Server for DICOM App Service 的 **URL**。
    3. 选择**注册**。
3. 选择**公开 API** > **设置**。您可以将 URI 指定为应用服务的 **URL** 或使用生成的 App ID URI。选择**保存**。
4. 选择**添加范围**：
    1. 在**范围名称**中，输入 *user_impersonation*。
    2. 在文本框中，添加您希望用户在同意页面上看到的管理员同意显示名称和管理员同意描述。例如，*访问我的应用*。

### 设置 App Service 的身份验证

1. 导航到您部署到 Azure 的 Medical Imaging Server for DICOM App Service。
2. 选择**配置**以更新 **Audience**、**Authority** 和 **Security:Enabled**：
    1. 将上面启用的**应用程序 ID URI** 设置为 **Audience**。
    2. **Authority** 是应用程序所在的租户，例如：```https://login.microsoftonline.com/<tenant-name>.onmicrosoft.com```。
    3.  将 **Security:Enabled** 设置为 ```True```。
    4.  保存对配置的更改。

### 创建服务客户端应用程序

1. 选择 **Azure Active Directory** > **应用注册** > **新注册**：
    1. 输入服务客户端的**名称**。您可以提供 **URI**，但通常不会使用。
    2. 选择**注册**。
2. 复制**应用程序（客户端）ID** 和**目录（租户）ID** 以供以后使用。
3. 选择**API 权限**以为服务客户端提供对资源应用程序的权限：
    1. 选择**添加权限**。
    2. 在**我的 API** 下，选择您上面为 Dicom App Service 创建的资源应用程序。
    3. 在**选择权限**下，从您在资源应用程序上定义的应用程序角色中选择应用程序角色。
    4. 选择**添加权限**。
4. 选择**证书和机密**以生成用于获取令牌的机密：
    1. 选择**新建客户端机密**。
    2. 提供机密的**描述**和持续时间。选择**添加**。
    3. 创建后立即复制机密。它只会在门户中显示一次。

### 使用 Azure CLI 获取访问令牌

1. 首先，更新您上面创建的应用程序以访问 Azure CLI：
    1. 选择**公开 API** > **添加客户端应用程序**。
    2. 对于**客户端 ID**，提供 Azure CLI 的客户端 ID：**04b07795-8ddb-461a-bbee-02f9e1bf7b46**。*注意：这在 [Azure CLI Github 存储库](https://github.com/Azure/azure-cli/blob/24e0b9ef8716e16b9e38c9bb123a734a6cf550eb/src/azure-cli-core/azure/cli/core/_profile.py#L65) 中可用*。
    3. 在**授权范围**下选择您的**应用程序 ID URI**。
    4. 选择**添加应用程序**。
2. [安装 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。
3. 登录 Azure：```az account```
4. 使用上面设置的**应用程序 ID URI** 请求访问令牌：```az account get-access-token --resource=<APP-ID-URI>```

### 使用 Postman 获取访问令牌

1. [安装 Postman](https://www.postman.com/downloads/) 或使用 [Postman Web 应用](https://web.postman.co/)。
2. 创建新的 **Post** 请求，使用以下表单数据：
    1. URL：```<Authority>/<tenant-ID>/oauth2/token```，其中 **Authority** 是您的应用程序所在的租户，在上面配置，**Tenant ID** 来自您的 Azure 应用注册。
        1. 如果使用 Azure Active Directory V2，则改用 URL：```<Authority>/<tenant-ID>/oauth2/v2.0/token```。
    2. *client_id*：服务客户端的**客户端 ID**。
    3. *grant_type*："client_credentials"
    4. *client_secret*：服务客户端的**客户端机密**。
    5. *resource*：资源应用程序的**应用程序 ID URI**。
        1. 如果使用 Azure Active Directory V2，则不要设置 *resource*，而是设置 *scope*：```<Application ID URI>/.default```，其中 Application ID URI 是您的资源应用程序的。
3. 选择**发送**以检索访问令牌。

## 总结

在本操作指南中，您学习了如何通过 Azure 配置 Medical Imaging Server for DICOM 的身份验证设置。要了解如何在开发和测试场景中管理身份验证，请参阅[使用 Identity Server 进行开发](../development/identity-server-authentication.md)。
