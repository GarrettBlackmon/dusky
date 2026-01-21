# Dual NVIDIA GPU Configuration - Setup Guide

## Your Hardware Configuration

| Component | Model | PCI Address | DRM Device | Purpose |
|-----------|-------|-------------|------------|---------|
| Gaming GPU | RTX 5090 | 01:00.0 | card1 (renderD128) | High-performance rendering |
| Display GPU | T400 4GB | 0a:00.0 | card2 (renderD129) | Display output (monitor connected here) |
| iGPU | AMD Radeon (Granite Ridge) | 71:00.0 | card0 (renderD130) | Unused |

**Monitor Connection**: Samsung Odyssey G95SC → T400 DP-5 (5120x1440@120Hz)

## The Problem You Had

### Issue #1: Display/Render Mismatch
Your Hyprland config was forcing the compositor to use the **5090** for display output via:
```bash
WLR_DRM_DEVICES=/dev/dri/by-path/pci-0000:01:00.0-card  # Wrong! This is the 5090
```

But your monitor is **physically connected to the T400**. This created a situation where:
1. Hyprland tried to render on the 5090
2. Frames had to be copied to the T400 for display output
3. This copy process was slow and inefficient
4. Result: 30fps at 1440p instead of hundreds

### Issue #2: Crude GPU Selection
Your Vulkan implicit layer forced ALL applications to use the 5090, which:
- Prevented per-application GPU selection
- Made the system less flexible
- Didn't help with OpenGL applications

### Issue #3: VRR Disabled
Variable Refresh Rate was turned off, causing potential screen tearing and stuttering.

## What Was Fixed

### ✅ 1. Corrected Display Configuration
**Files Changed:**
- `~/.config/hypr/source/environment_variables.conf`
- `~/.config/uwsm/env-hyprland`

**What Changed:**
```bash
# OLD (WRONG):
WLR_DRM_DEVICES=/dev/dri/by-path/pci-0000:01:00.0-card  # 5090

# NEW (CORRECT):
WLR_DRM_DEVICES=/dev/dri/by-path/pci-0000:0a:00.0-card  # T400
AQ_DRM_DEVICES=/dev/dri/by-path/pci-0000:0a:00.0-card   # T400 (for Aquamarine)
```

Now Hyprland correctly uses the T400 for display output where your monitor is connected.

### ✅ 2. Per-Application GPU Selection
**Created New Tools:**
- `gpu-5090 <command>` - Force application to use RTX 5090 (gaming)
- `gpu-t400 <command>` - Force application to use T400 (low-power)
- `gpu-info` - Comprehensive GPU diagnostics
- Removed the crude Vulkan implicit layer

**Usage Examples:**
```bash
# Launch a game with the 5090
gpu-5090 steam
gpu-5090 lutris
gpu-5090 gamescope -- %command%

# Test which GPU is being used
gpu-5090 vkcube        # Should show RTX 5090
gpu-5090 glxgears      # Should show RTX 5090
```

### ✅ 3. Enabled VRR (Variable Refresh Rate)
**File:** `~/.config/hypr/source/monitors.conf`
```
vrr = 1  # Was: 0
```

## How This Setup Works Now

### Desktop/Compositor (Low Power)
- Hyprland runs on the **T400**
- Regular desktop apps use the **T400**
- Low power consumption when idle
- Monitor displays through T400 (native, no copying)

### Gaming (High Performance)
- Launch games with `gpu-5090 <command>`
- Games render on the **5090**
- Frames are presented to T400 for display
- Full 5090 power available

### VM Passthrough
Your existing scripts still work:
- `gpu-detach` - Unbind 5090 for VM use
- `gpu-attach` - Rebind 5090 to host
- `gpu-status` - Check passthrough status

## Testing Your Configuration

### 1. Verify Display Setup
```bash
gpu-info
```
Should show:
- Monitor connected to T400 (card2-DP-5)
- WLR_DRM_DEVICES pointing to T400
- Both GPUs detected in Vulkan

### 2. Test OpenGL with 5090
```bash
gpu-5090 glxinfo | grep "OpenGL renderer"
```
Should output: `OpenGL renderer string: NVIDIA GeForce RTX 5090`

### 3. Test Vulkan with 5090
```bash
gpu-5090 vulkaninfo --summary | grep deviceName | head -1
```
Should output: `deviceName = NVIDIA GeForce RTX 5090`

### 4. Restart Hyprland
```bash
# Log out and log back in, or:
hyprctl reload
```

## Expected Performance After Fix

### Before (30fps @ 1440p):
- Wrong GPU for display output
- Constant frame copying overhead
- Display/render mismatch penalties

### After (should be much better):
- Native display output through correct GPU
- 5090 renders, T400 displays (efficient)
- Should see **200-400+ fps** in simple games
- Should see **native performance** for AAA games at your monitor's capabilities

## Limitations of This Setup

### Why Not Full PRIME Offload?
Traditional PRIME offload (Intel iGPU → NVIDIA dGPU) doesn't work well with two discrete NVIDIA cards because:
1. PRIME was designed for integrated + discrete setups
2. Both your GPUs are discrete (not integrated)
3. NVIDIA doesn't support PRIME render offload between two of their own discrete GPUs efficiently

### Current Approach: Explicit Selection
Instead, we use **explicit GPU selection** per-application:
- Lighter apps: automatic (T400)
- Heavy apps/games: explicit via `gpu-5090` wrapper
- Works reliably with dual discrete NVIDIA cards

### Best Solution (Hardware)
For **absolute maximum gaming performance**, physically connect your monitor to the **5090** instead of the T400:
- Eliminates all display copy overhead
- Native scan-out from gaming GPU
- T400 can be used for secondary displays or disabled

## Troubleshooting

### Still Getting Poor Performance?
1. Run `gpu-info` and verify configuration
2. Make sure you're launching games with `gpu-5090` prefix
3. Check if game is actually using the 5090:
   ```bash
   # While game is running:
   nvidia-smi
   # Should show GPU utilization on GPU 0 (5090)
   ```

### Hyprland Won't Start?
1. Check logs: `journalctl --user -u uwsm@hyprland -b`
2. Verify DRM device exists: `ls -l /dev/dri/by-path/pci-0000:0a:00.0-card`
3. Fall back to auto-detection (remove WLR_DRM_DEVICES)

### Games Launch on Wrong GPU?
Some games need additional configuration:
```bash
# Steam launch options:
gpu-5090 %command%

# Lutris: Edit game → System options → Environment variables:
MESA_VK_DEVICE_SELECT=10de:2b85
__NV_PRIME_RENDER_OFFLOAD=1
```

## Additional Optimizations

### Consider These Next Steps:
1. **Test 240Hz**: Your monitor supports it, try bumping from 120Hz
2. **Enable VRR in games**: Most modern games have VRR/G-SYNC options
3. **Monitor temps**: RTX 5090 is powerful but hot, ensure good cooling
4. **Power limits**: Consider using `nvidia-smi` to set power limits if needed

### Steam Integration
Edit `~/.local/bin/steam` to automatically use the 5090:
```bash
#!/bin/bash
exec gpu-5090 /usr/bin/steam "$@"
```

## Files Modified

- ✏️ `~/.config/hypr/source/environment_variables.conf` - Fixed DRM device selection
- ✏️ `~/.config/hypr/source/monitors.conf` - Enabled VRR
- ✏️ `~/.config/uwsm/env-hyprland` - Added GPU configuration
- ➕ `~/.local/bin/gpu-5090` - Gaming GPU launcher
- ➕ `~/.local/bin/gpu-t400` - Display GPU launcher
- ➕ `~/.local/bin/gpu-info` - Diagnostic tool
- ❌ `~/.config/vulkan/implicit_layer.d/nvidia-5090.json` - Removed (replaced with better solution)

---

**Created**: 2026-01-16
**GPU Setup**: Dual NVIDIA (T400 display + 5090 gaming)
**Compositor**: Hyprland (Aquamarine/WLR)
