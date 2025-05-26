# SPIDER Fixture Improvements

## Overview
Complete enhancement of the SPIDER fixture control system with comprehensive scenes, effects, and virtual console integration. The SPIDER fixtures (ZQ02089 - 48W 8LEDs RGBW 4 in 1 Color LED Mini Spider Moving Head Stage Light) are now fully integrated with professional lighting control capabilities.

## SPIDER Fixtures
- **Fixture ID 5**: ZQ02089 - 48W 8LEDs RGBW Spider Moving Head #1 Rechts (Right)
- **Fixture ID 6**: ZQ02089 - 48W 8LEDs RGBW Spider Moving Head #2 Links (Left)
- **Mode**: 15 channel mode
- **Channels**: 
  - Ch 0: Pan (Motor 1)
  - Ch 1: Tilt (Motor 2)
  - Ch 2: Intensity/Dimmer
  - Ch 3: Strobe
  - Ch 4: Red
  - Ch 5: Green
  - Ch 6: Blue
  - Ch 7: White
  - Ch 8-11: Individual LED control
  - Ch 12: Effects
  - Ch 13: Effect Speed
  - Ch 14: Reset/Control

## New Functions Created

### Basic Color Scenes (Both SPIDERs)
- **Function 150**: SCN_SPD:RL:R (SpiderRed) - Both SPIDER fixtures red
- **Function 151**: SCN_SPD:RL:G (SpiderGreen) - Both SPIDER fixtures green  
- **Function 152**: SCN_SPD:RL:B (SpiderBlue) - Both SPIDER fixtures blue
- **Function 153**: SCN_SPD:RL:W (SpiderWhite) - Both SPIDER fixtures white
- **Function 154**: SCN_SPD:RL:C (SpiderCyan) - Both SPIDER fixtures cyan
- **Function 155**: SCN_SPD:RL:Y (SpiderYellow) - Both SPIDER fixtures yellow
- **Function 156**: SCN_SPD:RL:M (SpiderMagenta) - Both SPIDER fixtures magenta
- **Function 157**: SCN_SPD:RL:O (SpiderOrange) - Both SPIDER fixtures orange

### Individual SPIDER Control
- **Function 158**: SCN_SPD:R:R (RightSpiderRed) - Right SPIDER only red
- **Function 159**: SCN_SPD:L:R (LeftSpiderRed) - Left SPIDER only red
- **Function 160**: SCN_SPD:R:G (RightSpiderGreen) - Right SPIDER only green
- **Function 161**: SCN_SPD:L:G (LeftSpiderGreen) - Left SPIDER only green
- **Function 162**: SCN_SPD:R:B (RightSpiderBlue) - Right SPIDER only blue
- **Function 163**: SCN_SPD:L:B (LeftSpiderBlue) - Left SPIDER only blue
- **Function 164**: SCN_SPD:R:W (RightSpiderWhite) - Right SPIDER only white
- **Function 165**: SCN_SPD:L:W (LeftSpiderWhite) - Left SPIDER only white

### Special Effects
- **Function 166**: SCN_SPD:RL:STROBE (SpiderStrobe) - White strobe effect both SPIDERs
- **Function 167**: SCN_SPD:RL:EFX:AUTO1 (SpiderAutoEffect1) - Moderate auto effect
- **Function 168**: SCN_SPD:RL:EFX:AUTO2 (SpiderAutoEffect2) - Advanced auto effect  
- **Function 169**: SCN_SPD:RL:EFX:FAST (SpiderFastEffect) - Fast auto effect

### Chasers/Effects
- **Function 170**: CHR_SPD:RL:COLORLOOP (SpiderColorLoop) - 7-step color loop with smooth transitions
- **Function 171**: CHR_SPD:LR:BOUNCE (SpiderBounce) - Left-right bouncing effect
- **Function 172**: CHR_SPD:RL:RANDOM (SpiderRandom) - Random color sequence

### Collections
- **Function 173**: COL_SPD:PARTY (SpiderParty) - Party mode: Color loop + strobe
- **Function 174**: COL_SPD:WARM (SpiderWarm) - Warm lighting: Orange + white

## Virtual Console Integration (Page 2 - SPIDERs)

### Normal Matrix (Frame ID="727")
**Color Control Buttons** (3x3 grid, 50x50px):
- Row 1: SP-RED (red), SP-GRN (green), SP-BLU (blue)
- Row 2: SP-WHT (white), SP-CYN (cyan), SP-YEL (yellow)  
- Row 3: SP-MAG (magenta), SP-ORG (orange), SP-STB (strobe - Flash action)

**Effect Control Buttons** (Row 4):
- LOOP (Color loop), BOUNCE (Left-right bounce), RANDOM (Random colors)

### SoloFrame (Frame ID="746") - Exclusive Functions
**Collection Buttons** (70x50px):
- SP-PARTY (Party collection), SP-WARM (Warm collection)

**Auto Effect Buttons**:
- AUTO1 (Moderate auto), AUTO2 (Advanced auto), FAST-EFX (Fast effect)

## Design Features

### Professional Color Coding
- **Red Background**: Red, magenta (4294901760, 4294902015)
- **Green Background**: Green (4278255360) 
- **Blue Background**: Blue, cyan (4278190335, 4278255615)
- **Yellow Background**: Yellow, orange (4294967040, 4294944512)
- **White Background**: White with black text (4294967295)
- **Gray Background**: Strobe effects (4291611852)
- **Purple Background**: Effect collections (4291546175)
- **Dark Blue**: Auto effects (4286611584)

### Operational Features
- **Toggle Actions**: All color and effect buttons for combinable control
- **Flash Action**: Strobe button for momentary activation
- **Raised Button Style**: Professional appearance
- **Optimized Text Contrast**: White text on dark backgrounds, black on light
- **Touch-Optimized Sizing**: 50x50px for colors, 70x50px for collections

## Usage Scenarios

### 1. Basic Color Control
Use the 3x3 color matrix for immediate color selection. Colors are combinable for mixed lighting effects.

### 2. Dynamic Effects
- **LOOP**: Smooth 7-color progression for ambient lighting
- **BOUNCE**: Left-right movement for dynamic shows
- **RANDOM**: Unpredictable color changes for party atmosphere

### 3. Professional Collections
- **SP-PARTY**: Combines color loop with strobe for maximum impact
- **SP-WARM**: Orange and white combination for comfortable ambient lighting

### 4. Auto Effects
- **AUTO1/AUTO2**: Built-in SPIDER effects with different intensities
- **FAST-EFX**: High-speed built-in effects for dynamic shows

### 5. Strobe Control  
Independent strobe control with Flash action for momentary activation or sustained strobe effects.

## Technical Implementation

### Channel Mapping
All functions properly set:
- Channel 2 (Intensity): 255 for full brightness
- Channels 4-7 (RGBW): Color-specific values
- Channel 3 (Strobe): 240 for strobe effects
- Channels 12-13 (Effects/Speed): Auto effect control

### Function Organization
- **Path Structure**: 
  - "Spider" - Basic scenes
  - "Spider/Effects" - Chasers and auto effects
  - "Spider/Collections" - Professional collections

### Timing Configuration
- **Color Loop**: 1000ms hold, 500ms fade for smooth transitions
- **Bounce**: 300ms duration for crisp left-right movement
- **Random**: 200ms duration for dynamic color changes

## Integration Benefits

1. **Complete Control**: Full color palette access with individual and combined fixture control
2. **Professional Effects**: Built-in auto effects plus custom chasers
3. **Emergency Features**: Immediate strobe access with Flash action
4. **Touch Optimization**: Proper button sizing for Raspberry Pi touchscreen
5. **Visual Consistency**: Matches PAR control design principles
6. **Scalable Design**: Framework supports additional SPIDER fixtures

## Compatibility
- Maintains backward compatibility with existing configuration
- Integrates seamlessly with existing PAR and moving head controls
- Professional color coding matches system-wide standards
- Function IDs 150-174 safely allocated without conflicts

This implementation provides complete professional control over the SPIDER fixtures, matching the sophistication of the PAR control system while leveraging the unique capabilities of the moving head fixtures. 