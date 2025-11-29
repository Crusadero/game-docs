# Attack System - Pattern-Based Combat

## Quick Reference

- Pattern-based architecture: Separate game logic (damage, timing) from animations
- Multi-phase attacks: Single attack can have multiple hits with different timing/shapes
- Three hit detection modes: Instant (single check), Continuous (sweep arc), Sampled (specific times)
- Patterns in MS, server in ticks: Auto-converts for readability
- Movement integration: Each phase can lock/restrict movement independently
- Item-based: Only weapons/tools have attackPatternId, empty hands use unarmed pattern

---

## Overview

**What:** Flexible pattern-based attack system supporting melee, ranged, AOE, and charge attacks with multi-phase damage and hit detection.

**Why:** Data-driven patterns are easy to balance, separate game logic from visuals, support complex attacks (spin, charge, combo), enable flexible hit detection per phase.

**Key Rule:** Patterns defined in readable MS (400ms, 1000ms), server executes in ticks, converts MS for comparison

---

## Core Mechanics

### Attack Pattern Structure

| Property | Type | Purpose |
|----------|------|---------|
| **id** | number | Unique pattern identifier |
| **name** | string | Pattern name (sword_swing, spear_thrust) |
| **type** | AttackType | Melee, Projectile, AOE, Charge, Dash |
| **damagePhases[]** | DamagePhase[] | Multiple hits with timing, damage, restrictions |
| **totalDurationMs** | number | Total attack duration for bookkeeping |

### Damage Phase Properties

| Property | Type | Purpose |
|----------|------|---------|
| **startMs/endMs** | number | When this phase is active |
| **damage** | number | Damage to deal in this phase |
| **lockMovement** | boolean | Block movement during phase |
| **lockRotation** | boolean | Block rotation during phase |
| **movementSpeedMultiplier** | number | Speed modifier (0.5 = 50% speed) |
| **hitDetection** | HitDetection | How/when to check for hits |
| **canHitPreviouslyHitTargets** | boolean | Can same target be hit by multiple phases |

### Attack Types

```typescript
enum AttackType {
  Melee = 0,       // Sword, axe, spear
  Projectile = 1,  // Bow, staff
  AOE = 2,         // Hammer slam, ground pound
  Charge = 3,      // Bow charge
  Dash = 4,        // Dash attack
}
```

---

## Hit Detection Modes

### Mode Comparison

| Mode | When Checks | Use Case | Example |
|------|-------------|----------|---------|
| **Instant** | Once at checkAtMs | Single precise hit | Spear thrust at 300ms |
| **Continuous** | Every tick while active | Sweeping arc | Sword swing 400-700ms |
| **Sampled** | Specific times | Multi-hit combo | Three punches at 200ms, 400ms, 600ms |

### Hit Shapes

| Shape | Parameters | Use Case |
|-------|-----------|----------|
| **Cone** | range, arcAngle | Sword swings, frontal arcs |
| **Rectangle** | length, width | Spear thrusts, line attacks |
| **Circle** | radius, centerOffset | Hammer slam, AOE around player |
| **Line** | startOffset, endOffset | Piercing arrows, beam attacks |

---

## Pattern Examples

### Simple Sword Swing

```typescript
{
  id: 1, name: "sword_swing", type: AttackType.Melee, totalDurationMs: 1000,
  damagePhases: [{
    startMs: 400, endMs: 700, damage: 30,
    lockRotation: true,
    hitDetection: {
      mode: 'continuous',
      continuousSweep: { startAngle: -45, endAngle: 45 },
      shape: { type: 'cone', range: 2.0, arcAngle: 20 }
    }
  }]
}
```

### Hammer Slam (Impact + Shockwave)

```typescript
{
  id: 3, name: "hammer_slam", type: AttackType.AOE, totalDurationMs: 1400,
  damagePhases: [
    // Direct impact
    { startMs: 800, endMs: 850, damage: 50, lockMovement: true,
      hitDetection: { mode: 'instant', checkAtMs: 800,
        shape: { type: 'circle', radius: 1.5 } } },

    // Shockwave (can hit same targets again)
    { startMs: 850, endMs: 900, damage: 25, lockMovement: true,
      canHitPreviouslyHitTargets: true,
      hitDetection: { mode: 'instant', checkAtMs: 850,
        shape: { type: 'circle', radius: 4.0 } } }
  ]
}
```

---

## Tick to Millisecond Conversion

### Server Execution Flow

**Configuration:**
- TICK_RATE = 14 tps
- TICK_MS = 71.43ms per tick

**Pattern defined:** startMs: 400, endMs: 700
**Server checks each tick:**

| Tick | Elapsed MS | Phase Active? | Action |
|------|-----------|---------------|--------|
| 0-5 | 0-357ms | No (< 400) | Skip |
| 6 | 428.58ms | Yes (400-700) | onPhaseEnter(), checkDamage() |
| 7-9 | 500-643ms | Yes | checkDamage() each tick |
| 10 | 714.29ms | No (> 700) | onPhaseExit(), markCompleted() |

### Instant Hit Detection Logic

**Pattern:** checkAtMs: 600

| Tick | Elapsed MS | Distance from 600 | Check? |
|------|-----------|-------------------|--------|
| 7 | 500ms | 100ms | No (> TICK_MS) |
| 8 | 571.43ms | 28.57ms | **Yes** (< 71.43ms) → Deal damage, mark done |
| 9 | 642.86ms | 42.86ms | Skip (already checked) |

### Continuous Sweep Calculation

**Every active tick, interpolate angle:**
- progress = (elapsedMs - startMs) / (endMs - startMs)
- currentAngle = lerp(startAngle, endAngle, progress)

Example: Phase 400-700ms with sweep -45° to 45°
- Tick 6 (428.58ms): progress = 0.095 → angle = -36.45°
- Tick 7 (500ms): progress = 0.333 → angle = -15°
- Tick 9 (642.86ms): progress = 0.809 → angle = 27.8°

---

## Item to Pattern Mapping

### Attack Capability

| Item Type | attackPatternId? | Can Attack? | Pattern Used |
|-----------|-----------------|-------------|--------------|
| **Weapon** | ✅ Yes | ✅ Yes | Item's pattern |
| **Tool** | ✅ Yes | ✅ Yes | Item's pattern (weaker) |
| **Empty Hands** | N/A | ✅ Yes | Unarmed pattern (ID: 0) |
| **Consumable** | ❌ No | ❌ No | - |
| **Structure** | ❌ No | ❌ No | - |

### Weapon Example

```typescript
export const ItemsList = {
  1: { itemId: 1, name: "Iron Sword", itemTypeBitMask: ItemType.Weapon, attackPatternId: 1 },
  2: { itemId: 2, name: "Spear", itemTypeBitMask: ItemType.Weapon, attackPatternId: 2 },
  10: { itemId: 10, name: "Pickaxe", itemTypeBitMask: ItemType.Tool, attackPatternId: 20 }
}
```

### Unarmed Pattern

```typescript
export const UNARMED_PATTERN_ID = 0;
export const AttackPatterns = {
  [UNARMED_PATTERN_ID]: {
    id: 0, name: "unarmed_punch", type: AttackType.Melee, totalDurationMs: 600,
    damagePhases: [{
      startMs: 200, endMs: 300, damage: 10,
      hitDetection: { mode: 'instant', checkAtMs: 250,
        shape: { type: 'cone', range: 1.2, arcAngle: 30 } }
    }]
  }
}
```

---

## Integration Points

**Depends On:**
- Entity Action System - Sets Attack action, blocks Consume/Building
- Movement Restriction System - Applies lock/speed from phases
- Inventory System - Gets equipped item to determine pattern
- Physics System - Hit detection queries

**Used By:**
- Health/Damage System - Receives damage events from hits
- Animation System - Syncs client animations to pattern phases

**Conflicts:**
- Blocked by Consume action (priority 4)
- Blocked by Building action (priority 5)
- Attack has priority 3 (cannot attack while consuming/building)

---

## Edge Cases

**Q: What if player switches weapons during attack?**
A: Attack continues with original pattern (pattern locked on attack start)

**Q: What if target moves between continuous sweep checks?**
A: Each tick re-checks hit detection at current angle, can hit if they move into arc

**Q: Can multiple damage phases hit the same target?**
A: Only if `canHitPreviouslyHitTargets: true`, otherwise skip already-hit targets

**Q: What happens if attack pattern doesn't exist?**
A: Server validates pattern on attack start, rejects if missing (logs error)

**Q: Can you attack while moving?**
A: Yes, unless phase has `lockMovement: true` or `movementSpeedMultiplier < 1.0`

**Q: What if player dies mid-attack?**
A: Attack system clears all restrictions, EntityAction removes Attack flag

---

## Implementation Notes

### Component

```typescript
export const C_Attackable = defineComponent({
  activePatternId: Types.ui8,
  attackStartTick: Types.ui32,
  currentPhaseIndex: Types.ui8,
});
```

### Server Constants

```typescript
export const TICK_RATE = 14;  // ticks per second
export const TICK_MS = 1000 / TICK_RATE;  // ~71.43ms per tick
```

### Phase Lifecycle

**onPhaseEnter:**
1. Apply movement restrictions (lock/speed multiplier)
2. Set currentPhaseIndex
3. Reset hit tracking for phase

**During Phase (each tick):**
1. Calculate elapsed MS from ticks
2. Check if within phase startMs-endMs
3. Perform hit detection based on mode
4. Track hit entities, apply damage

**onPhaseExit:**
1. Mark phase completed
2. Clear phase-specific restrictions (next phase or end will apply new ones)

**Attack End:**
1. Clear all movement restrictions (RestrictionSource.Attack)
2. Clear hit tracking
3. Remove EntityAction.Attack

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| TTK (1v1 melee) | 8-12s | Skill-based, not too fast/slow |
| Sword swing | 1000ms total, 400-700ms damage | 300ms window to hit, feels responsive |
| Spear thrust | 500ms total, 300-350ms damage | Fast, precise, high skill ceiling |
| Hammer slam | 1400ms total, 800-900ms hits | Slow, powerful, vulnerable windup |
| Unarmed punch | 600ms, 10 damage | Weak but fast, emergency weapon |

---

## See Also

- [Movement Restrictions](./MOVEMENT_RESTRICTIONS.md) - How phases apply movement/rotation locks
- [Entity Action System](./../architecture/ENTITY_ACTION_STATUS_SYSTEM.md) - Action coordination, mutual exclusivity
- [Item Animation System](./../architecture/ITEM_ANIMATION_SYSTEM.md) - Client animation sync
