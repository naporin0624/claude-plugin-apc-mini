# Color Palette Reference (Detailed)

Complete color palette and RGB control documentation for APC Mini MK2.

## Full Color Palette

### Reds (Velocity 1-8)

| Velocity | Hex | Description |
|----------|-----|-------------|
| 1 | 0x01 | Dark Red |
| 2 | 0x02 | Red-Orange Dark |
| 3 | 0x03 | White/Warm |
| 4 | 0x04 | Light Red |
| 5 | 0x05 | **Red** |
| 6 | 0x06 | Red-Orange |
| 7 | 0x07 | Orange-Red |
| 8 | 0x08 | Dark Orange |

### Oranges/Yellows (Velocity 9-16)

| Velocity | Hex | Description |
|----------|-----|-------------|
| 9 | 0x09 | **Orange** |
| 10 | 0x0A | Light Orange |
| 11 | 0x0B | Amber |
| 12 | 0x0C | Yellow-Orange |
| 13 | 0x0D | **Yellow** |
| 14 | 0x0E | Light Yellow |
| 15 | 0x0F | Pale Yellow |
| 16 | 0x10 | Yellow-Green |

### Greens (Velocity 17-28)

| Velocity | Hex | Description |
|----------|-----|-------------|
| 17 | 0x11 | Lime |
| 18 | 0x12 | Yellow-Green |
| 19 | 0x13 | Light Green |
| 20 | 0x14 | Pale Green |
| 21 | 0x15 | **Green** |
| 22 | 0x16 | Green Dark |
| 23 | 0x17 | Forest Green |
| 24 | 0x18 | Teal-Green |
| 25 | 0x19 | Sea Green |
| 26 | 0x1A | Aqua-Green |
| 27 | 0x1B | Turquoise |
| 28 | 0x1C | Cyan-Green |

### Cyans/Blues (Velocity 29-48)

| Velocity | Hex | Description |
|----------|-----|-------------|
| 29 | 0x1D | Mint |
| 30 | 0x1E | Aqua |
| 31 | 0x1F | Light Cyan |
| 32 | 0x20 | Pale Cyan |
| 33 | 0x21 | **Cyan** |
| 37 | 0x25 | Sky Blue |
| 41 | 0x29 | Light Blue |
| 45 | 0x2D | **Blue** |

### Purples/Magentas (Velocity 49-64)

| Velocity | Hex | Description |
|----------|-----|-------------|
| 49 | 0x31 | **Purple** |
| 50 | 0x32 | Violet |
| 51 | 0x33 | Lavender |
| 52 | 0x34 | Light Purple |
| 53 | 0x35 | **Magenta** |
| 54 | 0x36 | Pink-Magenta |
| 55 | 0x37 | Hot Pink |
| 56 | 0x38 | Deep Pink |
| 57 | 0x39 | Pink |

## Brightness Control via MIDI Channel

The same velocity color appears at different brightness levels:

| Channel | Brightness | Usage |
|---------|------------|-------|
| 0 | 10% | Very dim |
| 1 | 25% | Dim |
| 2 | 40% | Low |
| 3 | 55% | Medium-Low |
| 4 | 70% | Medium |
| 5 | 85% | High |
| 6 | 100% | **Full** (recommended) |

Example: Red at different brightness levels
```typescript
import type { Output } from 'easymidi';

const setRedAtBrightness = (output: Output, note: number, channel: number): void =>
  output.send('noteon', { note, velocity: 5, channel });

// Usage
setRedAtBrightness(output, 0, 0); // 10% red
setRedAtBrightness(output, 0, 3); // 55% red
setRedAtBrightness(output, 0, 6); // 100% red
```

## Animation Effects via MIDI Channel

| Channel | Effect | Rate |
|---------|--------|------|
| 7 | Pulse | 1/16 note |
| 8 | Pulse | 1/8 note |
| 9 | Pulse | 1/4 note |
| 10 | Pulse | 1/2 note |
| 11 | Blink | 1/24 note |
| 12 | Blink | 1/16 note |
| 13 | Blink | 1/8 note |
| 14 | Blink | 1/4 note |
| 15 | Blink | 1/2 note |

Example: Pulsing red
```typescript
const setPulsingRed = (output: Output, note: number): void =>
  output.send('noteon', { note, velocity: 5, channel: 9 }); // Red pulse 1/4
```

## Custom RGB via SysEx

### RGB Encoding

```typescript
type RGBPair = readonly [number, number];

const encodeRGB = (value: number): RGBPair => [
  (value >> 7) & 0x01,
  value & 0x7F
] as const;
```

### Complete Custom RGB Function

```typescript
import type { Output } from 'easymidi';

const setCustomRGB = (
  output: Output,
  pad: number,
  r: number,
  g: number,
  b: number
): void => {
  const [rMSB, rLSB] = encodeRGB(r);
  const [gMSB, gLSB] = encodeRGB(g);
  const [bMSB, bLSB] = encodeRGB(b);

  output.send('sysex', [
    0xF0,                   // SysEx start
    0x47,                   // Akai manufacturer ID
    0x7F,                   // Device broadcast
    0x4F,                   // APC Mini MK2 product ID
    0x24,                   // RGB LED command
    0x00, 0x08,             // Length (8 bytes)
    pad, pad,               // Start/end pad (single pad)
    rMSB, rLSB,             // Red
    gMSB, gLSB,             // Green
    bMSB, bLSB,             // Blue
    0xF7                    // SysEx end
  ]);
};
```

### Range RGB (Multiple Pads)

```typescript
const setRangeRGB = (
  output: Output,
  startPad: number,
  endPad: number,
  r: number,
  g: number,
  b: number
): void => {
  const [rMSB, rLSB] = encodeRGB(r);
  const [gMSB, gLSB] = encodeRGB(g);
  const [bMSB, bLSB] = encodeRGB(b);

  output.send('sysex', [
    0xF0, 0x47, 0x7F, 0x4F, 0x24,
    0x00, 0x08,
    startPad, endPad,
    rMSB, rLSB,
    gMSB, gLSB,
    bMSB, bLSB,
    0xF7
  ]);
};
```

## Common Color Examples

```typescript
// Primary colors
setCustomRGB(output, 0, 255, 0, 0);     // Red
setCustomRGB(output, 1, 0, 255, 0);     // Green
setCustomRGB(output, 2, 0, 0, 255);     // Blue

// Secondary colors
setCustomRGB(output, 3, 255, 255, 0);   // Yellow
setCustomRGB(output, 4, 0, 255, 255);   // Cyan
setCustomRGB(output, 5, 255, 0, 255);   // Magenta

// Pastels
setCustomRGB(output, 6, 255, 182, 193); // Light Pink
setCustomRGB(output, 7, 173, 216, 230); // Light Blue

// Brand colors
setCustomRGB(output, 8, 29, 185, 84);   // Spotify Green
setCustomRGB(output, 9, 255, 69, 0);    // SoundCloud Orange
```
