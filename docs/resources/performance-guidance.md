# Medical Imaging Server for DICOM 性能指南

当您部署 Medical Imaging Server for DICOM 实例时，以下资源对于生产工作负载的性能很重要：

- **App Service Plan**：托管 Medical Imaging Service for DICOM。
- **Azure SQL**：索引 Medical Imaging Server for DICOM 元数据的子集，以支持查询并维护可查询的变更日志。
- **存储帐户**：Blob 存储，持久化所有 Medical Imaging Server for DICOM 数据和元数据。

本资源提供有关 Medical Imaging Server for DICOM 的 Azure App Service、SQL 数据库和存储帐户设置的指导。请注意，这些是建议，可能不完全适合您的工作负载需求。

## Azure App Service Plan

S1 层是部署时启用的默认 App Service Plan SKU。您可以通过 Medical Imaging Server for DICOM [快速入门部署到 Azure](../quickstarts/deploy-via-azure.md) 在部署期间自定义 App Service Plan SKU。您也可以在部署后更新 App Service Plan。您可以在[配置 Medical Imaging Server for DICOM 设置](../how-to-guides/configure-dicom-server-settings.md) 找到相关说明。

Azure 提供多种计划以满足您的工作负载需求。要了解有关各种计划的更多信息，请查看 [App Service 定价](https://azure.microsoft.com/pricing/details/app-service/windows/)。要了解如何更新 Azure App Service Plan，请参考[配置 Medical Imaging Server for DICOM 设置](../how-to-guides/configure-dicom-server-settings.md)。

## Azure SQL 数据库层级

默认情况下，部署时启用基于 DTU 的 SQL 性能层级的标准层。我们建议 Medical Imaging Server for DICOM 使用 DTU 购买模型而不是 vCore 模型。在基于 DTU 的 SQL 购买模型中，通过性能层级为数据库分配一组固定资源：基本、标准和高级。

要查看 Azure 的各种 SQL 数据库层级，请参考 [Azure SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/single/)。要了解如何更新 Azure SQL 数据库层级，请参考[配置 Medical Imaging Server for DICOM 设置](../how-to-guides/configure-dicom-server-settings.md)。

## 地理冗余

对于生产工作负载，我们强烈建议配置 Medical Imaging Server for DICOM 以支持地理冗余。

### 地理冗余 Azure 存储

Azure 存储提供地理冗余存储，以确保即使在区域中断的情况下也能保持高可用性。如果您正在运行生产工作负载，我们强烈建议选择支持地理冗余的 Azure 存储帐户 SKU。Azure 存储为地理冗余复制提供两个选项：地理区域冗余存储 (GZRS) 和地理冗余存储 (GRS)。请参考本文以决定哪个地理冗余 Azure 存储选项适合您：[使用地理冗余设计高可用性应用程序](https://docs.microsoft.com/en-us/azure/storage/common/geo-redundant-design)。

您可以通过 Medical Imaging Server for DICOM [快速入门部署到 Azure](../quickstarts/deploy-via-azure.md) 在部署期间自定义 Azure 存储帐户 SKU。默认情况下，选择 Standard LRS，即标准本地冗余存储。

### SQL 数据库的地理复制

除了配置地理冗余 Azure 存储外，我们建议为 Azure SQL 数据库配置活动地理复制。这允许您在同一或不同数据中心区域的服务器上创建单个数据库的可读辅助数据库。有关如何配置此功能的教程，请参阅[创建和使用活动地理复制 - Azure SQL 数据库](https://docs.microsoft.com/azure/azure-sql/database/active-geo-replication-overview)。

## Azure SQL 数据库和 Azure App Service 的工作负载场景

### 场景 1：测试 DICOM 服务器

如果您正在测试 Medical Imaging Server for DICOM 并且不运行生产工作负载，我们建议使用 S1 Azure App Service 层级以及标准 Azure SQL 数据库 (S1、S2、S3)。对于不需要冗余的小型系统，使用这些层级，您可以在 Azure App Service 和 Azure SQL 数据库上花费低至约 $70/月。

在这些层级下，通过适当混合 STOW、WADO 和 QIDO 请求，您可以期望处理每分钟 2,000 到 20,000 个请求，响应时间低于 1 秒。较大的文件、严重偏向 STOW 的使用模式以及较差的带宽将减少每分钟的请求数。

### 场景 2：医院系统的生产工作负载

对于生产工作负载，我们建议将 Azure SQL 数据库扩展到 S12。对于 Azure App Service，任何标准层级都应该足够。如果您要进入生产环境，还需要确保 Medical Imaging Server for DICOM 支持地理冗余。请参考我们的[地理冗余指南](#地理冗余)。

我们建议使用 S1 标准 Azure App Service 层级以及 S12 的 Azure SQL 层级。在这些层级下，通过适当混合 STOW、WADO 和 QIDO 请求，您可以期望处理每分钟 1,000 到 20,000 个请求，响应时间低于 400 毫秒。较大的文件、严重偏向 STOW 的使用模式以及较差的带宽将减少每分钟的请求数。我们建议使用您的数据和预期用途测试性能。

### 场景 3：批量摄取 DICOM 文件

如果您的工作负载需要批量摄取 DICOM 文件或自动化工具来处理 DICOM 文件，我们建议将 Azure App Service 和 Azure SQL 数据库扩展到高级层级。如果您要进入生产环境，还需要确保 Medical Imaging Server for DICOM 支持地理冗余。请参考我们的[地理冗余指南](#地理冗余)。

为了支持每天大量 DICOM 事务，我们建议使用 P1v2 层级的 Azure App Service 以及 P11 层级的 Azure SQL 数据库。在出色的带宽、相对较小的图像以及适当混合 STOW、WADO 和 QIDO 请求的情况下，您可以期望处理每分钟 40,000 到 100,000 个请求，响应时间低于 200 毫秒。较大的文件、严重偏向 STOW 的使用模式以及较差的带宽将减少每分钟的请求数。我们建议使用您的数据和预期用途测试性能。

### 关于性能结果的注释

为了估算工作负载，执行了自动化规模测试以模拟生产环境。本文档中的指导是建议，可能需要修改以满足您的环境。在配置服务时需要考虑的几个重要注意事项：

- 一旦数据库使用率超过约 60%，您将开始看到性能下降。
- 本文档指导的规模测试是使用 500 KB DICOM 文件执行的。
- 此外，规模测试是在多个并发调用者的情况下运行的。在较少的并发调用者的情况下，您可能有更高的请求/分钟和响应时间。

## 总结

在本资源中，我们回顾了 Azure App Service 层级、Azure SQL 层级和存储帐户设置的建议指导，以便 Medical Imaging Server for DICOM 能够满足您的工作负载需求：

- 要开始使用 Medical Imaging Server for DICOM，请[部署到 Azure](../quickstarts/deploy-via-azure.md)。
- 如果您已经配置了 Medical Imaging Server for DICOM 实例，请[配置 DICOM 服务器设置](../how-to-guides/configure-dicom-server-settings.md)。
