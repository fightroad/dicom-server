# Medical Imaging Server for DICOM 常见问题

## Medical Imaging Server for DICOM 是什么？

Medical Imaging Server for DICOM 是一个开源的 DICOMweb&trade; 服务器，可轻松部署到 Azure。它允许与任何支持 DICOMweb&trade; 的系统进行基于标准的通信以进行数据交换，并引入 DICOM 和 FHIR 之间的集成。服务器识别并提取 DICOM 元数据，并将其注入到 FHIR 端点（如 Azure API for FHIR 或 FHIR Server for Azure）中，以创建患者数据的整体视图。

## 使用 Medical Imaging Server for DICOM 的关键要求是什么？

Medical Imaging Server for DICOM 需要 Azure 订阅来配置和运行所需的组件。默认情况下，这些组件在现有或新的 Azure 资源组中创建，以简化管理。此外，还需要 Azure Active Directory 帐户。

## 使用 Medical Imaging Server for DICOM 的数据存储在哪里？

客户控制 Medical Imaging Server for DICOM 持久化的所有数据。以下组件用于持久化数据：
- Blob 存储：持久化所有 DICOM 数据和元数据
- Azure SQL：索引 DICOM 元数据的子集以支持查询，并维护可查询的变更日志
- Azure Key Vault：存储关键安全信息

## 哪些数据格式与 Medical Imaging Server for DICOM 兼容？

Medical Imaging Server for DICOM 公开了一个与 NEMA 指定和维护的 [DICOMweb&trade; 标准](https://www.dicomstandard.org/dicomweb/) 兼容的 REST API。

服务器不支持 DICOM DIMSE，它主要在局域网上工作，不适合现代基于互联网的 API。DIMSE 是一个非常流行的标准，几乎所有医学影像设备都使用它来与医疗提供者的医学影像解决方案的其他组件（如 PACS（图片存档和通信系统）和医学影像查看器）进行通信。然而，许多现代系统，特别是 PACS 和查看器，已经开始支持相关（且兼容）的 DICOMweb&trade; 标准。对于仅支持 DICOM DIMSE 的系统，有适配器可用于在本地支持 DIMSE 的系统与 Medical Imaging Server for DICOM 之间实现无缝通信。

## Medical Imaging Server for DICOM 支持什么版本的 DICOM？

DICOM 标准自 1993 年以来一直固定在版本 3.0。但是，该标准继续通过各种工作组添加破坏性和非破坏性变更。

没有一个产品，包括 Medical Imaging Server for DICOM，完全支持 DICOM 标准。相反，每个产品都包含一个 DICOM 符合性文档，该文档明确指定了支持的内容。（传统上不明确说明不支持的功能。）符合性文档可在[这里](conformance-statement.md)找到。

## Medical Imaging Server for DICOM 中的 REST API 版本控制是什么？

Medical Imaging Server for DICOM 具有用于访问服务器的 REST API 版本。版本在请求中指定为 URL 路径。有关更多信息，请访问 [API 版本控制文档](../api-versioning.md)。

## Medical Imaging Server for DICOM 是否存储任何 PHI？

绝对是的。Medical Imaging Server for DICOM 的核心目标之一是支持标准和创新的放射科医生工作流。这些工作流需要使用 PHI 数据。

## Medical Imaging Server for DICOM 如何维护隐私和安全？

信任、数据隐私和安全是 Microsoft 的最高优先级，并且仍然是管理云中 PHI 数据的基础。Medical Imaging Server for DICOM 旨在支持安全性和隐私。OSS 代码将 DICOM 图像的结构化元数据映射到 FHIR 数据框架，允许通过 FHIR API 进行下游数据交换。

Microsoft Azure 提供了一套全面的产品，帮助您的组织遵守管理数据收集和使用的国家、地区和特定行业的要求。[了解有关 Azure 合规性的更多信息](https://azure.microsoft.com/overview/trusted-cloud/compliance/)。

## DICOM 是什么？

DICOM（医学数字成像和通信）是传输、存储、检索、打印、处理和显示医学影像信息的国际标准，是医疗保健领域接受的主要医学影像标准。尽管存在一些例外（牙科、兽医），但几乎所有医学专业、设备制造商、软件供应商和个人从业者在涉及影像的任何医疗工作流程的某个阶段都依赖 DICOM。DICOM 确保医学影像符合质量标准，从而可以保持诊断的准确性。大多数影像模式，包括 CT、MRI 和超声，都必须符合 DICOM 标准。DICOM 格式的图像需要通过专门的 DICOM 应用程序访问和使用。

## 检索、查询和存储之间的区别是什么？

查询、检索和存储是标准的 DICOMweb&trade; 动词。查询 (QIDO) 搜索 DICOM 对象。QIDO 使您能够按患者 ID 搜索研究、序列和实例。检索 (WADO) 使您能够通过引用检索特定的研究、序列和实例。存储 (STOW-RS) 使您能够将特定实例存储到 DICOM 服务器。

您可以从 [DICOMweb&trade;](https://www.dicomstandard.org/dicomweb) 了解更多关于 QIDO、WADO 和 STOW 的详细信息。

## 支持哪些版本的 .NET？

Web 服务器始终支持最新的 .NET 版本，无论是标准期限支持 (STS) 还是长期支持 (LTS) 版本。另一方面，用于执行长时间运行操作的 Azure Functions 根据 Azure Functions 框架提供的支持，支持最新的 LTS 版本的 .NET。Medical Imaging Server for DICOM docker 镜像始终包含必要的运行时组件。
