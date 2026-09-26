<div align="center">

# 🖤 ricename — Erebus

*A minimal, mood-reactive Wayland desktop.*

![OS](https://img.shields.io/badge/OS-Ubuntu%2026-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![WM](https://img.shields.io/badge/WM-Hyprland-58E1FF?style=for-the-badge&logo=wayland&logoColor=white)
![Shell](https://img.shields.io/badge/Shell-DankMaterialShell-9146FF?style=for-the-badge)
![Theme Engine](https://img.shields.io/badge/Colors-Matugen-B99B96?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-black?style=for-the-badge)

</div>

---

## Table of Contents
- [Overview](#overview)
- [Specs](#specs)
- [Moods](#moods)
- [Architecture](#architecture)
- [Keybindings](#keybindings)
- [Install](#install)
- [Credits](#credits)

---

## Overview

A dual-session Ubuntu setup: GNOME/GDM kept fully intact as a permanent fallback, with Hyprland + DankMaterialShell as the daily driver. The centerpiece is a **mood engine** — a single wallpaper-driven theming pipeline that recolors the compositor, terminal, system monitor, and system-info banner together, per mood, with zero mood-specific scripts duplicated.

## Specs

| Component | Spec |
|---|---|
| OS | Ubuntu 26.04 LTS |
| Compositor | Hyprland |
| Shell | DankMaterialShell (DMS) + Quickshell |
| GPU | NVIDIA RTX 3050 (PRIME on-demand) |
| Terminal | Ghostty |
| Color Engine | Matugen (live wallpaper-derived) |
| Fonts | Plus Jakarta Sans (UI) · Geist Mono (terminal) |
| Display | 1920x1200 @ 144Hz, VRR enabled |

## Moods

Four moods included in this repo. Each is a self-contained `profile.conf` + wallpaper set — no other files change.

| Mood | Identity | Notes |
|---|---|---|
| **Nexus** | Cyan/teal cyber-precision | Sharp corners, no blur, instant linear animation |
| **The Gallery** | Painterly, museum-focus | Muted watercolor palette, UI recedes behind content |
| **The Paths** | Field/expedition terminal identity | Custom prompt, terminal shader, live system telemetry widget |
| **Cursed** | Violet/crimson high-contrast | Gradient borders, custom notification center, animated border-flash on window close |

> Two additional personal moods exist on the author's own machine and are intentionally not published here.

## Architecture

- **`mood-switch`** — one script, driven entirely by each mood's `profile.conf` (opacity, blur, rounding, animation curve/speed, accent fallback). Adding a mood never requires touching the script.
- **Live accent parsing** — Matugen generates a palette from the active wallpaper; the script parses the resulting Qt color file and pushes it into Hyprland border colors via `hyprctl --batch`.
- **Terminal layer** — Ghostty theme + config are swapped via an atomic pointer-swap file (`current.ghostty`), so terminal color/opacity/shader follow the active mood with zero restart.
- **System monitor / fetch layer** — `btop` and `fastfetch` configs follow the same pointer-swap pattern, keyed off the same mood state file.
- **Bar widget** — a config-driven DMS plugin reads a per-mood `telemetry.json` (metrics + thresholds only, no arbitrary code) to render mood-specific system stats in the bar.

State lives in one file (`~/.config/mood-engine/state`); every subsystem reads it independently and self-corrects on the next switch.

## Keybindings

| Keybind | Action |
|---|---|
| `SUPER + Enter` | Open terminal |
| `SUPER + F` | App launcher / search |
| `SUPER + Q` | Close window |
| `SUPER + SHIFT + Q` | Toggle floating scratchpad terminal |
| `SUPER + A` | Power-profile picker |
| `CTRL + SHIFT + [1-4]` | Switch mood |
| _TBD_ | _screenshot / lock / etc. — placeholder_ |

## Install

```bash
sudo apt install hyprland xdg-desktop-portal-hyprland
# DMS + Quickshell (avengemedia PPAs)
sudo add-apt-repository ppa:avengemedia/danklinux
sudo add-apt-repository ppa:avengemedia/dms
sudo apt update && sudo apt install dms quickshell ghostty btop fastfetch

git clone https://github.com/<you>/<repo>.git ~/.config/mood-engine-src
cp -r ~/.config/mood-engine-src/mood-engine ~/.config/
cp -r ~/.config/mood-engine-src/ghostty ~/.config/
ln -sf ~/.config/mood-engine-src/bin/mood-switch ~/.local/bin/mood-switch

# switch mood
mood-switch nexus
```

## Credits

- [DankMaterialShell](https://github.com/AvengeMedia/DankMaterialShell)
- Cursor shaders: [sahaj-b/ghostty-cursor-shaders](https://github.com/sahaj-b/ghostty-cursor-shaders) (MIT)
- Matugen — wallpaper-driven color generation
