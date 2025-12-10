# 为启用 Private Link 的 DICOM 配置 DICOM Cast

1. 在计划创建 Health Data Services Workspace 的同一订阅和区域中创建虚拟网络，包含两个子网：
    1. 默认子网
    2. 委派给 Microsoft.containerinstance/containergroups 的子网  
![Alt text](image.png)

2. 在同一区域部署 Health Data Services workspace、DICOM 与 FHIR。
3. 为 Health Data Service Workspace 启用 Private Link，使用步骤 1 创建的虚拟网络及默认子网。  
![Alt text](image-1.png)

4. 使用[此模板](DicomcastDeploymentTemplate.md)在步骤 1 的虚拟网络中部署 DICOM Cast，使用委派给 Microsoft.containerinstance/containergroups 的子网（见步骤 1 图示）。
5. 为容器实例的系统分配托管身份在 Health Data Service Workspace 中添加角色：
    1. DICOM Data Owner
    2. FHIR Data Contributor  
![Alt text](image-2.png)

6. DICOM Cast 处理 Change Feed 时需要访问存储账户的表存储。为步骤 4 部署时创建的存储账户启用 Private Link，该 Private Link 也应位于步骤 1 创建的同一 VNet 和默认子网。  
![Alt text](image-3.png)

7. 关闭存储账户的公共网络访问（`Security + Networking` > `Networking` > `Firewalls and virtual networks` > `Public Network Access` > `Disabled`）。

8. 确保步骤 4 中创建的 Key Vault 已填充此处要求的所有字段：https://github.com/microsoft/dicom-server/blob/main/docs/how-to-guides/sync-dicom-metadata-to-fhir.md#update-key-vault-for-dicom-cast

9. 重启 DICOM Cast 容器，确认正常运行。 
