# Score System - Death-Based Leaderboard & Queen Ant Economy

## Quick Reference

- Score saves ONLY on death (logout = death)
- Kill reward: 100 base + 25% victim score + 10% victim gear value
- Queen model: Resource deposits give score to base-core owner (not depositor)
- High-score players visible on map (bounty targets)
- Hourly deposit caps prevent infinite scaling
- Anti-abuse: Kill farming DR, structure one-time score, territory restrictions

---

## Overview

**What:** Death-based scoring where score accumulates during each life and permanently saves to leaderboard only when you die, combined with queen ant economy where base-core owners receive score from worker deposits.

**Why:** Create risk/reward tension, enable economic warfare, force clan coordination, prevent safe score farming, make high-score players valuable targets.

**Key Rule:** Logout = Death (no safe score banking)

---

## Core Mechanics

### Death Banking Flow

1. Player spawns → currentScore = 0
2. Earn points through kills/deposits/raids
3. Die (combat OR logout) → currentScore → leaderboardScore (permanent)
4. Respawn → currentScore = 0 (starts over)
5. Leaderboard ranks by cumulative leaderboardScore

**Score Types:**
- **currentScore**: Accumulated this life, lost on death (risky)
- **leaderboardScore**: Permanent cumulative total (safe)

---

## Score Sources

### 1. PvP Kills (Personal)

**Formula:**
```
killScore = 100 + (victim.currentScore × 0.25) + (victim.gearValue × 0.1)
finalScore = killScore × survivalMultiplier
```

**Survival Multiplier:**
| Time Alive | Multiplier |
|------------|-----------|
| 0-15 min | 1.0x |
| 15-30 min | 1.2x |
| 30-60 min | 1.5x |
| 60+ min | 2.0x (max) |

**Example:**
- Kill player with 10,000 current score, 5,000 gear value
- Base: 100 + (10,000 × 0.25) + (5,000 × 0.1) = 3,100
- Alive 45min (1.5x multiplier) → 3,100 × 1.5 = **4,650 points**

**Victim:**
- Current score (10,000) → Saved to leaderboard (keeps it)
- Killer steals 25% (2,500) as bonus
- Respawns with currentScore = 0

---

### 2. Base-Core Deposits (Queen)

**Concept:** Resources deposited to base-core give score to **base-core owner**, not depositor.

**Resource Values:**

| Resource | Points Each |
|----------|-------------|
| Iron/Copper | 0.5 |
| Gold/Silver | 2 |
| Amethyst/Jade | 5 |
| Meteorite | 15 |
| Dragon Scale | 40 |
| Wood/Stone | 0 (crafting only) |

**Example Transaction:**
```
Worker deposits: 100 Gold, 20 Amethyst
→ Upkeep: 4,000 upkeep points (keeps base alive)
→ Score: (100 × 2) + (20 × 5) = 300 points to base-core owner
→ Depositor: 0 points (feeds the queen)
```

**Dual Benefit:**
- Upkeep: Keeps base alive (existing system)
- Score: Gives points to owner (new system)

---

### 3. Raiding (Personal)

**Structure Destruction:**

| Structure | Base Points | Tier Multiplier |
|-----------|-------------|-----------------|
| Wall/Door | 20 | Tier 1: 1x, Tier 2: 2x, Tier 3: 3x, Tier 4: 5x |
| Storage | 50 | Same |
| Workbench | 100 | Same |
| Turret | 200 | Same |
| **Base-Core** | **2,000** | **Same** |

**Example:** Destroy Tier 4 base-core → 2,000 × 5.0 = **10,000 points**

**Resource Theft:**
- Steal resources → No score
- Deposit stolen resources to YOUR base-core → Your owner gets score
- Net effect: Economic warfare, score transfer

---

### 4. Survival Bonus (Small)

- Every 10 minutes alive = +100 points
- Max +1,000 points (100 minutes)
- Applied at death only (not during life)

---

## Queen Ant Model

### Roles

**Queen (Base-Core Owner):**
- Receives score from all worker deposits
- Highest score potential (fed by clan)
- Big target (high-score = bounty)
- Must maintain expensive high-tier base

**Workers (Clan Members):**
- Farm resources, deposit to queen
- Build personal score via PvP/raids
- Less pressure, flexible

**Solo (Independent):**
- Place own base-core (own queen)
- Farm and deposit to self
- Slower growth than fed queens

### Multi-Queen Clans

Large clans can have multiple bases:
- Main base (Leader = queen, 15 workers)
- Outpost bases (Officers = queens, 8 workers each)
- Solo bases (Members with own bases)

---

## Balance & Rate Limits

### Hourly Deposit Caps

**Purpose:** Prevent infinite scaling from mega-clans

| Base Tier | Cap (per hour) |
|-----------|----------------|
| Tier 1 | 500 pts/hour |
| Tier 2 | 1,500 pts/hour |
| Tier 3 | 4,000 pts/hour |
| Tier 4 | 10,000 pts/hour |

**Once cap hit:**
- Deposits still give upkeep (base survives)
- Deposits give 0 score to queen (cap reached)
- Resets every hour (rolling window)

**Result:**
- 20-worker clan: Hits 10k cap in 30 min
- 50-worker clan: Still capped at 10k (diminishing returns)

---

### Realistic Hourly Rates

| Playstyle | Pts/Hour |
|-----------|----------|
| Solo casual | 500-1,000 |
| Solo PvP | 1,500-3,000 |
| Solo raiding | 2,500-5,000 |
| 5-man queen | 2,000-4,000 |
| 15-man queen | 6,000-10,000 |
| 30-man queen | 10,000-12,000 (cap) |

---

## Leaderboard System

### Individual Leaderboard

**Ranks by:** Cumulative leaderboardScore (all deaths combined)

**Shows:**
- Rank, player name, clan, total score
- Best single life score
- K/D ratio
- Score breakdown (kills, deposits, raids, survival)

### Clan Leaderboard

**Option A (Recommended):** Clan rank = Highest queen score
**Option B:** Clan rank = Sum of all member scores

### Live Leaderboard (Currently Alive)

**High-Value Targets:**
- Shows currently alive players with high currentScore
- Approximate scores (rounded to nearest 1,000)
- Color-coded threat levels:
  - 🟢 < 5,000 (low)
  - 🟡 5,000-15,000 (medium)
  - 🟠 15,000-40,000 (high)
  - 🔴 40,000+ (legendary)

**Creates bounty hunting:** Everyone can see who has high score → becomes target

---

## Anti-Abuse

### 1. Kill Farming Prevention

**Same victim DR (15-min window):**
- 1st kill: 100% score
- 2nd kill: 50% score
- 3rd kill: 25% score
- 4th+ kill: 0% score

**Clan protection:** Killing own clan = 0 score

---

### 2. Structure Farming Prevention

- Structure gives score ONCE (tracked by structure ID)
- Must exist 10+ minutes to give score
- Can't destroy own clan structures for score

---

### 3. Deposit Farming Prevention

**Resource origin tracking:**
- Gathered (world) → Can deposit ✅
- Looted (player) → Can deposit ✅
- Stolen (raid) → Can deposit ✅
- Withdrawn from storage → Cannot re-deposit ❌

**Territory restriction:**
- Cannot gather inside own base territory (100m radius)
- Must gather in neutral/enemy zones
- Prevents safe farming loops

---

### 4. AFK Prevention

- No input for 5 min → Survival bonus paused
- AFK for 10 min → Auto-logout → Death
- Deposit cooldown: Once per 5 min per player

---

### 5. Territory Limits

**Base territory (100m radius):**
- Resource gathering inside = 0 score
- Deposits still give upkeep (base alive)
- Deposits give 0 score to queen
- Forces players to venture out

---

## Integration Points

**Depends On:**
- Health/Damage System - Triggers death events
- Building System - Base-core ownership, structure destruction
- Inventory System - Resource deposits
- Session Persistence - Logout = death mechanic

**Used By:**
- Leaderboard UI - Displays rankings
- HUD - Shows current/leaderboard score
- Death Screen - Shows life breakdown

**Conflicts:**
- None (additive system)

---

## Edge Cases

**Q: What if base owner is offline when workers deposit?**
A: Score accumulates in owner's currentScore, banks when they next die

**Q: Can you kill yourself for instant banking?**
A: No advantage (death resets currentScore to 0)

**Q: What if you deposit to enemy base-core (griefing)?**
A: Cannot deposit to enemy bases (ownership check)

**Q: What if worker steals from own queen and deposits elsewhere?**
A: Stealing from own clan storage = not allowed (protection)

**Q: What if you change clans mid-life with high currentScore?**
A: Score stays with YOU, not transferred (personal progress)

**Q: What happens to score if base-core is destroyed?**
A: Queen keeps all earned leaderboard score (permanent)

---

## Balance Targets

| Metric | Target Value | Reasoning |
|--------|--------------|-----------|
| Solo hourly rate | 1,000-3,000 | Competitive with small clans |
| Large clan queen | 10,000-12,000 | Cap prevents infinite scaling |
| Average session (60min) | 2,000-5,000 | Feels productive |
| Great session | 15,000-40,000 | Rare, rewarding |
| Kill value | 100-5,000 | High-score players worth hunting |

---

## Implementation Notes

### Component Structure

```typescript
C_Score {
  currentScore: u32           // Lost on death
  leaderboardScore: u32       // Permanent cumulative
  totalDeaths: u16
  bestSingleLife: u32

  killScoreThisLife: u32      // Breakdown tracking
  depositScoreThisLife: u32
  raidScoreThisLife: u32
}

C_BaseCore {
  ownerId: u32
  tier: u8
  depositScoreThisHour: u32   // Rolling cap
  hourStartTimestamp: u32     // Reset tracking
}
```

### Key Handlers

**Death Handler:**
1. Sum all life scores (kills + deposits + raids + survival)
2. Add to leaderboardScore (permanent)
3. Reset currentScore to 0
4. Show death screen with breakdown
5. Update leaderboard rankings

**Deposit Handler:**
1. Calculate upkeep points (existing)
2. Calculate score value from resources
3. Check hourly cap (base-core)
4. Add capped score to base-core owner's currentScore
5. Track contribution for stats

**Kill Handler:**
1. Check same-clan (0 score)
2. Check kill farming DR (15-min window)
3. Calculate base score (100 + 25% victim + 10% gear)
4. Apply survival multiplier
5. Apply DR penalty
6. Add to killer's currentScore

---

## See Also

- [GAME_DESIGN.md](./../GAME_DESIGN.md) - Economy model, upkeep system, competitive philosophy
- [SESSION_PERSISTENCE_SYSTEM.md](./SESSION_PERSISTENCE_SYSTEM.md) - Logout = death mechanics
- [BUILDING_SYSTEM.md](./BUILDING_SYSTEM.md) - Base-core ownership, structure destruction
- [ATTACK_SYSTEM.md](./ATTACK_SYSTEM.md) - Combat mechanics for PvP kills
