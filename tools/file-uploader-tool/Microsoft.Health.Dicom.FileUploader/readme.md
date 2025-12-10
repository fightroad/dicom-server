# 概述
命令行工具，用于将 DICOM 文件上传到目标 DICOMWeb 端点。默认上传 [Images](./Images) 目录下的文件，也可指定目录批量上传 `.dcm` 文件。

可本地运行，或在启用托管身份的 Azure VM 上运行。

# 参数
## --dicomServiceUrl
目标 DICOM 服务 URL，未指定时默认本地标准端口：`https://localhost:63838`。

## --path
包含 `.dcm` 文件的目录，未指定时默认 `/Images`。

# 示例
上传示例文件：
```
dotnet run --dicomServiceUrl "https://testdicomweb-testdicom.dicom.azurehealthcareapis.com"
```

将指定目录的 `.dcm` 批量上传到本地服务：
```
dotnet run --path C:\dicomdir
```
