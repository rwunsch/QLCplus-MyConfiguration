# QLC+ XML Format Documentation

This document describes the structure of the QLC+ XML format to make it easier for LLMs to understand and modify QLC+ files.

## File Structure Overview

QLC+ uses XML files with the `.qxw` extension for workspace files. The main structure is:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE Workspace>
<Workspace xmlns="http://www.qlcplus.org/Workspace" CurrentWindow="FunctionManager">
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

### Fixtures

Fixtures are defined with unique IDs and contain information about the fixture type, universe, address, and channels:

```xml
<Fixture>
  <Manufacturer>UKing</Manufacturer>
  <Model>ZQ01024 - PAR Light 9LEDs 4-in-1 RGBW</Model>
  <Mode>8 channel</Mode>
  <ID>7</ID>
  <n>ZQ01024 - PAR Light 9LEDs 4-in-1 RGBW #1 Rechts</n>
  <Universe>0</Universe>
  <Address>71</Address>
  <Channels>8</Channels>
</Fixture>
```

### ChannelsGroups

Channel groups combine related channels across fixtures for easier control:

```xml
<ChannelsGroup ID="19" Name="PARs Red" Value="0">7,4,8,4,9,0</ChannelsGroup>
<ChannelsGroup ID="21" Name="PARs Green" Value="0">7,5,8,5,9,1</ChannelsGroup>
<ChannelsGroup ID="22" Name="PARs Blue" Value="0">7,6,8,6,9,2</ChannelsGroup>
<ChannelsGroup ID="23" Name="PARs White" Value="0">7,7,8,7</ChannelsGroup>
```

The values in these groups reference fixture IDs and channel numbers, in the format: `fixture_id,channel_index`.

### Functions

Functions define scenes, chasers, EFX (effects), and other operations:

#### Scenes

Scenes set specific fixture values:

```xml
<Function ID="68" Type="Scene" Name="PAR RED ALL" Path="PAR">
  <Speed FadeIn="0" FadeOut="0" Duration="0"/>
  <FixtureVal ID="9">0,255</FixtureVal>
  <FixtureVal ID="7">0,255,4,255</FixtureVal>
  <FixtureVal ID="8">0,255,4,255</FixtureVal>
  <FixtureVal ID="5"/>
  <FixtureVal ID="6"/>
</Function>
```

The `FixtureVal` elements specify channel values for each fixture in the format `channel_index,value`.

For PAR fixtures with ID 7 and 8 (ZQ01024 - PAR Light 9LEDs 4-in-1 RGBW), the channels are:
- Channel 0: Total dimming
- Channel 1: Total strobe
- Channel 2: Function selection
- Channel 3: Function speed
- Channel 4: Red
- Channel 5: Green
- Channel 6: Blue
- Channel 7: White

For fixture with ID 9 (Generic RGB Strobe), the channels are:
- Channel 0: Red
- Channel 1: Green
- Channel 2: Blue

#### Chasers

Chasers run a sequence of scenes or other functions:

```xml
<Function ID="77" Type="Chaser" Name="PAR COLOR" Path="Color">
  <Speed FadeIn="0" FadeOut="0" Duration="200"/>
  <Direction>Forward</Direction>
  <RunOrder>Random</RunOrder>
  <SpeedModes FadeIn="Default" FadeOut="Default" Duration="Common"/>
  <Step Number="0" FadeIn="0" Hold="0" FadeOut="0">70</Step>
  <Step Number="1" FadeIn="0" Hold="0" FadeOut="0">68</Step>
  <!-- More steps -->
</Function>
```

Each step references another function by ID.

#### EFX (Effects)

EFX generate movement patterns for fixtures:

```xml
<Function ID="7" Type="EFX" Name="MovingHead L - CirlceLeft">
  <Fixture>
    <ID>4</ID>
    <Head>0</Head>
    <Mode>0</Mode>
    <Direction>Forward</Direction>
    <StartOffset>0</StartOffset>
  </Fixture>
  <PropagationMode>Parallel</PropagationMode>
  <Speed FadeIn="0" FadeOut="0" Duration="3000"/>
  <Direction>Forward</Direction>
  <RunOrder>Loop</RunOrder>
  <Algorithm>Circle</Algorithm>
  <!-- More configuration -->
</Function>
```

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

The QLC+ configuration uses a systematic approach for PAR light scenes. For each color or color combination, there are four variants that determine which fixtures are affected:

1. **ALL** - Activates all PAR fixtures with the specified color
2. **CENTER** - Activates only the center PAR fixture(s)
3. **LEFT** - Activates only the left PAR fixture(s)
4. **RIGHT** - Activates only the right PAR fixture(s)

### Color Groups

The following color combinations are defined:

#### Basic Colors
- RED (R) - Uses channel 4 for PAR fixtures 7/8 and channel 0 for fixture 9
- GREEN (G) - Uses channel 5 for PAR fixtures 7/8 and channel 1 for fixture 9
- BLUE (B) - Uses channel 6 for PAR fixtures 7/8 and channel 2 for fixture 9
- WHITE (W) - Uses channel 7 for PAR fixtures 7/8

#### Combined Colors
- CYAN (GB) - Combination of green and blue channels
- YELLOW (RG) - Combination of red and green channels
- MAGENTA (RB) - Combination of red and blue channels

This organized scene structure allows for easy control of color and position combinations through the virtual console or chasers.

## MovingHead Scene Patterns

The MovingHead fixtures are arranged with LEFT, CENTER, and RIGHT positions:

- **MovingHead - L**: Left MovingHead (Fixture ID 4)
- **MovingHead - C**: Center MovingHead (Fixture ID 3)
- **MovingHead - R**: Right MovingHead (Fixture ID 2)
- **MovingHead - RCL**: All three MovingHeads (Right, Center, Left)

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
<Function ID="113" Type="Scene" Name="MovingHead - L - PURPLE" Path="MovingHead">
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

## Best Practices for Modifying QLC+ XML

1. Always maintain unique Function IDs when adding new functions
2. Preserve existing structures and naming conventions
3. Test changes incrementally rather than making large batch modifications
4. Back up the original file before making changes
5. Make sure the XML remains well-formed

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
   <Function ID="85" Type="Scene" Name="PAR WHITE CENTER" Path="PAR">
   ```

2. **Maintain consistent naming conventions**:
   ```xml
   Name="PAR WHITE CENTER" Path="PAR"
   ```

3. **Copy the structure of similar existing scenes**: For example, when adding a new PAR scene, use the same structure as existing PAR scenes:
   ```xml
   <Function ID="85" Type="Scene" Name="PAR WHITE CENTER" Path="PAR">
    <Speed FadeIn="0" FadeOut="0" Duration="0"/>
    <FixtureVal ID="9"/>
    <FixtureVal ID="7"/>
    <FixtureVal ID="8"/>
    <FixtureVal ID="5"/>
    <FixtureVal ID="6"/>
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
   <Function ID="77" Type="Chaser" Name="PAR COLOR" Path="Color">
   ```

2. **Add steps sequentially** with your new scene IDs:
   ```xml
   <Step Number="13" FadeIn="0" Hold="200" FadeOut="0">85</Step>
   <Step Number="14" FadeIn="0" Hold="200" FadeOut="0">86</Step>
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

To add CENTER, LEFT, and RIGHT positions for a specific color (e.g., CYAN):

```xml
<Function ID="88" Type="Scene" Name="PAR CYAN CENTER" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="9">1,255,2,255</FixtureVal>
 <FixtureVal ID="7"/>
 <FixtureVal ID="8"/>
 <FixtureVal ID="5"/>
 <FixtureVal ID="6"/>
</Function>
<Function ID="89" Type="Scene" Name="PAR CYAN LEFT" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="8">0,255,5,255,6,255</FixtureVal>
</Function>
<Function ID="90" Type="Scene" Name="PAR CYAN RIGHT" Path="PAR">
 <Speed FadeIn="0" FadeOut="0" Duration="0"/>
 <FixtureVal ID="7">0,255,5,255,6,255</FixtureVal>
</Function>
```

Note how each position variant activates different fixtures:
- CENTER: Sets values for fixture ID 9 (center light)
- LEFT: Sets values only for fixture ID 8 (left light)
- RIGHT: Sets values only for fixture ID 7 (right light)

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