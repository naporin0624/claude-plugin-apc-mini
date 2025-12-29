# APC Mini MK2 Claude Plugin

Claude Code plugin for Akai APC Mini MK2 MIDI controller development.

## Overview

This plugin provides comprehensive support for developing applications that control the APC Mini MK2's RGB LEDs and handle button/fader inputs using TypeScript and Node.js.

## Plugin Structure

```
.claude-plugin/
  plugin.json          # Plugin manifest

skills/
  midi-protocol-lookup/  # MIDI protocol reference
  led-mapping-guide/     # Button/LED mapping
  color-palette-ref/     # Color palette & RGB
  apc-code-samples/      # Code examples

agents/
  apc-developer.md      # Development specialist agent

reports/
  apc-mini-led.md       # Original research document
```

## Available Skills

| Skill | Purpose |
|-------|---------|
| `midi-protocol-lookup` | MIDI note numbers, channels, velocity values |
| `led-mapping-guide` | Visual button/LED layout, coordinate conversion |
| `color-palette-ref` | 128-color palette, custom RGB via SysEx |
| `apc-code-samples` | TypeScript implementation examples |

## Agent

- **apc-developer**: Specialized agent for APC Mini MK2 development tasks

## Quick Reference

### Libraries
- **easymidi**: Rapid prototyping, object-based API
- **@julusian/midi**: Production use, prebuilt binaries

### Key Note Ranges
- Pad Grid: 0-63
- Track Buttons: 100-107
- Scene Buttons: 112-119
- Shift: 122

### MIDI Channels
- 0-6: Solid brightness (10%-100%)
- 7-10: Pulse animation
- 11-15: Blink animation

### Primary Colors (Velocity)
- Red: 5
- Green: 21
- Blue: 45
- Yellow: 13
- Cyan: 33

## Development Commands

```bash
# Install dependencies
npm install easymidi
# or
npm install @julusian/midi

# List MIDI devices
npx ts-node -e "import e from 'easymidi'; console.log(e.getInputs(), e.getOutputs())"
```

## Contributing

See `reports/apc-mini-led.md` for the original research documentation.
