# Unique Items System - Quest & Skill Unlocks

## Quick Reference

- Two unlock paths: Quest-exclusive (soulbound, powerful) vs Skill-unlocked (tradeable, moderate)
- Zero combat advantage: All items provide economic efficiency only (faster gathering, QoL)
- Session-based: Reset on death like all items/skills
- Quest items: 1 per life, infinite durability, best-in-slot, requires quest completion (1-3 hours)
- Skill items: Craftable 3-5 per life, limited durability, tradeable, unlocks at skill levels 4/6
- Power scaling: +20-30% efficiency at mid-tier, +60-100% at endgame (non-combat only)

---

## Overview

**What:** Two separate systems for specialty items - prestigious quest rewards (soulbound, powerful) and practical skill-unlocked recipes (tradeable, useful).

**Why:** Quest items reward solo completionists with best-in-slot tools, skill items enable clan supply chains and trading economy, both provide economic efficiency without combat advantages.

**Key Rule:** PvE utility only - zero combat stat bonuses (damage, HP, speed), session-based (lost on death)

---

## Core Mechanics

### System Comparison

| Aspect | Quest-Exclusive Items | Skill-Unlocked Items |
|--------|----------------------|---------------------|
| **Acquisition** | Complete challenging quest (1-3 hours) | Level skill, craft with resources |
| **Quantity** | ONE per life | 3-5 per life (craftable) |
| **Tradeable** | NO (soulbound) | YES |
| **Power Level** | Very high (+50-60% efficiency) | Moderate (+15-25% efficiency) |
| **Durability** | Infinite | Limited (150-300 uses) |
| **Target Audience** | Solo players, completionists | Clan suppliers, traders |

### Design Rules

| Rule | Reason |
|------|--------|
| **PvE utility only** | Zero combat stats (fair PvP) |
| **Economic efficiency** | Faster gathering, automation, QoL |
| **Visible value** | Can see someone using unique tools |
| **Not mandatory** | Game playable without them |
| **Session-based** | Reset on death (no permanent advantage) |

---

## Quest-Exclusive Items

### Quest Trigger System

**How it works:**
- Quest auto-appears when player reaches skill level threshold
- Must complete quest to unlock soulbound reward
- One reward per life (cannot repeat until death)

| Quest Tier | Skill Level | Estimated Time | Power Level |
|------------|-------------|----------------|-------------|
| **Expert** | Level 3 | 45-90 minutes | +40-50% efficiency |
| **Master** | Level 5 | 1.5-2.5 hours | +60% efficiency |

### Quest Examples by Skill

#### Farming - Enchanted Scythe (Master)

**Trigger:** Farming Level 5
**Quest:** "Master of Agriculture"
**Tasks:**
- Harvest 200 crops in one life
- Grow 30 gold-quality crops
- Maintain farm for 60 minutes continuously
- Trade crops worth 10,000 score
- Survive in PvP zone with farm for 20 min

**Reward: Enchanted Scythe**
- Harvest 5x5 area in one swing (vs 1x1)
- +60% crop yield
- Auto-replant seeds (20% chance)
- Infinite durability

#### Mining - Dwarven Drill (Master)

**Trigger:** Mining Level 5
**Quest:** "Depths of the Earth"
**Tasks:**
- Mine 500 ore nodes
- Find 10 gems (emerald/ruby/diamond)
- Mine in all 4 corner regions
- Survive 30 minutes in deepest mine
- Defeat cave guardian (PvE boss)

**Reward: Dwarven Drill**
- Mine entire vein with one use (8-12 nodes)
- +50% ore yield
- 30% gem find chance
- Infinite durability
- Cooldown: 30 seconds

#### Foraging - Verdant Staff (Master)

**Trigger:** Foraging Level 5
**Quest:** "Guardian of the Forest"
**Tasks:**
- Gather 1000 wood in one life
- Collect 500 berries
- Find 20 rare herbs in dangerous areas
- Plant 50 trees
- Survive in forest biome for 45 minutes

**Reward: Verdant Staff**
- Auto-collect wood/berries in 15m radius (passive)
- Plant trees that grow in 5 min (vs 20 min)
- Trees you plant yield 2x wood
- Can create Sacred Grove (safe zone, 10 min)
- Infinite durability

### Quest Item Characteristics

**Soulbound mechanics:**
- Cannot drop, trade, or transfer
- Lost on death (like all items)
- Must re-earn each life

**Prestigious:**
- Rare to see (1-3 hour quest per life)
- Shows mastery and dedication
- Best-in-slot for function

---

## Skill-Unlocked Items

### Recipe Unlock System

**How it works:**
- Reach skill level → Recipe unlocks automatically
- Craft item using resources
- Limited crafts per life (3-5 depending on item)

| Unlock Level | Typical Power | Durability | Max Crafts |
|--------------|---------------|------------|------------|
| **Level 4** | +15-20% efficiency | 150-200 uses | 5 per life |
| **Level 6** | +20-30% efficiency | 200-300 uses | 3 per life |

### Craftable Examples by Skill

#### Farming - Seed Pouch (Level 6)

**Craft Cost:** 30 leather, 20 gold thread, 50 rare seeds
**Max Crafts:** 3 per life
**Tradeable:** Yes
**Durability:** 200 uses

**Effect:**
- Plant 3x3 area in one click (9 seeds consumed)
- 15% faster planting animation
- Holds 100 seeds

#### Mining - Ore Satchel (Level 6)

**Craft Cost:** 40 leather, 20 iron ingots
**Max Crafts:** 3 per life
**Tradeable:** Yes
**Durability:** 250 uses

**Effect:**
- Carry capacity +50% for ore/gems only
- Ore doesn't slow you down (normally ore = weight)
- Auto-smelt iron ore to ingots (every 10th ore)

#### Foraging - Lumberjack's Kit (Level 4)

**Craft Cost:** 20 iron, 15 leather, 10 wood
**Max Crafts:** 5 per life
**Tradeable:** Yes
**Durability:** 250 uses

**Effect:**
- Chop trees 30% faster
- +1 wood per tree
- 10% chance tree drops rare wood

#### Building - Builder's Toolkit (Level 6)

**Craft Cost:** 40 iron, 30 wood, 15 rope
**Max Crafts:** 3 per life
**Tradeable:** Yes
**Durability:** 300 uses

**Effect:**
- Build 25% faster
- Structures cost 15% less resources
- Repair structures from 10m distance

#### Fishing - Fish Trap Net (Level 6)

**Craft Cost:** 30 rope, 20 iron, 10 bait
**Max Crafts:** 3 per life
**Tradeable:** Yes
**Durability:** 100 uses (10 fish or 30 min)

**Effect:**
- Place trap in water (placeable structure)
- Passively catches 1 fish per 3 minutes
- Can have 3 traps active simultaneously
- Lasts 30 minutes or 10 fish (whichever first)

---

## Power Curve & Balance

### Efficiency Gains by Investment

| Time Investment | Items Obtained | Efficiency Gain | Player Type |
|-----------------|----------------|-----------------|-------------|
| 0 hours | None | +0% (baseline) | New players |
| 30 min | 1 Craftable (Lvl 4) | +20% | Casual |
| 1 hour | 2 Craftables (Lvl 4+6) | +40% | Regular |
| 1-1.5 hours | 1 Expert Quest | +40-50% | Solo completionist |
| 2 hours | 1 Master Quest | +60% | Dedicated specialist |
| 2-3 hours | Full setup (quests + crafts) | +80-100% | Endgame optimizer |

**Result:** Every hour invested = ~20-30% efficiency gain (diminishing returns)

### Clan vs Solo Dynamics

**10-player clan example:**

| Role | Count | Focus | Efficiency |
|------|-------|-------|-----------|
| **Quest Completionists** | 1-2 | Master quests (2-3 hours) | +60-80% personal (soulbound) |
| **Skill Crafters** | 3-5 | Level to 6-8, craft items | +20-30% for all (tradeable) |
| **Regular Members** | 6-10 | Use borrowed craftables | +20-30% from clan tools |

**Solo player path:**
- Session 1: Level to 5 → Complete Expert quest → +40-50% efficiency
- Session 2: Complete Master quest + craft items → +80% total efficiency
- Result: Solo can match clan efficiency through quests (time investment)

---

## Integration Points

**Depends On:**
- Skill System - Level progression triggers quest unlocks
- Crafting System - Resource costs, durability tracking
- Inventory System - Soulbound flag, trade restrictions
- Quest System - Task tracking, reward distribution

**Used By:**
- Economy System - Resource gathering efficiency
- Trading System - Craftable items as tradeable goods

**Conflicts:**
- None (additive utility system)

---

## Edge Cases

**Q: What if you complete quest but die before using reward?**
A: Reward granted immediately on quest completion, lost on death like all items

**Q: Can you drop quest items to bypass soulbound?**
A: No - soulbound flag prevents dropping, trading, or any transfer

**Q: What if you craft max items (5/5) then die?**
A: Counter resets on death, can craft 5 more next life

**Q: Can you repeat same quest multiple times per life?**
A: No - quest completes once per life, must die to reset

**Q: What happens to craftable items in storage when you die?**
A: All items lost on death (storage included), no persistence between sessions

**Q: Can quest items break?**
A: No - quest items have infinite durability (reward for difficulty)

**Q: What if clan member crafts items then leaves clan?**
A: Items stay with whoever holds them (tradeable), not tied to clan

---

## Anti-Exploit Design

### Preventing Overflow

| Exploit | Prevention |
|---------|-----------|
| **Item spam** | Max crafts per life (3-5 limit) |
| **Hoarding** | All items lost on death (session reset) |
| **Trading dupes** | Durability tracking, items break after X uses |
| **Quest farming** | One quest completion per life, must die to reset |
| **Resource loops** | High-tier crafts require gold/gems (limited supply) |

### Quest Rushing Prevention

| Mechanic | Purpose |
|----------|---------|
| **Time-gated tasks** | "Maintain farm 60 min" cannot be rushed |
| **Rare spawns** | "Find Ancient Tome" requires exploration |
| **PvP requirements** | "Survive in PvP zone 20 min" forces risk |
| **Boss fights** | PvE challenges (Leviathan, Cave Guardian) |
| **Skill gates** | Must reach Level 3-5 before quest appears |

---

## Implementation Notes

### Quest Trigger

```typescript
interface Quest {
  id: string;
  triggerSkill: 'farming' | 'mining' | 'foraging' | 'building' | 'fishing';
  triggerLevel: number;      // Quest appears at this level
  difficulty: 'expert' | 'master';
  estimatedTime: number;     // Minutes to complete
  reward: UniqueItem;
}
```

### Craftable Recipe

```typescript
interface CraftableRecipe {
  id: string;
  unlockSkill: string;
  unlockLevel: number;       // Recipe unlocks at this level
  craftCost: Resources;
  maxCraftsPerLife: number;  // Limit per session
  tradeable: boolean;
  durability: number;        // Uses before breaking
}
```

### Soulbound Flag

```typescript
interface Item {
  isSoulbound: boolean;      // Cannot drop/trade if true
  durability?: number;       // undefined = infinite (quest items)
  craftsRemainingThisLife?: number;  // For craftable limit tracking
}
```

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Quest Expert time | 45-90 min | Achievable in single session |
| Quest Master time | 1.5-2.5 hours | Significant but not excessive |
| Craftable durability | 150-300 uses | 1-2 hour lifespan before breaking |
| Max crafts per life | 3-5 | Enough to share, not enough to spam |
| Power ceiling | +80-100% | 2x efficiency at endgame (non-combat) |
| Solo vs clan balance | Equal potential | Solo quests match clan craftables |

---

## See Also

- [Score System](./SCORE_SYSTEM.md) - Economic efficiency impacts score generation
- [Session Persistence](./SESSION_PERSISTENCE_SYSTEM.md) - Logout = death, all items lost
- [Game Design](./../GAME_DESIGN.md) - Economy model, upkeep, competitive philosophy
