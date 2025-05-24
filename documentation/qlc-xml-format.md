# QLC+ XML Format Documentation

This document describes the structure of the QLC+ XML format to make it easier for LLMs to understand and modify QLC+ files.

## File Structure Overview

QLC+ uses XML files with the `.qxw` extension for workspace files. The main structure is:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE Workspace>
<Workspace xmlns="http://www.qlcplus.org/Workspace" CurrentWindow="VirtualConsole">
  <Creator>
    <!-- Creator information -->
  </Creator>
  <Engine>
    <!-- Configuration for inputs, outputs, fixtures, functions, etc. -->
  </Engine>
  <VirtualConsole>
    <!-- UI configuration -->
  </VirtualConsole>
</Workspace>
```

## Key Elements

### Input/Output Configuration

The system uses ArtNet networking for lighting control across multiple universes:

```xml
<InputOutputMap>
  <BeatGenerator BeatType="Disabled" BPM="0"/>
  <Universe Name="Universe 0 - Rack" ID="0">
    <Output Plugin="ArtNet" UID="192.168.0.89" Line="9"/>
  </Universe>
  <Universe Name="Universe 1 - Lightbar" ID="1">
    <Output Plugin="ArtNet" UID="172.24.32.1" Line="7"/>
  </Universe>
  <Universe Name="Universe - Behringer XTouch" ID="4">
    <Input Plugin="MIDI" UID="None" Line="0" Profile="Behringer X-Touch Universal Controller Surface (CTRL Mode)"/>
    <Feedback Plugin="MIDI" UID="None" Line="1"/>
  </Universe>
</InputOutputMap>
```

### Fixture Types and Configuration

The system includes several fixture types with specific addressing:

#### Moving Heads (UKing ZQ02024)
- **Fixture IDs 2, 3, 4**: Mini Gobo Moving Heads (12 channels each)
- **Position mapping**: ID 2 (Right), ID 3 (Center), ID 4 (Left)
- **Universe**: 0
- **Addresses**: 0, 26, 13 respectively
- **Special**: ID 4 has channel inversion modifier

#### Spider Moving Heads (UKing ZQ02089)
- **Fixture IDs 5, 6**: 48W 8LEDs RGBW Spider Moving Heads (15 channels each)
- **Position mapping**: ID 5 (Right), ID 6 (Left)
- **Universe**: 0
- **Addresses**: 39, 55

#### PAR Lights (UKing ZQ01024)
- **Fixture IDs 7, 8**: PAR Light 9LEDs RGBW (8 channels each)
- **Position mapping**: ID 7 (Right), ID 8 (Left)
- **Universe**: 0
- **Addresses**: 71, 79

#### Generic RGB Strobe
- **Fixture ID 9**: Center strobe light (5 channels)
- **Universe**: 0
- **Address**: 89

#### Laser (UKing ZQ03058)
- **Fixture ID 10**: 9-eye laser strobe light (7 channels)
- **Universe**: 0
- **Address**: 95

#### Lightbars (Generic RGBW)
- **Fixture IDs 1, 11, 12**: RGBW lightbars (4 channels each)
- **Universe**: 1
- **Addresses**: 0, 5, 10

#### Party Bar (ETEC LED Partybar 2)
- **Fixture ID 13**: 15 channel party bar
- **Universe**: 1
- **Address**: 29

### ChannelsGroups

Channel groups organize fixture controls by function:

```xml
<ChannelsGroup ID="20" Name="Intensity" Value="0">5,2,6,2,7,0,8,0</ChannelsGroup>
<ChannelsGroup ID="0" Name="Red channel" Value="0">5,4,5,8,6,4,6,8,7,4,8,4,10,1,10,2</ChannelsGroup>
<ChannelsGroup ID="4" Name="Moving Head - Color" Value="0">2,4,3,4,4,4</ChannelsGroup>
<ChannelsGroup ID="19" Name="PARs Red" Value="0">7,4,8,4,9,0</ChannelsGroup>
```

The values reference fixture IDs and channel numbers in the format: `fixture_id,channel_index`.

### Functions and Naming Convention

The configuration follows a systematic naming convention:

#### Scene Functions (SCN)
- **Format**: `SCN_[FIXTURE]:[POSITION]:[COLOR]:[EFFECT]`
- **Examples**:
  - `SCN_MH:L:R:STT` - Moving Head Left Red Static
  - `SCN_PAR:RCL:C` - PAR Right Center Left Cyan
  - `SCN_LSR:C:G` - Laser Center Green

#### Effect Functions (EFX)
- **Format**: `EFX_[FIXTURE]:[POSITION]:[COLOR]:[MOVEMENT]`
- **Examples**:
  - `EFX_MH:L:RGB:CIRL` - Moving Head Left RGB Circle Left
  - `EFX_MH:RCL:RGB:LNUD` - Moving Head All RGB Line Up/Down

#### Chaser Functions (CHR)
- **Format**: `CHR_[FIXTURE]:[POSITION]:[COLOR]:[EFFECT]`
- **Examples**:
  - `CHR_PAR:RCL:R:NR` - PAR All Red Night Rider
  - `CHR_PB:A:RGB:LOOP` - Party Bar All RGB Loop

### Position Codes
- **R**: Right
- **C**: Center  
- **L**: Left
- **RCL**: Right Center Left (All)
- **RL**: Right Left
- **A**: All

### Color Codes
- **R**: Red
- **G**: Green
- **B**: Blue
- **W**: White
- **C**: Cyan (Green+Blue)
- **Y**: Yellow (Red+Green)
- **M**: Magenta (Red+Blue)
- **P**: Purple
- **O**: Orange
- **RGB**: Full color
- **MC**: Multiple colors

### Movement/Effect Codes
- **STT**: Static
- **CIRL**: Circle Left
- **CIRR**: Circle Right
- **LNUD**: Line Up/Down
- **CHY**: Choppy
- **NR**: Night Rider
- **AW**: Away
- **IN**: Inward
- **LOOP**: Loop

## Lighting Color Combinations

For RGB lighting fixtures, there are 7 possible color combinations when each channel is either fully on (255) or off:

1. **R** - Red only
2. **G** - Green only
3. **B** - Blue only
4. **RG** - Red + Green = Yellow
5. **RB** - Red + Blue = Magenta
6. **GB** - Green + Blue = Cyan
7. **RGB** - Red + Green + Blue = White

These combinations create the standard lighting color palette. Note that the order of colors doesn't matter for the visual output - only which channels are active and at what intensity levels.

## PAR Scene Patterns

The QLC+ configuration uses a systematic approach for PAR light scenes. For each color or color combination, there are multiple variants that determine which fixtures are affected:

1. **RCL** (Right Center Left) - Activates all PAR fixtures with the specified color
2. **C** (Center) - Activates only the center PAR fixture (ID 9 - Generic RGB Strobe)
3. **L** (Left) - Activates only the left PAR fixture (ID 8)
4. **R** (Right) - Activates only the right PAR fixture (ID 7)

### PAR Channel Mappings
For PAR fixtures 7/8 (ZQ01024):
- Channel 0: Total dimming
- Channel 1: Total strobe
- Channel 2: Function selection
- Channel 3: Function speed
- Channel 4: Red
- Channel 5: Green
- Channel 6: Blue
- Channel 7: White

For fixture 9 (Generic RGB Strobe):
- Channel 0: Red
- Channel 1: Green
- Channel 2: Blue

#### Color Groups

The following color combinations are defined:

#### Basic Colors
- RED (R) - Uses channel 4 for PAR fixtures 7/8 and channel 0 for fixture 9
- GREEN (G) - Uses channel 5 for PAR fixtures 7/8 and channel 1 for fixture 9
- BLUE (B) - Uses channel 6 for PAR fixtures 7/8 and channel 2 for fixture 9
- WHITE (W) - Uses channel 7 for PAR fixtures 7/8

#### Combined Colors
- CYAN (C) - Combination of green and blue channels
- YELLOW (Y) - Combination of red and green channels
- MAGENTA (M) - Combination of red and blue channels

## MovingHead Scene Patterns

The MovingHead fixtures are arranged with LEFT, CENTER, and RIGHT positions:

- **MovingHead - L**: Left MovingHead (Fixture ID 4)
- **MovingHead - C**: Center MovingHead (Fixture ID 3)
- **MovingHead - R**: Right MovingHead (Fixture ID 2)
- **MovingHead - RCL**: All three MovingHeads (Right, Center, Left)

### MovingHead Channel Mappings
For MovingHead fixtures (ZQ02024):
- Channel 0: Pan
- Channel 1: Pan fine
- Channel 2: Tilt
- Channel 3: Tilt fine
- Channel 4: Color
- Channel 5: Gobo
- Channel 6: Strobe
- Channel 7: Dimmer
- Channel 8: Focus
- Channel 9: Gobo effects
- Channel 10: Movement effects
- Channel 11: LEDs

For each position, there are scenes for different colors:

- **WHITE**: Channel 4 value 0-9 (typically 5)
- **RED**: Channel 4 value 10-19 (typically 13)
- **GREEN**: Channel 4 value 20-29 (typically 25)
- **BLUE**: Channel 4 value 30-39 (typically 35)
- **YELLOW**: Channel 4 value 40-49 (typically 45)
- **ORANGE**: Channel 4 value 50-59 (typically 55)
- **CYAN**: Channel 4 value 60-69 (typically 65)
- **PURPLE**: Channel 4 value 70-79 (typically 75)

For single fixtures (L, C, R positions), the position parameters are also set:
- Pan value (channel 0)
- Tilt value (channel 2)

Example structure:
```xml
<Function ID="113" Type="Scene" Name="SCN_MH:L:P:STT (LeftPurple)" Path="MovingHead">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="4">0,119,2,37,4,75,7,255</FixtureVal>
</Function>
```

In this example:
- Fixture ID 4 (Left MovingHead)
- Channel 0 (Pan) = 119
- Channel 2 (Tilt) = 37
- Channel 4 (Color) = 75 (PURPLE)
- Channel 7 (Dimmer) = 255 (full brightness)

## Spider Light Patterns

Spider lights (fixtures 5 and 6) have specialized effects:

### Spider Channel Mappings
For Spider fixtures (ZQ02089):
- Channel 0: Motor 1 (Pan)
- Channel 1: Motor 2 (Tilt)
- Channel 2: Dimmer
- Channel 3: Strobe
- Channel 4: Red
- Channel 5: Green
- Channel 6: Blue
- Channel 7: White
- Channel 8: Red (back LEDs)
- Channel 9: Green (back LEDs)
- Channel 10: Blue (back LEDs)
- Channel 11: White (back LEDs)
- Channel 12: Effects
- Channel 13: Effect speed
- Channel 14: Reset

Spider scenes use the format:
- `SCN_SPD:[POSITION]:[COLOR]:[LED_GROUP]`
- Where LED_GROUP can be F (Front), B (Back), or FB (Front+Back)

## Laser Light Patterns

Laser light (fixture 10) has color combination scenes:

### Laser Channel Mappings
For Laser fixture (ZQ03058):
- Channel 0: Control
- Channel 1: Red
- Channel 2: Red strobe
- Channel 3: Green
- Channel 4: Green strobe
- Channel 5: Blue
- Channel 6: Blue strobe

Laser scenes use combinations:
- `SCN_LSR:[POSITION]:[COLOR_COMBO]`
- Examples: RB (Red+Blue), RG (Red+Green), GB (Green+Blue)

## Virtual Console Organization

The Virtual Console is organized into functional frames:

### Frame Structure
- **16 Colours - Generic RGB Strobe**: Color preset buttons for the center strobe
- **PAR Controls**: Position and color controls for PAR lights
- **Moving Head Controls**: Position, color, and effect controls for moving heads
- **Master Controls**: Overall system controls and emergency stops

### Button Organization
Buttons are typically organized in grids:
- Color buttons use background colors matching their function
- Position buttons are grouped logically (L-C-R)
- Emergency/stop functions are prominently placed

## Actual Virtual Console Layout (Current Implementation)

Based on the live QLC+ interface, the Virtual Console is organized into several specialized control sections:

### Left Panel: Fixture Control Section
**FIXTURES - MovingHeads** frame provides direct hardware control:

#### Individual Channel Control
- **Dim**: Dimmer/intensity slider (0-100%)
- **Strobe**: Strobe rate control
- **Color**: Color wheel selection  
- **Gobo**: Gobo pattern selection
- **Pan**: Horizontal positioning
- **Tilt**: Vertical positioning
- **Gobo Effect**: Gobo rotation and effects
- **Move Effect**: Movement pattern effects
- **Ring LEDs**: LED ring control

#### Control Features
- Real-time sliders for immediate fixture adjustment
- Individual channel percentage display (000% format)
- Direct hardware manipulation capability
- Button 320 for additional functions

### Center Panel: 2D Position Control
**MovingHeads - Right** provides visual positioning interface:

#### 2D Positioning Grid
- **Visual crosshair target**: Shows current Pan/Tilt position
- **Interactive positioning**: Click-to-position functionality
- **Coordinate display**: Shows exact X/Y coordinates (e.g., "Red 658 %, Tilt 072 %")
- **Real-time feedback**: Live position visualization
- **Precise control**: Alternative to slider-based positioning

#### Features
- Intuitive visual positioning
- Real-time position feedback
- Coordinate-based precision
- Visual representation of movement range

### Right Panel: Function Matrix and Controls

#### Button Matrix Grid (Buttons 300-338+)
Organized in systematic 6-column layout:
- **Row 1**: Buttons 300, 301, 302, 321, 322, 323
- **Row 2**: Buttons 303, 304, 305, 324, 325, 326  
- **Row 3**: Buttons 306, 307, 308, 327, 328, 329
- **Row 4**: Buttons 309, 310, 311, 330, 331, 332
- **Row 5**: Buttons 312, 313, 314, 333, 334, 335
- **Row 6**: Buttons 315, 316, 317, 336, 337, 338

**Purpose**: Likely scene/function triggers organized by fixture type or color

#### Function Control Section
**Purpose**: Show/sequence timing control
- **Duration controls**: Tap tempo and timing adjustment
- **Playback controls**: Start/stop/pause functionality  
- **Timing display**: Shows current duration settings (1x, 0ms format)
- **Speed controls**: Rate adjustment for sequences

#### Audio Control Section  
**Purpose**: Audio-responsive lighting control
- **Duration settings**: Audio sync timing
- **Audio integration**: Music-synchronized effects
- **Tempo controls**: Beat-matching functionality

#### Audio Trigger Section
**Purpose**: Real-time audio analysis and triggering
- **Spectrum analyzer**: Visual frequency analysis display
- **Audio input monitoring**: Real-time audio level visualization
- **Trigger sensitivity**: Audio-reactive threshold controls
- **Frequency bands**: Multi-band audio analysis for different fixture responses

### Bottom Panel: Cue List and Playback

#### Cue List Display
- **Active sequences**: Shows running sequences (e.g., "MovingHead RL - InverseCircle")
- **Timing information**: Duration and fade settings
- **Status indicators**: Shows sequence progress and state

#### Playback Controls
- **Transport controls**: Play, pause, stop, forward/back
- **Sequence management**: Cue list navigation
- **Manual override**: Direct sequence control

## Advanced UI Features Identified

### Real-time Control Integration
1. **Multi-modal control**: Sliders, 2D positioning, and button triggers
2. **Visual feedback**: Live position and status indicators  
3. **Audio synchronization**: Integrated audio analysis and triggering
4. **Precision control**: Percentage-based channel control with exact values

### Professional Features
1. **Cue list management**: Professional sequence control
2. **Audio reactive**: Real-time music synchronization
3. **Visual positioning**: 2D interface for moving fixtures
4. **Channel-level access**: Direct DMX channel manipulation

### User Experience Design
1. **Logical grouping**: Controls organized by function type
2. **Visual hierarchy**: Important controls prominently placed
3. **Real-time feedback**: Live status and position indication
4. **Professional layout**: Industry-standard control paradigms

## Button Matrix Mapping Analysis

Based on the visible UI layout showing buttons 300-338, here's the likely function mapping structure:

### Current Button Matrix Layout (6x6+ Grid)
```
Button Layout (Left to Right, Top to Bottom):
Row 1: 300  301  302  |  321  322  323
Row 2: 303  304  305  |  324  325  326  
Row 3: 306  307  308  |  327  328  329
Row 4: 309  310  311  |  330  331  332
Row 5: 312  313  314  |  333  334  335
Row 6: 315  316  317  |  336  337  338
```

### Probable Function Assignments

#### Left Column Group (300-317): Moving Head Controls
Based on the comprehensive Moving Head scenes in the configuration:
- **300-302**: MH Left position colors (Red, Green, Blue)
- **303-305**: MH Center position colors (Red, Green, Blue)  
- **306-308**: MH Right position colors (Red, Green, Blue)
- **309-311**: MH All positions colors (Red, Green, Blue)
- **312-314**: MH Additional colors (White, Yellow, Cyan)
- **315-317**: MH Special colors (Purple, Orange, effects)

#### Right Column Group (321-338+): Multi-Fixture Controls
Likely organized by fixture type and common combinations:
- **321-323**: PAR Light controls (Red, Green, Blue positions)
- **324-326**: Spider Light controls (Color/position combinations)
- **327-329**: Laser controls (Color combinations)
- **330-332**: Strobe/Party Bar controls
- **333-335**: System controls (Blackout, House, Master)
- **336-338+**: Special effects and chasers

### Virtual Console Button Configuration Pattern

The button matrix appears to follow this organizational pattern:
```xml
<!-- Moving Head Scenes (300-317) -->
<Button Caption="MH-L-RED" ID="300" Function="24"/>    <!-- SCN_MH:L:R:STT -->
<Button Caption="MH-L-GRN" ID="301" Function="97"/>    <!-- SCN_MH:L:G:STT -->
<Button Caption="MH-L-BLU" ID="302" Function="101"/>   <!-- SCN_MH:L:B:STT -->

<!-- Multi-Fixture Scenes (321-338+) -->
<Button Caption="PAR-ALL-RED" ID="321" Function="68"/>  <!-- SCN_PAR:RCL:R -->
<Button Caption="PAR-ALL-GRN" ID="322" Function="69"/>  <!-- SCN_PAR:RCL:G -->
<Button Caption="PAR-ALL-BLU" ID="323" Function="70"/>  <!-- SCN_PAR:RCL:B -->
```

### Button Color Coding Strategy
The interface likely uses background colors to match function purposes:
- **Red buttons**: Red lighting scenes
- **Green buttons**: Green lighting scenes
- **Blue buttons**: Blue lighting scenes
- **White buttons**: White/house lighting
- **Yellow buttons**: Yellow/amber lighting
- **Cyan buttons**: Cyan lighting
- **Magenta buttons**: Magenta lighting
- **Orange buttons**: Orange lighting
- **Gray buttons**: System controls (blackout, stop, etc.)

This color-coding provides immediate visual feedback about the lighting effect each button will trigger.

## Best Practices for Modifying QLC+ XML

1. Always maintain unique Function IDs when adding new functions
2. Preserve existing structures and naming conventions
3. Test changes incrementally rather than making large batch modifications
4. Back up the original file before making changes
5. Make sure the XML remains well-formed
6. Follow the established naming convention for consistency
7. Maintain position mappings (R=Right, C=Center, L=Left)
8. Use appropriate channel values for different fixture types

## How to Modify QLC+ XML Files

### Step 1: Understanding the File Structure

Before making changes, analyze the file to understand:
- The highest Function ID currently in use (to avoid duplicates)
- The naming conventions used for different function types
- The patterns used for similar functions (e.g., PAR scenes)

### Step 2: Adding New Scenes

When adding new scenes, follow these guidelines:

1. **Use sequential Function IDs**: Start from the highest existing ID + 1
   ```xml
   <Function ID="128" Type="Scene" Name="SCN_PAR:L:O" Path="PAR">
   ```

2. **Maintain consistent naming conventions**:
   ```xml
   Name="SCN_PAR:L:O" Path="PAR"
   ```

3. **Copy the structure of similar existing scenes**: For example, when adding a new PAR scene, use the same structure as existing PAR scenes:
   ```xml
   <Function ID="128" Type="Scene" Name="SCN_PAR:L:O" Path="PAR">
    <Speed FadeIn="0" FadeOut="0" Duration="0"/>
    <FixtureVal ID="8">0,255,4,255,5,128</FixtureVal>
   </Function>
   ```

4. **Understand channel mappings**: For fixtures like PAR lights:
   - Channel 0: Total dimming
   - Channel 4: Red
   - Channel 5: Green
   - Channel 6: Blue
   - Channel 7: White
   
   Set channels like `<FixtureVal ID="8">0,255,5,255,6,255</FixtureVal>` for a cyan light (green+blue)

### Step 3: Updating Chasers

After adding new scenes, update any chasers that should include them:

1. **Locate the chaser function** that should include your new scenes:
   ```xml
   <Function ID="77" Type="Chaser" Name="CHR_PAR:RCL:MC:RND (RandomColor)" Path="Color">
   ```

2. **Add steps sequentially** with your new scene IDs:
   ```xml
   <Step Number="13" FadeIn="0" Hold="200" FadeOut="0">128</Step>
   <Step Number="14" FadeIn="0" Hold="200" FadeOut="0">129</Step>
   ```

3. **Maintain step numbering** starting from the last existing step number

### Step 4: Testing Changes

1. Make incremental changes and test them after each modification
2. If the file doesn't load in QLC+, check for:
   - Duplicate Function IDs
   - Malformed XML
   - Missing closing tags
   - Incorrect channel values or fixture IDs

### Example: Adding New Color Positions for PAR Lights

To add CENTER, LEFT, and RIGHT positions for a specific color (e.g., ORANGE):

```xml
<Function ID="128" Type="Scene" Name="SCN_PAR:C:O" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="9">0,255,1,128</FixtureVal>
</Function>
<Function ID="129" Type="Scene" Name="SCN_PAR:L:O" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="8">0,255,4,255,5,128</FixtureVal>
</Function>
<Function ID="130" Type="Scene" Name="SCN_PAR:R:O" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="7">0,255,4,255,5,128</FixtureVal>
</Function>
```

Note how each position variant activates different fixtures:
- CENTER: Sets values for fixture ID 9 (center strobe - using mixed red+green for orange)
- LEFT: Sets values only for fixture ID 8 (left PAR - using red+green channels)
- RIGHT: Sets values only for fixture ID 7 (right PAR - using red+green channels)

## Comprehensive Naming Convention

To maintain consistency across all functions and fixtures, the following standardized naming convention is recommended:

```
[Function-Type]:[Fixture-Type]:[Position]:[Color]:[Movement/Orientation]:[Effect]
```

Where:

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
Indicates physical placement:
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
- **MC** - Multiple colors (for color transitions)

### Movement/Orientation (When applicable)
For moving fixtures:
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

### Examples:

- `SCN:MH:L:R:STT` - Scene for Left Moving Head, Red color, Static position
- `SCN:PAR:RCL:C` - Scene for all PAR lights (Right, Center, Left), Cyan color
- `EFX:MH:RCL:RGB:CIRL` - Effect for all Moving Heads, RGB colors, Circle Left movement
- `CHR:LSR:RL:R:ALT` - Chaser for Right and Left Lasers, Red color, Alternating effect
- `SCN:SPD:R:B:F` - Scene for Right Spider, Blue color, Front LEDs only

### Implementation Strategy

When implementing this naming convention:
1. Separate each component with a colon (:)
2. Use abbreviations consistently
3. Omit components that don't apply to a particular function
4. Maintain the order of components for easy parsing

This convention allows for quick visual identification of function purpose and properties, making programming and operation more efficient. 