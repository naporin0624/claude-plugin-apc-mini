# Code Samples Reference (Detailed)

Complete implementation examples for APC Mini MK2.

## Library Comparison

| Library | TypeScript | API Style | SysEx Support | Best For |
|---------|------------|-----------|---------------|----------|
| easymidi | Built-in | Object-based | Yes | Rapid development |
| @julusian/midi | Built-in | Byte arrays | Yes | Production, Electron |
| webmidi | Full | High-level | Yes | Browser/Node hybrid |

## easymidi Setup

```typescript
import easymidi from 'easymidi';

// List available MIDI devices
const listDevices = (): void => {
  console.log('Inputs:', easymidi.getInputs());
  console.log('Outputs:', easymidi.getOutputs());
};

// Connect to APC Mini MK2
const createConnection = () => ({
  output: new easymidi.Output('APC mini mk2'),
  input: new easymidi.Input('APC mini mk2')
} as const);
```

## @julusian/midi Setup

```typescript
import midi from '@julusian/midi';

// List available devices
const listOutputPorts = (): readonly string[] => {
  const output = new midi.Output();
  const ports = Array.from(
    { length: output.getPortCount() },
    (_, i) => output.getPortName(i)
  );
  return ports;
};

// Open port by name
const openOutputByName = (name: string): midi.Output => {
  const out = new midi.Output();
  const portIndex = Array.from(
    { length: out.getPortCount() },
    (_, i) => i
  ).find((i) => out.getPortName(i).includes(name));

  if (portIndex === undefined) {
    throw new Error(`Device "${name}" not found`);
  }

  out.openPort(portIndex);
  return out;
};
```

## Functional APC Mini MK2 API

```typescript
import easymidi from 'easymidi';
import type { Output, Input } from 'easymidi';

// Types
type ButtonState = 'off' | 'on' | 'blink';
type RGBPair = readonly [number, number];

interface PadOptions {
  readonly brightness?: number;
  readonly color: number;
}

interface APCConnection {
  readonly output: Output;
  readonly input: Input;
}

// RGB encoding
const encodeRGB = (value: number): RGBPair => [
  (value >> 7) & 0x01,
  value & 0x7F
] as const;

// Create connection
const createAPCConnection = (): APCConnection => ({
  output: new easymidi.Output('APC mini mk2'),
  input: new easymidi.Input('APC mini mk2')
});

// Pad control
const setPad = (output: Output, pad: number, options: PadOptions): void =>
  output.send('noteon', {
    note: pad,
    velocity: options.color,
    channel: options.brightness ?? 6
  });

const setPadOff = (output: Output, pad: number): void =>
  output.send('noteon', { note: pad, velocity: 0, channel: 0 });

const setPadAt = (output: Output, row: number, col: number, options: PadOptions): void =>
  setPad(output, (row - 1) * 8 + (col - 1), options);

// Custom RGB
const setPadRGB = (output: Output, pad: number, r: number, g: number, b: number): void => {
  const [rMSB, rLSB] = encodeRGB(r);
  const [gMSB, gLSB] = encodeRGB(g);
  const [bMSB, bLSB] = encodeRGB(b);

  output.send('sysex', [
    0xF0, 0x47, 0x7F, 0x4F, 0x24,
    0x00, 0x08,
    pad, pad,
    rMSB, rLSB, gMSB, gLSB, bMSB, bLSB,
    0xF7
  ]);
};

// Button control
const velocityFromState = (state: ButtonState): number =>
  state === 'off' ? 0 : state === 'on' ? 1 : 2;

const setTrackButton = (output: Output, track: number, state: ButtonState): void =>
  output.send('noteon', {
    note: 99 + track,
    velocity: velocityFromState(state),
    channel: 0
  });

const setSceneButton = (output: Output, scene: number, state: ButtonState): void =>
  output.send('noteon', {
    note: 111 + scene,
    velocity: velocityFromState(state),
    channel: 0
  });

// Clear all
const clearAll = (output: Output): void => {
  Array.from({ length: 64 }, (_, i) => i).forEach((pad) => setPadOff(output, pad));
  Array.from({ length: 8 }, (_, i) => i + 1).forEach((track) => setTrackButton(output, track, 'off'));
  Array.from({ length: 8 }, (_, i) => i + 1).forEach((scene) => setSceneButton(output, scene, 'off'));
};

// Event handlers
const onPadPress = (input: Input, callback: (pad: number, velocity: number) => void): void =>
  input.on('noteon', (msg) => {
    if (msg.note < 64 && msg.velocity > 0) {
      callback(msg.note, msg.velocity);
    }
  });

const onPadRelease = (input: Input, callback: (pad: number) => void): void =>
  input.on('noteoff', (msg) => {
    if (msg.note < 64) callback(msg.note);
  });

const onTrackButton = (input: Input, callback: (track: number, pressed: boolean) => void): void =>
  input.on('noteon', (msg) => {
    if (msg.note >= 100 && msg.note <= 107) {
      callback(msg.note - 99, msg.velocity > 0);
    }
  });

const onSceneButton = (input: Input, callback: (scene: number, pressed: boolean) => void): void =>
  input.on('noteon', (msg) => {
    if (msg.note >= 112 && msg.note <= 119) {
      callback(msg.note - 111, msg.velocity > 0);
    }
  });

const onFader = (input: Input, callback: (fader: number, value: number) => void): void =>
  input.on('cc', (msg) => {
    if (msg.controller >= 48 && msg.controller <= 56) {
      callback(msg.controller - 47, msg.value);
    }
  });

const closeConnection = (connection: APCConnection): void => {
  connection.input.close();
  connection.output.close();
};
```

## Usage Examples

### Rainbow Pattern

```typescript
const RAINBOW_COLORS = [5, 9, 13, 21, 33, 45, 49, 53] as const;

const showRainbow = (output: Output): void => {
  const pads = Array.from({ length: 64 }, (_, i) => ({
    note: i,
    row: Math.floor(i / 8),
    col: i % 8
  }));

  pads.forEach(({ note, row, col }) => {
    const color = RAINBOW_COLORS[(row + col) % 8];
    setPad(output, note, { color });
  });
};
```

### Interactive Toggle (Immutable State)

```typescript
const createToggleHandler = (output: Output) => {
  const initialState: ReadonlyMap<number, boolean> = new Map();

  const toggle = (state: ReadonlyMap<number, boolean>, pad: number): ReadonlyMap<number, boolean> => {
    const isOn = state.get(pad) ?? false;
    const newState = new Map(state);

    if (isOn) {
      setPadOff(output, pad);
      newState.set(pad, false);
    } else {
      setPad(output, pad, { color: 5 });
      newState.set(pad, true);
    }

    return newState;
  };

  return { initialState, toggle };
};

// Usage
const { output, input } = createAPCConnection();
const { initialState, toggle } = createToggleHandler(output);
let state = initialState;

onPadPress(input, (pad) => {
  state = toggle(state, pad);
});
```

### Sequencer Animation

```typescript
const createSequencer = (output: Output, bpm: number) => {
  const stepTime = (60 / bpm) * 1000 / 4;

  const animate = (currentStep: number): number => {
    const prevStep = (currentStep - 1 + 8) % 8;

    // Clear previous column
    Array.from({ length: 8 }, (_, row) => row * 8 + prevStep)
      .forEach((note) => setPadOff(output, note));

    // Light current column
    Array.from({ length: 8 }, (_, row) => row * 8 + currentStep)
      .forEach((note) => setPad(output, note, { color: 5 }));

    return (currentStep + 1) % 8;
  };

  return { stepTime, animate };
};

// Usage
const connection = createAPCConnection();
const { stepTime, animate } = createSequencer(connection.output, 120);
let step = 0;

const intervalId = setInterval(() => {
  step = animate(step);
}, stepTime);

process.on('SIGINT', () => {
  clearInterval(intervalId);
  clearAll(connection.output);
  closeConnection(connection);
  process.exit();
});
```

### Gradient Fill

```typescript
interface RGB {
  readonly r: number;
  readonly g: number;
  readonly b: number;
}

const lerp = (start: number, end: number, t: number): number =>
  Math.round(start + (end - start) * t);

const lerpRGB = (start: RGB, end: RGB, t: number): RGB => ({
  r: lerp(start.r, end.r, t),
  g: lerp(start.g, end.g, t),
  b: lerp(start.b, end.b, t)
});

const fillGradient = (output: Output, start: RGB, end: RGB): void => {
  Array.from({ length: 64 }, (_, i) => ({
    pad: i,
    color: lerpRGB(start, end, i / 63)
  })).forEach(({ pad, color }) => {
    setPadRGB(output, pad, color.r, color.g, color.b);
  });
};

// Usage: Red to Blue gradient
fillGradient(output, { r: 255, g: 0, b: 0 }, { r: 0, g: 0, b: 255 });
```

## TypeScript Constants

```typescript
const Colors = {
  OFF: 0,
  WHITE: 3,
  RED: 5,
  ORANGE: 9,
  YELLOW: 13,
  LIME: 17,
  GREEN: 21,
  MINT: 29,
  CYAN: 33,
  SKY: 37,
  BLUE: 45,
  PURPLE: 49,
  MAGENTA: 53,
  PINK: 57,
} as const;

const Channels = {
  DIM_10: 0,
  DIM_25: 1,
  DIM_40: 2,
  DIM_55: 3,
  DIM_70: 4,
  DIM_85: 5,
  FULL: 6,
  PULSE_16: 7,
  PULSE_8: 8,
  PULSE_4: 9,
  PULSE_2: 10,
  BLINK_24: 11,
  BLINK_16: 12,
  BLINK_8: 13,
  BLINK_4: 14,
  BLINK_2: 15,
} as const;

const Notes = {
  PAD_MIN: 0,
  PAD_MAX: 63,
  TRACK_MIN: 100,
  TRACK_MAX: 107,
  SCENE_MIN: 112,
  SCENE_MAX: 119,
  SHIFT: 122,
} as const;

type ColorName = keyof typeof Colors;
type ChannelName = keyof typeof Channels;

export { Colors, Channels, Notes, ColorName, ChannelName };
```

## Error Handling

```typescript
import easymidi from 'easymidi';
import type { Input, Output } from 'easymidi';

interface ConnectionResult {
  readonly success: true;
  readonly input: Input;
  readonly output: Output;
}

interface ConnectionError {
  readonly success: false;
  readonly error: string;
  readonly availableInputs: readonly string[];
  readonly availableOutputs: readonly string[];
}

type ConnectionAttempt = ConnectionResult | ConnectionError;

const connectToAPC = (): ConnectionAttempt => {
  const inputs = easymidi.getInputs();
  const outputs = easymidi.getOutputs();

  const inputName = inputs.find((n) => n.includes('APC mini'));
  const outputName = outputs.find((n) => n.includes('APC mini'));

  if (!inputName || !outputName) {
    return {
      success: false,
      error: 'APC Mini MK2 not found',
      availableInputs: inputs,
      availableOutputs: outputs
    };
  }

  try {
    return {
      success: true,
      input: new easymidi.Input(inputName),
      output: new easymidi.Output(outputName)
    };
  } catch (e) {
    return {
      success: false,
      error: `Failed to connect: ${e instanceof Error ? e.message : String(e)}`,
      availableInputs: inputs,
      availableOutputs: outputs
    };
  }
};
```
