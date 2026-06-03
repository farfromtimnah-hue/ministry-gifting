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

---

## Session 8 (2026-06-01) — Emotional Profile Recalibration

### Problem solved
Old logic: `emotional_profile = DISC_EMOTIONAL[primary]` — a flat primary-only lookup.
Two flaws: ignored borderline secondary influence; Brazilian cultural S baseline made Steady Carrier nearly universal.

### Change (index.html only — no Worker or App.jsx changes)
- Added `deriveEmotionalProfile(disc_d,disc_i,disc_s,disc_c)` function before `deriveDISC`
- Gap-based: if top two DISC scores differ by <=2, secondary type influences the result
- I always wins emotionally in borderline cases (S/I, D/I, C/I all → Expressive Processor)
- S+C borderline → Steady Carrier; D+C borderline → Driven Processor; D+S borderline → Driven Processor
- Portuguese display label derived from a reverse-lookup map inside `deriveDISC`
- **Lines changed:** old lines 1068-1070 replaced; new function inserted before `deriveDISC` (net +32 lines)
- **Commit:** `560a633`

### D1 SQL (Nicole runs manually)
SELECT first to review, then 5 UPDATE statements (one per borderline combination) — see session transcript.
D primary + C secondary is already Driven Processor; no UPDATE needed for that case.

### Open question — gap threshold
Nicole's scores: D=9, I=8, S=12, C=12. Result: Steady Carrier (gap between S and I = 4, outside <=2 threshold).
Nicole self-describes as verbal processor. Decision pending: widen S vs. I threshold to <=4 specifically?
Do not implement until Nicole confirms.

_Last updated: 2026-06-01 — Session 8 (emotional profile recalibration) complete._

---

## Session 9 (2026-06-01) — Emotional Profile: Calibration Signal + New CA Questions

### Problem solved
Two-part fix:
1. Old CA questions (group adaptation, decision speed) were not useful signals for emotional processing style.
2. The gap-based DISC logic alone (Session 8) was insufficient for Nicole's own profile (S=12, I=8, gap=4 > threshold).

### Changes (index.html only — commit 54ef385)

**CA1 (line 917):**
- Old: "In a group of new people, I usually adapt to the group's rhythm before taking initiative."
- New: "When I'm carrying something heavy emotionally, talking it through with someone I trust is what actually helps me feel better."
- PT: "Quando estou carregando algo pesado emocionalmente, conversar com alguem de confianca e o que realmente me ajuda a me sentir melhor."

**CA2 (line 918):**
- Old: "Before acting, I prefer to think carefully about consequences and make sure I have done everything right."
- New: "When I'm carrying something heavy emotionally, I need time and space alone to process before I'm ready to talk."
- PT: "Quando estou carregando algo pesado emocionalmente, preciso de tempo e espaco sozinho para processar antes de estar pronto para conversar."

**`deriveEmotionalProfile(d,i,s,c,calA,calB)` (line 1057):**
- Now 6 params. SIGNAL 1: if caDiff >= 2 -> Expressive Processor; if caDiff <= -2 -> Steady Carrier.
- SIGNAL 2 (fallback): gap-based DISC logic from Session 8, unchanged.

**`deriveDISC(d,i,s,c,g1,g2,g3,leadershipScore,calA,calB)` (line 1097):**
- Added calA, calB as params 9 and 10.
- Passes calA, calB to deriveEmotionalProfile (line 1109).

**`handleSubmit` (lines 1511-1534):**
- `calA = answers[0]+1` (CA1 answer, 1-5 scale)
- `calB = answers[1]+1` (CA2 answer, 1-5 scale)
- Both passed to deriveDISC call (line 1518)
- Both added to submit payload as `calibration_a` and `calibration_b`

### D1 action needed
- Run `UPDATE submissions SET emotional_profile = 'Expressive Processor' WHERE id = 19;` (1 row — D/I borderline confirmed from prior session data review)
- Existing records cannot be retroactively corrected via calibration signal (old CA questions measured different things)

### Worker update (Session 10 — same day)
- D1 ALTER TABLE done by Nicole: `ALTER TABLE submissions ADD COLUMN calibration_a INTEGER DEFAULT NULL` and `calibration_b`.
- Worker `/submit` handler updated: `calibration_a` and `calibration_b` extracted from body via `num()`, added to both UPDATE (resume path) and INSERT (fresh path) SQL statements and `.bind()` calls.
- Deployed: Cloudflare Worker Version ID `590002ea-23d8-46f0-aba6-1f397eadcef7`
- Worker git initialized at `/Users/nicolel/ltc-api` — commit `bd0a670`
- INSERT: 27 columns, 26 `?` + hardcoded `'complete'` — verified match.

_Last updated: 2026-06-01 — Session 9 (calibration signal + new CA questions) complete._

---

## Session 10 (2026-06-01) — Person Modal UX Fixes + Group Leader View Enhancements

### D1
- Created `group_attendance_log` table (CREATE TABLE IF NOT EXISTS)
- Backfilled id=15 (Legacy, SHINE, English Service) with submitted_at as joined_at

### Worker (ltc-api/worker.js) — deployed 2fa8487a
- **2A** PUT /person/:id/name: new endpoint, auth required, updates name + preferred_name via COALESCE
- **2B** POST /submit: writes group_attendance_log rows after connection insert if body.group_attendance provided
- **2C** PUT /person/:id/connection: appends new groups to attendance_log (pastor path only, append-only, no deletes)
- **2D** GET /person/:id: now returns attendance_log array alongside notes and group_roles

### Dashboard (ltc-dashboard/src/App.jsx) — commit d4a2b7e

**Person Modal (PersonPanel):**
- Added `role` prop; threaded through GiftingTab + both PersonPanel call sites
- New state: pastoralInfoPopup, assignedPastorOpen, groupsRolesOpen, nameEditMode, nameEditFull, nameEditPreferred, nameSaving
- **3A** Special Groups section removed from display (data preserved in D1)
- **3B** Assigned Pastor: collapsible header showing assigned name when set; open when no pastor assigned
- **3C** Group Attendance + Group Roles edit sections wrapped in "Grupos e Funcoes / Groups & Roles" collapsible; collapsed by default when data exists, open when empty
- **3D** Pastoral label: "Candidato Pastoral / Pastoral Candidate" + "Remover indicacao / Remove Flag"; info popup (i) button with tooltip text (EN + PT)
- **3E** Name edit: pencil icon next to name header (hidden for group_leader); inline form with full name + preferred name inputs + Save/Cancel; calls PUT /person/:id/name
- **3F** Current Ministries read-only display added below Suggested Placements in the lower (read-only) section; order: Suggested Placements > Current Ministries > Groups Attended > Group Roles

**Group Leader View (GroupLeaderView):**
- **4A** Tab order swapped: Attending first, Serving second; default active tab: Attending
- **4B** Attending tab rebuilt: all attendees included (including servers); "Also Serving" sub-section (collapsible, compact avatar+roles); "Not Yet Serving" sub-section (full PersonRow, recruiting view)
- **4C** Skipped — recharts not installed. To implement mini analytics, run `npm install recharts` in ltc-dashboard and create a separate session.
- **4D** Sunday Pool (Section B) now collapsible by default; each ministry row collapses/expands independently; only one ministry open at a time via expandedMinistry state

_Last updated: 2026-06-01 — Session 10 (Person Modal UX + Group Leader View) complete._
