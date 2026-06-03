# SPOT CHECK — v1.1 Spec (UX & Retention Fixes)

**Version:** 1.1
**Date:** June 3, 2026
**Owner:** Xavier Singletary / Gather AI (GTM Engineering)
**Status:** Draft — pending review before build
**Builds on:** `SPOT_CHECK_SPEC.md` (v1.0). This document specifies *changes and additions only*. Anything not mentioned here is unchanged from v1.0.
**Scope constraint:** v1.1 is **fully static / client-only** — no backend, no Railway, no hosting change, no domain work, no email capture. Anything requiring a server is explicitly deferred to **v1.2** (see §11).

---

## 1. Purpose

v1.0 shipped a playable daily warehouse-audit puzzle, but playtesting surfaced three problems that block the game from doing its actual job (ICP engagement + organic distribution to warehouse buyers):

1. **Players don't know how to play.** The core mechanic — cross-referencing the *Reference panel* ("should be") against the *grid cells* ("on the floor") and tapping mismatches — is never taught. The How-To is text-only, the briefing auto-launches into a live timer in 2 seconds, and the six error types are bare tags with no explanation.
2. **Players don't know how to win.** A grade ladder exists (`PERFECT AUDIT` → `SEE YOUR SUPERVISOR`) but is only revealed *after* the game. Going in, the player sees a timer and a rising score with no stated target and no per-round error count.
3. **No reason to replay.** The retention loop is half-built: the leaderboard is an `alert('coming soon')` stub, share has no hook, and nothing pulls the player back the next day.

v1.1 fixes #1 and #2 fully, and addresses #3 with the **Wordle model**: replay is driven by a daily streak + a spoiler-free, challenge-framed share — *not* a live leaderboard. A real cross-player leaderboard and a true percentile benchmark both require a backend and are deferred to v1.2.

**Success in one sentence:** A first-time player understands the mechanic without instruction-reading, plays *toward* a visible goal, and has a concrete reason (streak, a colleague's score to beat) to come back tomorrow.

---

## 2. Scope

### In scope (all client-side, ships on current static hosting)
- **P1 — Onboarding:** Forced, un-timed interactive practice round on first-ever visit (skippable, replayable from menu). Persistent in-game "should be / on the floor" labeling. Per-error-type plain-English definitions.
- **P2 — Win clarity:** Stated objective per round (error count shown up front). Visible grade ladder + target *before and during* play.
- **P3 — Replay (Wordle model):** Strengthen the existing client-side streak loop. End-screen streak-risk nudge + next-puzzle countdown. "Challenge a colleague" share variant (spoiler-free). Promote the existing **My Stats** screen (personal bests / streak) as the player's progress surface. Replace the leaderboard stub with an honest "coming soon" state — no fake data.

### Explicitly out of scope (deferred to v1.2 — see §11)
- Any backend, API, or database. Railway hosting. Hosting/domain changes.
- **Real leaderboard** (needs server to read other players' scores).
- **True percentile benchmark** ("better than X% of auditors") — needs server data. *No seeded/fake percentile in v1.1.*
- **Email capture** and any HubSpot / GTM pipeline tie-in.
- Accounts / login / auth.

### Out of scope (not planned)
- New puzzle content, new error types, or changes to daily-puzzle generation.
- Changes to the scoring formula (+100 correct / −50 false flag / time bonus / streak multiplier) — unchanged from v1.0.
- Native app, push notifications.

---

## 3. Inputs

| Input | Source | Notes |
|---|---|---|
| Daily puzzle | Existing client-side deterministic generator (v1.0) | Unchanged. Same puzzle for everyone, keyed by date/puzzle number. |
| Player identity | First name (already captured) | No email in v1.1. |
| Player history | `localStorage` (streak, completions, best score, onboarded flag) | Source of truth for all of v1.1. No server. |

---

## 4. Outputs

| Output | Destination | Format |
|---|---|---|
| Onboarded flag | `localStorage` (`spotcheck_onboarded`) | Set on practice completion/skip |
| Share text | Clipboard / native share | v1.0 emoji grid + grade + challenge framing + URL |
| Stats display | My Stats screen | Current streak, best streak, best score (all existing local data) |

**Sample share text (challenge variant):**
```
SPOT CHECK #173 — 1,240 pts (SENIOR AUDITOR) 🔥 6-day streak
🟢🟢🟢⚫ 🟢🟢🟢 🟢🟢⚫ 🟢🟢🟢 🟢
Think you know the floor better? gather.ai/spotcheck
```

---

## 5. System Components

| Component | Tool | New / Changed |
|---|---|---|
| Game client | Single `index.html` (vanilla JS/CSS, Canvas 2D) | Changed — adds P1/P2/P3 UI |
| Hosting | **Unchanged** (current static host) | — |
| Backend | **None in v1.1** | Deferred to v1.2 |

No new credentials required (see §5a).

---

## 5a. Credentials Required

```
## Credentials Required
- (none — v1.1 is fully client-side)

## Already Available
- [x] Game source — index.html in this repo
```

---

## 6. Logic / Rules

### P1 — Onboarding

**6.1 First-visit detection.** IF `localStorage` has no `spotcheck_onboarded` flag THEN route first visit into the practice round before the title's normal flow. Set the flag on completion or skip. Always re-runnable via "How to Play" → "Try a practice bay."

**6.2 Practice round (un-timed, scripted).**
- Uses a fixed, hand-authored mini-bay (not the daily puzzle) — small grid, 1–2 obvious errors.
- Guided callouts, step-sequenced:
  1. "This is the **manifest** — what *should* be in each slot." (highlight Reference panel)
  2. "This is the **floor** — what the drone actually counted." (highlight grid)
  3. "Slot A-12 should hold **24 units**. The floor shows **40**. That's a miscount — **tap it**." (wait for correct tap; gently reject wrong taps, no penalty)
  4. "Found it. Now flag any others, then hit **SUBMIT**." (let player complete)
- No timer, no score penalty. Tap-to-skip at every step ("Skip — I've got it").

**6.3 Persistent in-game labels.** Reference panel header reads **"WHAT IT SHOULD BE — WMS Manifest"**; grid area labeled **"WHAT'S ON THE FLOOR."** Persist in all real rounds (both modes).

**6.4 Error-type definitions.** Each of the six error types (Wrong Count, Wrong Slot, FIFO Violation, Phantom Inventory, Weight Violation, Zone Violation) gets a one-line plain-English definition, shown on the How-To screen (expand/tooltip per tag) and appended to Easy mode's "Look for:" briefing hint.
Example: *"FIFO Violation — newer stock staged in front of older stock that should ship first."*

### P2 — Win clarity

**6.5 Stated objective per round.** Pre-round briefing and in-game top bar show that round's error count, e.g. **"3 discrepancies hidden — find them all, don't flag clean bays."** (Source: `round.errorCells.length`.) Shown in *both* modes — it's a goal, not a hint (does not reveal which cells or what type).

**6.6 Visible grade ladder.** The existing grade ladder (`PERFECT AUDIT`, `SENIOR AUDITOR`, `PASSING GRADE`, `NEEDS TRAINING`, `SEE YOUR SUPERVISOR`, keyed off errors-found thresholds) is surfaced (a) on the title/How-To as "what you're playing for," and (b) as a **live target during play** — a compact "Current grade: …" indicator that updates as rounds resolve. Exact placement is a design call; requirement is the player can see the tier they're tracking toward *before* the end screen.

### P3 — Replay (Wordle model, no backend)

**6.7 Streak loop.** Reuse v1.0 streak logic (count, freeze-at-7). No changes to the mechanic; surface it more prominently (see §6.8).

**6.8 Daily pull (end screen).**
- **Streak-risk nudge:** IF active streak THEN "🔥 {n}-day streak — come back tomorrow to keep it." Surface freeze status when streak ≥ 7 and a freeze is available.
- **Tomorrow teaser:** "Next audit drops in {countdown to local midnight}."

**6.9 Challenge-a-colleague share.** Add a share variant framed as a challenge ("I scored {score} on today's Spot Check ({grade}). Think you know the floor better? {URL}"). Keep the v1.0 spoiler-free emoji grid. This is the GTM lever — a prospect forwarding it to a peer is the entire top-of-funnel distribution mechanism in the no-backend model.

**6.10 Leaderboard placeholder (honest).** Replace `alert('coming soon')` with a proper screen that states a leaderboard is coming, and points the player to **My Stats** + share in the meantime. **No invented scores or names.**

---

## 7. Edge Cases & Guardrails

- **First-visit flag corruption / cleared storage:** Re-runs onboarding; acceptable (better than skipping it).
- **Player skips practice immediately:** Allowed; flag still set so they aren't re-forced. Practice reachable from menu.
- **Easy vs Hard:** Easy remains unscored (matches v1.0). Practice round is neither scored nor streak-affecting.
- **Repeat play same day:** Unchanged from v1.0 (first completion stands; "already played today").
- **Share API unavailable (desktop / unsupported browser):** Fall back to clipboard copy (existing v1.0 behavior).
- **Grade ladder live indicator:** Must never reveal which cells are errors — only the tier tracked, derived from already-resolved rounds.
- **No misleading social claims:** Because there's no backend, the UI must not imply real peer comparison anywhere (no "X% of players," no fake ranks).

---

## 8. Success Criteria

Falsifiable checks (full matrix → `demo-review.md`):

1. **Onboarding forced:** A cleared-storage client is routed into the practice round, can complete it by tapping the scripted error, then reaches the real game. The flag prevents re-forcing on reload.
2. **Onboarding replayable:** Practice is reachable from the menu after onboarding.
3. **In-game labels:** Both real modes display "WHAT IT SHOULD BE" and "WHAT'S ON THE FLOOR."
4. **Error definitions:** All six error types show a plain-English definition on the How-To screen.
5. **Round objective:** Each round states its discrepancy count before/during play without revealing which cells.
6. **Live grade:** The current/target grade is visible during play, not only on the end card.
7. **Streak nudge:** End screen shows streak status + freeze state when applicable.
8. **Countdown:** End screen shows a live countdown to the next puzzle.
9. **Challenge share:** Share produces challenge-framed text + URL + spoiler-free grid; falls back to clipboard where native share is unavailable.
10. **No fake social data:** No percentile, rank, or invented player appears anywhere in the build.
11. **Static-only:** The build runs from a single `index.html` with no network calls to any game backend.

---

## 9. Open Questions (resolve before/at build)

1. **Live-grade placement:** Exact UI treatment of the in-play grade indicator (§6.6) — design call.
2. **Practice bay content:** Confirm the scripted mini-bay (which error type to teach — recommend Wrong Count as the most universally obvious).
3. **All-time stats framing:** My Stats currently shows current/best streak + best score — confirm that's the full set we promote, or add "puzzles played / win-rate."

---

## 10. Next Artifact

On spec approval → `tdd-plan.md`. Anticipated phases (all client-side):
1. P1 — onboarding: practice round + first-visit routing + flag.
2. P1 — persistent labels + error-type definitions.
3. P2 — per-round objective display.
4. P2 — live grade indicator + grade ladder on title/How-To.
5. P3 — end-screen streak nudge + next-puzzle countdown.
6. P3 — challenge share variant + honest leaderboard placeholder.
7. Final integration test (full first-time playthrough, cleared storage).

---

## 11. Deferred to v1.2 (requires backend)

Captured here so the v1.1 build leaves clean seams for it:
- **Railway hosting + scores API + datastore.**
- **Real daily / all-time leaderboard** (first-name based).
- **True percentile benchmark** on the end card ("better than X% of auditors today"), with a "first player of the day" fallback.
- **Optional email capture** + potential HubSpot/GTM follow-up.
- **Domain mapping** (path vs subdomain) and `localStorage` migration continuity.

v1.1 design note: keep score-submission and benchmark hooks isolated (e.g., a single `submitScore()` / `getBenchmark()` no-op stub) so v1.2 can swap in real calls without reworking the end-screen UI.
