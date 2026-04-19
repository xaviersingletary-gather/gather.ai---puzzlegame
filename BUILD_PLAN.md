# SPOT CHECK — Build Plan

**Version:** 1.0  
**Date:** April 19, 2026  
**Scope:** v1 must-haves per §12.6 of the spec  
**Output:** Single HTML file, vanilla JS/CSS, < 100KB

---

## Architecture Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Grid rendering | DOM (not Canvas) | Simpler tap targets, CSS transitions, easier accessibility |
| PRNG | mulberry32 | Fast, deterministic, well-tested for this pattern |
| State | Single `gameState` JS object | Simple, inspectable, no framework needed |
| Persistence | localStorage | No backend required for v1 |
| Audio | Web Audio API | No external files, instant load |
| Fonts | Google Fonts CDN | Black Ops One, Inter, Share Tech Mono |
| Layout | CSS Grid + Flexbox | No frameworks, full responsive control |

---

## Build Phases

### Phase 0 — Scaffold
**Goal:** Working HTML shell with no game logic. Open in browser, see the title screen placeholder.

- [ ] HTML5 boilerplate with mobile viewport meta (`width=device-width, initial-scale=1, user-scalable=no`)
- [ ] Google Fonts CDN link (Black Ops One, Inter, Share Tech Mono)
- [ ] CSS custom properties: full palette, spacing scale, font stack
- [ ] Safe area inset variables (`env(safe-area-inset-*)`) for iPhone notch
- [ ] `gameState` object schema (round, scores, flags, streak, playerName, completed)
- [ ] mulberry32 PRNG implementation + seed formula (`spotcheck-YYYY-MM-DD` → 32-bit int)
- [ ] Screen router function (`showScreen(id)`) — single active screen at a time
- [ ] Static title screen HTML (no logic yet)

---

### Phase 1 — Puzzle Generation Engine
**Goal:** Call `generatePuzzle(dateString)` and get back a fully-formed puzzle object for all 5 rounds.

**Puzzle object shape:**
```js
{
  seed, date, puzzleNumber,
  facilityName,
  rounds: [{
    type, briefing, gridSize: {cols, rows},
    referencePanel: [{id, sku, qty, ...}],
    cells: [{id, sku, qty, badge, expDate, lotCode, weight, isEmpty, isError, errorType}]
  }]
}
```

**Tasks:**
- [ ] Bay ID generator: format `[A-F]-[01-24]-[01-04]`, sequential layout
- [ ] SKU pool: 200 pre-defined codes, consistent across rounds
- [ ] Quantity generator: standard pallet quantities per SKU (48, 60, 72, 96, 120) + partial variants
- [ ] Date generator: expiration dates relative to puzzle date (2 weeks ago → 6 months out)
- [ ] Lot code generator: alphanumeric format `[0-9][A-Z]-[000-999]`
- [ ] Weight generator: 800–2,800 lbs range
- [ ] Badge assignment: cold/hazmat/perishable/standard/high-value per SKU type
- [ ] Facility name picker: pool of 10-15 DC names (e.g., "DC-42 Memphis")
- [ ] 6 error generators (one function each):
  - `errorCountMismatch` — deviate qty from reference
  - `errorWrongSlot` — swap SKU with adjacent cell
  - `errorFIFO` — newer expDate in front position, older behind
  - `errorPhantomInventory` — cell shows qty but has `isEmpty: true` flag
  - `errorWeightViolation` — heavy pallet (>2,000 lbs) on level 3+
  - `errorZoneViolation` — frozen badge (snowflake) placed in ambient bay
- [ ] Error placement engine:
  - Respects per-round error count (2, 2, 3, 3, 4)
  - Enforces adjacency constraint (no 2 errors touching)
  - Enforces complexity progression (simple errors in R1-2, full pool in R5)
  - Saves yesterday's error positions to localStorage to prevent positional repeat
- [ ] Reference panel generator: internally consistent "expected" state per round type (Receiving, Pick Face, Bulk Storage, Staging, Full Floor)
- [ ] Puzzle number calculator: days since launch date (May 1, 2026 = #1)
- [ ] Daily difficulty tier: Easy (Mon) → Expert (Sun) — affects timer offset and error complexity flag

---

### Phase 2 — Screen Navigation & Title Screen
**Goal:** Full title screen with menu, How to Play, returning player state.

- [ ] Title screen layout: logo, subtitle, puzzle number + date, streak indicator
- [ ] Four menu buttons: START TODAY'S AUDIT, HOW TO PLAY, MY STATS, LEADERBOARD (stub)
- [ ] How to Play modal: single screen, 4-bullet rules, dismissable
- [ ] Returning player states:
  - Haven't played today → show streak, "TODAY'S AUDIT READY"
  - Already played → show today's score, "COME BACK TOMORROW"
  - Streak broken → "STREAK: 0 — Start a new one today"
  - Freeze used → "STREAK SAVED — don't miss again!"
- [ ] First visit detection → auto-show How to Play before title
- [ ] Name entry modal: display name (3–16 chars) + optional company, triggers on first play

---

### Phase 3 — Core Game Loop
**Goal:** Play a full 5-round game from START to round 5 SUBMIT.

- [ ] Pre-round briefing screen: round number, briefing text, 2-second auto-advance (tappable to skip)
- [ ] Game screen layout (portrait-first):
  - Top bar: puzzle ID + round indicator
  - Timer bar (thin, full-width, drains left→right)
  - Reference panel (scrollable, light blue-gray bg)
  - Grid (CSS Grid, responsive cell sizing)
  - SUBMIT button (full-width, 56px tall)
- [ ] Grid cell component: bay ID, SKU, qty (large bold), badge icon, optional fields (expDate, lotCode, weight)
- [ ] Cell state classes: `default` / `flagged` / `resolved-found` / `resolved-missed` / `resolved-false` / `resolved-clean`
- [ ] Tap handler: first tap → flag (yellow border + "!" badge), second tap → unflag
- [ ] Desktop hover: scale 1.02x, shadow deepens
- [ ] Timer logic:
  - Per-round durations: 20, 20, 25, 25, 30s (adjusted by difficulty tier)
  - Color shift: normal → yellow → orange → red (last 5s)
  - Auto-submit on expiry
- [ ] SUBMIT handler: marks round as submitted, triggers resolution phase
- [ ] Mid-puzzle state persistence: save `gameState` to localStorage on each action (survives refresh)

---

### Phase 4 — Round Resolution & Scoring
**Goal:** Animated round resolution + correct score calculation.

- [ ] Resolution animation sequence (left-to-right, top-to-bottom, 0.15s stagger):
  - Correct finds: green border + checkmark, "FOUND" label, scale pulse
  - False flags: orange border, "CLEAN" label, flash
  - Missed errors: red border, "MISSED" label, discrepancy value blinks red
  - Clean cells: fade slightly, no animation
- [ ] Per-round score calculation:
  - Correct flag: +100
  - Time bonus: +5 × seconds remaining
  - False flag: −50
  - Perfect round bonus: +50
- [ ] Round scorecard overlay (2-second display, tappable to skip):
  ```
  ROUND 3 — BULK STORAGE
  Errors found: 2/3
  False flags: 0
  Time bonus: +45
  Round score: 245
  ```
- [ ] Score count-up animation (0.5s per line item)
- [ ] Auto-advance to next round (or end screen after R5)

---

### Phase 5 — End Screen & Audit Report
**Goal:** Full end-of-game audit report with accuracy rating and Gather AI card.

- [ ] Audit report layout (white card, "report" feel):
  - Header: puzzle ID, date, facility name
  - Summary: errors found, false flags, accuracy %, total score, streak
  - Round breakdown: dot grid (green/gray), per-round scores
  - Per-game bonuses: perfect game (+200), streak multiplier
  - Streak multiplier tiers (5%/10%/15%/20%)
- [ ] Score tally animation on render
- [ ] Accuracy title assignment: PERFECT AUDIT / SENIOR AUDITOR / PASSING GRADE / NEEDS TRAINING / SEE YOUR SUPERVISOR
- [ ] Manager one-liner below title
- [ ] Action buttons: SHARE, LEADERBOARD (stub), MY STATS
- [ ] Gather AI insight card:
  - Slides up 2 seconds after report renders (does NOT cover report)
  - Content variant based on performance tier (12+, 8-11, <8, perfect)
  - "See How It Works" CTA button with UTM link
  - Dismissable (x button); dismissed state saved to localStorage until midnight

---

### Phase 6 — Persistence & Streak
**Goal:** All localStorage logic — one play per day, streak, history, resume.

- [ ] localStorage key schema:
  ```
  spotcheck_completed_YYYY-MM-DD: {score, errors, falseFlags, accuracy, puzzleNumber}
  spotcheck_inprogress_YYYY-MM-DD: {gameState snapshot}
  spotcheck_streak: {count, lastPlayed, freezeAvailable, freezeUsed}
  spotcheck_player: {name, company}
  spotcheck_mute: boolean
  ```
- [ ] One-play-per-day enforcement: on game start, check completion key; redirect to "already played" state if present
- [ ] Resume mid-puzzle: on game start, check in-progress key; restore `gameState` and jump to correct round/state
- [ ] Streak logic:
  - Increment on completion
  - Check for missed day on title screen load
  - Freeze availability: grant freeze at day 7+, consume on first missed day, reset on second consecutive miss
  - Streak milestone check: 7, 14, 30, 50, 100 — trigger flame bounce animation
- [ ] Score history: rolling 30-day array, used by MY STATS screen
- [ ] MY STATS screen: personal history table (date, score, accuracy, streak), best score, current streak, longest streak

---

### Phase 7 — Share Card
**Goal:** Copy-to-clipboard share text with emoji grid, native share on mobile.

- [ ] Emoji grid generator:
  - 14 positions (one per error across all rounds)
  - 🟢 = found, ⚫ = missed
  - Always 14 chars, same layout every day
- [ ] Share text template:
  ```
  SPOT CHECK #127 — SENIOR AUDITOR
  🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢⚫⚫
  12/14 found | 1,487 pts | 13-day streak 🔥
  gather.ai/spotcheck
  ```
- [ ] Copy to clipboard (primary, all platforms)
- [ ] `navigator.share` API on mobile with fallback to clipboard
- [ ] "Copied!" confirmation toast (1.5s, then dismiss)

---

### Phase 8 — Audio
**Goal:** All 9 sound types synthesized via Web Audio API, with mute toggle.

- [ ] Web Audio API context (lazy-init on first user interaction to bypass autoplay policy)
- [ ] Sound synthesizer functions (one per type):
  - `sfxTap()` — short click (flag)
  - `sfxUntap()` — softer reverse click (unflag)
  - `sfxSubmit()` — rubber stamp thud
  - `sfxCorrect()` — ascending two-note chime
  - `sfxMissed()` — soft low tone
  - `sfxFalseFlag()` — quick double-tap "nah"
  - `sfxPerfectRound()` — three-note ascending chime
  - `sfxTimerWarning()` — subtle ticking, tempo increases
  - `sfxPerfectGame()` — triumphant 4-note chord
- [ ] Mute toggle: visible on title screen, persisted to localStorage
- [ ] Volume: very quiet default (gainNode at 0.3)

---

### Phase 9 — Mobile Polish & Cross-Browser
**Goal:** Smooth on iPhone, no layout issues, passes performance targets.

- [ ] All tap targets ≥ 48×48px (verify in DevTools)
- [ ] SUBMIT button: full-width, 56px height
- [ ] No pinch-zoom on game grid (viewport meta already set in Phase 0)
- [ ] Grid cells scroll vertically if they exceed viewport — never shrink below readable size
- [ ] Font size floor: 14px body, 12px secondary data in cells
- [ ] Landscape orientation: reference panel shifts to left sidebar, grid centers
- [ ] Test on: Chrome mobile, Safari iOS, Chrome desktop, Firefox desktop
- [ ] File size audit: verify < 100KB total (target < 80KB)
- [ ] Load time test on simulated 4G throttle
- [ ] No layout shift on orientation change (test with DevTools)

---

### Phase 10 — Final QA Pass
**Goal:** Clean, shippable v1.

- [ ] Full game playthrough: title → 5 rounds → end screen → share → Gather AI card
- [ ] Resume mid-puzzle: close browser at round 3, reopen, verify state restored
- [ ] One-play-per-day: complete puzzle, reopen, verify locked state
- [ ] Streak: verify increment, freeze logic, break logic
- [ ] Streak multiplier: verify correct % applied to total
- [ ] Perfect game: all 14 found, 0 false flags — verify +200 bonus, "PERFECT AUDIT" title
- [ ] SEE YOUR SUPERVISOR: <50% found — verify title and one-liner
- [ ] Timer auto-submit: let timer expire mid-round, verify unflagged errors counted as misses
- [ ] Share card: verify emoji grid matches actual results
- [ ] Gather AI card: verify correct performance variant, dismiss persists until midnight
- [ ] Audio: verify all 9 sounds, mute toggle persists across reload
- [ ] Validate HTML (W3C), check console for errors
- [ ] Spell-check all user-facing strings

---

## File Structure

```
spotcheck.html          ← the entire game (single file)
  <style>               ← all CSS (custom properties, layout, components, animations)
  <body>                ← screen containers (title, howtoplay, game, endscreen, stats)
  <script>              ← all JS (PRNG, generator, gameState, screens, audio, share)
```

Internal JS sections (comment-delimited):
```
// === CONFIG ===
// === PRNG ===
// === PUZZLE GENERATOR ===
// === GAME STATE ===
// === SCREEN ROUTER ===
// === TITLE SCREEN ===
// === GAME LOOP ===
// === RESOLUTION ===
// === SCORING ===
// === END SCREEN ===
// === PERSISTENCE ===
// === SHARE ===
// === AUDIO ===
// === INIT ===
```

---

## Open Items (Blocking for Launch)

From §13 of the spec — resolve before deploying:

| # | Item | Default if unresolved |
|---|------|----------------------|
| 1 | Hosting URL | Hardcode `gather.ai/spotcheck` in share card and CTA |
| 3 | Demo booking CTA link | Use `gather.ai/demo` as placeholder |
| 4 | HubSpot form ID | Skip daily reminder opt-in (v1.1 item anyway) |
| 6 | Launch puzzle number | Default to #1 launch on May 1, 2026 |
| 7 | Daily reset time | Default to midnight UTC |

---

## v1.1 Backlog (Post-Ship)

- Remaining 4 error types (lot code split, damaged hold, short ship, duplicate LPN)
- Global leaderboard (Firebase or Supabase)
- Company leaderboard tab
- Daily reminder email opt-in → HubSpot
- Weekly recap screen (Sunday)
- Yesterday's puzzle (practice mode)
- Daily difficulty scaling (Mon Easy → Sun Expert)
- Analytics event tracking (GA4)
