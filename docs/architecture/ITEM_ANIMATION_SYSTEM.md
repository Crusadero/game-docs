# Item Animation System

## Overview

This document describes how items influence entity animations, particularly for held items that affect body part movements (arms, body, etc.) during different actions (idle, walk, attack).

## Problem

Different items require different animations:
- **Sword:** Two-hand grip, wide swing arc
- **Spear:** Two-hand thrust, narrow motion
- **Bow:** Draw back motion, hold steady
- **Axe:** Overhead swing, heavy motion
- **Unarmed:** Punching motion

Additionally, the same item needs different animations for different states:
- **Idle:** How arms rest while holding item
- **Walk:** How arms move during walking
- **Attack:** How arms move during attack animation

---

## Current Frontend Architecture

### Entity Structure

```typescript
PlayerEntity
├── bodySprite
├── leftArmSprite
├── rightArmSprite
├── headSprite
├── heldItemSprite
└── ...
```

### Animation State System

```typescript
// States control which animations play based on entity action
export class PlayerStateRegistry extends EntityAnimationRegistry<PlayerEntity> {
  constructor(entity: PlayerEntity) {
    super(entity, new LinearTransitionHandler(0.3));

    // Register states with priority (higher = more important)
    this.registerState(new PlayerIdleState('idle', entity), 0);
    this.registerState(new PlayerWalkingState('walk', entity), 1);
    this.registerState(new PlayerAttackState('attack', entity), 2);
  }
}

// Each state defines sprite animations
export class PlayerIdleState extends EntityAnimationState<PlayerEntity> {
  constructor(id: string, entity: PlayerEntity) {
    super(id, entity);

    // Create animations for each body part
    const leftArmAnimation = new WorldSpriteAnimation(entity.leftArmSprite, {
      returnOnFinish: true,
      xTranslateEasing: [/* ... */],
      // ...
    });

    this.animations.push(leftArmAnimation, /* ... */);
  }

  canActivate(): boolean {
    return !!(this.entity.action & EntityActionBitmask.Idle);
  }
}
```

---

## Design Options

### Option 1: Hardcoded States per Weapon (Current Issue)

```typescript
// Problem: Need different state class for each weapon type
export class PlayerSwordAttackState extends EntityAnimationState<PlayerEntity> {
  // Sword-specific arm motions hardcoded
}

export class PlayerSpearAttackState extends EntityAnimationState<PlayerEntity> {
  // Spear-specific arm motions hardcoded
}

export class PlayerBowAttackState extends EntityAnimationState<PlayerEntity> {
  // Bow-specific arm motions hardcoded
}
```

**Issues:**
- ❌ Explosion of state classes
- ❌ Repetitive code
- ❌ Hard to add new weapons
- ❌ Can't share logic between similar weapons

---

### Option 2: Data-Driven Animation Configs (Recommended)

Store animation data separately from state logic, allowing states to be generic and reusable.

#### Architecture Overview

```
Item Definition → Animation Set → State applies animations
     ↓                 ↓                    ↓
  "Iron Sword"      "sword"         PlayerMeleeAttackState
                      ↓                    ↓
                idle/walk/attack   Builds sprite animations
```

---

## Implementation

### 1. Animation Set Definitions

Create reusable animation configurations for item categories.

```typescript
// Animation configuration for a single body part
interface BodyPartAnimationConfig {
  xTranslateEasing?: EasingConfig[];
  yTranslateEasing?: EasingConfig[];
  xScaleEasing?: EasingConfig[];
  yScaleEasing?: EasingConfig[];
  rotationEasing?: EasingConfig[];
}

interface EasingConfig {
  startValue: number;
  endValue: number;
  duration: number; // seconds
  easingType: AnimationEasing;
}

// Complete animation config for an action
interface ActionAnimationConfig {
  leftArm?: BodyPartAnimationConfig;
  rightArm?: BodyPartAnimationConfig;
  body?: BodyPartAnimationConfig;
  head?: BodyPartAnimationConfig;
  item?: BodyPartAnimationConfig; // For held item sprite
  duration?: number; // Override duration (or use attack pattern duration)
}

// Animation set for an item type (all actions)
interface ItemAnimationSet {
  idle: ActionAnimationConfig;
  walk: ActionAnimationConfig;
  attack: ActionAnimationConfig;
  block?: ActionAnimationConfig;
  // ... other actions
}
```

### 2. Animation Set Registry

```typescript
// Define animation sets for each item type
const AnimationSets: Record<string, ItemAnimationSet> = {
  // Unarmed
  unarmed: {
    idle: {
      leftArm: {
        yTranslateEasing: [
          { startValue: 0, endValue: 2, duration: 1.5, easingType: AnimationEasing.EaseInOut }
        ]
      },
      rightArm: {
        yTranslateEasing: [
          { startValue: 0, endValue: 2, duration: 1.5, easingType: AnimationEasing.EaseInOut }
        ]
      }
    },
    walk: {
      leftArm: {
        rotationEasing: [
          { startValue: -15, endValue: 15, duration: 0.6, easingType: AnimationEasing.Linear }
        ]
      },
      rightArm: {
        rotationEasing: [
          { startValue: 15, endValue: -15, duration: 0.6, easingType: AnimationEasing.Linear }
        ]
      }
    },
    attack: {
      leftArm: {
        xTranslateEasing: [
          { startValue: 0, endValue: 20, duration: 0.3, easingType: AnimationEasing.EaseOut },
          { startValue: 20, endValue: 0, duration: 0.2, easingType: AnimationEasing.EaseIn }
        ]
      },
      // Punch motion
    }
  },

  // Sword (two-handed)
  sword: {
    idle: {
      leftArm: {
        xTranslateEasing: [{ startValue: 0, endValue: 3, duration: 2, easingType: AnimationEasing.Linear }]
      },
      rightArm: {
        xTranslateEasing: [{ startValue: 0, endValue: -3, duration: 2, easingType: AnimationEasing.Linear }]
      },
      // Arms holding sword at side
    },
    walk: {
      body: {
        rotationEasing: [
          { startValue: -5, endValue: 5, duration: 0.8, easingType: AnimationEasing.Linear }
        ]
      },
      // Sword sways as walking
    },
    attack: {
      leftArm: {
        rotationEasing: [
          { startValue: 0, endValue: -90, duration: 0.6, easingType: AnimationEasing.EaseOut },
          { startValue: -90, endValue: 0, duration: 0.4, easingType: AnimationEasing.EaseIn }
        ]
      },
      rightArm: {
        rotationEasing: [
          { startValue: 0, endValue: -90, duration: 0.6, easingType: AnimationEasing.EaseOut },
          { startValue: -90, endValue: 0, duration: 0.4, easingType: AnimationEasing.EaseIn }
        ]
      },
      // Swing arc motion
    }
  },

  // Spear (thrust motion)
  spear: {
    idle: {
      leftArm: {
        xTranslateEasing: [{ startValue: 0, endValue: -5, duration: 2, easingType: AnimationEasing.Linear }]
      },
      rightArm: {
        xTranslateEasing: [{ startValue: 0, endValue: 5, duration: 2, easingType: AnimationEasing.Linear }]
      },
    },
    walk: {
      // Similar to sword but held more vertically
    },
    attack: {
      leftArm: {
        xTranslateEasing: [
          { startValue: 0, endValue: -30, duration: 0.3, easingType: AnimationEasing.EaseOut },
          { startValue: -30, endValue: 0, duration: 0.2, easingType: AnimationEasing.EaseIn }
        ]
      },
      rightArm: {
        xTranslateEasing: [
          { startValue: 0, endValue: 40, duration: 0.3, easingType: AnimationEasing.EaseOut },
          { startValue: 40, endValue: 0, duration: 0.2, easingType: AnimationEasing.EaseIn }
        ]
      },
      // Forward thrust motion
    }
  },

  // Bow (draw and release)
  bow: {
    idle: {
      leftArm: {
        // Hold bow down at side
      },
      rightArm: {
        // Hand on string
      }
    },
    walk: {
      // Carry bow
    },
    attack: {
      leftArm: {
        rotationEasing: [
          { startValue: 0, endValue: -45, duration: 0.2, easingType: AnimationEasing.EaseOut }
        ]
      },
      rightArm: {
        xTranslateEasing: [
          { startValue: 0, endValue: -20, duration: 0.2, easingType: AnimationEasing.EaseOut },
          { startValue: -20, endValue: 0, duration: 0.1, easingType: AnimationEasing.EaseIn }
        ]
      },
      // Draw back and release
    }
  }
};
```

### 3. Item Provider System

```typescript
export class ItemAnimationProvider {
  private animationSets: Map<string, ItemAnimationSet> = new Map();

  // Register animation set
  registerAnimationSet(setId: string, animationSet: ItemAnimationSet): void {
    this.animationSets.set(setId, animationSet);
  }

  // Register category-based config (current system)
  registerCategoryAnimationConfig(
    category: ItemTypeBitmask,
    config: {
      baseAnimation: Partial<ItemAnimationSet>;
      subCategories?: Record<number, Partial<ItemAnimationSet>>;
    }
  ): void {
    // Store base animations
    const setId = `category_${category}`;
    this.animationSets.set(setId, config.baseAnimation as ItemAnimationSet);

    // Store subcategory overrides
    if (config.subCategories) {
      for (const [subCategory, overrides] of Object.entries(config.subCategories)) {
        const subSetId = `${setId}_${subCategory}`;
        this.animationSets.set(subSetId, {
          ...config.baseAnimation,
          ...overrides
        } as ItemAnimationSet);
      }
    }
  }

  // Get animation config for action
  getAnimationConfig(
    setId: string,
    action: 'idle' | 'walk' | 'attack' | 'block'
  ): ActionAnimationConfig | null {
    const animSet = this.animationSets.get(setId);
    return animSet?.[action] ?? null;
  }

  // Get config by item
  getAnimationConfigForItem(
    item: Item | null,
    action: 'idle' | 'walk' | 'attack' | 'block'
  ): ActionAnimationConfig {
    if (!item) {
      return this.getAnimationConfig('unarmed', action) ?? {};
    }

    // Try item-specific animation set first
    if (item.animationSetId) {
      const config = this.getAnimationConfig(item.animationSetId, action);
      if (config) return config;
    }

    // Fall back to category-based
    const setId = `category_${item.category}_${item.subCategory}`;
    return this.getAnimationConfig(setId, action) ?? {};
  }
}
```

### 4. Generic States

States become generic and build animations from configs.

```typescript
export class PlayerAttackState extends EntityAnimationState<PlayerEntity> {
  private pattern: AttackPattern | null = null;

  constructor(
    id: string,
    entity: PlayerEntity,
    private itemAnimationProvider: ItemAnimationProvider
  ) {
    super(id, entity);
  }

  canActivate(): boolean {
    return this.entity.primaryAction === EntityAction.Attack;
  }

  onEnter(context?: { pattern: AttackPattern }): void {
    this.animations = []; // Clear previous

    this.pattern = context?.pattern ?? null;

    // Get held item
    const heldItem = this.entity.getHeldItem();

    // Get animation config for this item
    const animConfig = this.itemAnimationProvider.getAnimationConfigForItem(
      heldItem,
      'attack'
    );

    // Determine duration
    const duration = this.pattern
      ? this.pattern.totalDurationMs / 1000
      : animConfig.duration ?? 1.0;

    // Build animations from config
    if (animConfig.leftArm) {
      this.animations.push(
        new WorldSpriteAnimation(this.entity.leftArmSprite, {
          returnOnFinish: true,
          ...animConfig.leftArm
        })
      );
    }

    if (animConfig.rightArm) {
      this.animations.push(
        new WorldSpriteAnimation(this.entity.rightArmSprite, {
          returnOnFinish: true,
          ...animConfig.rightArm
        })
      );
    }

    if (animConfig.body) {
      this.animations.push(
        new WorldSpriteAnimation(this.entity.bodySprite, {
          returnOnFinish: true,
          ...animConfig.body
        })
      );
    }

    // Schedule hit effects if pattern has damage points
    if (this.pattern?.type === AttackType.Melee) {
      this.pattern.damagePoints.forEach(dp => {
        this.scheduleAt(dp.timeMs / 1000, () => {
          this.showHitEffect(dp.angleOffset, dp.range);
        });
      });
    }
  }

  private showHitEffect(angleOffset: number, range: number): void {
    // Visual hit effect at damage point
  }
}

// Similar for walk and idle states
export class PlayerWalkingState extends EntityAnimationState<PlayerEntity> {
  constructor(
    id: string,
    entity: PlayerEntity,
    private itemAnimationProvider: ItemAnimationProvider
  ) {
    super(id, entity);
  }

  canActivate(): boolean {
    return this.entity.primaryAction === EntityAction.Walk;
  }

  onEnter(): void {
    this.animations = [];

    const heldItem = this.entity.getHeldItem();
    const animConfig = this.itemAnimationProvider.getAnimationConfigForItem(
      heldItem,
      'walk'
    );

    // Build animations from config
    // ... (same pattern as attack state)
  }
}
```

---

## Item Definitions

### Add Animation Set Reference to Items

```typescript
// In item definitions
export const ItemsList = {
  1: {
    itemId: 1,
    name: "Iron Sword",
    attackPatternId: 1,
    animationSetId: "sword", // References AnimationSets["sword"]
    // ... other properties
  },
  2: {
    itemId: 2,
    name: "Legendary Blade",
    attackPatternId: 3, // Different attack pattern (spin)
    animationSetId: "sword", // Same animations as iron sword
  },
  5: {
    itemId: 5,
    name: "Spear",
    attackPatternId: 2,
    animationSetId: "spear",
  },
  10: {
    itemId: 10,
    name: "Bow",
    attackPatternId: 5,
    animationSetId: "bow",
  },
  20: {
    itemId: 20,
    name: "Flaming Sword",
    attackPatternId: 1,
    animationSetId: "sword",
    animationOverrides: {
      attack: {
        // Custom attack animation with fire effects
        item: {
          // Particle trail config
        }
      }
    }
  }
};
```

---

## Benefits

✅ **Reusable:** Multiple items share animation sets
✅ **Data-driven:** Easy to add new weapons without code changes
✅ **Flexible:** Can override specific animations per item
✅ **Maintainable:** Animation configs separate from state logic
✅ **Synced:** Animation duration matches attack pattern timing
✅ **Extensible:** Easy to add new body parts or effects

---

## Integration with Attack Patterns

Animation sets work together with attack patterns:

**Attack Pattern (Server + Client):**
- Defines timing (windup, hit points, recovery)
- Defines damage and hitboxes
- Determines which state to use

**Animation Set (Client only):**
- Defines sprite movements (easing curves)
- Determines visual style
- Syncs duration with attack pattern

**Example Flow:**
```
1. Server: Player attacks with "Iron Sword"
   → attackPatternId: 1 (sword_swing)
   → Send to client

2. Client receives attack action
   → Look up pattern: AttackPatterns[1]
   → totalDurationMs: 1000ms

3. Client transitions to attack state
   → Get item: "Iron Sword"
   → Get animation set: AnimationSets["sword"]
   → Get action config: animSet.attack

4. Build animations
   → Use configs from animSet.attack
   → Duration = pattern.totalDurationMs (1000ms)
   → Schedule hit effects at pattern.damagePoints[0].timeMs (600ms)

5. Play animations
   → Arms swing according to easing curves
   → Hit effect appears at 600ms
   → Returns to idle at 1000ms
```

---

## Advanced Use Cases

### Animation Variants

Some items may need multiple animation sets (light vs heavy attack):

```typescript
export const ItemsList = {
  1: {
    itemId: 1,
    name: "Greatsword",
    attackPatternId: 10,
    animationSets: {
      light: "sword_fast",  // Quick swing
      heavy: "sword_slow",  // Powerful overhead
    }
  }
};

// State determines which to use based on input
const animSetId = isHeavyAttack
  ? item.animationSets.heavy
  : item.animationSets.light;
```

### Two-Weapon Fighting

Different items in each hand:

```typescript
// Get configs for both hands
const primaryConfig = itemAnimationProvider.getAnimationConfigForItem(
  this.entity.primaryHandItem,
  'attack'
);
const secondaryConfig = itemAnimationProvider.getAnimationConfigForItem(
  this.entity.secondaryHandItem,
  'attack'
);

// Blend or choose based on which hand is attacking
```

### Procedural Modifications

Dynamically modify animations based on stats:

```typescript
// Faster attack speed = faster animation
const speedMultiplier = playerStats.attackSpeed;
const adjustedDuration = pattern.totalDurationMs / speedMultiplier;

// Apply multiplier to all easing durations
const adjustedConfig = this.scaleAnimationSpeed(animConfig, speedMultiplier);
```

---

## Migration Path

1. **Keep current hardcoded states** for now
2. **Create ItemAnimationProvider** and define animation sets
3. **Update one state at a time** to use configs
4. **Test each migration** before moving to next
5. **Remove hardcoded animations** once all states migrated

---

## See Also

- [Entity Action & Status System](./ENTITY_ACTION_STATUS_SYSTEM.md)
- [Attack System Design](./../systems/ATTACK_SYSTEM.md)
