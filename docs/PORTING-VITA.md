# Building and debugging OpenMoHAA Vita

This document covers the experimental PlayStation Vita target in this fork. For the user-facing status, installation instructions and controls, start with the [project README](../README.md).

## Scope

The current Vita release targets the base *Medal of Honor: Allied Assault* campaign using data in `ux0:data/openmohaa/main/`.

Single-player is the focus. Multiplayer is disabled because the existing network and GameSpy paths are stubbed on Vita. Spearhead and Breakthrough are not part of the current hardware-test scope.

## Host requirements

- Linux, macOS or Windows with WSL
- CMake 3.25 or newer
- Bison 3.5.1 or newer
- Flex 2.6.4 or newer
- [VitaSDK](https://vitasdk.org/), with `VITASDK` set

The CI workflow in [`.github/workflows/vita-build.yml`](../.github/workflows/vita-build.yml) is the reproducible reference build.

## Install VitaSDK dependencies

Using `vdpm`:

```sh
vdpm install \
  zlib bzip2 libpng libjpeg-turbo sdl2 openal-soft openssl curl \
  libogg libvorbis opus opusfile libmad libmathneon vitaShaRK vitaGL \
  SceShaccCgExt kubridge taihen vita-rss-libdl
```

Some dependency metadata requests `librt`, although VitaSDK provides the required time functions through libc. If the linker cannot find `-lrt`, create the same compatibility archive used by CI:

```sh
printf 'void __vita_librt_stub(void) {}\n' > /tmp/librt_stub.c
"$VITASDK/bin/arm-vita-eabi-gcc" -c /tmp/librt_stub.c -o /tmp/librt_stub.o
"$VITASDK/bin/arm-vita-eabi-ar" rcs \
  "$VITASDK/arm-vita-eabi/lib/librt.a" /tmp/librt_stub.o
```

## Configure and build

```sh
cmake -S . -B build-vita \
  -DCMAKE_TOOLCHAIN_FILE="$VITASDK/share/vita.toolchain.cmake" \
  -DCMAKE_BUILD_TYPE=Release \
  -G Ninja

cmake --build build-vita -j2
```

The package is written to `build-vita/OpenMoHAA.vpk`.

Keep the unstripped `openmohaa`, `game.elf`, `cgame.elf` and VELF files from the exact build. Vita crash addresses are useful only when resolved against matching symbols.

## Install and data layout

Install `OpenMoHAA.vpk` with VitaShell, then copy legally obtained retail data into:

```text
ux0:data/openmohaa/main/
├── Pak0.pk3
├── Pak1.pk3
├── Pak2.pk3
├── Pak3.pk3
├── Pak4.pk3
├── Pak5.pk3
├── music/              # loose files when present
├── sound/              # loose files when present
└── configs/
```

Localized installations may contain an additional language pak. Copy every `Pak*.pk3` from the legitimate installation. Missing `Pak2.pk3`, for example, leaves much of the world without its expected textures.

The helper script prepares a Vita data directory and normalizes loose sound paths:

```sh
misc/console/prepare-data.sh /path/to/retail/main ./out-main vita
```

## Vita-specific source layout

| Path | Purpose |
|---|---|
| `cmake/platforms/vita.cmake` | Vita toolchain, libraries and VPK packaging |
| `code/sys/sys_vita.c` | Platform startup and Vita memory setup |
| `code/qcommon/net_vita.c` | Disabled network implementation |
| `code/sdl/vita_gl_stubs.c` | Legacy GL entry points not supplied by vitaGL |
| `code/gamespy/gamespy_vita_stub.c` | No-op GameSpy layer |
| `misc/vita/sce_sys/` | LiveArea assets |
| `misc/vita/main/autoexec.cfg` | Bundled controls and Vita defaults |

Most changes shared with desktop code are guarded by `__vita__`. A Vita fix should stay inside that boundary unless the same defect is demonstrated on other platforms.

## Current technical state

The engine, filesystem, renderer, audio, scripting, AI and base campaign all run on real hardware. Important Vita-specific fixes currently include:

- BSP triangle submission through a vitaGL-compatible path;
- diffuse and lightmap multitexturing;
- textured skies using vertex arrays rather than an unsafe immediate-mode path;
- cgame event and temporary-effect lifetime handling across mission transitions;
- complete manual save payloads and recovery of missing temporary-effect TIKI pointers;
- separate CI symbol artifacts for dump analysis.

Known limitations:

- performance is highly scene-dependent;
- save restoration can take close to three minutes;
- some save thumbnails capture black or noisy framebuffer data;
- the complete base campaign has not been validated;
- multiplayer and expansion campaigns are not supported test targets;
- untested effect and mission paths may still crash.

## Debugging a hardware crash

Collect all of the following before changing code:

1. The exact release, commit or Actions run used to build the VPK.
2. The matching `boot.log`.
3. The newest `psp2core-*.psp2dmp`.
4. Exact reproduction steps, including mission, checkpoint/manual-load history and the action that triggered the crash.
5. The save folder when the failure depends on a particular slot.

Resolve the dump only against symbols from the exact same VPK. A nearby commit can move every function offset and produce a convincing but false diagnosis.

Useful distinctions when triaging:

- a crash during save creation is different from a crash after `Game Loaded`;
- a transition crash is different from a later combat-effect crash;
- malformed `.tga` thumbnails do not prove that the `.sav` world archive is corrupt;
- a complete slot normally contains `.sav`, `.ssv` and `.tga` files.

## Verification before release

A green Vita build is only the first gate. Run a hardware sequence that covers:

1. a natural mission transition;
2. combat with bullets, explosions and temporary effects;
3. a fresh manual save and load;
4. a checkpoint load followed by loading the manual save;
5. closing the app, reopening it and loading again;
6. continued play after restoration.

If a crash occurs, keep the exact VPK and its symbol artifact together until the dump has been resolved.
