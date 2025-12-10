# 使用 cURL 的 DICOMWeb&trade; 标准 API

本教程使用 cURL 演示如何使用 Medical Imaging Server for DICOM。

在本教程中，我们将使用这里的 DICOM 文件：[示例 DICOM 文件](../dcms)。示例 DICOM 文件的文件名、studyUID、seriesUID 和 instanceUID 如下：

| 文件 | StudyUID | SeriesUID | InstanceUID |
| --- | --- | --- | ---|
|green-square.dcm|1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420|1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652|1.2.826.0.1.3680043.8.498.12714725698140337137334606354172323212|
|red-triangle.dcm|1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420|1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652|1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395|
|blue-circle.dcm|1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420|1.2.826.0.1.3680043.8.498.77033797676425927098669402985243398207|1.2.826.0.1.3680043.8.498.13273713909719068980354078852867170114|

> 注意：这些文件中的每一个都代表一个实例，并且是同一研究的一部分。此外，green-square 和 red-triangle 是同一序列的一部分，而 blue-circle 在单独的序列中。

## 先决条件

为了使用 DICOMWeb&trade; 标准 API，您必须部署 Medical Imaging Server for DICOM 的实例。如果您尚未部署 Medical Imaging Server，请[将 Medical Imaging Server 部署到 Azure](../quickstarts/deploy-via-azure.md)。

部署 Medical Imaging Server for DICOM 实例后，检索 App Service 的 URL：

1. 登录 [Azure 门户](https://portal.azure.com/)。
2. 搜索 **App Services** 并选择您的 Medical Imaging Server for DICOM App Service。
3. 复制 App Service 的 **URL**。
4. 将您要使用的版本附加到应用服务的末尾（例如 `https://<app_service_url>/v<version>/`），并在以下所有示例中将其用作 DICOM 服务的基础 URL。有关版本控制的更多信息，请访问 [API 版本控制文档](../api-versioning.md)。

对于此代码，我们将访问不安全的开发/测试服务。请不要上传任何私人健康信息 (PHI)。

## 使用 Medical Imaging Server for DICOM

DICOMweb&trade; 标准大量使用 `multipart/related` HTTP 请求结合 DICOM 特定的 accept 标头。熟悉其他基于 REST 的 API 的开发人员通常发现使用 DICOMweb&trade; 标准很麻烦。但是，一旦启动并运行，它很容易使用。刚开始需要一点技巧。

每个 cURL 命令都包含至少一个，有时两个必须替换的变量。为了简化命令的运行，请对以下变量进行搜索和替换，将它们替换为您的特定值：

* {base-url} : 这是您在上面步骤中创建的 URL，包括服务 URL 和版本。
* {path-to-dicoms} : 包含 red-triangle.dcm 文件的目录路径，例如 `C:/dicom-server/docs/dcms`
    * 确保使用正斜杠作为分隔符，并且目录末尾*没有*尾随正斜杠。     

---
## 上传 DICOM 实例 (STOW)
---
### 使用 multipart/related 存储实例

此请求旨在演示如何使用 multipart/related 上传 DICOM 文件。

> 注意：Medical Imaging Server for DICOM 比 DICOM 标准更宽松。但是，下面的示例演示了严格符合标准的 POST 请求。

_详细信息：_

* 路径：../studies
* 方法：POST
* 标头：
    * `Accept: application/dicom+json`
    * `Content-Type: multipart/related; type="application/dicom"`
* 正文：
    * 每个上传文件的 `Content-Type: application/dicom`，由边界值分隔

> 某些编程语言和工具的行为不同。例如，有些要求您定义自己的边界。对于这些，您可能需要使用稍微修改的 Content-Type 标头。以下已成功使用。
 > * `Content-Type: multipart/related; type="application/dicom"; boundary=ABCD1234`
 > * `Content-Type: multipart/related; boundary=ABCD1234`
 > * `Content-Type: multipart/related`

`curl --location --request POST "{base-url}/studies" --header "Accept: application/dicom+json" --header "Content-Type: multipart/related; type=\"application/dicom\"" --form "file1=@{path-to-dicoms}/red-triangle.dcm;type=application/dicom" --trace-ascii "trace.txt"`

---

### 为特定研究存储实例

此请求演示如何使用 multipart/related 将 DICOM 文件上传到指定的研究。

_详细信息：_
* 路径：../studies/{study}
* 方法：POST
* 标头：
    * `Accept: application/dicom+json`
    * `Content-Type: multipart/related; type="application/dicom"`
* 正文：
    * 每个上传文件的 `Content-Type: application/dicom`，由边界值分隔

> 某些编程语言和工具的行为不同。例如，有些要求您定义自己的边界。对于这些，您可能需要使用稍微修改的 Content-Type 标头。以下已成功使用。
 > * `Content-Type: multipart/related; type="application/dicom"; boundary=ABCD1234`
 > * `Content-Type: multipart/related; boundary=ABCD1234`
 > * `Content-Type: multipart/related`

`curl --request POST "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420" --header "Accept: application/dicom+json" --header "Content-Type: multipart/related; type=\"application/dicom\"" --form "file1=@{path-to-dicoms}/blue-circle.dcm;type=application/dicom"`

---

### 存储单个实例

> 注意：这是一个非标准 API，允许上传单个 DICOM 文件，而无需为 multipart/related 配置 POST。尽管 cURL 可以很好地处理 multipart/related，但此 API 允许 Postman 等工具将文件上传到 Medical Imaging Server for DICOM。

以下是上传单个 DICOM 文件所需的。

_详细信息：_
* 路径：../studies
* 方法：POST
* 标头：
   *  `Accept: application/dicom+json`
   *  `Content-Type: application/dicom`
* 正文：
    * 包含单个 DICOM 文件作为二进制字节。

`curl --location --request POST "{base-url}/studies" --header "Accept: application/dicom+json" --header "Content-Type: application/dicom" --data-binary "@{path-to-dicoms}/green-square.dcm"`

---
## 检索 DICOM (WADO)
---
### 检索研究中的所有实例

此请求检索单个研究中的所有实例，并将它们作为 multipart/related 字节集合返回。

_详细信息：_
* 路径：../studies/{study}
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/dicom"; transfer-syntax=*`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420" --header "Accept: multipart/related; type=\"application/dicom\"; transfer-syntax=*" --output "suppressWarnings.txt"`

> 此 cURL 命令将在输出文件 (suppressWarnings.txt) 中显示下载的字节，但这些不是直接的 DICOM 文件，只是 multipart/related 下载的文本表示。

---
### 检索研究中所有实例的元数据

此请求检索单个研究中所有实例的元数据。

_详细信息：_
* 路径：../studies/{study}/metadata
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

> 此 cURL 命令将在输出文件 (suppressWarnings.txt) 中显示下载的字节，但这些不是直接的 DICOM 文件，只是 multipart/related 下载的文本表示。

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/metadata" --header "Accept: application/dicom+json"`

---
### 检索序列中的所有实例

此请求检索单个序列中的所有实例，并将它们作为 multipart/related 字节集合返回。

_详细信息：_
* 路径：../studies/{study}/series/{series}
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/dicom"; transfer-syntax=*`

> 此 cURL 命令将在输出文件 (suppressWarnings.txt) 中显示下载的字节，但它不是 DICOM 文件，只是 multipart/related 下载的文本表示。

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652" --header "Accept: multipart/related; type=\"application/dicom\"; transfer-syntax=*" --output "suppressWarnings.txt"`

---
### 检索序列中所有实例的元数据

此请求检索单个研究中所有实例的元数据。

_详细信息：_
* 路径：../studies/{study}/series/{series}/metadata
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/metadata" --header "Accept: application/dicom+json"`

---
### 检索研究的序列中的单个实例

此请求检索单个实例，并将其作为 DICOM 格式的字节流返回。

_详细信息：_
* 路径：../studies/{study}/series{series}/instances/{instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom; transfer-syntax=*`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/instances/1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395" --header "Accept: application/dicom; transfer-syntax=*" --output "suppressWarnings.txt"`

---
### 检索研究的序列中的单个实例的元数据

此请求检索单个研究和序列中的单个实例的元数据。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances/{instance}/metadata
* 方法：GET
* 标头：
  * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/instances/1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395/metadata" --header "Accept: application/dicom+json"`

---
### 从单个实例检索一个或多个帧

此请求从单个实例检索一个或多个帧，并将它们作为 multipart/related 字节集合返回。可以通过传递逗号分隔的帧号列表来检索多个帧。所有带有图像的 DICOM 实例至少有一帧，这通常只是与实例本身关联的图像。

_详细信息：_
* 路径：../studies/{study}/series{series}/instances/{instance}/frames/1,2,3
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/octet-stream"; transfer-syntax=1.2.840.10008.1.2.1` (默认) 或
   * `Accept: multipart/related; type="application/octet-stream"; transfer-syntax=*` 或
   * `Accept: multipart/related; type="application/octet-stream";`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/instances/1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395/frames/1" --header "Accept: multipart/related; type=\"application/octet-stream\"; transfer-syntax=1.2.840.10008.1.2.1" --output "suppressWarnings.txt"`

---
## 查询 DICOM (QIDO)

在以下示例中，我们使用其唯一标识符搜索项目。您也可以搜索其他属性，例如 PatientName。

---
### 搜索研究

此请求支持按 DICOM 属性搜索一个或多个研究。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../studies?StudyInstanceUID={study}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies?StudyInstanceUID=1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420" --header "Accept: application/dicom+json"`

---
### 搜索序列

此请求支持按 DICOM 属性搜索一个或多个序列。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../series?SeriesInstanceUID={series}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/series?SeriesInstanceUID=1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652" --header "Accept: application/dicom+json"`

---
### 在研究内搜索序列

此请求支持按 DICOM 属性在单个研究内搜索一个或多个序列。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../studies/{study}/series?SeriesInstanceUID={series}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series?SeriesInstanceUID=1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652" --header "Accept: application/dicom+json"`

---
### 搜索实例

此请求支持按 DICOM 属性搜索一个或多个实例。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/instances?SOPInstanceUID=1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395" --header "Accept: application/dicom+json"`

---
### 在研究内搜索实例

此请求支持按 DICOM 属性在单个研究内搜索一个或多个实例。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../studies/{study}/instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/instances?SOPInstanceUID=1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395" --header "Accept: application/dicom+json"`

---
### 在研究和序列内搜索实例

此请求支持按 DICOM 属性在单个研究和单个序列内搜索一个或多个实例。

> 请参阅 [Conformance.md](../docs/resources/conformance-statement.md) 文件以了解支持的 DICOM 属性。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

`curl --request GET "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/instances?SOPInstanceUID=1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395" --header "Accept: application/dicom+json"`

---
## 删除 DICOM 
---
### 删除研究和序列中的特定实例

此请求删除单个研究和单个序列中的单个实例。

> 删除不是 DICOM 标准的一部分，但已添加以便于使用。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances/{instance}
* 方法：DELETE
* 标头：不需要特殊标头

`curl --request DELETE "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652/instances/1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395"`

---
### 删除研究中的特定序列

此请求删除单个研究中的单个序列（以及所有子实例）。

> 删除不是 DICOM 标准的一部分，但已添加以便于使用。

_详细信息：_
* 路径：../studies/{study}/series/{series}
* 方法：DELETE
* 标头：不需要特殊标头

`curl --request DELETE "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420/series/1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652"`

---
### 删除特定研究

此请求删除单个研究（以及所有子序列和实例）。

> 删除不是 DICOM 标准的一部分，但已添加以便于使用。

_详细信息：_
* 路径：../studies/{study}
* 方法：DELETE
* 标头：不需要特殊标头

`curl --request DELETE "{base-url}/studies/1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420"`
