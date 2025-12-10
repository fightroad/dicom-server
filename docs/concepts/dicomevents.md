# DICOM 事件

事件是 Azure Health Data Services 中的通知和订阅功能。事件使客户能够利用和增强医学数字成像和通信 (DICOM) 图像的分析和工作流。当 DICOM 图像更改成功写入 Azure Health Data Services 时，事件功能会向事件订阅者发送通知消息。这些事件通知可以发送到多个端点，以触发自动化，从启动工作流到发送电子邮件和短信，以支持源自健康数据的更改。事件功能与 Azure Event Grid 服务集成，并为 Azure Health Data Services 工作区创建系统主题。

## 事件类型：

DicomImageCreated - DICOM 图像成功创建后发出的事件。

DicomImageDeleted - DICOM 图像成功删除后发出的事件。

## 事件消息结构：

|名称 | 类型 | 必需	| 描述
|-----|------|----------|-----------|
|topic	| string	| 是	| 主题是 Azure Health Data Services 工作区的 Azure 资源 ID。
|subject | string | 是 | 已更改的 DICOM 图像的统一资源标识符 (URI)。客户可以使用带有 https:// 方案的主题访问图像。客户应使用 dataVersion 或 data.resourceVersionId 访问与此事件相关的特定数据版本。
| eventType	| string(enum)	| 是	| DICOM 图像上的更改类型。
| eventTime	| string(datetime)	| 是	| 提交 DICOM 图像更改的 UTC 时间。
| id	| string	| 是	| 事件的唯一标识符。
| data	| object	| 是	| DICOM 图像更改事件详细信息。
| data.imageStudyInstanceUid	| string	| 是 | 图像的 Study Instance UID
| data.imageSeriesInstanceUid	| string	| 是	| 图像的 Series Instance UID
| data.imageSopInstanceUid	| string	| 是	| 图像的 SOP Instance UID
| data.serviceHostName	| string	| 是	| 发生更改的 dicom 服务的主机名。 
| data.sequenceNumber	| int	| 是	| DICOM 服务中更改的序列号。每个图像创建和删除在服务中都有一个唯一的序列。此数字与 DICOM 服务的 Change Feed 的序列号相关联。使用此序列号查询 DICOM 服务 Change Feed 将为您提供创建此事件的更改。
| dataVersion	| string	| 否	| DICOM 图像的数据版本
| metadataVersion	| string	| 否	| 事件元数据的架构版本。这由 Azure Event Grid 定义，大多数时候应该是常量。

## 示例

## Microsoft.HealthcareApis.DicomImageCreated

### Event Grid 架构

```
{
  "id": "d621839d-958b-4142-a638-bb966b4f7dfd",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.HealthcareApis/workspaces/{workspace-name}",
  "subject": "{dicom-account}.dicom.azurehealthcareapis.com/v1/studies/1.2.3.4.3/series/1.2.3.4.3.9423673/instances/1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
  "data": {
    "imageStudyInstanceUid": "1.2.3.4.3",
    "imageSeriesInstanceUid": "1.2.3.4.3.9423673",
    "imageSopInstanceUid": "1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
    "serviceHostName": "{dicom-account}.dicom.azurehealthcareapis.com",
    "sequenceNumber": 1
  },
  "eventType": "Microsoft.HealthcareApis.DicomImageCreated",
  "dataVersion": "1",
  "metadataVersion": "1",
  "eventTime": "2022-09-15T01:14:04.5613214Z"
}
```

### Cloud Events 架构

```
{
  "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.HealthcareApis/workspaces/{workspace-name}",
  "subject": "{dicom-account}.dicom.azurehealthcareapis.com/v1/studies/1.2.3.4.3/series/1.2.3.4.3.9423673/instances/1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
  "type": "Microsoft.HealthcareApis.DicomImageCreated",
  "time": "2022-09-15T01:14:04.5613214Z",
  "id": "d621839d-958b-4142-a638-bb966b4f7dfd",
  "data": {
    "imageStudyInstanceUid": "1.2.3.4.3",
    "imageSeriesInstanceUid": "1.2.3.4.3.9423673",
    "imageSopInstanceUid": "1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
    "serviceHostName": "{dicom-account}.dicom.azurehealthcareapis.com",
    "sequenceNumber": 1
  },
  "specVersion": "1.0"
}
```

## Microsoft.HealthcareApis.DicomImageDeleted

### Event Grid 架构

```
{
  "id": "eac1c1a0-ffa8-4b28-97cc-1d8b9a0a6021",
  "topic": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.HealthcareApis/workspaces/{workspace-name}",
  "subject": "{dicom-account}.dicom.azurehealthcareapis.com/v1/studies/1.2.3.4.3/series/1.2.3.4.3.9423673/instances/1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
  "data": {
    "imageStudyInstanceUid": "1.2.3.4.3",
    "imageSeriesInstanceUid": "1.2.3.4.3.9423673",
    "imageSopInstanceUid": "1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
    "serviceHostName": "{dicom-account}.dicom.azurehealthcareapis.com",
    "sequenceNumber": 2
  },
  "eventType": "Microsoft.HealthcareApis.DicomImageDeleted",
  "dataVersion": "1",
  "metadataVersion": "1",
  "eventTime": "2022-09-15T01:16:07.5692209Z"
}
```

### Cloud Events 架构

```
{
  "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.HealthcareApis/workspaces/{workspace-name}",
  "subject": "{dicom-account}.dicom.azurehealthcareapis.com/v1/studies/1.2.3.4.3/series/1.2.3.4.3.9423673/instances/1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
  "type": "Microsoft.HealthcareApis.DicomImageDeleted",
  "time": "2022-09-15T01:14:04.5613214Z",
  "id": "eac1c1a0-ffa8-4b28-97cc-1d8b9a0a6021",
  "data": {
    "imageStudyInstanceUid": "1.2.3.4.3",
    "imageSeriesInstanceUid": "1.2.3.4.3.9423673",
    "imageSopInstanceUid": "1.3.6.1.4.1.45096.2.296485376.2210.1633373143.864442",
    "serviceHostName": "{dicom-account}.dicom.azurehealthcareapis.com",
    "sequenceNumber": 2
  },
  "specVersion": "1.0"
}
```

## 常见问题

### 我可以将事件与 Azure Health Data Services DICOM 服务以外的其他 DICOM 服务一起使用吗？
不可以。Azure Health Data Services 事件功能目前仅支持 Azure Health Data Services DICOM 服务。

### 事件支持哪些 DICOM 图像事件？
事件从以下 DICOM 服务类型生成：

DicomImageCreated - DICOM 图像成功创建后发出的事件。

DicomImageDeleted - DICOM 图像成功删除后发出的事件。

### 事件消息的有效负载是什么？
有关事件消息结构以及必需和非必需元素的详细描述，请参阅 `事件消息结构` 部分。

### 事件消息的吞吐量是多少？
DICOM 事件的吞吐量由 DICOM 服务和 Event Grid 的吞吐量决定。当对 DICOM 服务的请求成功时，它将返回 2xx HTTP 状态代码。它还会生成 DICOM 图像更改事件。当前限制是每个工作区中所有 DICOM 服务实例每秒 5,000 个事件。

### 使用事件如何收费？
使用 Azure Health Data Services 事件不收取额外费用。但是，Event Grid 的适用费用将计入您的 Azure 订阅。

### 如何分别订阅同一工作区中的多个 DICOM 服务？
您可以使用 Event Grid 过滤功能。事件消息有效负载中有唯一标识符来区分不同的帐户和工作区。您可以在源字段中找到工作区的全局唯一标识符，这是 Azure 资源 ID。您可以在 `data.serviceHostName` 字段中找到该工作区中的唯一 DICOM 帐户名称。创建订阅时，可以使用过滤运算符选择要在该订阅中获取的事件。

### 我可以将同一订阅者用于多个工作区或多个 DICOM 帐户吗？
可以。我们建议您为每个单独的 DICOM 帐户使用不同的订阅者，以便在隔离的范围内处理。

### Event Grid 是否与 HIPAA 和 HITRUST 合规性义务兼容？
是的。Event Grid 支持客户的健康保险流通与责任法案 (HIPAA) 和健康信息信任联盟 (HITRUST) 义务。有关更多信息，请参阅 Microsoft Azure 合规性产品。

### 接收事件消息的预期时间是多少？
平均而言，您应该在成功的 HTTP 请求后十秒内收到事件消息。99.99% 的事件消息应在二十秒内送达，除非达到 DICOM 服务或 Event Grid 的限制。

### 是否可能收到重复的事件消息？
是的。Event Grid 保证至少一次事件消息传递（推送模式）。由于随机原因，事件传递请求可能会返回瞬态失败状态代码。在这种情况下，Event Grid 会将其视为传递失败并重新发送事件消息。有关更多信息，请参阅 Azure Event Grid 传递和重试。

通常，我们建议开发人员确保事件订阅者的幂等性。事件 ID 或消息内容数据属性中所有字段的组合对于每个事件都是唯一的。开发人员可以依靠它们进行去重。
