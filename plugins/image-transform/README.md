# image-transform

Image conversion and transformation skills for Claude Code.

## Skills

- **`svg-to-png`** — Convert a single SVG file to a PNG using `rsvg-convert` (librsvg). Default render width 1600px, suited to Medium and similar publishing platforms.

## Installation

This plugin is distributed via the [bgorkem-skills marketplace](https://github.com/bgorkem/bgorkem-skills). Inside Claude Code:

```
/plugin marketplace add bgorkem/bgorkem-skills
/plugin install image-transform@bgorkem-skills
```

After installation, run `/reload-plugins` to pick it up.

## Requirements

- `rsvg-convert` (from librsvg) installed on the host.
  - macOS: `brew install librsvg`
  - Ubuntu / Raspberry Pi OS: `sudo apt install librsvg2-bin`
