# Medical Imaging Server for DICOM

 [![Build Status](https://microsofthealthoss.visualstudio.com/DicomServer/_apis/build/status/CI-Build-OSS?branchName=main)](https://microsofthealthoss.visualstudio.com/DicomServer/_build/latest?definitionId=34&branchName=main)

## 项目简介

Medical Imaging Server for DICOM 是一个开源的 DICOM 服务器，可轻松部署到 Azure。它基于 DICOMweb&trade; 标准，支持与任何符合 DICOMweb&trade; 标准的系统进行通信，并将 DICOM 元数据注入到 FHIR 服务器中，以创建完整的患者数据视图。

本项目是 DICOMweb&trade; 标准的 .NET Core 实现，与 [FHIR Server for Azure](https://github.com/microsoft/fhir-server) 紧密集成，使医疗专业人员、ISV 和医疗设备供应商能够创建创新的解决方案。

![架构图](docs/images/DICOM-arch.png)

**生产环境建议**：推荐使用 Azure Health Data Service [DICOM 服务](https://docs.microsoft.com/en-us/azure/healthcare-apis/dicom/deploy-dicom-services-in-azure)作为托管服务。如需自行部署，请参考[维护指南](./docs/resources/dicom-server-maintaince-guide.md)。   

## 项目目录结构

### 根目录文件

- **`Microsoft.Health.Dicom.sln`** - Visual Studio 解决方案文件，包含所有项目
- **`global.json`** - 指定 .NET SDK 版本（当前为 8.0.201）
- **`Directory.Build.props`** - MSBuild 全局属性配置，定义编译选项、目标框架等
- **`Directory.Packages.props`** - 集中管理 NuGet 包版本
- **`GitVersion.yml`** - GitVersion 配置文件，用于自动版本管理
- **`nuget.config`** - NuGet 包源配置
- **`renovate.json`** - Renovate 自动依赖更新配置
- **`CredScanSuppressions.json`** - 安全扫描排除配置
- **`GeoPol.xml`** - 地理政治合规性配置

### `/src` - 源代码目录

核心源代码目录，包含所有主要功能模块：

#### 核心层（Core Layer）
- **`Microsoft.Health.Dicom.Core`** - 核心业务逻辑层，实现 DICOMweb&trade; 标准的核心功能
  - 包含 DICOM 数据处理、验证、序列化等核心功能
  - 定义服务接口和业务模型
  - 包含异常处理、扩展方法、消息处理等

#### API 层（REST API Layer）
- **`Microsoft.Health.Dicom.Api`** - RESTful API 实现层
  - 实现 DICOMweb&trade; REST API 端点
  - 包含控制器、中间件、过滤器等
  - 处理请求/响应转换和授权决策

#### 宿主层（Hosting Layer）
- **`Microsoft.Health.Dicom.Web`** - Web 应用程序宿主
  - ASP.NET Core Web 应用，用于开发和测试
  - 包含前端 UI（OHIF 查看器等）
  - 配置文件：`appsettings.json`, `appsettings.Development.json`

#### 持久化层（Persistence Layer）
- **`Microsoft.Health.Dicom.SqlServer`** - SQL Server 数据访问层
  - 实现数据库操作和索引管理
  - 包含 SQL 脚本和数据库迁移逻辑
  - 支持全文搜索和查询优化

- **`Microsoft.Health.Dicom.Blob`** - Azure Blob 存储访问层
  - 实现 DICOM 文件和元数据的 Blob 存储操作
  - 支持上传、下载、删除等操作
  - 包含重试策略和并发控制

- **`Microsoft.Health.Dicom.Azure`** - Azure 服务集成
  - Azure Key Vault 集成
  - Azure 特定配置和服务注册

#### Azure Functions 层
- **`Microsoft.Health.Dicom.Functions`** - Azure Functions 实现
  - 实现后台任务：数据清理、索引、导出、更新等
  - 使用 Durable Functions 处理长时间运行的任务

- **`Microsoft.Health.Dicom.Functions.Abstractions`** - Functions 抽象定义
  - 定义 Functions 的接口和抽象类

- **`Microsoft.Health.Dicom.Functions.Client`** - Functions 客户端
  - 提供调用 Functions 的客户端接口

- **`Microsoft.Health.Dicom.Functions.App`** - Functions 应用宿主
  - Azure Functions 应用的入口点和配置

#### 客户端库
- **`Microsoft.Health.Dicom.Client`** - DICOM 客户端库
  - 提供与 DICOM 服务器交互的客户端 SDK
  - 可用于第三方应用集成

#### 工具和实用程序
- **`Microsoft.Health.Dicom.SchemaManager`** - 数据库架构管理器
  - 管理数据库架构版本和迁移

- **`Microsoft.Health.Dicom.SchemaManager.Console`** - 架构管理器控制台应用
  - 命令行工具，用于执行数据库架构更新

- **`Microsoft.Health.Dicom.WebUtilities`** - Web 实用工具
  - Web 相关的辅助类和扩展方法

#### 测试项目
- **`Microsoft.Health.Dicom.*.UnitTests`** - 各模块的单元测试
- **`Microsoft.Health.Dicom.Tests.Common`** - 测试公共库
  - 包含测试用的 DICOM 文件、测试辅助类等

### `/test` - 集成测试目录

- **`Microsoft.Health.Dicom.Tests.Integration`** - 集成测试
  - 测试各组件之间的集成

- **`Microsoft.Health.Dicom.Web.Tests.E2E`** - 端到端测试
  - 完整的端到端功能测试

### `/converter` - 转换器目录

- **`dicom-cast`** - DICOM Cast 转换器
  - 将 DICOM 元数据同步到 FHIR 服务器的服务
  - 包含核心逻辑、表存储、托管应用等
  - 支持从 DICOM Change Feed 读取变更并转换为 FHIR 资源

### `/docker` - Docker 配置目录

包含各种 Docker Compose 配置文件：

- **`docker-compose.yml`** - 主配置文件，包含 DICOM 服务器、SQL Server、Azurite 等
- **`docker-compose.https.yml`** - HTTPS 配置
- **`docker-compose.https.windows.yml`** - Windows 平台 HTTPS 配置
- **`docker-compose.https.linux.yml`** - Linux 平台 HTTPS 配置
- **`docker-compose.cast.yml`** - 包含 DICOM Cast 的配置
- **`docker-compose.features.yml`** - 功能特性配置
- **`docker-compose.vs.yml`** - Visual Studio 调试配置
- **`docker-compose.ports.azurite.yml`** - Azurite 端口暴露配置
- **`sql/Dockerfile`** - SQL Server 容器镜像定义

### `/build` - 构建和 CI/CD 配置

包含 Azure DevOps 的构建管道配置：

- **`ci/`** - 持续集成配置
  - 部署、发布、测试等 CI 流程

- **`common/`** - 通用构建脚本和模板
  - 包含可重用的构建步骤
  - **`scripts/`** - PowerShell 构建脚本

- **`pr/`** - Pull Request 构建配置
  - PR 触发时的测试流程

### `/docs` - 文档目录

完整的项目文档：

- **`concepts/`** - 概念文档
  - DICOM、Change Feed、DICOM Cast、数据分区等概念说明

- **`development/`** - 开发文档
  - 代码组织、命名规范、异常处理、测试指南等

- **`how-to-guides/`** - 操作指南
  - 配置、认证、授权、数据导出等操作步骤

- **`quickstarts/`** - 快速入门
  - Azure 部署、Docker 部署、DICOM Cast 设置

- **`tutorials/`** - 教程
  - 使用 API、C#/Python/cURL 示例

- **`resources/`** - 资源文档
  - FAQ、性能指南、健康检查 API、符合性声明等

- **`images/`** - 文档图片资源

- **`dcms/`** - 示例 DICOM 文件
  - 用于测试的示例 DICOM 文件

### `/tools` - 工具目录

开发和测试工具：

- **`dicom-web-electron`** - Electron 桌面应用
  - 用于上传和管理 DICOM 文件的图形界面工具
  - 支持文件上传、Change Feed 查看等功能

- **`file-uploader-tool`** - 文件上传工具
  - 命令行工具，用于批量上传 DICOM 文件

- **`file-generator-tool`** - 文件生成工具
  - 用于生成测试 DICOM 文件

- **`uploader-function`** - 上传 Function
  - Azure Function 形式的文件上传工具

- **`Microsoft.Health.Dicom.Tools.sln`** - 工具解决方案文件

### `/samples` - 示例和模板

- **`templates/`** - Azure 部署模板
  - ARM 模板，用于快速部署到 Azure

- **`scripts/`** - PowerShell 脚本
  - 身份验证、部署相关的 PowerShell 脚本

### `/release` - 发布相关

- **`scripts/`** - 发布脚本
  - PowerShell 模块和脚本，用于版本发布和部署

- **`templates/`** - 发布模板
  - 发布相关的 ARM 模板

### `/swagger` - API 文档

OpenAPI/Swagger 规范文件：

- **`v1/`** - API v1 版本规范
- **`v1-prerelease/`** - API v1 预发布版本规范
- **`v2/`** - API v2 版本规范

### `/forks` - Fork 的第三方库

- **`Microsoft.Health.FellowOakDicom`** - Fork 的 fo-dicom 库
  - 包含对原始库的修改和扩展

### `/lang` - 语言支持

- **`IsExternalInit.cs`** - .NET 5+ 的 Init 访问器支持
  - 用于向后兼容的 C# 语言特性支持

## 快速开始

### 部署到 Azure

如有 Azure 订阅，可直接部署到 Azure：

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fsamples%2Ftemplates%2Fdefault-azuredeploy.json" target="_blank"><img src="https://aka.ms/deploytoazurebutton"/></a>

如需将 DICOM 元数据同步到 FHIR 服务器，可部署 **DICOM Cast**：

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Fdicom-server%2Fmain%2Fconverter%2Fdicom-cast%2Fsamples%2Ftemplates%2Fdefault-azuredeploy.json" target="_blank"><img src="https://aka.ms/deploytoazurebutton"/></a>

详细部署说明请参考：[Azure 部署指南](docs/quickstarts/deploy-via-azure.md)

### 本地部署

开发环境可使用 Docker 容器进行本地部署，参考：[开发环境设置](docs/development/setup.md)

**注意**：本地部署使用 Azurite 模拟 Azure Storage API，仅用于开发测试，不适用于生产环境。

## 文档

- **快速入门**：[Azure 部署](docs/quickstarts/deploy-via-azure.md) | [Docker 部署](docs/quickstarts/deploy-via-docker.md) | [DICOM Cast 设置](docs/quickstarts/deploy-dicom-cast.md)
- **教程**：[API 使用](docs/tutorials/use-the-medical-imaging-server-apis.md) | [C# 示例](docs/tutorials/use-dicom-web-standard-apis-with-c%23.md) | [Python 示例](docs/tutorials/use-dicom-web-standard-apis-with-python.md) | [cURL 示例](docs/tutorials/use-dicom-web-standard-apis-with-curl.md)
- **操作指南**：[服务器配置](docs/how-to-guides/configure-dicom-server-settings.md) | [身份验证](docs/how-to-guides/enable-authentication-with-tokens.md) | [授权](docs/how-to-guides/enable-authorization.md) | [数据导出](docs/how-to-guides/export-data.md)
- **开发文档**：[环境设置](docs/development/setup.md) | [代码组织](docs/development/code-organization.md) | [测试指南](docs/development/tests.md)
- **资源**：[FAQ](docs/resources/faq.md) | [符合性声明](docs/resources/conformance-statement.md) | [性能指南](docs/resources/performance-guidance.md)

## 贡献

欢迎贡献代码和建议！提交 PR 前需要签署 [贡献者许可协议 (CLA)](https://cla.microsoft.com)。

贡献方式：
- [提交问题](https://github.com/Microsoft/dicom-server/issues)
- [代码审查](https://github.com/Microsoft/dicom-server/pulls)
- [提交修复](CONTRIBUTING.md)

更多信息请参考 [贡献指南](CONTRIBUTING.md)。

本项目遵循 [Microsoft 开源行为准则](https://opensource.microsoft.com/codeofconduct/)。
