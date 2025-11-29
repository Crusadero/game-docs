# Consumable System - Context-Aware Item Usage

## Quick Reference

- Food/water: 1-1.5s (fast maintenance)
- Medical: 2.5-4s (tactical healing, creates vulnerability)
- Timing depends on equipped weapon (1H vs 2H)
- 2H weapons: +1s penalty (unequip → consume → re-equip)
- Cannot consume during Attack or Building actions
- Movement allowed during consumption (slowed to 70%)

---

## Overview

**What:** Flexible consumable system handling food, water, and medical items with weapon-aware timing.

**Why:** Create tactical tradeoffs where 2H weapons (higher damage) have slower healing, prevent tedious survival maintenance, balance PvP healing.

**Key Rule:** Cannot consume during Attack or Building actions (Entity Action mutual exclusivity)

---

## Core Mechanics

### Consumable Categories

| Category | Duration | Movement | Use Case |
|----------|----------|----------|----------|
| **Food/Water** | 1-1.5s | 70% speed | Survival maintenance (frequent) |
| **Medical** | 2.5-4s | 20% speed | Tactical healing (combat) |

**Food/Water:**
- Restores hunger/thirst/temperature
- Fast (not tedious since frequent)
- Can move while consuming

**Medical:**
- Restores HP (50-80)
- Slow (creates vulnerability)
- Interrupted if damaged
- Healing over time (HoT) not instant

---

## Consumption Flows

### Timing by Equipped Item

| Equipped | Total Time | Breakdown | Tradeoff |
|----------|------------|-----------|----------|
| **Empty Hands** | 1s | Consume (1s) | Fast, defenseless |
| **1H Weapon** | 1s | Consume (1s) | Fast, keep weapon ready |
| **2H Weapon** | 2s | Unequip (0.5s) + Consume (1s) + Re-equip (0.5s) | **Slow, tactical penalty** |

### Flow Details

**Empty Hands:**
1. Consumable appears in main hand
2. Play consume animation (1s)
3. Apply effects
4. Consumable disappears

**1H Weapon:**
1. Weapon stays in main hand
2. Consumable appears in off-hand
3. Play consume animation (1s)
4. Apply effects
5. Consumable disappears (weapon still equipped)

**2H Weapon:**
1. Unequip weapon animation (0.5s)
2. Weapon disappears
3. Consumable appears in main hand
4. Play consume animation (1s)
5. Apply effects
6. Re-equip weapon animation (0.5s)
7. Weapon reappears

**Design Intent:** 2H weapons deal more damage but slower to heal = vulnerability window

---

## Movement Restrictions

### During Consumption

**Applied by Movement Restriction System:**
- Source: RestrictionSource.Consume
- Speed multiplier: 0.7 (food/water) or 0.2 (medical)
- Cannot sprint
- Can move/rotate (unless medical item locks)

**Why Allow Movement:**
- Food/water: Routine maintenance, shouldn't root player
- Medical: Slight movement allowed (tactical retreat) but very slow

---

## Integration Points

**Depends On:**
- Entity Action System - Blocks during Attack/Building, sets Consume state
- Movement Restriction System - Applies slowdown during consumption
- Inventory System - Item selection, removal on consume
- Equipment System - Checks equipped weapon type (empty/1H/2H)

**Used By:**
- Health System - Applies HP restoration from medical
- Survival System - Applies hunger/thirst/temp from food/water
- Buff System - Applies status effects (future)

**Conflicts:**
- Blocked by Attack action (priority 3)
- Blocked by Building action (priority 5)
- Consume has priority 4 (cannot consume while attacking/building)

---

## Edge Cases

**Q: What if player dies while consuming?**
A: Consume cancels, item NOT consumed, effects NOT applied

**Q: What if player takes damage while using medical item?**
A: Medical consumption interrupted (cancelled), item NOT consumed

**Q: Can you swap weapons while consuming?**
A: No. Consume action blocks equipment changes (Entity Action)

**Q: What if you consume with 1H weapon then it breaks mid-consume?**
A: Consumption continues (timing already set), weapon removal handled separately

**Q: Can you consume while moving?**
A: Yes, at reduced speed (70% food, 20% medical)

**Q: What happens if you logout while consuming?**
A: Logout = death, consumption cancelled, item NOT consumed

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Food/water time | 1-1.5s | Not tedious, frequent use |
| Medical time (1H) | 2.5-3s | Creates vulnerability, skill-based timing |
| Medical time (2H) | 3.5-4.5s | Penalty for higher damage weapons |
| Medical interrupt | On damage | Prevents heal-tanking in combat |
| TTK vs healing | 8-12s TTK, 3-4s heal | Must retreat to heal successfully |

---

## Item Configuration

### Food Example

```typescript
{
  id: 'apple',
  type: 'consumable',
  category: 'food',
  consumeDurationMs: 1000,
  canConsumeWhileMoving: true,
  movementSpeedMultiplier: 0.7,
  cancelOnDamage: false,

  effects: {
    hunger: +30,
    health: +5
  }
}
```

### Medical Example

```typescript
{
  id: 'bandage',
  type: 'consumable',
  category: 'medical',
  consumeDurationMs: 3000,
  canConsumeWhileMoving: true,
  movementSpeedMultiplier: 0.2,
  cancelOnDamage: true,

  effects: {
    health: +50,
    healOverTime: {
      duration: 10000,
      tickRate: 1000,
      amountPerTick: 5
    }
  }
}
```

---

## Implementation Notes

### Component

```typescript
C_Consume {
  active: bool                    // Currently consuming
  itemId: u32                     // What's being consumed
  startTick: u32                  // When started
  totalDurationTicks: u16         // Total time (converted from MS)

  // Phases (for 2H weapons)
  unequipDurationTicks: u8
  consumeDurationTicks: u8
  reequipDurationTicks: u8
}
```

### Flow (Server)

**Start Consumption:**
1. Check Entity Action (not attacking/building)
2. Check equipped item (empty/1H/2H)
3. Calculate timing based on weapon type
4. Set EntityAction.Consume
5. Apply Movement Restriction (slowdown)
6. Start consume timer

**During Consumption:**
1. Track elapsed time each tick
2. Check for interrupts (damage for medical)
3. Sync phases to client (timeline-based)

**End Consumption:**
1. Verify completion (not interrupted)
2. Remove item from inventory
3. Apply effects (HP, hunger, thirst)
4. Clear EntityAction.Consume
5. Remove Movement Restriction

**Interrupt (Medical):**
1. Player takes damage
2. Cancel consumption immediately
3. Item NOT consumed (stays in inventory)
4. Clear EntityAction.Consume
5. Remove Movement Restriction

---

## Client Synchronization

**Server sends:**
```typescript
{
  type: 'CONSUME_START',
  entityId: number,
  itemId: number,
  totalDurationMs: number,
  phases: {
    unequipMs: number,
    consumeMs: number,
    reequipMs: number
  }
}
```

**Client calculates:**
- Current phase based on elapsed time
- Which animation to play
- When to show/hide consumable model
- When to show/hide weapon model

**Timeline:**
```
0ms ────── unequipMs ────── (unequip+consume)Ms ────── totalMs
  |            |                    |                      |
Unequip      Consume              Re-equip              Done
```

---

## Anti-Patterns

❌ **Don't:** Instant healing (removes tactical element)
❌ **Don't:** Allow consuming while attacking (action conflict)
❌ **Don't:** Make food/water tediously slow (needed frequently)
❌ **Don't:** Allow heal-tanking (medical must be interruptible)
❌ **Don't:** Root player during food consumption (feels bad)

✅ **Do:** Create tradeoff between 1H (fast heal) and 2H (high damage)
✅ **Do:** Keep food/water fast and simple (survival maintenance)
✅ **Do:** Make medical slow enough to create tactical vulnerability
✅ **Do:** Allow movement during consumption (retreat while healing)

---

## See Also

- [Entity Action System](./../architecture/ENTITY_ACTION_STATUS_SYSTEM.md) - Action coordination, mutual exclusivity
- [Movement Restrictions](./MOVEMENT_RESTRICTIONS.md) - Slowdown during consumption
- [Equipment System](STATUS.md) - 1H vs 2H weapon detection
- [Attack System](./ATTACK_SYSTEM.md) - Why attack blocks consumption
