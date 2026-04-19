# SPOT CHECK — Game Design Specification

**Version:** 1.0
**Date:** April 19, 2026
**Author:** Xavier (GTM Engineering, Gather AI) + Claude
**Status:** Draft — pending team review

---

## 1. Overview

**What it is:** A daily warehouse audit puzzle game. Five rounds, ~90 seconds total. The player reviews warehouse bay snapshots and taps the inventory errors before time runs out. One puzzle per day — same puzzle for everyone. Runs in the browser as a single HTML file. Zero learning curve: if you've ever worked in a warehouse, you already know how to play.

**What it is not:** A Gather AI product demo. The game is fun-first. Warehouse domain expertise is the skill being tested — not reflexes, not dexterity. The only Gather AI touchpoint is a non-intrusive post-game insight card.

**Design thesis:** Domain expertise as a competitive flex. People who run warehouses for a living should score higher than people who don't — and they should want to prove it.

**Inspiration models:**
- **Wordle** — one daily puzzle, same for everyone, shareable emoji grid, streak retention
- **NYT Connections** — pattern recognition that rewards domain knowledge
- **Geoguessr** — expertise-based scoring with social comparison

**Target audience:** Same ICP as Gather AI. Directors of CI, VPs of Operations, Inventory Control Managers, Automation Engineers, Financial Sponsors — warehouse professionals who will *recognize* these errors because they've seen them on the floor. Age range 35-65. Not gamers. Playing on a phone during a meeting lull or on desktop between emails.

**Distribution:** Standalone URL (e.g., `gather.ai/spotcheck`), embeddable on gather.ai, shareable via LinkedIn/Slack/email. No install, no login required to play. Mobile-first design — works on desktop but optimized for phone.

**Tech stack:** Single HTML file, vanilla JS, CSS. Canvas 2D for puzzle rendering. No Three.js, no frameworks, no build tools. Google Fonts via CDN. Total file size target: < 100KB.

---

## 2. Controls

**The whole game is tap/click.** That's it.

| Platform | Action |
|----------|--------|
| Desktop | Click on a location to flag it as an error |
| Mobile | Tap on a location to flag it as an error |

No keyboard required. No WASD. No hotkeys. A VP of Operations in a Suburban picking up their phone can play this immediately.

**Tap behavior:**
- First tap on a location = flag it (yellow highlight, small "!" icon)
- Second tap on a flagged location = unflag it (changed your mind)
- Tap "SUBMIT" when done with the round (or auto-submit when timer expires)

---

## 3. Player Experience — Full Walkthrough

### 3.1 Title Screen

**Visual:** Clean, bright industrial design. White/light gray background with warehouse yellow (#f5c518) accents. A simplified warehouse rack illustration as the background graphic — think blueprint/schematic style, not photorealistic.

**Logo:** "SPOT CHECK" in bold industrial font. Subtitle: *"Find the errors. Beat the clock. Prove you know the floor."*

**Below the logo:**
- Daily puzzle number and date: `#127 — Apr 19, 2026`
- Streak indicator (if returning player): `12-day streak`

**Menu options:**
- **START TODAY'S AUDIT** — begins the daily puzzle
- **HOW TO PLAY** — single-screen rules (4 bullet points max)
- **MY STATS** — personal history, streak, best scores
- **LEADERBOARD** — daily and all-time rankings

**No ambient audio.** This is a quick puzzle, not an atmosphere piece. Subtle tap/feedback sounds only.

### 3.2 How to Play (One Screen)

Shown on first visit, accessible from menu anytime. Maximum simplicity:

```
HOW TO PLAY

You're the night auditor. Each round shows a warehouse bay
with inventory data. Some locations have errors.

  1. TAP locations you think have errors
  2. Hit SUBMIT (or let the timer run)
  3. Correct finds = points. Wrong flags = penalty.
  4. Five rounds. One daily puzzle. Beat your streak.

Error types: wrong counts, misplaced SKUs, FIFO violations,
phantom inventory, safety violations, and more.

The better you know warehouses, the faster you'll spot them.
```

### 3.3 Pre-Round Briefing (2 seconds, auto-advance)

Before each round, a quick one-line briefing appears for 2 seconds:

| Round | Briefing Text |
|-------|--------------|
| 1 | `ROUND 1 — Receiving Dock: Verify the inbound against the manifest.` |
| 2 | `ROUND 2 — Pick Face: Check the forward pick locations.` |
| 3 | `ROUND 3 — Bulk Storage: Audit the reserve inventory.` |
| 4 | `ROUND 4 — Staging Lane: Validate outbound orders.` |
| 5 | `ROUND 5 — Full Floor: The auditor is here. Find everything.` |

### 3.4 Gameplay — Core Loop

#### 3.4.1 The Puzzle Grid

Each round displays a **warehouse bay view** — a 2D grid representing inventory locations in a section of racking.

**Grid layout:**
- Rounds 1-2: 4 columns x 3 rows (12 locations)
- Rounds 3-4: 5 columns x 4 rows (20 locations)
- Round 5: 6 columns x 4 rows (24 locations)

**Each grid cell (location) shows:**
- **Bay/slot ID** (e.g., `A-12-03`) — top-left, small, gray
- **SKU code** (e.g., `SKU-7741`) — center-top, medium
- **Quantity** (e.g., `QTY: 48`) — center, large bold number
- **Visual indicator** — a simple icon/color badge showing item type:
  - Blue snowflake = cold chain / frozen
  - Orange flame = hazmat
  - Green leaf = perishable / date-sensitive
  - Gray box = standard dry goods
  - Purple diamond = high-value
- **Additional data** (context-dependent per round type):
  - Date code (e.g., `EXP: 2026-03-15`) for perishable items
  - Weight indicator (e.g., `2,400 lbs`) for heavy items
  - Lot code (e.g., `LOT: 8A-224`) for quality-tracked items

**Visual design of each cell:**
- Looks like a physical bin label or rack placard
- White card with subtle shadow
- Status indicators are small colored badges, not overwhelming
- On tap: yellow border + "!" icon = flagged
- Clean cells that resolve correctly: green border + checkmark
- Errors correctly found: green border + checkmark + "FOUND" label
- Errors missed: red border + "MISSED" label
- False flags (you flagged a clean cell): orange border + "CLEAN" label

#### 3.4.2 The Reference Panel

Along the top or side of the screen (depending on orientation), a **reference panel** provides the "expected" state — this is what the WMS *says* should be true. The puzzle is: does reality match the system?

**Reference data varies by round type:**

| Round Type | Reference Panel Shows |
|------------|----------------------|
| **Receiving** | Inbound manifest: expected SKUs, quantities, and lot codes for this delivery |
| **Pick Face** | WMS slot plan: which SKU belongs in which location, expected quantities |
| **Bulk Storage** | Inventory snapshot: system-of-record counts and locations as of last cycle count |
| **Staging Lane** | Outbound order: what should be staged, quantities, ship-to labels |
| **Full Floor** | Mixed — some cells reference a manifest, some a slot plan, some a count sheet |

The reference panel is always visible. The player compares it against what each grid cell shows. Mismatches are errors.

#### 3.4.3 Error Types

Each error rewards warehouse domain knowledge. A civilian could eventually spot them by comparing data carefully. A warehouse professional sees them instantly.

| Error Type | What It Looks Like | Why Warehouse Pros Spot It Fast |
|------------|-------------------|-------------------------------|
| **Count Mismatch** | Grid cell says QTY: 36, reference says QTY: 48 | They reconcile counts every day. Obvious. |
| **Wrong Slot** | SKU-7741 is in location B-04-02, but reference says B-04-02 should have SKU-3319 | Putaway errors are their #1 headache. |
| **FIFO Violation** | Location has EXP: 2026-06 behind EXP: 2026-03. Newer stock in front of older stock. | First-In-First-Out is drilled into every warehouse worker from day one. |
| **Phantom Inventory** | Grid cell shows QTY: 24, but the cell has a subtle "EMPTY" visual cue (lighter background, no pallet icon) | Ghost inventory in the WMS is a constant battle. |
| **Weight/Safety Violation** | A 2,400 lb pallet on rack level 4 (top), or hazmat next to food | OSHA and insurance — they think about this constantly. |
| **Zone Violation** | Frozen item (snowflake badge) in an ambient storage bay (no temp indicator) | Cold chain integrity is non-negotiable in food/pharma. |
| **Lot Code Split** | Two cells have same SKU but different lot codes, and the reference says they should be consolidated | Quality hold and recall tracking depends on lot segregation. |
| **Damaged Hold** | Cell shows a subtle damage indicator (small crack icon) but status shows "AVAILABLE" not "HOLD" | Shipping damaged goods = chargebacks and customer complaints. |
| **Short Ship** | Staging lane shows QTY: 36 staged, but the order calls for QTY: 48 | Shorts get caught at the dock... or they become customer escalations. |
| **Duplicate LPN** | Two cells have the same LPN (license plate number), which is impossible — one is a system error | LPN collisions break WMS logic. Ops teams know this. |

**Error distribution per round:**

| Round | Total Locations | Errors Hidden | Error Complexity |
|-------|----------------|---------------|-----------------|
| 1 | 12 | 2 | Simple (count mismatch, wrong slot) |
| 2 | 12 | 2 | Simple-medium (FIFO, phantom inventory) |
| 3 | 20 | 3 | Medium (zone violations, weight issues, lot splits) |
| 4 | 20 | 3 | Medium-hard (short ships, damaged holds, duplicate LPNs) |
| 5 | 24 | 4 | Mixed — any error type, all at once |

Total errors per daily puzzle: **14**

#### 3.4.4 Timer

Each round has a countdown timer. The timer is generous — the goal is speed-based scoring, not elimination.

| Round | Timer |
|-------|-------|
| 1 | 20 seconds |
| 2 | 20 seconds |
| 3 | 25 seconds |
| 4 | 25 seconds |
| 5 | 30 seconds |

**Total max play time: 120 seconds (2 minutes).**

When the timer runs out, the round auto-submits whatever the player has flagged. Unflagged errors count as misses. The game does NOT end on timer expiry — you just lose the time bonus for that round.

Timer display: a thin progress bar at the top of the screen that drains left-to-right. Color shifts yellow → orange → red in the last 5 seconds. No large distracting clock — keep it peripheral.

#### 3.4.5 Scoring

**Per round:**
- **Correct flag** (found a real error): +100 points
- **Time bonus**: +5 points per second remaining when submitted (rewards speed)
- **False flag** (flagged a clean location): -50 points (discourages spam-clicking)
- **Missed error** (didn't flag a real error): +0 (no penalty beyond lost points)
- **Perfect round** (all errors found, zero false flags): +50 bonus

**Per game:**
- Sum of all 5 rounds
- **Perfect game bonus** (all 14 errors found, zero false flags): +200 bonus
- **Streak multiplier**: current streak day count as percentage bonus on total score
  - Day 1-4: no bonus
  - Day 5-9: +5%
  - Day 10-19: +10%
  - Day 20-49: +15%
  - Day 50+: +20%

**Score range:**
- Perfect game, fast, long streak: ~2,000-2,500 points
- Good game (12/14 found, few false flags): ~1,200-1,600 points
- Casual play (8/14 found): ~700-1,000 points
- Button-mashing (flag everything): negative score due to false flag penalties

This scoring system rewards precision and speed — exactly the skills valued in warehouse operations.

### 3.5 Round Resolution

After the player submits (or timer expires), the grid animates to show results:

1. **Correctly flagged errors** light up green with a satisfying "ding" sound and "FOUND" label (0.3s stagger per cell)
2. **False flags** flash orange briefly with a soft "bonk" and "CLEAN" label
3. **Missed errors** pulse red with "MISSED" label — the cell briefly highlights the actual discrepancy text (e.g., the mismatched number turns red)
4. **Clean cells** (correctly left alone) fade slightly — no animation, no noise

**Round score card** appears for 2 seconds:
```
ROUND 3 — BULK STORAGE
Errors found: 2/3
False flags: 0
Time bonus: +45
Round score: 245
```

Auto-advances to next round (or tap to skip).

### 3.6 End Screen — Audit Report

**Visual:** Clean white card with a "report" feel. Similar to an actual cycle count summary.

**Layout:**

```
SPOT CHECK #127 — Apr 19, 2026
DC-42 Memphis

AUDIT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━
Errors found:    12 / 14
False flags:     1
Accuracy:        92%
Total score:     1,487
Streak:          13 days

ROUND BREAKDOWN
R1  ●●       2/2  +210     ← green dots = found, gray = missed
R2  ●○       1/2  +145
R3  ●●●      3/3  +295
R4  ●●○      2/3  +190
R5  ●●●○     3/4  +347
              bonus +100
              streak +148 (10%)
━━━━━━━━━━━━━━━━━━━━━━━━

[SHARE]  [LEADERBOARD]  [MY STATS]
```

**Accuracy rating (displayed as a title above the score):**

| Accuracy | Title |
|----------|-------|
| 100% (14/14, 0 false flags) | PERFECT AUDIT |
| 90%+ errors found, ≤1 false flag | SENIOR AUDITOR |
| 75%+ errors found | PASSING GRADE |
| 50%+ errors found | NEEDS TRAINING |
| <50% errors found | SEE YOUR SUPERVISOR |

**Manager one-liners (below the title):**

| Title | Line |
|-------|------|
| PERFECT AUDIT | "Zero variance. The auditors have nothing to do. You're expensive but worth it." |
| SENIOR AUDITOR | "One of the cleanest counts we've had. Corporate is impressed." |
| PASSING GRADE | "Good enough for quarterly. But we both know there's room to improve." |
| NEEDS TRAINING | "Let's get you shadowing the A-team on the next cycle count." |
| SEE YOUR SUPERVISOR | "I'm going to pretend I didn't see this report." |

### 3.7 Gather AI Insight Card

**Trigger:** Appears 2 seconds after the audit report renders. Slides up from the bottom — does NOT cover the report. Dismissable.

**Design principle:** Contextual, not salesy. It should feel like a relevant data point, not an ad.

**Content varies based on performance:**

| Performance | Card Text |
|-------------|-----------|
| Found 12+ errors | "You found [X] errors in [Y] seconds. A Gather AI drone audits 200 bays in that time — every day, with 99%+ accuracy." |
| Found 8-11 errors | "You missed [X] errors. In a real warehouse, each missed discrepancy costs an average of $250 in downstream impact. Gather AI catches them autonomously." |
| Found <8 errors | "[X] errors slipped through. Multiply that across 50 facilities and a million SKUs. That's the problem Gather AI solves." |
| Perfect game | "Perfect audit. Impressive. Now imagine doing that across every bay, every night, without a human. That's what we built." |

**CTA button:** "See How It Works" → links to `gather.ai/demo?utm_source=spotcheck&utm_medium=game&utm_campaign=post-game`

**Dismiss:** Small "x" button. Once dismissed, does not reappear until next day's puzzle.

---

## 4. Daily Puzzle System

### 4.1 Seed Generation

Each day's puzzle is deterministically generated from the date string. No server required.

**Seed formula:** `spotcheck-YYYY-MM-DD` → hashed to a 32-bit integer → feeds a seeded PRNG (mulberry32).

**The seed determines:**
- Which error types appear in each round
- Which cells contain errors
- All SKU codes, quantities, bay IDs, dates, and lot codes
- The facility name (cosmetic)
- Reference panel data

**Same seed = same puzzle for everyone.** This is critical for fair leaderboard comparison and the "did you get #127 today?" social mechanic.

### 4.2 Puzzle Number

Puzzles are numbered sequentially from a launch date. Example: if launch date is May 1, 2026, then May 1 = #1, May 2 = #2, etc. The puzzle number appears on the title screen and in the share card.

### 4.3 One Play Per Day

The player gets **one attempt** at each day's puzzle. After completion, the puzzle is locked until tomorrow. This creates:
- **Scarcity** — you can't brute-force a better score
- **Stakes** — each round matters because you can't retry
- **Water-cooler conversation** — "how'd you do on today's Spot Check?" only works if everyone has one shot

**Implementation:** Store completion flag in localStorage: `spotcheck_completed_YYYY-MM-DD: true`. Also store the score for that day.

**Edge case:** If the player closes the browser mid-puzzle, they can resume where they left off (round state saved to localStorage). But once all 5 rounds are submitted, it's locked.

### 4.4 Yesterday's Puzzle

After completing today's puzzle, the player can access "YESTERDAY'S AUDIT" — a replay of the previous day's puzzle for practice. Yesterday's score does NOT count toward the leaderboard or streak. Labeled clearly as "PRACTICE — does not count."

This serves players who want more than one play per day without undermining the daily competition mechanic.

---

## 5. Procedural Generation

### 5.1 Data Generation

The PRNG generates realistic warehouse data for each cell:

**Bay IDs:** Format `[Letter]-[Row]-[Level]`. Letters A-F for columns, rows 01-24, levels 01-04. Generated sequentially so they look like a real racking layout.

**SKU codes:** Format `SKU-[4 digits]`. Drawn from a pool of ~200 pre-defined SKUs so they feel consistent across rounds (you might see the same SKU in different locations — that's realistic).

**Quantities:** Standard pallet quantities that warehouse people recognize:
- Full pallet: 48, 60, 72, 96, 120 (common case-pack multiples)
- Partial: anything below the standard quantity for that SKU
- The PRNG selects a "standard qty" for each SKU, then errors show deviations from it

**Dates:** Generated relative to the puzzle date. Expiration dates range from 2 weeks ago (expired!) to 6 months out. Lot codes are alphanumeric (e.g., `LOT: 4B-118`).

**Weights:** Realistic range: 800 lbs (light pallets) to 2,800 lbs (heavy). Weight violations trigger on levels 3+ with weight > 2,000 lbs.

### 5.2 Error Placement

The PRNG places errors with constraints:
- Errors are never in the same cell position two days in a row (prevents positional pattern learning)
- Each round must have at least one "obvious" error and one "subtle" error
- Round 5 errors are drawn from the full type pool with no duplicates
- No more than 2 errors adjacent to each other (prevents "click the cluster" strategy)

### 5.3 Reference Panel Generation

The reference panel is generated to be **internally consistent** — the "expected" state makes logical sense. Errors are introduced by making specific cells deviate from the reference, not by making the reference wrong.

This is critical: the reference panel should always feel like a real WMS printout. If the reference says location A-04-02 should have SKU-3319 at QTY: 48, that should feel like a plausible warehouse state.

---

## 6. Visual Design

### 6.1 Aesthetic Direction

**Tone:** Clean, professional, industrial. This is not a dark moody game — it's a quick sharp puzzle that feels like a well-designed internal tool. Think "Notion meets a warehouse audit sheet."

**Palette:**
- Background: Off-white (#f8f7f4)
- Cards/cells: White (#ffffff) with subtle shadow
- Primary accent: Warehouse yellow (#f5c518)
- Secondary accent: Industrial blue (#1a5276)
- Success: Green (#27ae60)
- Error/miss: Red (#e74c3c)
- False flag: Orange (#f39c12)
- Text: Dark gray (#2c3e50)
- Muted text: (#7f8c8d)
- Reference panel: Light blue-gray (#eaf2f8) background

### 6.2 Typography

- **Title/Logo:** `"Black Ops One"` (shared with Aisle Unknown for brand continuity)
- **UI/Body:** `"Inter"` or `"Share Tech Mono"` — clean, legible at small sizes
- **Data in grid cells:** Monospace (`"Share Tech Mono"`) — makes numbers and codes scannable, like a real label
- **Score/results:** `"Inter"` bold for numbers, mono for breakdowns

### 6.3 Grid Cell Design

Each cell is a card that looks like a warehouse location label:

```
┌──────────────────────┐
│ A-12-03         [❄️]  │  ← Bay ID + item type badge
│                      │
│ SKU-7741             │  ← SKU code
│ QTY: 48             │  ← Quantity (large, bold)
│                      │
│ EXP: 2026-06-15     │  ← Date (if applicable)
│ LOT: 4B-118         │  ← Lot code (if applicable)
│ 1,800 lbs           │  ← Weight (if applicable)
└──────────────────────┘
```

**States:**
- Default: white card, subtle border
- Hovered (desktop): slight scale-up (1.02x), shadow deepens
- Flagged: yellow border (3px), small "!" badge in top-right corner
- Resolved — correct find: green border, checkmark icon, "FOUND" label
- Resolved — missed: red border, red highlight on the discrepancy data, "MISSED" label
- Resolved — false flag: orange border, "CLEAN" label
- Resolved — clean (correctly left alone): no border change, slight fade

### 6.4 Mobile Layout

**Portrait orientation (primary):**
```
┌─────────────────────┐
│  SPOT CHECK #127    │
│  Round 3/5          │
│  ▓▓▓▓▓▓░░░░  15s   │  ← timer bar
├─────────────────────┤
│ REFERENCE: WMS Count│
│ A-12: 48  B-12: 60 │  ← scrollable reference
│ C-12: 36  D-12: 72 │
├─────────────────────┤
│ ┌───┐ ┌───┐ ┌───┐  │
│ │   │ │   │ │   │  │
│ └───┘ └───┘ └───┘  │
│ ┌───┐ ┌───┐ ┌───┐  │  ← tappable grid
│ │   │ │   │ │   │  │
│ └───┘ └───┘ └───┘  │
│ ┌───┐ ┌───┐ ┌───┐  │
│ │   │ │   │ │   │  │
│ └───┘ └───┘ └───┘  │
├─────────────────────┤
│     [SUBMIT]        │
└─────────────────────┘
```

**Desktop layout:** Reference panel on the left (sidebar), grid centered, timer at top. More horizontal space — cells can be larger and show more data.

**Touch targets:** All tappable cells minimum 48x48px. SUBMIT button is 56px tall, full width. This is non-negotiable for the 55-year-old VP checking this on their iPhone.

### 6.5 Animations

Keep animations tight and informative. No gratuitous motion.

- **Cell tap:** Quick scale pulse (1.0 → 1.05 → 1.0, 150ms) + yellow border fade-in
- **Round resolve:** Cells animate in sequence (0.15s stagger), left-to-right, top-to-bottom
- **Correct find:** Green sweep from left + subtle confetti particles (3-4 particles, not a celebration cannon)
- **Missed error:** Red pulse (2 pulses, 0.3s each) + the discrepant data value blinks
- **Score tally:** Numbers count up (not appear instantly) — 0.5s per line item
- **Streak counter:** Flame icon does a small bounce on increment

---

## 7. Audio Design

Minimal. Professional. Not "gamey."

| Element | Sound | Priority |
|---------|-------|----------|
| Cell tap (flag) | Short, satisfying click — like a clipboard pen click | High |
| Cell tap (unflag) | Softer reverse click | High |
| Submit button | Firm "stamp" sound — like a rubber stamp on paper | High |
| Correct find | Quick ascending two-note chime (same as Aisle Unknown scan) | High |
| Missed error | Soft low tone — not harsh, just informative | High |
| False flag | Quick double-tap "nah" tone | Medium |
| Perfect round | Three-note ascending chime | Medium |
| Timer warning (5s) | Subtle ticking, increases tempo | Medium |
| Perfect game | Triumphant 4-note chord | High |
| Streak milestone (10, 25, 50) | Special chime variant | Low |

**All audio synthesized via Web Audio API.** No external audio files.

**Volume:** Very quiet by default. Mute toggle visible on title screen. Remembers preference.

---

## 8. Leaderboard & Social

### 8.1 Leaderboard Structure

Three views:

| Tab | Content | Storage |
|-----|---------|---------|
| **TODAY** | Top scores for today's puzzle (#127) | Shared (`shared: true`) — via artifact storage if available, otherwise Firebase/Supabase |
| **ALL TIME** | Top 100 highest single-day scores ever | Shared |
| **MY HISTORY** | Player's daily scores, rolling 30 days | Local (localStorage) |

### 8.2 Name Entry

On first play, prompt for a display name (3-16 characters) and optional company name (for bragging rights). Format on leaderboard: `Xavier S. — Gather AI`.

No email required. No login. Just a name.

**Optional email capture (below name entry):** "Get reminded when tomorrow's puzzle drops?" — opt-in email field. Not required. If provided, send to HubSpot. UTM: `utm_source=spotcheck&utm_medium=game&utm_campaign=daily-reminder`

### 8.3 Share Card

**This is the primary viral mechanic.** After completing the daily puzzle, the SHARE button generates a text block optimized for LinkedIn/Slack:

```
SPOT CHECK #127 — SENIOR AUDITOR
🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢⚫⚫
12/14 found | 1,487 pts | 13-day streak 🔥
gather.ai/spotcheck
```

**Emoji grid explanation:**
- Each position represents one of the 14 errors across all 5 rounds
- Green circle = found it
- Black circle = missed it
- The grid is always 14 characters — same length every day, visually consistent

**Why this works:**
- It's mysterious to non-players ("what is this?") — drives curiosity clicks
- It's instantly parseable to players ("they missed 2, I only missed 1")
- The streak creates FOMO ("they're on day 13? I need to start")
- The URL is right there

**Share targets:** Copy to clipboard (primary), with native share sheet on mobile (navigator.share API).

### 8.4 Company Leaderboard (v1.1)

If the player entered a company name, they can see a "MY COMPANY" tab on the leaderboard — scores from other players who entered the same company name. This drives internal viral loops: "the ops team at DC Memphis is competing against corporate."

Implementation: Company name is normalized (lowercase, trimmed) and used as a shared storage key prefix. No admin/verification — honor system is fine for a game.

---

## 9. Streak & Retention Mechanics

### 9.1 Daily Streak

Playing (completing all 5 rounds) every day builds a streak counter. The streak is the primary retention mechanic.

**Streak display:**
- Shown on title screen: `12-day streak 🔥`
- Shown on share card
- Streak milestone animations at 7, 14, 30, 50, 100 days

**Streak protection:** Missing one day does NOT immediately break the streak. The player gets a "streak freeze" — one free miss per 7-day streak. So:
- Days 1-6: miss a day = streak resets
- Day 7+: one free miss, then resets
- Day 14+: still one free miss (doesn't stack)

This is generous enough to retain people who travel or have a busy day, without removing the stakes entirely.

### 9.2 Returning Player Experience

When a returning player opens the game:
- If they haven't played today: title screen shows "TODAY'S AUDIT READY" with streak counter
- If they already played today: shows today's score, "COME BACK TOMORROW — new puzzle at midnight"
- If they missed yesterday and used their freeze: "STREAK SAVED — don't miss again!"
- If their streak broke: "STREAK: 0 — Start a new one today"

### 9.3 Weekly Recap (v1.1)

Every Sunday, a "WEEKLY AUDIT REPORT" screen shows:
- Days played this week
- Average score
- Best round of the week
- Errors most commonly missed (pattern feedback)
- Comparison to global average: "You scored 18% above the average auditor this week"

---

## 10. Difficulty Scaling

### 10.1 Within Each Puzzle

Difficulty increases across rounds as described in §3.4.3 — more cells, more errors, more complex error types.

### 10.2 Across Days

The daily puzzle generator includes a **difficulty tier** that cycles weekly:

| Day | Difficulty | Changes |
|-----|-----------|---------|
| Monday | Easy | Error types are simpler (count mismatches, wrong slots). Good for new players. |
| Tuesday | Easy-Medium | Introduces FIFO and phantom inventory errors. |
| Wednesday | Medium | Full error type pool for rounds 3-5. |
| Thursday | Medium-Hard | Reference panel has more data to parse. Timer is 3s shorter per round. |
| Friday | Hard | Round 5 has 5 errors instead of 4. One "red herring" — a cell that looks wrong but is actually correct. |
| Saturday | Challenge | All rounds use the full 6x4 grid. Timer is 5s shorter per round. |
| Sunday | Expert | "Lights out" mode — reference panel is only visible for the first 10 seconds of each round, then hides. You must remember it. |

This gives players a reason to come back every day AND creates predictable difficulty conversation: "Friday's puzzle was brutal."

---

## 11. Lead Capture Strategy

**Philosophy:** Identical to Aisle Unknown — no gates, no email walls. The game earns attention first.

**Mechanisms (ranked by priority):**

1. **Post-game Gather AI insight card** (§3.7) — contextual stat + "See How It Works" CTA. UTM: `utm_source=spotcheck&utm_medium=game&utm_campaign=post-game`

2. **Share card → URL → new player** — the viral loop. Every share includes `gather.ai/spotcheck`. Each new player who arrives and sees the Gather AI card is a potential lead.

3. **Daily reminder email opt-in** (§8.2) — "Get reminded when tomorrow's puzzle drops." Low-friction email capture. Into HubSpot. UTM: `utm_source=spotcheck&utm_medium=game&utm_campaign=daily-reminder`

4. **Company leaderboard → internal spread** — one player at a prospect company brings colleagues. More players = more people seeing the Gather AI card.

5. **LinkedIn organic** — share cards posted by players create organic impressions in the ICP's feed. The warehouse vocabulary in the share card acts as a filter — only relevant people engage.

**Analytics events to track (v1.1):**
- Daily active players
- Completion rate (started vs finished all 5 rounds)
- Share button clicks and share method (clipboard vs native)
- CTA click rate on Gather AI card
- Email opt-in rate
- Returning player rate (day 2, day 7, day 30)
- Average score by day of week (difficulty tuning)
- Most-missed error types (content insight)

---

## 12. Implementation Notes

### 12.1 Build Target
- Single HTML file
- Vanilla JS + CSS
- Canvas 2D for grid rendering (or pure DOM — prototype both, pick whichever is more responsive)
- Google Fonts via CDN (Inter, Share Tech Mono, Black Ops One)
- No Three.js — this is a 2D puzzle game
- No frameworks, no build tools
- Total file size: < 100KB
- Works on: Chrome, Firefox, Safari (desktop + mobile)

### 12.2 Performance Targets
- Instant load (< 1 second on 4G)
- 60fps animations
- Touch response < 16ms
- No layout shift on mobile orientation change

### 12.3 State Management
- All game state in a single JS object (`gameState`)
- Daily puzzle state persisted to localStorage (survives browser close mid-puzzle)
- Score history in localStorage (rolling 30 days)
- Streak counter in localStorage with date tracking
- Display name + company name in localStorage

### 12.4 Mobile Considerations
- Portrait-first layout — landscape is supported but portrait is the primary design target
- All tap targets ≥ 48x48px (WCAG 2.5.5)
- No pinch-zoom on the puzzle grid (viewport meta tag)
- Font sizes: minimum 14px body, 12px for secondary data in grid cells
- Safe area insets for iPhone notch/Dynamic Island
- Grid cells should scroll if they don't fit — never shrink below readable size

### 12.5 Accessibility
- High contrast between text and background (WCAG AA minimum)
- Color is never the only indicator — always paired with icons or text labels
- Screen reader labels on all interactive elements
- Keyboard navigation (Tab through cells, Enter to flag, Tab to Submit)

### 12.6 Scope Prioritization for v1

**Must have (ship-blocking):**
- Core puzzle loop (5 rounds, tap to flag, submit, resolve)
- 6 error types (count mismatch, wrong slot, FIFO, phantom, weight violation, zone violation)
- Per-round timer with auto-submit
- Scoring system
- End screen with audit report
- Share card (copy to clipboard)
- Daily puzzle from date seed (one play per day)
- Personal score history (localStorage)
- Streak counter
- Post-game Gather AI card
- Mobile-first responsive layout
- Audio feedback (synthesized)

**Should have (v1.1):**
- All 10 error types
- Global leaderboard (shared storage or lightweight backend)
- Company leaderboard
- Daily reminder email opt-in
- Weekly recap screen
- Yesterday's puzzle (practice mode)
- Daily difficulty scaling
- Analytics event tracking

**Nice to have (v1.2+):**
- Weekly/monthly tournament mode
- "Challenge a colleague" — generate a link that puts two players on the same puzzle
- Industry-specific puzzle variants (pharma warehouse, cold chain, eComm fulfillment)
- Integration with LinkedIn API for one-click sharing
- Push notifications for daily puzzle (via PWA)
- Offline support (PWA service worker caches next day's puzzle)

---

## 13. Open Items / Questions for Team

| # | Question | Owner | Status |
|---|----------|-------|--------|
| 1 | Hosting URL — `gather.ai/spotcheck` or separate subdomain? | Xavier / Rob | Open |
| 2 | Shared leaderboard backend — artifact storage, Firebase, Supabase, or skip for v1? | Xavier | Open |
| 3 | Demo booking CTA link for Gather AI card — current URL? | Xavier / Marketing | Open |
| 4 | HubSpot form ID for daily reminder email capture | Xavier / RevOps | Open |
| 5 | Analytics tool — GA4, Mixpanel, Amplitude? | Xavier | Open |
| 6 | Launch puzzle number — start at #1 or pick a higher number for perceived maturity? | Xavier | Open |
| 7 | Daily puzzle reset time — midnight UTC, midnight ET, or midnight local? | Xavier | Open |
| 8 | OG image for social sharing — need design asset (1200x630) | Xavier / Design | Open |
| 9 | Cross-link with Aisle Unknown — link from Spot Check to Aisle Unknown and vice versa? | Xavier | Open |

---

## 14. Success Metrics

| Metric | Target | Why This Target | Measurement |
|--------|--------|-----------------|-------------|
| Day 1 → Day 2 return rate | > 40% | Wordle-like daily puzzles see 40-60% D1 retention | Analytics |
| Day 7 return rate | > 20% | If 1 in 5 players comes back for a week, the habit is forming | Analytics |
| Average streak length | > 5 days | Proves the daily mechanic works | localStorage aggregation |
| Share rate | > 15% of completed games | Higher than Aisle Unknown target because sharing is the core loop | Analytics |
| Post-game CTA click rate | > 5% of completed games | Higher than Aisle Unknown because the insight card is more contextual | UTM tracking |
| Email opt-in rate | > 8% of players who set a name | "Remind me tomorrow" is a low-friction ask | HubSpot |
| Plays per day (month 1) | > 100 | Baseline for organic growth | Analytics |
| Plays per day (month 3) | > 500 | Target with viral loop working | Analytics |
| LinkedIn share impressions | > 1,000/month | Visible in ICP feeds | Social monitoring |
| Qualified leads attributed | > 10/quarter | 2x Aisle Unknown target — daily touchpoint = more CTA exposure | HubSpot attribution |

---

## 15. Competitive Advantage: Why This Works for Gather AI Specifically

Most B2B marketing games are generic trivia or product demos in disguise. Spot Check is different because:

1. **The game mechanic IS the problem Gather AI solves.** Finding inventory discrepancies manually is literally what the game asks you to do — and what Gather AI automates. The post-game card doesn't have to explain the connection; the player just lived it.

2. **The scoring validates the prospect's expertise.** A high score means "I know warehouses." That identity reinforcement makes the player an advocate for the game within their network. Nobody shares a product demo; everyone shares something that makes them look smart.

3. **The daily cadence builds a habit touchpoint.** Aisle Unknown is a one-time play. Spot Check is a daily relationship. Even if the player never clicks the CTA, they see the Gather AI name every day for their entire streak. That's brand awareness you can't buy.

4. **The share mechanic targets the exact ICP.** When a VP of Operations posts their Spot Check score on LinkedIn, the people who engage are other warehouse professionals. Organic, targeted top-of-funnel.

5. **The difficulty rewards the right people.** A warehouse professional should consistently outscore a civilian. That exclusivity makes the game feel like an insider thing — "our industry's game." That's the network effect.

---

*End of spec. Ready for implementation.*
