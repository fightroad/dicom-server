# 配置 Medical Imaging Server for DICOM 设置

本操作指南说明如何在部署后配置 Medical Imaging Server for DICOM 的设置。

## 先决条件

要配置 Medical Imaging Server for DICOM，您需要部署一个实例。如果尚未部署 Medical Imaging Server，请[部署一个实例到 Azure](../quickstarts/deploy-via-azure.md)。

## 管理身份验证

要使用 Azure AD 配置 Medical Imaging Server for DICOM 的身份验证，请参阅[使用令牌启用身份验证](../how-to-guides/enable-authentication-with-tokens.md)。

要在不使用 AAD 集成的开发和测试场景中使用 Identity Provider 管理身份验证，请参阅[Identity Server 身份验证](../development/identity-server-authentication.md)。

## 管理 Azure App Service

S1 层是部署时启用的默认 App Service Plan SKU。Azure 提供多种计划以满足您的工作负载需求。要了解有关各种计划的更多信息，请查看 [App Service 定价](https://azure.microsoft.com/pricing/details/app-service/windows/)。

如果您想将 App Service 计划扩展到不同层级：

1. 在 Azure 门户中导航到 Medical Imaging Server for DICOM **App Service**。
2. 从菜单中选择**扩展（App Service 计划）**：
![扩展](../images/scale-up-1.png)
3. 选择适合您工作负载需求的计划：
![扩展 2](../images/scale-up-2.png)
4. 单击**应用**。

自动扩展是一项内置功能，可在需求变化时帮助应用程序发挥最佳性能。您可以选择手动将资源扩展到特定实例计数，或通过基于指标阈值的自定义自动扩展策略，或在指定时间窗口内扩展的计划实例计数。自动扩展通过根据需求添加和删除实例，使您的资源具有高性能和成本效益

除了扩展之外，您还可以扩展 App Service Plan 以满足工作负载的需求。您可以选择手动扩展服务以维护固定实例计数，或基于任何指标自定义自动扩展服务。如果您想扩展 App Service Plan：

1. 在 Azure 门户中导航到 Medical Imaging Server for DICOM **App Service**。
2. 从菜单中选择**横向扩展（App Service 计划）**：
![横向扩展](../images/scale-out-1.png)
3. 选择最适合您需求的横向扩展选项：
![横向扩展 2](../images/scale-out-2.png)
4. 选择**保存**。

有关 Azure App Service 层级的建议指导，请参阅 [Medical Imaging Server for DICOM 性能指南](../resources/performance-guidance.md)。

## 管理 SQL 数据库

默认情况下，部署时启用基于 DTU 的 SQL 性能层级的标准层。在基于 DTU 的 SQL 购买模型中，通过性能层级为数据库分配一组固定资源：基本、标准和高级。要了解有关各种层级的更多信息，请查看 [Azure SQL 数据库定价](https://azure.microsoft.com/pricing/details/sql-database/single/)。

如果您想更新 SQL 数据库层级：

1. 导航到部署 Medical Imaging Server for DICOM 时创建的 **SQL 数据库**。
2. 选择**配置**：
![配置 SQL1](../images/configure-sql-1.png)
3. 选择满足您工作负载需求的性能层级和 DTU 级别：
![配置 SQL2](../images/configure-sql-2.png)
4. 单击**应用**。

有关 SQL 数据库层级的建议指导，请参阅 [Medical Imaging Server for DICOM 性能指南](../resources/performance-guidance.md)。

## 其他配置设置

## Azure Monitor

[Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) 提供多种解决方案来收集、分析和处理遥测数据，包括 Application Insights Log Analytics。

### Application Insights

如果您使用我们的[快速入门部署到 Azure](../quickstarts/deploy-via-azure.md) 部署 Medical Imaging Server for DICOM，Application Insights 默认部署并启用。要查看和自定义 Application Insights：

1. 导航到 Medical Imaging Server for DICOM **Application Insights** 资源。
2. 选择**可用性**、**失败**或**性能**以了解 App Service 的性能。
3. 要将 Application Insights 资源链接到 Medical Imaging Server for DICOM Web 应用：
    1. 导航到 Medical Imaging Server for DICOM **App Service**。
    2. 在**设置**下选择**Application Insights**。选择*启用* Application Insights。选择已部署的现有 Application Insights 资源。
    3. 可选地，您可以启用 Application Insights 功能，如*Profiler*、*Snapshot Debugger* 和 *SQL Commands*。（注意，这些可以稍后打开）。
    4. 单击*应用*。
4. 要了解如何根据您的需求自定义 Application Insights，请参阅 [Application Insights 概述](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)。

如果您在部署期间未启用 Application Insights，可以通过 Azure 门户启用：

1. 导航到 Medical Imaging Server for DICOM **App Service**。
2. 从菜单中选择**Application Insights**：
![App Insights 1](../images/app-insights-1.png)
3. 选择**启用 Application Insights**：
![App Insights 2](../images/app-insights-2.png)
4. 将 App Service 链接到 Application Insights 资源。您可以为 **Application Insights** 资源创建新名称或使用默认名称。选择**应用**。
5. 通过导航到创建的 **Application Insights** 资源来查看和自定义 **Application Insights**。

### 诊断设置和 Log Analytics

要监控 SQL 数据库，创建流式传输到 Log Analytics 的诊断设置：

1. 导航到您的 **SQL 数据库**。
2. 选择**诊断设置**：
![诊断设置 1](../images/diagnostic-settings-1.png)
3. 选择**添加诊断设置**：
![诊断设置 2](../images/diagnostic-settings-2.png)
4. 选择要监控的日志和/或指标诊断设置，以及这些日志和/或指标的目标：
![诊断设置 3](../images/diagnostic-settings-3.png)
5. 选择**保存**。

要了解如何进一步自定义诊断设置，请参阅[诊断设置](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings?WT.mc_id=Portal-Microsoft_Azure_Monitoring)。要了解如何使用 Log Analytics 编写查询，请参阅[日志查询概述](https://docs.microsoft.com/azure/azure-monitor/log-query/log-query-overview)。

## OHIF 查看器

默认情况下，当您将 Medical Imaging Server for DICOM 部署到 Azure 时，OHIF 查看器已启用。要更新此设置：

1. 在 Azure 门户中导航到 Medical Imaging Server for DICOM **App Service**。
2. 从菜单中选择**配置**：
![OHIF 查看器1](../images/ohif-viewer-1.png)
3. 选择 **DicomServer:Features:EnableOhifViewer** 的*编辑*按钮：
![OHIF 查看器2](../images/ohif-viewer-2.png)
4. 将**值**更新为 *False* 并选择**确定**。
5. 单击**保存**以更新设置。

## 总结

本操作指南说明了如何在部署后配置 Medical Imaging Server for DICOM 的设置。一旦 Medical Imaging Server for DICOM 部署并配置完成，您可以[使用 Medical Imaging Server for DICOM API](../tutorials/use-the-medical-imaging-server-apis.md)。
