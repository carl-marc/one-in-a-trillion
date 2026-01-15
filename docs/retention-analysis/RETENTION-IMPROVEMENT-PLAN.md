# One in a Trillion - Retention Improvement Plan

**Created:** January 2026
**Status:** In Progress
**Current Phase:** Phase 1 (Implementation)

---

## Executive Summary

Day 1 retention has dropped from ~40% (Aug 2022 / Mar 2023) to ~22% (current). Analysis suggests the primary issues are:

1. **Multiplayer prominence** - "Play | Multiplayer" toggle signals competitive game to collectors
2. **No clear first-session goal** - Players don't know what success looks like
3. **Information overload** - Too many unexplained UI elements for new players
4. **Demotivating progress display** - "Egglopedia 0.41%" feels like failure

The core gameplay (tap to collect rare eggs) is solid. The problem is **new players don't understand what they're doing or why** before they leave.

---

## Key Data Points

| Metric | Value | Notes |
|--------|-------|-------|
| Current D1 Retention | ~22% | Down from 40% in 2022-2023 |
| Retention drop period | Mar 2023 → Jan 2024 | Steady decline over ~10 months |
| Churn point | 50% leave by 5k taps | That's only ~3 minutes of gameplay |
| Daily new players | ~80 | Too low for effective A/B testing |
| Tech stack | Flutter | No game engine |

---

## Current New Player Experience

### What New Players See (< Day 2, not logged in):

```
┌─────────────────────────────────────────────────────┐
│  [Play]  [Multiplayer 👑]                           │  ← Multiplayer visible
├─────────────────────────────────────────────────────┤
│  💎55  💎18  💎5  │ SPEED 1974 │  [ ][ ][ ]        │  ← 3 gems + speed + tripply boxes
│                   │  🔥 2122   │                    │
├─────────────────────────────────────────────────────┤
│  PROGRESS                                           │
│  Taps 👆 3,104                                      │
│  Egglopedia 🌐 0.41%                                │  ← Demotivating percentage
├─────────────────────────────────────────────────────┤
│                                                     │
│              (gameplay area)                        │
│                                                     │
├─────────────────────────────────────────────────────┤
│  [🥚11]  [🔴8034]  [🟢1022]  [≡]                   │  ← Unexplained buttons
└─────────────────────────────────────────────────────┘
```

### Problems Identified:

1. **Multiplayer tab** - Visible immediately, draws attention with crown icon
2. **Tripply boxes** - 3 empty dashed boxes with no explanation
3. **Speed display** - Two numbers (current + daily best) unexplained
4. **Egglopedia 0.41%** - Percentage feels like failure, not progress
5. **Bottom buttons** - Mini-games (11), Instant Yolks (8034), Dubbly Yolks (1022) - all unexplained
6. **No explicit goal** - Player knows stats but not "what should I do?"

---

## Existing Good Features

- First 10k taps have **controlled egg spawning** including guaranteed 1-in-25k rare egg
- Rare egg discovery triggers **popup explaining rarity** and collection progress
- New players get **slightly simplified UI** (no daily stats on day 1)
- **Nail tap tutorial** exists at start explaining basic controls

---

## Game Mechanics Reference

### Core Loop
- Hold finger (nail tap for comfort) on screen
- Taps register automatically as you hold
- Swipe around to collect spawning items
- Items: Eggs (various rarities), Gems (blue/red/yellow), Yolks, Tripply items

### Rarity System
- Common: 1 in 250
- Uncommon: 1 in 500
- Rare: 1 in 5k, 25k, 50k
- Very Rare: 1 in 250k, 500k, 2m
- Legendary: 1 in 5m, 50m, 250m
- Mythic: Up to 1 in a trillion

### Speed System
- Speed = taps per minute
- Collecting items increases speed
- Missing items decreases speed (proportional to current speed)
- Minimum speed: 1500
- Uncapped maximum (can exceed 3000+)

### Tripply System
- 3 boxes that fill when tripply items collected
- Decay bar decreases with each tap
- Collect 3 before decay empties to win prizes
- Creates tension/anticipation for experienced players
- Confusing for new players who don't understand it

### Mini-Games
- **Power**: One of three egg collection games
- **Scramble**: One of three egg collection games
- **Frenzy**: One of three egg collection games
- Each has levels, yolk costs, and own mechanics
- Alternative ways to increase Egglopedia %

### Power-Ups
- **Instant Yolks**: Simulates X taps instantly, spawning all items at once
- **Dubbly Yolks**: Doubles item spawns for X taps

---

## Phased Implementation Plan

### PHASE 1: Reduce Early Confusion (Target: Week 1-2)

| Task | Description | Priority | Effort |
|------|-------------|----------|--------|
| 1.1 | Hide Multiplayer tab for players < 25k taps | Critical | Low |
| 1.2 | Add "Find 5 eggs" first-session goal | Critical | Medium |
| 1.3 | Hide daily best speed for players < 10k taps | High | Low |
| 1.4 | Show "Eggs Found: X" instead of "Egglopedia X%" for < 50k taps | Medium | Low |

**Success Criteria:** D1 retention improves from 22% to 25%+

---

### PHASE 2: Progressive Complexity (Target: Week 3-4)

| Task | Description | Priority | Effort |
|------|-------------|----------|--------|
| 2.1 | Progressive bottom button unlock (hide until relevant) | High | Medium |
| 2.2 | Tripply explainer overlay on first encounter | High | Low |
| 2.3 | Enhanced rare egg celebration (bigger, more exciting) | Medium | Medium |
| 2.4 | Tooltips on bottom buttons on first tap | Medium | Low |

**Success Criteria:** D1 retention reaches 28%+

---

### PHASE 3: Deeper Fixes (Target: Week 5-6, if needed)

| Task | Description | Priority | Effort |
|------|-------------|----------|--------|
| 3.1 | Audit mini-games menu for complexity | Medium | Medium |
| 3.2 | Review speed penalty system for new players | Medium | Medium |
| 3.3 | Consider guided feature tour | Low | High |

---

### PHASE 4: Iterate Based on Data (Week 7+)

- Analyze retention data from Phase 1-3
- Identify remaining drop-off points
- Survey Discord community
- Plan next iteration

---

## Measurement Framework

### Baseline Metrics (Document Before Phase 1)
- D1 Retention: 22%
- D3 Retention: [measure]
- D7 Retention: [measure]
- Median churn point: ~5k taps

### Post-Phase Measurement
- Wait 14 days after each phase release (80 × 14 = 1,120 new players)
- Compare D1 retention to baseline
- Track any changes in churn point

### Rollback Criteria
If D1 retention drops below 20%, immediately rollback changes and investigate.

---

## Technical Notes

- **Platform:** Flutter (no game engine)
- **Key flag:** `player.lifetimeTaps` - used for progressive feature unlocking
- **Session tracking:** Track unique eggs found per session for goal system
- **Storage:** Need to persist `hasSeenFirstGoal`, `hasSeenTripplyExplainer` flags

---

## Questions Answered During Analysis

1. **Where do players leave?** Unknown exactly - could be gameplay screen or menus
2. **Is there early rare egg guarantee?** Yes, 1-in-25k within first 10k taps
3. **What was different in 2022?** No multiplayer panel, possibly simpler overall
4. **Is there a new player flag?** Yes, simplified UI exists for new players
5. **Can we A/B test?** Not effectively with 80 players/day

---

## Links & Resources

- Support: support@hamlab.dev
- Discord: https://discord.gg/swGQAmGtqX
- Website: https://dev.oneinatrillion.fun

---

## Change Log

| Date | Change |
|------|--------|
| Jan 2026 | Initial analysis and Phase 1-4 plan created |

