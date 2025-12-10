# Blob 迁移

> 本文档仅与 Medical Imaging Server for DICOM 开源项目相关。Azure Health Data Services Dicom Services 中的所有数据已经迁移。

目前 DICOM 文件以 DICOM UID 作为 blob 名称存储在 blob 存储中，使用模板 `{account}/{container}/{studyUid}/{seriesUid}/{sopInstanceUid}_{watermark}.dcm`。
如果您保存了任何文件，可以在 blob 容器中看到此命名方案。以下是使用蓝色圆圈示例图像的示例：

![dicomwebcontainer-bluecircle-old-blob-format](../images/dicomwebcontainer-bluecircle-old-blob-format.png)

由于 UID 可能包含有关其创建上下文个人信息，例如患者信息或标识符，我们决定更改存储 DICOM 文件的方式。在以下部分中，我们列出了将现有 blob 从旧格式迁移到新格式的步骤。

## Blob 迁移配置

下面是与 blob 迁移相关的 `appsettings.json` 配置。需要更新几个属性以触发迁移。

```json
"DicomServer": {
    "Services": {
      "BlobMigration": {
        "FormatType": "Old",
        "StartCopy": false,
        "StartDelete": false,
        "CopyFileOperationId": "1d4689da-ca3b-4659-b0c7-7bf6c9ff25e1",
        "DeleteFileOperationId": "ce38a27e-b194-4645-b47a-fe91c38c330f",
        "CleanupDeletedFileOperationId": "d32a0469-9c27-4df3-a1e8-12f7f8fecbc8",
        "CleanupFilterTimeStamp": "2022-08-01"
      }
    }
}
```

如果您使用了 README 中的[部署到 Azure 选项](https://github.com/microsoft/dicom-server#deploy-to-azure)，这些设置可以作为 Azure App Service 配置设置的一部分进行调整：

![app-service-settings-configuration](../images/app-service-settings-configuration.png)


## 迁移步骤

### 如果部署了新服务且尚未创建任何文件
1. [您可以升级到最新版本的服务](../resources/dicom-server-maintaince-guide.md) 并跳过迁移步骤。当服务处于最新版本时，配置部分 `Blob Migration` 将不存在。

### 如果您已经上传了 DICOM 文件但不关心迁移数据
1. 如果您已经上传了 DICOM 文件但不关心迁移数据，可以使用 [Delete API](../resources/conformance-statement.md#delete) 删除所有现有研究。
2. [您可以升级到最新版本的服务](../resources/dicom-server-maintaince-guide.md) 并跳过迁移步骤。当服务处于最新版本时，配置部分 `Blob Migration` 将不存在。

### 如果您已经上传了 DICOM 文件并希望迁移数据
如果您已经上传了 DICOM 文件并希望迁移数据，您需要在升级之前执行以下步骤。此场景有两个选项，取决于您是否希望服务中断。在开始迁移之前，请确保配置了 Azure Monitor 以监控服务（有关如何配置 Azure monitor 的更多信息，请参考 [Azure Monitor 指南](../how-to-guides/configure-dicom-server-settings.md#azure-monitor)）。

#### 服务中断
如果您可以接受服务中断，可以按照以下步骤操作。这里的中断是自我管理的，在复制过程中以及切换到新格式之前使用服务可能会损坏数据路径或检索错误数据。

1. 将 `BlobMigration.StartCopy` 设置为 `true` 并重启服务。
   1. 重启服务时，确保它以能够获取新应用程序设置的方式进行，这将根据服务的部署方式而有所不同。
   2. 这将触发 `CopyFiles` Durable Function，它将旧格式的 DICOM 文件复制到新格式。
   3. **此时不要使用服务**。我们希望确保所有文件都已复制。
2. 为确保复制操作已完成，您可以检查 Azure Monitor 日志中的 `"Completed copying files."` 消息。这将指示操作已完成：

   ![dicomwebcontainer-bluecircle-copy-logs](../images/dicomwebcontainer-bluecircle-copy-logs.png)

    此时，您将同时拥有新旧文件：

    ![dicomwebcontainer-bluecircle-old-blob-format-dual](../images/dicomwebcontainer-bluecircle-old-blob-format-dual.png)

3. 复制完成后，您可以将 `BlobMigration.FormatType` 更改为 `"New"`，将 `BlobMigration.StartDelete` 设置为 `true` 并重启服务。
   1. 这将触发一个 Durable Function，它将删除所有旧格式的 blob，但前提是存在相应的新格式 blob。这是一个安全操作，不会在不检查新格式 blob 是否存在的情况下删除任何 blob。
   2. **开始使用您的服务**存储和检索数据，这将与 `New` 格式一起工作。
4. 为确保删除已完成，您可以检查 Azure Monitor 日志中的 `"Completed deleting files."` 消息。这将指示删除已完成。

#### 无服务中断
如果您不能接受服务中断，可以按照以下步骤操作。

1. 将 `BlobMigration.FormatType` 更改为 `"Dual"`。这将复制任何新上传的 DICOM 文件到旧格式和新格式，因为您在复制操作期间继续使用服务。
2. 按照[服务中断](#服务中断)中的步骤操作，但可以继续使用服务。

> **请在相关的 [GitHub Discussion](https://github.com/microsoft/dicom-server/discussions/1561) 中发布迁移过程中遇到的任何问题或问题。**
