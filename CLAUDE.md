# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Chấm Công Pro** is a Vietnamese-language workforce time-tracking and salary-calculation PWA (Progressive Web App). It runs entirely in the browser with no backend — all data is persisted in `localStorage`. The entire application is a **single HTML file** (`ChamCongPro20.html`) containing embedded CSS and vanilla JavaScript.

## Development Workflow

There is no build system, package manager, or test suite. To develop:

- Open `ChamCongPro20.html` directly in a browser (or serve it via any static file server, e.g. `python3 -m http.server`).
- All edits are made directly to `ChamCongPro20.html`.
- The app can be deployed as-is to GitHub Pages.

## Rules for AI Assistants

1. **UI is off-limits for logic fixes** — Do not modify HTML/CSS when the task is a logic or calculation bug fix.
2. **Never delete localStorage keys** — Existing keys (`ccp_cfg`, `ccp_hs`, `ccp_cc`, `ccp_gio`, `ccp_gc`) must not be removed; only add new ones.
3. **No public function renames without a full-file search** — Before renaming any function called via `onclick`/`oninput` or referenced in other JS, verify all call-sites in `ChamCongPro20.html`.
4. **State machine changes must preserve `[SA/SA]` log markers** — Any edit to the screen-switching logic (`.man-chao`, `.ob-wrap`, `.app-ch` visibility transitions) must keep existing `[SA/SA]` markers intact.
5. **Syntax-check JS after every edit** — Extract the `<script>` block and run `node --check <file>` (or a JS linter) before reporting completion.
6. **Always report: root cause, files changed, remaining risks** — Every bug-fix response must include these three items explicitly.
7. **No new dependencies without approval** — Do not introduce CDN `<script>` tags or any external resources without explicit user permission.

## Architecture

### File Structure

The single file is organized in order:
1. `<head>` — minified CSS in a `<style>` block (large, ~900 lines)
2. `<body>` — all HTML markup for every screen/panel (hidden/shown via JS)
3. Inline `<script>` — all JavaScript (~400 lines), structured as:
   - Constants and data tables
   - Global state variables
   - Initialization and onboarding flow
   - Calendar rendering
   - Attendance recording and salary calculation
   - Settings, export, and notification logic

### UI Screens & State Machine

The app has three main display states, toggled by showing/hiding elements:

| State | Element | Description |
|---|---|---|
| Welcome/Mode select | `.man-chao` | First-run screen to pick operating mode |
| Onboarding wizard | `.ob-wrap` | 4–5 step setup (name, shift config, salary, etc.) |
| Main app | `.app-ch` | Calendar + header; the primary working screen |

Panels/modals layer on top via `.panel.mo` and `.mp.mo` classes.

### Operating Modes

`cheDoc` (global variable) controls which features are active:
- `'cc'` — Timekeeping only (no salary calculation)
- `'cl'` — Timekeeping + salary calculation

### Global State

| Variable | Purpose |
|---|---|
| `cfg` | App configuration (shifts, salary settings, colors, etc.) |
| `hoSo` | Employee profile (name, job title, company, initials) |
| `ccD` | Daily attendance records, keyed by `'YYYY-M-D'` |
| `gioD` | Worked-hours records per day |
| `ghiChuD` | Notes per day |
| `dsCa` | Active shift list |
| `cheDoc` | Operating mode (`'cc'` or `'cl'`) |

All state is serialized to `localStorage` under these keys:
- `ccp_cfg`, `ccp_hs`, `ccp_cc`, `ccp_gio`, `ccp_gc`

### Attendance Record Schema

```js
ccD['2026-4-21'] = {
  tt: 'cm',    // status: cm=present, np=leave, vang=absent, ll=holiday-work
  v: '08:00',  // check-in time
  r: '17:00',  // check-out time
  gc: ''       // note
}
```

### Key Functions

| Function | Purpose |
|---|---|
| `khoi()` | App initialization — loads data, decides first screen |
| `chonCD(cd)` | Choose operating mode on welcome screen |
| `batDauOB()` | Start onboarding wizard |
| `hienApp()` | Render/show main app view |
| `veLich()` | Render the monthly calendar grid |
| `luuCC()` | Save an attendance record |
| `tinhGio()` | Calculate worked hours for a day |
| `tinhLuong()` | Calculate daily salary |
| `tinhTangCa()` | Calculate overtime pay |
| `moCS()` | Open settings panel |

### Shift System

Shifts are stored in `cfg.dsCa` (array of shift objects). Each shift has:
- `ten`: name
- `v`/`r`: start/end times (HH:MM)
- `dem`: boolean — night shift flag (triggers +30% premium per Vietnamese law)
- `nghi`: rest days bitmask

Status codes used in `TT_CFG`: `cm` (present), `np` (leave/nghỉ phép), `vang` (absent), `ll` (holiday work/làm lễ).

### Salary Calculation Rules (Vietnamese Labor Law)

- **Night shift premium**: +30% (`dem` flag on shift)
- **Overtime**: +50% (standard days), varies for holidays
- **Social/health insurance deductions**: BHXH, BHYT, TNCN
- **Regional minimum wage**: based on Decree 293/2025, hardcoded in constants
- **Vietnamese public holidays 2025–2026**: hardcoded in `NL` constant

## Naming Conventions

Variable and function names are Vietnamese abbreviations:
- `cc` = chấm công (timekeeping)
- `ca` = ca (shift)
- `gio` / `g` = giờ (hours)
- `luong` / `l` = lương (salary)
- `nghi` = nghỉ (day off)
- `dem` = đêm (night)
- `tang ca` / `tc` = overtime
- `ob` = onboarding
- `mp` = modal panel
- `dtr` = header area
- `vl` = calendar view (lịch)
- `hs` = hồ sơ (profile)
- `cfg` = cấu hình (configuration)

HTML element IDs use prefixes: `ob-` (onboarding), `mp-` (modal), `el-` (generic element).

Inline event handlers (`onclick`, `oninput`, `onchange`) are used throughout — this is the established pattern, not a bug.
