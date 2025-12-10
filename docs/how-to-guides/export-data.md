# 导出 DICOM 文件

Medical Imaging Server for DICOM 支持将数据批量导出到 [Azure Blob 存储帐户](https://azure.microsoft.com/en-us/services/storage/blobs/)。在开始之前，请确保在 [appsettings.json](../../src/Microsoft.Health.Dicom.Web/appsettings.json) 或环境中通过将值 `DicomServer:Features:EnableExtendedExport` 设置为 `true` 来启用该功能。

导出 API 在 `POST /export` 可用。给定*源*（要导出的数据集）和*目标*（数据将导出到的位置），端点返回对新启动的长时间运行导出操作的引用。此操作的持续时间取决于要导出的数据量。

## 示例

下面的示例请求将以下 DICOM 资源导出到本地 Azure 存储模拟器中名为 `"export"` 的 blob 容器：
- Study Instance UID 为 `1.2.3` 的研究中的所有实例
- Study Instance UID 为 `12.3` 且 Series Instance UID 为 `4.5.678` 的序列中的所有实例
- Study Instance UID 为 `123.456`、Series Instance UID 为 `7.8` 且 SOP Instance UID 为 `9.1011.12` 的实例

```http
POST /export HTTP/1.1
Accept: */*
Content-Type: application/json

{
    "sources": {
        "type": "identifiers",
        "settings": {
            "values": [
                "1.2.3",
                "12.3/4.5.678",
                "123.456/7.8/9.1011.12"
            ]
        }
    },
    "destination": {
        "type": "azureblob",
        "settings": {
            "blobContainerName": "export",
            "connectionString": "UseDevelopmentStorage=true"
        }
    }
}
```

## 请求

请求体由导出源和目标组成。

```json
{
    "source": {
        "type": "identifiers",
        "settings": {
            "setting1": "<value>",
            "setting2": "<value>"
        }
    },
    "destination": {
        "type": "azureblob",
        "settings": {
            "setting3": "<value>"
        }
    }
}
```

### 源设置

唯一的设置是要导出的标识符列表。

| 属性 | 必需 | 默认值 | 描述 |
| :------- | :------- | :------ | :---------- |
| `Values` | 是      |         | 一个或多个 DICOM 研究、序列和/或 SOP 实例标识符的列表，格式为 `"<study instance UID>[/<series instance UID>[/<SOP instance UID>]]"` |

### 目标设置

与 Azure Blob 存储帐户的连接可以使用 `ConnectionString` 和 `BlobContainerName` 或 `BlobContainerUri` 指定。需要这些设置之一。

如果存储帐户需要身份验证，可以在 `ConnectionString` 或 `BlobContainerUri` 中包含 [SAS 令牌](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview)，对 `Blob` 服务中的 `Objects` 具有 `Write` 权限。也可以使用 `UseManagedIdentity` 选项使用托管标识访问存储帐户。DICOM 服务器和函数都必须使用该标识，并且必须为其分配 [`Storage Blob Data Contributor`](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor) 角色。

| 属性             | 必需 | 默认值 | 描述 |
| :------------------- | :------- | :------ | :---------- |
| `BlobContainerName`  | 否       | `""`    | blob 容器的名称。仅在指定 `ConnectionString` 时需要 |
| `BlobContainerUri`   | 否       | `""`    | blob 容器的完整 URI                      |
| `ConnectionString`   | 否       | `""`    | [Azure 存储连接字符串](https://docs.microsoft.com/en-us/azure/storage/common/storage-configure-connection-string)，必须至少包含 blob 存储的信息 |
| `UseManagedIdentity` | 否       | `false` | 指示是否应使用托管标识对 blob 容器进行身份验证的可选标志 |

## 响应

成功启动导出操作后，导出 API 返回 `202` 状态代码。响应体包含对操作的引用，而 `Location` 标头的值是导出操作状态的 URL（与正文中的 `href` 相同）。

在目标容器内，DCM 文件可以使用以下路径格式找到：`<operation id>/results/<study>/<series>/<sop instance>.dcm`

```http
HTTP/1.1 202 Accepted
Content-Type: application/json

{
    "id": "df1ff476b83a4a3eaf11b1eac2e5ac56",
    "href": "<base url>/<version>/operations/df1ff476b83a4a3eaf11b1eac2e5ac56"
}
```

### 操作状态

可以轮询上述 `href` URL 以获取导出操作的当前状态，直到完成。终端状态由 `200` 状态而不是 `202` 表示。

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "operationId": "df1ff476b83a4a3eaf11b1eac2e5ac56",
    "type": "export",
    "createdTime": "2022-09-08T16:40:36.2627618Z",
    "lastUpdatedTime": "2022-09-08T16:41:01.2776644Z",
    "status": "completed",
    "results": {
        "errorHref": "<container uri>/4853cda8c05c44e497d2bc071f8e92c4/errors.log",
        "exported": 1000,
        "skipped": 3
    }
}
```

## 错误

如果在导出 DICOM 文件时出现任何错误（已确定不是客户端的问题），则跳过该文件并记录相应的错误。此错误日志也与 DICOM 文件一起导出，调用者可以查看。错误日志可以在 `<export blob container uri>/<operation ID>/errors.log` 找到。

### 格式

错误日志的每一行都是一个 JSON 对象，具有以下属性。请注意，给定的错误标识符可能在日志中出现多次，因为对日志的每次更新都会*至少处理一次*。

| 属性     | 描述 |
| ------------ | ----------- |
| `Timestamp`  | 发生错误的日期和时间 |
| `Identifier` | DICOM 研究、序列或 SOP 实例的标识符，格式为 `"<study instance UID>[/<series instance UID>[/<SOP instance UID>]]"` |
| `Error`      | 错误消息 |
