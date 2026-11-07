# aseprite-builder（便携版 / 已修正）

从 Aseprite 官方源码编译**你自己的** Windows 便携版，只需一个 GitHub Actions 免费 runner，不用在本地装 Visual Studio。

## 怎么用

1. 把本仓库 init 到你自己的 GitHub 账号下（新建仓库后上传，或 `git push`）
2. 进 **Actions** → 左侧 `Build Aseprite (Windows 便携版)` → **Run workflow**
   - 版本留空 = 取最新，也可填 `v1.3.18.6`
3. 等 25–40 分钟，在 run 页面底部 **Artifacts** 下载 `Aseprite-vX.Y.Z-Windows-x64.zip`

首次运行前请确认 **Settings → Actions → General → Workflow permissions** 已设为 **Read and write**，否则最后的发布步骤会 403（产物 zip 仍可下载）。

产物解压即用，`aseprite.ini` 已内置便携标记，配置写在程序目录内。

## 相对上游 theguywhoslate/aseprite-builder 的改动

| 上游问题 | 本仓库处理 |
|---|---|
| Skia 硬编码 `m102`（2022 年），而当前源码要求 `m124` | 运行时读源码 `laf/misc/skia-tag.txt` 决定版本，升版本自动跟随 |
| 补 `icudtl.dat` 的直链已 404，落盘位置也不是 CMake 期望路径 | Skia 包内自带该文件 → 删掉下载步骤，改为构建前断言 |
| `choco install cmake` 会装到 4.x，而 CMake 4 对 `third_party` 里 `cmake_minimum_required(2.8/3.0)` 直接报错 | 固定 CMake 3.31.6 |
| `aseprite_build.bat` 只下载解压、不编译（123 行止于解压 Skia） | 已移除，云端流程为 cmake + ninja + 打包 |
| `aseprite_build.sh` 虽能编译，但同样硬编码 m102 Skia、无 icudtl 处理 | 已移除，避免误导 |
| `build-linux.yml` / `build-all-platforms.yml` 仍是 m102 + 404 链接 | 已移除，只保留经过验证的 Windows 便携版流程 |
| Windows PowerShell 5.1 用 `Out-File -Encoding utf8` 写 `$GITHUB_ENV` 会带 BOM，变量名变 `\ufeffASE_TAG` 导致后续取空、报莫名的 404 | 统一改用 `Add-Content -Encoding ascii` |

附带取舍：`-DENABLE_UPDATER=OFF -DENABLE_NEWS=OFF`，避开 libcurl / OpenSSL 依赖链，构建更快更稳（代价：Home 页新闻不显示、不自动检查更新）。Lua 脚本支持默认开启（源码包自带 `third_party/lua`），缺失时会自动关闭而不是中断构建。

## 法律边界

Aseprite 是 source-available 软件。其 EULA 允许**为你个人**编译使用，但**不得转发、售卖或公开分发**编译产物。请把仓库保持私有，产物自己用。喜欢的话请支持官方：<https://www.aseprite.org/>
