# 代码组织

## 项目
代码库设计为支持不同的数据存储、身份提供者、操作系统，并且不绑定到任何特定的云或托管环境。为了实现这些目标，项目被分解为以下层：

| 层              | 示例                                                      | 注释                                                                              |
| ------------------ | ------------------------------------------------------------ |---------------------------------------------------------------------------------------|
| 宿主层      | `Microsoft.Health.Dicom.Web`                                 | 支持在不同环境中托管，具有自定义的 IoC 容器配置。仅用于开发目的。 |
| REST API 层     | `Microsoft.Health.Dicom.Api`                                 | 实现 RESTful DICOMweb&trade; |
| 核心逻辑层   | `Microsoft.Health.Dicom.Core`                                | 实现支持 DICOMweb&trade; 的核心逻辑 |
| 持久化层  | `Microsoft.Health.Dicom.Sql` `Microsoft.Health.Dicom.Blob`   | 可插拔的持久化提供程序 |

## 模式

DICOM 服务器代码遵循以下**模式**来组织这些层中的代码。

### [MediatR Handler](https://github.com/jbogard/MediatR)：

<em>用于从 Controller 方法分派消息。用于将请求和响应从宿主层转换到服务。</em>
- 职责：授权决策、消息反序列化
- 命名指南：`Resource`Handler
-  示例：[DeleteHandler](/src/Microsoft.Health.Dicom.Core/Features/Delete/DeleteHandler.cs)

### [Resource Service](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0)： 
<em>用于实现业务逻辑。如输入验证（内联或调用）、编排或核心响应对象。</em>
- 职责：实现服务类提供者职责，包括编排持久化提供程序。
- 将服务范围限定为资源操作。
- 命名指南：`Resource`Service
-  示例：[IQueryService](/src/Microsoft.Health.Dicom.Core/Features/Query/IQueryService.cs)

### [Store Service](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0)：
<em>存储/检索/删除数据的特定于数据存储的实现。</em>
- 职责：为单个持久化存储提供抽象。
- 接口在核心中定义，实现在特定的持久化层中。
- 它们不应在服务外部访问。
- 命名指南：`Resource`Store
- 示例：[SqlIndexDataStore](/src/Microsoft.Health.Dicom.SqlServer/Features/Store/SqlIndexDataStore.cs)

### [Middleware](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-6.0)：
 <em>标准/常见关注点，如身份验证、路由、日志记录、异常处理，这些需要对每个请求执行，被分离到它们自己的组件中。</em>

- 命名指南：`Responsibility`Middleware。
- 示例：[ExceptionHandlingMiddleware](/src/Microsoft.Health.Dicom.Api/Features/Exceptions/ExceptionHandlingMiddleware.cs)。

### [Action Filters](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-6.0)：
<em>DICOM 代码使用预操作过滤器。用于身份验证的授权过滤器和用于可接受内容类型验证的自定义过滤器。</em>

- 命名指南：`Responsibility`FilterAttribute。
- 示例：[AcceptContentFilterAttribute](/src/Microsoft.Health.Dicom.Api/Features/Filters/AcceptContentFilterAttribute.cs)。
