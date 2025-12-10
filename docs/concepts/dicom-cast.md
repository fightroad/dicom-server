# DICOM Cast 概述

DICOM Cast 允许将 Medical Imaging Server for DICOM 的数据同步到 [FHIR Server for Azure](https://github.com/microsoft/fhir-server)，使医疗组织能够集成临床和影像数据。DICOM Cast 通过支持纵向患者数据的简化视图以及有效创建医疗研究、分析和机器学习的队列的能力，扩展了健康数据的使用场景。

## 架构

![架构图](/docs/images/dicom-cast-architecture.png)

1. **轮询批量变更**：DICOM Cast 通过 [Change Feed](../concepts/change-feed.md) 轮询任何变更，该功能捕获 Medical Imaging Server for DICOM 中发生的任何变更。
2. **获取相应的 FHIR 资源（如果有）**：如果任何变更对应 FHIR 资源，DICOM Cast 将获取这些变更。DICOM Cast 将 DICOM 标签同步到 FHIR 资源类型 *Patient* 和 *ImagingStudy*。
3. **合并 FHIR 资源并作为事务中的 bundle PUT**：与 DICOM Cast 捕获的变更对应的 FHIR 资源将被合并。FHIR 资源将作为事务中的 bundle PUT 到您的 Azure API for FHIR 服务器。
4. **持久化状态并处理下一批**：DICOM Cast 然后将持久化当前状态，为下一批变更做准备。

DICOM Cast 的当前实现支持：

- 从 DICOM 变更源读取并写入 FHIR 服务器的单线程进程。
- 该进程在我们的示例模板中由 Azure Container Instance 托管，但可以在其他地方运行。
- 将 DICOM 标签同步到 *Patient* 和 *ImagingStudy* FHIR 资源类型。
- 配置以在从变更源同步数据到 FHIR 资源类型时忽略无效标签。
    - 如果启用了 `EnforceValidationOfTagValues`，则除非映射的每个标签（见下面的映射）都有效，否则变更源条目不会写入 FHIR 服务器
    - 如果 `EnforceValidationOfTagValues` 被禁用（默认），则如果值无效但不是必需映射的（见下面 Patient 和 Imaging Study 的必需标签），则该特定标签不会被映射，但变更源条目的其余部分将映射到 FHIR 资源。如果必需标签无效，则变更源条目不会写入 FHIR 服务器
- 配置以忽略由于格式错误的 DICOM json 导致的 DicomWebClient 的 Json 解析错误
  - 如果启用了 `IgnoreJsonParsingErrors`，则将跳过格式错误的变更源条目。由于无法解析出表条目所需的 Study、Series 和 Instance UID，此错误不会记录在异常表中
  - 如果 `IgnoreJsonParsingErrors` 被禁用，当 DICOM Cast 尝试处理格式错误的 DICOM json 时将抛出异常
- 将错误存储到 Azure Table Storage
    - 处理变更源条目时的错误根据错误原因持久化到 Azure Table Storage 的不同表中。
        - `InvalidDicomTagExceptionTable`：存储有关任何具有无效值的标签的信息。这里的条目不一定意味着整个变更源条目未存储在 FHIR 中，而是特定值存在验证问题。
        - `DicomFailToStoreExceptionTable`：存储由于变更源条目问题（如无效的必需标签）而未存储到 FHIR 的变更源条目信息。此表中的所有条目都未存储到 FHIR。
        - `FhirFailToStoreExceptionTable`：存储由于 FHIR 服务器问题（如已存在冲突的资源）而未存储到 FHIR 的变更源条目信息。此表中的所有条目都未存储到 FHIR。
        - `TransientRetryExceptionTable`：存储遇到瞬态错误（如 FHIR 服务器太忙）并正在重试的变更源条目信息。此表中的条目记录了它们已重试的次数，但不一定意味着它们最终失败或成功存储到 FHIR。
        - `TransientFailureExceptionTable`：存储具有瞬态错误并经过重试策略后仍无法存储到 FHIR 的变更源条目信息。此表中的所有条目都未能存储到 FHIR。

## 映射

DICOM Cast 的当前实现具有以下映射：

**Patient：**

| 属性 | 标签 ID | 标签名称 | 必需标签？| 备注 |
| :------- | :----- | :------- | :----- | :----- |
| Patient.identifier.where(system = 'system') | (0010,0020) | PatientID | 是 | 患者系统 ID 将设置为 patientSystemId 配置的值或患者 ID DICOM 标签的 Issuer (0010, 0021)，具体取决于 isIssuerIdUsed 布尔设置。如果未定义变量，则默认设置为空字符串。 |
| Patient.name.where(use = 'usual') | (0010,0010) | PatientName | 否 | PatientName 将被拆分为组件并作为 HumanName 添加到 Patient 资源。 |
| Patient.gender | (0010,0040) | PatientSex | 否 | |
| Patient.birthDate | (0010,0030) | PatientBirthDate | 否 | PatientBirthDate 仅包含日期。此实现假设 FHIR 和 DICOM 服务器具有来自相同时区的数据。 |

**Endpoint：**

| 属性 | 标签 ID | 标签名称 | 备注 |
| :------- | :----- | :------- | :--- |
| Endpoint.status ||| 创建 Endpoint 时将使用值 'active'。 |
| Endpoint.connectionType ||| 创建 Endpoint 时将使用系统 'http://terminology.hl7.org/CodeSystem/endpoint-connection-type' 和值 'dicom-wado-rs'。 |
| Endpoint.address ||| 创建 Endpoint 时将使用 DICOMWeb 服务的根 URL。规则在 'http://hl7.org/fhir/imagingstudy.html#endpoint' 中描述 |

**ImagingStudy：**

| 属性 | 标签 ID | 标签名称 | 必需 | 备注 |
| :------- | :----- | :------- | :--- | :--- |
| ImagingStudy.identifier.where(system = 'urn:dicom:uid') | (0020,000D) | StudyInstanceUID | 是 | 值将具有 `urn:oid:` 前缀。 |
| ImagingStudy.status | | | 否 | 创建 ImagingStudy 时将使用值 'available'。 |
| ImagingStudy.modality | (0008,0060) | Modality | 否 | 或者应该是 (0008,0061) ModalitiesInStudy？ |
| ImagingStudy.subject | | | 否 | 它将链接到上面的 Patient。 |
| ImagingStudy.started | (0008,0020), (0008,0030), (0008,0201) | StudyDate, StudyTime, TimezoneOffsetFromUTC | 否 | 有关如何构造时间戳的更多详细信息[如下](#timestamp)。 |
| ImagingStudy.endpoint | | | | 它将链接到上面的 Endpoint。 |
| ImagingStudy.note | (0008,1030) | StudyDescription | 否 | |
| ImagingStudy.series.uid | (0020,000E) | SeriesInstanceUID | 是 | |
| ImagingStudy.series.number | (0020,0011) | SeriesNumber | 否 | |
| ImagingStudy.series.modality | (0008,0060) | Modality | 是 | |
| ImagingStudy.series.description | (0008,103E) | SeriesDescription | 否 | |
| ImagingStudy.series.started | (0008,0021), (0008,0031), (0008,0201) | SeriesDate, SeriesTime, TimezoneOffsetFromUTC | 否 | 有关如何构造时间戳的更多详细信息[如下](#timestamp)。 |
| ImagingStudy.series.instance.uid | (0008,0018) | SOPInstanceUID | 是 | |
| ImagingStudy.series.instance.sopClass | (0008,0016) | SOPClassUID | 是 | |
| ImagingStudy.series.instance.number | (0020,0013) | InstanceNumber | 否| |
| ImagingStudy.identifier.where(type.coding.system='http://terminology.hl7.org/CodeSystem/v2-0203' and type.coding.code='ACSN')) | (0008,0050) | Accession Number | 否 | 参考 http://hl7.org/fhir/imagingstudy.html#notes。 |

### 时间戳

DICOM 具有不同的日期时间 VR 类型。某些标签（如 Study 和 Series）分别存储日期、时间和 UTC 偏移量。这意味着日期可能是部分的。此代码尝试将其转换为 FHIR 服务器允许的部分日期语法。

## 总结

在本概念文档中，我们回顾了 DICOM Cast 的架构和映射。要在 Medical Imaging for DICOM Server 中实现 DICOM Cast，请参考以下文档：

- [DICOM Cast 快速入门](../quickstarts/deploy-dicom-cast.md)
- [将 DICOM 元数据同步到 FHIR](../how-to-guides/sync-dicom-metadata-to-fhir.md)
