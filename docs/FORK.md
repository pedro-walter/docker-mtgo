# About this fork

Fork of [pauleve/docker-mtgo](https://github.com/pauleve/docker-mtgo) (last
upstream commit: Aug 2024, wine 9.14) that combines unmerged work from other
forks and pull requests so MTGO runs on current wine with working sound.

Upstream was surveyed on 2026-10-03: of its 44 forks, 6 had recent changes.
Only the ones below were needed.

## What is included

| Change | Source | Notes |
|---|---|---|
| Wine 11.2 (`panard/wine:11.2-wow64`) | [upstream PR #204](https://github.com/pauleve/docker-mtgo/pull/204), commit `5fa86f9` from [g0dnerd/docker-mtgo](https://github.com/g0dnerd/docker-mtgo) | Same one-line bump as #204. |
| Installer started as `C:\mtgo-setup.exe` | [g0dnerd/docker-mtgo](https://github.com/g0dnerd/docker-mtgo), commit `5f30e67` | Wine 11 segfaults when starting an exe by unix path while the `Z:` drive is removed (which `extra/mtgo.sh` does on purpose). Without this the stock launcher crashes on wine 11. |
| `--sound` that actually produces sound | [upstream PR #219](https://github.com/pauleve/docker-mtgo/pull/219) by emostar (`extra/mtgo-audio-patch.py`, `SOUND.md`, `sound/tests/`) | Patches the MTGO client in the user's own volume; no patched binary is distributed. See `SOUND.md`. |
| `libpython3-stdlib` in the sound image | This fork | `python3-minimal` on Debian 13 lacks `shutil`/`wave`, so the PR #219 script failed with `ModuleNotFoundError: No module named 'shutil'`. |

## Looked at and left out

| Fork / PR | What it does | Why not |
|---|---|---|
| apd-f | Wine 11.2 bump only | Covered by the above. |
| lfaoro | Wine 10.19 + GitHub Actions push to GHCR | Superseded by 11.2. The CI workflow could be added later. |
| akonskarm | Mesa/Vulkan libs, dxvk, `/dev/dri`, forced 1920x1080 virtual desktop, sound disabled | Aimed at Mesa GPUs, and the defaults are bad (forced desktop size, no sound). |
| jcastonguay / upstream PR #215 | Podman support | Docker is used here. |
| notwarp | Wayland/XWayland Xauthority detection | Only needed on a Wayland session. |

## Tested on

- Linux Mint (Cinnamon, X11 session), Docker 29, NVIDIA RTX 2060 (not used by
  the container), PipeWire with the PulseAudio socket.
- MTGO starts and **sound works** with `--sound`.
- Not tested: macOS, podman, Wayland, GPU passthrough.

## Building and running locally

The default image in `run-mtgo` is still the upstream `panard/mtgo:latest`
(wine 9.14), so pass the locally built image explicitly:

```
make image                                # panard/mtgo:<today>
make sound BASE=panard/mtgo:<today>       # panard/mtgo:sound (adds audio support)
./run-mtgo --sound panard/mtgo:sound
```

`BASE` defaults to today's date, so it only needs to be given if the sound image
is built on a different day than the base image.

The first build is slow (`winetricks dotnet48` runs on a single CPU).
