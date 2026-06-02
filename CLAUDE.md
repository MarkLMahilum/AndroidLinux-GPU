# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Bash-based installer for running a Linux desktop environment with GPU acceleration on Android via Termux. No build system or package manager is involved — the scripts run directly in Termux on Android (shebang: `#!/data/data/com.termux/files/usr/bin/bash`).

## Scripts

- **[install.sh](install.sh)** — 13-step installer: detects device/GPU, installs Termux packages (`pkg`), creates launcher scripts at `~/start-hacklab.sh`, `~/hacktools.sh`, `~/stop-hacklab.sh`, and desktop `.desktop` shortcuts.
- **[uninstall-hacklaab.sh](uninstall-hacklaab.sh)** — Reverses every install step: stops processes, removes packages via `pkg uninstall`, deletes generated scripts and configs.

## Architecture

The installer follows a linear step pattern: each `step_*` function calls `update_progress` (increments `CURRENT_STEP` out of `TOTAL_STEPS=13`) then runs `install_pkg` calls. `install_pkg` runs `pkg install` in a subshell and passes the PID to `spinner` for animated feedback.

GPU driver selection happens in `detect_device`: Qualcomm Adreno phones get `mesa-vulkan-icd-freedreno` (Turnip), everything else falls back to `mesa-vulkan-icd-swrast`. The chosen driver is stored in `$GPU_DRIVER` and consumed by `step_gpu`.

The GPU environment variables (`GALLIUM_DRIVER=zink`, `MESA_GL_VERSION_OVERRIDE`, etc.) are written to `~/.config/hacklab-gpu.sh` and sourced from `~/.bashrc`.

## Testing

These scripts must be tested on a physical Android device running Termux — they rely on `getprop`, Termux-specific paths (`/data/data/com.termux/files/usr/`), and Android-only packages. There is no way to unit-test or dry-run them on a desktop Linux system.

## Deployment

The canonical install command (run inside Termux on Android):
```bash
curl -sL https://raw.githubusercontent.com/AbuZar-Ansarii/AndroidLinux-GPU/main/install.sh | bash
```

After installation, users interact with the generated scripts:
```bash
bash ~/start-hacklab.sh   # Start XFCE4 desktop
bash ~/hacktools.sh       # Interactive security tools menu
bash ~/stop-hacklab.sh    # Stop the desktop
```
