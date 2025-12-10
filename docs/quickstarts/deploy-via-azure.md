# 使用 Azure 门户部署 Medical Imaging Server for DICOM

在本快速入门中，您将学习如何使用 Azure 门户部署 Medical Imaging Server for DICOM。

如果您没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free)。

拥有订阅后，单击下面的按钮开始部署：<br/>
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fsamples%2Ftemplates%2Fdefault-azuredeploy.json" target="_blank"><img src="https://aka.ms/deploytoazurebutton"/></a>

有关如何部署 ARM 模板的说明可以在以下文档中找到
* [通过门户部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-portal)
* [通过 CLI 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-cli)
* [通过 PowerShell 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-powershell)

## 输入帐户详细信息

1. 选择您的 Azure 订阅。
2. 选择现有资源组或创建新资源组。
3. 选择部署 Medical Imaging Server for DICOM 的区域。
4. 为部署选择服务名称。请注意，服务名称将包含在您用于访问应用程序的 URL 中。

![必需部署配置](../images/required-deployment.png)

## 配置部署设置

为 Medical Imaging Server for DICOM 配置剩余的部署设置。默认设置适合优秀的开发/测试或概念验证环境，因为它们价格低廉，但在中小型负载下表现良好。对于生产环境，建议升级到区域冗余存储、故障转移数据库和自动扩展应用程序服务器。

有关更多配置说明，请参考[配置 Medical Imaging Server for DICOM](../how-to-guides/configure-dicom-server-settings.md)。

> 注意：设置 SQL 管理员密码时，请参考 [SQL Server 密码复杂性要求](https://docs.microsoft.com/sql/relational-databases/security/password-policy?view=sql-server-ver15)。

## 总结

在本快速入门中，您学习了如何使用 Azure 门户部署和配置 Medical Imaging Server for DICOM。

部署完成后，您可以使用 Azure 门户导航到新创建的 App Service 以查看详细信息。访问 Medical Imaging Server for DICOM 的默认 URL 将是：```https://<SERVICE NAME>.azurewebsites.net```。发出请求时，请确保在 URL 中指定版本。更多信息可以在 [API 版本控制文档](../api-versioning.md) 中找到

要开始使用新部署的 Medical Imaging Server for DICOM，请参考以下文档：

* [配置 Medical Imaging Server for DICOM](../how-to-guides/configure-dicom-server-settings.md)
* [使用 Medical Imaging Server for DICOM API](../tutorials/use-the-medical-imaging-server-apis.md)
* [通过 Electron 工具上传 DICOM 文件](../../tools/dicom-web-electron)
* [启用 Azure AD 身份验证](../how-to-guides/enable-authentication-with-tokens.md)
