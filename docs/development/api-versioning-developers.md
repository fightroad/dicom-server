# DICOM 服务器 API 版本控制 - 开发人员指南

本指南概述了 DICOM 服务器 REST 端点的 API 版本控制。

## 路由

API 版本号在路由中设置。示例：
`/v1/studies`

要添加路由，使用 `[VersionedRoute]` 属性自动将版本号添加到路由。示例：
```C#
[HttpPost]
[VersionedRoute("studies")]
public async Task<IActionResult> PostAsync(string studyInstanceUid = null)
```

## 递增版本

我们只会递增 API 的主版本，而省略次版本。例如：1、2、3 等。

我们的预发布版本包含次版本：`1.0-prerelease`，但我们已经不再使用该语法。

### 破坏性变更
如果引入了破坏性变更，必须递增主版本。

我们认为属于破坏性变更的事项列表
1. 重命名或删除端点
2. 删除参数或添加必需参数
3. 更改状态代码
4. 删除响应中的属性或更改响应类型（但可以向响应添加属性）
5. 更改属性的类型
6. API 的行为发生变化（业务逻辑的变化，过去做 foo，现在做 bar）

有关破坏性变更的更多信息，请参阅 [REST 指南](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md#123-definition-of-a-breaking-change)

附加更改不被视为破坏性变更。例如，添加响应字段或添加新路由。

错误修复不被视为破坏性变更。

### 预发布版本

如果您有仍可能更改或尚未准备好用于生产的破坏性变更要添加，添加状态为 "prerelease" 的版本是一个好主意。
预发布版本可能会经历破坏性变更，不建议客户在生产环境中使用。

`[ApiVersion("x.0-prerelease")]`

或

`ApiVersion prereleaseVersion = new ApiVersion(x, 0, "prerelease");`

### 测试破坏性变更

目前，我们在 pr 和 ci 管道中有一个测试，用于检查确保任何定义的 API 版本都没有任何破坏性变更（不向后兼容的更改）。我们使用 [OpenAPI-diff](https://github.com/OpenAPITools/openapi-diff) 将每个版本的基线 OpenApi 文档与管道构建步骤后生成的版本进行比较。如果在签入存储库的基线与管道中生成的 OpenApi 文档之间检测到破坏性变更，则管道失败。

### 如何递增版本

1. 添加一个新控制器来保存新版本的端点，并使用 `[ApiVersion("<desiredVersion>")]` 进行注释。所有现有端点都必须获得新版本。
2. 将新版本号添加到 `test/Microsoft.Health.Dicom.Api.UnitTests/Features/Routing/UrlResolverTests.cs` 以测试新端点。
3. 测试以验证破坏性变更未添加到先前版本。
4. 执行以下操作以在 pr 和 ci 管道中添加检查，以验证开发人员不会意外创建破坏性变更。
    1. 将新版本添加到 `build/common/versioning.yml` 中的参数。PowerShell 脚本接受版本数组，因此新版本可以添加到参数中。
    2. 为新版本生成 yaml 文件并保存到 `/dicom-server/swagger/{Version}/swagger.yaml`。这将允许我们将其用作新的基线，在 pr 和 ci 管道中进行比较，以确保不会意外引入破坏性变更。每个新版本只需要执行一次此步骤，但是，如果版本仍在开发中，则可以多次更新。
5. 更新 Electron 工具中的 index.html 文件 `tools\dicom-web-electron\index.html` 以允许用户选择新版本。

## 弃用

我们可以通过将版本标记为已弃用来弃用旧版本，如下所示：
```c#
[ApiVersion("2")]
[ApiVersion("1", Deprecated = true)]
```

待定：何时弃用以及何时停用旧版本

## 向客户传达变更
待定：是否需要开发人员记录其变更以向客户传达的过程
