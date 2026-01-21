# QUICK FIX - RTX 5090 Performance Issue

## 🔥 THE PROBLEM
Your monitor is plugged into the **T400** (weak GPU).  
Your games try to use the **RTX 5090** (strong GPU).  
The 30fps bottleneck is the frame copying between them.

## ⚡ THE FIX (Do This Now!)

### Physical Solution (30 seconds - BEST)
```
1. Shut down PC
2. Unplug monitor cable from T400
3. Plug monitor cable into RTX 5090
4. Boot up
5. Done - enjoy 200-400+ fps
```

**This fixes everything.** No software hacks needed.

## 🔧 Software Solution (If You Can't Move Cable)

### Step 1: Logout & Login
Changes won't apply until you restart your session.

```bash
# Logout from Hyprland, then login again
# Or reboot
```

### Step 2: Launch Games Correctly
```bash
# For Steam games, add to launch options:
gpu-5090 %command%

# For Lutris/other launchers:
gpu-5090 <game-command>
```

### Step 3: Check Status
```bash
gpu-info
```

## 📊 What Works After Software Fix

| Game Type | Performance | Notes |
|-----------|-------------|-------|
| Modern Vulkan games | ✅ Good | Use `gpu-5090` wrapper |
| Old OpenGL games | ❌ Still 30fps | NVIDIA limitation |
| Desktop apps | ✅ Normal | Using T400 (fine) |
| CUDA apps | ✅ Good | Use `gpu-5090` wrapper |

## 🎯 Commands You Need

```bash
# Check GPU configuration
gpu-info

# Launch game with 5090
gpu-5090 steam

# Test Vulkan performance
gpu-5090 vkcube

# VM passthrough (existing)
gpu-detach  # Unbind 5090 for VM
gpu-attach  # Rebind 5090 to host
```

## 📈 Expected FPS After Fix

### Current: 30fps @ 1440p

### After Software Fix:
- Vulkan games: 100-200fps (better)
- OpenGL games: still 30fps (limited by NVIDIA driver)

### After Moving Cable to 5090:
- ALL games: 200-400+ fps (full power)
- Can enable 240Hz refresh rate
- Can enable ray tracing, DLSS, etc.

## ❓ Why Not Just Software Fix?

NVIDIA PRIME offload **doesn't work** with two discrete GPUs.  
It only works for: iGPU → dGPU (laptop/APU scenarios).

Your setup: dGPU → dGPU (not supported by NVIDIA).

## 🎮 Bottom Line

**Hardware fix (move cable) > Software fix (wrappers)**

You bought a 5090. Plug your monitor into it!

---

**Read Full Details:**
- `~/.config/GPU_DIAGNOSIS.md` - Complete analysis
- `~/.config/GPU_SETUP_README.md` - Configuration guide
