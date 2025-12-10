# Functions

我们利用 "functions" 将工作卸载到后台。这对于不需要立即事务性完成的工作很有用。这种情况的实例是更新或删除文件。

为了完成工作，我们使用 [Durable Functions](https://learn.microsoft.com/en-us/azure/azure-functions/durable/)，它是 Azure Functions 的扩展，允许您在无服务器计算环境中编写有状态函数。

## 破坏性变更
有几个需要注意的破坏性变更示例，以及如何处理这些变更的示例可以在[这里](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-versioning?tabs=csharp#how-to-handle-breaking-changes)找到。

此存储库中用于版本化函数的特定策略是通过更新函数名称中的版本名称，称为[并行部署](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-versioning?tabs=csharp#side-by-side-deployments)。

部署带有新函数版本的新代码后，正在运行/运行的函数保留的函数状态可能是先前版本的状态。因此，我们需要注意向后兼容性和破坏性变更。

请始终记住，编排的函数状态保存在 JSON 中，必须能够来回进行序列化/反序列化。通常，为了避免不必要的复杂性，对函数甚至编排进行版本化比尝试编写测试以确保一切都能在所做的更改中继续工作更容易。

此外，请确保在新函数发布后快速清理"旧"函数，这样我们就不会有过时的代码。

## 常见陷阱

当由于输入参数已更改而对函数进行版本化时，请注意参数的 CTOR 并确保字段名称等同于它们被分配给的属性。如果不这样做，函数主机的 JSON 序列化/反序列化器将无法理解如何将参数序列化/反序列化到 CTOR，您可能会看到*看似传入且非空的参数被分配为 null*。

## host.json
您会看到 host.json 和 Web 服务器的 appsettings 之间存在一些重复。在本地测试时，请确保设置所有需要的值，以便本地主机知道如何利用这些功能。

## 本地测试

无论是使用 E2E 测试还是运行 Web 服务器，默认情况下将使用本地运行的 Azure 模拟器来存储状态。如果函数失败，可能需要手动清理此状态，方法是在 Azure Storage Explorer 中删除 DicomTaskHubInstances 和 DicomTaskHubHistory 表。

请注意，为 IDP 启用 External Store 仅意味着 dcm 文件 blob 将写入外部。所有函数数据将继续写入默认本地存储。
如果启用 External Store，请确保也使用设置的标志运行测试：
dotnet test "Microsoft.Health.Dicom.Web.Tests.E2E.dll" --filter "Category=bvt-dp" -e DicomServer__Features__EnableExternalStore="true"

通常，此标志也可以在您的 IDE 中设置。例如：
![img.png](img.png)

### E2E 测试

运行 E2E 测试并在函数代码中放置调试点以逐步执行代码。

### 向服务器发出请求以激活函数

要在本地测试，您需要打开两个独立的 IDE 窗口：
1. 一个用于运行 Web 服务器
2. 一个用于运行函数主机

这样，您可以通过 Postman 或 curl 向 Web 服务器发出请求，函数主机将接收请求。
这将允许您调试并逐步执行函数代码。这是我们除了使用 E2E 测试之外，在本地测试完整集成最接近的方法。

## 构建失败
如果由于函数失败而导致构建失败，您可以查看 Application Insights 应用程序的日志以识别原因。不要忽略这些。这些不应该是不稳定的，如果有任何不稳定的，请修复它们。
