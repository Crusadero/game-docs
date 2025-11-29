# Game Server Documentation

Welcome to the game server documentation! This guide will help you navigate the docs and understand where to find information.

---

## 📚 Quick Navigation

### 🚀 **Start Here**

| Document | Description | When to Read |
|----------|-------------|--------------|
| [STATUS.md](./STATUS.md) | **🔥 CHECK THIS FIRST** - Current implementation status, what's done vs planned | Know what exists before reading design docs |
| [GAME_DESIGN.md](./GAME_DESIGN.md) | **⭐ READ SECOND** - Economy model, upkeep system, competitive philosophy, resource scarcity | Understanding WHY the game works this way |
| [SYSTEMS_OVERVIEW.md](./SYSTEMS_OVERVIEW.md) | **⭐ READ THIRD** - How all systems connect, gameplay loops, balance matrix | Understanding HOW everything fits together |

### 🔧 **Game Systems** (Gameplay Mechanics)

All gameplay systems live in [`systems/`](./systems/):

| System | Description | Key Features |
|--------|-------------|--------------|
| [ATTACK_SYSTEM.md](./systems/ATTACK_SYSTEM.md) | Combat patterns, multi-phase damage, hit detection | Pattern-based, MS timing, flexible configs |
| [SCORE_SYSTEM.md](./systems/SCORE_SYSTEM.md) | Leaderboard, death banking, queen ant economy, PvP bounties | Death saves score, base core feeds workers |
| [BUILDING_SYSTEM.md](./systems/BUILDING_SYSTEM.md) | Structure placement, construction phase, anti-spam | Fast placement with slowdown penalty |
| [CONSUMABLE_SYSTEM.md](./systems/CONSUMABLE_SYSTEM.md) | Food/healing, tactical timing, weapon-aware consumption | 1H vs 2H weapon tradeoffs |
| [INVENTORY_UI_SYSTEM.md](./systems/INVENTORY_UI_SYSTEM.md) | Button-based transfers, cooldown display, buff icons | Fast PvP-ready inventory |
| [MOVEMENT_RESTRICTIONS.md](./systems/MOVEMENT_RESTRICTIONS.md) | Multi-source locks, speed multipliers | Coordinated movement control |
| [SESSION_PERSISTENCE_SYSTEM.md](./systems/SESSION_PERSISTENCE_SYSTEM.md) | Logout = death, session handling | No safe logout, forces commitment |
| [UNIQUE_ITEMS_SYSTEM.md](./systems/UNIQUE_ITEMS_SYSTEM.md) | Special item mechanics | Unique gameplay items |

### 🏗️ **Architecture** (Technical Implementation)

Technical implementation details live in [`architecture/`](./architecture/):

| Document | Description | Use Case |
|----------|-------------|----------|
| [ENTITY_ACTION_STATUS_SYSTEM.md](./architecture/ENTITY_ACTION_STATUS_SYSTEM.md) | ECS action coordination, mutual exclusivity | Understanding action state management |
| [ITEM_ANIMATION_SYSTEM.md](./architecture/ITEM_ANIMATION_SYSTEM.md) | Data-driven animation configs for items | Syncing animations with game state |
| [CAMERA_VISIBILITY_OPTIMIZATION.md](./architecture/CAMERA_VISIBILITY_OPTIMIZATION.md) | Spatial grid + region merging (15-40x faster) | Performance optimization |
| [BOX2D_CPP_MIGRATION.md](./architecture/BOX2D_CPP_MIGRATION.md) | Path to native C++ Box2D (6-10x speedup) | Scaling to 200-500 players |

---

## 🗺️ Finding What You Need

### By Task Type

**Working on gameplay features?** → [`systems/`](./systems/)
- New weapon/attack → [ATTACK_SYSTEM.md](./systems/ATTACK_SYSTEM.md)
- Inventory/UI changes → [INVENTORY_UI_SYSTEM.md](./systems/INVENTORY_UI_SYSTEM.md)
- Building mechanics → [BUILDING_SYSTEM.md](./systems/BUILDING_SYSTEM.md)
- Economy/scoring → [SCORE_SYSTEM.md](./systems/SCORE_SYSTEM.md)

**Working on technical implementation?** → [`architecture/`](./architecture/)
- ECS coordination → [ENTITY_ACTION_STATUS_SYSTEM.md](./architecture/ENTITY_ACTION_STATUS_SYSTEM.md)
- Animation sync → [ITEM_ANIMATION_SYSTEM.md](./architecture/ITEM_ANIMATION_SYSTEM.md)
- Performance → [CAMERA_VISIBILITY_OPTIMIZATION.md](./architecture/CAMERA_VISIBILITY_OPTIMIZATION.md) or [BOX2D_CPP_MIGRATION.md](./architecture/BOX2D_CPP_MIGRATION.md)

**Understanding the game's vision?** → Start with [GAME_DESIGN.md](./GAME_DESIGN.md)

**New to the codebase?**
1. Check [STATUS.md](./STATUS.md) (what's implemented vs planned)
2. Read [GAME_DESIGN.md](./GAME_DESIGN.md) (understand the "why")
3. Read [SYSTEMS_OVERVIEW.md](./SYSTEMS_OVERVIEW.md) (see how it all connects)
4. Check [../CLAUDE.md](../CLAUDE.md) for high-level decisions and code conventions
5. Dive into specific system docs as needed

---

## 📂 Folder Structure

```
docs/
├── STATUS.md                   # 🔥 Current implementation status
├── GAME_DESIGN.md              # ⭐ Core design philosophy, economy model
├── SYSTEMS_OVERVIEW.md         # ⭐ How all systems connect
├── DOCUMENTATION_GUIDE.md      # 📘 How to write AI-optimized docs
├── README.md                   # You are here!
│
├── systems/                    # Gameplay mechanics & features
│   ├── ATTACK_SYSTEM.md
│   ├── BUILDING_SYSTEM.md
│   ├── CONSUMABLE_SYSTEM.md
│   ├── INVENTORY_UI_SYSTEM.md
│   ├── MOVEMENT_RESTRICTIONS.md
│   ├── SCORE_SYSTEM.md
│   ├── SESSION_PERSISTENCE_SYSTEM.md
│   └── UNIQUE_ITEMS_SYSTEM.md
│
└── architecture/               # Technical implementation
    ├── ENTITY_ACTION_STATUS_SYSTEM.md
    ├── ITEM_ANIMATION_SYSTEM.md
    ├── CAMERA_VISIBILITY_OPTIMIZATION.md
    └── BOX2D_CPP_MIGRATION.md
```

---

## 🎯 Design Principles (Quick Reference)

### Core Philosophy
- **Economic warfare** - Upkeep forces conflict
- **Resource scarcity** - Not enough for everyone
- **No safe logout** - Logout = death
- **Skill-based PvP** - No stat buffs, fair combat
- **Fast-paced** - Quick building, fast consumption, mobile gameplay

### Balance Targets
- **TTK:** 8-12 seconds (1v1 melee)
- **Server target:** 200-300 concurrent players
- **Tick rate:** 14-20 TPS
- **Session length:** 1-hour productive sessions
- **Season length:** 2-4 weeks

### Key Decisions
✅ **Point-based upkeep** - Flexible resource acceptance
✅ **Death banking** - Score saves on death
✅ **One hand slot** - Simple, no off-hand complexity
✅ **Pattern-based combat** - Data-driven, easy to balance
✅ **Fast building** - With slowdown anti-spam

❌ **No stat buffs** - Fair PvP
❌ **No instant wall-trapping** - Construction phase
❌ **No safe logout** - Forces commitment

---

## 🔗 External Resources

- **Main README:** [../README.md](../game-server/README.md) - Project setup, installation
- **CLAUDE.md:** [../CLAUDE.md](../CLAUDE.md) - Developer guide, conventions, quick start
- **Source code:** [`../src/`](../game-server/src) - Implementation

---

## 📝 Contributing to Docs

**📘 [Read the Documentation Style Guide](./DOCUMENTATION_GUIDE.md)** for detailed guidelines on writing clear, searchable, AI-friendly documentation.

### Quick Guidelines

1. **Design first, code second** - Explain WHAT and WHY, not implementation details
2. **Use examples** - Real scenarios with numbers, edge cases
3. **Show relationships** - Link to related systems, explain integration points
4. **Keep it readable** - Plain language, clear headers, minimal code
5. **Cross-reference** - Link to related docs
6. **Update this README** - If you add new docs

### Doc Categories

- **Design docs** (`GAME_DESIGN.md`, `SYSTEMS_OVERVIEW.md`) → Design philosophy, "why" (no code)
- **Systems docs** (`systems/*.md`) → Gameplay mechanics, features (minimal code)
- **Architecture docs** (`architecture/*.md`) → Technical implementation (more code allowed)

### Documentation Checklist

Before marking a doc complete:
- [ ] Overview explains WHAT and WHY
- [ ] Core mechanics in plain language
- [ ] Real examples with numbers
- [ ] Edge cases addressed
- [ ] System integration points clear
- [ ] Code is minimal (configs/enums only)
- [ ] Cross-references to related systems

---

## 💡 Tips

- **Can't find something?** Use your IDE's search across all docs (Cmd/Ctrl + Shift + F)
- **Understanding a system?** Start with its system doc, then check architecture docs for implementation
- **Working on balance?** Check [GAME_DESIGN.md](./GAME_DESIGN.md) for design targets
- **Need design context?** Check "See Also" sections at the bottom of each doc

---

*Last Updated: 2025-01*
