# 使用 Identity Server 进行开发

本文还解释了如何在不使用 AAD 集成的情况下，使用 Identity Provider 在开发和测试场景中管理身份验证。要了解有关身份验证设置的更多信息，请参阅[身份验证设置概述](../how-to-guides/enable-authentication-with-tokens.md#L15)。

对于 F5 体验和测试环境，包含一个进程内身份提供者，可以作为 DICOMweb&trade; API 的身份验证提供者。

## TestAuthEnvironment.json

位于根目录的 [`testauthenvironment.json`](/testauthenvironment.json) 文件保存服务器使用的配置。**此文件仅用于本地和测试环境。** 此文件中表示的项目包括 API 可用的角色以及有权访问 API 的用户和客户端应用程序。在 F5 体验和本地测试期间，用户和客户端应用程序的密码/机密与项目的 id 相同。

## 为测试启用开发身份提供者

[启动设置](/src/Microsoft.Health.Dicom.Web/Properties/launchSettings.json) 具有 `DicomWebSecurityEnabled` 配置文件，该文件具有用于启用开发身份提供者的预设设置。

## 使用内置 IdentityServer 进行身份验证

要获取令牌，请发出以下命令。

```
POST /connect/token HTTP/1.1
Host: https://localhost:63838
Content-Type: application/x-www-form-urlencoded

client_id=globalAdminServicePrincipal&client_secret=globalAdminServicePrincipal&grant_type=client_credentials&scope=health-api
```

要使用 Dicom API 进行身份验证，请从上一个命令中获取 `access_token`，并将其作为 `Authorization` 标头附加，语法为：`Bearer {access_token}`。

示例令牌响应

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6Ijc4YWJlMDM0OGEyNDg4NzU0MmUwOGJjNTg3YWFjY2Q4IiwidHlwIjoiSldUIn0.eyJuYmYiOjE1MjM1NTQ3OTQsImV4cCI6MTUyMzU1ODM5NCwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo1MzcyNyIsImF1ZCI6WyJodHRwOi8vbG9jYWxob3N0OjUzNzI3L3Jlc291cmNlcyIsImZoaXItYXBpIl0sImNsaWVudF9pZCI6Imtub3duLWNsaWVudC1pZCIsInNjb3BlIjpbImZoaXItYXBpIl19.pZWIWy3RdDHp5zgcYs8bb9VrxIHXbYu8LolC3YTy6xWsPxMoPUQwbAltYmC6WDXFiDygpsC5ofkGlR4BH0Bt1FMvFWqFYhPcOOKvBqLLc055EHZfTcNcmiUUf4y4KRuQFqWZsH_HrfWwykSGVio2OnYcQvytrbjAi_EzHf2vrHJUHX2JFY4A_F6WpJbQiI1hUVEOd7h1jfmAptWlNGwNRbCF2Wd1Hf_Hodym8mEOKQz21VHdvNJ_B-owPMvLjalV5Nrvpv0yC9Ly5YablrkzB583eHwQNSA7A4ZMm49O8MWv8kUwwF5TF0lJJDyyw3ruqmPWCM-058chenU0rtCsPQ",
    "expires_in": 3600,
    "token_type": "Bearer"
}
```

示例授权标头
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6Ijc4YWJlMDM0OGEyNDg4NzU0MmUwOGJjNTg3YWFjY2Q4IiwidHlwIjoiSldUIn0.eyJuYmYiOjE1MjM1NTQ3OTQsImV4cCI6MTUyMzU1ODM5NCwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo1MzcyNyIsImF1ZCI6WyJodHRwOi8vbG9jYWxob3N0OjUzNzI3L3Jlc291cmNlcyIsImZoaXItYXBpIl0sImNsaWVudF9pZCI6Imtub3duLWNsaWVudC1pZCIsInNjb3BlIjpbImZoaXItYXBpIl19.pZWIWy3RdDHp5zgcYs8bb9VrxIHXbYu8LolC3YTy6xWsPxMoPUQwbAltYmC6WDXFiDygpsC5ofkGlR4BH0Bt1FMvFWqFYhPcOOKvBqLLc055EHZfTcNcmiUUf4y4KRuQFqWZsH_HrfWwykSGVio2OnYcQvytrbjAi_EzHf2vrHJUHX2JFY4A_F6WpJbQiI1hUVEOd7h1jfmAptWlNGwNRbCF2Wd1Hf_Hodym8mEOKQz21VHdvNJ_B-owPMvLjalV5Nrvpv0yC9Ly5YablrkzB583eHwQNSA7A4ZMm49O8MWv8kUwwF5TF0lJJDyyw3ruqmPWCM-058chenU0rtCsPQ
```

## 资源

要了解如何通过 Azure AD 管理身份验证，请参阅 [Azure AD 身份验证](../how-to-guides/enable-authentication-with-tokens.md)。
