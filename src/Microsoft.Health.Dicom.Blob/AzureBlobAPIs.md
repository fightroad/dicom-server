# Azure Blob APIs

本文档说明 Azure Blob SDK 中若干下载 API 的区别（参考：https://github.com/Azure/azure-sdk-for-net/issues/22022）。

- `DownloadStreamingAsync` 是 `DownloadAsync` 的替代，除返回类型略有差别，可视作同功能的重命名/别名。
- 新增 `DownloadContentAsync` 主要用于小文件（如 JSON）并返回 `BinaryData`，便于区分下载 API。

`DownloadStreamingAsync` 与 `OpenReadAsync` 的差异：
- 前者提供网络流（保持单个连接）；后者分块拉取并缓冲，会发起多次请求。
- 如果消费端处理很快且网络带宽好，前者可减少往返；如果消费端较慢，后者能在读完并缓冲下一块后归还连接，可能更合适。建议两种方式都做性能测试以确定最佳选择。
