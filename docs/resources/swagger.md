# Swagger

我们使用 [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) 生成 Swagger/API 文档。
如果您以前从未使用过 Swagger，查看[此示例](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/6.x/SwashbuckleSample) 并尝试使用它可能会有所帮助。

工作流程的快速摘要：

- 对 API 或我们为 swagger 拥有的某些自定义或默认值进行更改
- 构建解决方案。生成的 .dll 然后用于生成 API 文档。
- 使用 `dotnet swagger` 从新的 .dll 生成文档

DICOM OSS 项目设置为在构建时自动为您生成此文档。

## 自定义和默认值

Swagger 和自定义设置在[这里](https://github.com/microsoft/dicom-server/blob/main/src/Microsoft.Health.Dicom.Api/Registration/DicomServerServiceCollectionExtensions.cs#L133)设置。

我们已经编写了一些自定义并添加了默认值。

默认值可以在 src/Microsoft.Health.Dicom.Api/Configs/SwaggerConfiguration.cs 中看到。
自定义可以在 src/Microsoft.Health.Dicom.Api/Features/Swagger 中看到。

请注意，我们还在 src/Microsoft.Health.Dicom.Web/appsettings.json 中指定了许可。如果您在 SwaggerConfiguration.cs 中删除许可默认值，appsettings.json 将在使用构建后钩子时用于获取许可。
但是，如果在构建之外的终端中使用 `dotnet swagger`，则不会使用这些设置。如果有人知道原因，请替换此内容。

## 更新 Swagger YAML

Swagger yaml 将在每次构建时使用 Microsoft.Health.Dicom.Web.csproj 中名为 `SwaggerPostBuildTarget` 的构建后钩子为您生成。

### 添加新版本

您可以通过在 `Microsoft.Health.Dicom.Web.csproj` 中添加新的 `Exec MSBuild 任务`来添加新版本。
输出应该转到 `swagger\<your-new-version>\swagger.yaml`，并在 `dotnet swagger tofile` 命令的末尾使用 `<your-new-version>`，指定`您要检索的 swagger 文档的名称，如启动类中配置的那样`。

示例：

```
<Exec Command="dotnet swagger tofile  --yaml --output ..\..\swagger\v1\swagger.yaml $(OutputPath)\Microsoft.Health.Dicom.Web.dll v1"></Exec>
```

请确保还更新 build/common/versioning.yml PowerShell 任务以检查新版本。

### ADO 检查

我们利用 [openapi-diff](https://github.com/OpenAPITools/openapi-diff) 来检查差异和破坏性 API 更改。

#### 检查最新 Swagger

作为确保我们始终保持 swagger yaml 最新的方法，我们的 ADO 管道中有一个步骤，如果生成的内容与签入的 yaml 有差异，将生成 swagger 并出错。
此脚本位于 ./build/common/scripts/CheckForSwaggerChanges.ps1

您也可以在本地运行此脚本：

```
.\build\common\scripts\CheckForSwaggerChanges.ps1  -SwaggerDir 'swagger' -AssemblyDir 'src\Microsoft.Health.Dicom.Web\bin\x64\Debug\net6.0\Microsoft.Health.Dicom.Web.dll' -Version 'v1-prerelease','v1'
```

请注意，此脚本不使用 `dotnet swagger` 的比较来检测更改，因为它只查看 API 更改。
我们希望比较整个文件，因此我们使用 PowerShell 的 `Compare-Object`。

#### 检查破坏性 API 更改

作为确保我们始终考虑破坏性 API 更改的方法，我们的 ADO 管道中有一个步骤，如果签入的内容与主分支中的 yaml 相比有破坏性更改，将出错。
此脚本位于 ./build/common/scripts/CheckForBreakingAPISwaggerChanges.ps1

您也可以在本地运行此脚本：

```
.\build\common\scripts\CheckForBreakingAPISwaggerChanges.ps1  -SwaggerDir 'swagger' -Version 'v1-prerelease','v1'
```

无更改的示例输出：

```

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         8/10/2022   9:22 AM                FromMain
Running comparison with baseline for version v1-prerelease
old: swagger\FromMain\v1-prerelease.yaml
new: swagger\v1-prerelease\swagger.yaml
No differences. Specifications are equivalents
Running comparison with baseline for version v1
old: swagger\FromMain\v1.yaml
new: swagger\v1\swagger.yaml
No differences. Specifications are equivalents


PS C:\dev\hls\dicom-server>
```
