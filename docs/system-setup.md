# Pop!_OS Setup Notes — ASUS ROG Zephyrus G14 2025 (GA403WM)

**Date started:** December 2025  
**OS:** Pop!_OS 24.04 LTS  
**Desktop session:** COSMIC (Wayland)  

This document captures system-specific fixes and learnings for this machine. Some items are appropriate for automation via dotfiles, while others should remain manual due to being hardware- or environment-specific.

## Services

### asusd

A systemd unit override was required for `asusd` to start correctly due to the D-Bus bus name.

**File:** `/etc/systemd/system/asusd.service`  
**Change:** `BusName` set to `xyz.ljones.Asusd`

After modifying the service:
```bash
sudo systemctl daemon-reload
sudo systemctl restart asusd
```

### supergfxctl

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

### system76-power

`system76-power` was masked to avoid conflicts with `supergfxctl` GPU management.

## Terminal

Kitty is used as the primary terminal.

- **Font:** JetBrains Mono Nerd Font
- **Theme:** Dracula (`~/.config/kitty/current-theme.conf`)

Notable key behavior:
- `Ctrl+C` uses Kitty’s `copy_or_interrupt` (copy if selection exists, otherwise SIGINT)
- `Ctrl+V` pastes from the clipboard

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
