# 部署 DICOM Cast

在本快速入门中，您将学习如何为 Medical Imaging Server for DICOM 部署 DICOM Cast。DICOM Cast 是一项服务，它将 Medical Imaging Server for DICOM 元数据推送到 FHIR&trade; 服务器，以支持跨临床和影像数据的集成查询。要了解有关 DICOM Cast 架构的更多信息，请参考 [DICOM Cast 概念](../concepts/dicom-cast.md)。

完成本快速入门后，您将为 Medical Imaging Server for DICOM 部署 DICOM Cast。要了解如何将 Medical Imaging Server for DICOM 元数据同步到 FHIR&trade; 服务器，请参阅[使用 DICOM Cast 将 DICOM 元数据同步到 FHIR&trade;](../how-to-guides/sync-dicom-metadata-to-fhir.md)。

## 通过 Docker 部署

如果您想通过 docker-compose 部署，请遵循快速入门[通过 Docker 部署](deploy-via-docker.md)。这将部署 Medical Imaging Server for DICOM 和 DICOM Cast。

## 通过 Azure 与 Azure Healthcare APIs 部署

DICOM Cast 可以作为 Azure Container Instance 部署，目标是具有 FHIR 服务和 DICOM 服务的 Azure Healthcare API 工作区，使用提供的 [ARM 模板](/converter/dicom-cast/samples/templates/deploy-with-healthcareapis.json)。

> 注意：目前 DICOM Cast 仅支持 FHIR 的 R4。

### 部署

如果您有 Azure 订阅，请单击下面的链接部署到 Azure：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fconverter%2Fdicom-cast%2Fsamples%2Ftemplates%2Fdeploy-with-healthcareapis.json" target="_blank">
    <img src="https://aka.ms/deploytoazurebutton"/>
</a>

如果您没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free)。

这将向指定的资源组部署以下资源：

* Azure Container Instance
  + 用于运行 DICOM Cast 可执行文件
  + 使用的镜像通过 `image` 参数指定，默认为最新的 CI 构建
  + 还配置了托管标识
* Application Insights
  + 用于记录 Medical Imaging Server for DICOM 生成的事件和错误
  + 如果指定了 `deployApplicationInsights`，则部署 Application Insights 实例用于日志记录
* 存储帐户
  + 用于跟踪服务状态
  + 在表存储中持久化有关异常的信息
* KeyVault
  + 用于存储存储连接字符串
  + 通过 ACI 上指定的托管标识访问
* Azure Healthcare APIs 工作区
* R4 FHIR 服务
* DICOM 服务
* 将 `DICOM Data Owner` 和 `FHIR Data Contributor` 的角色分配分配给 ACI 托管标识。

有关如何部署 ARM 模板的说明可以在以下文档中找到
* [通过门户部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-portal)
* [通过 CLI 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-cli)
* [通过 PowerShell 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-powershell)

## 通过 Azure 与 OSS 部署

DICOM Cast 可以使用提供的 [ARM 模板](/converter/dicom-cast/samples/templates/default-azuredeploy.json) 作为 Azure Container Instance 部署。

### 先决条件

* 已部署的 [FHIR Server for Azure](https://github.com/microsoft/fhir-server)
* 已部署的 [Medical Imaging Server for DICOM](https://github.com/microsoft/dicom-server)

> 注意：目前 DICOM Cast 仅由开源 FHIR Server for Azure 或 Azure Healthcare APIs FHIR 服务支持，不支持 Azure API for FHIR。确保 FHIR 服务器在 FHIR 的 R4 版本上。

### 部署

如果您有 Azure 订阅，请单击下面的链接部署到 Azure：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fconverter%2Fdicom-cast%2Fsamples%2Ftemplates%2Fdefault-azuredeploy.json" target="_blank">
    <img src="https://aka.ms/deploytoazurebutton"/>
</a>

如果您没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free)。

这将向指定的资源组部署以下资源：

* Azure Container Instance
  + 用于运行 DICOM Cast 可执行文件
  + 使用的镜像通过 `image` 参数指定，默认为最新的 CI 构建
  + 还配置了托管标识
* Application Insights
  + 用于记录 Medical Imaging Server for DICOM 生成的事件和错误
  + 如果指定了 `deployApplicationInsights`，则部署 Application Insights 实例用于日志记录
* 存储帐户
  + 用于跟踪服务状态
  + 在表存储中持久化有关异常的信息
* KeyVault
  + 用于存储存储连接字符串
  + 通过 ACI 上指定的托管标识访问

有关如何部署 ARM 模板的说明可以在以下文档中找到
* [通过门户部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-portal)
* [通过 CLI 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-cli)
* [通过 PowerShell 部署](https://docs.microsoft.com/azure/azure-resource-manager/templates/deploy-powershell)

#### 部署 DICOM 服务器、FHIR 服务器和 DICOM Cast

如果您尚未部署 DICOM 或 FHIR 服务器，可以使用单个 ARM 模板一次性部署 Medical Imaging Server for DICOM、FHIR OSS 服务器和 DICOM Cast 实例。如果您还没有 Azure 订阅，请在部署之前创建一个[免费帐户](https://azure.microsoft.com/free)。

要进行简化的快速部署，请单击下面的链接部署到 Azure：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fsamples%2Ftemplates%2Fdicomcast-quick-deploy.json" target="_blank">
  <img src="https://aka.ms/deploytoazurebutton"/>
</a>

> 注意：此选项非常适合快速设置 DICOM Cast。如果您想自定义配置，需要在部署后进行。

如果您想在部署期间自定义所有 DICOM、FHIR 和 DICOM Cast 设置，请单击下面的链接部署到 Azure：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fsamples%2Ftemplates%2Fdicomcast-fhir-dicom-azuredeploy.json" target="_blank">
  <img src="https://aka.ms/deploytoazurebutton"/>
</a>

这两个 ARM 模板都将部署：

* Medical Imaging Server for DICOM
* [FHIR OSS 服务器](https://github.com/microsoft/fhir-server)
* DICOM Cast

这三个实例将托管在同一资源组和 App Service Plan 中。

## 总结

在本快速入门中，您学习了如何为 Medical Imaging Server for DICOM 部署 DICOM Cast。参考 [DICOM Cast 概念](../concepts/dicom-cast.md) 了解更多信息。要开始使用 DICOM Cast，请参阅[使用 DICOM Cast 将 DICOM 元数据同步到 FHIR&trade;](../how-to-guides/sync-dicom-metadata-to-fhir.md)。要配置启用 Private Link 的 DICOM 服务和 FHIR 服务的 DICOM Cast，请参阅此处的说明：[使用启用 Private Link 的 DICOM 和 FHIR 配置 DICOM Cast](../../converter/dicom-cast/docs/workingWithPrivateLink.md)
