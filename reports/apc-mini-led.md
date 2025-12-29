ontrolling Akai APC mini MK2 LEDs with Node.js

The APC mini MK2's RGB LEDs are controlled via standard MIDI Note On messages where **velocity selects from 128 preset colors** and **MIDI channel controls brightness/animation behavior**. For custom RGB values beyond the palette, SysEx messages provide full 8-bit color control per channel. The `easymidi` library offers the cleanest API for this use case, though `@julusian/midi` provides better TypeScript support and prebuilt binaries.

The MK2 represents a significant protocol change from the original—brightness is now controlled via MIDI channels (0-6), animation via channels 7-15, and button note mappings have shifted entirely. Existing MK1 code will not work without modification.

## Node.js MIDI library comparison favors easymidi for simplicity

**easymidi** (npm: `easymidi`) provides the most ergonomic API for LED control with built-in TypeScript definitions. It wraps `@julusian/midi` and offers intuitive message syntax:

```typescript
import easymidi from 'easymidi';

const output = new easymidi.Output('APC mini mk2');

// Set pad 0 to red at full brightness (channel 6 = 100%)
output.send('noteon', { note: 0, velocity: 5, channel: 6 });

// SysEx for custom RGB
output.send('sysex', [0xF0, 0x47, 0x7F, 0x4F, 0x24, 0x00, 0x08, 
                       0x00, 0x00, 0x01, 0x7F, 0x00, 0x00, 0x00, 0x40, 0xF7]);
```

**@julusian/midi** is the better choice for production TypeScript projects—it includes prebuilt binaries for all platforms (no native compilation required), maintains active development, and provides comprehensive type definitions. The trade-off is a lower-level API using raw byte arrays:

```typescript
import midi from '@julusian/midi';

const output = new midi.Output();
output.openPort(0);
output.sendMessage([0x96, 0x00, 0x05]); // Note On, Ch7, Note 0, Red
```

**webmidi** (v3.x) offers the best TypeScript experience with full type definitions and works in both browser and Node.js contexts (uses `jzz` internally for Node). Its high-level API (`playNote`, `sendSysex`) suits rapid prototyping but adds abstraction overhead.

| Library | TypeScript | API Style | SysEx Support | Best For |
|---------|------------|-----------|---------------|----------|
| easymidi | Built-in | Object-based | ✅ | Rapid development |
| @julusian/midi | Built-in | Byte arrays | ✅ | Production, Electron |
| webmidi | Full | High-level | ✅ | Browser/Node hybrid |
| jzz | Partial | Chaining | ✅ | MIDI 2.0, advanced features |

## The MK2 MIDI protocol uses channels for brightness and velocity for color

Official documentation exists at `cdn.inmusicbrands.com/akai/attachments/APC mini mk2 - Communication Protocol - v1.0.pdf`. The protocol centers on three factors: **note number** (which LED), **MIDI channel** (brightness/behavior), and **velocity** (color from 128-color palette).

### Pad grid mapping (Notes 0-63)

The 8×8 RGB pad matrix maps bottom-left to top-right:

```
Row 8: 56 57 58 59 60 61 62 63
Row 7: 48 49 50 51 52 53 54 55
...
Row 1:  0  1  2  3  4  5  6  7
```

### MIDI channels control LED behavior

This is the critical insight—**channel selection determines brightness and animation**:

| Channel | Status Byte | Effect |
|---------|-------------|--------|
| 0-6 | 0x90-0x96 | Solid at 10%-100% brightness |
| 7-10 | 0x97-0x9A | Pulsing at 1/16 to 1/2 note rates |
| 11-15 | 0x9B-0x9F | Blinking at 1/24 to 1/2 note rates |

**Channel 6 (0x96) provides full brightness** and is the standard choice for most applications.

### Color palette velocity values

The velocity byte selects from 128 indexed colors. Key values:

| Color | Velocity | Hex |
|-------|----------|-----|
| Off | 0 | 0x00 |
| White | 3 | 0x03 |
| Red | 5 | 0x05 |
| Orange | 9 | 0x09 |
| Yellow | 13 | 0x0D |
| Green | 21 | 0x15 |
| Cyan | 33 | 0x21 |
| Blue | 45 | 0x2D |
| Purple | 49 | 0x31 |
| Magenta | 53 | 0x35 |
| Pink | 95 | 0x5F |

### Peripheral button mappings

Single-color LEDs on Track and Scene buttons use Channel 0 only:

- **Track buttons 1-8**: Notes 100-107 (0x64-0x6B), red LEDs
- **Scene Launch 1-8**: Notes 112-119 (0x70-0x77), green LEDs
- **Shift**: Note 122 (no LED)

Velocity controls state: **0 = off, 1 = on, 2 = blink**.

## SysEx enables custom RGB values beyond the palette

For precise color control, SysEx messages bypass the 128-color limitation. The format:

```
F0 47 7F 4F 24 [len-MSB] [len-LSB] [start] [end] [R-MSB] [R-LSB] [G-MSB] [G-LSB] [B-MSB] [B-LSB] F7
```

The header bytes: **0x47** (Akai manufacturer ID), **0x7F** (device broadcast), **0x4F** (APC mini MK2 product ID), **0x24** (RGB LED command).

RGB values use 8-bit encoding split across MSB/LSB pairs. To encode a value V (0-255): `MSB = V >> 7`, `LSB = V & 0x7F`.

```typescript
function setCustomRGB(output: easymidi.Output, pad: number, r: number, g: number, b: number) {
  const encodeColor = (v: number) => [(v >> 7) & 0x01, v & 0x7F];
  const [rMSB, rLSB] = encodeColor(r);
  const [gMSB, gLSB] = encodeColor(g);
  const [bMSB, bLSB] = encodeColor(b);
  
  output.send('sysex', [
    0xF0, 0x47, 0x7F, 0x4F, 0x24,  // Header
    0x00, 0x08,                     // Length (8 bytes)
    pad, pad,                       // Start/end pad (single pad)
    rMSB, rLSB, gMSB, gLSB, bMSB, bLSB,
    0xF7
  ]);
}

// Set pad 0 to purple (128, 0, 255)
setCustomRGB(output, 0, 128, 0, 255);
```

## Existing implementations provide reference patterns

The **akai-apc-mini-mk2** npm package (`npm install akai-apc-mini-mk2`) offers a browser-based Web MIDI implementation with a clean coordinate-based API:

```javascript
const mk2 = new APCMiniMk2();
await mk2.connect({ sysex: true });
mk2.pad33.color = "#123456";  // Hex color support
mk2.pad11.pulse();            // Animation control
```

For Node.js-specific implementations, **ArtGateOne's lighting control projects** (ma2apcmini, ma3apcmini) demonstrate integration patterns with professional lighting software using the `midi` package directly.

A complete TypeScript interface for the device:

```typescript
interface APCMiniMK2 {
  pads: Map<number, Pad>;       // Notes 0-63
  trackButtons: Map<number, Button>;  // Notes 100-107
  sceneButtons: Map<number, Button>;  // Notes 112-119
}

interface Pad {
  note: number;
  setColor(velocity: number, brightness?: number): void;
  setRGB(r: number, g: number, b: number): void;
  pulse(rate: '1/16' | '1/8' | '1/4' | '1/2'): void;
  blink(rate: '1/24' | '1/16' | '1/8' | '1/4' | '1/2'): void;
  off(): void;
}

interface Button {
  note: number;
  on(): void;
  off(): void;
  blink(): void;
}
```

## MK2 protocol breaks compatibility with original APC mini

The original APC mini used a fundamentally different approach: **velocity values 0-6 directly mapped to 3 colors** (off, green, green-blink, red, red-blink, amber, amber-blink), with all messages on MIDI channel 1. The MK2 requires complete code rewrites.

**Critical breaking changes:**

- **Channel usage**: MK1 used channel 1 for everything; MK2 uses channels 0-15 for brightness/animation
- **Bottom row notes**: 64-71 → **100-107**
- **Side column notes**: 82-89 → **112-119**  
- **Shift button**: 98 → **122**
- **Color encoding**: 7 velocity values → **128-color palette**

The 8×8 pad matrix (notes 0-63) and fader CC numbers (48-56) remain identical.

## Conclusion

Building an APC mini MK2 LED controller in TypeScript requires understanding the channel-based brightness system and the dual-path color control (velocity palette vs. SysEx RGB). Start with `@julusian/midi` for production code or `easymidi` for rapid prototyping. The official protocol PDF provides the complete velocity-to-color mapping table for the 128 preset colors. For visual applications requiring precise color matching, SysEx RGB control is essential despite its added complexity.
