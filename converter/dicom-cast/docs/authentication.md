# Authentication（身份验证）

DICOM Cast 支持连接需要身份验证的 DICOM 与 FHIR 服务器，当前两者均支持三种认证方式。可在应用配置中设置对应服务器的 `Authentication` 属性。

## Managed Identity（托管身份）

使用已部署的 DICOM Cast 实例自身的身份与服务器通信。

```json
{
  "DicomWeb": {
    "Endpoint": "https://dicom-server.example.com",
    "Authentication": {
      "Enabled": true,
      "AuthenticationType": "ManagedIdentity",
      "ManagedIdentityCredential": {
        "Resource": "https://dicom-server.example.com/"
      }
    }
  }
}
```

## OAuth2 Client Credential

使用 `client_credentials` 授权获取身份与服务器通信。

```json
{
  "DicomWeb": {
    "Endpoint": "https://dicom-server.example.com",
    "Authentication": {
      "Enabled": true,
      "AuthenticationType": "OAuth2ClientCredential",
      "OAuth2ClientCredential": {
        "TokenUri": "https://idp.example.com/connect/token",
        "Resource": "https://dicom-server.example.com",
        "Scope": "https://dicom-server.example.com",
        "ClientId": "bdba742b-8138-4b7c-a6d8-03cbb7a8c053",
        "ClientSecret": "d8147077-d907-4551-8f40-90c6e86f3f0e"
      }
    }
  }
}
```

## OAuth2 User Password

使用 `password` 授权获取身份与服务器通信。

```json
{
  "DicomWeb": {
    "Endpoint": "https://dicom-server.example.com",
    "Authentication": {
      "Enabled": true,
      "AuthenticationType": "OAuth2UserPasswordCredential",
      "OAuth2ClientCredential": {
        "TokenUri": "https://idp.example.com/connect/token",
        "Resource": "https://dicom-server.example.com",
        "Scope": "https://dicom-server.example.com",
        "ClientId": "bdba742b-8138-4b7c-a6d8-03cbb7a8c053",
        "ClientSecret": "d8147077-d907-4551-8f40-90c6e86f3f0e",
        "Username": "user@example.com",
        "Password": "randomstring"
      }
    }
  }
}
```

## Secrets 管理

当前提供两种方式存储机密：

### User-Secrets

当 `EnvironmentName` 为 `Development` 时启用。详见 [ASP.NET Core 开发环境安全存储机密](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-3.1)。

### KeyVault

在配置中填写 `KeyVault:Endpoint` 可启用 KeyVault 读取机密。应用启动时将使用[应用当前身份](https://docs.microsoft.com/en-us/aspnet/core/security/key-vault-configuration?view=aspnetcore-3.1#use-managed-identities-for-azure-resources)读取 KeyVault 并添加配置提供程序。

以下为配置 OAuth2ClientCredential 认证时需要添加到 KeyVault 的示例：

* 在 KeyVault 中为 Medical Imaging Server for DICOM 添加与身份验证相关的机密，例如：
    - DicomWeb--Authentication--Enabled : True
    - DicomWeb--Authentication--AuthenticationType : OAuth2ClientCredential
    - DicomWeb--Authentication--OAuth2ClientCredential--TokenUri : ```<AAD tenant token uri>```
    - DicomWeb--Authentication--OAuth2ClientCredential--Resource : ```资源应用的 Application ID URI```
    - DicomWeb--Authentication--OAuth2ClientCredential--Scope : ```资源应用的 Application ID URI```
    - DicomWeb--Authentication--OAuth2ClientCredential--ClientId : ```客户端应用的 Client Id```
    - DicomWeb--Authentication--OAuth2ClientCredential--ClientSecret : ```客户端应用的密钥```
* 为 FHIR&trade; 服务器添加类似的机密。
* 停止并重启容器以加载新配置。
