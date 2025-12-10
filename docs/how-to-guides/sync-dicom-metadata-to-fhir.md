# 将 Medical Imaging Server for DICOM 元数据同步到 FHIR Server for Azure

在本操作指南中，您将学习如何将 Medical Imaging Server for DICOM 元数据与 FHIR 同步。为此，您将学习如何通过使用托管标识进行身份验证来启用 DICOM Cast。

对于寻求通过 FHIR® 标准集成临床和影像数据的医疗组织，DICOM Cast 支持将 DICOM 图像变更同步到 FHIR&trade; ImagingStudy 资源。这允许您将 DICOM 研究、序列和实例同步到 FHIR&trade; ImagingStudy 资源。

一旦您完成了[先决条件](#先决条件)并在 Medical Imaging Server for DICOM、FHIR 服务器和 DICOM Cast 部署之间[启用了身份验证](#使用托管标识配置身份验证)，您就启用了 DICOM Cast。当您将文件上传到 Medical Imaging Server for DICOM 时，相应的 FHIR 资源将持久化到您的 FHIR 服务器中。

要了解有关 DICOM Cast 的更多信息，请参阅 [DICOM Cast 概念](../concepts/dicom-cast.md)。

## 先决条件

要启用 DICOM Cast，您需要完成以下步骤：

1. [部署 Medical Imaging Server for DICOM](../quickstarts/deploy-via-azure.md)
2. [部署 FHIR 服务器](https://github.com/microsoft/fhir-server)
3. [部署 DICOM Cast](../quickstarts/deploy-dicom-cast.md)

> 注意：部署 OSS FHIR 服务器时，将 **Sql Schema Automatic Updates Enabled** 设置设置为 *true*。这确定是否应在服务器设置时自动初始化和升级 sql 架构。

## 使用托管标识配置身份验证

目前，FHIR Server for Azure 和 Medical Imaging Server for DICOM 都支持三种类型的身份验证：托管标识、OAuth2 客户端凭据和 OAuth2 用户密码。可以通过在给定服务器的 `Authentication` 属性中设置适当的值，通过应用程序设置配置身份验证。有关这三种类型的详细信息，请参阅 [DICOM Cast 身份验证](/converter/dicom-cast/docs/authentication.md)。

本节将提供使用托管标识配置身份验证的端到端指南。

### 为 FHIR 和 DICOM 服务器创建资源应用程序

对于 FHIR 和 DICOM 服务器，您将在 Azure 中创建资源应用程序。请按照以下说明为每个服务器执行一次，一次为 Medical Imaging Server for DICOM，一次为 FHIR 服务器。

1. 登录 [Azure 门户](https://ms.portal.azure.com/)。搜索 **App Services** 并选择 FHIR 或 DICOM App Service。复制 App Service 的 **URL**。
2. 选择 **Azure Active Directory** > **应用注册** > **新注册**：
    1. 输入应用注册的**名称**。
    2. 在**重定向 URI** 中，选择 **Web** 并输入 App Service 的 **URL**。
    3. 选择**注册**。
3. 选择**公开 API** > **设置**。您可以将 URI 指定为应用服务的 **URL** 或使用生成的 App ID URI。选择**保存**。
4. 选择**添加范围**：
    1. 在**范围名称**中，输入 *user_impersonation*。
    2. 在文本框中，添加您希望用户在同意页面上看到的管理员同意显示名称和管理员同意描述。例如，*访问我的应用*。

### 为 FHIR 和 DICOM App Services 设置身份验证

对于 FHIR 和 DICOM 服务器，您将设置身份验证的 Audience 和 Authority。请按照以下说明为每个服务器执行一次，一次为 Medical Imaging Server for DICOM，一次为 FHIR 服务器。

1. 导航到您部署到 Azure 的 App Service。
2. 选择**配置**以更新 **Audience**、**Authority** 和 **Security:Enabled**：
    1. 将 App Service 的**应用程序 ID URI** 设置为 **Audience**。
    2. **Authority** 是您的应用程序所在的租户，例如：```https://login.microsoftonline.com/<tenant-name>.onmicrosoft.com```。
    3.  将 **Security:Enabled** 设置为 ```True```。
    4.  保存对配置的更改。

### 更新 DICOM Cast 的 Key Vault

1. 导航到部署 DICOM Cast 时创建的 DICOM Cast Key Vault。
2. 在菜单栏中选择**访问策略**，然后单击**添加访问策略**。
    1. 在**从模板配置**下，选择**机密管理**。
    2. 在**选择主体**下，单击**未选择**。搜索您的服务主体，单击**选择**，然后单击**添加**。
    3. 选择**保存**。
3. 在菜单栏中选择**机密**，然后单击**生成/导入**。使用下面的表格为 DICOM 和 FHIR 服务器添加机密。对于每个机密，使用**手动上传选项**并单击**创建**：

#### Medical Imaging Server for DICOM 机密

| 名称 | 值 |
| :------- | :----- |
| DICOM--Endpoint | ```<dicom-server-url>``` |
| DicomWeb--Authentication--Enabled | true |
| DicomWeb--Authentication--AuthenticationType | ManagedIdentity |
| DicomWeb--Authentication--ManagedIdentityCredential--Resource | ```<dicom-audience>``` |

#### FHIR 服务器机密

| 名称 | 值 |
| :------- | :----- |
| Fhir--Endpoint | ```<fhir-server-url>``` |
| Fhir--Authentication--Enabled | true |
| Fhir--Authentication--AuthenticationType | ManagedIdentity |
| Fhir--Authentication--ManagedIdentityCredential--Resource | ```<fhir-server-url>``` |

### 重启 DICOM Cast 的 Azure Container Instance

现在您已经为 DICOM Cast 启用了身份验证，您必须停止并启动 Azure Container Instance 以获取新配置：

1. 导航到部署 DICOM Cast 时创建的 Container Instance。
2. 单击**停止**，然后单击**启动**。

## 总结

在本操作指南中，您学习了如何通过使用托管标识进行身份验证来启用 DICOM Cast。现在您可以将 DICOM 文件上传到 Medical Imaging Server for DICOM，相应的 FHIR 资源将填充到您的 FHIR 服务器中。

要使用 OAuth2 客户端凭据或 OAuth2 用户密码管理身份验证，请参阅 [DICOM Cast 身份验证](/converter/dicom-cast/docs/authentication.md)。

有关 DICOM Cast 的概述，请参阅 [DICOM Cast 概念](../concepts/dicom-cast.md)。

要将文件上传到 DICOM 服务器，请参考[使用 Medical Imaging Server API](../tutorials/use-the-medical-imaging-server-apis.md)。

您可以[使用 Postman 访问 FHIR 服务器](https://docs.microsoft.com/azure/healthcare-apis/access-fhir-postman-tutorial) 以查看通过 DICOM Cast 填充的 FHIR 资源。
