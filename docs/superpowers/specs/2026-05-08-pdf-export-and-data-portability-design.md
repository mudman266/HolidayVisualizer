# PDF Export & Data Portability — Design Spec
_Date: 2026-05-08_

## Overview

Add two capabilities to the Holiday Visualizer single-file app:

1. **PDF export** — a generated print page (stats summary + 12-month calendar) opened in a new tab
2. **Data portability** — JSON backup (download) and JSON restore (file upload) to protect against localStorage loss and enable cross-browser/cross-device use

Both features are surfaced via a single **Export ▾** dropdown button added to the header.

---

## Constraints

- Single-file vanilla app (`index.html`) — no build step, no external libraries, no CDN
- All new CSS must use CSS custom properties (no hardcoded hex in rule declarations), except inside the generated PDF document (which is cross-document and cannot inherit variables)
- Dark mode must be respected for all new UI elements

---

## 1. Header Dropdown

### Placement
A new `Export ▾` button is added to the header, immediately to the left of the existing gear (settings) button.

### Behaviour
- Clicking the button toggles a small absolutely-positioned dropdown panel below it
- The dropdown contains three items (in order):
  1. Export PDF
  2. Backup JSON
  3. Restore JSON
- Clicking any item closes the dropdown and executes the action
- Clicking outside the dropdown (or pressing Escape) closes it without action
- Only one dropdown can be open at a time (Escape handler already closes day modal and settings — extend it to close this too)

### Styling
- The button and its dropdown panel are wrapped in a `<div style="position: relative">` so the absolute dropdown anchors to the button
- Button style matches the existing Quick Add button: border, border-radius, height, font-size, font-weight, color
- Dropdown panel: `position: absolute`, `top: 100%`, `right: 0`, `margin-top: 4px`, `background: var(--surface)`, `border: 1px solid var(--border)`, `border-radius: var(--radius)`, `box-shadow`, `z-index: 200`, `min-width: 180px`
- Dropdown items: full-width, `padding: 9px 16px`, `font-size: 13px`, hover background `var(--bg)`, cursor pointer
- A thin divider separates "Export PDF" from the two JSON actions

---

## 2. JSON Backup

### Trigger
User clicks "Backup JSON" in the dropdown.

### Behaviour
1. Serialize `{ settings: S.settings, holidays: S.holidays }` to JSON (pretty-printed, 2-space indent)
2. Create a `data:application/json` URL from the serialized string
3. Programmatically click a hidden `<a download="holiday-backup.json">` element to trigger browser download
4. No confirmation dialog — download is instant and non-destructive

### What is exported
Only `settings` and `holidays` — not UI state (`view`, `viewMonth`, `bankPinned`, `qaEnabled`, etc.). UI state is device-specific and should not transfer.

---

## 3. JSON Restore

### Trigger
User clicks "Restore JSON" in the dropdown.

### Behaviour
1. Programmatically click a hidden `<input type="file" accept=".json">` element
2. On file selection, read the file via `FileReader.readAsText`
3. Parse the JSON; validate that the result has both `settings` (object) and `holidays` (object) keys — if invalid, show `alert("Invalid backup file.")` and abort
4. Show `confirm("This will replace your current holiday data. Continue?")` — if the user cancels, abort
5. Merge into state:
   - `S.settings = { ...DEFAULTS, ...parsed.settings }`
   - `S.holidays = parsed.holidays`
6. Call `persist()` then `render()`

### Notes
- The file input is reused across invocations; reset its `value` to `""` after each use so selecting the same file twice in a row fires the `change` event

---

## 4. PDF Print Page

### Trigger
User clicks "Export PDF" in the dropdown.

### Behaviour
1. Call `window.open('', '_blank')` to open a new browser tab
2. Write a complete self-contained HTML document into the new window's `document` via `doc.open()` / `doc.write()` / `doc.close()`
3. The page auto-prints on load via an inline `window.addEventListener('load', () => window.print())`
4. A visible "Print / Save as PDF" button is shown as fallback for popup-blocked scenarios

### Page structure

```
┌─────────────────────────────────────────────┐
│  Holiday Visualizer — 2026                  │  ← title
│  Generated: 8 May 2026                      │  ← subtitle
├─────────────────────────────────────────────┤
│  TOTAL        SPENT         REMAINING       │  ← stats row
│  160 hrs      48 hrs        112 hrs         │
│  (20 tokens)  (6 tokens)    (14 tokens)     │
├─────────────────────────────────────────────┤
│  [Print / Save as PDF]   ← button (hidden on print)
├─────────────────────────────────────────────┤
│  January      February      March           │
│  [mini grid]  [mini grid]   [mini grid]     │
│                                             │
│  April        May           June            │
│  ...                                        │
└─────────────────────────────────────────────┘
```

### Calendar rendering
- 12 mini-month grids in a 3-column layout (matching the year view)
- Each grid: day-of-week headers (S M T W T F S), day cells
- Holiday (full day): solid blue background (`#4f7ef0`), white text
- Holiday (half day): left-split gradient (blue left half, light blue right half), white text
- Weekend days: slightly muted text color
- Today: thin blue ring (`box-shadow: inset 0 0 0 1.5px #4f7ef0`)
- Holiday name shown as a small label below the day number if present

### Styles
- All styles are hardcoded hex values (no CSS variables — cross-document)
- Light theme only for print output (print PDFs are always light)
- `@page { margin: 1.5cm; size: A4 portrait; }`
- `@media print`: hide the print button, avoid page breaks inside `.mini-month` blocks (`break-inside: avoid`)
- 3-column grid for months; months flow naturally across pages

### Data scope
- Only holidays for `S.year` are rendered (the currently viewed year)

---

## 5. localStorage transparency

No code change needed — localStorage persistence already works. The Restore JSON flow effectively addresses the "I lost my data" concern by giving users an explicit backup they control.

---

## Out of scope

- Cloud sync or server-side storage
- Multi-year PDF export
- Cookie-based storage (localStorage is strictly better for this use case)
- Dark-mode PDF output
