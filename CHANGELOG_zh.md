# 更新日志

DODHooks 的所有重要变更都会记录在此文件中。

格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，本项目采用 SourceMod 风格版本号，扩展针对 SourceMod 1.12 构建。

## [1.6.1] - 2026-08-24

### 修复

- **关键修复：扩展自动加载功能失效。** `dodhooks.inc` 头文件中 `public Extension` 块的 `file` 字段被错误地写为 `"dodhooks.ext.2.dods"`，但 SourceMod 的自动加载机制期望填写**基础文件名** `"dodhooks.ext"`，由引擎自动追加游戏后缀（`.2.dods`）和平台扩展名（`.so`/`.dll`）。写死完整文件名导致解析失败，扩展无法自动加载，必须手动执行 `sm exts load dodhooks` 才能生效。现已修正为 `file = "dodhooks.ext"`——与 2015 年 psychonic 原版及所有标准 SourceMod 扩展（sdkhooks、dhooks 等）保持一致。

### 新增

- `__ext_dodhooks_SetNTVOptional()` — 当未定义 `REQUIRE_EXTENSIONS` 时，将全部 19 个 native 标记为可选，使插件在扩展不可用时仍能正常加载（调用时抛出运行时错误而非阻止插件加载）。恢复自 2015 年 psychonic 原版头文件。
- 通过 `AUTOLOAD_EXTENSIONS` 和 `REQUIRE_EXTENSIONS` 宏条件控制 `autoload` / `required`，允许插件作者在编译时控制扩展加载行为。恢复自 2015 年 psychonic 原版头文件。
- 恢复 2015 年 psychonic 原版中的枚举定义：
  - `DODRoundState` — 回合状态（RoundInit、PreGame、StartGame、PreRound、RoundRunning、AlliesWin、AxisWin、Restart、GameOver）
  - `DODPlayerState` — 玩家状态（Active、Welcome、PickingTeam、PickingClass、DeathAnim、ObserverMode）
  - `DODBombTargetState` — 炸弹点状态（Inactive、Active、Armed）
  - `DODVoiceCommand` — 全部 39 个语音命令 ID（Attack、Hold、Move、Medic、Grenade、Sniper 等）
- `#define MAX_CONTROL_POINTS 8` 常量。
- `stock bool IsPlayerClassValid(DODPlayerClass playerClass)` 工具函数。

### 变更

- `DOD_SetRoundState`、`DOD_SetPlayerState` 和 `DOD_SetBombTargetState` 现在使用强类型枚举参数（`DODRoundState`、`DODPlayerState`、`DODBombTargetState`）替代原始 `int`，为插件作者提供编译期类型检查。

## [1.6.0] - 2026-08-23

### 新增

- 通过 SourcePawn 头文件自动加载：`dodhooks.inc` 中新增 `public Extension __ext_dodhooks` 声明，任意 `#include <dodhooks>` 的插件都会由 SourceMod 在运行时自动加载 `dodhooks.ext`（`dodhooks.ext.2.dods`），无需 `.autoload` 标记文件或手动 `sm exts load`。
- 单命令同时构建两种架构：
  - `build.bat`（Windows）构建 x86 + x64，并生成 `DODHooks-<版本>-sm1.12-windows.zip`
  - `build.sh`（Linux）构建 x86 + x64，并生成 `DODHooks-<版本>-sm1.12-linux.tar.gz`
  - `build_linux_docker.sh` 在官方 AlliedModders 构建容器内运行 Linux 构建
- 发布打包结构：
  - 32 位二进制放在 `extensions/`（默认）
  - 64 位二进制放在 `extensions/x64/` 子目录
  - GameData 取自仓库内的 `gamedata/dodhooks.txt`
  - `dodhooks.inc` 一并打包到 `scripting/include/`
- GitHub Actions 工作流：构建 Windows（x86+x64）和 Linux（x86+x64），并在打 tag 时发布三个压缩包：
  - `DODHooks-<tag>-sm1.12-windows.zip`
  - `DODHooks-<tag>-sm1.12-linux.zip`
  - `DODHooks-<tag>-source.zip`
- 双语文档（`README.md` / `README_zh.md`）及本更新日志。

### 修复

- 修正 `configure.py` 参数：所有构建脚本、`Dockerfile` 与 CI 统一使用 `--arch=x86|x64`（原先为 `--target`）以及 `--sdks=dods`（原先为 `dod`）。
- `PackageScript` 现在从仓库的 `gamedata/dodhooks.txt` 复制 GameData，并从 `sourcemod/scripting/include/dodhooks.inc` 复制头文件（原先引用了错误路径）。
- 依赖版本统一：构建脚本、CI 与依赖设置脚本现在均使用 Metamod:Source `1.12-dev` 分支（原先 Windows 相关构建使用 `1.11-dev`）。

### 变更

- 整合构建脚本，移除了无法工作的 Windows→Linux 交叉编译脚本，以及冗余的 `docker/`、`scripts/`、`build_linux.sh` 文件。
- GameData 默认段明确面向胜利之日：起源（游戏目标 `dod`）。

### 兼容性

- SourceMod 1.12 / 1.13
- Metamod:Source 1.12
- Windows / Linux，32 位与 64 位
