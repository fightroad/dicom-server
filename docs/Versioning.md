# DICOM 服务器的语义版本控制

本指南概述了此项目中使用的语义版本控制实现。
为了实现一致且可靠的语义版本控制，使用了 [GitVersion](https://github.com/GitTools/GitVersion) 库。

## Git Version

### 概述
GitVersion 是一个软件库和构建任务，使用 Git 历史记录来计算应用于当前构建的版本。以下部分说明如何配置它以及可用于协助版本控制的命令。

### 设置
根目录中包含一个[配置文件](https://github.com/microsoft/dicom-server/blob/master/GitVersion.yml)，用于设置版本策略并指定应如何针对默认分支和其他分支计算版本。目前，对 main 的所有提交都将被视为发布，对其他分支（包括拉取请求）的所有提交都将被视为预发布（例如 `1.2.0-my-branch+1`）。

配置的 GitVersion 版本控制策略是[主线开发](https://gitversion.net/docs/reference/versioning-modes/mainline-development)，它会在每次提交到 main 分支时递增补丁版本。我们当前的开发工作流假设 main 分支将在每次提交时暂存发布，但某些发布不会被批准。

当发布被批准时，应该将资产发布到 nuget 源，并针对代码创建标签以标记发布。

### 命令
在压缩合并期间可以使用多个命令来允许递增主版本/次版本号。

对于主要功能或主要破坏性变更，可以将以下命令添加到提交消息中：
```
+semver: breaking
或
+semver: major
```

较小的更改可以选择递增次版本：
```
+semver: feature
或
+semver: minor
```

对于错误修复或其他增量更改，无需添加任何内容，这将自动发生。

## 何时递增版本的示例

| 操作  | 命令  |
|---|---|
| 更新了次要 nuget 包版本  | 无 :beach_umbrella: |
| 修复了错误  | 无 :beach_umbrella: |
| 小的向后兼容更改  | 无 :beach_umbrella: :tropical_drink: |
| 更新了文档  | 无或 `+semver: skip` |
| 添加新功能/组件/库 | `+semver: feature` :bowtie: |
| 主要产品级别更改 | `+semver: major` |
| 不兼容的二进制更改 | `+semver: major` :boom: |
<br />

:exclamation: 注意：程序集版本使用主版本和静态次版本和补丁版本（例如 {major}.0.0），因此使用 `+semver: major` 将强制下游应用程序重新编译。递增次版本或补丁版本将保持生成的程序集二进制兼容。
