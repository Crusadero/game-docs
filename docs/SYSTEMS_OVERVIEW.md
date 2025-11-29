# Systems Overview - Architecture & Integration

## Quick Reference

- Vision: 200-player 2D PvP sandbox with skill-based combat, fast building, environmental challenges
- Core pillars: Fair PvP (no stat buffs), tactical building (anti-spam), environmental pressure, resource economy
- Central coordination: Entity Action System (mutual exclusivity) + Movement Restrictions (multi-source)
- Implementation phases: MVP Combat → Advanced Combat → Consumables → Building → Environment (optional)
- Key decision: No second hand slot (quick-use from inventory instead)

---

## Overview

**What:** Complete system architecture showing how combat, building, consumables, movement, and survival systems integrate through central coordination layers.

**Why:** Multiple systems need to restrict player actions simultaneously - Entity Action enforces mutual exclusivity (can't attack while consuming), Movement Restrictions allow multi-source stacking (attack + cold = very slow).

**Key Rule:** Entity Action = one primary action at a time, Movement Restrictions = multiple sources can stack

---

## Core Pillars

| Pillar | Mechanism | Result |
|--------|-----------|--------|
| **Fair PvP Combat** | Skill-based, no stat buffs, fast TTK (8-12s) | Balanced competitive combat |
| **Tactical Building** | Quick placement (0.3-0.5s), slowdown penalty, construction phase | Mobile gameplay, anti-spam |
| **Environmental Challenges** | Harsh biomes (winter, desert), quests | Survival pressure without tedium |
| **Resource Economy** | Gathering, crafting, trading, raiding | Wealth transfer, upkeep fuel |

---

## System Integration

### Central Coordination Flow

```
Player Input
    ↓
┌─────────────────────────────────────┐
│     Entity Action System            │  ← One primary action (Attack OR Consume OR Building)
│  (mutual exclusivity coordination)  │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│   Movement Restriction System       │  ← Multiple sources stack (Attack + Cold + Terrain)
│    (multi-source coordination)      │
└──────────────┬──────────────────────┘
               ↓
        ┌──────┴──────┬──────────┐
        ▼             ▼          ▼
    ┌────────┐  ┌──────────┐  ┌────────┐
    │ Attack │  │ Consume  │  │Building│
    │ System │  │  System  │  │ System │
    └───┬────┘  └─────┬────┘  └────┬───┘
        │             │            │
        └─────────────┼────────────┘
                      ▼
              ┌──────────────┐
              │   Physics    │
              │   Position   │
              └──────┬───────┘
                     ▼
              ┌──────────────┐
              │    Health    │
              │    System    │
              └──────────────┘
```

---

## System Chains

### Attack Chain

| Step | Action | System Involved |
|------|--------|----------------|
| 1 | Player presses attack | Input |
| 2 | Check EntityAction (can attack?) | Entity Action |
| 3 | Check cooldown | Cooldown |
| 4 | Get equipped weapon → attackPatternId | Inventory |
| 5 | Load AttackPattern (phases, timing) | Attack System |
| 6 | Set EntityAction.Attack | Entity Action |
| 7 | Apply movement restrictions | Movement Restrictions |
| 8 | Process damage phases (hit detection) | Attack System |
| 9 | Deal damage to targets | Health/Damage |
| 10 | Clear EntityAction, start cooldown | Entity Action, Cooldown |
| 11 | Broadcast to clients | Network |

**Affects:** Movement (locked/slowed), EntityAction (blocks consume/building), Health (targets)
**Blocked by:** Consume action, Building action, Attack cooldown

---

### Consumable Chain

| Step | Action | System Involved |
|------|--------|----------------|
| 1 | Player right-clicks consumable | Input |
| 2 | Check EntityAction (can consume?) | Entity Action |
| 3 | Get consumable config (timing, effects) | Item System |
| 4 | Get equipped weapon (1H vs 2H) | Equipment |
| 5 | Calculate timing (2H = unequip + consume + equip) | Consumable System |
| 6 | Set EntityAction.Consume | Entity Action |
| 7 | Apply movement restrictions | Movement Restrictions |
| 8 | Visual flow (show/hide consumable/weapon) | Client |
| 9 | Apply effects (heal, temperature) | Health, Temperature |
| 10 | Clear EntityAction, restore weapon | Entity Action, Equipment |

**Affects:** Health (healing), Temperature (cooling/warming), Movement (slowed), Inventory (item removed)
**Blocked by:** Attack action, Building action

---

### Building Chain

| Step | Action | System Involved |
|------|--------|----------------|
| 1 | Player right-clicks structure | Input |
| 2 | Check EntityAction (can build?) | Entity Action |
| 3 | Check resources | Inventory |
| 4 | Check cooldown (per-type) | Cooldown |
| 5 | Enter placement mode (ghost preview) | Building System |
| 6 | Validate placement (range, collision) | Physics |
| 7 | Apply slowdown (50% speed, 0.3s) | Movement Restrictions |
| 8 | Deduct resources, spawn building | Building System |
| 9 | Construction phase (25%→100% HP over 2-3s) | Building System |
| 10 | Set per-type cooldown, exit placement | Cooldown, Building |

**Affects:** Movement (slowdown), Resources (deducted), World (new building)
**Blocked by:** Attack action, Consume action

---

### Temperature Chain (Optional)

| Step | Action | System Involved |
|------|--------|----------------|
| 1 | Check player's biome | Biome System |
| 2 | Get temperature effect (winter = cold, desert = heat) | Temperature |
| 3 | Check warmth sources (campfire) | Building System |
| 4 | Check equipment (warm clothes) | Equipment |
| 5 | Apply temperature effects (damage, slow) | Health, Movement Restrictions |
| 6 | Player counters (place campfire, move) | Player action |

**Affects:** Health (damage), Movement (slowed when cold)
**Mitigated by:** Buildings (campfire), Consumables (warming items), Equipment (clothing)

---

### Movement Restriction Chain

**How multiple sources coordinate:**

| Source | Sets | Effect |
|--------|------|--------|
| **Attack** | lockRotation: true, speed: 0.8 | Can move but can't rotate |
| **Cold (Temperature)** | speed: 0.7 | Slowed down |
| **Terrain (Water)** | speed: 0.7 | Slowed down |

**Resolution:**
- Movement lock: If ANY source locks → can't move
- Rotation lock: If ANY source locks → can't rotate
- Speed: Multiply all multipliers (0.8 × 0.7 × 0.7 = 0.392 = 39% speed)

**Result:** Player attacking while cold in water = very slow + can't rotate

---

## System Dependencies

### Build Order

**Phase 1: Foundation**
1. Item System (weapons, consumables, structures)
2. ECS Components (EntityBase, Mobility)
3. Entity Action System (action flags, restrictions)
4. Movement Restriction System (locks, speed multipliers)

**Phase 2: Combat**
1. Attack System (patterns, hit detection)
2. Health/Damage System (apply damage, death)
3. Cooldown System (attack cooldowns)

**Phase 3: Survival**
1. Consumable System (timing, effects)
2. Temperature System (biome effects, optional)

**Phase 4: Building**
1. Building System (placement, construction phase)
2. Cooldown (per-type cooldowns)

---

## Implementation Phases

### Phase 1: MVP (Core Combat) ✅

**Priority:** MUST HAVE
**Status:** COMPLETE

- Item system (weapons)
- Attack patterns (sword swing)
- Attack system (single damage phase)
- Movement restrictions (basic)
- Entity action system (Attack flag)
- Health/Damage system

**Result:** Players can fight with basic attacks

---

### Phase 2: Advanced Combat 🔄

**Priority:** HIGH
**Status:** IN PROGRESS

- Multiple attack patterns (bow, spear, hammer)
- Multi-phase attacks (charge, combo)
- All hit detection modes (continuous, sampled)
- Attack cooldowns
- Movement during attacks (dash)

**Result:** Diverse combat

---

### Phase 3: Consumables ⏳

**Priority:** HIGH
**Status:** PLANNED

- Consumable items (food, water, bandages)
- Consume system (fast timing)
- Healing/effect application
- 1H vs 2H weapon handling

**Result:** Players can heal

---

### Phase 4: Building ⏳

**Priority:** MEDIUM
**Status:** PLANNED

- Structure items
- Placement mode system
- Ghost preview (client)
- Construction phase
- Cooldowns and costs

**Result:** Tactical structures

---

### Phase 5: Environment ⏳

**Priority:** LOW (Optional)
**Status:** PLANNED

- Temperature system
- Biome effects
- Campfires (warmth)
- Environmental damage

**Result:** Harsh biomes for quests

---

## Gameplay Loops

### Micro Loop (30s - 2min)

Combat encounter:
- Spot enemy → Attack → Take damage → Consume healing → Place wall to escape → Re-engage or flee

### Meso Loop (5-15min)

Resource gathering:
- Leave base → Travel to resources → Gather → Return → Store → Craft → Re-equip

### Macro Loop (30-60min)

Quest completion:
- Accept quest → Prepare → Travel to biome → Complete objective → Claim reward → Use for upgrades

### Meta Loop (Hours/Days)

Base progression:
- Build shelter → Gather resources → Expand base → Defend/raid → Accumulate wealth → Join clan → Control territory

---

## Balance Targets

### Combat

| Metric | Target | Factors |
|--------|--------|---------|
| **TTK (1v1)** | 8-12s | Attack damage, cooldowns, healing |
| **Escape (solo vs 3)** | 30-50% | Wall placement, slowdown penalty |
| **Heal time** | 2-4s | Creates vulnerability window |

### Building

| Metric | Target | Reasoning |
|--------|--------|-----------|
| **Placement speed** | 0.3-0.5s | Fast enough for mobile play |
| **Slowdown penalty** | 50% | Prevents spam, doesn't root |
| **Construction time** | 2-3s | Escape spam-trap, not too long |

### Environment

| Metric | Target | Purpose |
|--------|--------|---------|
| **Cold damage onset** | 5 min | Enough time to react |
| **Heat damage onset** | 3 min | Faster (more punishing) |
| **Quest difficulty** | 60-80% success | Challenging but fair |

---

## Design Decisions

### ✅ Decided

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Second hand slot** | Removed | Quick-use from inventory faster, less complexity |
| **Construction phase** | Include for combat buildings | Prevents instant wall-trapping |
| **Attack patterns** | MS-based (not ticks) | Readable, easy to balance |
| **Movement restrictions** | Multi-source stacking | Multiple systems can restrict simultaneously |

### ⏳ Undecided

| Decision | Status | Options |
|----------|--------|---------|
| **Survival meters (hunger/thirst)** | TBD after MVP | Temperature only vs Full survival |
| **Auto-feed system** | Conditional | Only if survival meters included |
| **Attack mid-animation sync** | Low priority | Skip vs Static pose vs Full seeking |

---

## Success Metrics

| Category | Metric | Target |
|----------|--------|--------|
| **Combat Feel** | TTK | 8-12s average |
| **Combat Feel** | Hit registration | 95%+ accuracy |
| **Combat Feel** | Player sentiment | "Fair and skill-based" |
| **Building UX** | Placement time | <0.5s |
| **Building UX** | Spam prevention | Can't place 3+ walls in 2s |
| **Building UX** | Player feedback | "Responsive, not spammy" |
| **Performance** | Concurrent players | 200 |
| **Performance** | Client FPS | 60 |
| **Performance** | Server tick | <100ms |

---

## Technical Architecture

### Server Authority

- Server validates all actions
- Client sends inputs, server confirms
- Client predicts for responsiveness

### Tick-Based Timing

- Server: 14 tps (~71ms per tick)
- Patterns: Defined in MS (readable)
- Execution: Converted to ticks
- Client: Real-time animations

### Component-Based ECS

- bitECS for performance
- Systems operate on queries
- Components = pure data
- No behavior in components

---

## See Also

- [Game Design](./GAME_DESIGN.md) - Economy model, upkeep, competitive philosophy
- [Attack System](./systems/ATTACK_SYSTEM.md) - Pattern-based combat
- [Movement Restrictions](./systems/MOVEMENT_RESTRICTIONS.md) - Multi-source coordination
- [Entity Action System](./architecture/ENTITY_ACTION_STATUS_SYSTEM.md) - Action mutual exclusivity
- [Building System](./systems/BUILDING_SYSTEM.md) - Fast tactical placement
- [Consumable System](./systems/CONSUMABLE_SYSTEM.md) - Context-aware item usage
