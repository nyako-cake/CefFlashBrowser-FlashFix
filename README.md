# CefFlashBrowser-FlashFix

> ⚠️ **这不是上游官方版本。** 本仓库是 [`Mzying2001/CefFlashBrowser`](https://github.com/Mzying2001/CefFlashBrowser) 的个人 fork，仅用于分发一个**修复构建**：解决「站点探测 Flash 失败 → 提示浏览器无法运行 Flash」的问题。上游代码、官方 Release、issue 与讨论，请一律以上游仓库为准。
>
> 名称里的 `-FlashFix` 就是为此加的区分后缀，避免和官方版混淆。

| 项目 | 说明 |
| --- | --- |
| 仓库性质 | 非官方 fork，仅作修复分发（MIT 许可沿用上游） |
| 基线 | 上游 `master` @ `507012a`（2026-10-28 的 Merge PR #204） |
| 修复分支 | [`fix/flash-plugin-power-saver`](https://github.com/nyako-cake/CefFlashBrowser-FlashFix/tree/fix/flash-plugin-power-saver) — 提交 `10505cf` |
| 构建产物 | [Release 1.1.8](https://github.com/nyako-cake/CefFlashBrowser-FlashFix/releases/tag/1.1.8) → `FlashBrowser_x64_v1.1.8.zip`（仅 x64，85.3 MB） |
| 版本号 | 1.1.8（相对官方 1.1.7 递增 0.0.1） |

## 修了什么

**现象**：用官方 1.1.7 打开 4399 的「奥拉星」入口 <https://www.4399.com/flash/35538.htm>（页面里 `//aola.100bt.com/play/play.html` 才是 Flash 客户端），只显示「当前浏览器无法运行 Flash」和下载第三方浏览器的引导页，游戏起不来。

**根因**：不是插件缺失，也不是 CefSharp 不支持 Flash——`pepflashplayer.dll` 已正确注册，普通尺寸的 SWF 能跑（`PercentLoaded=100`）。真正的门槛在站点侧脚本 `//resource.a0bi.com/resource/js/lib/normalize-flash-launch.js`：它插入一个 **1×1 的「心跳」SWF**（`//aola.100bt.com/resource/flash_heart_beat.swf`），要求 3 秒内回调 `window.flashHeartBeat()`，收不到就判定「Flash 不可用」并显示那句提示（提示文案来自脚本内嵌的中文串，不是浏览器报错）。而 Chromium 80+ 的**插件省电策略（Plugin Power Saver）会挂起尺寸过小的 Flash 实例**，1×1 实例从未启动，心跳必然无回调。

**修复**：`CefFlashBrowser/Program.cs` 的 `InitCefFlash()` 里，初始化 CEF 时增加一个启动开关：

```csharp
settings.CefCommandLineArgs["override-plugin-power-saver-for-testing"] = "never";
```

## 与上游的差异

| # | 差异 | 位置 | 是否进了修复分支 |
| --- | --- | --- | --- |
| 1 | 关闭插件省电策略的一行开关 + 3 行英文注释 | `CefFlashBrowser/Program.cs` 的 `InitCefFlash()`（`autoplay-policy` 之后） | 是（+5 行，唯一改动） |
| 2 | 目标框架 `net462` → `net48`（`App.config` 同步 `v4.8`，两个 C++/CLI 工程同步改 `TargetFrameworkVersion`） | 各 `*.csproj` / `*.vcxproj` / `App.config` | 否（仅本机构建环境权宜，分支里不含） |
| 3 | 本仓库改名与 README 头部说明 | 仓库设置 / `README.md` | 否（只影响本 fork 展示） |

其余源码、图标、界面、语言文件、发布脚本均与上游一致，未做改动。

## 修复前后对照（同机、同页面实测）

| 对照项 | 官方 1.1.7 | 本构建 1.1.8 |
| --- | --- | --- |
| 1×1 心跳复现页 | `heartbeat: NOT RECEIVED` | `heartbeat: RECEIVED` |
| 6 种 1×1 嵌入写法（embed / object 混排） | 全部取不到插件 API | 全部 `PercentLoaded=100`、`TotalFrames=1` |
| 尺寸矩阵（1×1、0×0 vs 10×10 以上） | 小尺寸失败、大尺寸正常 | 全部正常 |
| netlog：`flash_heart_beat.swf` 请求数 | 0 | 6 |
| netlog：游戏客户端 `start~2107300001.swf` | 3 | 6 |
| netlog：H5 兜底跳转 `aola.100bt.com/h5` | 无 | 无 |

## 下载与校验

- Release：<https://github.com/nyako-cake/CefFlashBrowser-FlashFix/releases/tag/1.1.8>
- 文件：`FlashBrowser_x64_v1.1.8.zip`（x64 / Release，解压即用，内含 `修复说明.txt`）
- 大小：`89,394,356` 字节
- SHA256：`3d4eecef609b049cf6611b8b1f907d8fec7b6f2532a50352bcb41e297fb41a0e`

> 安装包里的程序本体仍叫 `CefFlashBrowser.exe`（窗口标题、右键菜单、语言文件均未改），仓库名与说明才是本 fork 的区分标识；如需连程序名一起改，可另开一个版本重建。

---

## 以下是上游原始 README 内容（未改动，仅作为来源与文档保留）


<h1 align="center">
    <img src="./Icons/large.svg" />
    <br />
    CefFlashBrowser
    <br />
    <img src="https://img.shields.io/github/license/Mzying2001/CefFlashBrowser" />
    <img src="https://img.shields.io/github/v/release/Mzying2001/CefFlashBrowser?include_prereleases" />
    <img src="https://img.shields.io/github/downloads/Mzying2001/CefFlashBrowser/total" />
</h1>

**中文** | [English](./README.en.md)

![main](./Images/MainWindow.png)

## 软件介绍

CefFlashBrowser 是一个自带 Flash Player 插件的浏览器，可以正常显示网页上的 Flash 内容，此外还支持打开本地 SWF 文件，管理 Flash 游戏存档等。

## 下载与使用

在 [Release](https://github.com/Mzying2001/CefFlashBrowser/releases/latest) 下载最新的软件压缩包，解压后双击 `CefFlashBrowser.exe` 即可使用。

> [!NOTE]
> 软件运行依赖 `.NET Framework 4.6.2`，若运行时提示应用程序无法启动，前往[这里](https://dotnet.microsoft.com/zh-cn/download/dotnet-framework/net462)下载安装即可运行。
<!---->
> [!NOTE]
> 若打开软件时提示 `System.IO.FileNotFoundException` 异常，一般是由于缺少 `Microsoft Visual C++ Redistributable` 导致的，前往[这里](https://learn.microsoft.com/zh-cn/cpp/windows/latest-supported-vc-redist#latest-microsoft-visual-c-redistributable-version)下载安装后即可正常打开。

## 存档管理

使用**SOL存档管理器**，可以对游戏存档进行导出、导入、修改等操作。

> [!TIP]
> 建议在修改存档之前先对存档文件进行备份，以防数据丢失。

![solmgr](./Images/SolSaveManager.png)
![soledit](./Images/SolEditor.png)

## Star 趋势

<a href="https://star-history.dera.page/Mzying2001/CefFlashBrowser?style=github">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&theme=dark&style=github" />
   <source media="(prefers-color-scheme: light)" srcset="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&style=github" />
   <img alt="Star History Chart" src="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&style=github" />
 </picture>
</a>
