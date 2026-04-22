# Holiday Visualizer

A web app that puts your days off into perspective — not the events of a day, but the days themselves.

Open `index.html` in any browser. No install, no server, no dependencies.

---

## What it does

Most calendar apps focus on scheduling events. Holiday Visualizer focuses on your leave budget as a finite resource. You see your allotted days as a physical bank of squares that deplete as you use them, making it immediately obvious how much you have left and how quickly it's going.

---

## Features

### Token Bank
The centrepiece of the app — a grid of squares where each square represents one token (one day off). Squares fill in from the left as you use them. Half-day tokens appear as half-filled squares. The bank auto-sizes based on your total token count.

### Token System
- Each day off costs one token; half-days cost half a token
- Configure your total holiday hours per year and hours per token in Settings
- Tokens and hours are shown together throughout (e.g. `40hrs (5d)`)

### Calendar Views
- **Year view** — all 12 months at a glance, with holiday days coloured. Click a month name to jump to month view.
- **Month view** — full-size calendar showing each day. Holiday entries appear as labelled chips inside their day cell.

### Quick Add Mode
Activate with the **Quick Add** button in the header for rapid data entry:
- **Click** any day to toggle its holiday status instantly (no modal)
- **Click and drag** across a date range to toggle all days in one gesture
- The drag direction is determined by the first day touched — empty days enter *add* mode, days with a holiday enter *remove* mode
- Choose **Full** or **Half** day tokens from the quick add bar

### Day Entry Modal
Click any day in normal mode to open the detail modal:
- Set token type: None, Half Day, or Full Day
- Add an optional label (e.g. "Annual Leave", "Christmas")
- Use the **Duration stepper** to apply the same settings across multiple consecutive days in one save

### Report Bar
Always visible at the top:
- **Total** — your full holiday budget for the year
- **Spent** — hours and days used so far
- **Remaining** — what's left (turns red if over budget)

### Token Bank Pin
The pin button (📌) in the token bank header makes the bank stick below the top navigation as you scroll, keeping your remaining balance visible at all times.

### Dark Mode
Fully supports `prefers-color-scheme: dark`. Works correctly with the Dark Reader browser extension.

---

## Settings

| Setting | Description |
|---|---|
| Holiday Hours / Year | Your total leave budget (e.g. 160 hrs) |
| Hours per Token | How many hours one token represents (e.g. 8 hrs = 1 work day) |

All data is saved automatically to `localStorage` — nothing leaves your browser.
