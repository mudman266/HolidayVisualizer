# PDF Export & Data Portability Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an Export dropdown to the header with PDF export (stats + 12-month calendar in a new tab), JSON backup download, and JSON restore from file.

**Architecture:** All changes are in `index.html`. The Export dropdown sits in the header as a `position: relative` wrapper around a toggle button and an absolutely-positioned panel. JSON backup/restore use a hidden `<a>` and `<input type="file">`. PDF export generates a complete self-contained HTML document written into a new blank tab via `document.write()`.

**Tech Stack:** Vanilla HTML/CSS/JS — no dependencies, no build step.

---

### Task 1: Export dropdown (HTML + CSS + toggle JS)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add CSS for the dropdown**

In the `<style>` block, after the `.qa-btn` rules (around line 633), add:

```css
/* === EXPORT DROPDOWN === */
.export-wrap { position: relative; }

.export-drop {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 4px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
  z-index: 200;
  min-width: 180px;
  overflow: hidden;
}
.export-drop.hidden { display: none; }

.export-item {
  display: block;
  width: 100%;
  padding: 9px 16px;
  font-size: 13px;
  font-weight: 500;
  text-align: left;
  background: none;
  border: none;
  color: var(--text);
  cursor: pointer;
  font-family: inherit;
  transition: background 0.1s;
}
.export-item:hover { background: var(--bg); }

.export-divider { height: 1px; background: var(--border); margin: 2px 0; }

@media (prefers-color-scheme: dark) {
  .export-drop { box-shadow: 0 8px 24px rgba(0,0,0,0.4); }
}
```

- [ ] **Step 2: Add dropdown HTML to the header**

In `<header>`, find the existing settings button:
```html
  <button class="settings-btn" id="settingsBtn" title="Settings">&#9881;</button>
```

Replace it with:
```html
  <div class="export-wrap">
    <button class="qa-btn" id="exportBtn" title="Export / Import data">
      Export &#9662;
    </button>
    <div class="export-drop hidden" id="exportDrop">
      <button class="export-item" id="exportPdfBtn">Export PDF</button>
      <div class="export-divider"></div>
      <button class="export-item" id="exportJsonBtn">Backup JSON</button>
      <button class="export-item" id="importJsonBtn">Restore JSON</button>
    </div>
  </div>
  <a id="exportAnchor" style="display:none"></a>
  <input type="file" accept=".json" id="importFileInput" style="display:none">
  <button class="settings-btn" id="settingsBtn" title="Settings">&#9881;</button>
```

- [ ] **Step 3: Add dropdown toggle JS**

In the `<script>`, after the `// SETTINGS MODAL` section (around line 1336), add a new section:

```javascript
// ─────────────────────────────────────────────
// EXPORT DROPDOWN
// ─────────────────────────────────────────────
function closeExportDrop() {
  document.getElementById('exportDrop').classList.add('hidden');
}

document.getElementById('exportBtn').addEventListener('click', e => {
  e.stopPropagation();
  document.getElementById('exportDrop').classList.toggle('hidden');
});

document.addEventListener('click', () => closeExportDrop());
```

- [ ] **Step 4: Extend the Escape handler to close the dropdown**

Find the existing keydown listener near the bottom of the script:
```javascript
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') { closeDayModal(); closeSettings(); }
});
```

Replace it with:
```javascript
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') { closeDayModal(); closeSettings(); closeExportDrop(); }
});
```

- [ ] **Step 5: Verify in browser**

Open `index.html` directly in a browser (`file://` or a local server).
- "Export ▾" button appears in the header to the left of the gear icon
- Clicking it opens the dropdown with three items: "Export PDF", "Backup JSON", "Restore JSON"
- Clicking outside the dropdown closes it
- Pressing Escape closes it
- Clicking an item closes it (items do nothing useful yet — that's fine)

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add export dropdown UI"
```

---

### Task 2: JSON backup

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add backup handler**

In the EXPORT DROPDOWN section of the script, after the `document.addEventListener('click', ...)` line, add:

```javascript
document.getElementById('exportJsonBtn').addEventListener('click', () => {
  closeExportDrop();
  const data = JSON.stringify({ settings: S.settings, holidays: S.holidays }, null, 2);
  const url  = 'data:application/json;charset=utf-8,' + encodeURIComponent(data);
  const a    = document.getElementById('exportAnchor');
  a.href     = url;
  a.download = 'holiday-backup.json';
  a.click();
});
```

- [ ] **Step 2: Verify in browser**

1. Open `index.html`. Add two or three holidays via the day modal.
2. Click "Export ▾" → "Backup JSON".
3. A file named `holiday-backup.json` downloads automatically.
4. Open the file in a text editor — it should look like:
```json
{
  "settings": { "totalHolidayHours": 160, "hoursPerToken": 8 },
  "holidays": {
    "2026-05-01": { "tokens": 1, "name": "May Day" }
  }
}
```
5. Confirm only `settings` and `holidays` are present (no `view`, `bankPinned`, etc.).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add JSON backup download"
```

---

### Task 3: JSON restore

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add restore handler**

In the EXPORT DROPDOWN section, after the backup handler, add:

```javascript
document.getElementById('importJsonBtn').addEventListener('click', () => {
  closeExportDrop();
  const input = document.getElementById('importFileInput');
  input.value = '';
  input.click();
});

document.getElementById('importFileInput').addEventListener('change', e => {
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = ev => {
    let parsed;
    try { parsed = JSON.parse(ev.target.result); } catch (_) {
      alert('Invalid backup file.'); return;
    }
    if (!parsed || typeof parsed.settings !== 'object' || typeof parsed.holidays !== 'object') {
      alert('Invalid backup file.'); return;
    }
    if (!confirm('This will replace your current holiday data. Continue?')) return;
    S.settings = { ...DEFAULTS, ...parsed.settings };
    S.holidays  = parsed.holidays;
    persist();
    render();
  };
  reader.readAsText(file);
});
```

- [ ] **Step 2: Verify in browser**

1. Add a few holidays, then do "Backup JSON" to save them.
2. Delete all holidays (Quick Add → drag to remove, or modal → None).
3. Click "Export ▾" → "Restore JSON", select the backup file.
4. The confirm dialog appears — click OK.
5. Holidays reappear and the calendar re-renders correctly.

Edge cases to check:
- Select the same backup file twice in a row — the `change` event still fires (because `input.value = ''` resets it).
- Cancel the confirm dialog — data stays as-is.
- Select a `.txt` file containing random text — "Invalid backup file." alert appears.
- Select a `.json` file containing `{}` (missing required keys) — "Invalid backup file." alert appears.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add JSON restore from file"
```

---

### Task 4: PDF export

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add buildPdfMonth helper**

In the EXPORT DROPDOWN section, after the restore handler, add:

```javascript
function buildPdfMonth(year, month) {
  const first = new Date(year, month, 1).getDay();
  const days  = new Date(year, month + 1, 0).getDate();
  const dows  = ['S','M','T','W','T','F','S'];
  let h = `<div class="month"><div class="month-name">${MONTHS[month]}</div><div class="month-grid">`;
  for (const d of dows) h += `<div class="dow">${d}</div>`;
  for (let i = 0; i < first; i++) h += `<div class="day empty"></div>`;
  for (let d = 1; d <= days; d++) {
    const ds  = `${year}-${pad(month+1)}-${pad(d)}`;
    const hol = S.holidays[ds];
    const dow = (first + d - 1) % 7;
    let cls   = 'day';
    if (dow === 0 || dow === 6) cls += ' weekend';
    if (ds === TODAY_STR)       cls += ' today';
    if (hol) cls += hol.tokens === 1 ? ' hol-full' : ' hol-half';
    h += `<div class="${cls}"><span class="day-num">${d}</span>`;
    if (hol && hol.name) h += `<span class="day-label">${escHtml(hol.name)}</span>`;
    h += `</div>`;
  }
  h += `</div></div>`;
  return h;
}
```

- [ ] **Step 2: Add exportPdf function**

Immediately after `buildPdfMonth`, add:

```javascript
function exportPdf() {
  const win = window.open('', '_blank');
  if (!win) { alert('Please allow popups for this page to export PDF.'); return; }

  const st      = stats();
  const year    = S.year;
  let   months  = '';
  for (let m = 0; m < 12; m++) months += buildPdfMonth(year, m);

  const genDate = new Date().toLocaleDateString('en-GB', { day: 'numeric', month: 'long', year: 'numeric' });

  const html = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Holiday Visualizer — ${year}</title>
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,sans-serif;color:#1e2a3a;font-size:14px;padding:32px}
.hdr{margin-bottom:20px}
.hdr-title{font-size:22px;font-weight:700;color:#4f7ef0}
.hdr-sub{font-size:12px;color:#64748b;margin-top:4px}
.stats{display:flex;gap:32px;background:#f1f3f8;border-radius:8px;padding:16px 24px;margin-bottom:16px;border:1px solid #e2e8f0}
.stat-label{font-size:10px;font-weight:700;text-transform:uppercase;letter-spacing:.6px;color:#64748b;margin-bottom:4px}
.stat-value{font-size:20px;font-weight:700;line-height:1}
.stat-sub{font-size:12px;color:#64748b;margin-top:2px}
.spent .stat-value{color:#f97316}
.remaining .stat-value{color:#10b981}
.print-btn{margin-bottom:20px;padding:9px 18px;background:#4f7ef0;color:#fff;border:none;border-radius:7px;font-size:13px;font-weight:600;cursor:pointer;font-family:inherit}
.months-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:16px}
.month{background:#fff;border:1px solid #e2e8f0;border-radius:8px;padding:12px;break-inside:avoid}
.month-name{font-weight:700;font-size:13px;color:#1e2a3a;margin-bottom:8px}
.month-grid{display:grid;grid-template-columns:repeat(7,1fr);gap:2px}
.dow{font-size:9px;text-align:center;color:#64748b;font-weight:700;padding-bottom:3px;text-transform:uppercase}
.day{aspect-ratio:1;display:flex;flex-direction:column;align-items:center;justify-content:center;font-size:9px;border-radius:3px;overflow:hidden}
.day-num{font-size:10px;line-height:1.2}
.day-label{font-size:6px;color:#64748b;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;max-width:95%;text-align:center}
.day.empty{pointer-events:none}
.day.weekend .day-num{color:#94a3b8}
.day.today{box-shadow:inset 0 0 0 1.5px #4f7ef0}
.day.today .day-num{font-weight:700}
.day.hol-full{background:#4f7ef0}
.day.hol-full .day-num{color:#fff;font-weight:700}
.day.hol-full .day-label{color:rgba(255,255,255,.8)}
.day.hol-half{background:linear-gradient(to right,#4f7ef0 50%,#c7d7fc 50%)}
.day.hol-half .day-num{color:#fff;font-weight:700}
@media print{
  .print-btn{display:none}
  body{padding:0}
}
@page{margin:1.5cm;size:A4 portrait}
</style>
</head>
<body>
<div class="hdr">
  <div class="hdr-title">Holiday Visualizer — ${year}</div>
  <div class="hdr-sub">Generated: ${genDate}</div>
</div>
<div class="stats">
  <div class="stat total">
    <div class="stat-label">Total</div>
    <div class="stat-value">${st.totalHrs} hrs</div>
    <div class="stat-sub">${st.totalTokens} tokens</div>
  </div>
  <div class="stat spent">
    <div class="stat-label">Spent</div>
    <div class="stat-value">${st.spentHrs} hrs</div>
    <div class="stat-sub">${st.spentTokens} tokens</div>
  </div>
  <div class="stat remaining">
    <div class="stat-label">Remaining</div>
    <div class="stat-value">${st.remHrs} hrs</div>
    <div class="stat-sub">${st.remTokens} tokens</div>
  </div>
</div>
<button class="print-btn" onclick="window.print()">Print / Save as PDF</button>
<div class="months-grid">${months}</div>
<script>window.addEventListener('load',function(){window.print();});<\/script>
</body>
</html>`;

  win.document.open();
  win.document.write(html);
  win.document.close();
}
```

- [ ] **Step 3: Wire the PDF button**

In the EXPORT DROPDOWN section, after the restore handler, add:

```javascript
document.getElementById('exportPdfBtn').addEventListener('click', () => {
  closeExportDrop();
  exportPdf();
});
```

- [ ] **Step 4: Verify in browser**

1. Add holidays across multiple months — include some full days, some half days, and some with names.
2. Click "Export ▾" → "Export PDF".
3. A new tab opens. Check:
   - Title reads "Holiday Visualizer — [year]" with today's generated date below
   - Stats row shows Total / Spent / Remaining with correct hrs and token counts
   - Spent value appears orange, Remaining appears green
   - 12 month grids are visible in 3 columns
   - Full-day holidays show solid blue with white day numbers
   - Half-day holidays show split blue/light-blue gradient
   - Today's date has a thin blue ring
   - Holiday names (where set) appear as tiny labels below the day number
   - "Print / Save as PDF" button is visible
4. The browser print dialog auto-appears on page load.
5. Cancel the dialog — click "Print / Save as PDF" — dialog appears again.
6. Block popups for the page (browser settings), click "Export PDF" — alert "Please allow popups…" appears.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add PDF export via print page"
```
