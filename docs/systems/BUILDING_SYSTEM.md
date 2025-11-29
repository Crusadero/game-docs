# Building System - Fast Tactical Placement

## Quick Reference

- Fast placement: 0.2-0.5s for combat buildings
- Slowdown penalty: 50-80% speed while placing (anti-spam)
- Construction phase: Combat buildings start at 25% HP, build to 100% over 2-3s
- Per-type cooldowns: Can't spam same building type
- Resource cost: Primary limiter
- Solo escape viable: Can place wall while fleeing

---

## Overview

**What:** Lightweight building system for 200-player PvP with emphasis on tactical placement and solo viability.

**Why:** Enable solo escape from groups via smart building, prevent spam-trapping with construction phase, make building skill-based not quantity-based.

**Key Rule:** Fast placement BUT slowdown penalty = can't spam effectively

---

## Building Categories

| Type | Placement | Slowdown | Construction | HP | Use Case |
|------|-----------|----------|--------------|-----|----------|
| **Survival** (campfire) | 0.2-0.3s | 80% speed | Instant | 100 | Warmth while running |
| **Combat** (wall) | 0.3-0.5s | 50% speed | 2-3s (25%→100%) | 100 | Escape, blocking, cover |
| **Utility** (workbench) | 1-2s | Stop | Instant | 100 | Crafting (safe zones) |
| **Base** (foundation) | 3-5s | Stop | 10-30s (10%→500) | 500 | Permanent base |

---

## Core Mechanics

### Combat Building Flow

**Example: Place wall while fleeing**

1. Player fleeing from 3 attackers
2. Activates wall placement
3. Slows to 50% speed during 0.5s placement
4. Wall appears at 25% HP (weak)
5. Player continues fleeing
6. Wall builds to 100% HP over 2-3s
7. Pursuers can destroy weak wall or go around
8. Smart placement > spam

**Construction Phase:**
- Building appears immediately (ghost state)
- Starts at 25% HP (can be destroyed)
- Auto-builds to 100% HP over time
- Cannot be repaired while building
- Attackers can destroy before completion

---

## Anti-Spam Mechanics

### 1. Slowdown Penalty

**How it works:**
- Placing building applies Movement Restriction
- Speed reduced to 50-80% (depending on type)
- Duration: Placement time (0.2-5s)

**Why it prevents spam:**
- Player who spams 5 walls in a row:
  - 5 × 0.5s placement = 2.5s total
  - Moving at 50% speed during all placements
  - Chaser covers more ground (runs at 100%)
  - Spammer loses distance, gets caught

**Solo smart building:**
- Place 1-2 walls strategically at corners
- Brief slowdown but gains distance via blocking path
- Tactical > spam

---

### 2. Construction Phase

**Combat buildings start weak:**
- Initial HP: 25% of final
- Build time: 2-3s
- Vulnerable during construction

**Prevents instant wall-trapping:**
- Can't instantly surround and trap someone
- Trapped player breaks through weak walls (25% HP)
- By time walls build to 100%, player escaped

**Spam-building fails:**
- Spammer places 5 walls (all 25% HP initially)
- Chaser breaks through weak wall (1-2 hits)
- Walls haven't built to full strength yet

---

### 3. Per-Type Cooldowns

**Cannot spam same type:**
- Wall placed → 5s cooldown on walls
- Can still place door, barricade, campfire (different types)
- Forces variety, prevents spam

---

### 4. Resource Cost

**Primary limiter:**
- Wall costs 20 wood
- Player has 100 wood → max 5 walls
- Running out of resources stops spam naturally

---

## Movement Restrictions

### During Placement

**Applied by Movement Restriction System:**
- Source: RestrictionSource.Building
- Speed multiplier: 0.5-0.8 (type dependent)
- Duration: Placement time
- Cannot sprint during placement

**Combat building (wall):**
- 0.5s placement at 50% speed
- Total slowdown: 0.5s at half speed
- Pursuit: Chaser gains ~0.25s distance

**Survival building (campfire):**
- 0.2s placement at 80% speed
- Minimal impact (mobile gameplay)

---

## Integration Points

**Depends On:**
- Entity Action System - Sets Building action, blocks Attack/Consume
- Movement Restriction System - Applies slowdown during placement
- Inventory System - Resource costs, removal on place
- Physics System - Collision, placement validation

**Used By:**
- None (terminal system)

**Conflicts:**
- Blocked by Attack action (priority 3)
- Blocked by Consume action (priority 4)
- Building has priority 5 (lowest, gets interrupted)

---

## Edge Cases

**Q: What if player dies while placing?**
A: Placement cancels, building NOT placed, resources NOT consumed

**Q: Can you place buildings while consuming?**
A: No. Consume blocks Building (Entity Action)

**Q: What if you place on invalid terrain (water, steep slope)?**
A: Placement rejected, resources NOT consumed

**Q: Can you destroy own buildings?**
A: Yes, but gives 0 score (anti-farming)

**Q: What happens if you logout during construction phase?**
A: Logout = death, building continues construction (half-built structure remains)

**Q: Can buildings overlap?**
A: No. Collision check prevents overlap

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Combat placement | 0.3-0.5s | Fast enough for mobile play, slow enough for cost |
| Construction time | 2-3s | Enough to escape spam-trap, not too long |
| Initial HP | 25% | Weak enough to break through, strong enough to block briefly |
| Slowdown penalty | 50% | Significant cost, but doesn't root player |
| Solo escape chance | 30-50% | Smart building enables escape vs 2-3 attackers |

---

## Building Configuration

### Combat Wall Example

```typescript
{
  id: 'wooden_wall',
  category: 'combat',
  placementDurationMs: 500,
  movementSpeedMultiplier: 0.5,
  cooldownMs: 5000,

  constructionPhase: {
    enabled: true,
    durationMs: 2500,
    initialHP: 25,
    finalHP: 100
  },

  cost: {
    wood: 20
  },

  collision: {
    width: 2,
    height: 3,
    blocksMovement: true
  }
}
```

---

## Implementation Notes

### Component

```typescript
C_Building {
  active: bool              // Placing or constructing
  buildingId: u32
  startTick: u32

  // Construction phase
  isConstructing: bool
  constructionStartTick: u32
  constructionEndTick: u32
  currentHP: u16
  finalHP: u16
}

C_BuildingCooldown {
  buildingType: u32         // Per-type cooldown
  cooldownEndTick: u32
}
```

### Placement Flow

1. Check Entity Action (not attacking/consuming)
2. Check cooldown for building type
3. Check resources in inventory
4. Validate placement location (terrain, collision)
5. Set EntityAction.Building
6. Apply Movement Restriction (slowdown)
7. Remove resources from inventory
8. Place building at initial HP
9. Start construction phase if applicable
10. Clear EntityAction.Building after placement time
11. Remove Movement Restriction

### Construction Phase

1. Building placed at 25% HP
2. Every tick: HP += (finalHP - initialHP) / constructionTicks
3. Visual: Building "fills in" over time
4. Can be damaged/destroyed during construction
5. Completes at 100% HP

---

## Anti-Patterns

❌ **Don't:** Instant full-HP buildings (enables spam-trapping)
❌ **Don't:** Allow building while attacking (action conflict)
❌ **Don't:** No placement cost (infinite spam)
❌ **Don't:** Root player during placement (feels bad, kills mobile gameplay)

✅ **Do:** Start buildings weak, build to full strength
✅ **Do:** Apply slowdown during placement (spam penalty)
✅ **Do:** Enable solo escape via smart placement
✅ **Do:** Resource cost as primary limiter

---

## See Also

- [Entity Action System](./../architecture/ENTITY_ACTION_STATUS_SYSTEM.md) - Action coordination
- [Movement Restrictions](./MOVEMENT_RESTRICTIONS.md) - Slowdown during placement
- [GAME_DESIGN.md](./../GAME_DESIGN.md) - Mobile gameplay, solo viability philosophy
