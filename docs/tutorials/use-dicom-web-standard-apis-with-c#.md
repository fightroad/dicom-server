# 使用 C# 的 DICOMweb&trade; 标准 API

本教程使用 C# 演示如何使用 Medical Imaging Server for DICOM。

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
4. 注意您要使用的 REST API 版本。在下面创建 `DicomWebClient` 时，我们建议传入版本以将客户端固定到特定版本。有关版本控制的更多信息，请访问 [API 版本控制文档](../api-versioning.md)。

在您的应用程序中安装以下 nuget 包：

1.  [Dicom Client](https://microsofthealthoss.visualstudio.com/FhirServer/_packaging?_a=package&feed=Public&package=Microsoft.Health.Dicom.Client&protocolType=NuGet)
2.  [fo-dicom](https://www.nuget.org/packages/fo-dicom/)

## 创建 `DicomWebClient`

部署 Medical Imaging Server for DICOM 后，您将创建一个 'DicomWebClient'。运行以下代码片段以创建 `DicomWebClient`，我们将在本教程的其余部分使用它。确保您已安装上面提到的两个 nuget 包。

```c#
string webServerUrl ="{Your DicomWeb Server URL}"
var httpClient = new HttpClient();
httpClient.BaseAddress = new Uri(webServerUrl);
IDicomWebClient client = new DicomWebClient(httpClient, "v<version>");
```

使用 `DicomWebClient`，我们现在可以执行 Store、Retrieve、Search 和 Delete 操作。

## 存储 DICOM 实例 (STOW)

使用我们创建的 `DicomWebClient`，我们现在可以存储 DICOM 文件。

### 存储单个实例

这演示了如何上传单个 DICOM 文件。

_详细信息：_

* POST /studies

```c#
DicomFile dicomFile = await DicomFile.OpenAsync(@"{Path To blue-circle.dcm}");
DicomWebResponse response = await client.StoreAsync(new[] { dicomFile });
```

### 为特定研究存储实例

这演示了如何将 DICOM 文件上传到指定的研究。

_详细信息：_

* POST /studies/{study}

```c#
DicomFile dicomFile = await DicomFile.OpenAsync(@"{Path To red-triangle.dcm}");
DicomWebResponse response = await client.StoreAsync(new[] { dicomFile }, "1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420");
```

在继续下一部分之前，也使用上述任一方法上传 green-square.dcm 文件。

## 检索 DICOM 实例 (WADO)

以下代码片段将演示如何使用之前创建的 `DicomWebClient` 执行每个检索查询。

以下变量将在其余示例中使用：

```c#
string studyInstanceUid = "1.2.826.0.1.3680043.8.498.13230779778012324449356534479549187420"; //StudyInstanceUID for all 3 examples
string seriesInstanceUid = "1.2.826.0.1.3680043.8.498.45787841905473114233124723359129632652"; //SeriesInstanceUID for green-square and red-triangle
string sopInstanceUid = "1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395"; //SOPInstanceUID for red-triangle
```

### 检索研究中的所有实例

这检索单个研究中的所有实例。

_详细信息：_

* GET /studies/{study}

```c#
DicomWebResponse response = await client.RetrieveStudyAsync(studyInstanceUid);
```

我们之前上传的所有三个 dcm 文件都是同一研究的一部分，因此响应应该返回所有 3 个实例。验证响应具有 OK 状态代码并且返回了所有三个实例。

### 使用检索到的实例

以下代码片段显示了如何访问检索到的实例，如何访问实例的一些字段，以及如何将其保存为 .dcm 文件。

```c#
DicomWebAsyncEnumerableResponse<DicomFile> response = await client.RetrieveStudyAsync(studyInstanceUid);
await foreach (DicomFile file in response)
{
    string patientName = file.Dataset.GetString(DicomTag.PatientName);
    string studyId = file.Dataset.GetString(DicomTag.StudyID);
    string seriesNumber = file.Dataset.GetString(DicomTag.SeriesNumber);
    string instanceNumber = file.Dataset.GetString(DicomTag.InstanceNumber);

    file.Save($"<path_to_save>\\{patientName}{studyId}{seriesNumber}{instanceNumber}.dcm");
}
```

### 检索研究中所有实例的元数据

此请求检索单个研究中所有实例的元数据。

_详细信息：_

* GET /studies/{study}/metadata

```c#
DicomWebResponse response = await client.RetrieveStudyMetadataAsync(studyInstanceUid);
```

我们之前上传的所有三个 dcm 文件都是同一研究的一部分，因此响应应该返回所有 3 个实例的元数据。验证响应具有 OK 状态代码并且返回了所有元数据。

### 检索序列中的所有实例

此请求检索单个序列中的所有实例。

_详细信息：_

* GET /studies/{study}/series/{series}

```c#
DicomWebResponse response = await client.RetrieveSeriesAsync(studyInstanceUid, seriesInstanceUid);
```

此序列有 2 个实例（green-square 和 red-triangle），因此响应应该返回两个实例。验证响应具有 OK 状态代码并且返回了两个实例。

### 检索序列中所有实例的元数据

此请求检索单个研究中所有实例的元数据。

_详细信息：_

* GET /studies/{study}/series/{series}/metadata

```c#
DicomWebResponse response = await client.RetrieveSeriesMetadataAsync(studyInstanceUid, seriesInstanceUid);
```

此序列有 2 个实例（green-square 和 red-triangle），因此响应应该返回两个实例的元数据。验证响应具有 OK 状态代码并且返回了两个实例的元数据。

### 检索研究的序列中的单个实例

此请求检索单个实例。

_详细信息：_

* GET /studies/{study}/series{series}/instances/{instance}

```c#
DicomWebResponse response = await client.RetrieveInstanceAsync(studyInstanceUid, seriesInstanceUid, sopInstanceUid);
```

这应该只返回实例 red-triangle。验证响应具有 OK 状态代码并且返回了实例。

### 检索研究的序列中的单个实例的元数据

此请求检索单个研究和序列中的单个实例的元数据。

_详细信息：_

* GET /studies/{study}/series/{series}/instances/{instance}/metadata

```c#
DicomWebResponse response = await client.RetrieveInstanceMetadataAsync(studyInstanceUid, seriesInstanceUid, sopInstanceUid);
```

这应该只返回实例 red-triangle 的元数据。验证响应具有 OK 状态代码并且返回了元数据。

### 从单个实例检索一个或多个帧

此请求从单个实例检索一个或多个帧。

_详细信息：_

* GET /studies/{study}/series/{series}/instances/{instance}/frames/{frames}

```c#
DicomWebResponse response = await client.RetrieveFramesAsync(studyInstanceUid, seriesInstanceUid, sopInstanceUid, frames: new[] { 1 });

```

这应该只返回 red-triangle 的唯一帧。验证响应具有 OK 状态代码并且返回了帧。

## 查询 DICOM (QIDO)

> 注意：请参阅[符合性声明](../resources/conformance-statement.md#supported-search-parameters)文件以了解支持的 DICOM 属性。

### 搜索研究

此请求按 DICOM 属性搜索一个或多个研究。

_详细信息：_

* GET /studies?StudyInstanceUID={study}

```c#
string query = $"/studies?StudyInstanceUID={studyInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个研究并且响应代码为 OK。

### 搜索序列

此请求按 DICOM 属性搜索一个或多个序列。

_详细信息：_

* GET /series?SeriesInstanceUID={series}

```c#
string query = $"/series?SeriesInstanceUID={seriesInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个序列并且响应代码为 OK。

### 在研究内搜索序列

此请求按 DICOM 属性在单个研究内搜索一个或多个序列。

_详细信息：_

* GET /studies/{study}/series?SeriesInstanceUID={series}

```c#
string query = $"/studies/{studyInstanceUid}/series?SeriesInstanceUID={seriesInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个序列并且响应代码为 OK。

### 搜索实例

此请求按 DICOM 属性搜索一个或多个实例。

_详细信息：_

* GET /instances?SOPInstanceUID={instance}

```c#
string query = $"/instances?SOPInstanceUID={sopInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个实例并且响应代码为 OK。

### 在研究内搜索实例

此请求按 DICOM 属性在单个研究内搜索一个或多个实例。

_详细信息：_

* GET /studies/{study}/instances?SOPInstanceUID={instance}

```c#
string query = $"/studies/{studyInstanceUid}/instances?SOPInstanceUID={sopInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个实例并且响应代码为 OK。

### 在研究和序列内搜索实例

此请求按 DICOM 属性在单个研究和单个序列内搜索一个或多个实例。

_详细信息：_

* GET /studies/{study}/series/{series}instances?SOPInstanceUID={instance}

```c#
string query = $"/studies/{studyInstanceUid}/series/{seriesInstanceUid}/instances?SOPInstanceUID={sopInstanceUid}";
DicomWebResponse response = await client.QueryAsync(query);
```

验证响应包括 1 个实例并且响应代码为 OK。

## 删除 DICOM

> 注意：删除不是 DICOM 标准的一部分，但已添加以便于使用。

### 删除研究和序列中的特定实例

此请求删除单个研究和单个序列中的单个实例。

_详细信息：_

* DELETE /studies/{study}/series/{series}/instances/{instance}

```c#
string sopInstanceUidRed = "1.2.826.0.1.3680043.8.498.47359123102728459884412887463296905395";
DicomWebResponse response = await client.DeleteInstanceAsync(studyInstanceUid, seriesInstanceUid, sopInstanceUidRed);
```

这从服务器删除 red-triangle 实例。如果成功，响应状态代码不包含内容。

### 删除研究中的特定序列

此请求删除单个研究中的单个序列（以及所有子实例）。

_详细信息：_

* DELETE /studies/{study}/series/{series}

```c#
DicomWebResponse response = await client.DeleteSeriesAsync(studyInstanceUid, seriesInstanceUid);
```

这从服务器删除 green-square 实例（它是序列中唯一剩余的元素）。如果成功，响应状态代码不包含内容。

### 删除特定研究

此请求删除单个研究（以及所有子序列和实例）。

_详细信息：_

* DELETE /studies/{study}

```c#
DicomWebResponse response = await client.DeleteStudyAsync(studyInstanceUid);
```

这从服务器删除 blue-circle 实例（它是序列中唯一剩余的元素）。如果成功，响应状态代码不包含内容。
