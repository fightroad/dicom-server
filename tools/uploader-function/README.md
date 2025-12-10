# Uploader Function

该方案使用 [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/) 框架实现自动上传到 DICOM Service。以下步骤使用 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) 完成环境配置。

步骤：
## 1. 创建 App Service 计划
创建可托管容器函数的 Linux Premium 计划。

```
az functionapp plan create --resource-group {resource-group-name} --name {app-service-plan-name} --location {REGION} --number-of-workers 1 --sku EP1 --is-linux
```

| Property | Value |
| --- | --- |
| {resource-group-name} | The resource group that you would like to place your app service plan in. |
| {app-service-plan-name} | The name of your app service plan. |

## 2. 创建 Function App
创建函数应用。

> 说明：可忽略关于容器注册表凭据的警告，镜像为公开访问。

```
az functionapp create --name {function-app-name} --storage-account {storage-account-name} --resource-group {resource-group-name} --plan {app-service-plan-name} --deployment-container-image-name dicomoss.azurecr.io/dicom-uploader:latest --functions-version 4 --assign-identity [system]
```

| Property | Value |
| --- | --- |
| {function-app-name} | The name of your uploader function app. |
| {storage-account-name} | The name of the storage account that the function will keep state in. Can be the same or different from the the DICOM file source. |
| {resource-group-name} | The resource group you would like to place your function app in. | 
| {app-service-plan-name} | The app service plan you created in the previous step. |

## 3. 为函数的托管身份授予权限

托管身份需要以下角色：

| Role | Resource |
| --- | --- |
| Dicom Data Owner | The DICOM Service that the uploader will write to. |
| [Storage Blob Data Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor) | The storage account used as the source of the DICOM files. |
| [Storage Queue Data Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#storage-queue-data-contributor) | The storage account used as the source of the DICOM files. |


## 4. 更新配置
为上传函数设置必要的应用配置。

```
az functionapp config appsettings set --name {function-app-name} --resource-group {resource-group-name} --settings "sourcestorage__blobcontainer={source-container-name}" "sourcestorage__blobServiceUri={source-blob-url}" "sourcestorage__queueServiceUri={source-queue-url}" "DicomWeb__Endpoint={dicom-service-endpoint}" "DicomWeb__Authentication__Enabled=true" "DicomWeb__Authentication__AuthenticationType=ManagedIdentity" "DicomWeb__Authentication__ManagedIdentityCredential__Resource=https://dicom.healthcareapis.azure.com"
```

| Property | Value |
| --- | --- |
| {function-app-name} | The name of your uploader function app. |
| {resource-group-name} | The resource group of your function app. | 
| {source-container-name} | The name of the container that contains your DICOM files located in the storage service referenced by {source-blob-url}. |
| {source-blob-url} | The URL of your blob service that contains your DICOM files. |
| {source-queue-url} | The URL of your queue service of the storage account that contains your DICOM files. This service is used to create a poison queue and messages in if there is a failure to upload a DICOM file. |
| {dicom-service-endpoint} | The DICOM Service endpoint that you wish to upload DICOM files to. |

## 故障排查

* 已发布的 Application Insights 会记录服务运行的跟踪和异常，可参考[查询遥测数据](https://docs.microsoft.com/en-us/azure/azure-functions/analyze-telemetry-data#query-telemetry-data)。
* 函数处理 Blob 后会在步骤 1 的 `{storage-account-name}` 写入 [blob receipt](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#blob-receipts)。
* 处理失败时，会在 `{source-queue-url}` 的队列写入 [Poison blob](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp#poison-blobs)。