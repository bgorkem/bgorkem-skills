---
name: svg-to-png
description: Converts a single SVG file into a PNG using the `rsvg-convert` command-line tool. Use when the user asks to render, export, rasterize, or convert an SVG to a PNG image — typically for embedding diagrams in places that don't support inline SVG (Medium, slide decks, documents). Requires `rsvg-convert` (librsvg) installed on the host. Do not use for generating new SVG content, batch conversion of folders, or other image formats.
---

# SVG to PNG

Convert a single SVG file to a PNG using `rsvg-convert`. Produces a high-resolution raster suitable for platforms that don't render SVG inline (Medium, Notion, slide decks, social posts).

The default render width is **1600px**, chosen as a sensible 2× density for Medium's ~700px content column. Override when the destination calls for something different.

## When to use

Trigger on any of:

- "Convert `<file>.svg` to png"
- "Render this svg as a png"
- "Export the diagram as png for Medium"
- "Make a png version of `<file>.svg`"

If the user asks to convert *multiple* SVGs at once, or to convert a folder of SVGs, this skill is **not** the right fit — it deliberately handles one file per call. Either run it once per file, or tell the user that batch conversion isn't supported by this skill and offer to do them one at a time.

If the user asks to convert to JPEG, WebP, PDF, or any non-PNG format, this skill isn't the right fit — say so and suggest `rsvg-convert -f <format>` directly (it supports `pdf`, `ps`, `eps`, but **not** JPEG/WebP — for those they'd need ImageMagick).

## Requirements

- `rsvg-convert` must be installed and on the system PATH. Install methods:
  - macOS: `brew install librsvg`
  - Ubuntu/Debian/Raspberry Pi OS: `sudo apt install librsvg2-bin`
  - Verify with: `rsvg-convert --version`

If `rsvg-convert` is missing, stop and tell the user how to install it for their platform — don't fall back to ImageMagick or other tools, because their SVG renderers handle CSS classes and modern SVG features poorly.

## Inputs

The user provides — or this skill infers from context — these values:

| Input | Required | Default | Notes |
| --- | --- | --- | --- |
| `input_path` | Yes | — | Path to the source SVG file. Can be absolute or relative to cwd. |
| `output_path` | No | Same path as input with `.svg` replaced by `.png` | Where to write the PNG. |
| `width` | No | `1600` | Render width in pixels. Height is computed from the SVG's aspect ratio. |
| `background_color` | No | transparent | Any CSS color value: named (`black`, `white`), hex (`#1e1e1e`), or `rgba(r,g,b,a)`. Omit the flag entirely to keep the default transparent background. |

If `input_path` is ambiguous (e.g. user says "convert my diagram" without naming a file), ask one quick clarifying question. Don't guess.

## Workflow

### Step 1: Validate the input

- Confirm the input file exists and ends in `.svg` (case-insensitive). If not, stop and tell the user what's wrong.
- Confirm `rsvg-convert` is available by running `which rsvg-convert`. If it isn't, follow the Requirements section above.

### Step 2: Decide the output path

- If the user specified an output path, use it.
- Otherwise, take the input path and swap the `.svg` extension for `.png`. Write to the same directory as the input.
- If the destination already exists, **overwrite it silently** — this is the expected behaviour when regenerating diagrams during a blog edit cycle. If the user wants a backup, they can use git.

### Step 3: Run the conversion

Execute (substituting actual values):

```bash
# No background (transparent — default)
rsvg-convert -w <width> "<input_path>" -o "<output_path>"

# With a background color
rsvg-convert -w <width> --background-color "<background_color>" "<input_path>" -o "<output_path>"
```

Notes:
- Use `-w` (width) not `-h` (height) — width is what matters for fitting a content column.
- Use `-d` / `--dpi-x` / `--dpi-y` only if the user explicitly asks for a specific DPI. Default rendering is fine for screen use.
- Quote both paths in case they contain spaces.
- Only include `--background-color` when the user explicitly requests one. Omitting it leaves the background transparent, which is the correct default for SVGs that manage their own background.
- Any CSS color value is valid: `black`, `white`, `#1e1e1e`, `rgba(30,30,30,1)`, etc.

### Step 4: Confirm to the user

Reply with: the output path, the rendered dimensions (read from the file using `file <output_path>` or `identify <output_path>` if ImageMagick is available — otherwise just state the requested width), and the file size in KB.

Example: "Rendered `blog/assets/post-02-memory-flow.png` at 1600×1412 (84 KB)."

Do **not** open or display the image — just confirm the path.

## Examples

### Example 1: Default conversion

**User says:** "Convert `blog/assets/post-02-memory-flow.svg` to PNG."

**Actions:**
1. Verify the file exists and `rsvg-convert` is on PATH.
2. Run: `rsvg-convert -w 1600 "blog/assets/post-02-memory-flow.svg" -o "blog/assets/post-02-memory-flow.png"`
3. Confirm: "Rendered `blog/assets/post-02-memory-flow.png` at 1600×1412 (84 KB)."

### Example 2: Custom width

**User says:** "Render `diagram.svg` as a PNG at 800px wide for a tweet."

**Actions:**
1. Verify input and tool.
2. Run: `rsvg-convert -w 800 "diagram.svg" -o "diagram.png"`
3. Confirm: "Rendered `diagram.png` at 800×706 (29 KB)."

### Example 3: Custom output path

**User says:** "Convert `flow.svg` and save it as `~/Desktop/flow-final.png`."

**Actions:**
1. Verify input and tool.
2. Run: `rsvg-convert -w 1600 "flow.svg" -o "/Users/<user>/Desktop/flow-final.png"`
3. Confirm: "Rendered `~/Desktop/flow-final.png` at 1600×1412 (84 KB)."

### Example 4: Dark background

**User says:** "Convert `diagram.svg` to PNG with a dark background."

**Actions:**
1. Verify input and tool.
2. Run: `rsvg-convert -w 1600 --background-color "#1e1e1e" "diagram.svg" -o "diagram.png"`
3. Confirm: "Rendered `diagram.png` at 1600×900 (72 KB) with a dark background (`#1e1e1e`)."

### Example 5: Transparent background (explicit)

**User says:** "Convert `diagram.svg` to PNG, keep the background transparent."

**Actions:**
1. Verify input and tool.
2. Run: `rsvg-convert -w 1600 "diagram.svg" -o "diagram.png"` (omit `--background-color` entirely)
3. Confirm: "Rendered `diagram.png` at 1600×900 (68 KB) with a transparent background."

### Example 6: Tool not installed

**User says:** "Convert `diagram.svg` to PNG."

**Context:** `which rsvg-convert` returns nothing.

**Action:** Don't attempt the conversion. Reply: "`rsvg-convert` isn't installed. On macOS run `brew install librsvg`; on Linux `sudo apt install librsvg2-bin`. Once that's done, try again."

## Edge cases

- **SVG with external references** (e.g. `<image href="logo.png">` pointing to a local file). `rsvg-convert` will resolve these relative to the SVG's own directory. If the referenced file is missing, the conversion still succeeds but the PNG will show a blank where the external image should be. Worth mentioning to the user if you notice external references in the SVG source.
- **SVG without a `viewBox`.** `rsvg-convert` may render at unexpected dimensions. If the output looks wrong, suggest the user add `viewBox="0 0 W H"` to the source SVG.
- **Web fonts referenced by URL.** `rsvg-convert` does *not* fetch web fonts — it falls back to system fonts. If the SVG's typography matters and uses something like Google Fonts, the result won't match the browser preview. Either inline the font as a data URI in the SVG, or accept the fallback.
- **Very large SVGs (`>5MB`).** Conversion is still fast but the output PNG can be huge at 1600px. If the user is producing many such files, suggest a lower width (e.g. `-w 1200`) or running through `oxipng`/`pngquant` afterwards for compression.
- **Output path's parent directory doesn't exist.** `rsvg-convert` will fail with a "no such file or directory" error. Create the parent directory first with `mkdir -p`.

## Troubleshooting

**Error: `rsvg-convert: command not found`**
Cause: librsvg not installed, or its `bin` directory not on PATH.
Solution: Install per Requirements. On macOS with Homebrew, also verify `brew --prefix librsvg` matches what's on PATH.

**Output PNG is blank or text is missing**
Cause: The SVG references fonts or external resources the renderer can't find.
Solution: Check the SVG source for `<style>` blocks importing web fonts, or `<image href="...">` referencing files outside the SVG's directory. Inline fonts as data URIs, or use system-installed fonts.

**Output PNG is the wrong size**
Cause: The SVG has no `viewBox` attribute, or has `width`/`height` set in non-pixel units.
Solution: Open the SVG, ensure it has `viewBox="0 0 W H"` and remove explicit `width`/`height` attributes (let `-w` control them).

**Colours look wrong / dark mode classes don't apply**
Cause: The SVG relies on CSS variables or `prefers-color-scheme` media queries that only resolve in a real browser.
Solution: `rsvg-convert` renders SVGs as if in a generic context — there's no `prefers-color-scheme`, no `:root` CSS variables from outside the SVG. Bake the intended colour values directly into the SVG source for export.

## What NOT to do

- Don't batch-convert folders. This skill handles one file per call by design — keep the contract simple.
- Don't fall back to ImageMagick (`convert`, `magick`) if `rsvg-convert` is missing. ImageMagick's SVG renderer drops modern features and produces visibly worse output. Telling the user to install librsvg is the right answer.
- Don't open, preview, or display the resulting PNG. Just confirm the path and dimensions.
- Don't strip or modify the SVG source as part of the conversion. If the SVG needs editing (e.g. to inline fonts), that's a separate task — surface it to the user, don't do it silently.
