# nerion-android-lwjgl-build

Self-built, bionic-linked LWJGL 3.3.3 Android native libraries for Nerion
(app-launcher): `lwjgl`, `lwjgl-opengl`, `lwjgl-stb`, `lwjgl-freetype`,
`lwjgl-tinyfd`.

## Why this repo exists

app-launcher currently downloads LWJGL3 natives for Android from Maven
Central at runtime (`org.lwjgl:*:natives-linux-arm64`). Those jars are built
for generic **glibc** Linux — they `DT_NEEDED` versioned glibc SONAMEs such
as `libpthread.so.0`, `libc.so.6`, `ld-linux-aarch64.so.1`, `libdl.so.2`.
Android's **bionic** libc does not export those exact SONAMEs (bionic folds
`pthread` into `libc.so`, with no versioned `libc.so.6`), so loading those
jars on-device fails with:

```
dlopen failed: library "libpthread.so.0" not found
```

[`AngelAuraMC/lwjgl3`](https://github.com/AngelAuraMC/lwjgl3) (branch
`wip/rebase_3.3.3`) carries a real, working Android CI build of LWJGL 3.3.3
that links only against bionic-native SONAMEs. This repo builds from that
same recipe in its own CI (not a re-host of AngelAuraMC's own released
artifact) and publishes the resulting `.so` files as a GitHub Release, using
the ABI naming convention app-launcher's Rust code already standardizes on
(`arm64-v8a` / `armeabi-v7a` / `x86` / `x86_64` — see
`src-tauri/src/commands/java.rs`'s `AndroidAbi::android_abi_name()`).

This mirrors the pattern already established by
[`nerion-android-jdk-build`](https://github.com/feeldev12/nerion-android-jdk-build):
build from source in our own CI, publish our own artifacts to our own
public Release, don't hotlink someone else's compiled output.

## Reference / fork source

- repo: `AngelAuraMC/lwjgl3`
- branch: `wip/rebase_3.3.3`
- pinned commit: `6a564a8b4b1438699e9d7ccb42548e4ba454fd30` (2026-07-08)
- upstream CI run that proved this commit builds cleanly on GitHub Actions:
  [run 28921361028](https://github.com/AngelAuraMC/lwjgl3/actions/runs/28921361028)
  (conclusion: success)

## License

LWJGL itself is BSD-3-Clause (permissive) — there is no GPL-style
source-correspondence requirement here, unlike the OpenJDK (GPLv2) pipeline
in `nerion-android-jdk-build`. We still build from source in our own CI, not
from AngelAuraMC's released artifact, for consistency, maintainability, and
control over re-builds, matching this project's other from-source
pipelines. See `AngelAuraMC/lwjgl3` and the upstream
[LWJGL project](https://github.com/LWJGL/lwjgl3) for the original license
terms.

## Output

Published to the `lwjgl-3.3.3-android` GitHub Release, one `.so` per module
per ABI, named `{original-stem}-{abi}.so` (e.g. `liblwjgl-arm64-v8a.so`,
`libfreetype-arm64-v8a.so`), each with a matching `.sha256` checksum file.

## Scope

This repo only builds and publishes the native binaries. Wiring
app-launcher's Rust downloader (`src-tauri/src/commands/minecraft.rs`) to
consume this release instead of Maven Central is a separate, follow-up
change.
