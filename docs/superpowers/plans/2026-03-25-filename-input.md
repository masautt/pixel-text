# Filename Input Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an editable filename input beside the download button that auto-populates from the text editor content.

**Architecture:** All changes are in `index.html` — the single-file app. A `slugify()` helper converts the editor text to a safe filename. The download row is restructured into a flex container holding the filename input and button side-by-side. The text-input listener updates the filename field unless the filename field currently has focus.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, no build step, no test framework — verification is manual browser testing.

---

## File Map

| File | Change |
|------|--------|
| `index.html` | CSS: add filename row + input styles; HTML: replace download button with two-element row; JS: add `slugify()`, wire listeners, update download handler |

---

### Task 1: Add CSS for the filename row and input

**Files:**
- Modify: `index.html` — `<style>` block

- [ ] **Step 1: Add styles for the filename row container and input**

In `index.html`, inside the `<style>` block, after the `.btn-secondary` rule (around line 181), add:

```css
  .download-row {
    display: flex;
    gap: 8px;
    align-items: flex-end;
  }

  .filename-field {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    min-width: 0;
  }

  .filename-field .section-label {
    margin-bottom: 0;
  }

  input[type=text].filename-input {
    width: 100%;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'IBM Plex Mono', monospace;
    font-size: 12px;
    padding: 8px 10px;
    outline: none;
    border-radius: 3px;
    transition: border-color 0.15s;
  }

  input[type=text].filename-input:focus {
    border-color: var(--accent);
  }
```

- [ ] **Step 2: Open `index.html` in a browser and confirm no visual regressions**

Open the file directly (`file://`) or via `npx serve .`. The app should look identical to before — no new elements are visible yet.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: add filename row and input styles"
```

---

### Task 2: Replace the download button with the filename row in HTML

**Files:**
- Modify: `index.html` — sidebar HTML (around line 296)

- [ ] **Step 1: Replace the existing download button markup**

Find and replace this block (around line 296):

```html
    <button class="btn btn-secondary" id="download-btn">↓ Download PNG</button>
```

With:

```html
    <div class="download-row">
      <div class="filename-field">
        <div class="section-label">Filename</div>
        <input type="text" class="filename-input" id="filename-input" value="pixel-text.png">
      </div>
      <button class="btn btn-secondary" id="download-btn" style="margin-top:0;white-space:nowrap">↓ PNG</button>
    </div>
```

- [ ] **Step 2: Verify layout in browser**

Open the file. The bottom of the sidebar should show:
- A small `FILENAME` label
- A text input containing `pixel-text.png`
- A `↓ PNG` button to the right, same height as the input, aligned to the bottom of the row

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "html: add filename input beside download button"
```

---

### Task 3: Add `slugify()` and wire the auto-populate listener

**Files:**
- Modify: `index.html` — `<script>` block

- [ ] **Step 1: Add the `slugify` helper function**

In the `<script>` block, after the `hexToRGB` function (around line 514), add:

```javascript
function slugify(text) {
  return text
    .toLowerCase()
    .replace(/\n/g, '-')
    .replace(/ /g, '-')
    .replace(/[^a-z0-9-]/g, '')
    .replace(/-+/g, '-')
    .replace(/^-|-$/g, '')
    || 'pixel-text';
}
```

- [ ] **Step 2: Add a reference to the filename input element**

In the UI variable declarations block (around line 520), after `const statusEl = ...`, add:

```javascript
const filenameInput = document.getElementById('filename-input');
```

- [ ] **Step 3: Wire the text-input listener to auto-populate the filename field**

Find the existing text-input event listener (inside the `forEach` block around line 572):

```javascript
[textInput, onColorInput, offAInput, offBInput].forEach(el => {
  el.addEventListener('input', update);
});
```

Replace with:

```javascript
[onColorInput, offAInput, offBInput].forEach(el => {
  el.addEventListener('input', update);
});

textInput.addEventListener('input', () => {
  if (document.activeElement !== filenameInput) {
    filenameInput.value = slugify(textInput.value) + '.png';
  }
  update();
});
```

- [ ] **Step 4: Verify auto-populate in browser**

1. Type `Hello World` in the text editor → filename field should update to `hello-world.png`
2. Type `GAME\nOVER` (use Shift+Enter for newline) → should show `game-over.png`
3. Type `!!!` → should show `pixel-text.png`
4. Clear the text editor → should show `pixel-text.png`
5. Click into the filename field and type a custom name, then edit the text editor → filename field should NOT change while it has focus; after clicking away and editing the text, it SHOULD update

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add slugify helper and auto-populate filename from text input"
```

---

### Task 4: Update the download handler to use the filename input

**Files:**
- Modify: `index.html` — download button event listener (around line 576)

- [ ] **Step 1: Update the download handler**

Find the existing download handler:

```javascript
document.getElementById('download-btn').addEventListener('click', () => {
  const link = document.createElement('a');
  link.download = 'pixel-text.png';
  link.href = canvasEl.toDataURL('image/png');
  link.click();
});
```

Replace with:

```javascript
document.getElementById('download-btn').addEventListener('click', () => {
  const raw = filenameInput.value.trim() || 'pixel-text.png';
  const filename = raw.endsWith('.png') ? raw : raw + '.png';
  const link = document.createElement('a');
  link.download = filename;
  link.href = canvasEl.toDataURL('image/png');
  link.click();
});
```

- [ ] **Step 2: Verify download in browser**

1. With default text `Pixels?` → filename field should show `pixels.png`; click `↓ PNG` and confirm the downloaded file is named `pixels.png`
2. Edit the filename field to `my-logo.png` and download → file should be named `my-logo.png`
3. Edit the filename field to `my-logo` (no extension) and download → file should be named `my-logo.png`
4. Clear the filename field and download → file should be named `pixel-text.png`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: use filename input value for PNG download"
```

---

### Task 5: Initial auto-populate on page load

**Files:**
- Modify: `index.html` — `update()` call at end of script (around line 583)

- [ ] **Step 1: Seed the filename field with the default text on load**

The page loads with `Pixels?` in the text editor but `pixel-text.png` hardcoded in the input's `value` attribute. Make them consistent by updating the `update()` call at the bottom of the script:

Find:

```javascript
update();
```

Replace with:

```javascript
filenameInput.value = slugify(textInput.value) + '.png';
update();
```

- [ ] **Step 2: Verify in browser**

Reload the page. The filename field should immediately show `pixels.png` (matching the default editor content), not `pixel-text.png`.

- [ ] **Step 3: Final end-to-end check**

1. Reload — filename shows `pixels.png`
2. Change text to `Marek Sautter` — filename updates to `marek-sautter.png`
3. Change text to `Score: 100!` — filename updates to `score-100.png`
4. Click into filename, type `custom-name` — text editor change does NOT overwrite it while focused
5. Click away from filename, change text — filename updates again
6. Download — correct file saved

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: seed filename input from default text on page load"
```
