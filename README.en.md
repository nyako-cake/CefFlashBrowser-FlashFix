# CefFlashBrowser-FlashFix

> ⚠️ **This is not the official release.** This repository is a personal fork of [`Mzying2001/CefFlashBrowser`](https://github.com/Mzying2001/CefFlashBrowser) that only ships a **fixed build**. For upstream code, official releases, issues and discussions, please use the upstream repository. The `-FlashFix` suffix exists purely to tell this fork apart from the official project.

| Item | Value |
| --- | --- |
| Repository | Unofficial fork for fix distribution (MIT, inherited) |
| Base | upstream `master` @ `507012a` (2026-10-28) |
| Fix branch | [`fix/flash-plugin-power-saver`](https://github.com/nyako-cake/CefFlashBrowser-FlashFix/tree/fix/flash-plugin-power-saver) — commit `10505cf` |
| Build | [Release 1.1.8](https://github.com/nyako-cake/CefFlashBrowser-FlashFix/releases/tag/1.1.8) → `FlashBrowser_x64_v1.1.8.zip` (x64 only, 85.3 MB) |
| Version | 1.1.8 (official is 1.1.7) |

## What was fixed

Pages such as 4399's Aola entry (<https://www.4399.com/flash/35538.htm> → `//aola.100bt.com/play/play.html`) show "this browser cannot run Flash" even though the PPAPI Flash plugin is registered and normal-sized SWFs execute fine.

The site's `normalize-flash-launch.js` inserts a **1×1 heartbeat SWF** and waits for a JS callback within 3 seconds; Chromium 80+'s **plugin power saver** suspends Flash instances that small, so the instance never starts, the callback never fires, and the page concludes Flash is unusable.

The fix is one CEF command-line switch in `CefFlashBrowser/Program.cs::InitCefFlash()`:

```csharp
settings.CefCommandLineArgs["override-plugin-power-saver-for-testing"] = "never";
```

## Differences from upstream

1. **Code (in the fix branch, +5 lines)**: the switch above, plus three comment lines in `InitCefFlash()`.
2. **Build-only (NOT in the fix branch)**: target framework `net462` → `net48` in the project files and `App.config`, needed because the build machine lacked the .NET 4.6.2 targeting pack.
3. **Repository presentation**: the fork name and this README header.

Everything else — icons, UI, language files, publishing scripts — is unchanged from upstream.

## Verification (same machine, same page)

| Check | Official 1.1.7 | This 1.1.8 build |
| --- | --- | --- |
| 1×1 heartbeat probe page | `heartbeat: NOT RECEIVED` | `heartbeat: RECEIVED` |
| Six 1×1 embed/object variants | no plugin API at all | all `PercentLoaded=100` |
| `flash_heart_beat.swf` requests (netlog) | 0 | 6 |
| Game client `start~2107300001.swf` | 3 | 6 |

## Download

- Release: <https://github.com/nyako-cake/CefFlashBrowser-FlashFix/releases/tag/1.1.8>
- File: `FlashBrowser_x64_v1.1.8.zip`, `89,394,356` bytes
- SHA256: `3d4eecef609b049cf6611b8b1f907d8fec7b6f2532a50352bcb41e297fb41a0e`

> The executable inside is still named `CefFlashBrowser.exe`; only the repository name and documentation are rebranded for this fork.

---

## Upstream README (unchanged, kept for provenance)


<h1 align="center">
    <img src="./Icons/large.svg" />
    <br />
    CefFlashBrowser
    <br />
    <img src="https://img.shields.io/github/license/Mzying2001/CefFlashBrowser" />
    <img src="https://img.shields.io/github/v/release/Mzying2001/CefFlashBrowser?include_prereleases" />
    <img src="https://img.shields.io/github/downloads/Mzying2001/CefFlashBrowser/total" />
</h1>

[中文](./README.md) | **English**

![main](./Images/MainWindow.png)

## About

CefFlashBrowser is a browser that comes with a built-in Flash Player plugin, allowing it to properly display Flash content on web pages. In addition, it supports opening local SWF files, managing Flash game save data, and more.

## Download & Usage

Download the latest release from the [Release page](https://github.com/Mzying2001/CefFlashBrowser/releases/latest). Extract the archive and double-click `CefFlashBrowser.exe` to run.

> [!NOTE]
> The application requires `.NET Framework 4.6.2`. If you see an error saying the application can’t start, you can download and install it from [here](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net462).
<!---->
> [!NOTE]
> If you encounter a `System.IO.FileNotFoundException` on startup, it’s usually because the `Microsoft Visual C++ Redistributable` is missing, you can download and install it from [here](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist#latest-microsoft-visual-c-redistributable-version).

## Save Manager

With the **SOL Save Manager**, you can export, import, and edit Flash game save files.

> [!TIP]
> It’s recommended to back up your save files before making changes to avoid data loss.

![solmgr](./Images/SolSaveManager.png)
![soledit](./Images/SolEditor.png)

## Stargazers over time

<a href="https://star-history.dera.page/Mzying2001/CefFlashBrowser?style=github">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&theme=dark&style=github" />
   <source media="(prefers-color-scheme: light)" srcset="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&style=github" />
   <img alt="Star History Chart" src="https://star-history.dera.page/svg?repos=Mzying2001/CefFlashBrowser&style=github" />
 </picture>
</a>
