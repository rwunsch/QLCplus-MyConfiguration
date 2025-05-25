# Raspberry Pi QLC+ Configuration Guide

## Table of Contents
- [SSH Access](#ssh-access)
- [Hardware Configuration](#hardware-configuration)
- [QLC+ Configuration Management](#qlc-configuration-management)
- [QLC+ Service Management](#qlc-service-management)
- [Qt Platform and EGLFS Configuration](#qt-platform-and-eglfs-configuration)
- [Display and Platform Options](#display-and-platform-options)
- [System Configuration](#system-configuration)
- [Software Updates](#software-updates)
- [File Paths and Directory Structure](#file-paths-and-directory-structure)
- [External Hardware](#external-hardware)

## SSH Access

### Network Connections
```bash
# WLAN Connection
ssh pi@192.168.1.80

# LAN Connection  
ssh pi@192.168.1.84
```

### Credentials
- **Username**: `pi`
- **Password**: `[REDACTED]`

## Hardware Configuration

### TouchScreen Setup (5-inch DSI TouchScreen - SPI)

Edit the Raspberry Pi configuration file:
```bash
sudo nano /boot/firmware/config.txt
```

#### Enable Touchscreen Driver
Change from:
```bash
# Enable DRM VC4 V3D driver
dtoverlay=vc4-kms-v3d
```

To:
```bash
# Enable DRM VC4 V3D driver with touchscreen support
dtoverlay=vc4-fkms-v3d
```

**Note**: The "f" in `fkms` stands for "fake" KMS driver, which provides compatibility with a broader range of touchscreens.

#### Optional: Disable Bluetooth for Better WLAN
```bash
dtoverlay=disable-bt
```

### Network Configuration
Enable WLAN using the Raspberry Pi configuration utility:
```bash
sudo raspi-config
```
Navigate to Network Options → Wi-Fi and configure your wireless connection.

## QLC+ Configuration Management

### Git Repository Setup

#### Install Git
```bash
sudo apt-get install git
```

#### Clone Configuration Repository
```bash
cd /home/pi/.qlcplus
mv eglfs.json ..                    # Backup EGLFS config
rm -rf *                            # Clear directory
git clone https://github.com/rwunsch/QLCplus-MyConfiguration.git .
mv ../eglfs.json .                  # Restore EGLFS config
```

**Git Token**: `[REDACTED - Replace with your personal access token]`

### Automatic Repository Updates

#### Create Update Script
```bash
nano /home/pi/update_qlc-config-repo.sh
```

Add the following content:
```bash
#!/bin/bash
cd /home/pi/.qlcplus
mv eglfs.json ..                    # Backup EGLFS config
git restore .                       # Reset any local changes
git pull origin main                # Update from repository
mv ../eglfs.json .                  # Restore EGLFS config
```

#### Make Script Executable
```bash
chmod +x /home/pi/update_qlc-config-repo.sh
```

#### Create Systemd Service for Auto-Update
```bash
sudo nano /etc/systemd/system/qlcplus-git-pull.service
```

Service configuration:
```ini
[Unit]
Description=Update Git Repository on Reboot
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/home/pi/update_qlc-config-repo.sh
RemainAfterExit=yes
User=pi
WorkingDirectory=/home/pi/.qlcplus

[Install]
WantedBy=multi-user.target
```

#### Enable and Test Auto-Update Service
```bash
sudo systemctl enable qlcplus-git-pull.service
sudo systemctl start qlcplus-git-pull.service
sudo systemctl status qlcplus-git-pull.service
```

## QLC+ Service Management

### Basic Service Commands
```bash
# Enable QLC+ to start on boot
sudo systemctl enable qlcplus

# Start QLC+ service
sudo systemctl start qlcplus

# Restart QLC+ service
sudo systemctl restart qlcplus

# Check QLC+ service status
sudo systemctl status qlcplus

# Stop QLC+ service
sudo systemctl stop qlcplus
```

### Service Configuration

#### Service Definition File
**Location**: `/lib/systemd/system/qlcplus.service`

**Content**:
```ini
[Unit]
Description=Q Light Controller Plus
Documentation=man:qlcplus(1)
After=basic.target

[Service]
Type=simple
User=pi
Restart=on-failure
RestartSec=3
ExecStart=/usr/sbin/qlcplus-start.sh

[Install]
WantedBy=multi-user.target
```

## Qt Platform and EGLFS Configuration

### Understanding EGLFS
**EGLFS** (Embedded OpenGL Full Screen) is Qt's platform plugin for embedded devices. It provides:
- Direct framebuffer rendering without a window manager
- Hardware-accelerated OpenGL rendering
- Touch input support
- Optimized performance for embedded systems

### Startup Script Configuration
**Location**: `/usr/sbin/qlcplus-start.sh`

#### Key EGLFS Environment Variables
The startup script configures several critical Qt/EGLFS parameters that affect UI rendering:

```bash
# Physical display dimensions (affects DPI calculation)
export QT_QPA_EGLFS_PHYSICAL_WIDTH=320     # Display width in millimeters
export QT_QPA_EGLFS_PHYSICAL_HEIGHT=200    # Display height in millimeters

# EGLFS behavior settings
export QT_QPA_EGLFS_ALWAYS_SET_MODE=1      # Force display mode setting

# GPU device configuration
export QT_QPA_EGLFS_KMS_CONFIG=$HOME/.qlcplus/eglfs.json
```

#### How Physical Dimensions Affect UI Scale
EGLFS calculates the display DPI using the formula: `DPI = screen_resolution / physical_size_in_inches`

- **Smaller values** = Higher calculated DPI = **Larger fonts/widgets**
- **Larger values** = Lower calculated DPI = **Smaller fonts/widgets**

#### Determining Your Screen's Physical Dimensions

**Why This Matters**: Incorrect physical dimensions cause UI scaling issues. If your UI elements appear too large or too small compared to Windows, the physical dimensions likely don't match your actual screen.

**How to Measure**:
1. **Measure your screen**: Use a ruler to measure the visible display area (not the bezel)
2. **Convert to millimeters**: Multiply centimeters by 10
3. **Update the configuration**: Modify the startup script with accurate values

**Common Screen Sizes**:
```bash
# 5-inch touchscreen (typical Pi accessory): ~110x65mm
QT_QPA_EGLFS_PHYSICAL_WIDTH=110
QT_QPA_EGLFS_PHYSICAL_HEIGHT=65

# 7-inch display: ~155x87mm
QT_QPA_EGLFS_PHYSICAL_WIDTH=155
QT_QPA_EGLFS_PHYSICAL_HEIGHT=87

# Large 35x22cm touchscreen (1920x1080): 350x220mm
QT_QPA_EGLFS_PHYSICAL_WIDTH=350
QT_QPA_EGLFS_PHYSICAL_HEIGHT=220
```

#### Updating Physical Dimensions

**Step 1: Backup Current Configuration**
```bash
sudo cp /usr/sbin/qlcplus-start.sh /usr/sbin/qlcplus-start.sh.backup
```

**Step 2: Edit the Startup Script**
```bash
sudo nano /usr/sbin/qlcplus-start.sh
```

Find and modify these lines:
```bash
export QT_QPA_EGLFS_PHYSICAL_WIDTH=XXX    # Your screen width in mm
export QT_QPA_EGLFS_PHYSICAL_HEIGHT=YYY   # Your screen height in mm
```

**Step 3: Apply Changes**
```bash
sudo systemctl restart qlcplus
```

#### Example: Updating for 35x22cm Screen

**Problem**: Default settings (320x200mm) caused oversized UI elements on a 35x22cm screen.

**Solution**:
```bash
# Old settings (causing oversized UI)
export QT_QPA_EGLFS_PHYSICAL_WIDTH=320
export QT_QPA_EGLFS_PHYSICAL_HEIGHT=200

# New settings (accurate for 35x22cm screen)
export QT_QPA_EGLFS_PHYSICAL_WIDTH=350    # 35cm = 350mm
export QT_QPA_EGLFS_PHYSICAL_HEIGHT=220   # 22cm = 220mm
```

**Commands used**:
```bash
# Update width
sudo sed -i 's/export QT_QPA_EGLFS_PHYSICAL_WIDTH=320/export QT_QPA_EGLFS_PHYSICAL_WIDTH=350/' /usr/sbin/qlcplus-start.sh

# Update height  
sudo sed -i 's/export QT_QPA_EGLFS_PHYSICAL_HEIGHT=200/export QT_QPA_EGLFS_PHYSICAL_HEIGHT=220/' /usr/sbin/qlcplus-start.sh

# Restart service
sudo systemctl restart qlcplus
```

**Result**: UI elements now scale appropriately for the larger screen, matching the intended design proportions.

### EGLFS Configuration Files

#### GPU Device Configuration
**File**: `~/.qlcplus/eglfs.json`

**Purpose**: Specifies which GPU device EGLFS should use

**Example content**:
```json
{ "device": "/dev/dri/card0" }
```

**Auto-generation**: The startup script automatically detects and configures the GPU device if this file doesn't exist.

### Platform Detection Logic
The startup script automatically selects the appropriate Qt platform:

```bash
# Default to EGLFS
QTPLATFORM="eglfs"

# Switch to offscreen if no display connected
kmsprint -m | grep connected > /dev/null
if [ $? -eq 1 ]; then
    QTPLATFORM="offscreen"
fi

# Switch to VNC if VNC file exists
if [ -f $HOME/.qlcplus/vnc.txt ]; then
    QTPLATFORM="vnc:size=1920x1080"
fi
```

## Display and Platform Options

### HDMI Display Mode
**Default configuration** for direct HDMI output with touchscreen support.

**Platform**: `eglfs`
**Features**:
- Hardware acceleration
- Touch input support
- Full-screen operation
- Direct framebuffer access

### VNC Display Mode
**Purpose**: Remote access without physical display

#### Enable VNC Mode
```bash
# Create VNC trigger file
touch ~/.qlcplus/vnc.txt

# Restart QLC+ service
sudo systemctl restart qlcplus
```

#### VNC Configuration
```bash
# Basic VNC platform
QLCPLUS_OPTS="-platform vnc --nowm --web --web-auth --operate --overscan"

# VNC with specific resolution
QLCPLUS_OPTS="-platform vnc:size=1920x1080 --nowm --web --web-auth --operate --overscan"
```

#### Disable VNC Mode
```bash
# Remove VNC trigger file
rm ~/.qlcplus/vnc.txt

# Restart service
sudo systemctl restart qlcplus
```

### Headless/Offscreen Mode
**Purpose**: Running without any display output

```bash
QLCPLUS_OPTS="--nowm --web-auth --operate --nogui -platform linuxfb:size=1x1"
```

### LinuxFB Mode
**Purpose**: Basic framebuffer access for minimal display

```bash
QLCPLUS_OPTS="-platform linuxfb --nowm --web --web-auth --operate --overscan"
```

## System Configuration

### Raspberry Pi Boot Configuration
**File**: `/boot/firmware/config.txt`

#### Key Settings for QLC+
```bash
# Overscan adjustments
overscan_left=16
overscan_right=16
overscan_top=16
overscan_bottom=16

# SPI interface (for touchscreen)
dtparam=spi=on

# Audio support
dtparam=audio=on

# Graphics driver with touchscreen support
dtoverlay=vc4-fkms-v3d
max_framebuffers=2

# Performance optimization
arm_boost=1

# QLC+ headless workaround
hdmi_force_hotplug=1
```

#### HDMI Configuration Options
```bash
# Force HDMI output (even without monitor)
hdmi_force_hotplug=1

# Specific HDMI mode (uncomment if needed)
#hdmi_group=2
#hdmi_mode=16    # 1920x1080 60Hz

# HDMI signal boost (if display issues)
#config_hdmi_boost=4
```

### Command Line Parameters
**File**: `/boot/firmware/cmdline.txt`

```bash
console=tty1 root=PARTUUID=76fe5b12-02 rootfstype=ext4 fsck.repair=yes rootwait quiet
```

## Software Updates

### QLC+ Updates

#### Download and Install New Version
1. **Download** from [QLC+ reserved area](https://www.qlcplus.org/reserved/)

2. **Upload to Pi**:
   ```bash
   scp qlcplus_4.14.1_arm64.deb pi@192.168.1.80:/home/pi/
   ```

3. **Install on Pi**:
   ```bash
   ssh pi@192.168.1.80
   
   # Stop running instance
   sudo systemctl stop qlcplus || true
   
   # Remove conflicting packages if needed
   # sudo apt remove qlcplus-data
   
   # Install new version
   sudo dpkg -i ~/qlcplus_4.14.1_arm64.deb
   
   # Install missing dependencies
   sudo apt -f install
   
   # Start service
   sudo systemctl start qlcplus
   
   # Verify version
   qlcplus -v
   ```

### Raspberry Pi OS Updates
```bash
sudo apt update
sudo apt upgrade
```

## File Paths and Directory Structure

### QLC+ Configuration Directories

#### Linux (Raspberry Pi)
```
/home/pi/.qlcplus/                  # Main configuration directory
├── autostart.qxw                  # Auto-loaded workspace
├── eglfs.json                     # GPU configuration
├── vnc.txt                        # VNC mode trigger (optional)
├── fixtures/                      # Custom fixture definitions
├── inputprofiles/                 # Input device profiles
├── miditemplates/                 # MIDI templates
├── modifierstemplates/             # Modifier templates
└── rgbscripts/                    # RGB matrix scripts

/usr/share/qlcplus/                 # System-wide QLC+ files
/usr/sbin/qlcplus-start.sh         # Service startup script
/home/pi/.config/qlcplus/          # Additional configuration
```

#### Windows (for comparison)
```
C:\QLC+\                           # Installation directory
C:\Users\[USERNAME]\QLC+\          # User configuration
```

### Important Configuration Files

| File | Purpose | Location |
|------|---------|----------|
| `qlcplus-start.sh` | Service startup script | `/usr/sbin/` |
| `qlcplus.service` | Systemd service definition | `/lib/systemd/system/` |
| `eglfs.json` | GPU device configuration | `~/.qlcplus/` |
| `autostart.qxw` | Default workspace | `~/.qlcplus/` |
| `config.txt` | Raspberry Pi hardware config | `/boot/firmware/` |

### Service Management Commands
```bash
# Start QLC+ service
sudo service qlcplus start

# Alternative systemctl commands
sudo systemctl start qlcplus
sudo systemctl stop qlcplus
sudo systemctl restart qlcplus
sudo systemctl status qlcplus
```

## External Hardware

### Behringer X-Touch Configuration
![Behringer QLC+ Input/Output Settings](Behringer_QLCplus-Input-Output_settings.png)

**Input/Output Configuration**:
- **MIDI Input**: Behringer X-Touch Universal Controller Surface (CTRL Mode)
- **MIDI Output**: Same device for feedback
- **Profile**: Behringer X-Touch Universal Controller Surface

## Troubleshooting

### Common Issues and Solutions

#### UI Elements Too Large/Small
- **Cause**: Incorrect EGLFS physical dimension settings don't match your actual screen size
- **Symptoms**: 
  - Fonts appear oversized compared to Windows version
  - Buttons and widgets are disproportionately large
  - UI doesn't utilize screen space efficiently
  - Touch targets are misaligned

**Diagnostic Steps**:
1. **Measure your screen**: Get actual physical dimensions in centimeters
2. **Check current settings**: `grep QT_QPA_EGLFS_PHYSICAL /usr/sbin/qlcplus-start.sh`
3. **Compare values**: Current settings vs. actual screen size

**Solution Process**:
```bash
# 1. Backup current configuration
sudo cp /usr/sbin/qlcplus-start.sh /usr/sbin/qlcplus-start.sh.backup

# 2. Update physical dimensions (example for 35x22cm screen)
sudo sed -i 's/QT_QPA_EGLFS_PHYSICAL_WIDTH=.*/QT_QPA_EGLFS_PHYSICAL_WIDTH=350/' /usr/sbin/qlcplus-start.sh
sudo sed -i 's/QT_QPA_EGLFS_PHYSICAL_HEIGHT=.*/QT_QPA_EGLFS_PHYSICAL_HEIGHT=220/' /usr/sbin/qlcplus-start.sh

# 3. Apply changes
sudo systemctl restart qlcplus

# 4. Verify service status
sudo systemctl status qlcplus
```

**Common Screen Dimension Examples**:
- **Small Pi touchscreens (5")**: 110x65mm
- **Medium displays (7")**: 155x87mm  
- **Large touchscreens (35x22cm)**: 350x220mm

**Note**: Values are in millimeters. Convert cm to mm by multiplying by 10.

#### Display Not Showing
- **Check**: HDMI connection and `hdmi_force_hotplug=1` in config.txt
- **Alternative**: Enable VNC mode for remote access

#### Touch Input Not Working
- **Check**: `dtoverlay=vc4-fkms-v3d` in config.txt
- **Check**: `dtparam=spi=on` enabled

#### Service Won't Start
- **Check**: Service status with `sudo systemctl status qlcplus`
- **Check**: Log output with `journalctl -u qlcplus`

#### Git Repository Issues
- **Backup**: Always preserve `eglfs.json` during git operations
- **Reset**: Use git restore to reset local changes

### Useful Commands for Diagnostics
```bash
# Check Qt platform and environment
ps aux | grep qlcplus

# View service logs
journalctl -u qlcplus -f

# Test display output
/bin/dd if=/dev/zero of=/dev/fb0 >/dev/null

# Check display connection
kmsprint -m | grep connected
```

---

**Last Updated**: December 2024  
**QLC+ Version**: 4.14.1  
**Raspberry Pi OS**: Current stable  
**Hardware**: Raspberry Pi 4 Model B with 5-inch DSI touchscreen