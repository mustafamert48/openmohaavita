# OpenMoHAA Vita

[![Vita build](https://github.com/ChatProductions/openmohaavita/actions/workflows/vita-build.yml/badge.svg?branch=vita-port)](https://github.com/ChatProductions/openmohaavita/actions/workflows/vita-build.yml)
[![Latest release](https://img.shields.io/github/v/release/ChatProductions/openmohaavita?include_prereleases&label=Vita%20release)](https://github.com/ChatProductions/openmohaavita/releases/tag/v0.0.2-vita.2)
[![License](https://img.shields.io/github/license/ChatProductions/openmohaavita)](COPYING.txt)

![OpenMoHAA](misc/openmohaa-text-sm.png)

An experimental PlayStation Vita port of [OpenMoHAA](https://github.com/openmoh/openmohaa), maintained by **Chat Productions**.

> [!IMPORTANT]
> **This is a vibe-coded, AI-assisted hobby project.** Much of the Vita-specific work has been drafted with ChatGPT/Codex, then built in CI and tested iteratively on real Vita hardware by Chat Productions. A green build does not prove that a change is correct; crash dumps, logs and repeatable hardware tests decide what ships. This fork is independent of, and not endorsed by, the upstream OpenMoHAA maintainers.

The port is playable, but it is not finished. Back up your saves and expect rough edges.

## Current progress

Status as of **29 August 2026**, using [`v0.0.2-vita.2`](https://github.com/ChatProductions/openmohaavita/releases/tag/v0.0.2-vita.2):

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

1. Download `OpenMoHAA.vpk` from the [latest release](https://github.com/ChatProductions/openmohaavita/releases/tag/v0.0.2-vita.2).
2. Install the VPK with VitaShell.
3. Copy your retail game data to:

   ```text
   ux0:data/openmohaa/main/
   ```

4. Copy the following data from your retail installation:

   | File or folder | Requirement | Notes |
   |---|---|---|
   | `Pak0.pk3` – `Pak3.pk3` | Required | Base-game data from the retail installation. |
   | `Pak4.pk3` – `Pak5.pk3` | Required | Data from the official 1.11 update. |
   | Other `Pak*.pk3` files | Copy when present | Includes language and additional official data such as `Pak6EnUk.pk3` or `pak7.pk3`. Copy every pak supplied by your installation. |
   | `music/` | Copy when present | Loose background-music files stored outside the paks. |
   | `sound/` | Copy when present | Loose audio must keep the correct subfolders and lowercase filenames for Vita's case-sensitive lookup. |
   | `video/` | Copy when present | RoQ intro and cinematic files stored outside the paks. |

5. Launch OpenMoHAA from LiveArea. The engine creates its own `configs/` and `save/` folders; do not copy them from the computer installation.

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
| Select | Open / close the Vita dev menu |
| Start | Menu |

The M1 Garand does not have a normal iron-sight zoom in the original game and cannot be topped up before the clip is empty. The L binding works only for weapons with a secondary attack or scope.

## Known issues and workarounds

### Long loading times

Loading a recent save was measured at roughly 178 seconds: about 122 seconds restoring the server/save state and 56 seconds initializing the client. CPU overclocking does not remove the underlying bottleneck.

### Mission briefings are currently skipped

The startup videos—EA logo, title and legal screens—are enabled. However, the six interactive campaign briefing maps are deliberately redirected to their corresponding first gameplay levels on Vita.

This workaround was added during the original Vita bring-up because loading a briefing and then starting its mission performed a second game-module initialization with stale state, causing crashes. Later transition fixes may make the redirect unnecessary, but the briefing path has not yet been safely re-enabled and tested on hardware.

### Select-button dev menu

Press **Select** during gameplay to open the Vita dev/performance menu. Use **D-pad left/right** to change category, **D-pad up/down** to select an item, **Cross** to toggle or run it, and **Circle** or **Select** to close the menu.

The menu provides quick access to level loading, game commands, world rendering, lighting, effects, aim settings, texture quality and diagnostic options. For example, the crosshair can be toggled globally under **Effects**, or separately for hip fire and aiming under **Aim**.

Some entries are experimental or intended for debugging and may not work as expected. Renderer and diagnostic toggles can also reduce performance or produce incorrect visuals, so change them one at a time when testing.

### Incomplete older saves

A loadable slot needs `.sav`, `.ssv` and `.tga` files. Old Vita builds sometimes wrote only metadata and a screenshot. A slot with no `.sav` world archive cannot be recovered.

## Reporting a Vita problem

[Open an issue](https://github.com/ChatProductions/openmohaavita/issues) and include:

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

- Chat Productions maintains the Vita port and performs the real-hardware testing documented here.
- [OpenMoHAA](https://github.com/openmoh/openmohaa) and its contributors provide the engine this port is based on.
- [VitaSDK](https://vitasdk.org/) supplies the Vita toolchain and platform libraries.
- [vitaGL](https://github.com/Rinnegatamante/vitaGL) provides the OpenGL-compatible rendering layer used on Vita.

OpenMoHAA is independent of and not endorsed by Electronic Arts. The engine source is distributed under the terms in [COPYING.txt](COPYING.txt); original game assets are not included.
