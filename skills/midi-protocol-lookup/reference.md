# MIDI Protocol Reference (Detailed)

Detailed MIDI protocol documentation for APC Mini MK2.

## MIDI Message Structure

APC Mini MK2 uses standard MIDI messages:
- **Note On**: `[status, note, velocity]` where status = `0x9n` (n = channel)
- **Note Off**: `[status, note, velocity]` where status = `0x8n`
- **Control Change**: `[status, cc, value]` where status = `0xBn`

## Pad Grid Mapping (Notes 0-63)

The 8x8 RGB pad matrix maps bottom-left to top-right:

```
Row 8: 56 57 58 59 60 61 62 63
Row 7: 48 49 50 51 52 53 54 55
Row 6: 40 41 42 43 44 45 46 47
Row 5: 32 33 34 35 36 37 38 39
Row 4: 24 25 26 27 28 29 30 31
Row 3: 16 17 18 19 20 21 22 23
Row 2:  8  9 10 11 12 13 14 15
Row 1:  0  1  2  3  4  5  6  7
```

**Formula**: `note = row * 8 + column` (0-indexed)

## MIDI Channels Control LED Behavior

Channel selection determines brightness and animation:

| Channel | Status Byte | Effect |
|---------|-------------|--------|
| 0 | 0x90 | 10% brightness |
| 1 | 0x91 | 25% brightness |
| 2 | 0x92 | 40% brightness |
| 3 | 0x93 | 55% brightness |
| 4 | 0x94 | 70% brightness |
| 5 | 0x95 | 85% brightness |
| 6 | 0x96 | **100% brightness** (recommended) |
| 7 | 0x97 | Pulse 1/16 note |
| 8 | 0x98 | Pulse 1/8 note |
| 9 | 0x99 | Pulse 1/4 note |
| 10 | 0x9A | Pulse 1/2 note |
| 11 | 0x9B | Blink 1/24 note |
| 12 | 0x9C | Blink 1/16 note |
| 13 | 0x9D | Blink 1/8 note |
| 14 | 0x9E | Blink 1/4 note |
| 15 | 0x9F | Blink 1/2 note |

## Peripheral Button Mappings

Single-color LEDs on Track and Scene buttons use Channel 0 only:

- **Track buttons 1-8**: Notes 100-107 (0x64-0x6B), red LEDs
- **Scene Launch 1-8**: Notes 112-119 (0x70-0x77), green LEDs
- **Shift**: Note 122 (no LED)

Velocity controls state: **0 = off, 1 = on, 2 = blink**.

## Fader CC Numbers

| Fader | CC Number | Hex |
|-------|-----------|-----|
| Fader 1-8 | 48-55 | 0x30-0x37 |
| Master | 56 | 0x38 |

## SysEx Custom RGB Format

For precise color control beyond the 128-color palette:

```
F0 47 7F 4F 24 [len-MSB] [len-LSB] [start] [end] [R-MSB] [R-LSB] [G-MSB] [G-LSB] [B-MSB] [B-LSB] F7
```

Header breakdown:
- `0xF0`: SysEx start
- `0x47`: Akai manufacturer ID
- `0x7F`: Device broadcast
- `0x4F`: APC Mini MK2 product ID
- `0x24`: RGB LED command
- `0xF7`: SysEx end

RGB encoding (8-bit value to MSB/LSB):
```typescript
const msb = (value >> 7) & 0x01;
const lsb = value & 0x7F;
```

## MK2 vs MK1 Breaking Changes

The MK2 requires complete code rewrites from MK1:

- **Channel usage**: MK1 used channel 1 for everything; MK2 uses channels 0-15 for brightness/animation
- **Bottom row notes**: 64-71 → **100-107**
- **Side column notes**: 82-89 → **112-119**
- **Shift button**: 98 → **122**
- **Color encoding**: 7 velocity values → **128-color palette**

The 8x8 pad matrix (notes 0-63) and fader CC numbers (48-56) remain identical.
