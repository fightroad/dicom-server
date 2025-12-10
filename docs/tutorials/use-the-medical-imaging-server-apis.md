# 使用 Medical Imaging Server for DICOM 的 DICOMweb&trade; 标准 API

本教程概述了如何将 DICOMweb&trade; 标准 API 与 Medical Imaging Server for DICOM 一起使用。

Medical Imaging Server for DICOM 支持 DICOMweb&trade; 标准的子集。支持包括：

- Store (STOW-RS)
- Retrieve (WADO-RS)
- Search (QIDO-RS)

此外，支持以下非标准 API：

- Delete
- Change Feed

您可以在我们的[符合性声明](../resources/conformance-statement.md)中了解更多关于我们对各种 DICOM Web 标准 API 的支持。

## 先决条件

为了使用 DICOMweb&trade; 标准 API，您必须部署 Medical Imaging Server for DICOM 的实例。如果您尚未部署 Medical Imaging Server，请[将 Medical Imaging Server 部署到 Azure](../quickstarts/deploy-via-azure.md)。

部署完成后，您可以使用 Azure 门户导航到新创建的 App Service 以查看详细信息和服务 URL。发出请求时，请确保在 URL 中指定版本。更多信息可以在 [API 版本控制文档](../api-versioning.md) 中找到

## 与 Medical Imaging Server for DICOM 一起使用的各种方法概述

因为 Medical Imaging Server for DICOM 作为 REST API 公开，您可以使用任何现代开发语言访问它。有关使用服务的语言无关信息，请参考我们的[符合性声明](../resources/conformance-statement.md)。

要查看特定语言的示例，请参阅下面的示例。或者，如果您打开 Postman 集合，您可以看到包括 Go、Java、Javascript、C#、PHP、C、NodeJS、Objective-C、OCaml、PowerShell、Python、Ruby 和 Swift 在内的多种语言的示例。

### C#

C# 示例使用此存储库中包含的库来简化对 API 的访问。参考 [C# 示例](../tutorials/use-dicom-web-standard-apis-with-c%23.md) 了解如何将 C# 与 Medical Imaging Server for DICOM 一起使用。

### cURL

cURL 是一种用于调用 Web 端点的常见命令行工具，可用于几乎任何操作系统。[下载 cURL](https://curl.haxx.se/download.html) 开始使用。要使用示例，您需要用实例名称替换服务器名称，并将此存储库中的[示例 DICOM 文件](../dcms) 下载到本地文件系统上的已知位置。参考 [cURL 示例](../tutorials/use-dicom-web-standard-apis-with-curl.md) 了解如何将 cURL 与 Medical Imaging Server for DICOM 一起使用。

### Postman

Postman 是设计、构建和测试 REST API 的优秀工具。[下载 Postman](https://www.postman.com/downloads/) 开始使用。您可以在 [Postman 学习网站](https://learning.postman.com/) 了解如何有效使用 Postman。

Postman 和 DICOMweb&trade; 标准的一个重要注意事项：Postman 只能支持使用 DICOM 标准中定义的单部分有效负载上传 DICOM 文件。这是因为 Postman 无法支持 multipart/related POST 请求中的自定义分隔符。有关更多信息，请参阅 [https://github.com/postmanlabs/postman-app-support/issues/576](https://github.com/postmanlabs/postman-app-support/issues/576) 了解有关此错误的更多信息。因此，Postman 集合中使用 multipart 请求上传 DICOM 文档的所有示例都带有前缀 [will not work - see description]。使用单部分请求上传的示例包含在集合中，并带有前缀 "Store-Single-Instance"。

要使用 Postman 集合，您需要将集合下载到本地，然后通过 Postman 导入集合，这些都可以在这里找到：[Postman 集合示例](../resources/Conformance-as-Postman.postman_collection.json)。在集合中，当要求您指定基础 URL 时，它是您的服务的完整 URL，后跟版本（例如 `https://<app_service_url>/v<version>/`）。有关版本控制的更多信息，请访问 [API 版本控制文档](../api-versioning.md)。

## 总结

本教程概述了 Medical Imaging Server for DICOM 支持的 API。使用以下工具开始使用这些 API：

- [使用 C# 的 DICOM Web 标准 API](../tutorials/use-dicom-web-standard-apis-with-c%23.md)
- [使用 cURL 的 DICOM Web 标准 API](../tutorials/use-dicom-web-standard-apis-with-curl.md)
- [使用 Postman 示例集合的 DICOM Web 标准 API](../resources/Conformance-as-Postman.postman_collection.json)
