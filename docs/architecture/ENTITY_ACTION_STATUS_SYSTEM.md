# Entity Action & Status System

## Overview

This document describes the unified system for managing entity actions and status effects. The goal is to separate "what an entity is doing" (actions) from "what is happening to an entity" (status effects).

## Problem We're Solving

**Old System:**
- Single bitmask (`EntityActionBitmask`) mixed different concepts:
  - Movement states (Walk, Idle)
  - Combat actions (Attack)
  - Status effects (Damaged, Healed, Withered)
  - Entity-specific flags (Opened, FollowingTarget)
- Unclear which flags can coexist vs. are mutually exclusive
- Frontend state machine needs priority handling for overlapping actions

**New System:**
- Separate actions from status effects
- Clear priority system for actions
- Multiple status effects can stack
- Clean separation of concerns

---

## Architecture

### 1. Actions (What entity is DOING)

Actions represent what the entity is actively doing. Multiple actions can be flagged as active, but only one "primary" action determines the animation.

#### Component: `C_EntityAction`

```typescript
export const C_EntityAction = defineComponent({
  flags: Types.ui16,           // Bitmask of all active actions
  primaryActionId: Types.ui8,  // Highest priority action (determines animation)
  actionData: Types.ui32,      // Metadata: attack pattern ID, interact target, etc.
  actionStartTick: Types.ui32, // When current primary action started
});
```

#### Enum: `EntityAction`

```typescript
export enum EntityAction {
  None = 0,
  Idle = 1,
  Walk = 2,
  Attack = 3,
  Block = 4,
  Interact = 5,
  Dash = 6,
}
```

#### Priority System

```typescript
export const ENTITY_ACTION_PRIORITY: Record<EntityAction, number> = {
  [EntityAction.None]: 0,
  [EntityAction.Idle]: 0,
  [EntityAction.Walk]: 1,
  [EntityAction.Block]: 2,
  [EntityAction.Interact]: 2,
  [EntityAction.Attack]: 3,
  [EntityAction.Dash]: 4,
};
```

**How it works:**
- Multiple actions can be active (Walk + Attack)
- `primaryActionId` is automatically set to the highest priority action
- Frontend uses `primaryActionId` to determine which animation to play
- Lower priority actions are tracked but don't affect animation

**Example:**
```
Player is walking and starts attacking:
- flags: Walk | Attack (both active)
- primaryActionId: Attack (priority 3 > 1)
- Frontend plays attack animation, ignores walk
```

---

### 2. Status Effects (What is HAPPENING TO entity)

Status effects are temporary visual/gameplay effects that can stack on top of any action.

#### Component: `C_EntityStatus`

```typescript
export const C_EntityStatus = defineComponent({
  effects: Types.ui16,           // Bitmask of active status effects
  effectTimers: [Types.ui16, 8], // Duration in ticks for each effect (0 = permanent)
});
```

#### Enum: `EntityStatusEffect`

```typescript
export enum EntityStatusEffect {
  Damaged = 1 << 0,      // Red flash (timed)
  Healed = 1 << 1,       // Green particles (timed)
  Burning = 1 << 2,      // Fire particles + DOT (timed)
  Poisoned = 1 << 3,     // Purple overlay + DOT (timed)
  Frozen = 1 << 4,       // Movement slow + blue tint (timed)
  Stunned = 1 << 5,      // Can't act (timed)
  Invulnerable = 1 << 6, // Can't be damaged (timed)
  Withered = 1 << 7,     // Shrink effect (timed)
  Scared = 1 << 8,       // Shake animation (timed)
  Angry = 1 << 9,        // Red aura/sound for mobs (timed or permanent)
  Sleeping = 1 << 10,    // Z's above head (until woken)
}
```

**How it works:**
- Multiple effects can be active simultaneously (Damaged + Burning + Poisoned)
- Each effect can have a timer (auto-removes when expires)
- Timer of 0 means permanent until manually removed
- Frontend renders all active effects as overlays/particles

**Example:**
```
Player is attacked by fire mob:
- Action: Walk (still moving)
- Status: Damaged | Burning
- Frontend: Plays walk animation + red flash + fire particles
```

---

## System Classes

### EntityActionSystem

Manages entity actions with automatic priority calculation.

```typescript
class EntityActionSystem {
  // Add an action
  static addAction(entityId: EntityId, action: EntityAction, data?: number): void

  // Remove an action
  static removeAction(entityId: EntityId, action: EntityAction): void

  // Check if entity has action
  static hasAction(entityId: EntityId, action: EntityAction): boolean

  // Get current primary action (for animation)
  static getPrimaryAction(entityId: EntityId): EntityAction

  // Get action metadata
  static getActionData(entityId: EntityId): number
}
```

**Key behaviors:**
- When adding action, automatically updates `primaryActionId` if higher priority
- When removing action, recalculates `primaryActionId` from remaining actions
- Stores metadata (like attack pattern ID) in `actionData`

### EntityStatusSystem

Manages status effects with timer support.

```typescript
class EntityStatusSystem {
  // Add status effect with optional duration
  static addEffect(entityId: EntityId, effect: EntityStatusEffect, durationTicks?: number): void

  // Remove status effect
  static removeEffect(entityId: EntityId, effect: EntityStatusEffect): void

  // Check if has effect
  static hasEffect(entityId: EntityId, effect: EntityStatusEffect): boolean

  // Update all timed effects (call every tick)
  static updateTimedEffects(world: World): void
}
```

**Key behaviors:**
- Effects with duration automatically expire and remove themselves
- `updateTimedEffects()` decrements timers each tick
- Multiple instances of same effect refresh the timer (not stack)

---

## Usage Examples

### Server-side

```typescript
// Player starts walking
EntityActionSystem.addAction(playerId, EntityAction.Walk);

// Player starts attacking while walking
const attackPatternId = 5; // sword_swing
EntityActionSystem.addAction(playerId, EntityAction.Attack, attackPatternId);
// Result: primaryActionId = Attack (higher priority than Walk)

// Player gets hit
EntityStatusSystem.addEffect(playerId, EntityStatusEffect.Damaged, secondsToTicks(0.3));
// Damaged effect will auto-remove after 0.3 seconds

// Player stops attacking (finishes attack cycle)
EntityActionSystem.removeAction(playerId, EntityAction.Attack);
// Result: primaryActionId recalculates to Walk

// Check if can act
if (EntityStatusSystem.hasEffect(playerId, EntityStatusEffect.Stunned)) {
  return; // Can't perform action
}
```

### Client-side

```typescript
// Receive entity update from server
interface EntityUpdatePacket {
  entityId: number;
  actionFlags: number;        // All active actions
  primaryAction: EntityAction; // Highest priority
  actionData: number;          // Metadata
  statusEffects: number;       // Active status effects
}

// Update entity
onEntityUpdate(update: EntityUpdatePacket) {
  const entity = this.getEntity(update.entityId);

  entity.actionFlags = update.actionFlags;
  entity.primaryAction = update.primaryAction;
  entity.statusEffects = update.statusEffects;

  // State machine picks animation based on primaryAction
  entity.stateRegistry.update();

  // Render status effect overlays
  if (entity.statusEffects & EntityStatusEffect.Damaged) {
    entity.flashRed();
  }
  if (entity.statusEffects & EntityStatusEffect.Burning) {
    entity.showFireParticles();
  }
}
```

### Frontend State System Integration

```typescript
// States check primary action (unchanged from current system)
export class PlayerAttackState extends EntityAnimationState<PlayerEntity> {
  canActivate(): boolean {
    // Only activate if attack is the primary action
    return this.entity.primaryAction === EntityAction.Attack;
  }

  onEnter(): void {
    // Get attack pattern from actionData
    const patternId = this.entity.actionData;
    const pattern = AttackPatterns[patternId];
    this.buildAttackAnimation(pattern);
  }
}

export class PlayerWalkingState extends EntityAnimationState<PlayerEntity> {
  canActivate(): boolean {
    return this.entity.primaryAction === EntityAction.Walk;
  }
}

// States registered with priority (for fallback order)
this.registerState(playerIdleState, 0);
this.registerState(playerWalkingState, 1);
this.registerState(playerAttackState, 2);
```

---

## Future Extensions

### AI System (For NPCs/Mobs)

```typescript
// Additional component for AI entities
export const C_AI = defineComponent({
  state: Types.ui8,          // AIState: Following, Fleeing, Patrolling
  mood: Types.ui8,           // AIMood: Aggressive, Passive, Scared
  targetEntityId: Types.ui32, // Who to follow/attack/flee from
  lastDecisionTick: Types.ui32,
});

export enum AIState {
  Idle = 0,
  Patrolling = 1,
  Following = 2,
  Fleeing = 3,
  Guarding = 4,
}

export enum AIMood {
  Passive = 0,    // Won't attack
  Neutral = 1,    // Attacks if provoked
  Aggressive = 2, // Attacks on sight
  Scared = 3,     // Flees from player
}
```

**How AI interacts with actions:**
```typescript
// AI system decides what actions to take
if (C_AI.state[mobId] === AIState.Following) {
  const distance = getDistanceTo(mobId, targetId);

  if (distance > 5) {
    EntityActionSystem.addAction(mobId, EntityAction.Walk);
  } else {
    EntityActionSystem.removeAction(mobId, EntityAction.Walk);
    EntityActionSystem.addAction(mobId, EntityAction.Attack);
  }
}

// Visual indicator for mood
if (C_AI.mood[mobId] === AIMood.Aggressive) {
  EntityStatusSystem.addEffect(mobId, EntityStatusEffect.Angry);
  // Shows red aura/plays angry sound
}
```

---

## Migration from Old System

### Current bitmask mapping

```typescript
// OLD (EntityActionBitmask)
Walk = 1,          // → EntityAction.Walk
Idle = 2,          // → EntityAction.Idle
Attack = 4,        // → EntityAction.Attack
Damaged = 8,       // → EntityStatusEffect.Damaged
Healed = 16,       // → EntityStatusEffect.Healed
Withered = 32,     // → EntityStatusEffect.Withered
Opened = 64,       // → Entity-specific component (not action/status)
Scared = 128,      // → EntityStatusEffect.Scared (or AIMood for NPCs)
FollowingTarget = 256, // → AIState.Following (for NPCs)
```

### Gradual migration plan

1. Add new components (`C_EntityAction`, `C_EntityStatus`)
2. Keep old `action` bitmask during transition
3. Update systems to write to both old and new
4. Update frontend to read from new system
5. Remove old `action` field once confirmed working

---

## Benefits

✅ **Clear separation:** Actions vs. Status Effects
✅ **Automatic priority:** Server calculates primary action
✅ **Stackable effects:** Multiple status effects work together
✅ **Timed effects:** Auto-cleanup for temporary effects
✅ **Frontend simplicity:** Just check `primaryAction` for animation
✅ **Extensible:** Easy to add new actions/effects
✅ **Type-safe:** Enums prevent invalid states

---

## See Also

- [Attack System Design](./../systems/ATTACK_SYSTEM.md)
- [Item Animation System](./ITEM_ANIMATION_SYSTEM.md)
