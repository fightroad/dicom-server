# Azure Active Directory 授权

本操作指南向您展示如何通过 Azure 配置 Medical Imaging Server for DICOM 的授权设置。要完成此配置，您将：

1. **在 Azure AD 中更新资源应用程序**：此资源应用程序将是 Medical Imaging Server for DICOM 的表示，可用于授权和获取令牌。需要更新应用程序注册以创建 appRoles。
2. **在 Azure AD 中分配应用程序角色**：需要为客户端应用程序注册、用户和组分配在应用程序注册上定义的角色。
3. **向 Medical Imaging Server for DICOM 提供配置**：更新资源应用程序后，您将设置 Medical Imaging Server for DICOM App Service 的授权设置。

## 先决条件

1. **完成身份验证配置**：启用身份验证的说明可以在 [Azure Active Directory 身份验证](enable-authentication-with-tokens.md) 文章中找到。

## 授权设置概述

配置中公开的当前授权设置如下：

```json

{
  "DicomServer" : {
    "Security": {
      "Authorization": {
        "Enabled": true,
        "RolesClaim": "role",
        "Roles": [
            <在 ROLES.JSON 中定义>
        ]
      }
    }
  }
}
```

| 元素                    | 描述 |
| -------------------------- | --- |
| Authorization:Enabled      | 服务器是否启用了任何授权功能。 |
| Authorization:RolesClaim   | 标识包含分配角色的 jwt 声明。这由 `DevelopmentIdentityProvider` 自动设置。 |
| Authorization:Roles        | 定义的角色。角色通过 `roles.json` 定义。[更多信息可以在这里找到](../development/roles.md) |

## 使用 Azure AD 设置授权

### Azure AD 说明

#### 创建应用角色
向 AAD 应用程序添加应用角色的说明可以在[本文档文章](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)中找到。本文档还可选地展示了如何将应用角色分配给应用程序。

创建的应用角色需要与 `roles.json` 中找到的角色名称匹配。

#### 将用户分配到应用角色
这可以通过 [Azure 门户](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/add-application-portal-assign-users) 或 [PowerShell cmdlet](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/assign-user-or-group-access-portal#assign-users-and-groups-to-an-app-using-powershell) 完成。

### 向 Medical Imaging Server for DICOM 提供配置
1. 确保您已将 `roles.json` 部署到 Web 应用程序
2. 更新配置以具有以下两个设置
    * `DicomServer:Security:Authorization:Enabled` = `true`
    * `DicomServer:Security:Authorization:RolesClaim` = `"role"`

## 总结

在本操作指南中，您学习了如何通过 Azure 配置 Medical Imaging Server for DICOM 的授权设置。
