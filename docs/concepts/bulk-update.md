# 批量更新概述

批量更新是一项功能，可以在不需要删除和重新添加的情况下更新 DICOM 属性/元数据。当前支持的属性包括[患者识别模块](https://dicom.nema.org/dicom/2013/output/chtml/part03/sect_C.2.html#table_C.2-2)、[患者人口统计模块](https://dicom.nema.org/dicom/2013/output/chtml/part03/sect_C.2.html#table_C.2-3)和[通用研究模块](https://dicom.nema.org/medical/dicom/2020b/output/chtml/part03/sect_C.7.2.html#table_C.7-3)中不是序列的属性，如下所列。

**患者识别模块**
| 属性名称   | 标签           | 描述           |
| ---------------- | --------------| --------------------- |
| Patient's Name   | (0010,0010)   | 患者全名   |
| Patient ID       | (0010,0020)   | 患者的主要医院识别号或代码。 |
| Other Patient IDs| (0010,1000) | 用于识别患者的其他识别号或代码。 
| Type of Patient ID| (0010,0022) |  此项目中的标识符类型。枚举值：TEXT RFID BARCODE 注意：无论类型如何，标识符都编码为字符串，而不是二进制值。 
| Other Patient Names| (0010,1001) | 用于识别患者的其他名称。 
| Patient's Birth Name| (0010,1005) | 患者出生时的姓名。 
| Patient's Mother's Birth Name| (0010,1060) | 患者母亲的出生姓名。 
| Medical Record Locator | (0010,1090) | 用于查找患者现有医疗记录（例如，胶片夹）的标识符。 
| Issuer of Patient ID | (0010,0021) | 颁发患者 ID 的分配机构（系统、组织、机构或部门）的标识符。 

**患者人口统计模块**
| 属性名称   | 标签           | 描述           |
| ---------------- | --------------| --------------------- |
| Patient's Age | (0010,1010) | 患者年龄。  |
| Occupation | (0010,2180) | 患者职业。  |
| Confidentiality Constraint on Patient Data Description | (0040,3001) | 对模态操作员关于患者信息保密性的特殊指示（例如，在其他患者在场时不应使用患者姓名）。  |
| Patient's Birth Date | (0010,0030) | 指定患者的出生日期  |
| Patient's Birth Time | (0010,0032) | 指定患者的出生时间  |
| Patient's Sex | (0010,0040) | 指定患者的性别。  |
| Quality Control Subject |(0010,0200) | 指示受试者是否为质量控制体模。  |
| Patient's Size | (0010,1020) | 患者的身高或长度（米）  |
| Patient's Weight | (0010,1030) | 患者体重（千克）  |
| Patient's Address | (0010,1040) | 指定患者的法定地址  |
| Military Rank | (0010,1080) | 患者军衔  |
| Branch of Service | (0010,1081) | 军种。也可能包括国家忠诚度（例如，美国陆军）。  |
| Country of Residence | (0010,2150) | 患者当前居住的国家  |
| Region of Residence | (0010,2152) | 患者居住国家内的地区  |
| Patient's Telephone Numbers | (0010,2154) | 可以联系到患者的电话号码  |
| Ethnic Group | (0010,2160) | 患者的种族或民族  |
| Patient's Religious Preference | (0010,21F0) | 患者的宗教信仰偏好  |
| Patient Comments | (0010,4000) | 关于患者的用户定义注释 | 
| Responsible Person | (0010,2297) | 对患者具有医疗决策权的人员姓名。  |
| Responsible Person Role | (0010,2298) | 负责人与患者的关系。  |
| Responsible Organization | (0010,2299) | 对患者具有医疗决策权的组织名称。  |
| Patient Species Description | (0010,2201) | 患者的物种。  |
| Patient Breed Description | (0010,2292) | 患者的品种。参见第 C.7.1.1.1.1 节。  |
| Breed Registration Number | (0010,2295) | 注册表中兽医患者的识别号。  |

**通用研究模块**
| 属性名称   | 标签           | 描述           |
| ---------------- | --------------| --------------------- |
| Referring Physician's Name | (0008,0090)   | 患者转诊医生的姓名   |
| Accession Number | (0008,0050)   | RIS 生成的用于识别研究订单的编号。 |
| Study Description | (0008,1030) | 机构生成的对所执行研究（组件）的描述或分类。 

研究更新后，可以检索两个版本的实例：原始的未修改实例和具有更新属性的最新版本。中间版本不会持久化。

## API 设计

以下 URI 假设隐式 DICOM 服务基础 URI。例如，本地运行的 DICOM 服务器的基础 URI 将是 `https://localhost:63838/`。
可以在 [Postman 集合](../resources/Conformance-as-Postman.postman_collection.json) 中发送示例请求。

### 批量更新研究

批量更新端点启动长时间运行的操作，使用指定的属性更新研究中的所有实例。

```http
POST ...v2/studies/$bulkUpdate
POST ...v2/partitions/{PartitionName}/studies/$bulkUpdate
```

#### 请求头

| 名称         | 必需  | 类型   | 描述                     |
| ------------ | --------- | ------ | ------------------------------- |
| Content-Type | False     | string | 支持 `application/json` |

#### 请求体

下面的 `UpdateSpecification` 作为请求体传递。`UpdateSpecification` 需要同时指定 `studyInstanceUids` 和 `changeDataset`。

```json
{
   "studyInstanceUids": ["1.113654.3.13.1026"],
    "changeDataset": { 
        "00100010": { 
            "vr": "PN", 
            "Value": 
            [
                { 
                    "Alphabetic": "New Patient Name 1" 
                }
            ] 
        } 
    }
}
```

#### 响应

成功启动批量更新操作后，批量更新 API 返回 `202` 状态代码。响应体包含对操作的引用。

```http
HTTP/1.1 202 Accepted
Content-Type: application/json
{
    "id": "1323c079a1b64efcb8943ef7707b5438",
    "href": "../v2/operations/1323c079a1b64efcb8943ef7707b5438"
}
```

| 名称              | 类型                                        | 描述                                                  |
| ----------------- | ------------------------------------------- | ------------------------------------------------------------ |
| 202 (Accepted)    | [Operation Reference](#operation-reference) | 已启动长时间运行的操作以更新 DICOM 属性 |
| 400 (Bad Request) |                                             | 请求体包含无效数据                                |

### 操作状态

可以轮询上述 `href` URL 以获取导出操作的当前状态，直到完成。终端状态由 `200` 状态而不是 `202` 表示。

```http
GET .../operations/{operationId}
```

#### URI 参数

| 名称        | 位置   | 必需 | 类型   | 描述      |
| ----------- | ---- | -------- | ------ | ---------------- |
| operationId | path | True     | string | 操作 ID |

#### 响应

**成功响应**

```json
{
    "operationId": "1323c079a1b64efcb8943ef7707b5438",
    "type": "update",
    "createdTime": "2023-05-08T05:01:30.1441374Z",
    "lastUpdatedTime": "2023-05-08T05:01:42.9067335Z",
    "status": "completed",
    "percentComplete": 100,
    "results": {
        "studyUpdated": 1,
        "instanceUpdated": 16
    }
}
```

**失败响应**
```
{
    "operationId": "1323c079a1b64efcb8943ef7707b5438",
    "type": "update",
    "createdTime": "2023-05-08T05:01:30.1441374Z",
    "lastUpdatedTime": "2023-05-08T05:01:42.9067335Z",
    "status": "failed",
    "percentComplete": 100,
    "results": {
        "studyUpdated": 0,
        "studyFailed": 1,
        "instanceUpdated": 0,
        "errors": [
            "Failed to update instances for study 1.113654.3.13.1026"
        ]
    }
}
```

如果有任何特定于实例的异常，它将被添加到 `errors` 列表中。它将包括实例的所有 UID，例如
`Instance UIDs - PartitionKey: 1, StudyInstanceUID: 1.113654.3.13.1026, SeriesInstanceUID: 1.113654.3.13.1035, SOPInstanceUID: 1.113654.3.13.1510`

| 名称            | 类型                    | 描述                                  |
| --------------- | ----------------------- | -------------------------------------------- |
| 200 (OK)        | [Operation](#operation) | 具有指定 ID 的操作已完成 |
| 202 (Accepted)  | [Operation](#operation) | 具有指定 ID 的操作正在运行 |
| 404 (Not Found) |                         | 未找到操作                   |

## 检索 (WADO-RS)

检索事务功能提供检索存储的研究、序列和实例的能力，包括原始版本和最新版本。

> 注意：下面列出了用于检索实例和元数据的支持端点。

| 方法 | 路径                                                                    | 描述 |
| :----- | :---------------------------------------------------------------------- | :---------- |
| GET    | ../studies/{study}                                                      | 检索研究中的所有实例。 |
| GET    | ../studies/{study}/metadata                                             | 检索研究内所有实例的元数据。 |
| GET    | ../studies/{study}/series/{series}                                      | 检索序列中的所有实例。 |
| GET    | ../studies/{study}/series/{series}/metadata                             | 检索序列内所有实例的元数据。 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}                 | 检索单个实例。 |
| GET    | ../studies/{study}/series/{series}/instances/{instance}/metadata        | 检索单个实例的元数据。 |

为了检索原始版本，应将 `msdicom-request-original` 标头设置为 `true`。

下面显示了检索实例原始版本的示例请求。

```http 
GET .../studies/{study}/series/{series}/instances/{instance}
Accept: multipart/related; type="application/dicom"; transfer-syntax=*
msdicom-request-original: true
Content-Type: application/dicom
 ```
## 删除

此事务将删除实例的原始版本和最新版本。

> 注意：删除事务后，已删除的实例将无法恢复。

## Change Feed

除了其他操作（创建和删除）之外，每次更新操作都会填充更新的变更源操作。

变更源操作的请求和响应示例可以在[这里](./change-feed.md)找到。

### 其他 API

其他 API 没有变化。所有其他 API 仅支持实例的最新版本。

### DICOM 文件中的更改

作为批量更新的一部分，仅更新 DICOM 元数据。不更新像素数据。像素数据将与原始版本相同。

除了更新元数据之外，DICOM 文件的文件元信息会使用以下信息进行更新。

| 标签           | 属性名称        | 描述           | 值
| --------------| --------------------- | --------------------- | --------------|
| (0002,0012)   | Implementation Class UID | 唯一标识写入此文件及其内容的实现。 | 1.3.6.1.4.1.311.129 |
| (0002,0013)   | Implementation Version Name | 标识实现类 UID (0002,0012) 的版本 | DICOM 服务的程序集版本（例如 0.1.4785） |

这里，UID `1.3.6.1.4.1.311.129` 在 IANA 的 [Microsoft OID arc](https://oidref.com/1.3.6.1.4.1.311) 下注册。

#### 限制

> 批量更新仅支持患者识别和人口统计属性。

> 一次最多可以更新 50 个研究。

> 一次只能执行一个更新操作。

> 无法仅删除最新版本或恢复到原始版本。

> 我们不支持将任何字段从非空值更新为空值。
