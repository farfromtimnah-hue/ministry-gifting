# Ministry Gifting Assessment — Progress Log

> Recovery log for future sessions. Update after each milestone.

---

## Project Overview

| Item | Value |
|---|---|
| Repo | https://github.com/farfromtimnah-hue/ministry-gifting |
| Primary file | `index.html` (single-file assessment app) |
| Worker | `https://ltc-api.farfromtimnah.workers.dev` |
| Structure | `ALL_QUESTIONS` = 2 calibration + 45 gifting + 12 DISC = 59 total |

---

## Question Array Structure

- `CAL_QUESTIONS` (2) — calibration questions (indices 0–1 in ALL_QUESTIONS)
- `QUESTIONS` (45) — gifting questions (ALL_QUESTIONS indices 2–46)
  - `QUESTIONS[6]` — "shows up" reliability question (line 831), tagged `g:"worship"`
- `DISC_QUESTIONS` (12) — DISC questions (ALL_QUESTIONS indices 47–58)

`answers` object is keyed by `ALL_QUESTIONS` index. `QUESTIONS[6]` = `ALL_QUESTIONS[8]` (offset by 2 calibration questions).

**CORRECTION for future sessions:** The `answers` object keys correspond to the `ALL_QUESTIONS` index, not the `QUESTIONS` index. When the reliability_flag was implemented (Session 2 Redux), `answers[6]` was used as coded in the prompt — verify this produces correct results vs `answers[8]`. The question at `QUESTIONS[6]` is `ALL_QUESTIONS[8]`. May need correction to `answers[8]` if the flag is always 0.

---

## Sessions Completed

### Session 5 — DISC Assessment, Draft/Resume, Preferred Name
- DISC questions, calibration questions added
- `calcDisc()`, `calcCalibration()`, `deriveDISC()` functions
- POST /draft + GET /resume flow
- Saiba Mais modals, Carisma Level 5 confirmation overlay
- `GIFTING_LEARN_MORE` constant with full theological body text
- Commit: `3a0fc21`

### Session 8 — Five UX Fixes
- Scale level 5 label change
- Level 5 confirmation overlay
- DISC section visual changes (pill label, fade-in animation, teal border-top)
- DISC removed from results display (still calculated and sent)
- Learn More modal content rewrite
- Commit: `7843749`

### Session 9 — deriveDISC Canonical Values + Pairing Logic
- Fixed `deriveDISC` to store canonical English values (not Portuguese) in D1
- Added `DISC_NATURAL_STR`, `DISC_LEADERSHIP`, `DISC_EMOTIONAL`, `DISC_MINISTRY_FIT`, `DISC_PAIRING` lookup tables
- `deriveDISC` now accepts `g1, g2, g3` for pairing + ministry_fit logic
- Commit: `6cd4565`

### Session 2 Redux (2026-05-31) — Reliability Flag
- Added reliability flag computation using `answers[8]` (= ALL_QUESTIONS index for QUESTIONS[6], offset by 2 calibration questions)
- `answers[8] >= 4` → `reliability_flag = 1`
- Added `reliability_flag` to POST /draft payload (saveDraft function, line ~1219)
- Added `reliability_flag` to POST /submit payload (handleSubmit function, line ~1367)
- Commits: `e720d39` (initial), `276801c` (index correction answers[6]→answers[8])

---

## Pending D1 / Worker Actions

### D1 column — MUST RUN BEFORE reliability_flag SAVES:
```sql
ALTER TABLE submissions ADD COLUMN reliability_flag INTEGER DEFAULT NULL;
```
Run in Cloudflare D1 console for the `ltc-db` database.

### Worker — check if reliability_flag is accepted:
The Worker's POST /submit handler may not pass `reliability_flag` to D1 bind. Verify the Worker reads and binds `reliability_flag` from the request body. If not, add it to the Worker's submit handler alongside `pastoral_flag`.

---

## Critical Rules

- iOS Safari: `addEventListener('touchstart', fn, {passive:false})` + `addEventListener('click', fn)`; no `onclick` attributes
- IIFE closures for all dynamically created buttons
- No dashes (hyphens as punctuation) in any user-facing string
- `ALL_QUESTIONS` index vs `QUESTIONS` index: differ by 2 (calibration offset)
- DISC labels: always Executor/Comunicador/Planejador/Analista — never D/I/S/C letters as labels
- Canonical D1 values always in English; only display translates

### Session 4 (2026-05-31) — Ministry/Group List Updates
- Fixes 1–5 were already implemented in prior sessions (confirm button text, DISC pill/fade/border, scale labels, logo, Carisma 2-option rendering)
- Fix 6: Added "WE CARE" to MINISTRY_LIST (line 528)
- Fix 7: Added "CRIE", "Gerações", "Carisma Student" (EN) / "Aluno do Carisma" (PT) to GROUP_VALS, GROUP_LIST_EN, GROUP_LIST_PT (lines 529–531)
- Commit: see below

---

_Last updated: 2026-05-31 — Session 4 (ministry/group list updates) complete._

---

## Session 5 (2026-05-31) — Pastoral Flag Redesign

### Fix A1 — Pastoral flag trigger logic (index.html)

**Lines changed:** 943 (function signature), 981–982 (trigger), 1342–1343 (call site)

**What changed:**
- Added `leadershipScore` as 8th parameter to `deriveDISC(d,i,s,c,g1,g2,g3,leadershipScore)`
- Old trigger: `(s>=12&&i>=10)||(s>=12&&c>=12)||(s>=13)?1:0` — fired too broadly on high-S profiles
- New trigger: ALL THREE must be true simultaneously:
  1. `s >= 12` (high relational steadiness)
  2. `leadershipScore >= 60` (gifting "Influence & Servant Leadership" at ≥60%)
  3. `leadership_en` is "Visionary Leader", "Relational Leader", or "Structural Leader"
- `handleSubmit` now computes `leadershipScore = scores["Influence & Servant Leadership"] || 0` before calling `deriveDISC`

**Why `leadershipScore` is a parameter:** Inside `deriveDISC`, the local `scores` object is `{D,I,S,C}` (DISC raw scores), not gifting percentages. Gifting pct scores are built in `handleSubmit`. Passing as parameter was the correct approach.

**Commit:** `1a85ebc`

---

## Session 6 (2026-05-31) — Mobile iOS Safari Fixes

### Fix A1 — WhatsApp share button iOS (line 1787)
- Removed `e.preventDefault()` from touchstart handler on `waShareBtn`
- Changed `passive:false` to `passive:true`
- Without this, iOS Safari blocks `window.open()` called from a touchstart with preventDefault as an untrusted popup
- **Commit:** `19cf95b`

### Fix A2 — Viewport height (lines 15, 525–528)
- Added `:root{--app-height:100vh}` CSS at top of style block
- Changed `.app` from `min-height:100vh` to `min-height:var(--app-height,100vh)`
- Added `setAppHeight()` JS function before global vars (line 525): sets `--app-height` to `window.innerHeight`
- Fires on load, `resize`, and `visualViewport.resize`

### Fix A3 — Input font-size
- `.field input` already has `font-size:16px` — no change needed
- `.prefname-input` has `font-size:1.3rem` — already above 16px
- `html,body` already has `-webkit-text-size-adjust:100%`

### Fix A4 — Name on results screen (line ~71)
- `.r-name` changed from `font-size:.72rem` (~11.5px) to `font-size:1.75rem`, `font-weight:800`, `display:block`
- Removed `::before` decorative line (no longer needed with block display)
- Name is now bold, teal (#2ABFBF), unmissable

### Fix A5 — Overlays (lines ~190–192)
- `.modal-overlay` updated: explicit `top/left/right/bottom:0`, `height:var(--app-height,100vh)`, `transform:translateZ(0)`, `-webkit-transform:translateZ(0)`
- `.modal-sheet` updated: added `-webkit-overflow-scrolling:touch`
- All buttons already had `touch-action:manipulation` and `-webkit-tap-highlight-color:transparent`
- **Commit (A2–A5):** `d752cac`

_Last updated: 2026-05-31 — Session 6 (iOS mobile fixes) complete._

---

## Session 7 (2026-06-01) — Group Roles Feature

### D1 Schema
- Created `group_roles` table: `id, submission_id, group_name, role, created_at`
- Added `group_attendance TEXT DEFAULT '[]'` column to `connections` table

### Worker (ltc-api/worker.js) — deployed 9019cd1f
- `PEOPLE_SELECT`: added `c.group_attendance`
- `GET /person/:id`: fetches `group_roles` rows and returns in response alongside notes
- `PUT /person/:id/connection` (both authenticated and unauthenticated paths): accepts `group_attendance` (JSON array) and `group_roles` (array of `{group_name, role}` objects); saves via COALESCE for attendance and DELETE+INSERT for roles

### Assessment App (ministry-gifting/index.html) — commit 630654b
- **Section A (always shown):** 9 group attendance chips with info (i) popup; GC connected Yes/No chips
- **Section B:** existing serving gate unchanged
- **Section C:** Sunday ministry chips with updated label + sublabel
- **Section D:** 10 group role chips with expandable role checklists per group
- New CSS: `.chip-with-info`, `.role-check-row`, `.gc-chip`, `.group-info-popup`, `.group-info-backdrop`
- New data: `ATTENDANCE_GROUPS` (descriptions PT/EN), `GROUP_ROLE_MAP` (roles per group)
- New vars: `shareSelectedGroupAttendance`, `shareGroupRoles`, `shareGcConnected`
- `submitShareMore` sends `group_attendance` and `group_roles` in payload

### Dashboard (ltc-dashboard/src/App.jsx) — commit 5ed4a05
- PersonPanel display: group attendance (grey chips) and group roles (teal chips, grouped by group name), shown above Notes for all roles
- PersonPanel edit: group attendance toggles (auto-save), group roles chip + role checklist with dirty-state Save button
- New constants: `ATTENDANCE_GROUPS_DASH`, `GROUP_ROLE_MAP_DASH`
- New state: `editGroupRoles`, `groupRolesDirty`

_Last updated: 2026-06-01 — Session 7 (group roles feature) complete._
