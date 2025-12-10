# DICOM 符合性声明

> API 版本 2 是最新的 API 版本，应该使用。
> 转到 [API 版本 2](v2-conformance-statement.md) 查看最新的符合性声明。

**Medical Imaging Server for DICOM** 支持 DICOMweb&trade; 标准的子集。支持包括：

- [Studies Service](#studies-service)
  - [Store (STOW-RS)](#store-stow-rs)
  - [Retrieve (WADO-RS)](#retrieve-wado-rs)
  - [Search (QIDO-RS)](#search-qido-rs)
  - [Delete (非标准)](#delete)
- [Worklist Service (UPS Push and Pull SOPs)](#worklist-service-ups-rs)
  - [Create Workitem](#create-workitem)
  - [Retrieve Workitem](#retrieve-workitem)
  - [Update Workitem](#update-workitem)
  - [Change Workitem State](#change-workitem-state)
  - [Request Cancellation](#request-cancellation)
  - [Search Workitems](#search-workitems)

此外，支持以下非标准 API：
- [Change Feed](../concepts/change-feed.md)
- [Extended Query Tags](../concepts/extended-query-tags.md)

下面的所有路径都包含服务器的隐式基础 URL，例如在本地运行时为 `https://localhost:63838`。

服务使用 REST API 版本控制。请注意，REST API 的版本必须明确指定为基础 URL 的一部分，如下例所示：

`https://localhost:63838/v1/studies`

有关如何在发出请求时指定版本的更多信息，请访问 [API 版本控制文档](../api-versioning.md)。

您可以在 [Postman 集合](../resources/Conformance-as-Postman.postman_collection.json) 中找到支持事务的示例请求。

## 前导码清理

服务忽略 128 字节的文件前导码，并将其内容替换为空字符。这确保通过服务的任何文件都不会受到[恶意前导码漏洞](https://dicom.nema.org/medical/dicom/current/output/chtml/part10/sect_7.5.html)的影响。但是，这也意味着[用于编码双格式内容的前导码](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6489422/)（如 TIFF）不能与服务一起使用。

# Studies Service

[Studies Service](https://dicom.nema.org/medical/dicom/current/output/html/part18.html#chapter_10) 允许用户存储、检索和搜索 DICOM 研究、序列和实例。我们添加了非标准 Delete 事务以启用完整的资源生命周期。

## Store (STOW-RS)

此事务使用 POST 方法存储请求有效负载中包含的研究、序列和实例的表示。

| 方法 | 路径               | 描述 |
| :----- | :----------------- | :---------- |
| POST   | ../studies         | 存储实例。 |
| POST   | ../studies/{study} | 为特定研究存储实例。 |

参数 `study` 对应于 DICOM 属性 StudyInstanceUID。如果指定，任何不属于提供的研究的实例将被拒绝，并返回 `43265` 警告代码。

支持以下响应 `Accept` 标头：

- `application/dicom+json`

支持以下 `Content-Type` 标头：

- `multipart/related; type="application/dicom"`
- `application/dicom`

> 注意：服务器将<u>不会</u>强制或替换与现有数据冲突的属性。所有数据将按提供的方式存储。

尝试存储的每个 DICOM 文件中必须存在以下 DICOM 元素：

- StudyInstanceUID
- SeriesInstanceUID
- SOPInstanceUID
- SOPClassUID
- PatientID

> 注意：所有 UID 必须介于 1 到 64 个字符之间，并且只能包含字母数字字符或以下特殊字符：`.`、`-`。PatientID 根据其 LO VR 类型进行验证。

每个存储的文件必须具有 StudyInstanceUID、SeriesInstanceUID 和 SopInstanceUID 的唯一组合。如果已存在具有相同标识符的文件，将返回警告代码 `45070`。

> DICOM 文件大小限制：默认情况下，DICOM 文件的大小限制为 2GB。

### Store 响应状态代码

| 代码                         | 描述 |
| :--------------------------- | :---------- |
| 200 (OK)                     | 请求中的所有 SOP 实例已存储。 |
| 202 (Accepted)               | 请求中的某些实例已存储，但其他实例失败。 |
| 204 (No Content)             | 存储事务请求中未提供内容。 |
| 400 (Bad Request)            | 请求格式错误。例如，提供的研究实例标识符不符合预期的 UID 格式。 |
| 401 (Unauthorized)           | 客户端未经过身份验证。 |
| 406 (Not Acceptable)         | 不支持指定的 `Accept` 标头。 |
| 409 (Conflict)               | 存储事务请求中的任何实例都未存储。 |
| 415 (Unsupported Media Type) | 不支持提供的 `Content-Type`。 |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

### Store 响应有效负载

响应有效负载将填充具有以下元素的 DICOM 数据集：

| 标签          | 名称                  | 描述 |
| :----------- | :-------------------- | :---------- |
| (0008, 1190) | RetrieveURL           | 如果在存储请求中提供了 StudyInstanceUID 并且至少成功存储了一个实例，则为研究的检索 URL。 |
| (0008, 1198) | FailedSOPSequence     | 存储失败的实例序列。 |
| (0008, 1199) | ReferencedSOPSequence | 已存储实例的序列。 |

`FailedSOPSequence` 中的每个数据集将具有以下元素（如果尝试存储的 DICOM 文件可以读取）：

| 标签          | 名称                     | 描述 |
| :----------- | :----------------------- | :---------- |
| (0008, 1150) | ReferencedSOPClassUID    | 存储失败的实例的 SOP 类唯一标识符。 |
| (0008, 1155) | ReferencedSOPInstanceUID | 存储失败的实例的 SOP 实例唯一标识符。 |
| (0008, 1197) | FailureReason            | 此实例存储失败的原因代码。 |
| (0074, 1048) | FailedAttributesSequence | 包含每个失败属性原因的 `ErrorComment` 序列。 |

`ReferencedSOPSequence` 中的每个数据集将具有以下元素：

| 标签          | 名称                     | 描述 |
| :----------- | :----------------------- | :---------- |
| (0008, 1150) | ReferencedSOPClassUID    | 已存储实例的 SOP 类唯一标识符。 |
| (0008, 1155) | ReferencedSOPInstanceUID | 已存储实例的 SOP 实例唯一标识符。 |
| (0008, 1190) | RetrieveURL              | 此实例在 DICOM 服务器上的检索 URL。 |

使用 `Accept` 标头 `application/dicom+json` 的示例响应：

```json
{
  "00081190":
  {
    "vr":"UR",
    "Value":["http://localhost/studies/d09e8215-e1e1-4c7a-8496-b4f6641ed232"]
  },
  "00081198":
  {
    "vr":"SQ",
    "Value":
    [{
      "00081150":
      {
        "vr":"UI","Value":["cd70f89a-05bc-4dab-b6b8-1f3d2fcafeec"]
      },
      "00081155":
      {
        "vr":"UI",
        "Value":["22c35d16-11ce-43fa-8f86-90ceed6cf4e7"]
      },
      "00081197":
      {
        "vr":"US",
        "Value":[43265]
      }
    }]
  },
  "00081199":
  {
    "vr":"SQ",
    "Value":
    [{
      "00081150":
      {
        "vr":"UI",
        "Value":["d246deb5-18c8-4336-a591-aeb6f8596664"]
      },
      "00081155":
      {
        "vr":"UI",
        "Value":["4a858cbb-a71f-4c01-b9b5-85f88b031365"]
      },
      "00081190":
      {
        "vr":"UR",
        "Value":["http://localhost/studies/d09e8215-e1e1-4c7a-8496-b4f6641ed232/series/8c4915f5-cc54-4e50-aa1f-9b06f6e58485/instances/4a858cbb-a71f-4c01-b9b5-85f88b031365"]
      }
    }]
  }
}
```

### Store 失败原因代码

| 代码  | 描述 |
| :---- | :---------- |
| 272   | 由于处理操作时的一般故障，存储事务未存储实例。 |
| 43264 | DICOM 实例验证失败。 |
| 43265 | 提供的实例 StudyInstanceUID 与存储请求中指定的 StudyInstanceUID 不匹配。 |
| 45070 | 已存储具有相同 StudyInstanceUID、SeriesInstanceUID 和 SopInstanceUID 的 DICOM 实例。如果您希望更新内容，请先删除此实例。 |
| 45071 | 另一个进程正在创建 DICOM 实例，或者之前的创建尝试失败且清理过程尚未有机会清理。请在再次尝试创建之前先删除实例。 |
| 45063 | DICOM 实例数据集与 SOP 类不匹配。研究存储事务（第 10.5 节）观察到在存储实例期间数据集不符合 SOP 类的约束。 |

### Store 错误代码

| 代码  | 描述 |
| :---- | :---------- |
| 100   | 提供的实例属性不符合验证标准。 |

## Retrieve (WADO-RS)

此检索事务支持通过引用检索存储的研究、序列、实例和帧。

| 方法 | 路径                                                                    | 描述 |
| :----- | :---------------------------------------------------------------------- | :---------- |
| GET    | ../studies/{study}                                                      | 检索研究中的所有实例。 |
| GET    | ../studies/{study}/metadata                                             | 检索研究中所有实例的元数据。 |
| GET    | ../studies/{study}/series/{series}                                      | 检索序列中的所有实例。 |
| GET    | ../studies/{study}/series/{series}/metadata                             | 检索序列中所有实例的元数据。 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}                 | 检索单个实例。 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}/metadata        | 检索单个实例的元数据。 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}/rendered        | 检索渲染为图像格式的实例 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}/frames/{frames} | 从单个实例检索一个或多个帧。要指定多个帧，请用逗号分隔每个要返回的帧，例如 /studies/1/series/2/instance/3/frames/4,5,6 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}/frames/{frame}/rendered | 检索渲染为图像格式的单个帧 |

### 在研究或序列中检索实例

支持以下 `Accept` 标头用于在研究或序列中检索实例：


- `multipart/related; type="application/dicom"; transfer-syntax=*`
- `multipart/related; type="application/dicom";`（未指定传输语法时，默认使用 1.2.840.10008.1.2.1）
- `multipart/related; type="application/dicom"; transfer-syntax=1.2.840.10008.1.2.1`
- `multipart/related; type="application/dicom"; transfer-syntax=1.2.840.10008.1.2.4.90`
- `*/*`（未指定传输语法时，默认使用 `*`，mediaType 默认为 `application/dicom`）

### 检索实例

支持以下 `Accept` 标头用于检索特定实例：

- `application/dicom; transfer-syntax=*`
- `multipart/related; type="application/dicom"; transfer-syntax=*`
- `application/dicom;`（未指定传输语法时，默认使用 `1.2.840.10008.1.2.1`）
- `multipart/related; type="application/dicom"`（未指定传输语法时，默认使用 `1.2.840.10008.1.2.1`）
- `application/dicom; transfer-syntax=1.2.840.10008.1.2.1`
- `multipart/related; type="application/dicom"; transfer-syntax=1.2.840.10008.1.2.1`
- `application/dicom; transfer-syntax=1.2.840.10008.1.2.4.90`
- `multipart/related; type="application/dicom"; transfer-syntax=1.2.840.10008.1.2.4.90`
- `*/*`（未指定传输语法时，默认使用 `*`，mediaType 默认为 `application/dicom`）

### 检索帧

支持以下 `Accept` 标头用于检索帧：
- `multipart/related; type="application/octet-stream"; transfer-syntax=*`
- `multipart/related; type="application/octet-stream";`（未指定传输语法时，默认使用 `1.2.840.10008.1.2.1`）
- `multipart/related; type="application/octet-stream"; transfer-syntax=1.2.840.10008.1.2.1`
- `multipart/related; type="image/jp2";`（未指定传输语法时，默认使用 `1.2.840.10008.1.2.4.90`）
- `multipart/related; type="image/jp2";transfer-syntax=1.2.840.10008.1.2.4.90`
- `application/octet-stream; transfer-syntax=*`（用于单帧检索）
- `*/*`（未指定传输语法时，默认使用 `*`，mediaType 默认为 `application/octet-stream`）

### 检索传输语法

当请求的传输语法与原始文件不同时，原始文件会被转码为请求的传输语法。原始文件需要是以下格式之一才能成功转码，否则转码可能失败：
- 1.2.840.10008.1.2（小端隐式）
- 1.2.840.10008.1.2.1（小端显式）
- 1.2.840.10008.1.2.2（显式 VR 大端）
- 1.2.840.10008.1.2.4.50（JPEG 基线过程 1）
- 1.2.840.10008.1.2.4.57（JPEG 无损）
- 1.2.840.10008.1.2.4.70（JPEG 无损选择值 1）
- 1.2.840.10008.1.2.4.90（JPEG 2000 仅无损）
- 1.2.840.10008.1.2.4.91（JPEG 2000）
- 1.2.840.10008.1.2.5（RLE 无损）

不支持的 `transfer-syntax` 将导致 `406 Not Acceptable`。

### 检索元数据（研究、序列或实例）

支持以下 `Accept` 标头用于检索研究、序列或实例的元数据：

- `application/dicom+json`

检索元数据不会返回具有以下值表示（VR）的属性：

| VR 名称 | 描述            |
| :------ | :--------------------- |
| OB      | 其他字节             |
| OD      | 其他双精度           |
| OF      | 其他浮点数            |
| OL      | 其他长整型             |
| OV      | 其他 64 位超长 |
| OW      | 其他字             |
| UN      | 未知                |

### 检索元数据缓存验证（研究、序列或实例）

支持使用 `ETag` 机制进行缓存验证。在元数据请求的响应中，ETag 作为标头之一返回。此 ETag 可以缓存并在后续对相同元数据的请求中作为 `If-None-Match` 标头添加。如果数据存在，可能出现两种类型的响应：
- 自上次请求以来数据未更改：将发送 HTTP 304（未修改）响应，无响应体。
- 自上次请求以来数据已更改：将发送带有更新 ETag 的 HTTP 200（OK）响应。所需数据也将作为响应体的一部分返回。

### 检索渲染图像（实例或帧）
支持以下 `Accept` 标头用于检索实例或帧的渲染图像：

- `image/jpeg`
- `image/png`

如果未指定 `Accept` 标头，服务将默认渲染 `image/jpeg`。

服务仅支持单帧渲染。如果请求渲染具有多个帧的实例，则默认仅渲染第一帧。

指定要返回的特定帧时，帧索引从 1 开始。

还支持 `quality` 查询参数。可以将 `1-100` 之间的整数值（1 表示最差质量，100 表示最佳质量）作为查询参数的值传递。这仅用于渲染为 `jpeg` 的图像，对于 `png` 渲染请求将被忽略。如果未指定，将默认为 `100`。


### Retrieve 响应状态代码

| 代码                         | 描述 |
| :--------------------------- | :---------- |
| 200 (OK)                     | 已检索所有请求的数据。 |
| 304 (Not Modified)           | 自上次请求以来请求的数据未修改。在这种情况下，响应体中不会添加内容。有关更多信息，请参阅[检索元数据缓存验证（研究、序列或实例）](###Retrieve-Metadata-Cache-Validation-(for-Study,-Series,-or-Instance))。 |
| 400 (Bad Request)            | 请求格式错误。例如，提供的研究实例标识符不符合预期的 UID 格式，或者不支持请求的传输语法编码。 |
| 401 (Unauthorized)           | 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 404 (Not Found)              | 找不到指定的 DICOM 资源，或者对于渲染请求，实例不包含像素数据 |
| 406 (Not Acceptable)         | 不支持指定的 `Accept` 标头，或者对于渲染和转码请求，请求的文件太大  |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

## Search (QIDO-RS)

基于 ID 的 DICOM 对象查询（QIDO）使您能够按属性搜索研究、序列和实例。

| 方法 | 路径                                            | 描述                       |
| :----- | :---------------------------------------------- | :-------------------------------- |
| *搜索研究*                                                                         |
| GET    | ../studies?...                                  | 搜索研究                |
| *搜索序列*                                                                          |
| GET    | ../series?...                                   | 搜索序列                 |
| GET    |../studies/{study}/series?...                    | 在研究内搜索序列      |
| *搜索实例*                                                                       |
| GET    |../instances?...                                 | 搜索实例              |
| GET    |../studies/{study}/instances?...                 | 在研究内搜索实例   |
| GET    |../studies/{study}/series/{series}/instances?... | 在序列内搜索实例  |

支持以下 `Accept` 标头用于搜索：

- `application/dicom+json`

### 支持的搜索参数

每个查询支持以下参数：

| 键              | 支持的值              | 允许计数 | 描述 |
| :--------------- | :---------------------------- | :------------ | :---------- |
| `{attributeID}=` | {value}                       | 0...N         | 在查询中搜索属性/值匹配。 |
| `includefield=`  | `{attributeID}`<br/>`all`   | 0...N         | 响应中要返回的附加属性。支持公共和私有标签。<br/>当提供 `all` 时，请参阅[搜索响应](###Search-Response)以了解每个查询类型将返回哪些属性的更多信息。<br/>如果混合提供 {attributeID} 和 'all'，服务器将默认使用 'all'。 |
| `limit=`         | {value}                       | 0..1          | 限制响应中返回值数量的整数值。<br/>值可以在 1 >= x <= 200 的范围内。默认为 100。 |
| `offset=`        | {value}                       | 0..1          | 跳过 {value} 个结果。<br/>如果提供的偏移量大于搜索查询结果的数量，将返回 204（无内容）响应。 |
| `fuzzymatching=` | `true` \| `false`             | 0..1          | 如果为 true，则对 PatientName 属性应用模糊匹配。它将对 PatientName 值内的任何名称部分进行前缀单词匹配。例如，如果 PatientName 是 "John^Doe"，则 "joh"、"do"、"jo do"、"Doe" 和 "John Doe" 都将匹配。但是 "ohn" 不会匹配。 |

#### 可搜索属性

我们支持在以下属性和搜索类型上进行搜索。

| Attribute Keyword | All Studies | All Series | All Instances | Study's Series | Study's Instances | Study Series' Instances |
| :---------------- | :---: | :----: | :------: | :---: | :----: | :------: |
| StudyInstanceUID | X | X | X |  |  |  |
| PatientName | X | X | X |  |  |  |
| PatientID | X | X | X |  |  |  |
| PatientBirthDate | X | X | X |  |  |  |
| AccessionNumber | X | X | X |  |  |  |
| ReferringPhysicianName | X | X | X |  |  |  |
| StudyDate | X | X | X |  |  |  |
| StudyDescription | X | X | X |  |  |  |
| ModalitiesInStudy | X | X | X |  |  |  |
| SeriesInstanceUID |  | X | X | X | X |  |
| Modality |  | X | X | X | X |  |
| PerformedProcedureStepStartDate |  | X | X | X | X |  |
| ManufacturerModelName | | X | X | X | X |  |
| SOPInstanceUID |  |  | X |  | X | X |

#### 搜索匹配

我们支持以下匹配类型。

| 搜索类型 | 支持的属性 | 示例 |
| :---------- | :------------------ | :------ |
| 范围查询 | StudyDate, PatientBirthDate | {attributeID}={value1}-{value2}。对于日期/时间值，我们支持标签上的包含范围。这将映射到 `attributeID >= {value1} AND attributeID <= {value2}`。如果未指定 {value1}，将匹配 {value2} 之前和包括 {value2} 的所有日期/时间出现。同样，如果未指定 {value2}，将匹配 {value1} 及后续日期/时间的所有出现。但是，必须存在这些值之一。`{attributeID}={value1}-` 和 `{attributeID}=-{value2}` 是有效的，但是 `{attributeID}=-` 是无效的。 |
| 精确匹配 | 所有支持的属性 | {attributeID}={value1} |
| 模糊匹配 | PatientName, ReferringPhysicianName | 匹配以该值开头的名称的任何组件。 |

#### 属性 ID

标签可以通过多种方式编码为查询参数。我们部分实现了 [PS3.18 6.7.1.1.1](http://dicom.nema.org/medical/dicom/2019a/output/chtml/part18/sect_6.7.html#sect_6.7.1.1.1) 中定义的标准。支持以下标签编码：

| 值            | 示例          |
| :--------------- | :--------------- |
| {group}{element} | 0020000D         |
| {dicomKeyword}   | StudyInstanceUID |

搜索实例的示例查询：
`../instances?Modality=CT&00280011=512&includefield=00280010&limit=5&offset=0`

### 搜索响应

响应将是 DICOM 数据集的数组。根据资源，*默认*返回以下属性：

#### Default Study tags

| Tag          | Attribute Name |
| :----------- | :------------- |
| (0008, 0005) | SpecificCharacterSet |
| (0008, 0020) | StudyDate |
| (0008, 0030) | StudyTime |
| (0008, 0050) | AccessionNumber |
| (0008, 0056) | InstanceAvailability |
| (0009, 0090) | ReferringPhysicianName |
| (0008, 0201) | TimezoneOffsetFromUTC |
| (0010, 0010) | PatientName |
| (0010, 0020) | PatientID |
| (0010, 0030) | PatientBirthDate |
| (0010, 0040) | PatientSex |
| (0020, 0010) | StudyID |
| (0020, 000D) | StudyInstanceUID |

#### Default Series tags

| Tag          | Attribute Name |
| :----------- | :------------- |
| (0008, 0005) | SpecificCharacterSet |
| (0008, 0060) | Modality |
| (0008, 0201) | TimezoneOffsetFromUTC |
| (0008, 103E) | SeriesDescription |
| (0020, 000E) | SeriesInstanceUID |
| (0040, 0244) | PerformedProcedureStepStartDate |
| (0040, 0245) | PerformedProcedureStepStartTime |
| (0040, 0275) | RequestAttributesSequence |

#### Default Instance tags

| Tag          | Attribute Name |
| :----------- | :------------- |
| (0008, 0005) | SpecificCharacterSet |
| (0008, 0016) | SOPClassUID |
| (0008, 0018) | SOPInstanceUID |
| (0008, 0056) | InstanceAvailability |
| (0008, 0201) | TimezoneOffsetFromUTC |
| (0020, 0013) | InstanceNumber |
| (0028, 0010) | Rows |
| (0028, 0011) | Columns |
| (0028, 0100) | BitsAllocated |
| (0028, 0008) | NumberOfFrames |

如果 `includefield=all`，以下属性将与默认属性一起包含。与默认属性一起，这是每个资源级别支持的完整属性列表。

#### 附加研究标签

| 标签          | 属性名称 |
| :----------- | :------------- |
| (0008, 1030) | Study Description |
| (0008, 0063) | AnatomicRegionsInStudyCodeSequence |
| (0008, 1032) | ProcedureCodeSequence |
| (0008, 1060) | NameOfPhysiciansReadingStudy |
| (0008, 1080) | AdmittingDiagnosesDescription |
| (0008, 1110) | ReferencedStudySequence |
| (0010, 1010) | PatientAge |
| (0010, 1020) | PatientSize |
| (0010, 1030) | PatientWeight |
| (0010, 2180) | Occupation |
| (0010, 21B0) | AdditionalPatientHistory |

#### 附加序列标签

| 标签          | 属性名称 |
| :----------- | :------------- |
| (0020, 0011) | SeriesNumber |
| (0020, 0060) | Laterality |
| (0008, 0021) | SeriesDate |
| (0008, 0031) | SeriesTime |

返回以下属性：

- 资源 URL 中的所有匹配查询参数和 UID。
- 该资源级别支持的 `IncludeField` 属性。
- 如果目标资源是 `All Series`，则还返回 `Study` 级别属性。
- 如果目标资源是 `All Instances`，则还返回 `Study` 和 `Series` 级别属性。
- 如果目标资源是 `Study's Instances`，则还返回 `Series` 级别属性。
- `NumberOfStudyRelatedInstances` 聚合属性在 `Study` 级别 includeField 中受支持。
- `NumberOfSeriesRelatedInstances` 聚合属性在 `Series` 级别 includeField 中受支持。

### 搜索响应代码

查询 API 将在响应中返回以下状态代码之一：

| 代码                      | 描述 |
| :------------------------ | :---------- |
| 200 (OK)                  | 响应有效负载包含所有匹配的资源。 |
| 204 (No Content)          | 搜索成功完成但未返回结果。 |
| 400 (Bad Request)         | 服务器无法执行查询，因为查询组件无效。响应体包含失败的详细信息。 |
| 401 (Unauthorized)        | 客户端未经过身份验证。 |
| 403 (Forbidden)           | 用户未授权。 |
| 503 (Service Unavailable) | 服务不可用或繁忙。请稍后重试。 |

### 附加说明

- 不支持使用 `TimezoneOffsetFromUTC` (`00080201`) 进行查询。
- 查询 API 不会返回 413（请求实体太大）。如果请求的查询响应限制超出可接受范围，将返回错误请求。在可接受范围内请求的任何内容都将被解析。
- 当目标资源是 Study/Series 时，多个实例之间可能存在不一致的研究/序列级别元数据。例如，两个实例可能具有不同的 patientName。在这种情况下，最新的将获胜，您只能搜索最新数据。
- 分页结果经过优化，首先返回匹配的*最新*实例，如果添加了匹配查询的新数据，这可能导致后续页面中出现重复记录。
- 对于 PN VR 类型，匹配不区分大小写且不区分重音。
- 对于其他字符串 VR 类型，匹配不区分大小写但区分重音。
- 对于错误地具有多个值的单值数据元素，仅索引第一个值。

## Delete

此事务不是官方 DICOMweb&trade; 标准的一部分。它使用 DELETE 方法从存储中删除研究、序列和实例的表示。

| 方法 | 路径                                                    | 描述 |
| :----- | :------------------------------------------------------ | :---------- |
| DELETE | ../studies/{study}                                      | 删除特定研究的所有实例。 |
| DELETE | ../studies/{study}/series/{series}                      | 删除研究中特定序列的所有实例。 |
| DELETE | ../studies/{study}/series/{series}/instances/{instance} | 删除序列中的特定实例。 |

参数 `study`、`series` 和 `instance` 分别对应于 DICOM 属性 StudyInstanceUID、SeriesInstanceUID 和 SopInstanceUID。

对请求的 `Accept` 标头、`Content-Type` 标头或正文内容没有限制。

> 注意：删除事务后，已删除的实例将无法恢复。

### 响应状态代码

| 代码                         | 描述 |
| :--------------------------- | :---------- |
| 204 (No Content)             | 当所有 SOP 实例都已删除时。 |
| 400 (Bad Request)            | 请求格式错误。 |
| 401 (Unauthorized)           | 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 404 (Not Found)              | 当在研究内找不到指定的序列，或在序列内找不到指定的实例时。 |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

### Delete 响应有效负载

响应体将为空。状态代码是返回的唯一有用信息。

# Worklist Service (UPS-RS)

DICOM 服务支持[Worklist Service (UPS-RS)](https://dicom.nema.org/medical/dicom/current/output/html/part18.html#chapter_11) 的 Push 和 Pull SOP。此服务提供对一个包含工作项的列表的访问，每个工作项代表一个统一过程步骤（UPS）。

在整个文档中，URI 模板中的变量 `{workitem}` 代表工作项 UID。

## Create Workitem

此事务使用 POST 方法创建新的工作项。

| 方法 | 路径               | 描述 |
| :----- | :----------------- | :---------- |
| POST   | `../workitems`         | 创建工作项。 |
| POST   | `../workitems?{workitem}` | 使用指定的 UID 创建工作项。 |


如果未在 URI 中指定，有效负载数据集必须在 SOPInstanceUID 属性中包含工作项。

请求中需要 `Accept` 和 `Content-Type` 标头，并且都必须具有值 `application/dicom+json`。

在特定事务的上下文中，有许多与 DICOM 数据属性相关的要求。属性可能被要求存在、要求不存在、要求为空或要求不为空。这些要求可以在[此表](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.2.5-3)中找到。

关于数据集属性的说明：
- **SOP Instance UID：** 尽管上表说 SOP Instance UID 不应存在，但此指导特定于 DIMSE 协议，在 DICOMWeb&trade; 中的处理方式不同。如果不在 URI 中，SOP Instance UID **应该存在于**数据集中。
- **条件要求代码：** 包括 1C 和 2C 在内的所有条件要求代码都被视为可选。

### Create 响应状态代码

| 代码                         | 描述 |
| :--------------------------- | :---------- |
| 201 (Created)                | 目标工作项已成功创建。 |
| 400 (Bad Request)            | 请求有问题。例如，请求有效负载不满足上述要求。 |
| 401 (Unauthorized)           | 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 409 (Conflict)               | 工作项已存在。 |
| 415 (Unsupported Media Type) | 不支持提供的 `Content-Type`。 |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

### Create 响应有效负载

成功响应将没有有效负载。`Location` 和 `Content-Location` 响应标头将包含对已创建工作项的 URI 引用。

失败响应有效负载将包含描述失败的消息。

## Request Cancellation

此事务使用户能够请求取消非拥有的工作项。

有[四种有效的工作项状态](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.1.1-1)：
- `SCHEDULED`
- `IN PROGRESS`
- `CANCELED`
- `COMPLETED`

此事务仅对处于 `SCHEDULED` 状态的工作项成功。任何用户都可以通过设置其事务 UID 并将其状态更改为 `IN PROGRESS` 来声明工作项的所有权。从那时起，用户只能通过提供正确的事务 UID 来修改工作项。虽然 UPS 定义了允许转发取消请求和其他事件的 Watch 和 Event SOP 类，但此 DICOM 服务不实现这些类，因此对处于 `IN PROGRESS` 状态的工作项的取消请求将返回失败。拥有的工作项可以通过[更改工作项状态](#change-workitem-state)事务取消。

| 方法  | 路径                                            | 描述                                      |
| :------ | :---------------------------------------------- | :----------------------------------------------- |
| POST    | ../workitems/{workitem}/cancelrequest           | 请求取消已计划的工作项 |

需要 `Content-Type` 标头，并且必须具有值 `application/dicom+json`。

请求有效负载可以包含[在 DICOM 标准中定义](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.2.2-1)的操作信息。

### Request Cancellation 响应状态代码

| 代码                         | 描述 |
| :--------------------------- | :---------- |
| 202 (Accepted)               | 服务器已接受请求，但目标工作项状态不一定已更改。 |
| 400 (Bad Request)            | 请求语法有问题。 |
| 401 (Unauthorized)           | 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 404 (Not Found)              | 找不到目标工作项。 |
| 409 (Conflict)               | 请求与目标工作项的当前状态不一致。例如，目标工作项处于 SCHEDULED 或 COMPLETED 状态。 |
| 415 (Unsupported Media Type) | 不支持提供的 `Content-Type`。 |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

### Request Cancellation 响应有效负载

成功响应将没有有效负载，失败响应有效负载将包含描述失败的消息。
如果工作项实例已处于取消状态，响应将包括以下 HTTP 警告标头：
`299: The UPS is already in the requested state of CANCELED.`


## Retrieve Workitem

此事务检索工作项。它对应于 UPS DIMSE N-GET 操作。

参考：https://dicom.nema.org/medical/dicom/current/output/html/part18.html#sect_11.5

如果工作项存在于源服务器上，工作项应以可接受的媒体类型返回。返回的工作项不应包含事务 UID (0008,1195) 属性。这对于保持此属性作为访问锁的角色是必要的。

| 方法  | 路径                    | 描述   |
| :------ | :---------------------- | :------------ |
| GET     | ../workitems/{workitem}	| 请求检索工作项	|

需要 `Accept` 标头，并且必须具有值 `application/dicom+json`。

### Retrieve Workitem 响应状态代码

| 代码                         	| 描述 |
| :---------------------------- | :---------- |
| 200 (OK)               		| 工作项实例已成功检索。 |
| 400 (Bad Request)            	| 请求有问题。	|
| 401 (Unauthorized)           	| 客户端未经过身份验证。 |
| 403 (Forbidden)               | 用户未授权。 |
| 404 (Not Found)              	| 找不到目标工作项。 |
| 503 (Service Unavailable)     | 服务不可用或繁忙。请稍后重试。 |

### Retrieve Workitem 响应有效负载

* 成功响应具有包含所选媒体类型中请求的工作项的单部分有效负载。
* 返回的工作项不应包含工作项的事务 UID (0008,1195) 属性，因为该属性应该只有所有者知道。

## Update Workitem

此事务修改现有工作项的属性。它对应于 UPS DIMSE N-SET 操作。

参考：https://dicom.nema.org/medical/dicom/current/output/html/part18.html#sect_11.6

要更新当前处于 SCHEDULED 状态的工作项，不应存在事务 UID 属性。对于处于 IN PROGRESS 状态的工作项，请求必须将当前事务 UID 作为查询参数包含。如果工作项已处于 COMPLETED 或 CANCELED 状态，响应将为 400（错误请求）。

| 方法  | 路径                            | 描述           |
| :------ | :------------------------------ | :-------------------- |
| POST     | ../workitems/{workitem}?{transaction-uid}	| 更新工作项事务	|

需要 `Content-Type` 标头，并且必须具有值 `application/dicom+json`。

请求有效负载包含要应用于目标工作项的更改的数据集。修改序列时，请求必须包括序列中的所有项，而不仅仅是要修改的项。
当需要将多个属性作为一组更新时，请在单个请求中作为多个属性执行此操作，而不是作为多个请求。

在特定事务的上下文中，有许多与 DICOM 数据属性相关的要求。属性可能被要求存在、要求不存在、要求为空或要求不为空。这些要求可以在[此表](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.2.5-3)中找到。

关于数据集属性的说明：
- **条件要求代码：** 包括 1C 和 2C 在内的所有条件要求代码都被视为可选。

请求无法设置过程步骤状态 (0074,1000) 属性的值。过程步骤状态使用更改状态事务或请求取消事务进行管理。

### Update Workitem Transaction 响应状态代码
| 代码                         	| 描述 |
| :---------------------------- | :---------- |
| 200 (OK)               		| 目标工作项已更新。 |
| 400 (Bad Request)            	| 请求有问题。例如：(1) 目标工作项处于 COMPLETED 或 CANCELED 状态。(2) 缺少事务 UID。(3) 事务 UID 不正确。(4) 数据集不符合要求。
| 401 (Unauthorized)           	| 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 404 (Not Found)              	| 找不到目标工作项。 |
| 409 (Conflict)              	| 请求与目标工作项的当前状态不一致。 |
| 415 (Unsupported Media Type) | 不支持提供的 `Content-Type`。 |
| 503 (Service Unavailable)    | 服务不可用或繁忙。请稍后重试。 |

### Update Workitem Transaction 响应有效负载
源服务器应支持[表 11.6.3-2](https://dicom.nema.org/medical/dicom/current/output/html/part18.html#table_11.6.3-2) 中要求的标头字段。

成功响应应没有有效负载，或包含状态报告文档的有效负载。

失败响应有效负载可能包含描述任何失败、警告或其他有用信息的状态报告。

## Change Workitem State

此事务用于更改工作项的状态。它对应于 UPS DIMSE N-ACTION 操作"更改 UPS 状态"。状态更改用于声明所有权、完成或取消工作项。

参考：https://dicom.nema.org/medical/dicom/current/output/html/part18.html#sect_11.7

如果工作项存在于源服务器上，工作项应以可接受的媒体类型返回。返回的工作项不应包含事务 UID (0008,1195) 属性。这对于保持此属性作为访问锁的角色是必要的，如[此处](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#sect_CC.1.1)所述。

| 方法  | 路径                            | 描述           |
| :------ | :------------------------------ | :-------------------- |
| PUT     | ../workitems/{workitem}/state	| 更改工作项状态	|

需要 `Accept` 标头，并且必须具有值 `application/dicom+json`。

请求有效负载应包含更改 UPS 状态数据元素。这些数据元素是：

* **事务 UID (0008,1195)**
请求有效负载应包含事务 UID。用户代理在请求将给定工作项转换到 IN PROGRESS 状态时创建事务 UID。用户代理在该工作项的后续事务中提供该事务 UID。

* **过程步骤状态 (0074,1000)**
合法值对应于请求的状态转换。它们是："IN PROGRESS"、"COMPLETED" 或 "CANCELED"。


### Change Workitem State 响应状态代码

| 代码                         	| 描述 |
| :---------------------------- | :---------- |
| 200 (OK)               		| 工作项实例已成功检索。 |
| 400 (Bad Request)            	| 由于以下原因之一无法执行请求：(1) 考虑到目标工作项的当前状态，请求无效。(2) 缺少事务 UID。(3) 事务 UID 不正确
| 401 (Unauthorized)           	| 客户端未经过身份验证。 |
| 403 (Forbidden)               | 用户未授权。 |
| 404 (Not Found)              	| 找不到目标工作项。 |
| 409 (Conflict)              	| 请求与目标工作项的当前状态不一致。 |
| 503 (Service Unavailable)     | 服务不可用或繁忙。请稍后重试。 |

### Change Workitem State 响应有效负载

* 响应将包括[第 11.7.3.2 节](https://dicom.nema.org/medical/dicom/current/output/html/part18.html#sect_11.7.3.2)中指定的标头字段
* 成功响应应没有有效负载。
* 失败响应有效负载可能包含描述任何失败、警告或其他有用信息的状态报告。

## Search Workitems

此事务使您能够按属性搜索工作项。

| 方法 | 路径                                            | 描述                       |
| :----- | :---------------------------------------------- | :-------------------------------- |
| GET    | ../workitems?                                   | 搜索工作项              |

支持以下 `Accept` 标头用于搜索：

- `application/dicom+json`

### 支持的搜索参数

每个查询支持以下参数：

| 键              | 支持的值              | 允许计数 | 描述 |
| :--------------- | :---------------------------- | :------------ | :---------- |
| `{attributeID}=` | {value}                       | 0...N         | 在查询中搜索属性/值匹配。 |
| `includefield=`  | `{attributeID}`<br/>`all`   | 0...N         | 响应中要返回的附加属性。只能指定要包含的顶级属性 - 不能是序列一部分的属性。支持公共和私有标签。<br/>当提供 `all` 时，请参阅[搜索响应](###Search-Response)以了解每个查询类型将返回哪些属性的更多信息。<br/>如果混合提供 {attributeID} 和 'all'，服务器将默认使用 'all'。 |
| `limit=`         | {value}                       | 0...1          | 限制响应中返回值数量的整数值。<br/>值可以在 1 >= x <= 200 的范围内。默认为 100。 |
| `offset=`        | {value}                       | 0...1          | 跳过 {value} 个结果。<br/>如果提供的偏移量大于搜索查询结果的数量，将返回 204（无内容）响应。 |
| `fuzzymatching=` | `true` \| `false`             | 0...1          | 如果为 true，则对具有人员姓名 (PN) 值表示 (VR) 的任何属性应用模糊匹配。它将对这些属性内的任何名称部分进行前缀单词匹配。例如，如果 PatientName 是 "John^Doe"，则 "joh"、"do"、"jo do"、"Doe" 和 "John Doe" 都将匹配。但是 "ohn" 不会匹配。 |

#### 可搜索属性

我们支持在这些属性上进行搜索：

| 属性关键字 |
| :---------------- |
| PatientName |
| PatientID |
| ReferencedRequestSequence.AccessionNumber |
| ReferencedRequestSequence.RequestedProcedureID |
| ScheduledProcedureStepStartDateTime |
| ScheduledStationNameCodeSequence.CodeValue |
| ScheduledStationClassCodeSequence.CodeValue |
| ScheduledStationGeographicLocationCodeSequence.CodeValue |
| ProcedureStepState |
| StudyInstanceUID |

#### 搜索匹配

我们支持这些匹配类型：

| 搜索类型 | 支持的属性 | 示例 |
| :---------- | :------------------ | :------ |
| 范围查询 | Scheduled​Procedure​Step​Start​Date​Time | {attributeID}={value1}-{value2}。对于日期/时间值，我们支持标签上的包含范围。这将映射到 `attributeID >= {value1} AND attributeID <= {value2}`。如果未指定 {value1}，将匹配 {value2} 之前和包括 {value2} 的所有日期/时间出现。同样，如果未指定 {value2}，将匹配 {value1} 及后续日期/时间的所有出现。但是，必须存在这些值之一。`{attributeID}={value1}-` 和 `{attributeID}=-{value2}` 是有效的，但是 `{attributeID}=-` 是无效的。 |
| 精确匹配 | 所有支持的属性 | {attributeID}={value1} |
| 模糊匹配 | PatientName | 匹配以该值开头的名称的任何组件。 |

> 注意：虽然我们不支持完整的序列匹配，但我们确实支持对上面列出的包含在序列中的属性进行精确匹配。

#### 属性 ID

标签可以通过多种方式编码为查询参数。我们部分实现了 [PS3.18 6.7.1.1.1](http://dicom.nema.org/medical/dicom/2019a/output/chtml/part18/sect_6.7.html#sect_6.7.1.1.1) 中定义的标准。支持以下标签编码：

| 值            | 示例          |
| :--------------- | :--------------- |
| {group}{element} | 00100010         |
| {dicomKeyword}   | PatientName |

示例查询：**../workitems?PatientID=K123&0040A370.00080050=1423JS&includefield=00404005&limit=5&offset=0**

### 搜索响应

响应将是 0...N 个 DICOM 数据集的数组。返回以下属性：

 - [DICOM PS 3.4 表 CC.2.5-3](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.2.5-3) 中返回键类型为 1 或 2 的所有属性。
 - [DICOM PS 3.4 表 CC.2.5-3](https://dicom.nema.org/medical/dicom/current/output/html/part04.html#table_CC.2.5-3) 中返回键类型为 1C 且满足条件要求的所有属性。
 - 作为匹配参数传递的所有其他工作项属性。
 - 作为 includefield 参数值传递的所有其他工作项属性。

### 搜索响应代码

查询 API 将在响应中返回以下状态代码之一：

| 代码                      | 描述 |
| :------------------------ | :---------- |
| 200 (OK)                  | 响应有效负载包含所有匹配的资源。 |
| 206 (Partial Content)     | 响应有效负载仅包含部分搜索结果，其余部分可以通过适当的请求请求。 |
| 204 (No Content)          | 搜索成功完成但未返回结果。 |
| 400 (Bad Request)         | 请求有问题。例如，无效的查询参数语法。响应体包含失败的详细信息。 |
| 401 (Unauthorized)        | 客户端未经过身份验证。 |
| 403 (Forbidden)              | 用户未授权。 |
| 503 (Service Unavailable) | 服务不可用或繁忙。请稍后重试。 |

### 附加说明

- 查询 API 不会返回 413（请求实体太大）。如果请求的查询响应限制超出可接受范围，将返回错误请求。在可接受范围内请求的任何内容都将被解析。
- 分页结果经过优化，首先返回匹配的*最新*实例，如果添加了匹配查询的新数据，这可能导致后续页面中出现重复记录。
- 对于 PN VR 类型，匹配不区分大小写且不区分重音。
- 对于其他字符串 VR 类型，匹配不区分大小写但区分重音。
- 如果同时发生取消工作项和查询同一工作项的情况，则查询很可能会排除正在更新的工作项，响应代码将为 206（部分内容）。
