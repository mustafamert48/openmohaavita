# OpenMoHAA Vita milestones

## v0.0.2-vita.1 — 2026-08-22

First hardware-tested milestone of the mustafamert48 Vita fork.

### Confirmed on real PS Vita hardware

- VPK builds reproducibly through GitHub Actions with VitaSDK.
- Engine boots and loads legitimate Medal of Honor: Allied Assault retail data from `ux0:data/openmohaa/main/`.
- Base campaign gameplay is functional on real Vita hardware.
- Forced multitexture (`r_vita_force_mtex 1`) restores substantially more correct diffuse + lightmap rendering than the legacy Vita fallback path.
- Textured sky rendering works through a Vita-safe vertex-array path.
- A practical baseline configuration was tested successfully at native 960x544 with a 30 FPS cap.
- Campaign progression from `m1l1` into `m1l2a` has been observed successfully on hardware.

### Recommended hardware-test baseline

```cfg
set r_lodbias 1
set r_subdivisions 16
set r_lodCurveError 250
set r_picmip 2

set r_dynamiclight 0
set r_flares 0
set cg_shadows 0
set r_shadows 0
set r_drawfog 0

set r_vita_force_mtex 1
set r_ext_multitexture 1
set r_vertexLight 0
set r_lightmap 0
set r_fastsky 0

set com_maxfps 30
```

### Known issues

- Performance remains highly scene-dependent. Roughly 25-30 FPS is possible in lighter scenes, while NPC-heavy combat can fall to around 10 FPS.
- Automatic transition saves remain disabled, while the tested cgame cleanup allows natural `m1l1 -> m1l2a` progression.
- Fresh manual saves now write the complete `.sav`, `.ssv`, and `.tga` set and load successfully. Incomplete slots made by older Vita builds cannot be recovered.
- `r_vita_vbo_world` remains experimental and disabled by default.
- Expansion folders (`mainta`, `maintt`) are not part of the current Vita base-game target.
- The port remains experimental; this is a development/testing milestone, not a stable release.

### Build / CI work in this fork

- Added/fixed Vita GitHub Actions build workflow.
- Made VitaSDK dependency installation noninteractive for CI.
- Preserved symbol/debug build support for Vita crash-dump investigation.

This milestone intentionally keeps the `0.0.x` version family because the Vita port is still under active renderer, performance, transition and save-system development.
