# 使用 Python 的 DICOMweb&trade; 标准 API

> 本文档是从 [这里](../resources/use-dicom-web-standard-apis-with-python.ipynb) 找到的 Jupyter Notebook 的 Markdown 导出。通过在 Jupyter 中打开笔记本，您可以在完全交互式体验中逐步完成示例。

本教程使用 Python 演示如何使用 Medical Imaging Server for DICOM。

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

对于此代码，我们将访问不安全的开发/测试服务。请不要上传任何私人健康信息 (PHI)。

## 使用 Medical Imaging Server for DICOM

DICOMweb&trade; 标准大量使用 `multipart/related` HTTP 请求结合 DICOM 特定的 accept 标头。熟悉其他基于 REST 的 API 的开发人员通常发现使用 DICOMweb&trade; 标准很麻烦。但是，一旦启动并运行，它很容易使用。刚开始需要一点技巧。

### 导入适当的 Python 库

首先，导入必要的 Python 库。

我们选择使用同步的 `requests` 库来实现此示例。对于异步支持，请考虑使用 `httpx` 或其他异步库。此外，我们从 `urllib3` 导入两个支持函数以支持使用 `multipart/related` 请求。

```python
import requests
import pydicom
from pathlib import Path
from urllib3.filepost import encode_multipart_formdata, choose_boundary
```

### 配置要在整个过程中使用的用户定义变量
将所有用 { } 包装的变量值替换为您自己的值。此外，验证任何构造的变量是否正确。例如，`base_url` 是使用 Azure App Service 的默认 URL 构造的，然后附加正在使用的 REST API 版本。有关版本控制的更多信息，请访问 [API 版本控制文档](../api-versioning.md)。如果您使用自定义 URL，则需要用自己的值覆盖该值。

```python
dicom_server_name = "{server-name}"
path_to_dicoms_dir = "{path to the folder that includes green-square.dcm and other dcm files}"
version = "{version of rest api to use}"

base_url = f"https://{dicom_server_name}.azurewebsites.net/v{version}"

study_uid = "1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420"; #StudyInstanceUID for all 3 examples
series_uid = "1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652"; #SeriesInstanceUID for green-square and red-triangle
instance_uid = "1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395"; #SOPInstanceUID for red-triangle
```

### 创建支持方法来支持 `multipart\related`
`Requests` 库（和大多数 Python 库）不支持以支持 DICOMweb&trade; 的方式使用 `multipart\related`。因此，我们需要添加一些方法来支持使用 DICOM 文件。

`encode_multipart_related` 接受一组字段（在 DICOM 情况下，这些通常是第 10 部分 dcm 文件）和一个可选的用户定义边界。它返回完整的主体以及 content_type，可以使用

```python
def encode_multipart_related(fields, boundary=None):
    if boundary is None:
        boundary = choose_boundary()

    body, _ = encode_multipart_formdata(fields, boundary)
    content_type = str('multipart/related; boundary=%s' % boundary)

    return body, content_type
```

### 创建 `requests` 会话
创建一个名为 `client` 的 `requests` 会话，用于与 Medical Imaging Server for DICOM 通信。

```python
client = requests.session()
```

--------------------
## 上传 DICOM 实例 (STOW)

以下示例重点介绍持久化 DICOM 文件。

### 使用 multipart/related 存储实例

这演示了如何上传单个 DICOM 文件。这使用了一点 Python 技巧将 DICOM 文件（作为字节）预加载到内存中。通过将文件数组传递给 encode_multipart_related 的 fields 参数，可以在单个 POST 中上传多个文件。这有时用于上传完整的序列或研究。

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

```python
#upload blue-circle.dcm
filepath = Path(path_to_dicoms_dir).joinpath('blue-circle.dcm')

# Hack. Need to open up and read through file and load bytes into memory 
with open(filepath,'rb') as reader:
    rawfile = reader.read()
files = {'file': ('dicomfile', rawfile, 'application/dicom')}

#encode as multipart_related
body, content_type = encode_multipart_related(fields = files)

headers = {'Accept':'application/dicom+json', "Content-Type":content_type}

url = f'{base_url}/studies'
response = client.post(url, body, headers=headers, verify=False)
```

### 为特定研究存储实例

这演示了如何将多个 DICOM 文件上传到指定的研究。这使用了一点 Python 技巧将 DICOM 文件（作为字节）预加载到内存中。

通过将文件数组传递给 `encode_multipart_related` 的 fields 参数，可以在单个 POST 中上传多个文件。这有时用于上传完整的序列或研究。

_详细信息：_
* 路径：../studies/{study}
* 方法：POST
* 标头：
    * `Accept: application/dicom+json`
    * `Content-Type: multipart/related; type="application/dicom"`
* 正文：
    * 每个上传文件的 `Content-Type: application/dicom`，由边界值分隔

```python

filepath_red = Path(path_to_dicoms_dir).joinpath('red-triangle.dcm')
filepath_green = Path(path_to_dicoms_dir).joinpath('green-square.dcm')

# Hack. Need to open up and read through file and load bytes into memory 
with open(filepath_red,'rb') as reader:
    rawfile_red = reader.read()
with open(filepath_green,'rb') as reader:
    rawfile_green = reader.read()  
       
files = {'file_red': ('dicomfile', rawfile_red, 'application/dicom'),
         'file_green': ('dicomfile', rawfile_green, 'application/dicom')}

#encode as multipart_related
body, content_type = encode_multipart_related(fields = files)

headers = {'Accept':'application/dicom+json', "Content-Type":content_type}

url = f'{base_url}/studies'
response = client.post(url, body, headers=headers, verify=False)
```

### 存储单个实例（非标准）

这演示了如何上传单个 DICOM 文件。此非标准 API 端点简化了上传单个文件，作为在请求正文中发送的二进制字节

_详细信息：_
* 路径：../studies
* 方法：POST
* 标头：
   *  `Accept: application/dicom+json`
   *  `Content-Type: application/dicom`
* 正文：
    * 包含单个 DICOM 文件作为二进制字节。

```python
#upload blue-circle.dcm
filepath = Path(path_to_dicoms_dir).joinpath('blue-circle.dcm')

# Hack. Need to open up and read through file and load bytes into memory 
with open(filepath,'rb') as reader:
    body = reader.read()

headers = {'Accept':'application/dicom+json', 'Content-Type':'application/dicom'}

url = f'{base_url}/studies'
response = client.post(url, body, headers=headers, verify=False)
response  # response should be a 409 Conflict if the file was already uploaded in the above request
```

--------------------
## 检索 DICOM 实例 (WADO)

以下示例重点介绍检索 DICOM 实例。

------

### 检索研究中的所有实例

这检索单个研究中的所有实例。

_详细信息：_
* 路径：../studies/{study}
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/dicom"; transfer-syntax=*`

我们之前上传的所有三个 dcm 文件都是同一研究的一部分，因此响应应该返回所有 3 个实例。验证响应具有 OK 状态代码并且返回了所有三个实例。

```python
url = f'{base_url}/studies/{study_uid}'
headers = {'Accept':'multipart/related; type="application/dicom"; transfer-syntax=*'}

response = client.get(url, headers=headers) #, verify=False)
```

### 使用检索到的实例

实例作为二进制字节检索。您可以循环遍历返回的项目，并将字节转换为可由 `pydicom` 读取的类似文件的结构。

```python
import requests_toolbelt as tb
from io import BytesIO

mpd = tb.MultipartDecoder.from_response(response)
for part in mpd.parts:
    # Note that the headers are returned as binary!
    print(part.headers[b'content-type'])
    
    # You can convert the binary body (of each part) into a pydicom DataSet
    #   And get direct access to the various underlying fields
    dcm = pydicom.dcmread(BytesIO(part.content))
    print(dcm.PatientName)
    print(dcm.SOPInstanceUID)
```

### 检索研究中所有实例的元数据

此请求检索单个研究中所有实例的元数据。

_详细信息：_
* 路径：../studies/{study}/metadata
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

我们之前上传的所有三个 dcm 文件都是同一研究的一部分，因此响应应该返回所有 3 个实例的元数据。验证响应具有 OK 状态代码并且返回了所有元数据。

```python
url = f'{base_url}/studies/{study_uid}/metadata'
headers = {'Accept':'application/dicom+json'}

response = client.get(url, headers=headers) #, verify=False)
```

### 检索序列中的所有实例

这检索单个序列中的所有实例。

_详细信息：_
* 路径：../studies/{study}/series/{series}
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/dicom"; transfer-syntax=*`

此序列有 2 个实例（green-square 和 red-triangle），因此响应应该返回两个实例。验证响应具有 OK 状态代码并且返回了两个实例。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}'
headers = {'Accept':'multipart/related; type="application/dicom"; transfer-syntax=*'}

response = client.get(url, headers=headers) #, verify=False)
```

### 检索序列中所有实例的元数据

此请求检索单个序列中所有实例的元数据。

_详细信息：_
* 路径：../studies/{study}/series/{series}/metadata
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

此序列有 2 个实例（green-square 和 red-triangle），因此响应应该返回两个实例的元数据。验证响应具有 OK 状态代码并且返回了两个实例的元数据。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/metadata'
headers = {'Accept':'application/dicom+json'}

response = client.get(url, headers=headers) #, verify=False)
```

### 检索研究的序列中的单个实例

此请求检索单个实例。

_详细信息：_
* 路径：../studies/{study}/series{series}/instances/{instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom; transfer-syntax=*`

这应该只返回实例 red-triangle。验证响应具有 OK 状态代码并且返回了实例。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/instances/{instance_uid}'
headers = {'Accept':'application/dicom; transfer-syntax=*'}

response = client.get(url, headers=headers) #, verify=False)
```

### 检索研究的序列中的单个实例的元数据

此请求检索单个研究和序列中的单个实例的元数据。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances/{instance}/metadata
* 方法：GET
* 标头：
  * `Accept: application/dicom+json`

这应该只返回实例 red-triangle 的元数据。验证响应具有 OK 状态代码并且返回了元数据。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/instances/{instance_uid}/metadata'
headers = {'Accept':'application/dicom+json'}

response = client.get(url, headers=headers) #, verify=False)
```

### 从单个实例检索一个或多个帧

此请求从单个实例检索一个或多个帧。

_详细信息：_
* 路径：../studies/{study}/series{series}/instances/{instance}/frames/1,2,3
* 方法：GET
* 标头：
   * `Accept: multipart/related; type="application/octet-stream"; transfer-syntax=1.2.840.10008.1.2.1` (默认) 或
   * `Accept: multipart/related; type="application/octet-stream"; transfer-syntax=*` 或
   * `Accept: multipart/related; type="application/octet-stream";`

这应该只返回 red-triangle 的唯一帧。验证响应具有 OK 状态代码并且返回了帧。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/instances/{instance_uid}/frames/1'
headers = {'Accept':'multipart/related; type="application/octet-stream"; transfer-syntax=*'}

response = client.get(url, headers=headers) #, verify=False)
```

--------------------
## 查询 DICOM (QIDO)

在以下示例中，我们使用其唯一标识符搜索项目。您也可以搜索其他属性，例如 PatientName。

> 请参阅[符合性声明](../resources/conformance-statement.md#supported-search-parameters)文件以了解支持的 DICOM 属性。

---
### 搜索研究

此请求按 DICOM 属性搜索一个或多个研究。

_详细信息：_
* 路径：../studies?StudyInstanceUID={study}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个研究并且响应代码为 OK。

```python
url = f'{base_url}/studies'
headers = {'Accept':'application/dicom+json'}
params = {'StudyInstanceUID':study_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

### 搜索序列

此请求按 DICOM 属性搜索一个或多个序列。

_详细信息：_
* 路径：../series?SeriesInstanceUID={series}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个序列并且响应代码为 OK。

```python
url = f'{base_url}/series'
headers = {'Accept':'application/dicom+json'}
params = {'SeriesInstanceUID':series_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

### 在研究内搜索序列

此请求按 DICOM 属性在单个研究内搜索一个或多个序列。

_详细信息：_
* 路径：../studies/{study}/series?SeriesInstanceUID={series}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个序列并且响应代码为 OK。

```python
url = f'{base_url}/studies/{study_uid}/series'
headers = {'Accept':'application/dicom+json'}
params = {'SeriesInstanceUID':series_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

### 搜索实例

此请求按 DICOM 属性搜索一个或多个实例。

_详细信息：_
* 路径：../instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个实例并且响应代码为 OK。

```python
url = f'{base_url}/instances'
headers = {'Accept':'application/dicom+json'}
params = {'SOPInstanceUID':instance_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

### 在研究内搜索实例

此请求按 DICOM 属性在单个研究内搜索一个或多个实例。

_详细信息：_
* 路径：../studies/{study}/instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个实例并且响应代码为 OK。

```python
url = f'{base_url}/studies/{study_uid}/instances'
headers = {'Accept':'application/dicom+json'}
params = {'SOPInstanceUID':instance_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

### 在研究和序列内搜索实例

此请求按 DICOM 属性在单个研究和单个序列内搜索一个或多个实例。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances?SOPInstanceUID={instance}
* 方法：GET
* 标头：
   * `Accept: application/dicom+json`

验证响应包括 1 个实例并且响应代码为 OK。

```python
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/instances'
headers = {'Accept':'application/dicom+json'}
params = {'SOPInstanceUID':instance_uid}

response = client.get(url, headers=headers, params=params) #, verify=False)
```

-----------------
## 删除 DICOM

> 注意：删除不是 DICOM 标准的一部分，但已添加以便于使用。

删除成功时返回 204 响应代码。如果项目从未存在或已被删除，则返回 404 响应代码。

### 删除研究和序列中的特定实例

此请求删除单个研究和单个序列中的单个实例。

_详细信息：_
* 路径：../studies/{study}/series/{series}/instances/{instance}
* 方法：DELETE
* 标头：不需要特殊标头

这从服务器删除 red-triangle 实例。如果成功，响应状态代码不包含内容。

```python
#headers = {'Accept':'anything/at+all'}
url = f'{base_url}/studies/{study_uid}/series/{series_uid}/instances/{instance_uid}'
response = client.delete(url) 
```

### 删除研究中的特定序列

此请求删除单个研究中的单个序列（以及所有子实例）。

_详细信息：_
* 路径：../studies/{study}/series/{series}
* 方法：DELETE
* 标头：不需要特殊标头

这从服务器删除 green-square 实例（它是序列中唯一剩余的元素）。如果成功，响应状态代码不包含内容。

```python
#headers = {'Accept':'anything/at+all'}
url = f'{base_url}/studies/{study_uid}/series/{series_uid}'
response = client.delete(url) 
```

### 删除特定研究

此请求删除单个研究（以及所有子序列和实例）。

_详细信息：_
* 路径：../studies/{study}
* 方法：DELETE
* 标头：不需要特殊标头

```python
#headers = {'Accept':'anything/at+all'}
url = f'{base_url}/studies/{study_uid}'
response = client.delete(url) 
```
