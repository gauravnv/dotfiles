# Pop!_OS Setup Notes — ASUS ROG Zephyrus G14 2025 (GA403WM)

**Date started:** December 2025  
**OS:** Pop!_OS 24.04 LTS  
**Desktop session:** COSMIC (Wayland)  

This document captures system-specific fixes and learnings for this machine. Some items are appropriate for automation via dotfiles, while others should remain manual due to being hardware- or environment-specific.

## Services

### asusd

**Source location:** `~/system-essentials/asusctl`  (You can put it wherever you like)
**Version:** 6.2.0 (as of January 2026)

A systemd unit override was required for `asusd` to start correctly due to the D-Bus bus name.

**File:** `/etc/systemd/system/asusd.service`  
**Change:** `BusName` set to `xyz.ljones.Asusd`

After modifying the service:
```bash
sudo systemctl daemon-reload
sudo systemctl restart asusd
```

#### Keyboard LED / Aura Support Fix

**Issue Reported: https://gitlab.com/asus-linux/asusctl/-/issues/710**

The GA403WM model was missing from the upstream `aura_support.ron`. Added manually:

**File:** `/usr/share/asusd/aura_support.ron`  
**Fix script:** `~/system-essentials/scripts/fix_aura_support.py`

This adds a `GA403W` entry with modes: Static, Breathe, Pulse, RainbowCycle, RainbowWave.

**Current status (GA403WM / 0b05:19b6):**
- Brightness control works via `/sys/class/leds/asus::kbd_backlight/brightness` (0-3)
- ROG Control Center / `asusctl` accepts mode + color changes and updates config
- Keyboard backlight does **not** change color on hardware (appears stuck in factory/out-of-box state)

This suggests the device is detected, but the HID RGB protocol used by `asusd` is not effective on this model (may require newer kernel support).

**Note:** After any asusctl update, you may need to reapply this fix until it's merged upstream. Run:
```bash
sudo python3 ~/system-essentials/scripts/fix_aura_support.py
sudo systemctl restart asusd
```

#### asus-armoury driver (Not Available)

ROG Control Center shows "The asus-armoury driver is not loaded". This driver is:
- Under development by flukejones
- Requires kernel patches from [github.com/flukejones/linux](https://github.com/flukejones/linux/tree/wip/ally-6.13)
- Provides advanced PPT/TDP tuning for newer laptops
- Not required for many basic features (fan curves, profiles, etc.), but may be required for full LED RGB control on GA403WM

#### Updating asusctl/supergfxctl

Check for updates and rebuild:
```bash
~/system-essentials/scripts/update-asusctl.sh
```

Or manually:
```bash
cd ~/system-essentials/asusctl
git fetch origin main
git log HEAD..origin/main --oneline  # See new commits
git pull origin main
make
sudo make install
sudo systemctl daemon-reload && sudo systemctl restart asusd
```

### supergfxctl

**Source location:** `~/system-essentials/supergfxctl`  
**Version:** 5.2.7 (as of January 2026)

GPU switching is handled via `supergfxctl`. GPU mode can be checked with:

```bash
supergfxctl -g
```

Example modes:
```bash
supergfxctl -m integrated
supergfxctl -m hybrid
supergfxctl -m dedicated
```

**DO NOT USE `ASUSMUXDGPU` MODE** — I accidentally enabled it and had to fight against black screens and boot issues to recover.

### system76-power

`system76-power` was masked to avoid conflicts with `supergfxctl` GPU management.

## Terminal

Kitty is used as the primary terminal.

- **Font:** JetBrains Mono Nerd Font
- **Theme:** Dracula (`~/.config/kitty/current-theme.conf`)

Notable key behavior:
- `Ctrl+C` uses Kitty’s `copy_or_interrupt` (copy if selection exists, otherwise SIGINT)
- `Ctrl+V` pastes from the clipboard

## App Scaling (COSMIC fractional scaling + Electron Flatpaks)

**My setup:** COSMIC (Wayland) with **175% display scaling** and **+10% additional scaling**.

**How I verified I was on Wayland:**

```bash
echo "$XDG_SESSION_TYPE"
```

**Problem I hit:** Some Electron apps (notably Discord + Obsidian) launched at a ridiculous scale (everything super-sized). The in-app zoom controls could make it usable, but those settings would reset on restart.

**What I found:** This is a common Linux pain point when Electron apps run via XWayland + fractional scaling. It can look like “double scaling” or inconsistent scaling between apps.

### What I tried

Because Discord and Obsidian are installed as Flatpaks, I tried forcing Electron to prefer Wayland rendering:

```bash
flatpak override --user --env=ELECTRON_OZONE_PLATFORM_HINT=wayland com.discordapp.Discord
flatpak override --user --env=ELECTRON_OZONE_PLATFORM_HINT=wayland md.obsidian.Obsidian
```

### What happened

- Discord: fixed the scaling immediately.
- Obsidian: stopped launching when I clicked it (it would fail early).

### What I found (root cause)

Obsidian was now trying to start with a Wayland backend, but the Flatpak did not have Wayland socket access enabled (it had `x11` but not `wayland`).

I confirmed permissions with:

```bash
flatpak info --show-permissions md.obsidian.Obsidian
flatpak override --user --show md.obsidian.Obsidian
```

### Fix

I explicitly granted Wayland socket access to the Obsidian Flatpak:

```bash
flatpak override --user --socket=wayland md.obsidian.Obsidian
```

After that, Obsidian launched normally again and scaling was sane.

### What I’d try next (if another Electron Flatpak acts weird)

These are the next things I’d personally check/try, in order:

```bash
# 1) Confirm whether the app is running under Wayland or XWayland
flatpak run --command=sh md.obsidian.Obsidian -lc 'echo "XDG_SESSION_TYPE=$XDG_SESSION_TYPE"; echo "WAYLAND_DISPLAY=$WAYLAND_DISPLAY"; echo "DISPLAY=$DISPLAY"'

# 2) Confirm the app has the right sockets
flatpak info --show-permissions <APP_ID>

# 3) Try explicit Wayland flags for Electron/Chromium apps (works for some apps)
flatpak run <APP_ID> --enable-features=UseOzonePlatform --ozone-platform=wayland

# 4) If the UI is still bizarre, try temporarily disabling GPU acceleration
flatpak run <APP_ID> --disable-gpu
```

### Undo (if needed)

If this causes regressions later, I can revert the overrides:

```bash
flatpak override --user --unset-env=ELECTRON_OZONE_PLATFORM_HINT com.discordapp.Discord
flatpak override --user --unset-env=ELECTRON_OZONE_PLATFORM_HINT md.obsidian.Obsidian

flatpak override --user --nosocket=wayland md.obsidian.Obsidian
```

## Audio Fix (Realtek ALC285)

**Problem:** The hardware mixer (Master/Speaker) behaves incorrectly. Volume adjustments can degrade audio quality instead of changing the volume level.

**Fix:** Force WirePlumber to use software volume instead of the hardware mixer.

**File:** `~/.config/wireplumber/wireplumber.conf.d/99-alsa-softvol.conf`

```lua
monitor.alsa.rules = [
  {
    matches = [
      { device.name = "~alsa_card.pci-0000_65_00.6.*" }
    ]
    actions = {
      update-props = {
        api.alsa.soft-mixer = true
      }
    }
  }
]
```

Apply by restarting WirePlumber:
```bash
systemctl --user restart wireplumber
```

## Wi‑Fi Stability (MediaTek MT7925e)

Driver instability was observed when roaming to 6GHz networks.

Mitigations:

1) Disable Wi‑Fi power management:

**File:** `/etc/NetworkManager/conf.d/99-wifi-powersave-off.conf`
```ini
[connection]
wifi.powersave = 2
```

Apply:
```bash
sudo systemctl restart NetworkManager
```

2) Pin a connection to a stable 5GHz band/BSSID (example):
```bash
sudo nmcli connection modify "<CONNECTION_NAME>" \
  802-11-wireless.bssid "<BSSID>" \
  802-11-wireless.band a \
  802-11-wireless.channel 44
```

## Fastfetch

Fastfetch is configured in `~/.config/fastfetch/config.jsonc`.

- Logo type: `kitty-direct`
- Image: `~/.config/fastfetch/images/cat-window.png`
- Logo sizing: width 60, height 15

To generate a compatible PNG logo image:
```bash
mkdir -p ~/.config/fastfetch/images
convert ~/Pictures/cat-window.jpg ~/.config/fastfetch/images/cat-window.png
```

## Notes on Fn / Extra Keys

Some Fn key combinations may not work depending on kernel/driver support and firmware behavior. If a function is missing, COSMIC custom shortcuts and small helper scripts can bridge gaps until upstream support improves.
