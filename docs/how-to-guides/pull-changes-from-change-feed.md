# 使用 Change Feed 拉取 DICOM 变更

Change Feed 为客户提供了遍历 Medical Imaging Server for DICOM 历史记录并处理服务中创建和删除事件的能力。本操作指南向您展示如何使用 Change Feed。

Change Feed 使用 [Change Feed 概念](/docs/concepts/change-feed.md) 中记录的 REST API 访问，它还提供了 Change Feed 的使用示例。

## 使用 Change Feed

使用提供的 DICOM 客户端包的示例 C# 代码：

```csharp
public async Task<IReadOnlyList<ChangeFeedEntry>> RetrieveChangeFeedAsync(long offset, CancellationToken cancellationToken)
{
    var _dicomWebClient = new DicomWebClient(
                    new HttpClient { BaseAddress = dicomWebConfiguration.Endpoint/v<version> },
                    sp.GetRequiredService<RecyclableMemoryStreamManager>(),
                    tokenUri: null);
    DicomWebResponse<IReadOnlyList<ChangeFeedEntry>> result = await _dicomWebClient.GetChangeFeed(
    $"?offset={offset}&limit={DefaultLimit}&includeMetadata={true}",
    cancellationToken);

    if (result?.Value != null)
    {
            return result.Value;
    }

    return Array.Empty<ChangeFeedEntry>();
}
```

您可以在这里找到完整的代码：[使用 Change Feed](../../converter/dicom-cast/src/Microsoft.Health.DicomCast.Core/Features/DicomWeb/Service/ChangeFeedRetrieveService.cs)

## 总结

本操作指南演示了如何使用 Change Feed。Change Feed 允许您监控 Medical Imaging Server for DICOM 的历史记录。要了解有关 Change Feed 的更多信息，请参考 [Change Feed 概念](../concepts/change-feed.md)。

### 后续步骤

DICOM Cast 通过 Change Feed 轮询任何变更，这允许将 Medical Imaging Server for DICOM 的数据同步到 Azure API for FHIR 服务器。要了解有关 DICOM Cast 的更多信息，请参考 [DICOM Cast 概念](../concepts/dicom-cast.md)。
