# Changelog

All notable changes to DODHooks are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.6.0/),
and this project follows SourceMod-style versioning where the extension is
built against SourceMod 1.12.

## [1.6.1] - 2026-08-24

### Fixed

- **Critical: Extension auto-load was broken.** The `dodhooks.inc` header
  declared `file = "dodhooks.ext.2.dods"` in the `public Extension` block,
  but SourceMod's auto-loader expects the **base name** `file = "dodhooks.ext"`
  and appends the game suffix (`.2.dods`) and platform extension (`.so`/`.dll`)
  automatically. The over-specified filename caused resolution to fail, so the
  extension would not auto-load and required manual `sm exts load dodhooks`.
  Fixed by reverting to `file = "dodhooks.ext"` — matching the 2015 psychonic
  original and all standard SourceMod extensions (sdkhooks, dhooks, etc.).

### Added

- `__ext_dodhooks_SetNTVOptional()` — marks all 19 natives as optional when
  `REQUIRE_EXTENSIONS` is not defined, so plugins load gracefully even if the
  extension is unavailable (natives throw a runtime error when called instead
  of blocking plugin load). Restored from the 2015 psychonic include.
- Conditional `autoload` / `required` via `AUTOLOAD_EXTENSIONS` and
  `REQUIRE_EXTENSIONS` defines, allowing plugin authors to control extension
  loading behavior at compile time. Restored from the 2015 psychonic include.
- Enumerations restored from the 2015 psychonic include:
  - `DODRoundState` — round states (RoundInit, PreGame, StartGame, PreRound,
    RoundRunning, AlliesWin, AxisWin, Restart, GameOver)
  - `DODPlayerState` — player states (Active, Welcome, PickingTeam,
    PickingClass, DeathAnim, ObserverMode)
  - `DODBombTargetState` — bomb target states (Inactive, Active, Armed)
  - `DODVoiceCommand` — all 39 voice command IDs (Attack, Hold, Move, Medic,
    Grenade, Sniper, etc.)
- `#define MAX_CONTROL_POINTS 8` constant.
- `stock bool IsPlayerClassValid(DODPlayerClass playerClass)` utility function.

### Changed

- `DOD_SetRoundState`, `DOD_SetPlayerState`, and `DOD_SetBombTargetState`
  now use strongly-typed enum parameters (`DODRoundState`, `DODPlayerState`,
  `DODBombTargetState`) instead of raw `int`, providing compile-time type
  checking for plugin authors.

## [1.6.0] - 2026-08-23

### Added

- Auto-load via the SourcePawn include: `dodhooks.inc` now declares
  `public Extension __ext_dodhooks`, so any plugin that
  `#include <dodhooks>` makes SourceMod automatically load `dodhooks.ext`
  (`dodhooks.ext.2.dods`) at runtime. No `.autoload` marker file or manual
  `sm exts load` is required.
- Single-command builds for both architectures:
  - `build.bat` (Windows) builds x86 + x64 and produces
    `DODHooks-<ver>-sm1.12-windows.zip`.
  - `build.sh` (Linux) builds x86 + x64 and produces
    `DODHooks-<ver>-sm1.12-linux.tar.gz`.
  - `build_linux_docker.sh` runs the Linux build inside the official
    AlliedModders build container.
- Release packaging:
  - 32-bit binary in `extensions/` (default).
  - 64-bit binary in `extensions/x64/` subfolder.
  - GameData copied from repository `gamedata/dodhooks.txt`.
  - `dodhooks.inc` packaged into `scripting/include/`.
- GitHub Actions workflow that builds Windows (x86+x64) and Linux (x86+x64)
  and publishes three release archives on tags:
  - `DODHooks-<tag>-sm1.12-windows.zip`
  - `DODHooks-<tag>-sm1.12-linux.zip`
  - `DODHooks-<tag>-source.zip`
- Bilingual documentation (`README.md` / `README_zh.md`) and this changelog.

### Fixed

- Correct `configure.py` flags: `--arch=x86|x64` (was `--target`) and
  `--sdks=dods` (was `dod`) across all build scripts, `Dockerfile` and CI.
- `PackageScript` now copies GameData from the repository's
  `gamedata/dodhooks.txt` and the include from
  `sourcemod/scripting/include/dodhooks.inc` (previously referenced the wrong
  paths).
- Unified dependency versions: build scripts, CI, and setup script now all use
  Metamod:Source `1.12-dev` branch (previously Windows builds used `1.11-dev`).

### Changed

- Consolidated build scripts; removed the broken Windows→Linux cross-compile
  attempt and redundant `docker/`, `scripts/`, `build_linux.sh` files.
- GameData default section documented as targeting Day of Defeat: Source
  (game folder `dod`).

### Compatibility

- SourceMod 1.12 / 1.13
- Metamod:Source 1.12 / 2.0
- Windows and Linux, 32-bit and 64-bit
