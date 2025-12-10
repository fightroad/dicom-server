# 开发

## 要求
- [Azurite 存储模拟器](https://go.microsoft.com/fwlink/?linkid=717179)
- SQL Server 2019（带全文索引功能）
- .NET core SDK 版本在[这里](/global.json)指定
   - https://dotnet.microsoft.com/download/dotnet-core/6.0 

## 在 Visual Studio 中开始

### 开发
- 安装 Visual Studio 2022
- [克隆 Medical Imaging Server for DICOM 存储库](https://github.com/microsoft/dicom-server.git)
- 导航到克隆的存储库
- 在 VS 中打开 `Microsoft.Health.Dicom.sln`
- 构建
- 确保存储模拟器正在运行
- 从测试资源管理器运行所有测试

# 测试
- 将 Microsoft.Health.Dicom.Web 设置为启动项目
- 运行项目
- Web 服务器现在在 https://localhost:63838/ 运行

## 使用 Fiddler 发布 DICOM 文件
- [安装 fiddler](https://www.telerik.com/download/fiddler)
- 在 fiddler 中转到 Tools->Options->HTTPS。单击协议并将 "tls1.2" 添加到协议列表。

![Fiddler 配置图像](/docs/images/FiddlerConfig.png)
- 从[这里](/docs/dcms) 下载 DCM 示例文件
- 上传 DCM 文件
   - 使用请求体部分的 `Upload file...` 链接，如下图所示
      - 位于 `Composer` 选项卡内的 `Parsed` 选项卡中
- 更新请求头：
   - `Accept: application/dicom+json`
   - `Content-Type: multipart/related` **（不要更改边界部分）**
- 更新请求体：
   - `Content-Type: application/dicom`
- 将请求发布到 `https://localhost:63838/v<version>/studies`
   - 单击执行按钮

![发布 DICOM 图像](/docs/images/FiddlerPost.png)
- 如果 POST 成功，您应该看到 HTTP 200 响应。

![发布成功](/docs/images/FiddlerSucceedPost.png)
- 注意：除非先删除，否则不能再次上传相同的 DCM 文件。这样做将导致 HTTP 409 冲突错误。

## 使用 Postman 发布 DICOM 文件
Postman 仅支持使用 DICOM 标准中定义的单部分有效负载上传 DICOM 文件。这是因为 Postman 无法支持 multipart/related POST 请求中的自定义分隔符。
- [安装 Postman](https://www.postman.com/downloads/)
- 将请求设置为对 url `https://localhost:63838/v<version>/studies` 的 post 请求
- 在 Postman 的 `Body` 选项卡中选择 `binary`，然后选择要上传的单部分 DICOM 文件
- 在标头部分添加以下内容。所有标头应如下面的图像所示匹配
   - Accept: `application/dicom+json`
   - Content-Type: `application/dicom`
- 单击 `Send`，成功的 post 将导致 200 OK

![Postman 标头](/docs/images/postman-singlepart-headers.PNG)

## Postman 用于 Get
- 示例 QIDO 获取所有研究
```http
GET https://localhost:63838/v<version>/studies
accept: application/dicom+json
```
