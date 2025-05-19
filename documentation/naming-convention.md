# QLC+ Naming Convention

## Overview

This document describes the comprehensive naming convention for QLC+ configuration files to ensure consistency and maintainability.

## Naming Format

The naming convention follows this structured format:
```
[Function-Type]_[Fixture-Type]:[Position]:[Color]:[Movement/Orientation]:[Effect]
```

Note that an underscore "_" is used after the Function Type, while colons ":" separate all other components.

### Function Type (Required)
- **SCN** - Scene
- **CHR** - Chaser
- **EFX** - Effect
- **COL** - Collection
- **SHW** - Show
- **SCP** - Script

### Fixture Type (Required)
- **MH** - Moving Head
- **PAR** - PAR Light
- **SPD** - Spider
- **LSR** - Laser
- **STB** - Strobe
- **PB** - Party Bar

### Position (Required)
- **R** - Right
- **C** - Center
- **L** - Left
- **A** - All
- **RC** - Right+Center
- **CL** - Center+Left
- **RL** - Right+Left
- **RCL** - Right+Center+Left
- **F** - Front (for fixtures with front/back orientation)
- **B** - Back (for fixtures with front/back orientation)
- **FB** - Front+Back

### Color (When applicable)
- **R** - Red
- **G** - Green
- **B** - Blue
- **W** - White
- **C** - Cyan (Green+Blue)
- **M** - Magenta (Red+Blue)
- **Y** - Yellow (Red+Green)
- **O** - Orange
- **P** - Purple
- **RGB** - Full color
- **MC** - Multiple colors

### Movement/Orientation (When applicable)
- **CIR** - Circle
- **CIRL** - Circle Left
- **CIRR** - Circle Right
- **LN** - Line
- **LNUD** - Line Up/Down
- **LNLR** - Line Left/Right
- **CHY** - Choppy
- **AW** - Away (from center)
- **IN** - Inward (toward center)
- **STT** - Static

### Effect (Optional)
- **STB** - Strobe
- **PLS** - Pulse
- **ALT** - Alternating
- **RND** - Random
- **FD** - Fade
- **LOOP** - Looping
- **NR** - Night Rider effect (ping-pong)

## Implementation Guidelines

1. Use an underscore "_" after the Function Type (SCN, CHR, EFX, etc.)
2. Use colons ":" as separators between all other components
3. Keep abbreviations consistent
4. Omit components that don't apply (e.g., movement for static scenes)
5. Keep the order consistent

## Example Definitions

Below are examples of applying the naming convention to various QLC+ functions:

### Moving Head Scenes

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `MovingHead - L - RED` | `SCN_MH:L:R:STT` | Scene for Left Moving Head, Red color, Static position |
| `MovingHead - C - GREEN` | `SCN_MH:C:G:STT` | Scene for Center Moving Head, Green color, Static position |
| `MovingHead - R - BLUE` | `SCN_MH:R:B:STT` | Scene for Right Moving Head, Blue color, Static position |
| `MovingHead - RCL - CYAN` | `SCN_MH:RCL:C` | Scene for all Moving Heads, Cyan color |
| `MovingHead - White Away` | `SCN_MH:RCL:W:AW` | Scene for all Moving Heads, White color, Away orientation |
| `MovingHead - White Center` | `SCN_MH:RCL:W:IN` | Scene for all Moving Heads, White color, Inward orientation |

### PAR Light Scenes

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `PAR RED RIGHT` | `SCN_PAR:R:R` | Scene for Right PAR light, Red color |
| `PAR BLUE CENTER` | `SCN_PAR:C:B` | Scene for Center PAR light, Blue color |
| `PAR MAGENTA LEFT` | `SCN_PAR:L:M` | Scene for Left PAR light, Magenta color |
| `PAR GREEN ALL` | `SCN_PAR:RCL:G` | Scene for all PAR lights, Green color |

### Effects

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `MovingHead L - CirlceLeft` | `EFX_MH:L:RGB:CIRL` | Effect for Left Moving Head, RGB colors, Circle Left movement |
| `MovingHead RCL - CircleRight` | `EFX_MH:RCL:RGB:CIRR` | Effect for all Moving Heads, RGB colors, Circle Right movement |
| `MovingHead RCL - Line UpDown` | `EFX_MH:RCL:RGB:LNUD` | Effect for all Moving Heads, RGB colors, Line Up/Down movement |
| `MovingHead RCL - Choppy` | `EFX_MH:RCL:RGB:CHY` | Effect for all Moving Heads, RGB colors, Choppy movement |

### Chasers

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `PAR COLOR` | `CHR_PAR:RCL:MC:RND` | Chaser for all PAR lights, Multiple colors, Random order |
| `PAR RED (Night Rider)` | `CHR_PAR:RCL:R:NR` | Chaser for all PAR lights, Red color, Night Rider effect |
| `RGB Chaser` | `CHR_PB:A:RGB:LOOP` | Chaser for Party Bar, All positions, RGB colors, Looping |

### Spider Light Scenes

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `Spider R - RED back` | `SCN_SPD:R:R:B` | Scene for Right Spider, Red color, Back LEDs only |
| `Spider R - RED front` | `SCN_SPD:R:R:F` | Scene for Right Spider, Red color, Front LEDs only |
| `Spider R - RED` | `SCN_SPD:R:R:FB` | Scene for Right Spider, Red color, Front and Back LEDs |

### Laser Scenes

| Current Name | New Name | Description |
|-------------|----------|-------------|
| `Laser RED left` | `SCN_LSR:L:R` | Scene for Left Laser, Red color |
| `Laser RED right` | `SCN_LSR:R:R` | Scene for Right Laser, Red color |
| `Laser RED` | `SCN_LSR:RL:R` | Scene for both Left and Right Lasers, Red color |

## Automatic Renaming

The `rename_functions.py` script can automatically rename functions in your QLC+ workspace file according to the new convention.

### Usage

```bash
python3 rename_functions.py [qxw_file] [output_file]
```

If no output file is specified, it will output to `[qxw_file].new`

### Safety

The script:
1. Always creates a new file, never modifying your original file
2. Renames both Function names and Button captions that reference those functions
3. Provides a count of how many names were changed

### After Running

1. Review the output file to make sure the changes are correct
2. Test the renamed file with QLC+ to ensure it loads properly
3. Backup your original file before replacing it with the renamed version 