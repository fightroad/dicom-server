# Change Feed 概述

Change Feed 提供 Medical Imaging Server for DICOM 中发生的所有变更的日志。Change Feed 提供有序、保证、不可变、只读的这些变更日志。Change Feed 提供了遍历 Medical Imaging Server for DICOM 历史记录并对其中的创建和删除操作进行处理的能力。

客户端应用程序可以随时以任意大小的批次读取这些日志。Change Feed 使您能够构建高效且可扩展的解决方案，以处理 Medical Imaging Server for DICOM 中发生的变更事件。

您可以异步、增量或完整地处理这些变更事件。任意数量的客户端应用程序可以独立、并行地读取 Change Feed，并以自己的节奏进行。

从 API v2 开始，Change Feed 可以查询特定时间窗口。

## API 设计

API 公开了两个 `GET` 端点用于与 Change Feed 交互。使用 Change Feed 的典型流程[如下所示](#example-usage-flow)。

动词 | 路由              | 返回     | 描述
:--- | :----------------- | :---------- | :---
GET  | /changefeed        | Json Array  | [读取 Change Feed](#read-change-feed)
GET  | /changefeed/latest | Json Object | [读取 Change Feed 中的最新条目](#get-latest-change-feed-item)

### 对象模型

字段               | 类型      | 描述
:------------------ | :-------- | :---
Sequence            | long      | 每个变更事件的唯一 ID
StudyInstanceUid    | string    | 研究实例 UID
SeriesInstanceUid   | string    | 序列实例 UID
SopInstanceUid      | string    | SOP 实例 UID
Action              | string    | 执行的操作 - `create` 或 `delete`
Timestamp           | datetime  | 执行操作的日期和时间（UTC）
State               | string    | [元数据的当前状态](#states)
Metadata            | object    | 可选，如果实例存在，则为当前 DICOM 元数据

#### 状态

状态    | 描述
:------- | :---
current  | 此实例是当前版本。
replaced | 此实例已被新版本替换。
deleted  | 此实例已被删除，在服务中不再可用。

## Change Feed

Change Feed 资源是在 DICOM 服务器内发生的事件集合。

### 版本 2

#### 请求
```http
GET /changefeed?startTime={datetime}&endtime={datetime}&offset={int}&limit={int}&includemetadata={bool} HTTP/1.1
Accept: application/json
Content-Type: application/json
```

#### 响应
```json
[
    {
        "Sequence": 1,
        "StudyInstanceUid": "{uid}",
        "SeriesInstanceUid": "{uid}",
        "SopInstanceUid": "{uid}",
        "Action": "create|delete",
        "Timestamp": "2020-03-04T01:03:08.4834Z",
        "State": "current|replaced|deleted",
        "Metadata": {
            // DICOM JSON
        }
    },
    {
        "Sequence": 2,
        "StudyInstanceUid": "{uid}",
        "SeriesInstanceUid": "{uid}",
        "SopInstanceUid": "{uid}",
        "Action": "create|delete",
        "Timestamp": "2020-03-05T07:13:16.4834Z",
        "State": "current|replaced|deleted",
        "Metadata": {
            // DICOM JSON
        }
    },
    // ...
]
```

#### 参数

名称            | 类型     | 描述 | 默认值 | 最小值 | 最大值 |
:-------------- | :------- | :---------- | :------ | :-- | :-- |
offset          | long     | 从结果集开头跳过的事件数 | `0` | `0` | |
limit           | int      | 返回的最大事件数 | `100` | `1` | `200` |
startTime       | DateTime | 变更事件的包含开始时间 | `"0001-01-01T00:00:00Z"` | `"0001-01-01T00:00:00Z"` | `"9999-12-31T23:59:59.9999998Z"`|
endTime         | DateTime |  变更事件的排除结束时间 | `"9999-12-31T23:59:59.9999999Z"` | `"0001-01-01T00:00:00.0000001"` | `"9999-12-31T23:59:59.9999999Z"` |
includeMetadata | bool     | 指示是否包含 DICOM 元数据 | `true` | | |

### 版本 1

#### 请求
```http
GET /changefeed?offset={int}&limit={int}&includemetadata={bool} HTTP/1.1
Accept: application/json
Content-Type: application/json
```

#### 响应
```json
[
    {
        "Sequence": 1,
        "StudyInstanceUid": "{uid}",
        "SeriesInstanceUid": "{uid}",
        "SopInstanceUid": "{uid}",
        "Action": "create|delete|update",
        "Timestamp": "2020-03-04T01:03:08.4834Z",
        "State": "current|replaced|deleted",
        "Metadata": {
            // DICOM JSON
        }
    },
    {
        "Sequence": 2,
        "StudyInstanceUid": "{uid}",
        "SeriesInstanceUid": "{uid}",
        "SopInstanceUid": "{uid}",
        "Action": "create|delete|update",
        "Timestamp": "2020-03-05T07:13:16.4834Z",
        "State": "current|replaced|deleted",
        "Metadata": {
            // DICOM JSON
        }
    },
    // ...
]
```

#### 参数
名称            | 类型     | 描述 | 默认值 | 最小值 | 最大值 |
:-------------- | :------- | :---------- | :------ | :-- | :-- |
offset          | long     | 事件的排除起始序列号 | `0` | `0` | |
limit           | int      | 相对于偏移量的序列号最大值。例如，如果偏移量为 10，限制为 5，则返回的最大序列号将为 15。 | `10` | `1` | `100` |
includeMetadata | bool     | 指示是否包含 DICOM 元数据 | `true` | | |

## Latest Change Feed

Latest Change Feed 资源表示 DICOM 服务器中发生的最新事件。

### 请求
```http
GET /changefeed/latest?includemetadata={bool} HTTP/1.1
Accept: application/json
Content-Type: application/json
```

### 响应
```json
{
    "Sequence": 2,
    "StudyInstanceUid": "{uid}",
    "SeriesInstanceUid": "{uid}",
    "SopInstanceUid": "{uid}",
    "Action": "create|delete|update",
    "Timestamp": "2020-03-05T07:13:16.4834Z",
    "State": "current|replaced|deleted",
    "Metadata": {
        // DICOM JSON
    }
}
```

### 参数

名称            | 类型 | 描述 | 默认值 |
:-------------- | :--- | :---------- | :------ |
includeMetadata | bool | 指示是否包含元数据 | `true` |

## 使用

### DICOMcast

[DICOMcast](/converter/dicom-cast) 是一个有状态处理器，从 Change Feed 拉取 DICOM 变更，转换它们并将它们发布到配置的 Azure API for FHIR 服务作为 [`ImagingStudy` 资源](https://www.hl7.org/fhir/imagingstudy.html)。DICOM Cast 可以从任何点开始处理 DICOM 变更事件，并继续增量地拉取和处理新变更。

### 用户应用程序

以下是希望对 DICOM 服务中的实例进行额外处理的示例应用程序的流程。

#### 版本 2

1. 应用程序定期在某个时间间隔查询 Change Feed
    * 例如，如果每小时查询一次，Change Feed 查询可能如下所示：`/changefeed?startTime=2023-05-10T16:00:00Z&endTime=2023-05-10T17:00:00Z`
    * 如果从头开始，Change Feed 查询可以省略 `startTime` 以读取所有变更，直到但不包括 `endTime`
        * 例如：`/changefeed?endTime=2023-05-10T17:00:00Z`
2. 根据 `limit`（如果提供），如果返回的事件数等于 `limit`（或默认值），应用程序通过更新后续查询的偏移量继续查询变更事件的附加页面
    * 例如，如果 `limit` 是 `100`，并且返回了 100 个事件，那么后续查询将包括 `offset=100` 以获取下一个结果"页面"。以下查询演示了该模式：
        * `/changefeed?offset=0&limit=100&startTime=2023-05-10T16:00:00Z&endTime=2023-05-10T17:00:00Z`
        * `/changefeed?offset=100&limit=100&startTime=2023-05-10T16:00:00Z&endTime=2023-05-10T17:00:00Z`
        * `/changefeed?offset=200&limit=100&startTime=2023-05-10T16:00:00Z&endTime=2023-05-10T17:00:00Z`
    * 如果返回的事件少于 `limit`，则应用程序可以假设时间范围内没有更多结果

#### 版本 1

1. 应用程序确定希望从哪个序列号开始读取变更事件：
    * 要从第一个事件开始，应用程序应使用 `offset=0`
    * 要从最新事件开始，应用程序应使用 `/changefeed/latest` 资源指定 `offset` 参数，其值为最新变更事件的 `Sequence`
2. 在某个定期轮询间隔，应用程序执行以下操作：
    * 从 `/changefeed/latest` 端点获取最新序列号
    * 通过使用当前偏移量查询变更源来获取下一组变更进行处理
        * 例如，如果应用程序当前已处理到序列号 15，并且它只想一次处理最多 5 个事件，那么它应该使用 URL `/changefeed?offset=15&limit=5`
    * 处理 `/changefeed` 资源返回的任何条目
    * 将其当前序列号更新为：
        1. `/changefeed` 资源返回的最大序列号
        2. 如果 `/changefeed` 资源没有返回变更事件，但 `/changefeed/latest` 返回的最新序列号大于用于 `offset` 的当前序列号，则为 `offset` + `limit`

### 其他潜在使用模式

Change Feed 支持非常适合基于已更改对象处理数据的场景。例如，它可以用于：

* 构建连接的应用程序管道，如 ML，对变更事件做出反应或基于创建或删除的实例安排执行。
* 基于对象发生的变更提取业务分析洞察和指标。
* 轮询 Change Feed 以创建推送通知的事件源。

## 总结

在本概念文档中，我们回顾了 Change Feed 的 REST API 设计和潜在使用场景。有关 Change Feed 的操作指南，请参阅[从 Change Feed 拉取变更](../how-to-guides/pull-changes-from-change-feed.md)。
