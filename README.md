# OpenMoHAA Vita

[![Vita build](https://github.com/mustafamert48/openmohaavita/actions/workflows/vita-build.yml/badge.svg?branch=vita-port)](https://github.com/mustafamert48/openmohaavita/actions/workflows/vita-build.yml)
[![Latest release](https://img.shields.io/github/v/release/mustafamert48/openmohaavita?include_prereleases&label=Vita%20release)](https://github.com/mustafamert48/openmohaavita/releases/latest)
[![License](https://img.shields.io/github/license/mustafamert48/openmohaavita)](COPYING.txt)

![OpenMoHAA](misc/openmohaa-text-sm.png)

An experimental PlayStation Vita port of [OpenMoHAA](https://github.com/openmoh/openmohaa), the open-source reimplementation of *Medal of Honor: Allied Assault*.

> [!IMPORTANT]
> **This is a vibe-coded, AI-assisted hobby project.** Much of the Vita-specific work has been drafted with ChatGPT/Codex, then built in CI and tested iteratively on real Vita hardware by the repository owner. A green build does not prove that a change is correct; crash dumps, logs and repeatable hardware tests decide what ships. This fork is independent of, and not endorsed by, the upstream OpenMoHAA maintainers.

The port is playable, but it is not finished. Back up your saves and expect rough edges.

## Current progress

Status as of **29 August 2026**, using [`v0.0.2-vita.2`](https://github.com/mustafamert48/openmohaavita/releases/tag/v0.0.2-vita.2):

| Area | Status | Hardware-tested result |
|---|---|---|
| Boot, menus and audio | Working | Starts from LiveArea and reaches gameplay on a real Vita. |
| Base campaign | In progress | Played through the early campaign, including the truck-sabotage section of the third mission. The full campaign is not yet validated. |
| Mission transitions | Working so far | Natural transitions through the tested missions complete without the earlier stale-event crash. |
| Manual and checkpoint saves | Working so far | Saves can be created, loaded, closed and loaded again after restarting the app. |
| World rendering | Working with limitations | BSP geometry, lightmaps and textured skies render through Vita-safe paths. Visual bugs may remain. |
| Performance | Needs work | Around 18–20 FPS in early tested scenes, with lower performance in busy combat. Save loading can take close to three minutes. |
| Save thumbnails | Buggy | Some `.tga` previews contain black or noisy pixels. The separate `.sav` payload can still be valid. |
| Multiplayer | Disabled | The Vita networking and GameSpy paths are stubbed. |
| Spearhead / Breakthrough | Not validated | The current Vita release targets the base game's `main/` data. |

### What changed in `v0.0.2-vita.2`

- Fixed use-after-free crashes in persistent client effects across mission transitions.
- Stored stable TIKI model names for temporary effects instead of renderer-owned pointers.
- Recovered recent complete saves whose temporary effects had a missing TIKI pointer.
- Rejected malformed skeletal render entities instead of dereferencing a null pointer.
- Retained the textured-sky, BSP triangle, multitexture/lightmap, transition and complete-save fixes from the previous build.

See the [Vita milestone history](docs/VITA_CHANGELOG.md) for more detail.

## Install

You need a homebrew-capable PS Vita and your own legitimate copy of *Medal of Honor: Allied Assault*. This repository and its releases do **not** contain EA game data.

1. Download `OpenMoHAA.vpk` from the [latest release](https://github.com/mustafamert48/openmohaavita/releases/tag/v0.0.2-vita.2).
2. Install the VPK with VitaShell.
3. Copy your retail game data to:

   ```text
   ux0:data/openmohaa/main/
   ```

4. Copy every `Pak*.pk3` from your installation. `Pak0.pk3` through `Pak5.pk3` are expected for the patched base game. Localized editions may include an additional language pak.
5. Copy loose `music/` and `sound/` files if your installation keeps them outside the paks.
6. Launch OpenMoHAA from LiveArea.

You can prepare a correctly laid-out data directory on a computer with:

```sh
misc/console/prepare-data.sh /path/to/retail/main ./out-main vita
```

Then copy the contents of `./out-main` to `ux0:data/openmohaa/main/`.

## Default controls

| Vita input | Action |
|---|---|
| Left stick | Move |
| Right stick | Look / aim |
| Cross | Use / interact |
| Circle | Crouch; hold for prone |
| Square | Reload |
| Triangle | Jump |
| L | Secondary attack / scope on weapons that support it |
| R | Fire |
| D-pad left / right | Previous / next weapon |
| D-pad up | Alternate use |
| Select | Objectives / score |
| Start | Menu |

The M1 Garand does not have a normal iron-sight zoom in the original game and cannot be topped up before the clip is empty. The L binding works only for weapons with a secondary attack or scope.

## Known issues and workarounds

### Long loading times

Loading a recent save was measured at roughly 178 seconds: about 122 seconds restoring the server/save state and 56 seconds initializing the client. CPU overclocking does not remove the underlying bottleneck.

### Cheat menu entries

The current menu's **Cheats ON** action is incomplete, and its **God Mode** action sends the wrong command. As a temporary workaround, add these lines to `ux0:data/openmohaa/main/configs/omconfig.cfg`:

```cfg
seta thereisnomonkey "1"
seta cheats "1"
alias god "dog"
```

Restart the app after editing the file. `dog` is the command used by the game; the alias makes the familiar `god` spelling work too.

### Incomplete older saves

A loadable slot needs `.sav`, `.ssv` and `.tga` files. Old Vita builds sometimes wrote only metadata and a screenshot. A slot with no `.sav` world archive cannot be recovered.

## Reporting a Vita problem

[Open an issue](https://github.com/mustafamert48/openmohaavita/issues) and include:

- the release or commit you tested;
- the mission/map and the exact sequence before the problem;
- whether the game was freshly started, checkpoint-loaded or manual-save-loaded;
- the matching `boot.log`;
- the newest `psp2core-*.psp2dmp` for a crash;
- the affected save folder when the crash depends on a particular save.

Please say whether the problem reproduces. A one-off crash and a deterministic crash need different investigation.

## Building the Vita version

The GitHub Actions workflow builds the VPK and uploads separate symbol files for crash-dump analysis. For a local VitaSDK build, see [the Vita build and debugging guide](docs/PORTING-VITA.md).

## Contributing

This fork accepts hardware test reports, documentation fixes and focused code changes. Read the [fork contribution policy](CONTRIBUTING.md), especially the disclosure and verification requirements for AI-assisted work.

Do not submit this fork's AI-derived changes to upstream OpenMoHAA as if they were independently human-authored. Upstream has its own contribution policy.

## Credits and license

- [OpenMoHAA](https://github.com/openmoh/openmohaa) and its contributors provide the engine this port is based on.
- [VitaSDK](https://vitasdk.org/) supplies the Vita toolchain and platform libraries.
- [vitaGL](https://github.com/Rinnegatamante/vitaGL) provides the OpenGL-compatible rendering layer used on Vita.

OpenMoHAA is independent of and not endorsed by Electronic Arts. The engine source is distributed under the terms in [COPYING.txt](COPYING.txt); original game assets are not included.
