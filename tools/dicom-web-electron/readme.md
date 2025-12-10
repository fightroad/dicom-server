# DICOM Web Electron

Electron 应用，用于与 Medical Imaging Server for DICOM 交互，可上传单个或多个 `.dcm` 文件到指定服务器。

## 前置条件

首次使用需安装最新版 Node.js，安装包下载：[链接](https://nodejs.org/en/download/)。

## 快速开始

在终端进入本工具目录，运行：

1. `npm install`
2. `npm start`

启动后进入 Server Settings，点击 “Change” 更新 DICOM 服务器地址。

**注意**：若通过 Azure 部署，可在 App Service 的 Overview 页面找到 URL。

![Electron tool settings](images/electron-tool-settings.png)

**注意**：若启用了认证，可用 Azure CLI 获取 Bearer Token，参考[此文](https://docs.microsoft.com/en-us/azure/healthcare-apis/get-healthcare-apis-access-token-cli)。

## 上传 DICOM 文件

点击 “Select File(s)” 选择本地文件，可一次上传多个 DICOM 文件。

示例 DICOM 文件见 [../../docs/dcms](../../docs/dcms)

![Electron tool upload](images/electron-tool-upload.png)

## Change Feed

Change Feed 可遍历 DICOM 服务的历史并处理创建/删除事件，详见[文档](https://github.com/microsoft/dicom-server/blob/main/docs/users/ChangeFeed.md)。

在 Change Feed 页面可设置 Offset（跳过的记录数）。

## 打包

应用使用 [`electron-builder`](https://www.electron.build/) 打包。

打包 Windows 版本运行：

```.\node_modules\.bin\electron-builder -w```
