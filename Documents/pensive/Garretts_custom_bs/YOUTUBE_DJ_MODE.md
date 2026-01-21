# YouTube DJ Mode

**An automated window management system for the ultimate YouTube viewing experience on ultrawide monitors.**

---

## Overview

YouTube DJ Mode is a custom Hyprland setup that automatically positions Firefox Picture-in-Picture (PiP) videos in the center 16:9 region of your ultrawide monitor, moves the main Firefox window to the right side, and hides the center portion of your waybar for a clean, distraction-free viewing experience.

Perfect for DJing, music listening, or watching content while browsing.

---

## Features

### 🎯 Core Features
- **Auto-Positioning**: PiP window automatically centers to 16:9 region (2560×1440)
- **Firefox Management**: Main Firefox window floats and positions to the right
- **Waybar Integration**: Center modules (clock, network) hide during DJ mode
- **Smart Recovery**: When PiP closes, Firefox automatically returns to center
- **DENON Mirror**: Center 16:9 region mirrors to DENON-AVR display via `wl-mirror`

### 🎨 Visual Experience
- Full 2560×1440 PiP video in center region
- Firefox positioned at right side (1280×1440)
- Clean waybar (only left/right modules visible)
- No window overlaps or clutter

---

## Quick Start

### Enable DJ Mode
```bash
# Keyboard shortcut
SUPER + SHIFT + Y

# Or via system menu
SUPER + SPACE → Utils → YouTube DJ Mode
```

### Trigger PiP
1. Open a YouTube video in Firefox
2. Press `Ctrl + Shift + ]` to enter PiP mode
3. Everything auto-positions!

### Disable DJ Mode
```bash
# Same toggle
SUPER + SHIFT + Y
```

---

## How It Works

### 1. **Enable DJ Mode**
- Starts `youtube-dj-daemon` (monitors Hyprland events)
- Hides waybar center modules (via CSS injection)
- Notification confirms activation

### 2. **Trigger PiP** (`Ctrl + Shift + ]`)
- PiP window opens (Firefox feature)
- Daemon detects `openwindow` event
- PiP resizes to 2560×1440 and centers at X=1280
- Main Firefox window floats and moves to X=3840

### 3. **Close PiP** (navigate away or close video)
- Daemon detects `closewindow` event
- Firefox automatically returns to center position
- Resizes to 2560×1440 at X=1280

### 4. **Disable DJ Mode**
- Daemon stops
- Waybar center modules restore
- Manual window control returns

---

## Components

### Scripts

#### `/home/garrett/.local/bin/youtube-dj-daemon`
**Main event listener daemon**
- Monitors Hyprland socket for window events
- Parses `openwindow` and `closewindow` events
- Auto-positions PiP and Firefox windows
- Handles workspace synchronization

**Key Functions:**
- `handle_pip_open()` - Positions PiP to center, Firefox to right
- `handle_pip_close()` - Returns Firefox to center
- `position_firefox_center()` - Centers Firefox in 16:9 region
- `position_firefox_right()` - Positions Firefox to right side

#### `/home/garrett/.local/bin/youtube-dj-toggle`
**Toggle script for enabling/disabling DJ mode**
- Starts/stops `youtube-dj-daemon`
- Toggles waybar center visibility
- Sends desktop notifications

#### `/home/garrett/.local/bin/waybar-dj-mode`
**Waybar CSS injection script**
- `enable`: Adds `dj-mode.css` import to waybar
- `disable`: Removes DJ mode CSS
- Uses `SIGUSR2` to reload waybar without restart

#### `/home/garrett/.local/bin/pip-to-center`
**Manual positioning script**
- Keybind: `SUPER + SHIFT + F`
- Forces PiP to center 16:9 region
- Forces Firefox to float and position right
- Useful for manual corrections

#### `/home/garrett/.local/bin/mirror-center-to-avr`
**Display mirroring script**
- Mirrors center 2560×1440 region to DENON-AVR
- Uses `wl-mirror` with region selection
- Auto-starts on boot

### Configuration Files

#### `/home/garrett/.config/hypr/source/window_rules.conf`
```ini
# Firefox PiP window rules
windowrulev2 = float,class:^(firefox)$,title:^(Picture-in-Picture)$
windowrulev2 = size 2560 1440,class:^(firefox)$,title:^(Picture-in-Picture)$
windowrulev2 = move 1280 0,class:^(firefox)$,title:^(Picture-in-Picture)$
windowrulev2 = nodim,class:^(firefox)$,title:^(Picture-in-Picture)$
windowrulev2 = opaque,class:^(firefox)$,title:^(Picture-in-Picture)$
```

#### `/home/garrett/.config/hypr/source/keybinds.conf`
```ini
# Position PiP window to center 16:9 region (for DENON TV viewing)
bindld = $mainMod SHIFT, F, Position PiP to Center, exec, pip-to-center

# Toggle YouTube DJ Mode (auto-position PiP and Firefox)
bindld = $mainMod SHIFT, Y, Toggle YouTube DJ Mode, exec, youtube-dj-toggle
```

#### `/home/garrett/.config/hypr/source/autostart.conf`
```ini
# YouTube DJ Daemon - Auto-starts on boot
exec-once = uwsm-app -- sh -c "youtube-dj-daemon && waybar-dj-mode enable"

# Center Region Mirror - Mirrors to DENON-AVR
exec-once = uwsm-app -- mirror-center-to-avr
```

#### `/home/garrett/.config/waybar/dj-mode.css`
```css
/* Hides center waybar modules during DJ mode */
.modules-center {
    opacity: 0;
    margin: 0;
    padding: 0;
}
```

#### `/home/garrett/user_scripts/rofi/system_menu.sh`
- Added "YouTube DJ Mode" to Utils menu
- Triggers `youtube-dj-toggle` when selected

---

## Keybinds

| Keybind | Action | Description |
|---------|--------|-------------|
| `SUPER + SHIFT + Y` | Toggle DJ Mode | Enable/disable YouTube DJ Mode |
| `Ctrl + Shift + ]` | Trigger PiP | Firefox native PiP shortcut |
| `SUPER + SHIFT + F` | Manual Position | Force PiP and Firefox positioning |
| `SUPER + SPACE` | System Menu | Access DJ Mode via Utils menu |

---

## Technical Details

### Display Configuration
- **Monitor**: Samsung Odyssey G9 (5120×1440, 32:9, 240Hz)
- **Center Region**: 2560×1440 (16:9) at X=1280
- **Right Region**: 1280×1440 at X=3840
- **DENON Mirror**: DP-4 output via `wl-mirror`

### Window Geometry
```
┌─────────────────────────────────────────────────────────┐
│                    5120×1440 Ultrawide                  │
├──────────────┬────────────────────────┬─────────────────┤
│   Left       │   Center (PiP)         │  Right (Firefox)│
│   1280px     │   2560×1440            │     1280×1440   │
│              │   X=1280, Y=0          │   X=3840, Y=0   │
│              │                        │                 │
└──────────────┴────────────────────────┴─────────────────┘
```

### Event Flow
1. User presses `Ctrl+Shift+]` in Firefox
2. Hyprland emits: `openwindow>>ADDRESS,WORKSPACE,firefox,Picture-in-Picture`
3. `youtube-dj-daemon` parses event via `socat`
4. Daemon executes `hyprctl dispatch` commands:
   - `resizewindowpixel exact 2560 1440,address:$pip_addr`
   - `movewindowpixel exact 1280 0,address:$pip_addr`
   - `togglefloating address:$firefox_addr`
   - `resizewindowpixel exact 1280 1440,address:$firefox_addr`
   - `movewindowpixel exact 3840 0,address:$firefox_addr`

### Dependencies
- **Hyprland** - Wayland compositor
- **Firefox** - Browser with PiP support
- **waybar** - Status bar
- **wl-mirror** - Wayland screen mirroring
- **socat** - Socket communication
- **jq** - JSON parsing
- **uwsm** - Session manager

---

## Troubleshooting

### PiP Not Auto-Positioning
**Check daemon status:**
```bash
ps aux | grep youtube-dj-daemon
```

**View daemon logs:**
```bash
tail -f /tmp/youtube-dj-daemon.log
```

**Restart daemon:**
```bash
pkill -f youtube-dj-daemon
youtube-dj-daemon > /tmp/youtube-dj-daemon.log 2>&1 &
```

### Firefox Not Moving to Right
- Ensure Firefox is the active window when PiP opens
- PiP and main Firefox must share same PID
- Check if Firefox is already floating (daemon only toggles once)

### Waybar Center Not Hiding
**Check CSS import:**
```bash
grep "dj-mode.css" ~/.config/waybar/horizontal_nerdy_modern/style.css
```

**Manual toggle:**
```bash
waybar-dj-mode enable
killall -SIGUSR2 waybar
```

### PiP Visible on All Workspaces
- This is Firefox's default PiP behavior (intentional)
- PiP windows are "sticky" across workspaces by design

### DENON Mirror Not Working
**Check wl-mirror process:**
```bash
ps aux | grep wl-mirror
```

**Restart mirror:**
```bash
pkill -f "wl-mirror.*DP-4"
mirror-center-to-avr
```

---

## Advanced Configuration

### Adjust PiP Size/Position
Edit `youtube-dj-daemon` and `pip-to-center`:
```bash
# Change dimensions (default: 2560×1440)
hyprctl dispatch resizewindowpixel exact WIDTH HEIGHT,address:$pip_addr

# Change position (default: X=1280, Y=0)
hyprctl dispatch movewindowpixel exact X Y,address:$pip_addr
```

### Adjust Firefox Position
Edit `youtube-dj-daemon`:
```bash
# Right side (default: X=3840, 1280×1440)
hyprctl dispatch resizewindowpixel exact 1280 1440,address:$firefox_addr
hyprctl dispatch movewindowpixel exact 3840 0,address:$firefox_addr
```

### Custom Waybar Hiding
Edit `/home/garrett/.config/waybar/dj-mode.css`:
```css
/* Hide specific modules */
#clock { opacity: 0; }
#custom-net-speed { opacity: 0; }

/* Or hide entire center section */
.modules-center { opacity: 0; }
```

### Change Check Interval
Edit `youtube-dj-daemon` (default: polls every 0.1s via `socat`):
- Uses event-driven architecture (no polling needed)
- Reacts instantly to `openwindow`/`closewindow` events

---

## Related Documentation

- **GPU Setup**: `GPU_SETUP_README.md` - Dual NVIDIA GPU configuration
- **DENON Mirror**: `DENON_MIRROR_SETUP.md` - Center region mirroring
- **Audio Config**: WirePlumber custom device naming

---

## Created

**Date**: January 16, 2026  
**Author**: Arch System Architect (AI) + User Collaboration  
**System**: Arch Linux + Hyprland + NVIDIA

---

## Notes

- DJ Mode starts automatically on boot (configured in autostart.conf)
- PiP positioning is workspace-aware (stays on source workspace)
- Firefox must be the active window for PiP to trigger correctly
- Waybar center modules hide via CSS (not position change)
- Compatible with master layout for ultrawide tiling

---

## Future Enhancements

- [ ] Browser extension for auto-PiP (explored but deemed unnecessary)
- [ ] Custom audio routing to DENON when DJ mode active
- [ ] Workspace-specific DJ mode activation
- [ ] OSD showing DJ mode status
- [ ] Integration with media controls
