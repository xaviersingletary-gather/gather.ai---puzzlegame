# SPOT CHECK v1.1 — Demo Review

**Spec:** `SPOT_CHECK_SPEC_v1.1.md` · **Plan:** `tdd-plan.md`
**Date:** June 3, 2026
**Build:** `index.html` (single file, fully client-side — no backend, no network calls to any game backend)

## Verification method

The game renders entirely with DOM nodes (not Canvas), so it was driven in **jsdom** (a real DOM + the actual inline script) with a behavioral harness (`/tmp/spotcheck_test.js`) asserting on DOM state and `localStorage` — falsifiable checks, not "looks good." Audio (`AudioContext`) and clipboard/share were stubbed; a virtual console captured any script/DOM errors for the console-clean gate.

**Result: 34/34 checks passed, console clean.**

---

## Results by success criterion (spec §8)

| # | Criterion | Verdict | Evidence |
|---|---|---|---|
| 1 | Practice forced once; never again after completion/skip | **PASS** | Fresh storage routes to `screen-practice`; `spotcheck_onboarded` unset before, `true` after; persists on re-check |
| 2 | Practice replayable from menu | **PASS** | `startPractice()` re-enters practice; skip sets flag + returns to title |
| 3 | "WHAT IT SHOULD BE" / "WHAT'S ON THE FLOOR" labels each round | **PASS** | Ref label matches `/WHAT IT SHOULD BE/`; floor label present in game + practice |
| 4 | 6 error definitions on How-To | **PASS** | 6 `.htp-def` rows, all definition texts non-empty |
| 5 | Per-round discrepancy count, no pre-reveal | **PASS** | Floor objective includes `errorCells.length`; 0 resolved-* classes pre-submit; briefing states objective + "find them all" |
| 6 | Live grade matches end-screen logic | **PASS** | Pill value === `computeGrade(found, errsSoFar, falseSoFar)`; `computeGrade(14,14,0)` = PERFECT AUDIT; ladder (5 rows) on How-To |
| 7 | Streak nudge + freeze state | **PASS** | "6-day streak" shown; freeze surfaced at count 8 w/ freeze available; neutral copy at 0 (no "0-day streak") |
| 8 | Countdown to next puzzle | **PASS** | `end-countdown` shows "Next audit drops in HH:MM:SS" to local midnight |
| 9 | Challenge share, spoiler-free, clipboard fallback | **PASS** | Text is challenge-framed, has puzzle # + grade, only 🟢/⚫ (no bay IDs); clipboard path captured the text |
| 10 | No fake social data | **PASS** | Leaderboard placeholder renders honest "coming soon"; no fabricated ranks/names/percentile anywhere |
| 11 | Static-only / no backend calls | **PASS** | `submitScore()`/`getBenchmark()` return `null` (inert); no network calls to a game backend (only the pre-existing Google Fonts CDN link, unchanged from v1.0) |

Extra (My Stats, spec §6 resolved decision): **PASS** — Puzzles Played counts completions (5/5); Pass Rate = 80% (4 of 5 at PASSING+).

---

## Edge cases checked (spec §7)

- **Wrong tap in practice** → no-penalty nudge, no advance. **PASS**
- **Streak 0** → neutral copy, never "0-day streak." **PASS**
- **Grade parity at round 5** → final pill update uses full 14-error set, equals end-screen grade (shared `computeGrade`). **PASS** (verified via parity of the shared function)
- **Console clean** across the full flow. **PASS**

## Notes / deferred (per spec §11, intentionally out of scope)

- Real leaderboard, true percentile benchmark, email capture, Railway hosting → **v1.2**. Seams left in place: inert `submitScore()` / `getBenchmark()` stubs so v1.2 can swap in real calls without UI rework.
- Manual cross-browser pass at 390px/1280px in a real browser is still worth a human spot-check before ship (jsdom validates logic/DOM, not pixel layout). No layout regressions expected — changes reuse existing component classes.

## Final verdict

- [x] Passes all 11 success criteria from spec
- [x] Known gaps are the explicitly deferred v1.2 items only
- [x] Approved to ship pending a quick human visual pass in a real browser
