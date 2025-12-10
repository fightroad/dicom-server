# 测试 DICOM Schema Manager

[主文档](https://github.com/microsoft/dicom-server/blob/main/docs/resources/schema-manager.md)

---

### 环境要求
1. [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)
2. [Visual Studio 2022](https://visualstudio.microsoft.com/downloads)
3. 本地 SQL Server 安装
   - [SQL Server Developer 或 Express](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

---

### 设置
1. 使用 SSMS 或 Azure Data Studio 创建本地数据库（建议命名 DICOM）。
2. [克隆 DICOM 仓库](https://github.com/microsoft/dicom-server.git)。
3. 在 Visual Studio 打开 **Microsoft.Health.Dicom.sln**。

---

### 在 Visual Studio 中测试

1. 将 **Microsoft.Health.Dicom.SchemaManager.Console** 设为启动项目。
2. 右键该项目选择 **Properties**。
3. 左侧选择 **Debug**，点击 **Open debug launch profiles UI**，在命令行参数中粘贴：
   `apply --connection-string "server=(local);Initial Catalog=DICOM;TrustServerCertificate=True;Integrated Security=True" --version 20`
4. 在 [ApplyCommand.cs 第 54 行](https://github.com/microsoft/dicom-server/blob/main/src/Microsoft.Health.Dicom.SchemaManager/ApplyCommand.cs#L54) 设置断点。
5. 按 F5 开始调试。

---

### 在 dicom-server 中调试 healthcare-shared-components

大部分 Schema Manager 逻辑来自 healthcare-shared-components，调试时可能会跳过部分步骤。按以下方式在 VS 中调试外部代码。

注意：当前存在问题，除微软员工账户外无法在 microsofthealthoss.visualstudio.com 完成认证。

1. 取消勾选 [“Enable Just My Code”](https://docs.microsoft.com/en-us/visualstudio/debugger/just-my-code)。
2. 添加 Microsoft OSS 符号服务器：
   - 服务器：`microsofthealthoss.visualstudio.com`
   - [添加符号服务器方法](https://docs.microsoft.com/en-us/visualstudio/debugger/specify-symbol-dot-pdb-and-source-files-in-the-visual-studio-debugger)
   - 可能需要在浏览器打开 `microsofthealthoss.visualstudio.com` 以完成认证。
3. 关闭并重新打开 Visual Studio。

验证步骤：
1. 在 DICOM Schema Manager 调用 healthcare-shared-components 的 ApplySchema 处下断点：
    - [Microsoft.Health.Dicom.SchemaManager/ApplyCommand.cs](https://github.com/microsoft/dicom-server/blob/main/src/Microsoft.Health.Dicom.SchemaManager/ApplyCommand.cs#L54)
2. 按 **F5** 开始调试。
3. 命中断点后按 **F11** 单步进入。
4. 确认进入 healthcare-shared-components 的 ApplySchema：
    - [Microsoft.Health.SqlServer/Features/Schema/Manager/SqlSchemaManager.cs](https://github.com/microsoft/healthcare-shared-components/blob/main/src/Microsoft.Health.SqlServer/Features/Schema/Manager/SqlSchemaManager.cs#L53)
