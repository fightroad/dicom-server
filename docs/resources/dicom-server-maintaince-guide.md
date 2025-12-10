# 在生产环境中管理 Medical Imaging Server for DICOM 的部署

以下是如果您想维护自己的部署需要考虑的事项列表。我们建议您对代码有适度的熟悉。

| 领域 | 描述 |
| --- | --- |
| ARM 部署模板 | [示例部署](../quickstarts/deploy-via-azure.md) 部署所有依赖项和配置，但尚未准备好用于生产。请将其视为模板。 |
| Web 应用程序 | 这是我们的托管层。考虑创建您自己的，具有正确的身份验证和授权服务组合。我们拥有的 [Web 应用程序](../../src/Microsoft.Health.Dicom.Web/) 使用开发 Identity 服务器，必须替换为生产用途。 |
| 身份验证和授权 | 示例 Azure 部署将在没有安全性的情况下部署，这意味着服务器对互联网上的每个人都是可访问的。查看我们的身份验证和授权文档并正确设置。|
| 升级 | 这是一个活跃的项目。我们不断修复问题、添加功能并重新架构以使服务更好。这意味着 [SQL 架构](https://github.com/microsoft/fhir-server/blob/main/docs/SchemaMigrationGuide.md) 和二进制文件都必须定期升级。我们的 SQL 架构版本很快就会停止支持。建议每周升级一次，并密切监控提交历史。 |
| 监控 | DICOM 服务器的日志可选地发送到 Application Insights。考虑添加主动监控和警报。 |
| 容量和规模 | 为预期的生产工作负载和性能需求制定计划。Azure SQL 数据库和 Azure App Service 独立扩展。考虑横向和纵向扩展。 |
| 网络安全 | 考虑网络安全，如私有端点和虚拟网络访问依赖服务。这可以为深度防御提供额外的安全功能。 |
| 定价 | 示例部署部署了基本的 App Service 计划和 SQL 数据库。即使不使用这些资源，也会产生基本费用。 |
| 数据冗余 | 确保正确的 SQL 数据库和存储帐户 SKU 以提供所需的数据冗余和可用性。 |
| 灾难恢复 | 考虑多区域故障转移、备份和其他技术以支持关键任务部署。 |
| 隐私 | 确保最新的策略、工具和资源以符合隐私要求。 |
| 合规性 | 应用程序审计日志、安全扫描、数据访问管理、安全开发生命周期、机密管理、访问审查...等等是您必须考虑的一些工具和流程，以便以 HIPPA、HITRUST 和 ISO 认证的方式存储 PHI 数据。 |

<br>

我们建议尝试 [Azure Health Data Services](https://azure.microsoft.com/en-us/services/health-data-services/#overview) 以快速启动生产工作负载。我们的托管服务处理所有上述复杂性，并提供保证的 SLA。了解更多关于我们的托管 DICOM 服务器产品[这里](https://docs.microsoft.com/en-us/azure/healthcare-apis/dicom/dicom-services-overview)。
