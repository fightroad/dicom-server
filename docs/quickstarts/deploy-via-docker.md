# 使用 Docker 本地部署 Medical Imaging Server for DICOM

本快速入门指南详细介绍了如何在 Docker 中构建和运行 Medical Imaging Server for DICOM。通过使用 Docker Compose，所有必要的依赖项都会在容器中自动启动，无需在开发机器上安装任何内容。特别是，Docker 中的 Medical Imaging Server for DICOM 会为 [SQL Server](https://docs.microsoft.com/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash) 和名为 [Azurite](https://github.com/Azure/Azurite) 的 Azure 存储模拟器启动容器。

> **重要提示**
>
> 此示例已创建用于启用开发/测试场景，不适合生产场景。密码包含在部署文件中，SQL 服务器连接未加密，Medical Imaging Server for DICOM 上的身份验证已禁用，数据在容器重启之间不会持久化。

## Visual Studio（仅 DICOM 服务器）

您可以直接从 Visual Studio 轻松运行和调试 Medical Imaging Server for DICOM。只需在 Visual Studio 2019（或更高版本）中打开解决方案文件 *Microsoft.Health.Dicom.sln* 并运行 "docker-compose" 项目。这应该构建每个镜像并在本地运行容器，无需任何额外操作。

准备就绪后，应该会自动打开 URL `https://localhost:8080` 的网页，您可以在其中与 Medical Imaging Server for DICOM 通信。

## 命令行

从 `microsoft/dicom-server` 存储库的根目录运行以下命令，将 `<SA_PASSWORD>` 替换为您选择的密码（请确保遵循 [SQL Server 密码复杂性要求](https://docs.microsoft.com/sql/relational-databases/security/password-policy?view=sql-server-ver15#password-complexity)）：

```bash
docker-compose -p healthcare -f docker/docker-compose.yml up --build -d
```

如果您希望指定自己的 SQL 管理员密码，也可以包含一个：

```bash
env SAPASSWORD='<SA_PASSWORD>' docker-compose -p healthcare -f docker/docker-compose.yml up --build -d
```

部署后，Medical Imaging Server for DICOM 应该在 `http://localhost:8080/` 可用。

### 包含 DICOMcast

如果您还想包含 DICOMcast，只需在 `docker-compose up` 命令中添加一个文件：

```bash
docker-compose -p healthcare -f docker/docker-compose.yml -f docker/docker-compose.cast.yml up --build -d
```

### 使用自定义配置在 Docker 中运行

要构建 `dicom-server` 镜像，请从 `microsoft/dicom-server` 存储库的根目录运行以下命令：

```bash
docker build -f src/microsoft.health.dicom.web/Dockerfile -t dicom-server .
```

运行容器时，还可以指定其他配置详细信息，例如：

```bash
docker run -d \
    -e DicomServer__Security__Enabled="false" \
    -e SqlServer__ConnectionString="Server=tcp:<sql-server-fqdn>,1433;Initial Catalog=Dicom;Persist Security Info=False;User ID=sa;Password=<sql-sa-password>;MultipleActiveResultSets=False;Connection Timeout=30;TrustServerCertificate=true" \
    -e SqlServer__AllowDatabaseCreation="true" \
    -e SqlServer__Initialize="true" \
    -e BlobStore__ConnectionString="<blob-connection-string>" \
    -p 8080:8080 \
    dicom-server
```

## 连接到依赖项

默认情况下，`azurite` 和 `sql` 等存储服务不会在本地公开，但您可以通过取消注释 `docker-compose.yml` 文件中的 `ports` 元素直接连接到它们。确保这些端口在本地未被使用！在不更改值的情况下，使用以下端口：
* SQL Server 在端口 `1433` 上公开 TCP 连接
  * 在 SQL 连接字符串中，使用 `localhost:1433` 或 `tcp:(local)`
* Azurite（Azure 存储模拟器）在端口 `10000` 上公开 blob 服务，在端口 `10001` 上公开队列服务，在端口 `10002` 上公开表服务
  * 模拟器使用明确定义的[连接字符串](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator#connect-to-the-emulator-account-using-the-well-known-account-name-and-key)
  * 使用 [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) 浏览其内容
* [FHIR](https://github.com/microsoft/fhir-server) 可以通过 `http://localhost:8081` 访问

您也可以通过它们的 IP 地址而不是通过 localhost 连接到它们。以下命令将帮助您了解服务公开的 IP 和端口：

```bash
docker inspect -f 'Name: {{.Name}} - IPs: {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}} - Ports: {{.Config.ExposedPorts}}' $(docker ps -aq)
```

## 后续步骤

部署完成后，您可以在 `https://localhost:8080` 访问 Medical Imaging Server。发出请求时，请确保在 URL 中指定版本。更多信息可以在 [API 版本控制文档](../api-versioning.md) 中找到

* [使用 Medical Imaging Server for DICOM API](../tutorials/use-the-medical-imaging-server-apis.md)
* [通过 Electron 工具上传 DICOM 文件](../../tools/dicom-web-electron)
* [启用 Azure AD 身份验证](../how-to-guides/enable-authentication-with-tokens.md)
* [启用 Identity Server 身份验证](../development/identity-server-authentication.md)
