# 健康检查 API

健康检查 API 允许用户检查 Medical Imaging Server for DICOM 和所有基础服务的健康状态。

## API 设计

健康检查 API 公开一个 GET 端点并以 JSON 内容响应。

动词 | 路由              | 返回     
:--- | :----------------- | :---------- 
GET  | /health/check      | Json Object 

## 对象模型

GET 请求返回一个具有以下字段的 JSON 对象：

字段         | 类型   | 描述
:------------ | :----- | :----------
overallStatus | string | 状态 `Healthy` 或 `Unhealthy`
details       | array  | 包含基础服务详细信息的对象数组

`details` 数组的对象具有以下模型：

字段         | 类型   | 描述
:------------ | :----- | :----------
name		  | string | 服务名称
status		  | string | 状态 `Healthy` 或 `Unhealthy`
description   | string | 状态描述

## 获取健康状态

在内部，使用 Microsoft.Extensions.Diagnostics.HealthChecks NuGet 包来获取健康状态。其文档可以在[这里](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.diagnostics.healthchecks?view=dotnet-plat-ext-3.1)找到。

要检查 Medical Imaging Server for DICOM 的健康状态，用户向 /health/check 发出 GET 请求。以下是如果所有基础服务都健康，示例 JSON 响应：
```
{
	"overallStatus":"Healthy",
	"details":
	[
		{
			"name":"DicomBlobHealthCheck",
			"status":"Healthy",
			"description":"Successfully connected to the blob data store."
		},
		{
			"name":"MetadataHealthCheck",
			"status":"Healthy",
			"description":"Successfully connected to the blob data store."
		},
		{
			"name":"SqlServerHealthCheck",
			"status":"Healthy",
			"description":"Successfully connected to the data store."
		}
	]
}
```

如果所有基础服务都健康，则返回 Healthy（HTTP 状态代码 200）作为总体状态。如果任何基础服务不健康，Medical Imaging Server for DICOM 的总体状态将返回为不健康（HTTP 状态代码 503）。

以下是 SQL Server 服务不健康时的示例 JSON：
```
{
	"overallStatus":"Unhealthy",
	"details":
	[
		{
			"name":"DicomBlobHealthCheck",
			"status":"Healthy",
			"description":"Successfully connected to the blob data store."
		},
		{
			"name":"MetadataHealthCheck",
			"status":"Healthy",
			"description":"Successfully connected to the blob data store."
		},
		{
			"name":"SqlServerHealthCheck",
			"status":"Unhealthy",
			"description":"Failed to connect to the data store."
		}
	]
}
```

响应中的详细信息数组包含所有服务及其健康状态的详细信息。
