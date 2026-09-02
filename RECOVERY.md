# Backup and Recovery

This document describes how backups are created, verified, restored, and used to recover the desktop environment.

The goal is to ensure that changes to Hyprland, DankMaterialShell (DMS), themes, keybindings, and related configuration can be safely reverted without affecting Ubuntu, GNOME, GDM, NVIDIA drivers, CUDA, or development environments.

## System reference

- OS: Ubuntu 26.04.1 LTS ("resolute")
- Kernel: 7.0.0-30-generic
- GPU: NVIDIA GeForce RTX 3050 4GB (driver 595.84, open kernel modules) + Intel Raptor Lake-P iGPU (hybrid)
- Display manager: GDM3
- Fallback desktop: GNOME Shell 50.1 on Wayland

## Backup strategy

Creating a snapshot of the current GNOME environment before installing anything.

### 1. Create the backup directory

 Creates a timestamped folder under the home directory to hold all backup artifacts.
 
```bash
BACKUP_DIR=~/backups/gnome-pre-hyprland-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP_DIR"
echo "Backup directory: $BACKUP_DIR"
```

### 2. Export dconf (GNOME settings, extensions, keybindings)

Dump the entire dconf tree — GNOME Shell settings, extension configuration, keybindings, GTK preferences — to a plain text file.
```bash
dconf dump / > "$BACKUP_DIR/dconf-full-backup.ini"
```

### 3. Record system state (human-readable reference)

Create a plain-text snapshot of theme, font, terminal, NVIDIA driver, and monitor configuration, separate from the dconf binary dump.

```bash
{
  echo "=== Enabled GNOME Extensions ==="
  gnome-extensions list --enabled
  echo ""
  echo "=== GTK Theme ==="
  gsettings get org.gnome.desktop.interface gtk-theme
  echo "=== Icon Theme ==="
  gsettings get org.gnome.desktop.interface icon-theme
  echo "=== Cursor Theme ==="
  gsettings get org.gnome.desktop.interface cursor-theme
  echo "=== Font ==="
  gsettings get org.gnome.desktop.interface font-name
  echo "=== Monospace Font ==="
  gsettings get org.gnome.desktop.interface monospace-font-name
  echo ""
  echo "=== Default Terminal ==="
  update-alternatives --display x-terminal-emulator
  echo ""
  echo "=== NVIDIA Driver ==="
  nvidia-smi --query-gpu=driver_version,name --format=csv
  echo ""
  echo "=== Monitor Config ==="
  cat ~/.config/monitors.xml
} > "$BACKUP_DIR/system-state.txt"
```

### 4. Export package manifests

Creates a full lists of installed APT packages, Snaps, and Flatpaks.

```bash
dpkg --get-selections > "$BACKUP_DIR/apt-packages.txt"
snap list > "$BACKUP_DIR/snap-packages.txt"
flatpak list > "$BACKUP_DIR/flatpak-packages.txt" 2>/dev/null
```

### 5. Selective `~/.config` backup (secrets excluded)

Copy terminal configuration, GTK theming files, monitor layout, and the raw dconf database file. Deliberately narrow — not a full `~/.config` copy.

```bash
mkdir -p "$BACKUP_DIR/config-selective"

rsync -av --exclude='BraveSoftware' --exclude='google-chrome' \
  --exclude='discord' --exclude='*.log' --exclude='Cache*' \
  ~/.config/ptyxis ~/.config/gtk-3.0 ~/.config/gtk-4.0 \
  ~/.config/monitors.xml ~/.config/dconf \
  "$BACKUP_DIR/config-selective/" 2>/dev/null

echo "Selective config backup complete."
```

### 6. Verify the backup

```bash
echo "=== Backup contents ==="
ls -lh "$BACKUP_DIR"
echo ""
echo "=== dconf backup size check (should be non-trivial, not 0 bytes) ==="
wc -l "$BACKUP_DIR/dconf-full-backup.ini"
echo ""
echo "=== Confirm apt package count matches system ==="
wc -l "$BACKUP_DIR/apt-packages.txt"
dpkg -l | grep -c '^ii'
```

The dconf file should contain hundreds or thousands of lines, not be empty. The package count in the backup should closely match the live installed count (a difference of 1 is expected, due to a trailing line in `dpkg --get-selections` output).

## What is intentionally NOT backed up

The following are excluded from this backup process, on purpose:

- Browser profiles (cookies, session tokens, saved logins)
- Password stores / `~/.local/share/keyrings`
- Chat application configuration directories (e.g. Discord)
- Any `.git` credential caches
- General cache directories

These are excluded because they can contain live secrets, and copying them into an unencrypted backup folder introduces risk without meaningful recovery benefit — GNOME's functional state is already fully captured by the dconf export and package manifests above.

## Restore procedure

If GNOME's configuration is damaged or altered unexpectedly during the Hyprland/DMS install process:

```bash
dconf load / < "$BACKUP_DIR/dconf-full-backup.ini"
```

This performs a full restore of dconf state, not a merge — it overwrites the current configuration with the backed-up one. Log out and back into the GNOME session afterward for all settings to take full effect.

This restore only touches the dconf database. It does not affect installed packages, NVIDIA drivers, CUDA, or development environments (Python/uv virtual environments, Node, etc.), which are entirely separate from this backup's scope.
