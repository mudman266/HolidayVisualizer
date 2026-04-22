# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file vanilla web app — no build step, no dependencies, no package manager. Open `index.html` directly in a browser to run it.

## Architecture

Everything lives in `index.html`: CSS custom properties, HTML structure, and JavaScript are all inline. There is no bundler, framework, or external library.

**State object `S`** — single source of truth, persisted to `localStorage` under the key `hv2`:
- `settings` — `totalHolidayHours`, `hoursPerToken`
- `holidays` — map of `"YYYY-MM-DD"` → `{ tokens: 0.5|1, name: string }`
- `year`, `view` (`"year"|"month"`), `viewMonth`
- `bankPinned` — whether the token bank sticks below the header on scroll
- `qaEnabled`, `qaTokens` — quick-add mode state

**Render flow** — `render()` is the single entry point, called after any state mutation:
1. `renderHeader()` — year label, view toggle, month nav visibility
2. `renderReport()` — spent/remaining stats
3. `renderTokenBank()` — visual grid of squares (used vs. remaining tokens)
4. `applyBankPin()` / `applyQuickAdd()` — apply sticky and QA UI state
5. `renderCalendar()` → `buildYearView()` or `buildMonthView()` → `bindCalendarClicks()`

**Token bank** — squares are sized automatically (`size-lg` through `size-xs` CSS classes) based on total token count. Each square = 1 token; half-token shown as a left-split gradient.

**Quick Add mode** — when `S.qaEnabled`, `bindCalendarClicks()` attaches `mousedown`/`mouseenter` instead of `click`. Drag state lives in the module-level `_qa` object (not persisted). A global `mouseup` listener on `document` commits the drag.

**Day modal** — handles both single-day entry and multi-day ranges via the duration stepper (`_modalDays`). `saveDayModal()` loops `addDays()` from the start date.

## Dark mode

The site implements `prefers-color-scheme: dark` natively via CSS custom property overrides. All colours must be expressed as CSS variables — no hardcoded hex values in rule declarations. Dark Reader defers to this native implementation.
