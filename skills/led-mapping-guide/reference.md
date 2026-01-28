# LED Mapping Reference (Detailed)

Detailed physical layout and mapping documentation for APC Mini MK2.

## Complete Physical Layout

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  [SHIFT]                              [SCENE 1-8]   │
│                                         112-119     │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┐    ┌───┐       │
│  │56 │57 │58 │59 │60 │61 │62 │63 │    │112│ Row 8  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │48 │49 │50 │51 │52 │53 │54 │55 │    │113│ Row 7  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │40 │41 │42 │43 │44 │45 │46 │47 │    │114│ Row 6  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │32 │33 │34 │35 │36 │37 │38 │39 │    │115│ Row 5  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │24 │25 │26 │27 │28 │29 │30 │31 │    │116│ Row 4  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │16 │17 │18 │19 │20 │21 │22 │23 │    │117│ Row 3  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │ 8 │ 9 │10 │11 │12 │13 │14 │15 │    │118│ Row 2  │
│  ├───┼───┼───┼───┼───┼───┼───┼───┤    ├───┤       │
│  │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │    │119│ Row 1  │
│  └───┴───┴───┴───┴───┴───┴───┴───┘    └───┘       │
│   C1  C2  C3  C4  C5  C6  C7  C8                   │
│                                                     │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┐                │
│  │100│101│102│103│104│105│106│107│  [TRACK 1-8]   │
│  └───┴───┴───┴───┴───┴───┴───┴───┘                │
│                                                     │
│  ════════════════════════════════  [FADERS 1-9]    │
│   F1  F2  F3  F4  F5  F6  F7  F8  F9(Master)       │
│  CC48 CC49 CC50 CC51 CC52 CC53 CC54 CC55 CC56      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Pad Grid Note Table

| Row | Col 1 | Col 2 | Col 3 | Col 4 | Col 5 | Col 6 | Col 7 | Col 8 |
|-----|-------|-------|-------|-------|-------|-------|-------|-------|
| 8 | 56 | 57 | 58 | 59 | 60 | 61 | 62 | 63 |
| 7 | 48 | 49 | 50 | 51 | 52 | 53 | 54 | 55 |
| 6 | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 |
| 5 | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 |
| 4 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 |
| 3 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 |
| 2 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
| 1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |

## Coordinate Conversion Functions

```typescript
// Convert (row, column) to MIDI note (1-indexed coordinates)
const coordToNote = (row: number, col: number): number =>
  (row - 1) * 8 + (col - 1);

// Convert MIDI note to (row, column) (1-indexed)
const noteToCoord = (note: number): { readonly row: number; readonly col: number } => ({
  row: Math.floor(note / 8) + 1,
  col: (note % 8) + 1
});

// Convert (x, y) to MIDI note (0-indexed coordinates)
const xyToNote = (x: number, y: number): number => y * 8 + x;

// Convert MIDI note to (x, y) (0-indexed)
const noteToXY = (note: number): { readonly x: number; readonly y: number } => ({
  x: note % 8,
  y: Math.floor(note / 8)
});
```

## Track Buttons (Notes 100-107)

Bottom row, single-color **red** LEDs:

| Button | Track 1 | Track 2 | Track 3 | Track 4 | Track 5 | Track 6 | Track 7 | Track 8 |
|--------|---------|---------|---------|---------|---------|---------|---------|---------|
| Note | 100 | 101 | 102 | 103 | 104 | 105 | 106 | 107 |
| Hex | 0x64 | 0x65 | 0x66 | 0x67 | 0x68 | 0x69 | 0x6A | 0x6B |

Velocity: `0` = Off, `1` = On, `2` = Blink

## Scene Launch Buttons (Notes 112-119)

Right column, single-color **green** LEDs:

| Button | Scene 1 | Scene 2 | Scene 3 | Scene 4 | Scene 5 | Scene 6 | Scene 7 | Scene 8 |
|--------|---------|---------|---------|---------|---------|---------|---------|---------|
| Note | 112 | 113 | 114 | 115 | 116 | 117 | 118 | 119 |
| Hex | 0x70 | 0x71 | 0x72 | 0x73 | 0x74 | 0x75 | 0x76 | 0x77 |
| Row | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |

Velocity: `0` = Off, `1` = On, `2` = Blink

## Common Pattern Functions

```typescript
import type { Output } from 'easymidi';

// Light entire row
const lightRow = (output: Output, row: number, velocity: number): void => {
  const notes = Array.from({ length: 8 }, (_, col) => (row - 1) * 8 + col);
  notes.forEach((note) => output.send('noteon', { note, velocity, channel: 6 }));
};

// Light entire column
const lightColumn = (output: Output, col: number, velocity: number): void => {
  const notes = Array.from({ length: 8 }, (_, row) => row * 8 + (col - 1));
  notes.forEach((note) => output.send('noteon', { note, velocity, channel: 6 }));
};

// Light diagonal
const lightDiagonal = (output: Output, velocity: number): void => {
  const notes = Array.from({ length: 8 }, (_, i) => i * 8 + i);
  notes.forEach((note) => output.send('noteon', { note, velocity, channel: 6 }));
};

// Clear all pads
const clearAllPads = (output: Output): void => {
  const notes = Array.from({ length: 64 }, (_, i) => i);
  notes.forEach((note) => output.send('noteon', { note, velocity: 0, channel: 0 }));
};
```
