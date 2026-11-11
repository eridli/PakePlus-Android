# 资产盘点助手（Android 安卓 APP）

一个轻量的资产盘点工具，采用 **Web 混合方案**（HTML5 + 摄像头扫码），无需安装任何依赖，单文件即可运行，后续可打包为正式 APK。

## 功能清单

| 功能 | 说明 |
|---|---|
| 导入盘点列表 | 支持 CSV/TXT/XLS/XLSX。**表头必须包含 SN 列**，可选列：固资号、资产类型、PN、位置 等。**在微信中打开本页面时，可直接从聊天文件中选取表格**；系统文件选择器中也可进入「微信文件」目录选取 |
| 免导入扫码采集 | 不导入列表也能直接扫码：扫到的编号自动按 SN 采集为待盘点资产，之后可在「资产列表」页点击补充固资号/类型/PN/位置 |
| 扫码盘点 | 摄像头扫描一维码/二维码，自动匹配列表中的 SN、固资号或 PN |
| 手动输入盘点 | 盘点页提供文本输入框，输入资产编号或 SN 回车确认，与扫码共用同一套匹配/拦截/采集逻辑（导出时标记为"手动"） |
| 显示资产信息 | 匹配后显示资产类型、位置、SN，供与实物核对 |
| 拒绝重复编号 | 可开关的拦截按钮：已盘点的资产再次扫码会被拦截提示 |
| 手动确认 / 自动通过 | 盘点页可开关（默认开启手动确认）。**关闭「手动确认」后，扫码/输入匹配到列表内待盘点资产时自动记「通过」**，结果卡可一键撤销改回手动核对；新采集资产和重复拦截仍按原逻辑处理 |
| 顺序引导 | 每盘完一条，自动按列表顺序显示下一条资产的位置信息和 SN |
| 手动核对 | 条码损坏时，可按编号/SN/位置搜索资产，手动标记「通过 / 不通过」 |
| 编辑与重置 | 资产列表页可点击任意资产编辑信息，已盘点的可一键重置 |
| 导出盘点表 | 导出 CSV（带 BOM，Excel 打开不乱码），含盘点状态、盘点方式、盘点时间 |

## 手机上使用（最快方式）

1. 把 `index.html` 传到安卓手机（微信/数据线/网盘均可）
2. 用 **Chrome 浏览器**打开该文件
3. 首次扫码时允许摄像头权限即可

> 摄像头扫码需要安全上下文：`file://` 打开时部分手机浏览器可能限制摄像头。
> 如遇此情况，推荐任选一种：
> - 把 `index.html` 放到任意 HTTPS 静态托管（GitHub Pages / 内网 nginx 等），手机访问网址使用
> - 直接按下文打包成 APK

## 扫码引擎说明

应用内置双引擎，自动适配运行环境：

| 环境 | 引擎 | 说明 |
|---|---|---|
| Chrome 手机浏览器 | 系统 BarcodeDetector | 优先使用，速度快 |
| APK / WebView | 内置 ZXing 解码库 | 从 `libs/zxing-browser.min.js` 本地加载，**离线可用**；若 8 秒无识别结果也会自动切换到该引擎兜底 |

支持的码制：QR 二维码、Code128、Code39、EAN-13/8、UPC、ITF、Codabar、DataMatrix、PDF417、Aztec 等常见一维码 / 二维码。

**识别度优化**（2026-10）：
- ZXing 引擎启用 TRY_HARDER 深度识别，并限定常见码制（减少误判、提速）
- **双模式可切换**（扫码全屏页底部「⚡ 极速」按钮，设置自动记住）：
  - **稳定模式（默认）**：全帧解码，兼容性最好——即最早两台手机均可识别的路径 + TRY_HARDER 增强
  - **极速模式**：只解码取景框区域（≤1400px），速度提升数倍；6 秒无结果自动切全帧兜底
- 相机优先请求 1920×1080（失败自动降级 720p），条码细节更清晰
- **点「扫码盘点」弹出全屏取景**，带扫描线动画，识别到自动返回主页核对结果
- 扫描中提供 **🔦 补光**（暗处扫码）和 **＋/－ 变焦**（远距离小条码拉近）按钮，按手机硬件能力自动显示
- BarcodeDetector 10 秒无结果自动切换 ZXing（WebView 兼容看门狗）

**打包 APK 时必须把整个文件夹打包（含 `libs/` 子目录）**，否则解码库缺失会导致扫码无反应。

## 打包成正式 APK

### 方案 A：Capacitor（推荐，官方维护、免费无限制）

前置：安装 [Android Studio](https://developer.android.com/studio)（自带 JDK 与 Android SDK）+ Node.js。

```bat
:: 1. 创建打包工程（在项目根目录执行）
mkdir capacitor-build && cd capacitor-build
npm init -y
npm install @capacitor/core @capacitor/cli @capacitor/android
mkdir www
xcopy /E /I ..\asset-inventory-app www

:: 2. 创建 capacitor.config.json（见下），然后：
npx cap add android
npx cap sync
```

`capacitor.config.json`：

```json
{
  "appId": "com.panpan.assetinventory",
  "appName": "资产盘点助手",
  "webDir": "www"
}
```

**加摄像头权限**：编辑 `android/app/src/main/AndroidManifest.xml`，在 `<application>` 外加：

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
```

**编译**：`npx cap open android` → Android Studio 中 Build → Build APK(s)，
产物在 `android/app/build/outputs/apk/debug/app-debug.apk`。

**以后更新**：把 `asset-inventory-app` 内容重新复制到 `www` → `npx cap sync` → 重新 Build。

### 方案 B：在线打包工具

将 `index.html` 上传至 websitetoapk、Applist 等在线 Web→APK 服务，
勾选「摄像头权限」即可生成安装包。

**界面优化**（2026-10）：
- 「📷 扫码盘点 / 手动核对」按钮固定在底部操作栏，拇指易触达
- 「🚫 拒绝重复编号」「✋ 手动确认」两个开关收进可折叠的「⚙️ 盘点设置」，默认收起，状态记忆
- 全局间距收紧、信息密度提高，主页一屏尽量显示完整，减少上下滚动
- 光标默认停留在手动输入框，开箱即可输入（点扫码盘点时自动收起键盘）

## 数据说明

- 所有数据保存在手机本地（localStorage），**不联网、不上传**，离线可用（仅解析 XLS/XLSX 时首次需联网加载一次解析组件）
- 「数据」页可载入 8 条示例数据快速体验流程
- Excel 模板列头建议：`固资号, 资产类型, PN, 位置, SN`（SN 为必填）
