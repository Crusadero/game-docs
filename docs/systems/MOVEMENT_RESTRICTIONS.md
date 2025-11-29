# Movement Restriction System - Multi-Source Coordination

## Quick Reference

- Multi-source restrictions: Attack, status effects, terrain can restrict simultaneously
- Bitmask locks: Any source can block movement/rotation independently
- Multiplicative speed stacking: All speed multipliers multiply together (MIN of all)
- Clean separation: Each source manages only its own restrictions
- 10% minimum speed floor: Prevents complete immobilization from stacking
- Easy queries: Simple canMove(), canRotate() checks for any system

---

## Overview

**What:** Flexible restriction system managing movement and rotation constraints from multiple simultaneous sources using bitmasks and multiplicative speed stacking.

**Why:** Multiple systems need to restrict movement (attacks, stuns, terrain, status effects), each source must manage independently without conflicts, clear composition rules prevent "who set this?" bugs.

**Key Rule:** Movement/rotation locks are binary (any source blocks), speed multipliers stack multiplicatively (most restrictive wins)

---

## Core Mechanics

### Restriction Sources

| Source | Bitmask | Typical Use | Example |
|--------|---------|-------------|---------|
| **Attack** | 1 << 0 | During attack phases | Sword swing locks rotation |
| **Stunned** | 1 << 1 | Stun/knockdown effects | Cannot move or rotate |
| **Channeling** | 1 << 2 | Channeling abilities | Stationary spellcast |
| **Rooted** | 1 << 3 | Root in place | Can rotate, cannot move |
| **Knockback** | 1 << 4 | Being knocked back | Cannot control movement |
| **Terrain** | 1 << 5 | Water, mud effects | Slows movement |
| **StatusEffect** | 1 << 6 | Slow/freeze debuffs | Speed reduction |
| **Carrying** | 1 << 7 | Heavy object | Cannot sprint |

### Component Structure

```typescript
export const C_MovementRestrictions = defineComponent({
  movementBlockedBy: Types.ui16,  // Bitmask: 0 = free, >0 = blocked
  rotationBlockedBy: Types.ui16,  // Bitmask: 0 = free, >0 = blocked
  speedMultiplier: Types.f32,     // Final result of all sources
});
```

---

## Restriction Resolution

### Binary Locks (Movement/Rotation)

**Rule:** If ANY source blocks → cannot move/rotate

| movementBlockedBy | Can Move? | Reason |
|-------------------|-----------|--------|
| 0000 (0) | ✅ Yes | No restrictions |
| 0001 (Attack) | ❌ No | Attack blocks |
| 0010 (Stunned) | ❌ No | Stun blocks |
| 0011 (Attack + Stunned) | ❌ No | Either blocks |

### Speed Multiplier Stacking

**Rule:** Multiply all multipliers together (MIN of all = most restrictive)

| Sources Active | Individual Multipliers | Final Speed | Calculation |
|----------------|----------------------|-------------|-------------|
| Water | 0.7 | 70% | base × 0.7 |
| Water + Attack | 0.7, 0.8 | 56% | base × 0.7 × 0.8 |
| Water + Attack + Slow | 0.7, 0.8, 0.6 | 33.6% | base × 0.7 × 0.8 × 0.6 |
| Attack ends | 0.7, 0.6 | 42% | base × 0.7 × 0.6 |

**Minimum Floor:** Even with extreme stacking, speed clamped to 10% (prevents full immobilization)

---

## System API

### Movement Restrictions

```typescript
// Block/allow
MovementRestrictionSystem.blockMovement(entityId, RestrictionSource.Attack);
MovementRestrictionSystem.allowMovement(entityId, RestrictionSource.Attack);

// Query
MovementRestrictionSystem.canMove(entityId);  // true if movementBlockedBy === 0
MovementRestrictionSystem.isMovementBlockedBy(entityId, RestrictionSource.Attack);
```

### Rotation Restrictions

```typescript
// Block/allow
MovementRestrictionSystem.blockRotation(entityId, RestrictionSource.Attack);
MovementRestrictionSystem.allowRotation(entityId, RestrictionSource.Attack);

// Query
MovementRestrictionSystem.canRotate(entityId);  // true if rotationBlockedBy === 0
MovementRestrictionSystem.isRotationBlockedBy(entityId, RestrictionSource.Stunned);
```

### Speed Multipliers

```typescript
// Set/clear
MovementRestrictionSystem.setSpeedMultiplier(entityId, RestrictionSource.Terrain, 0.7);
MovementRestrictionSystem.clearSpeedMultiplier(entityId, RestrictionSource.Terrain);

// Query
const mult = MovementRestrictionSystem.getSpeedMultiplier(entityId);  // Returns final value
```

### Cleanup

```typescript
// Clear all restrictions from one source
MovementRestrictionSystem.clearAllRestrictions(entityId, RestrictionSource.Attack);

// Clear everything (entity death/removal)
MovementRestrictionSystem.clearAllRestrictionsForEntity(entityId);
```

---

## Usage Examples

### Attack System

```typescript
class AttackSystem {
  private applyPhaseRestrictions(entityId: number, phase: DamagePhase): void {
    // Movement lock
    if (phase.lockMovement) {
      MovementRestrictionSystem.blockMovement(entityId, RestrictionSource.Attack);
    } else {
      MovementRestrictionSystem.allowMovement(entityId, RestrictionSource.Attack);
    }

    // Rotation lock
    if (phase.lockRotation) {
      MovementRestrictionSystem.blockRotation(entityId, RestrictionSource.Attack);
    }

    // Speed multiplier
    if (phase.movementSpeedMultiplier !== undefined) {
      MovementRestrictionSystem.setSpeedMultiplier(
        entityId, RestrictionSource.Attack, phase.movementSpeedMultiplier
      );
    }
  }

  private endAttack(attackerId: number): void {
    MovementRestrictionSystem.clearAllRestrictions(attackerId, RestrictionSource.Attack);
  }
}
```

### Status Effect System

```typescript
// Stun: block both movement and rotation
function applyStun(entityId: number, durationMs: number): void {
  MovementRestrictionSystem.blockMovement(entityId, RestrictionSource.Stunned);
  MovementRestrictionSystem.blockRotation(entityId, RestrictionSource.Stunned);

  setTimeout(() => {
    MovementRestrictionSystem.allowMovement(entityId, RestrictionSource.Stunned);
    MovementRestrictionSystem.allowRotation(entityId, RestrictionSource.Stunned);
  }, durationMs);
}

// Slow: reduce speed by percent
function applySlow(entityId: number, durationMs: number, slowPercent: number): void {
  const multiplier = 1.0 - slowPercent;  // 40% slow = 0.6 multiplier
  MovementRestrictionSystem.setSpeedMultiplier(
    entityId, RestrictionSource.StatusEffect, multiplier
  );

  setTimeout(() => {
    MovementRestrictionSystem.clearSpeedMultiplier(entityId, RestrictionSource.StatusEffect);
  }, durationMs);
}

// Root: can rotate but cannot move
function applyRoot(entityId: number, durationMs: number): void {
  MovementRestrictionSystem.blockMovement(entityId, RestrictionSource.Rooted);
  // Rotation still allowed
}
```

### Terrain System

```typescript
class TerrainSystem {
  update(): void {
    for (const entityId of movingEntities) {
      const terrain = this.getTerrainAt(getPosition(entityId));

      switch (terrain) {
        case TerrainType.Water:
          MovementRestrictionSystem.setSpeedMultiplier(entityId, RestrictionSource.Terrain, 0.7);
          break;
        case TerrainType.Mud:
          MovementRestrictionSystem.setSpeedMultiplier(entityId, RestrictionSource.Terrain, 0.5);
          break;
        case TerrainType.Ice:
          MovementRestrictionSystem.setSpeedMultiplier(entityId, RestrictionSource.Terrain, 1.2);
          break;
        default:
          MovementRestrictionSystem.clearSpeedMultiplier(entityId, RestrictionSource.Terrain);
      }
    }
  }
}
```

### Movement System

```typescript
class MovementSystem {
  update(): void {
    for (const entityId of movingEntities) {
      // Check if movement allowed
      if (!MovementRestrictionSystem.canMove(entityId)) {
        continue;  // Skip this entity
      }

      // Get base speed
      const baseSpeed = C_Movement.speed[entityId];

      // Apply multiplier
      const multiplier = MovementRestrictionSystem.getSpeedMultiplier(entityId);
      const finalSpeed = baseSpeed * multiplier;

      // Move entity
      this.moveEntity(entityId, finalSpeed);
    }
  }
}
```

---

## Complex Scenario Example

### Overlapping Restrictions Timeline

**Initial:** Normal movement (speed = 50 units/tick)

**T+0s:** Enter water
- setSpeedMultiplier(Terrain, 0.7)
- Speed: 50 × 0.7 = 35

**T+2s:** Start attacking (bow charge)
- blockMovement(Attack)
- movementBlockedBy: 0001
- Cannot move (blocked)

**T+3s:** Get stunned mid-charge
- blockMovement(Stunned)
- blockRotation(Stunned)
- movementBlockedBy: 0011
- rotationBlockedBy: 0010
- Cannot move or rotate

**T+5s:** Attack ends
- clearAllRestrictions(Attack)
- movementBlockedBy: 0010 (still stunned)
- rotationBlockedBy: 0010
- Still cannot move/rotate

**T+6s:** Stun expires
- allowMovement(Stunned)
- allowRotation(Stunned)
- movementBlockedBy: 0000 (free!)
- Speed: 50 × 0.7 = 35 (still in water)

**T+8s:** Exit water
- clearSpeedMultiplier(Terrain)
- Speed: 50 × 1.0 = 50 (back to normal)

---

## Integration Points

**Depends On:**
- None (foundational system)

**Used By:**
- Attack System - Phase restrictions (lock, speed)
- Consumable System - Slowdown during consumption
- Building System - Slowdown during placement
- Status Effect System - Stuns, slows, roots
- Terrain System - Water, mud, ice effects

**Conflicts:**
- None (designed for multi-source use)

---

## Edge Cases

**Q: What if two sources set different speed multipliers?**
A: Both apply multiplicatively (0.7 × 0.5 = 0.35 final speed)

**Q: What if source clears restriction it didn't set?**
A: Bitmask approach allows it safely (clears that bit, others remain)

**Q: Can speed go negative or above 100%?**
A: Multipliers can be >1.0 (speed boost), min floor is 0.1 (10%)

**Q: What happens on entity death?**
A: clearAllRestrictionsForEntity() called, removes all restrictions

**Q: Can you block movement but not rotation?**
A: Yes, common for roots/snares (blockMovement only)

**Q: What if multiplier set to 0.0?**
A: Valid, but min floor clamps to 0.1 (10% speed)

---

## Implementation Notes

### Per-Source Speed Tracking

Speed multipliers stored separately per source (not in component):

```typescript
// System memory (not component)
const speedMultiplierSources = new Map<number, Map<RestrictionSource, number>>();
// entityId -> Map<source, multiplier>

private static updateFinalMultiplier(entityId: number): void {
  const sources = speedMultiplierSources.get(entityId);
  if (!sources || sources.size === 0) {
    C_MovementRestrictions.speedMultiplier[entityId] = 1.0;
    return;
  }

  // Multiply all multipliers
  let final = 1.0;
  for (const mult of sources.values()) {
    final *= mult;
  }

  // Enforce min 10%
  final = Math.max(final, 0.1);
  C_MovementRestrictions.speedMultiplier[entityId] = final;
}
```

### Bitmask Operations

```typescript
// Set bit (block)
movementBlockedBy |= RestrictionSource.Attack;  // 0000 | 0001 = 0001

// Clear bit (allow)
movementBlockedBy &= ~RestrictionSource.Attack;  // 0011 & 1110 = 0010

// Check bit (query)
const isBlocked = (movementBlockedBy & RestrictionSource.Attack) !== 0;
```

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Minimum speed | 10% base | Prevents full freeze, allows minimal retreat |
| Water slowdown | 70% speed | Noticeable but not punishing |
| Mud slowdown | 50% speed | Significant risk, avoid if possible |
| Attack slowdown | 50-80% | Depends on weapon/phase |
| Stun duration | 1-3s | Long enough to punish, short enough to recover |

---

## See Also

- [Attack System](./ATTACK_SYSTEM.md) - How attack phases apply restrictions
- [Consumable System](./CONSUMABLE_SYSTEM.md) - Slowdown during consumption
- [Building System](./BUILDING_SYSTEM.md) - Slowdown during placement
- [Entity Action System](./../architecture/ENTITY_ACTION_STATUS_SYSTEM.md) - Action coordination (separate from movement)
