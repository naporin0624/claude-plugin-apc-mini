---
name: midi-protocol-lookup
description: Look up APC Mini MK2 MIDI protocol details including note numbers, MIDI channels, velocity values, and SysEx format. Use when user asks about "MIDI note", "channel", "velocity", "status byte", "SysEx", "protocol", or needs to understand how MIDI messages control the APC Mini MK2.
---

# APC Mini MK2 MIDI Protocol Reference

Quick reference for MIDI protocol details when developing APC Mini MK2 applications.

## MIDI Message Structure

APC Mini MK2 uses standard MIDI messages:
- **Note On**: `[status, note, velocity]` where status = `0x9n` (n = channel)
- **Note Off**: `[status, note, velocity]` where status = `0x8n`
- **Control Change**: `[status, cc, value]` where status = `0xBn`

## Note Number Quick Reference

### Pad Grid (8x8 RGB Pads)
Notes 0-63, bottom-left to top-right:

| Row | Notes | Hex Range |
|-----|-------|-----------|
| 8 (top) | 56-63 | 0x38-0x3F |
| 7 | 48-55 | 0x30-0x37 |
| 6 | 40-47 | 0x28-0x2F |
| 5 | 32-39 | 0x20-0x27 |
| 4 | 24-31 | 0x18-0x1F |
| 3 | 16-23 | 0x10-0x17 |
| 2 | 8-15 | 0x08-0x0F |
| 1 (bottom) | 0-7 | 0x00-0x07 |

**Formula**: `note = row * 8 + column` (0-indexed)

### Peripheral Buttons

| Button | Notes | Hex | LED Color |
|--------|-------|-----|-----------|
| Track 1-8 | 100-107 | 0x64-0x6B | Red |
| Scene Launch 1-8 | 112-119 | 0x70-0x77 | Green |
| Shift | 122 | 0x7A | None |

### Faders (Control Change)

| Fader | CC Number | Hex |
|-------|-----------|-----|
| Fader 1-8 | 48-55 | 0x30-0x37 |
| Master | 56 | 0x38 |

## MIDI Channel Effects

Channel selection determines LED brightness and animation:

| Channel | Hex | Effect |
|---------|-----|--------|
| 0 | 0x90 | 10% brightness |
| 1 | 0x91 | 25% brightness |
| 2 | 0x92 | 40% brightness |
| 3 | 0x93 | 55% brightness |
| 4 | 0x94 | 70% brightness |
| 5 | 0x95 | 85% brightness |
| 6 | 0x96 | **100% brightness** (recommended) |
| 7 | 0x97 | Pulse 1/16 |
| 8 | 0x98 | Pulse 1/8 |
| 9 | 0x99 | Pulse 1/4 |
| 10 | 0x9A | Pulse 1/2 |
| 11 | 0x9B | Blink 1/24 |
| 12 | 0x9C | Blink 1/16 |
| 13 | 0x9D | Blink 1/8 |
| 14 | 0x9E | Blink 1/4 |
| 15 | 0x9F | Blink 1/2 |

## Status Byte Construction

```
Status byte = 0x90 + channel
```

Examples:
- Full brightness: `0x96` (channel 6)
- Pulse 1/4: `0x99` (channel 9)
- Blink 1/8: `0x9D` (channel 13)

## SysEx Format for Custom RGB

For colors beyond the 128-color palette:

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

## Quick Examples

### Set pad to red at full brightness
```
[0x96, 0x00, 0x05]  // Channel 6, Note 0, Red
```

### Turn off pad
```
[0x90, 0x00, 0x00]  // Any channel, Note 0, Off
```

### Set Track button 1 on
```
[0x90, 0x64, 0x01]  // Channel 0, Note 100, On
```

### Set Scene button 1 blinking
```
[0x90, 0x70, 0x02]  // Channel 0, Note 112, Blink
```

## Official Documentation

Protocol PDF: `cdn.inmusicbrands.com/akai/attachments/APC mini mk2 - Communication Protocol - v1.0.pdf`
