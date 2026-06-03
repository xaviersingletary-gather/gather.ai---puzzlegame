# SPOT CHECK v1.1 — TDD Execution Plan

**Spec:** `SPOT_CHECK_SPEC_v1.1.md` (v1.1, fully static / client-only)
**Date:** June 3, 2026
**Owner:** Xavier Singletary / Gather AI
**Target file:** `index.html` (single file, vanilla JS/CSS, Canvas 2D)

---

## Resolved decisions (best-practice defaults applied)

| § | Question | Decision |
|---|---|---|
| 6.6 | Live-grade placement | Compact pill in the top bar; updates **between rounds only** (on scorecard overlay dismiss), never mid-round. Feedback at natural breakpoints, not during the timed task. |
| 6.2 | Practice bay error type | **Wrong Count** — most universally recognizable discrepancy. |
| MyStats | Stats to promote | Existing (current streak, best streak, best score) **+ Puzzles Played + Pass Rate** (% of completions at `PASSING GRADE` or better). |

---

## Testing approach (observability)

This is a single static HTML file with no test framework. Tests are **falsifiable behavioral checks run in a browser** (Chrome desktop + mobile emulation), asserting on **specific DOM state and `localStorage` keys** via DevTools — not "looks good."

Standing harness for every phase:
- **Fresh-player reset:** DevTools → `localStorage.clear()` → reload = simulates first-ever visit.
- **Returning-player reset:** seed known keys (`spotcheck_streak`, `spotcheck_completed_*`, `spotcheck_onboarded`) → reload.
- **Console-clean gate:** no uncaught JS errors in console during any flow (regression guard for a single-file app where one error halts everything).
- **Both viewports:** every visual check verified at 390px (mobile) and 1280px (desktop).

A phase **passes its gate** only when its test assertions hold AND the console-clean gate holds. Do not advance on a failed gate.

---

## Phase 1 — Onboarding: practice round + first-visit routing

**Build:**
- New scripted practice screen/flow with a fixed mini-bay (small grid, 1 obvious **Wrong Count** error + ≥1 clean bay). Un-timed, no scoring, no streak effect.
- Step-sequenced callouts (spec §6.2 steps 1–4); wrong taps get a no-penalty nudge; correct tap advances.
- "Skip — I've got it" affordance at every step.
- First-visit routing: `IF !localStorage.spotcheck_onboarded THEN run practice before title flow`. Set `spotcheck_onboarded = true` on completion **or** skip.

**Test:**
- Fresh-player reset → reload → asserts: practice screen is shown before the normal title/start flow.
- Tapping the scripted error cell → asserts: callout advances to step 4; tapping a clean cell first → asserts: nudge shown, no advance, no score change.
- Complete practice → asserts: `localStorage.spotcheck_onboarded === true`; reload → practice **not** shown again.
- Click "Skip" at step 1 → asserts: flag set, practice exits, normal flow reachable.

**Gate:** All four assertions pass; console clean; verified both viewports.

---

## Phase 2 — Onboarding: persistent labels + error-type definitions

**Build:**
- Reference panel header → **"WHAT IT SHOULD BE — WMS Manifest"**; grid region labeled **"WHAT'S ON THE FLOOR."** Rendered in every real round (both Hard and Easy).
- How-To screen: each of the 6 error-type tags gets a one-line plain-English definition (expand/tooltip). Append definition text to Easy-mode "Look for:" briefing hint.

**Test:**
- Start a Hard round → assert both label strings present in DOM. Repeat in Easy round → both present.
- Open How-To → assert all 6 error types render a non-empty definition string (check each tag's definition element).
- Easy briefing → assert "Look for:" hint includes the type name **and** its definition.

**Gate:** Labels present in both modes; 6/6 definitions present; Easy hint enriched; console clean; both viewports.

---

## Phase 3 — Win clarity: per-round objective display

**Build:**
- Pre-round briefing + in-game top bar show the round's discrepancy count from `round.errorCells.length`, e.g. "3 discrepancies hidden — find them all, don't flag clean bays."
- Count only; never reveals which cells or which type.

**Test:**
- For each of the 5 rounds of today's puzzle → assert displayed count === `round.errorCells.length`.
- Assert no cell carries a pre-reveal marker/class tied to being an error before submit (grep rendered DOM for any error-flagging class on error cells pre-submit).
- Shown in both Hard and Easy.

**Gate:** Counts correct for all 5 rounds; no pre-reveal; both modes; console clean.

---

## Phase 4 — Win clarity: live grade indicator + grade ladder surfacing

**Build:**
- Compact grade pill in top bar showing current tracked tier, derived from rounds resolved **so far** using the existing tier thresholds (`PERFECT AUDIT` / `SENIOR AUDITOR` / `PASSING GRADE` / `NEEDS TRAINING` / `SEE YOUR SUPERVISOR`).
- Pill updates **only on scorecard-overlay dismiss** (between rounds), not mid-round.
- Grade ladder shown on title and/or How-To as "what you're playing for."

**Test:**
- Play round 1, miss/hit a known number of errors → dismiss scorecard → assert pill tier matches the tier computed from cumulative errors-found via the same threshold logic the end screen uses.
- Assert pill does **not** change during an active round (value stable from round-start until next scorecard dismiss).
- Assert end-screen final grade === last pill value after round 5.
- Assert grade ladder visible on title/How-To.

**Gate:** Pill matches end-screen logic at every breakpoint; stable mid-round; ladder visible; console clean.

---

## Phase 5 — Replay: end-screen streak nudge + next-puzzle countdown

**Build:**
- End screen: streak-risk nudge — `IF streak.count > 0 THEN "🔥 {n}-day streak — come back tomorrow to keep it."` Surface freeze status when `count >= 7 && freezeAvailable`.
- Live countdown to next puzzle (local midnight), ticking.

**Test:**
- Seed `spotcheck_streak = {count: 6, ...}` → finish game → assert nudge shows "6-day streak"; seed `count: 8, freezeAvailable: true` → assert freeze status surfaced.
- Seed `count: 0` → assert no misleading streak nudge (or shows neutral first-day copy, not "0-day streak").
- Assert countdown element decrements over a few seconds and targets local midnight.

**Gate:** Nudge reflects seeded streak states correctly; countdown live and correct; console clean; both viewports.

---

## Phase 6 — Replay: challenge share + honest leaderboard placeholder + My Stats additions

**Build:**
- Challenge-framed share variant: "I scored {score} on today's Spot Check ({grade}). Think you know the floor better? {URL}" + spoiler-free v1.0 emoji grid + streak line. Native share with **clipboard fallback** where Web Share API is unavailable.
- Replace `alert('Leaderboard coming soon!')` with an honest placeholder screen → points to My Stats + share. **No invented scores/names/percentiles.**
- My Stats: add **Puzzles Played** (count of `spotcheck_completed_*` keys) and **Pass Rate** (% of those at `PASSING GRADE`+).
- Add no-op stubs `submitScore()` / `getBenchmark()` (spec §11) so v1.2 can swap in real calls without UI rework.

**Test:**
- Trigger share → assert text contains score, grade, URL, and emoji grid; assert grid reveals **no** cell positions/types (spoiler-free). On a no-Web-Share context → assert clipboard fallback fires.
- Open Leaderboard → assert placeholder screen renders and contains **no** numeric ranks or player names (grep for any fabricated data).
- Seed 4 completions (3 at PASSING+, 1 below) → open My Stats → assert Puzzles Played = 4, Pass Rate = 75%.
- Assert `submitScore()`/`getBenchmark()` exist and are inert (no network calls — confirm Network tab empty).

**Gate:** Share correct + spoiler-free + fallback works; placeholder honest (no fake data); stats math correct; no network calls; console clean.

---

## Final Integration Test

**Input:** Cleared `localStorage`, today's puzzle, Chrome desktop (1280px) + mobile emulation (390px).

**Flow:** Load → forced practice (complete it) → title → start Hard → observe per-round objective counts (P3) + persistent labels (P2) + grade pill updating between rounds (P4) → finish → end screen shows final grade, streak nudge + countdown (P5) → challenge share spoiler-free with clipboard fallback (P6) → My Stats shows Puzzles Played / Pass Rate (P6) → Leaderboard shows honest placeholder (P6) → reload: practice not re-forced (P1).

**Pass criteria (all must hold):**
1. Practice forced once, never again after completion/skip.
2. "WHAT IT SHOULD BE" / "WHAT'S ON THE FLOOR" labels present every round.
3. All 6 error definitions present on How-To.
4. Per-round discrepancy count correct for all 5 rounds; no error pre-reveal.
5. Live grade pill matches end-screen grade logic at each between-round breakpoint.
6. Streak nudge + live countdown correct for seeded streak states.
7. Challenge share is spoiler-free, contains score/grade/URL, clipboard fallback works.
8. Leaderboard placeholder contains no invented data; no percentile/rank anywhere.
9. My Stats Puzzles Played + Pass Rate compute correctly from seeded completions.
10. Zero uncaught console errors across the entire flow; zero network calls to any game backend.

**Verdict recorded in:** `demo-review.md` (per-criterion Pass/Fail with actual observed values).

---

## Sequencing & gate discipline

- Build phases **in order**; run each phase's test and confirm its gate before starting the next.
- P1→P2→P3→P4→P5→P6 is dependency-ordered (onboarding/labels before objective/grade before replay surfaces).
- On a failed gate: diagnose → fix → retest the same phase. Do not paper over with assumptions or advance early.
- Commit per passed phase (small, reviewable diffs in the single file).
