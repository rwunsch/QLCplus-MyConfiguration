# QLC+ Font Tag Parameters Documentation

## Overview

QLC+ (Q Light Controller Plus) workspace files (.qxw) use XML format to store configuration data. Font styling information is stored in `<Font>` tags using Qt framework's font specification format. This document provides a comprehensive explanation of each parameter in the font specification string.

## Font Tag Structure

The Font tag follows this general format:
```xml
<Font>FontFamily,Size,StyleHint,Weight,Italic,Underline,Strikethrough,FixedPitch,RawMode,StyleName</Font>
```

### Example
```xml
<Font>DejaVu Sans Condensed,8,-1,5,50,0,0,0,0,0,Book</Font>
```

## Parameter Breakdown

### 1. Font Family Name
**Position:** 1st parameter  
**Example:** `DejaVu Sans Condensed`  
**Type:** String  

The name of the font family to be used. This can be:
- A specific font name (e.g., "Arial", "Times New Roman", "DejaVu Sans")
- A generic font family (e.g., "serif", "sans-serif", "monospace")
- The keyword "Default" to use the system default font

**Notes:**
- Font names with spaces should be kept as-is (no quotes needed in the XML)
- If the specified font is not available, Qt will fall back to a similar font

### 2. Font Size
**Position:** 2nd parameter  
**Example:** `8`  
**Type:** Integer  
**Range:** Typically 6-72, but can be larger  

The size of the font in points (pt). One point equals 1/72 of an inch.

**Common sizes:**
- 8pt: Small text, good for compact interfaces
- 10pt: Standard readable text
- 12pt: Larger text, good for headers
- 14pt+: Large text for emphasis or accessibility

### 3. Style Hint
**Position:** 3rd parameter  
**Example:** `-1`  
**Type:** Integer  

Provides hints to the font matching algorithm about the intended use of the font.

**Values:**
- `-1`: AnyStyle (no specific hint)
- `0`: Helvetica (sans-serif fonts)
- `1`: Times (serif fonts)
- `2`: Courier (monospace fonts)
- `3`: OldEnglish (decorative fonts)
- `4`: System (system default style)

**Usage:** Mostly used internally by Qt for font matching when the exact font isn't available.

### 4. Font Weight (Deprecated)
**Position:** 4th parameter  
**Example:** `5`  
**Type:** Integer  
**Status:** Deprecated in newer Qt versions  

This parameter is largely obsolete and typically set to `5`. The actual weight is controlled by parameter 5.

### 5. Font Weight (Active)
**Position:** 5th parameter  
**Example:** `50`  
**Type:** Integer  
**Range:** 0-99  

Controls the thickness/boldness of the font strokes.

**Standard Values:**
- `0`: Thin/Ultra Light
- `12`: Extra Light
- `25`: Light
- `50`: Normal/Regular (default)
- `57`: Medium
- `63`: Demi Bold
- `75`: Bold
- `81`: Extra Bold
- `87`: Black/Heavy
- `99`: Ultra Black

### 6. Italic Flag
**Position:** 6th parameter  
**Example:** `0`  
**Type:** Boolean (0 or 1)  

Controls whether the font is italicized.

**Values:**
- `0`: Normal/Roman (not italic)
- `1`: Italic

### 7. Underline Flag
**Position:** 7th parameter  
**Example:** `0`  
**Type:** Boolean (0 or 1)  

Controls whether the text is underlined.

**Values:**
- `0`: No underline
- `1`: Underlined

### 8. Strikethrough Flag
**Position:** 8th parameter  
**Example:** `0`  
**Type:** Boolean (0 or 1)  

Controls whether the text has a line through it.

**Values:**
- `0`: No strikethrough
- `1`: Strikethrough

### 9. Fixed Pitch Flag
**Position:** 9th parameter  
**Example:** `0`  
**Type:** Boolean (0 or 1)  

Controls whether the font uses fixed-width characters (monospace).

**Values:**
- `0`: Proportional spacing (variable character widths)
- `1`: Fixed pitch/Monospace (all characters same width)

**Note:** This setting may be overridden by the font family's inherent characteristics.

### 10. Raw Mode Flag
**Position:** 10th parameter  
**Example:** `0`  
**Type:** Boolean (0 or 1)  

Controls font hinting and anti-aliasing behavior.

**Values:**
- `0`: Normal rendering with hinting and anti-aliasing
- `1`: Raw mode - disables hinting and anti-aliasing

**Usage:** Rarely changed from 0 unless specific rendering control is needed.

### 11. Style Name (Optional)
**Position:** 11th parameter  
**Example:** `Book`  
**Type:** String  

Specifies a particular style variant of the font family.

**Common Style Names:**
- `Book`: Book weight (lighter than regular)
- `Regular`: Standard style
- `Medium`: Medium weight
- `Bold`: Bold style
- `Light`: Light weight
- `Condensed`: Narrow/compressed variant
- `Extended`: Wide variant
- `Oblique`: Slanted (similar to italic)

**Note:** This parameter is optional and may not be present in all font specifications.

## Practical Examples

### Basic Text Font
```xml
<Font>Arial,10,-1,5,50,0,0,0,0,0</Font>
```
- Arial font, 10pt, normal weight, no special formatting

### Bold Header Font
```xml
<Font>Arial,14,-1,5,75,0,0,0,0,0</Font>
```
- Arial font, 14pt, bold weight

### Monospace Code Font
```xml
<Font>Courier New,9,-1,5,50,0,0,0,1,0</Font>
```
- Courier New, 9pt, fixed-pitch for code display

### Italic Emphasis
```xml
<Font>Times New Roman,11,-1,5,50,1,0,0,0,0</Font>
```
- Times New Roman, 11pt, italic

## Editing Font Parameters

When modifying font parameters in QLC+ workspace files:

1. **Always backup** your workspace file before making changes
2. **Test changes** in a copy first
3. **Use consistent fonts** across your interface for better appearance
4. **Consider readability** when choosing sizes and weights
5. **Verify font availability** on target systems

## Font Size Recommendations for QLC+

- **Control labels:** 8-10pt
- **Button text:** 8-12pt
- **Header text:** 12-16pt
- **Status displays:** 10-14pt
- **Small indicators:** 6-8pt

## Troubleshooting

### Font Not Displaying Correctly
- Check if the font family is installed on the system
- Try using a generic family name like "sans-serif"
- Verify the font specification string is properly formatted

### Text Too Small/Large
- Adjust the size parameter (2nd position)
- Consider the display resolution and viewing distance

### Font Appears Bold When It Shouldn't
- Check the weight parameter (5th position)
- Ensure it's set to 50 for normal weight

## Related Documentation

- Qt Font Documentation: https://doc.qt.io/qt-5/qfont.html
- QLC+ Manual: Font and appearance settings
- CSS Font Properties (similar concepts): https://developer.mozilla.org/en-US/docs/Web/CSS/font

---

*Document Version: 1.0*  
*Last Updated: December 2024*  
*For QLC+ Version: 4.13.x and later* 