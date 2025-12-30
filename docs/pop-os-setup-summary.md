# Pop!_OS Setup Summary - ASUS ROG Zephyrus G14 2025

**Date:** December 24, 2025  
**Machine:** ASUS ROG Zephyrus G14 2025 (GA403WM)  
**OS:** Pop!_OS 24.04 LTS  
**Kernel:** 6.17.9-76061709-generic

---

## Hardware Specifications

| Component | Details |
|-----------|---------|
| CPU | AMD Ryzen AI 9 HX 370 (12 cores / 24 threads) |
| GPU (Integrated) | AMD Radeon 890M |
| GPU (Discrete) | NVIDIA GeForce RTX 5060 Laptop |
| RAM | 32GB LPDDR5X |
| Storage | 929GB NVMe |
| Audio | Realtek ALC285 (via PipeWire 1.4.2) |

---

## System Services Status

### Working Services

| Service | Version | Status | Notes |
|---------|---------|--------|-------|
| `asusd` | 6.2.0 | ✅ Running | Required BusName fix |
| `supergfxd` | 5.2.7 | ✅ Running | GPU switching works |

### Disabled/Masked Services

| Service | Reason |
|---------|--------|
| `asusd-user` | Crashes due to missing Aura layout file and DBus issues |
| `system76-power` | Masked - conflicts with supergfxctl GPU management, was turning on dGPU |

**Note:** Use `supergfxctl` for GPU mode switching, not system76-power. The COSMIC Settings "Power mode" section will show "Backend not found" - this is expected.

---

## Configuration Changes Made

### 1. asusd Service Fix

**File:** `/etc/systemd/system/asusd.service`

**Problem:** Service was failing to start due to incorrect BusName.

**Solution:** Changed BusName from `org.asuslinux.Daemon` to `xyz.ljones.Asusd`

```ini
[Unit]
Description=ASUS ROG Daemon
After=dbus.service

[Service]
Type=dbus
BusName=xyz.ljones.Asusd
ExecStartPre=/bin/sleep 1
ExecStart=/usr/bin/asusd
TimeoutSec=10
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

**Commands used:**
```bash
sudo systemctl daemon-reload
sudo systemctl restart asusd
```

---

### 2. Terminal Setup - Kitty

**Installed via:** APT

**Theme:** Dracula (selected via `kitty +kitten themes`)

**Font:** JetBrains Mono Nerd Font

**Config file:** `~/.config/kitty/kitty.conf`

**Key customizations:**
- `Ctrl+C` → copy_or_interrupt (copies if selection, otherwise sends SIGINT)
- `Ctrl+V` → paste from clipboard
- `Ctrl+X` → send SIGINT (alternative kill)
- `F11` → toggle fullscreen
- Splits enabled with `Ctrl+Shift+Enter`

**Font installation:**
```bash
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
curl -fLO https://github.com/ryanoasis/nerd-fonts/releases/download/v3.3.0/JetBrainsMono.zip
unzip JetBrainsMono.zip -d JetBrainsMono
rm JetBrainsMono.zip
fc-cache -fv
```

---

### 3. Shell Setup - Zsh + Oh My Zsh + Powerlevel10k

**Installed:**
- Zsh (via APT)
- Oh My Zsh (via install script)
- Powerlevel10k theme

**Config file:** `~/.zshrc`

**Plugins enabled:**
- `git`
- `zsh-autosuggestions`
- `zsh-syntax-highlighting`
- `zsh-completions`

**Plugin installation:**
```bash
# Autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Syntax highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Completions
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions
```

**Powerlevel10k installation:**
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

**Note:** Run `p10k configure` to customize the prompt.

**To set Zsh as default shell:**
```bash
chsh -s /usr/bin/zsh
```

---

### 4. Browser - Vivaldi

**Installed via:** APT (official Vivaldi repository)

**Repository setup:**
```bash
curl -fsSL https://repo.vivaldi.com/archive/linux_signing_key.pub | sudo gpg --dearmor -o /usr/share/keyrings/vivaldi-browser.gpg
echo "deb [signed-by=/usr/share/keyrings/vivaldi-browser.gpg arch=$(dpkg --print-architecture)] https://repo.vivaldi.com/archive/deb/ stable main" | sudo tee /etc/apt/sources.list.d/vivaldi-archive.list
sudo apt update && sudo apt install vivaldi-stable
```

**Note:** Flatpak version was installed accidentally and was removed:
```bash
flatpak uninstall com.vivaldi.Vivaldi
```

---

### 5. Audio - EasyEffects

**Installed via:** Flatpak

```bash
flatpak install flathub com.github.wwmm.easyeffects
```

**Purpose:** Audio enhancement and equalization for the laptop speakers.

---

### 5a. Audio Fix - Broken Hardware Mixer (Critical)

**Hardware:** Realtek ALC285 audio codec  
**Issue:** Hardware mixer controls (Master/Speaker) are broken on ASUS ROG Zephyrus G14 2025. Adjusting volume degrades audio quality instead of properly changing volume level.

**Symptoms:**
- Volume at 0%: Correctly mutes
- Volume at 1-100%: Same loud volume, but lowering volume causes audio quality degradation (sounds like reduced bit depth/sample rate)
- Only the PCM channel in `alsamixer` works correctly

**Solution:** Configure WirePlumber to use software mixer instead of hardware mixer.

**File created:** `~/.config/wireplumber/wireplumber.conf.d/99-alsa-softvol.conf`

```lua
# Fix for ASUS ROG Zephyrus G14 2025 (GA403WM) audio issue
# The hardware mixer (Master/Speaker) is broken - adjusting it degrades audio quality
# instead of changing volume. This forces WirePlumber to use software volume control.
# Reference: https://bbs.archlinux.org/viewtopic.php?id=310368

monitor.alsa.rules = [
  {
    matches = [
      {
        # Match the Realtek ALC285 audio device
        device.name = "~alsa_card.pci-0000_65_00.6.*"
      }
    ]
    actions = {
      update-props = {
        # Use software mixer - leaves hardware mixer untouched
        api.alsa.soft-mixer = true
      }
    }
  }
]
```

**Apply the fix:**
```bash
# Create config directory
mkdir -p ~/.config/wireplumber/wireplumber.conf.d

# Create the config file (content above)
# Then restart WirePlumber
systemctl --user restart wireplumber
```

**Result:** Volume control now works perfectly without audio quality degradation.

**Tradeoffs and Technical Impact:**

Using software mixer instead of hardware mixer has these implications:

| Aspect | Hardware Mixer | Software Mixer (Configured Fix) |
|--------|----------------|--------------------------|
| **CPU Usage** | None (dedicated audio chip) | Minimal (negligible on Ryzen AI 9) |
| **Latency** | Lowest possible | +few microseconds (imperceptible) |
| **Power Consumption** | Slightly lower | Slightly higher (unmeasurable in practice) |
| **Volume Control** | Broken on this hardware | Works perfectly |
| **Per-App Volume** | Not available | Available (PipeWire feature) |
| **Reliability** | Depends on driver quality | Consistent across devices |

**Why this is the right choice:**
1. **The hardware mixer is broken** - it causes audio quality degradation, making it unusable
2. **Performance impact is negligible** - software volume control is extremely lightweight, and the Ryzen AI 9 HX 370 has more than enough power
3. **Better functionality** - PipeWire's software mixer provides features like per-application volume control
4. **No perceptible quality loss** - modern software mixers operate at high precision (32-bit float)

**What was given up:** In theory, offloading volume control to dedicated audio hardware saves a tiny amount of CPU cycles and power. In practice, on this laptop with 12 cores and modern power management, the difference is unmeasurable.

**What was gained:** Working volume control without audio degradation, plus PipeWire's advanced mixing features.

**Reference:** [Arch Linux Forums - Same Issue & Solution](https://bbs.archlinux.org/viewtopic.php?id=310368)

---

### 6. GPU Configuration

**Current Mode:** Integrated (NVIDIA dGPU powered off)

**Check status:**
```bash
supergfxctl -g
```

**Switch modes:**
```bash
supergfxctl -m integrated  # Battery saving, dGPU off
supergfxctl -m hybrid      # Auto-switch, dGPU available on demand
supergfxctl -m dedicated   # Always use NVIDIA
```

**Preference:** Hybrid mode for daily use, with dGPU available for LLM work.

---

## Known Limitations

### Critical: Missing asus-armoury Kernel Module

**Impact:** The `asus-armoury` kernel module is **NOT in the mainline Linux kernel** yet. It's currently being upstreamed.

**Affected functionality:**
- M1-M4 macro keys only send generic KEY_PROG1-4, cannot be fully customized through asusd
- Some Fn key combinations may not work as expected
- ROG key functionality may be limited

**Kernel modules currently loaded:**
- `asus_nb_wmi` ✅
- `hid_asus` ✅
- `asus_wmi` ✅
- `asus_armoury` ❌ (not available)

**Workaround:** Use `keyd` or other key remapping tools to bind M1-M4 (KEY_PROG1-4) to desired actions at the userspace level.

### Fn Key Behavior

**Current state:** Fn keys send standard F1-F12 codes (not special function codes).

**evtest findings on /dev/input/event7:**
| Key Pressed | Code Sent |
|-------------|-----------|
| F2 (Mute) | KEY_F2 (60) |
| F3 (Vol Down) | KEY_F3 (61) |
| F4 (Vol Up) | KEY_F4 (62) |
| F5 (Brightness Down) | KEY_F5 (63) |
| F6 (Brightness Up) | KEY_F6 (64) |
| M4 | KEY_PROG1 (148) |

**Fn lock state:** `/sys/devices/platform/asus-nb-wmi/fnlock_default` = Y

---

## Pending Tasks

### Installed Tooling

| Tool | Purpose | Notes |
|------|---------|-------|
| Python tooling (`uv`) | Development | Preferred for fast env + dependency management |
| Node.js version manager (`fnm`) | Development | Used for Node/TypeScript workflows |
| GitHub CLI (`gh`) | Git workflow | Installed via APT |
| Podman | Container runtime | Installed; Docker may also be present |

### Not Yet Configured

| Task | Notes |
|------|-------|
| Copilot key remapping | Plan: Use `keyd` to remap to Super key |
| Screenshot shortcuts | Need to configure Fn+F10 or alternative |
| Screencast/screen sharing | F9 key functionality |
| Keyboard layout | Currently `ca eng` (Canadian English) |

---

## Preferences Summary

| Category | Preference |
|----------|------------|
| GPU Usage | Hybrid mode (dGPU for LLMs, otherwise integrated) |
| Battery Priority | Important but not aggressive |
| Primary Languages | Python, TypeScript |
| Container Runtime | Podman (preferred) |
| Browser | Vivaldi (Android sync, privacy-focused) |
| Terminal | Kitty with Dracula theme |
| Shell | Zsh with Powerlevel10k |
| Font | JetBrains Mono Nerd Font |

---

## Useful Commands Reference

### ASUS Control
```bash
asusctl -c 80              # Set charge limit to 80%
asusctl profile -P quiet   # Set to quiet profile
asusctl led-mode static    # Keyboard backlight mode
supergfxctl -g             # Check GPU mode
supergfxctl -m hybrid      # Switch to hybrid mode
```

### System Info
```bash
asusctl -v                 # asusd version
supergfxctl -V             # supergfxd version
uname -r                   # Kernel version
lspci | grep -i nvidia     # Check NVIDIA GPU
lspci | grep -i amd        # Check AMD GPU
```

### Service Management
```bash
systemctl status asusd
systemctl status supergfxd
journalctl -u asusd -f     # Follow asusd logs
```

---

## Files Modified/Created

| File | Purpose |
|------|---------|
| `/etc/systemd/system/asusd.service` | Fixed asusd service configuration |
| `/etc/NetworkManager/conf.d/99-wifi-powersave-off.conf` | Wi-Fi power management disabled |
| `~/.config/wireplumber/wireplumber.conf.d/99-alsa-softvol.conf` | Audio fix - software mixer for broken hardware |
| `~/.config/fastfetch/config.jsonc` | Fastfetch system info configuration |
| `~/.config/fastfetch/images/cat-window.png` | Fastfetch logo image |
| `~/.config/kitty/kitty.conf` | Kitty terminal configuration |
| `~/.zshrc` | Zsh shell configuration |
| `~/.p10k.zsh` | Powerlevel10k prompt configuration |
| `~/.local/share/fonts/JetBrainsMono/` | Nerd Font installation |

---

## 7. Fastfetch Configuration

**Installed via:** APT

**Config file:** `~/.config/fastfetch/config.jsonc`

**Logo setup:**
- **Type:** `kitty-direct` (most stable for Kitty terminal)
- **Source image:** `~/.config/fastfetch/images/cat-window.png`
- **Original:** `~/Pictures/cat-window.jpg` (converted to PNG for compatibility)
- **Dimensions:** 736×411 pixels, displayed at width 60, height 15
- **Conversion command:**
  ```bash
  mkdir -p ~/.config/fastfetch/images
  convert ~/Pictures/cat-window.jpg ~/.config/fastfetch/images/cat-window.png
  ```

**Layout:** Boxed sections with color-coded categories:
- **System** (cyan): OS, Kernel, Uptime, Packages, Shell
- **Desktop** (magenta): DE, WM, Terminal, Terminal Font
- **Hardware** (yellow): CPU, GPU, Memory, Disk, Battery
- **Network** (green): Local IP, WiFi

**Key features:**
- Clean icon-based keys (no text labels, just Nerd Font icons)
- Battery shows capacity and time remaining: `{capacity} ({time-formatted} remaining)`
- Fixed glyphs: WM icon (), Disk icon (󰋊)
- Color palette displayed at bottom

**Complete config:**
```jsonc
{
  "$schema": "https://github.com/fastfetch-cli/fastfetch/raw/dev/doc/json_schema.json",
  "logo": {
    "type": "kitty-direct",
    "source": "~/.config/fastfetch/images/cat-window.png",
    "preserveAspectRatio": true,
    "width": 60,
    "height": 15,
    "padding": {
      "top": 1,
      "left": 2,
      "right": 3
    }
  },
  "display": {
    "separator": " ",
    "color": {
      "keys": "magenta",
      "title": "cyan"
    }
  },
  "modules": [
    {
      "type": "title",
      "format": "{#cyan}{user-name}{#0}@{#magenta}{host-name}"
    },
    {
      "type": "separator",
      "string": "━"
    },
    {
      "type": "custom",
      "format": "{#90}╭─────────────── {#cyan}System{#90} ────────────────╮"
    },
    {
      "type": "os",
      "key": "{#cyan}│{#0}    ",
      "format": "{name} {version}"
    },
    {
      "type": "kernel",
      "key": "{#cyan}│{#0}    "
    },
    {
      "type": "uptime",
      "key": "{#cyan}│{#0}   󰅐 "
    },
    {
      "type": "packages",
      "key": "{#cyan}│{#0}   󰏖 "
    },
    {
      "type": "shell",
      "key": "{#cyan}│{#0}    "
    },
    {
      "type": "custom",
      "format": "{#90}╰─────────────────────────────────────╯"
    },
    {
      "type": "custom",
      "format": "{#90}╭────────────── {#magenta}Desktop{#90} ───────────────╮"
    },
    {
      "type": "de",
      "key": "{#magenta}│{#0}   󰧨 "
    },
    {
      "type": "wm",
      "key": "{#magenta}│{#0}    "
    },
    {
      "type": "terminal",
      "key": "{#magenta}│{#0}    "
    },
    {
      "type": "terminalfont",
      "key": "{#magenta}│{#0}    "
    },
    {
      "type": "custom",
      "format": "{#90}╰─────────────────────────────────────╯"
    },
    {
      "type": "custom",
      "format": "{#90}╭────────────── {#yellow}Hardware{#90} ──────────────╮"
    },
    {
      "type": "cpu",
      "key": "{#yellow}│{#0}   󰻠 ",
      "format": "{name}"
    },
    {
      "type": "gpu",
      "key": "{#yellow}│{#0}   󰢮 ",
      "format": "{name}"
    },
    {
      "type": "memory",
      "key": "{#yellow}│{#0}   󰍛 "
    },
    {
      "type": "disk",
      "key": "{#yellow}│{#0}   󰋊 ",
      "folders": "/"
    },
    {
      "type": "battery",
      "key": "{#yellow}│{#0}   󰁹 ",
      "format": "{capacity} ({time-formatted} remaining)"
    },
    {
      "type": "custom",
      "format": "{#90}╰─────────────────────────────────────╯"
    },
    {
      "type": "custom",
      "format": "{#90}╭────────────── {#green}Network{#90} ───────────────╮"
    },
    {
      "type": "localip",
      "key": "{#green}│{#0}   󰩟 "
    },
    {
      "type": "wifi",
      "key": "{#green}│{#0}   󰖩 "
    },
    {
      "type": "custom",
      "format": "{#90}╰─────────────────────────────────────╯"
    },
    {
      "type": "colors",
      "paddingLeft": 1,
      "symbol": "circle"
    }
  ]
}
```

**Quick alias:** Added `alias ff='fastfetch'` to `~/.zshrc`

---

## 8. Wi-Fi Stability Fixes (Critical)

### Problem: mt7925e Driver Instability

**Hardware:** MediaTek MT7925e Wi-Fi 7 card  
**Kernel:** 6.17.9-76061709-generic  
**Issue:** Kernel hang when roaming to 6GHz networks, causing NetworkManager to freeze and system reboot to hang at Pop!_OS logo

**Symptoms from logs:**
```
kernel: INFO: task NetworkManager:1265 blocked for more than 122 seconds
kernel: mt7925_mac_link_sta_remove+0x80/0x2a0 [mt7925_common]
NetworkManager: device (wlp99s0): link timed out
NetworkManager: Activation: failed for connection '<SSID>' (reason 'ssid-not-found')
```

### Solution 1: Disable Wi-Fi Power Management

**File created:** `/etc/NetworkManager/conf.d/99-wifi-powersave-off.conf`

```ini
[connection]
wifi.powersave = 2
```

**Purpose:** Prevents Wi-Fi card from entering low-power states that can cause disconnections  
**Performance impact:** None (only affects latency/stability, not throughput)

**Apply changes:**
```bash
sudo systemctl restart NetworkManager
```

### Solution 2: Pin Connection to Stable 5GHz Band

**Issue:** Driver has known bugs with 6GHz roaming (reported in upstream bugs)  
**Network:** "<SSID>" SSID with multiple BSSIDs (2.4GHz, 5GHz, 6GHz)

**Applied settings:**
```bash
# Pin to 5GHz BSSID at 5220 MHz (channel 44, non-DFS)
sudo nmcli connection modify "<SSID>" \
  802-11-wireless.bssid "<BSSID>" \
  802-11-wireless.band a \
  802-11-wireless.channel 44

# Reconnect
sudo nmcli connection down "<SSID>"
sudo nmcli connection up "<SSID>"
```

**Verify current connection:**
```bash
nmcli -f IN-USE,BSSID,FREQ,SSID,SIGNAL dev wifi list ifname wlp99s0
```

**Performance trade-off:**
- ✅ Stable connection (no more kernel hangs)
- ✅ 5GHz signal strength: 100% (vs 6GHz at 75%)
- ⚠️ No 6GHz access until driver bugs are fixed
- ⏱️ 6GHz potentially ~20-30% faster in ideal conditions, but unusable with current driver

**Future:** When kernel with mt7925e 6GHz fixes is available, remove constraints:
```bash
sudo nmcli connection modify "<SSID>" \
  802-11-wireless.bssid "" \
  802-11-wireless.band "" \
  802-11-wireless.channel ""
```

### Check Connection Settings

```bash
# View all connection settings
nmcli connection show "<SSID>" | grep -i "wireless"

# Current active connection
nmcli device show wlp99s0
```

---

## 9. Shell Enhancements

### Ripgrep Integration

**Added to `~/.zshrc`:**
```bash
alias grep='rg'
```

**Purpose:** Use ripgrep (faster, smarter grep) as default search tool  
**Installed via:** APT (`ripgrep` package)

**Verify:**
```bash
grep --version  # Should show ripgrep version
```

---

## Next Steps for Clean Install

1. **Fix audio** (create WirePlumber software mixer config)
2. **Apply Wi‑Fi stability fixes** (disable powersave + optionally pin to a stable 5GHz BSSID)
3. **Install Vivaldi** (optional)
4. **Install Kitty** and configure theme/font
5. **Install JetBrains Mono Nerd Font**
6. **Install Zsh + Oh My Zsh + Powerlevel10k**
7. **Install fastfetch** and configure with custom layout
8. **Install EasyEffects** (optional)
9. **Configure GPU mode** with `supergfxctl`
10. **Install dev tools** (uv, fnm, gh, Podman/Docker)
11. **Optional:** set up key remapping tools for Copilot/M-keys

---

*Document last updated: December 26, 2025*
