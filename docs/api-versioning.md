# DICOM 服务器的 API 版本控制

本指南概述了 DICOM 服务器的 API 版本策略。

DICOM API 的所有版本都将始终符合 DICOMweb™ 标准规范，但版本可能根据我们的[符合性声明](https://github.com/microsoft/dicom-server/blob/main/docs/resources/conformance-statement.md)公开不同的 API。

## 在请求中指定 REST API 版本

REST API 的版本必须在请求 URL 中明确指定，如下例所示：

`https://<service_url>/v<version>/studies`

**注意：**不再支持没有版本的路由。

### 支持的版本

当前支持的版本是：
- v1.0-prerelease
- v1

支持版本的 OpenApi 文档可以在以下 URL 找到：`https://<service_url>/{version}/api.yaml`

### 预发布版本

带有 "prerelease" 标签的 API 版本表示该版本尚未准备好用于生产，应仅在测试环境中使用。这些端点可能会在不通知的情况下经历破坏性变更。

### 版本如何递增

我们目前只在存在不被视为向后兼容的破坏性变更时递增主版本。

破坏性变更的一些示例（递增主版本）：
1. 重命名或删除端点
2. 删除参数或添加必需参数
3. 更改状态代码
4. 删除响应中的属性或更改响应类型（但可以向响应添加属性）
5. 更改属性的类型
6. API 的行为发生变化（业务逻辑的变化，过去做 foo，现在做 bar）

非破坏性变更（不递增版本）：
1. 添加可为空或具有默认值的属性
2. 向响应模型添加属性
3. 更改属性的顺序

### 响应中的标头

`ReportApiVersions` 已开启，这意味着我们将在适当时返回 `api-supported-versions` 和 `api-deprecated-versions` 标头。

- `api-supported-versions` 将列出请求的 API 支持哪些版本。仅在调用使用 `[ApiVersion("<someVersion>")]` 注释的端点时返回。

- `api-deprecated-versions` 将列出请求的 API 已弃用哪些版本。仅在调用使用 `[ApiVersion("<someVersion>", Deprecated = true)]` 注释的端点时返回。

示例：

```
[ApiVersion("1")]
[ApiVersion("1.0-prerelease", Deprecated = true)]
```

![响应标头](images/api-headers-example.PNG)

### API 文档 / Swagger 更新
请确保对 swagger 文件进行适当的更新，并在必要时添加新版本检查。有关在何处以及如何执行此操作的信息在[这里](./resources/swagger.md)。
