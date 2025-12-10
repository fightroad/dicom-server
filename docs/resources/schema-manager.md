# DICOM Schema Manager

### 它是什么？
Schema Manager 是一个命令行应用程序，通过迁移脚本将数据库中的架构从一个版本升级到下一个版本。

------------

### 如何使用它？
DICOM Schema Manager 目前有一个命令（**apply**），具有以下选项：

| 选项 | 描述 |
| ------------ | ------------ |
| `-cs, --connection-string` | 要应用架构更新的 SQL 服务器的连接字符串。（必需） |
| `-mici, --managed-identity-client-id` | 要使用的托管标识的客户端 ID。 |
| `-at, --authentication-type` | 要使用的身份验证类型。有效值为 `ManagedIdentity` 和 `ConnectionString`。 |
| `-v, --version` | 从当前数据库版本到指定版本应用所有可用版本。 |
| `-n, --next` | 应用下一个可用数据库版本。 |
| `-l, --latest` | 从当前数据库版本到最新版本应用所有可用版本。 |
| `-f, --force` | 在不验证指定版本的情况下运行架构迁移。 |
| `-?, -h, --help` | 显示帮助和使用信息。 |

您可以通过运行以下命令查看最新的选项：
`.\Microsoft.Health.Dicom.SchemaManager.Console.exe apply -?`

示例命令行用法：
`.\Microsoft.Health.Dicom.SchemaManager.Console.exe apply --connection-string "server=(local);Initial Catalog=DICOM;TrustServerCertificate=True;Integrated Security=True" --version 20`

`.\Microsoft.Health.Dicom.SchemaManager.Console.exe apply -cs "server=(local);Initial Catalog=DICOM;TrustServerCertificate=True;Integrated Security=True" --latest`

------------

### 重要数据库表

**SchemaVersion**
- 此表保存已应用于数据库的所有架构版本。

**InstanceSchema**
- 每个 DICOM 实例将其架构版本以及它兼容的版本报告给 InstanceSchema 数据库表。

------------

### 术语

**当前数据库版本**
- 数据库中的最大 SchemaVersion 版本。

**当前实例版本**
- 数据库中等于或小于 SchemaVersionConstants.Max 值的最大 SchemaVersion 版本。例如，如果当前数据库版本是 25，但 SchemaVersionConstants.Max 是 23，则实例的当前版本将是 23。

**可用版本**
- 任何大于当前数据库版本的版本。

**兼容版本**
- 从 SchemaVersionConstants.Min 到 SchemaVersionConstants.Max（含）的任何版本。

------------

### 它是如何工作的？

Schema Manager 执行以下步骤：
1. 验证所有参数都已提供且有效。
2. 调用 [healthcare-shared-components ApplySchema 函数](https://github.com/microsoft/healthcare-shared-components/blob/main/src/Microsoft.Health.SqlServer/Features/Schema/Manager/SqlSchemaManager.cs#L53)，该函数：
	1. 确保基础架构存在。
	2. 确保实例架构记录存在。
		1. 由于 DICOM 服务器实现了自己的 ISchemaClient (DicomSchemaClient)，如果没有实例架构记录，升级将继续进行而不会中断。在 healthcare-shared-components 中，这将抛出异常并取消升级。
	3. 获取所有可用版本并将它们与所有兼容版本进行比较。
	4. 基于当前数据库架构版本：
		1. 如果没有版本（仅基础架构），则应用最新的完整迁移脚本。
		2. 如果当前版本 >= 1，则一次应用一个可用版本，直到数据库的架构版本达到用户输入的所需版本（最新、下一个或特定版本）。

------------

### 注意事项

Schema Manager 在假设它将与任何 DICOM 二进制文件同时更新的情况下工作。当使用与 DICOM 二进制文件不同的标记版本运行 Schema Manager 时，可能会使数据库处于不良状态。例如，您可能有一个升级到架构版本 25 的数据库，但二进制文件仅支持到架构版本 23。

Schema Manager 被编程为升级现有、正在运行的 DICOM 实例的数据库，或针对新数据库。如果 SchemaManager 针对没有运行实例的现有数据库运行，SchemaManager 将应用最新的 SchemaVersion，而不考虑运行实例的兼容性。这是因为 InstanceSchema 表仅在 DICOM 服务运行时填充。

------------

### SQL 脚本位置

- [基础架构脚本](https://github.com/microsoft/healthcare-shared-components/blob/main/src/Microsoft.Health.SqlServer/Features/Schema/Migrations/BaseSchema.sql)

- [DICOM 迁移脚本](https://github.com/microsoft/dicom-server/tree/main/src/Microsoft.Health.Dicom.SqlServer/Features/Schema/Migrations)
