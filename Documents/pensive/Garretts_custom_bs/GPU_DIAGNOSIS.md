# GPU Performance Issue - Root Cause Analysis

## TL;DR - The Core Problem

**Your monitor is plugged into the wrong GPU.**

Your Samsung Odyssey is connected to the **T400** (weak GPU), but you're trying to force everything through the **5090** (strong GPU). This creates a performance nightmare because:

1. Games try to render on the 5090
2. Frames must be copied from 5090 → T400 for display
3. This copy process is slow and unreliable
4. Result: 30fps instead of 300+fps

## Why This Happens

### The Hardware Reality
```
Physical Setup:
┌─────────────┐     ┌──────────────────┐
│   RTX 5090  │     │   T400 4GB       │──── Samsung Odyssey 5120x1440
│  (Gaming)   │     │  (Display Out)   │     (Your Monitor)
│  No Monitor │     │  Monitor HERE ✓  │
└─────────────┘     └──────────────────┘
      ↓                      ↑
      └──────────────────────┘
        Slow frame copying
        (30fps bottleneck)
```

### Why Dual Discrete NVIDIA GPUs Are Problematic

**NVIDIA PRIME Offload Doesn't Work Here**

PRIME render offload was designed for:
- **Laptop iGPU → Laptop dGPU** (Intel → NVIDIA)
- **Desktop iGPU → Desktop dGPU** (AMD APU → NVIDIA)

It was **NOT** designed for:
- **Desktop dGPU → Desktop dGPU** (NVIDIA → NVIDIA)

When you have two discrete NVIDIA cards, PRIME offload is broken:
- ✅ Vulkan device selection: **WORKS** (we can force apps to use 5090)
- ❌ OpenGL/GLX offload: **BROKEN** (apps use compositor's GPU = T400)
- ❌ Automatic GPU switching: **NOT SUPPORTED**

### Current Performance Breakdown

**What's Using Which GPU Right Now:**
| Component | Current GPU | Performance Impact |
|-----------|-------------|-------------------|
| Compositor (Hyprland) | T400* | Trying to use 5090, but monitor on T400 = copy overhead |
| Desktop apps (Firefox, etc) | T400 | OK (low power) |
| Vulkan games | 5090 ✓ | Good IF using gpu-5090 wrapper |
| OpenGL games | T400 ❌ | TERRIBLE (weak GPU) |
| Frame presentation | T400 (monitor) | Bottleneck if rendering on 5090 |

*Currently misconfigured - will be fixed after logout

## What I Fixed (Partial Solution)

### ✅ Fixed Configuration Files

1. **Corrected compositor GPU** (`~/.config/hypr/source/environment_variables.conf`)
   - OLD: Forced Hyprland to use 5090 (wrong - no monitor there)
   - NEW: Hyprland uses T400 (where monitor is connected)
   - **Requires logout to take effect**

2. **Created GPU selection tools** (`~/.local/bin/gpu-5090`)
   - Vulkan games: Will use 5090 correctly ✓
   - CUDA apps: Will use 5090 correctly ✓
   - OpenGL games: Still broken (NVIDIA limitation) ❌

3. **Enabled VRR** (Variable Refresh Rate)
   - Monitor can now use adaptive sync
   - Reduces tearing once performance is fixed

### ⚠️ What's Still Broken

**OpenGL Games Won't Use the 5090**

Even with the `gpu-5090` wrapper, OpenGL/GLX games will render on the T400 because:
- NVIDIA doesn't support PRIME offload between two discrete GPUs
- The compositor determines which GPU OpenGL contexts use
- XRandR offload doesn't work (no providers available)

**This Means:**
- Old games using OpenGL: Slow (T400) ❌
- Modern games using Vulkan: Fast (5090) ✓
- Most AAA games since ~2019: Vulkan (you're fine) ✓
- Indie/older games: May use OpenGL (still slow) ❌

## The Real Solutions

### Option 1: Move Monitor to 5090 (BEST - DO THIS)

**Physically plug your monitor into the RTX 5090.**

**Why This Fixes Everything:**
1. ✅ No frame copying overhead
2. ✅ Native scan-out from gaming GPU
3. ✅ ALL games (OpenGL + Vulkan) use 5090
4. ✅ Full 240Hz support (your monitor can do it!)
5. ✅ No software hacks needed
6. ✅ Maximum possible performance

**What About the T400?**
- Use it for secondary monitors (if you add any)
- Or remove it entirely (optional)
- Keep it for VM passthrough testing (doesn't interfere)

**Expected Performance:**
- Current: 30fps @ 1440p
- After fix: 200-400+ fps in most games
- You have a 5090 - use it properly!

### Option 2: Accept Limitations (CURRENT)

Keep the current setup but understand:
- ✅ Vulkan games work (use `gpu-5090` wrapper)
- ❌ OpenGL games stay slow
- ⚠️ Always have frame copy overhead
- ⚠️ Compositor on weaker GPU

**When to use this:**
- You NEED the T400 for display output (Why?)
- You only play modern Vulkan games
- You're okay with reduced performance

### Option 3: Single GPU Mode

Remove T400 entirely:
- Simplifies configuration
- 5090 does everything
- No dual-GPU complexity
- Loses VM passthrough flexibility (you'd pass through your only GPU)

## Testing After Logout

After you log out and back in, the config changes will apply:

### 1. Check Configuration
```bash
gpu-info
```

Should show:
- WLR_DRM_DEVICES: `/dev/dri/by-path/pci-0000:0a:00.0-card` (T400) ✓
- Monitor: Connected to card2-DP-5 (T400) ✓
- Compositor: Using T400 (matches monitor) ✓

### 2. Test Vulkan Game Performance
```bash
gpu-5090 vkcube
```

Should show:
- Device: NVIDIA GeForce RTX 5090 ✓
- Performance: High FPS (hundreds) ✓

### 3. Test Real Game
```bash
# If using Steam, add to launch options:
gpu-5090 %command%

# Or from terminal:
gpu-5090 steam steam://rungameid/XXXXXX
```

**Expected Results:**
- Vulkan games: Good performance (with wrapper)
- OpenGL games: Still slow (30fps issue persists)
- Desktop: Works normally

## Why You Were Getting 30fps

Your Vulkan hack (`nvidia-5090.json` implicit layer) was forcing Vulkan to the 5090, but:

1. **The compositor was trying to use the 5090 for display**
   - But the monitor isn't connected to the 5090
   - Required copying frames: 5090 → T400 → Monitor
   - This copy is SLOW and limited to ~30fps

2. **OpenGL games were using the T400 directly**
   - T400 is a weak professional GPU
   - Not designed for gaming
   - ~30fps at 1440p is expected from a T400

3. **No VRR/Adaptive Sync**
   - Additional tearing and stuttering
   - Fixed now (after logout)

## Recommendation

**Move your monitor cable from the T400 to the 5090.**

This is a 30-second hardware change that will:
- ✅ Give you native 5090 performance
- ✅ Fix ALL games (OpenGL + Vulkan)  
- ✅ Enable full 240Hz support
- ✅ Eliminate frame copying
- ✅ Make all my software fixes unnecessary (but keep them for flexibility)

**After moving the cable:**
1. Update monitors.conf to use the new DP port (will be different)
2. Remove WLR_DRM_DEVICES override (let it auto-detect)
3. Enjoy full 5090 performance
4. Consider bumping to 240Hz refresh rate

## Files Changed

### Configuration Files (Logout Required)
- ✏️ `~/.config/hypr/source/environment_variables.conf` - Fixed DRM device
- ✏️ `~/.config/hypr/source/monitors.conf` - Enabled VRR
- ✏️ `~/.config/uwsm/env-hyprland` - GPU environment

### New Tools (Ready Now)
- ➕ `~/.local/bin/gpu-5090` - Force 5090 (Vulkan + CUDA only)
- ➕ `~/.local/bin/gpu-t400` - Force T400
- ➕ `~/.local/bin/gpu-info` - Diagnostic tool

### Removed
- ❌ `~/.config/vulkan/implicit_layer.d/nvidia-5090.json` - Crude hack removed

## VM Passthrough Notes

Your existing VM scripts still work:
- `gpu-detach` - Unbinds 5090 from host
- `gpu-attach` - Rebinds 5090 to host  
- `gpu-status` - Shows current state

**If you move the monitor to the 5090:**
- You won't have display output when VM is running (5090 is passed through)
- This is expected behavior
- Use a second monitor on T400, or remote access (Moonlight/Parsec)
- Or move monitor back to T400 when using VM

## Performance Expectations

### Current Setup (After Logout)
- Vulkan games with `gpu-5090`: 60-144fps (better, but still overhead)
- OpenGL games: 30fps (unchanged - hardware limitation)
- Desktop: Normal (T400 is fine for this)

### After Moving Monitor to 5090
- Modern games: 200-400+ fps (full 5090 power)
- Old games: 100-200+ fps (full 5090 power)
- Desktop: Normal (5090 is fine for this too)
- Power draw: Higher (5090 runs for everything)

---

**Created:** 2026-01-16  
**Issue:** 30fps at 1440p with RTX 5090  
**Root Cause:** Monitor connected to wrong GPU (T400 instead of 5090)  
**Best Fix:** Move monitor cable to 5090 ⚡
