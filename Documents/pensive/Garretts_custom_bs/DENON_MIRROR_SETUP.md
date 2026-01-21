# DENON-AVR Center Region Mirror Setup

## What This Does

Mirrors just the **center 16:9 region** (2560x1440 area) of your ultrawide Samsung Odyssey to your DENON-AVR display, eliminating black bars.

## How It Works

1. **DP-4 (DENON-AVR)** is configured as a separate display at 1920x1080@60Hz
2. **wl-mirror** captures the center region: X=1280, Y=0, Size=2560x1440
3. Window automatically appears on DP-4 (workspace 10) in fullscreen
4. Starts automatically on login via autostart

## Files Modified

### Monitor Configuration
- `~/.config/hypr/source/monitors.conf`
  - DP-4 configured as separate display at position 0x-1080 (top left corner)
  - Workspace 10 bound to DP-4 monitor

### Window Rules  
- `~/.config/hypr/source/window_rules.conf`
  - wl-mirror window automatically goes to workspace 10
  - Automatically set to fullscreen

### Autostart
- `~/.config/hypr/source/autostart.conf`
  - Launches `mirror-center-to-avr` on boot

### Scripts Created
- `~/.local/bin/mirror-center-to-avr`
  - Kills any existing wl-mirror instances
  - Starts wl-mirror with region selection
  - Region: 1280,0 2560x1440 (center 16:9 of ultrawide)
- `~/.local/bin/mirror-toggle`
  - Toggle script for easy on/off control
  - Bound to **SUPER + M** keybind

## Manual Control

### Toggle mirroring (Keybind):
- **SUPER + M** - Toggle mirror on/off

### Toggle mirroring (Command):
```bash
mirror-toggle
```

### Start mirroring:
```bash
mirror-center-to-avr
```

### Stop mirroring:
```bash
pkill -f wl-mirror
```

### Check if running:
```bash
ps aux | grep wl-mirror
```

## Troubleshooting

### If AVR display is black:
1. Check if wl-mirror is running: `ps aux | grep wl-mirror`
2. Check if window is on DP-4: `hyprctl clients | grep -A 5 wl_mirror`
3. Manually move to workspace 3: `hyprctl dispatch focuswindow at.yrlf.wl_mirror && hyprctl dispatch movecurrentworkspacetomonitor DP-4`

### If mirror window appears on main display:
```bash
# Get window address
ADDR=$(hyprctl clients -j | jq -r '.[] | select(.class == "at.yrlf.wl_mirror") | .address')

# Move to workspace 10 (which is on DP-4)
hyprctl dispatch focuswindow address:$ADDR
hyprctl dispatch movecurrentworkspacetomonitor DP-4
hyprctl dispatch fullscreen address:$ADDR
```

### If you need to adjust the region:
Edit `~/.local/bin/mirror-center-to-avr` and change the `-r "X,Y WIDTHxHEIGHT"` values:
- Current: `-r "1280,0 2560x1440"` (center 16:9)
- X offset: (5120 - desired_width) / 2
- Y offset: usually 0
- Width: desired horizontal resolution
- Height: 1440 (your monitor height)

## Protection Against Accidental Interaction

To prevent windows and your mouse from accidentally interfering with the mirror display, the following protections are in place:

### Workspace 10 Keybind Protection
All keybinds for workspace 10 are disabled in `~/.config/hypr/source/keybinds.conf`:
- `SUPER+0` (switch to workspace 10) - **DISABLED**
- `SUPER+SHIFT+0` (move window to workspace 10) - **DISABLED**
- `SUPER+ALT+0` (move window silently to workspace 10) - **DISABLED**

### Window Rules Protection
A special window rule in `~/.config/hypr/source/window_rules.conf` prevents any non-mirror windows from being placed on workspace 10 or DP-4.

### Mouse Cursor
The mouse can still reach DP-4 by moving to the top left corner, but since workspace 10 is inaccessible via keybinds and the monitor is positioned above your main display (0x-1080), accidental cursor movement is very unlikely during normal use.

**Note**: While this isn't "true mirroring" (Wayland/Hyprland doesn't support that natively), these protections make workspace 10 effectively invisible to your normal workflow while keeping the mirror functional.

## Uninstalling

To remove the mirror functionality:

1. Comment out autostart line in `~/.config/hypr/source/autostart.conf`:
   ```
   #exec-once = uwsm-app -- mirror-center-to-avr
   ```

2. Remove window rules from `~/.config/hypr/source/window_rules.conf`:
   ```
   # Comment out the DENON-AVR MIRROR DISPLAY PROTECTION section
   ```

3. Remove workspace binding from `~/.config/hypr/source/monitors.conf`:
   ```
   # Comment out: workspace=10, monitor:DP-4
   ```

4. Re-enable workspace 10 keybinds in `~/.config/hypr/source/keybinds.conf`

5. Reload: `hyprctl reload`

---

**Created**: 2026-01-16
**Updated**: 2026-01-20
**Purpose**: Mirror center 16:9 region to DENON-AVR without black bars
