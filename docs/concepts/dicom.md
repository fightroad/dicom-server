# Medical Imaging Server for DICOM 概述

## 医学影像

医学影像是为临床分析和医疗干预创建身体内部视觉表示的技术和过程，以及某些器官或组织功能的视觉表示（生理学）。医学影像旨在揭示被皮肤和骨骼隐藏的内部结构，以及诊断和治疗疾病。医学影像还建立了正常解剖学和生理学的数据库，以便能够识别异常。尽管可以对移除的器官和组织进行影像检查以用于医疗目的，但此类程序通常被认为是病理学的一部分，而不是医学影像。[Wikipedia, 2020](https://en.wikipedia.org/wiki/Medical_imaging)

## DICOM

DICOM（医学数字成像和通信）是传输、存储、检索、打印、处理和显示医学影像信息的国际标准，是医疗保健领域接受的主要医学影像标准。尽管存在一些例外（牙科、兽医），但几乎所有医学专业、设备制造商、软件供应商和个人从业者在涉及影像的任何医疗工作流程的某个阶段都依赖 DICOM。DICOM 确保医学影像符合质量标准，从而可以保持诊断的准确性。大多数影像模式，包括 CT、MRI 和超声，都必须符合 DICOM 标准。DICOM 格式的影像需要通过专门的 DICOM 应用程序访问和使用。

## Medical Imaging Server for DICOM

Medical Imaging Server for DICOM 是一个开源的 DICOM 服务器，可轻松部署到 Azure。Medical Imaging Server for DICOM 将 DICOM 元数据注入到 [Azure API for FHIR 服务](https://docs.microsoft.com/azure/healthcare-apis/)，允许临床数据和影像元数据使用单一数据源。它允许与任何支持 DICOMweb&trade; 的系统进行基于标准的通信。

有效集成非临床数据的需求变得迫切。为了有效治疗患者、研究新的治疗方法或诊断解决方案，或者简单地提供单个患者健康历史的有效概述，组织必须整合来自多个来源的数据。最紧迫的集成之一是在临床数据和影像数据之间。

FHIR&trade; 正在成为临床数据的重要标准，并提供可扩展性以支持直接或通过引用集成其他类型的数据。通过使用 Medical Imaging Server for DICOM，组织可以在 FHIR&trade; 中存储对影像数据的引用，并启用跨临床和影像数据集的查询。这可以实现许多不同的场景，例如：

- **为研究创建队列。** 通常通过查询匹配临床和影像系统中数据的患者，例如这个查询（它触发了集成 FHIR&trade; 和 DICOM 数据的工作）：“给我所有在过去 2 年中年龄超过 45 岁且被诊断为骨肉瘤的患者的处方药物、所有 CT 扫描文档及其相关的放射学报告。”
- **查找类似患者的结果以了解选项并规划治疗。** 当面对患者诊断时，医生可以识别过去具有类似诊断的患者结果和治疗计划，即使这些包括影像数据。
- **在诊断过程中提供患者的纵向视图。** 放射科医生，特别是远程放射科医生，通常无法完全访问患者的医疗历史和相关的影像研究。通过 FHIR&trade; 集成，可以轻松提供这些数据，甚至提供给组织本地网络之外的放射科医生。
- **与远程放射科医生建立反馈循环。** 理想情况下，放射科医生可以访问医院的临床数据，以便在提出建议后建立反馈循环。然而，对于远程放射科医生来说，情况往往并非如此。相反，他们通常在执行诊断后无法建立反馈循环，因为他们在初始读取后无法访问患者数据。由于无法（或有限）访问临床结果或结果，他们无法获得提高技能所需的反馈。正如一位远程放射科医生所说：“以甲状旁腺为例。我们做的比全国任何其他诊所都多，但我必须恳求外科医生告诉我他们实际发现了什么。在我每月做的 500 多项研究中，我只得到三到四项的直接反馈。” 通过与 FHIR&trade; 集成，组织可以轻松创建一个工具，为远程放射科医生提供直接反馈，帮助他们磨练技能并在未来做出更好的建议。
- **为 AI/ML 模型建立反馈循环。** 当可以使用真实世界的反馈来改进模型时，机器学习模型表现最佳。然而，第三方 ML 模型提供商很少获得他们需要的反馈来随着时间的推移改进他们的模型。例如，一位 ISV 这样说：“我们使用机器学习模型和人类专家的组合来推荐心脏手术的治疗计划。然而，我们很少从医生那里获得关于我们的计划有多准确的反馈。例如，我们经常推荐支架尺寸。我们很想知道我们的预测是否正确，但我们只在我们的建议出现重大问题时才听到客户的声音。” 与远程放射科医生的反馈一样，与 FHIR&trade; 的集成允许组织创建一个机制，为模型重新训练管道提供反馈。

## 将 Medical Imaging Server for DICOM 部署到 Azure

Medical Imaging Server for DICOM 需要 Azure 订阅来配置和运行所需的组件。默认情况下，这些组件在现有或新的 Azure 资源组中创建，以简化管理。此外，还需要 Azure Active Directory 帐户。下图描述了在资源组中创建的所有资源。

![资源部署](../images/dicom-deployment-architecture.png)

- **Azure SQL**：索引 Medical Imaging Server for DICOM 元数据的子集，以支持查询并维护可查询的变更日志。
- **App Service Plan**：托管 Medical Imaging Server for DICOM。
- **Azure Key Vault**：存储关键安全信息。
- **存储帐户**：Blob 存储，持久化所有 Medical Imaging Server for DICOM 数据和元数据。
- **Application Insights**（可选）：监控 Medical Imaging Server for DICOM 的性能。
- **Azure Container Instance**（可选）：托管用于 Azure API for FHIR 集成的 DICOM Cast 服务。
- **Azure API for FHIR**（可选）：与其他临床数据一起持久化 DICOM 元数据。

## 总结

本概念文档提供了 DICOM、医学影像和 Medical Imaging Server for DICOM 的概述。要开始使用 Medical Imaging Server：

- [将 Medical Imaging Server 部署到 Azure](../quickstarts/deploy-via-azure.md)
- [部署 DICOM Cast](../quickstarts/deploy-dicom-cast.md)
- [使用 Medical Imaging Server for DICOM API](../tutorials/use-the-medical-imaging-server-apis.md)
