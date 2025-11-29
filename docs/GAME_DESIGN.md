# Game Design - Economic Warfare Sandbox

## Quick Reference

- Genre: Competitive 2D PvP sandbox with economic warfare
- Core Loop: Farm → Build → Fight → Raid → Score → Repeat (fast-paced, minutes not hours)
- Economy: Point-based upkeep (bases consume points from ANY accepted resource tier)
- Scarcity: Intentional (not enough rare resources for all clans to have Tier 4 bases)
- Map: Small & flat (forces player interaction, 2-5 min travel max)

---

## Overview

**What:** 200-player competitive sandbox where clans compete for resources, territory, and leaderboard dominance through economic warfare and raiding.

**Why:** Traditional survival games allow peaceful coexistence - this forces conflict through resource scarcity, upkeep pressure, and compact map design.

**Key Rule:** Raiding is economic necessity, not griefing - bases drain resources constantly, forcing farming and raiding for survival

---

## Core Design Pillars

| Pillar | Mechanism | Result |
|--------|-----------|--------|
| **Resource Sink Economy** | Bases consume upkeep points hourly | Must constantly farm or raid |
| **Forced Scarcity** | Rare resources insufficient for all clans | Competition over top-tier nodes |
| **Compact Map** | Small, flat, 200 players | Constant player interaction |
| **Raiding as Economy** | Stealing more efficient than farming | Profitable, not just griefing |
| **Self-Cleaning Map** | Decay without upkeep | No permanent clutter |
| **Fast-Paced Timing** | Minutes, not hours | Skill-based, accessible |
| **Resources as Currency** | No stat progression | Economic management, not RPG |
| **Intentional Conflict** | Every system pushes PvP | Can't win by avoiding combat |

---

## Upkeep Economy

### Point-Based System

**Innovation:** Bases consume "upkeep points" from ANY accepted resource tier (not specific resources)

**How it works:**
1. Base Heart has upkeep point pool (fuel tank)
2. Players deposit resources → Convert to points
3. Points drain over time based on tier
4. Points reach 0 → Base starts decaying

**Benefits:**
- Flexibility (use iron, gold, or amethyst depending on tier)
- Natural trading (efficiency vs bulk)
- All raid loot valuable (mix converts to points)
- Easy balance (tweak point values, not spawns)

### Tier-Gated Resources

**Higher-tier bases ONLY accept higher-tier resources**

| Tier | Drain | Accepted Resources | Example Upkeep |
|------|-------|--------------------|----------------|
| **Tier 1** | 100 pts/hour | Iron+, Copper+, Gold+, Amethyst+ | 10 iron OR 2.5 gold OR 1 amethyst/hour |
| **Tier 2** | 500 pts/hour | Iron+ (nerfed 5 pts), Gold+, Amethyst+ | 100 iron OR 16.7 gold OR 5 amethyst/hour |
| **Tier 3** | 2,000 pts/hour | Gold+, Amethyst+ (NO iron/copper) | 100 gold OR 33 amethyst/hour |
| **Tier 4** | 5,000 pts/hour | Amethyst+ ONLY (NO gold/silver) | 100 amethyst/hour ONLY |

**Design philosophy:**
- Bigger base = rarer resources required
- Can't spam wood/stone at all
- Iron becomes inefficient at Tier 2+
- Tier 4 forces competition over amethyst nodes
- Flexibility within tier (trade-off choices)

---

## Forced Resource Scarcity

### Supply Limits (Amethyst Example)

**Server:** 200 players, 15-20 amethyst nodes

| Metric | Value |
|--------|-------|
| **Total nodes** | 15-20 |
| **Yield per node** | 8-15 amethyst |
| **Respawn time** | 10-15 minutes |
| **Theoretical supply** | 1,000-1,200 amethyst/hour |
| **Tier 4 demand** | 100 amethyst/hour per base |
| **Theoretical max bases** | 10-12 |
| **Real efficiency** | 60-70% (contested, PvP, deaths) |
| **Sustainable bases** | **6-8 Tier 4 bases** |
| **Clans that can afford** | **3-5 large clans** |

**Why it matters:**
- ✅ Creates competition (not enough for everyone)
- ✅ Forces clans to fight over resources
- ✅ Makes raiding economically necessary
- ✅ Prevents server stagnation (can't peacefully coexist)
- ✅ Rewards skill/organization (efficient clans win)

---

## Raiding as Economic Necessity

### Why Clans MUST Raid

**Economic pressure:**
- Top-tier base costs 5,000 pts/hour (100 amethyst equivalent)
- Farming all needed resources = 60+ min/hour
- Can't dedicate entire clan to farming

**Raid profit example:**
- Enemy stockpiles: 50 amethyst + 200 gold + 1000 iron
- You steal 75%: 37 amethyst (1,850 pts) + 150 gold (3,000 pts) + 750 iron (3,750 pts)
- **Total: 8,600 points = 1.7 hours of Tier 4 upkeep**
- **Time invested: 20-minute raid**
- **Result: Raiding more efficient than farming**

**Point system benefit:**
- ALL stolen resources valuable (convert to points)
- Don't need specific types
- Mixed loot = efficient upkeep fuel
- Can use immediately

---

## Map Design

### Compact & Flat for Forced Interaction

| Characteristic | Purpose |
|----------------|---------|
| **Small size** | 2-5 min travel max between any points |
| **Flat 2D** | No height, no vision blocking |
| **Dense resources** | Overlapping routes create PvP |
| **No safe corners** | Will be found, can't hide forever |
| **Visible bases** | Large clan bases can be scouted |

**Comparison:**
- ❌ NOT Rust huge maps (hide for days)
- ✅ Agar.io density (constant interaction)
- ✅ Territory control matters (limited space)

---

## Self-Cleaning Map

### Decay System

| Structure Type | Decay Rate | Time to Destroy |
|----------------|------------|-----------------|
| **No Base Heart** | Start immediately, 10% HP/hour | 10 hours |
| **Base Heart (upkeep failed)** | Start after 2 hours, 20% HP/hour | 5 hours |
| **Temporary (outside territory)** | Start immediately | 1-2 hours |

**Result:**
- Dead clans = bases auto-delete
- Inactive players = structures removed
- New seasons start fresh
- Performance stays good

---

## Fast-Paced Timing

**All timing in minutes, not hours**

| Action | Time | Philosophy |
|--------|------|-----------|
| **Gathering** | 3-10 seconds per harvest | Fast = skill-based |
| **Building** | 0.3-0.5s (combat), 30-60s (base) | Fast = accessible |
| **Crafting** | Instant to 2 minutes | Fast = competitive |
| **Combat TTK** | 5-10 seconds | Fast = high stakes |
| **Respawn** | 5-10 seconds | Fast = back to action |

**Session length (1 hour):**
- 10 min: Farm resources
- 15 min: Craft gear, organize
- 20 min: PvP (defend, raid)
- 10 min: Base maintenance
- 5 min: Travel, scouting

---

## Solo Player Role

### NOT Primary Audience, But Viable

**Solo player types:**

| Type | Percentage | Role | Viability |
|------|-----------|------|-----------|
| **Clan Members Preparing** | 80% | Farm for clan, scout enemies | Full |
| **Newbies** | 15% | Learning, will join clan | Temporary |
| **True Solo** | 5% | Guerrilla, hit-and-run | Tier 2-3 only |

**Design balance:**
- Solo CAN play (access to resources)
- Solo CAN'T compete at top tier (by design)
- Solo CAN survive (Tier 2-3 sustainable)
- Solo role: Farming, scouting, guerrilla raiding

---

## Resources as Currency

### No Stat Progression

**NOT like:**
- ❌ RPG progression (level up = stronger)
- ❌ Tech trees (unlock better gear)
- ❌ Permanent upgrades

**IS like:**
- ✅ Economic management (income vs expenses)
- ✅ Territory control (control = more income)
- ✅ Skill-based PvP (gear matters, skill wins)

### Resource Flow

| Use | Percentage | Type |
|-----|-----------|------|
| **Base Upkeep** | 70% | Primary sink, constant drain |
| **Gear Crafting** | 20% | Consumable, lost on death |
| **Raiding Tools** | 10% | Temporary, single-use |

**Economic loop:**
Farm → Deposit (convert to points) → Craft Gear → PvP → Die/Win → Repeat

**Death:** Lose gear, lose carried resources, base upkeep continues
**Victory:** Steal gear, steal resources, raid base, use for upkeep/expansion

---

## Intentional Conflict Design

### Every System Pushes PvP

| System | Conflict Mechanism |
|--------|-------------------|
| **Resource Scarcity** | Not enough for all → must fight |
| **Map Design** | Small, flat → can't avoid players |
| **Upkeep Pressure** | Constant drain → must farm → PvP risk |
| **Scoring** | Points from kills/raids → aggressive play |
| **Decay** | Can't build infinite walls → must move |

**Outcome:** Can't win by avoiding PvP

---

## Balance Targets

### Economic Balance

| Metric | Target | Calculation |
|--------|--------|-------------|
| **Point gen (iron)** | 2,000 pts/hour | 200 iron × 10 pts |
| **Point gen (gold)** | 1,200 pts/hour | 60 gold × 20 pts |
| **Point gen (amethyst)** | 1,500 pts/hour | 30 amethyst × 50 pts |
| **Tier 1 upkeep** | 100 pts/hour | 10 iron OR 2.5 gold |
| **Tier 2 upkeep** | 500 pts/hour | 100 iron OR 16.7 gold |
| **Tier 3 upkeep** | 2,000 pts/hour | 100 gold OR 33 amethyst |
| **Tier 4 upkeep** | 5,000 pts/hour | 100 amethyst ONLY |

### Clan Efficiency

| Clan Size | Farming Commitment | Sustainable Tier |
|-----------|-------------------|------------------|
| **5-player** | 1 player, 30 min/hour | Tier 2 |
| **10-player** | 2 players, 40 min/hour | Tier 3 |
| **20-player** | 5 players, 60 min/hour + raids | Tier 4 |

### PvP Balance

| Metric | Target |
|--------|--------|
| **TTK (1v1 equal gear)** | 8-12 seconds |
| **Escape (solo vs 5)** | 30-50% with smart building |
| **Heal time** | 2-4 seconds (vulnerability) |
| **Raid profit margin** | 2-5x cost (if successful) |

---

## Design Checklist

Before adding features, ask:

| Question | Why It Matters |
|----------|----------------|
| Does it force players to leave base? | Prevents turtling |
| Does it create PvP opportunities? | Core gameplay |
| Can one clan monopolize it? | Needs balancing |
| Does it consume resources (sink)? | Prevents infinite stockpiling |
| Is it fast enough (<5 min)? | Competitive pace |
| Does it reward skill or just numbers? | Small clans need chances |
| Does it create economic pressure? | Drives raiding |
| Can it clutter map permanently? | Needs decay |
| Fits in 1-hour session? | Casual accessibility |
| Drives leaderboard scoring? | Competitive alignment |

---

## What Makes This Unique

1. **Point-Based Upkeep** - Flexible resource acceptance, natural trading
2. **Tier-Gated Resources** - Higher bases require rarer resources
3. **Forced Scarcity** - Intentional (not enough for all)
4. **Economic Raiding** - Profitable and necessary, not griefing
5. **Compact Map** - Forces interaction
6. **Self-Cleaning** - Decay keeps map clean
7. **Fast-Paced** - Minutes, not hours
8. **Seasonal Competition** - Fresh starts, leaderboards
9. **Conflict-Driven** - Every system pushes PvP

**NOT a survival sim.** Economic warfare sandbox with seasonal competition where point-based upkeep creates flexibility while tier-gating forces competition.

---

## See Also

- [Systems Overview](./SYSTEMS_OVERVIEW.md) - How all systems connect
- [Score System](./systems/SCORE_SYSTEM.md) - Death banking, queen ant economy
- [Building System](./systems/BUILDING_SYSTEM.md) - Base Heart, upkeep, decay
- [Session Persistence](./systems/SESSION_PERSISTENCE_SYSTEM.md) - Logout = death
