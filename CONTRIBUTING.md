# 参与贡献指南（Medical Imaging Server for DICOM）

本文档说明如何为 Medical Imaging Server for DICOM 仓库贡献代码。

## 提交 Pull Request

- **请务必** 通过 PR 提交变更，至少一名团队成员会进行同行评审并决定合入。
- **请务必** 为 PR 取简洁且有描述性的标题。
- **请务必** 撰写简要且有价值的 PR 描述。
- **请务必** 关联相关 issue，并使用可自动关闭 issue 的[关键词](https://help.github.com/articles/closing-issues-using-keywords/)。
- **请务必** 确保每个提交都能成功构建，整个 PR 需通过全部检查才会被合并。
- **请务必** 用额外提交响应评审意见，而非改写已有提交。
- **默认** 使用 [Squash and Merge](https://blog.github.com/2016-04-01-squash-your-commits/) 合并，除非另有要求。
- **请勿** 提交“进行中”PR，仅在准备好评审时提交。
- **请勿** 让 PR 超过 4 周无提交地保持打开，陈旧 PR 会被关闭，待恢复开发再重新提交。
- **请勿** 在同一个 PR 混入无关的独立变更。

## 代码风格

代码风格由 [.NET 分析器](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/overview) 和 [.editorconfig](.editorconfig) 强制约束，提交前请遵循：

- **请务必** 处理 .NET 分析器报错。
- **请务必** 遵循 [.editorconfig](.editorconfig) 设置。

## 创建 Issue

- **请务必** 使用能清晰识别问题或需求的标题。
- **请务必** 撰写详细的问题或需求描述。
- **请务必** 提供以下细节：
  - 期望行为与实际行为。
  - 相关的异常信息或 OperationOutcome。
- **请务必** 订阅所创建 issue 的通知，便于跟进交流。
