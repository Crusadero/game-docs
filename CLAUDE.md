# Game - Developer Guide

## What This File Is

**Navigation and conventions for the game project** - Shared across server, client, and shared package.

**For AI context** - Helps Claude Code understand game design, systems, and conventions.

---

## 🗺️ Documentation Navigator

### Read in This Order

| # | Document | Purpose |
|---|----------|---------|
| 1 | [STATUS.md](./docs/STATUS.md) | **🔥 START HERE** - What's implemented vs planned |
| 2 | [GAME_DESIGN.md](./docs/GAME_DESIGN.md) | **⭐ Core Philosophy** - Economy, upkeep, why systems work this way |
| 3 | [SYSTEMS_OVERVIEW.md](./docs/SYSTEMS_OVERVIEW.md) | **⭐ Architecture** - How all systems connect |
| 4 | This file | Quick reference, code conventions |

### Documentation Structure

```
docs/
├── STATUS.md                    # Implementation status
├── GAME_DESIGN.md               # Economy, upkeep, competitive model
├── SYSTEMS_OVERVIEW.md          # System integration & architecture
├── DOCUMENTATION_GUIDE.md       # How to write docs
├── README.md                    # Navigation guide
│
├── systems/                     # Gameplay mechanics
│   ├── ATTACK_SYSTEM.md
│   ├── BUILDING_SYSTEM.md
│   ├── CONSUMABLE_SYSTEM.md
│   ├── INVENTORY_UI_SYSTEM.md
│   ├── MOVEMENT_RESTRICTIONS.md
│   ├── SCORE_SYSTEM.md
│   ├── SESSION_PERSISTENCE_SYSTEM.md
│   └── UNIQUE_ITEMS_SYSTEM.md
│
└── architecture/                # Technical implementation
    ├── ENTITY_ACTION_STATUS_SYSTEM.md
    ├── ITEM_ANIMATION_SYSTEM.md
    ├── CAMERA_VISIBILITY_OPTIMIZATION.md
    └── BOX2D_CPP_MIGRATION.md
```

---

## 🔍 Find What You Need

### By Task Type

| Task | Doc to Read |
|------|-------------|
| **Understand game vision** | [GAME_DESIGN.md](./docs/GAME_DESIGN.md) |
| **Check implementation status** | [STATUS.md](./docs/STATUS.md) |
| **See how systems connect** | [SYSTEMS_OVERVIEW.md](./docs/SYSTEMS_OVERVIEW.md) |
| **New weapon/attack pattern** | [ATTACK_SYSTEM.md](./docs/systems/ATTACK_SYSTEM.md) |
| **Building mechanics** | [BUILDING_SYSTEM.md](./docs/systems/BUILDING_SYSTEM.md) |
| **Item consumption** | [CONSUMABLE_SYSTEM.md](./docs/systems/CONSUMABLE_SYSTEM.md) |
| **Movement behavior** | [MOVEMENT_RESTRICTIONS.md](./docs/systems/MOVEMENT_RESTRICTIONS.md) |
| **Inventory/UI features** | [INVENTORY_UI_SYSTEM.md](./docs/systems/INVENTORY_UI_SYSTEM.md) |
| **Leaderboard/scoring** | [SCORE_SYSTEM.md](./docs/systems/SCORE_SYSTEM.md) |
| **Action coordination** | [ENTITY_ACTION_STATUS_SYSTEM.md](./docs/architecture/ENTITY_ACTION_STATUS_SYSTEM.md) |
| **Performance optimization** | [CAMERA_VISIBILITY_OPTIMIZATION.md](./docs/architecture/CAMERA_VISIBILITY_OPTIMIZATION.md) or [BOX2D_CPP_MIGRATION.md](./docs/architecture/BOX2D_CPP_MIGRATION.md) |

---

## 🏗️ Multi-Repository Structure

This game is split across **3 repositories**:

| Repository | Purpose | Tech Stack |
|------------|---------|------------|
| **game-server** | Authoritative server, game logic, physics | Node.js, bitECS, Box2D |
| **game-client** | Browser client, rendering, input | TBD |
| **game-shared** | Shared types, enums, configs (npm package) | TypeScript |

**This documentation** is shared via git submodule in all 3 repos at `docs/` folder.

**When working on features:**
- Server logic → game-server repo
- Client rendering → game-client repo
- Shared types → game-shared repo (remember to version + publish!)
- Documentation → Update here (game-docs repo)

---

## 📝 Code Conventions

### Naming

| Type | Convention | Example |
|------|-----------|---------|
| **Components** | `C_<Name>` | `C_EntityBase`, `C_Attackable`, `C_Mobility` |
| **Systems** | `<Name>System` | `AttackSystem`, `MovementRestrictionSystem` |
| **Enums** | PascalCase | `EntityAction`, `AttackType`, `RestrictionSource` |
| **Config files** | kebab-case | `attack-patterns.ts`, `item-configs.ts` |

### Timing Convention

**IMPORTANT:** Patterns use **milliseconds**, server uses **ticks**

```typescript
// ✅ CORRECT: Patterns in MS (readable)
const pattern = {
  totalDurationMs: 1000,
  damagePhases: [{
    startMs: 400,
    endMs: 700
  }]
};

// Server converts to ticks for execution
const elapsedMs = ticksElapsed * TICK_MS; // TICK_MS = 71.43ms
const isPhaseActive = elapsedMs >= 400 && elapsedMs <= 700;
```

**Why:** MS is human-readable for balancing, ticks for server execution

### ECS Pattern

```typescript
// ✅ Components: Pure data only
export const C_Attackable = defineComponent({
  activePatternId: Types.ui8,
  attackStartTick: Types.ui32,
  currentPhaseIndex: Types.ui8,
});

// ✅ Systems: Logic on queries
const attackableQuery = defineQuery([C_Attackable, C_EntityBase]);

class AttackSystem {
  update() {
    const entities = attackableQuery(world);
    for (const eid of entities) {
      // Process logic here
    }
  }
}

// ❌ NO: Behavior in components
// ❌ NO: Direct entity manipulation outside systems
```

---

## 🧰 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Runtime** | Node.js + TypeScript |
| **ECS** | bitECS |
| **Physics** | Box2D (@box2d/core) |
| **Tick Rate** | 14 TPS (~71ms/tick) |
| **Architecture** | Server-authoritative |

---

## ✅ Design Checklist

Before adding features, ask:

1. **Does it add depth or just complexity?**
2. **Can skilled solo players still compete?**
3. **Does it interrupt core gameplay flow?**
4. **Is it server-authoritative?**
5. **Can it be abused/spammed?**

---

## 🚫 Anti-Patterns

These are **project decisions**, not code patterns:

- ❌ No stat buffs from gear/food
- ❌ No instant wall-trapping (construction phase required)
- ❌ No spam building (slowdown + cooldown + cost)
- ❌ No tedious survival mechanics
- ❌ No off-hand complexity (one hand only)

**Why these exist:** See [GAME_DESIGN.md](./docs/GAME_DESIGN.md)

---

## 📊 Current Status

**For detailed implementation status:** See [STATUS.md](./docs/STATUS.md)

---

## 📚 Learn More

- **New to the project?** Read docs in order (STATUS → GAME_DESIGN → SYSTEMS_OVERVIEW)
- **Working on a feature?** Find the relevant doc in the table above
- **Writing new docs?** Follow [DOCUMENTATION_GUIDE.md](./docs/DOCUMENTATION_GUIDE.md)
- **Need design context?** Check "See Also" sections in each doc

---

*Last Updated: 2025-01*
