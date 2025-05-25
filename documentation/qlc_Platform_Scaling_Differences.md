# QLC+ Platform Scaling Differences: Windows vs Raspberry Pi

## Overview

This document explains why QLC+ appears with different font and widget sizes between Windows and Raspberry Pi platforms, and provides solutions to achieve consistent appearance across platforms.

## Background: The Root Cause

### Platform Architecture Differences

QLC+ uses the Qt framework, which renders differently depending on the underlying platform and display system:

#### **Windows Platform**
- **Qt Platform**: Standard windowing system (`windows` platform)
- **Display System**: Windows Desktop Window Manager (DWM)
- **Font Rendering**: DirectWrite/GDI+ with ClearType
- **DPI Handling**: System-wide DPI scaling (typically 125%-150%)
- **Window Management**: Full window manager with title bars, borders, etc.

#### **Raspberry Pi Platform (as configured)**
- **Qt Platform**: EGLFS (Embedded OpenGL Full Screen)
- **Display System**: Direct framebuffer rendering
- **Font Rendering**: FreeType with different hinting
- **DPI Handling**: Hardware-based DPI detection
- **Window Management**: No window manager (`--nowm` flag)

### Why EGLFS Causes Larger Fonts

**EGLFS (Embedded Graphics Library Full Screen)** is designed for embedded systems and makes different assumptions about display density:

1. **DPI Detection**: EGLFS calculates DPI based on physical display size and resolution
2. **No System Scaling**: Unlike Windows, there's no system-wide DPI scaling applied
3. **Direct Rendering**: Bypasses desktop environment scaling mechanisms
4. **Embedded Defaults**: Optimized for touch interfaces and larger UI elements

## Investigation Results

Based on our diagnosis of your Raspberry Pi system:

### System Configuration
- **Model**: Raspberry Pi 4 Model B Rev 1.2
- **Resolution**: 1920x1200 framebuffer
- **QLC+ Version**: 4.14.1
- **Qt Platform**: EGLFS (`-platform eglfs`)
- **Startup Mode**: Full-screen with no window manager (`--nowm`)

### Font Analysis
- **Required Font**: DejaVu Sans Condensed (used 643 times in workspace)
- **Font Availability**: ✅ Properly installed and detected
- **Font Resolution**: Correctly resolving to `DejaVuSansCondensed.ttf`
- **Font Size**: 8pt specified in workspace file

### Scaling Analysis
- **Qt Environment**: No custom scaling variables set initially
- **System DPI**: Hardware-calculated by EGLFS
- **Workspace**: Optimized for Windows scaling behavior

## Visual Comparison

Based on the provided screenshots:
- **Raspberry Pi**: Larger fonts and widgets due to EGLFS DPI calculations
- **Windows**: Smaller, more compact interface due to system DPI scaling

## Solution Options

### Option A: Qt Environment Variables (Recommended) ✅ IMPLEMENTED

**What it does**: Sets specific Qt scaling factors to override EGLFS defaults

```bash
export QT_FONT_DPI=96
export QT_SCALE_FACTOR=0.8
```

**Parameters Explained**:

#### `QT_FONT_DPI=96`
- **Purpose**: Forces Qt to use 96 DPI (Windows standard) instead of hardware-calculated DPI
- **Effect**: Standardizes font size calculations across platforms
- **Range**: Typically 72-144 DPI
- **Why 96**: This is the traditional Windows DPI baseline

#### `QT_SCALE_FACTOR=0.8`
- **Purpose**: Applies 80% scaling to all UI elements
- **Effect**: Reduces widget sizes to match Windows appearance
- **Range**: 0.5-2.0 (0.8 = 80% of original size)
- **Why 0.8**: Compensates for EGLFS rendering at effectively 125% compared to Windows

**Implementation Status**: ✅ Added to `/usr/sbin/qlcplus-start.sh` (systemd service startup script)

**Critical Note**: Initially attempted to add to `~/.bashrc`, but systemd services don't source user shell configurations. The correct approach is to modify the actual startup script used by the service.

### Option B: Qt Platform Change (Alternative)

**What it does**: Changes Qt rendering platform from EGLFS to standard windowing

```bash
qlcplus -platform wayland  # or -platform xcb
```

**Trade-offs**:
- ✅ **Pros**: More similar to Windows rendering
- ❌ **Cons**: Requires desktop environment, loses full-screen optimization
- ❌ **Cons**: May conflict with existing startup configuration

### Option C: QLC+ Startup Script Modification (Advanced)

**What it does**: Modifies the system QLC+ startup script to include scaling parameters

```bash
# Edit /usr/sbin/qlcplus-start.sh
export QT_FONT_DPI=96
export QT_SCALE_FACTOR=0.8
```

**Trade-offs**:
- ✅ **Pros**: System-wide application
- ❌ **Cons**: Requires root access, affects all QLC+ instances
- ❌ **Cons**: May be overwritten on package updates

## Fine-Tuning Parameters

If the implemented 0.8 scaling factor doesn't perfectly match your Windows interface, you can adjust:

### Scaling Factor Adjustments
```bash
# For slightly larger interface (closer to Windows)
export QT_SCALE_FACTOR=0.85

# For smaller interface (more compact than Windows)
export QT_SCALE_FACTOR=0.75

# For matching specific Windows DPI settings
export QT_SCALE_FACTOR=0.8  # ~125% Windows scaling
export QT_SCALE_FACTOR=0.67  # ~150% Windows scaling
```

### DPI Adjustments
```bash
# Higher DPI for sharper fonts (may increase size)
export QT_FONT_DPI=120

# Lower DPI for smaller fonts
export QT_FONT_DPI=72

# Match specific display characteristics
export QT_FONT_DPI=96   # Standard Windows
export QT_FONT_DPI=144  # High-DPI Windows (150%)
```

## Testing Procedure

### 1. Apply Changes (Done) ✅ CORRECTED IMPLEMENTATION
The environment variables have been added to the **QLC+ startup script** (not .bashrc):
- **File**: `/usr/sbin/qlcplus-start.sh`
- **Variables added**:
```bash
export QT_FONT_DPI=96
export QT_SCALE_FACTOR=0.8
```

**Why startup script instead of .bashrc**: Systemd services don't source user shell configurations. The environment variables must be set in the actual startup script that the service executes.

### 2. Restart QLC+ Service (Done) ✅ COMPLETED
Applied the new scaling by restarting the service:
```bash
sudo systemctl restart qlcplus
```

### 3. Visual Comparison
Compare the following elements between platforms:
- **Button sizes**: Should be similar between Windows and Pi
- **Text readability**: 8pt DejaVu Sans Condensed should appear consistent
- **Interface density**: Overall layout should match Windows proportions
- **Widget spacing**: Margins and padding should be proportional

### 4. Adjustment Process
If further tuning is needed:
```bash
# Edit the scaling factor
ssh pi@192.168.1.80
nano ~/.bashrc
# Modify QT_SCALE_FACTOR value
# Restart QLC+ to test
```

## Platform-Specific Considerations

### Windows Characteristics
- **DPI Awareness**: System handles DPI scaling automatically
- **Font Smoothing**: ClearType provides consistent font appearance
- **Window Management**: Full desktop integration
- **User Scaling**: Users can adjust system DPI settings

### Raspberry Pi (EGLFS) Characteristics
- **Performance Optimized**: Direct GPU rendering for better performance
- **Full-Screen**: Optimized for kiosk/embedded applications
- **Hardware Integration**: Direct access to display hardware
- **Limited User Control**: Fewer runtime scaling options

## Troubleshooting

### If Interface is Still Too Large
```bash
# Reduce scaling factor further
export QT_SCALE_FACTOR=0.7
```

### If Interface is Too Small
```bash
# Increase scaling factor
export QT_SCALE_FACTOR=0.9
```

### If Fonts Look Blurry
```bash
# Adjust DPI instead of scaling
export QT_FONT_DPI=120
export QT_SCALE_FACTOR=1.0
```

### If Changes Don't Apply
```bash
# Verify environment variables are set
env | grep QT

# Ensure QLC+ process uses new environment
sudo systemctl restart qlcplus
# or
sudo reboot
```

## Systemd Service Considerations

### Environment Variables in Systemd Services
When QLC+ runs as a systemd service, it has specific requirements for environment variables:

1. **User shell configurations** (`.bashrc`, `.profile`) are **NOT sourced**
2. **Service-specific environment** must be set in the startup script or service file
3. **System environment** (`/etc/environment`) can be used but affects all processes

### Implementation Methods for Systemd Services

#### Method 1: Startup Script Modification (Used)
```bash
# Modify /usr/sbin/qlcplus-start.sh
export QT_FONT_DPI=96
export QT_SCALE_FACTOR=0.8
```
- ✅ **Pros**: Direct control, immediate effect
- ⚠️ **Cons**: May be overwritten on package updates

#### Method 2: Systemd Environment File
```bash
# Create /etc/systemd/system/qlcplus.service.d/override.conf
[Service]
Environment="QT_FONT_DPI=96"
Environment="QT_SCALE_FACTOR=0.8"
```
- ✅ **Pros**: Systemd-native, update-resistant
- ❌ **Cons**: More complex setup

#### Method 3: System-wide Environment
```bash
# Add to /etc/environment
QT_FONT_DPI=96
QT_SCALE_FACTOR=0.8
```
- ✅ **Pros**: Affects all Qt applications
- ❌ **Cons**: May affect other applications unintentionally

## Best Practices

### 1. Workspace Synchronization
- Use the same workspace file (`.qxw`) on both platforms
- Keep font specifications consistent
- Test on both platforms after changes

### 2. Documentation
- Document any custom scaling factors used
- Note the reasoning for specific DPI settings
- Keep platform-specific configurations separate

### 3. Updates and Maintenance
- Backup `.bashrc` before modifications
- Test scaling after QLC+ updates
- Monitor for package updates that might reset configurations

## Technical References

### Qt Environment Variables
- **QT_SCALE_FACTOR**: Global scaling factor for all UI elements
- **QT_FONT_DPI**: Font-specific DPI override
- **QT_AUTO_SCREEN_SCALE_FACTOR**: Automatic scaling based on screen DPI
- **QT_SCREEN_SCALE_FACTORS**: Per-screen scaling factors

### Qt Platform Plugins
- **eglfs**: Embedded OpenGL Full Screen (current Pi configuration)
- **xcb**: X11 windowing system
- **wayland**: Wayland compositor
- **linuxfb**: Linux framebuffer

### QLC+ Specific Parameters
- **--nowm**: No window manager mode
- **-platform**: Qt platform selection
- **--operate**: Start in operate mode
- **--web**: Enable web interface

---

**Document Version**: 1.1  
**Last Updated**: December 2024  
**Status**: ✅ Solution CORRECTLY implemented in systemd startup script  
**Implementation**: Modified `/usr/sbin/qlcplus-start.sh` with QT_SCALE_FACTOR=0.8 and QT_FONT_DPI=96  
**Service Status**: ✅ QLC+ service restarted successfully  
**Next Steps**: Test visual consistency with Windows platform and fine-tune scaling if needed 