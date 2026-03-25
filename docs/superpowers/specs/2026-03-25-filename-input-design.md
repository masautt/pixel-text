# Filename Input for PNG Download

**Date:** 2026-03-25
**Repo:** pixel-text (`index.html`)

## Summary

Add a filename text input to the sidebar so the user can name the downloaded PNG before clicking download. The field auto-populates from the text editor content and is editable.

## Layout

The bottom of the sidebar replaces the current bare download button with a two-element row:

```
[ Filename label     ]
[ filename input     ] [ ↓ PNG ]
```

- A small `FILENAME` section label sits above the input (matches existing label style: 9px, uppercase, muted colour, 2px letter-spacing)
- The text input (`<input type="text">`) takes all remaining width (`flex: 1`)
- The download button (`↓ PNG`) sits to the right, baseline-aligned with the input, same height
- Button text shortened to `↓ PNG` to fit the compact layout; styling unchanged (accent background, dark text, monospace, uppercase)

## Slugify Rules

Applied whenever the text editor content changes to produce the auto-populated filename:

1. Lowercase the entire string
2. Replace newlines with dashes (joins multi-line text)
3. Replace spaces with dashes
4. Strip any character that is not a letter (`a-z`), digit (`0-9`), or dash (`-`)
5. Collapse consecutive dashes into one
6. Trim leading/trailing dashes
7. If the result is empty, fall back to `pixel-text`
8. Append `.png`

**Examples:**

| Text input | Filename |
|---|---|
| `Pixels?` | `pixels.png` |
| `GAME OVER` | `game-over.png` |
| `GAME\nOVER` | `game-over.png` |
| `hello, world!` | `hello-world.png` |
| `!!!` | `pixel-text.png` |
| *(empty)* | `pixel-text.png` |

## Auto-populate Behaviour

- Fires on every `input` event on the text editor (`#text-input`)
- Does **not** fire while the user is actively editing the filename field (i.e. when the filename input has focus)
- If the user clears the filename field and unfocuses, it does **not** re-populate automatically — the field stays empty until the next text-editor change
- The user may type any value; the download uses whatever is in the field at click time

## Download Behaviour

- On click: read the filename field value as-is, append `.png` only if the value doesn't already end in `.png`
- Create anchor, set `download` attribute to the filename, trigger click

## Files Changed

- `index.html` — only file in the project; all changes are here:
  - HTML: replace `<button class="btn btn-secondary" id="download-btn">` with the new two-element row
  - CSS: add styles for the filename input (matches existing `textarea` / `input` aesthetic) and the row flex container
  - JS: add `slugify()` helper, wire text-input listener to update filename field, update download handler to read filename from input

## Out of Scope

- No validation of the filename beyond the slugify transform
- No format options (PNG only)
- No memory of the last used filename across sessions
