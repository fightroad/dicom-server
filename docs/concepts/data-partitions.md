# 数据分区概述

数据分区是可为 DICOM 服务启用的可选功能。它实现了轻量级数据分区方案，使客户能够在单个 DICOM 服务上存储具有相同标识实例 UID 的同一图像的多个副本。

虽然 UID **应该**在所有上下文中[唯一](http://dicom.nema.org/dicom/2013/output/chtml/part05/chapter_9.html)，但医疗提供者将 DICOM 文件写入便携式存储介质并交给患者，然后患者将文件交给另一个医疗提供者，后者将文件传输到新的 DICOM 存储系统中，这是常见做法。因此，一个 DICOM 文件的多个副本通常存在于孤立的 DICOM 系统中。随着 DICOM 功能迁移到云端，统一以前断开连接的系统，数据分区可以为现有数据存储和工作流提供入口。

## 功能启用

数据分区功能可以通过本地 [appsettings.json](../../src/Microsoft.Health.Dicom.Web/appsettings.json) 文件或特定于主机的选项将配置键 `DicomServer:Features:EnableDataPartitions` 设置为 `true` 来启用。

启用后，该功能会修改 DICOM 服务器的 API 表面，并使任何以前的数据在 `Microsoft.Default` 分区下可访问。

> *如果存在除 `Microsoft.Default` 之外的分区，数据分区功能**无法禁用** - 启动时将抛出 `DataPartitionsFeatureCannotBeDisabledException`。*

## API 变更

以下所有 URI 都假设隐式 DICOM 服务基础 URI。例如，本地运行的 DICOM 服务器的基础 URI 将是 `https://localhost:63838/`。
可以通过为 `partitionName` 集合变量提供值，在 [Postman 集合](../resources/Conformance-as-Postman.postman_collection.json) 中发送示例请求。

### 列出分区

列出所有数据分区。

```http
GET /partitions
```

#### 请求头

| 名称         | 必需  | 类型   | 描述                     |
| ------------ | --------- | ------ | ------------------------------- |
| Content-Type | False     | string | 支持 `application/json` |

#### 响应

| 名称              | 类型                          | 描述                           |
| ----------------- | ----------------------------- | ------------------------------------- |
| 200 (OK)          | [Partition](#partition)`[]`   | 返回分区列表      |
| 204 (No Content)  |                               | 不存在分区                   |
| 400 (Bad Request) |                               | 数据分区功能已禁用   |

### STOW、WADO、QIDO 和 Delete

启用分区后，STOW、WADO、QIDO 和 Delete 请求**必须**在基础 URI 之后包含数据分区 URI 段，格式为 `/partitions/{partitionName}`，其中 `partitionName` 是：
 - 最多 64 个字符长
 - 由字母数字字符、`.`、`-` 和 `_` 的任意组合组成，以允许 DICOM UID 和 GUID 格式以及人类可读的标识符

| 操作  | 示例 URI                                                         |
| ------- | ------------------------------------------------------------------- |
| STOW    | `POST /partitions/myPartition-1/studies`                            |
| WADO    | `GET /partitions/myPartition-1/studies/2.25.0000`                   |
| QIDO    | `GET /partitions/myPartition1/studies?StudyInstanceUID=2.25.0000`   |
| Delete  | `DELETE /partitions/myPartition1/studies/2.25.0000`                 |

#### 新响应

| 名称              | 消息                                                   |
| ----------------- | --------------------------------------------------------- |
| 400 (Bad Request) | 数据分区功能已禁用                       |
| 400 (Bad Request) | 路由段中缺少 PartitionName 值。      |
| 400 (Bad Request) | 指定的 PartitionName {PartitionName} 不存在。   |

### 其他 API

所有其他 API（包括[扩展查询标签](../how-to-guides/extended-query-tags.md)、[操作](../how-to-guides/extended-query-tags.md#get-operation)和[变更源](change-feed.md)）将继续在基础 URI 访问。

## 管理分区

目前，分区支持的唯一管理操作是在 STOW 请求期间的**隐式**创建。
如果 URI 中指定的分区不存在，它将隐式创建，响应将返回包含分区路径的检索 URI。

## 限制
 - 如果存在除 `Microsoft.Default` 之外的分区，则无法禁用该功能
 - 不支持跨分区查询
 - 不支持更新和删除分区

## 定义

### Partition

逻辑隔离和数据唯一性的单位。

| 名称          | 类型   | 描述                                                                      |
| ------------- | ------ | -------------------------------------------------------------------------------- |
| PartitionKey  | int    | 系统分配的标识符                                                       |
| PartitionName | string | 客户端分配的唯一名称，最多 64 个字母数字字符、`.`、`-` 或 `_`  |
| CreatedDate   | string | 创建分区的日期和时间                                 |
