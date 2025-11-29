# Documentation Style Guide

**Primary Purpose:** Enable AI to quickly understand, search, and navigate the game system with minimal token usage while providing clear context for complex design decisions.

**Secondary Purpose:** Human-readable reference for game designers and developers.

---

## 🤖 AI-First Documentation Principles

### Why This Matters

This is a **large, complex game project** with many interconnected systems. You (the developer) can't remember everything, so AI needs to:

1. **Scan quickly** - Find relevant information in 1-2 reads
2. **Understand relationships** - See how systems connect without deep code diving
3. **Preserve tokens** - Get maximum context with minimum tokens
4. **Maintain state** - Remember design intent across long conversations

### The Problem with Traditional Docs

❌ **Token waste:**
- Long code examples that don't add context
- Verbose explanations of obvious concepts
- Repeated information across sections
- Implementation details that change frequently

❌ **Poor searchability:**
- Inconsistent terminology
- Missing cross-references
- Buried key information in prose

❌ **Lost context:**
- No clear "why" behind decisions
- Missing edge cases and conflicts
- Unclear system priorities

---

## 📐 Documentation Structure (AI-Optimized)

Every doc should be **scannable in 3-5 seconds** to determine relevance, then **fully understood in 30-60 seconds** of reading.

### Template Structure

```markdown
# [System Name] - [One-Line Purpose]

## Quick Reference
[Dense, scannable summary - AI reads this first]

## Overview
**What:** [System purpose]
**Why:** [Design motivation]
**Key Rule:** [Most important constraint]

## Core Mechanics
[Game rules in plain language - bullet points preferred]

## Integration Points
**Depends On:** [List systems this needs]
**Used By:** [List systems that need this]
**Conflicts:** [List incompatible states]

## Edge Cases
[Question → Answer format]

## Balance Targets
[Table format - numbers only]

## See Also
[Links to related docs]
```

---

## 🎯 Writing for AI Efficiency

### 1. **Front-Load Critical Information**

✅ **GOOD (AI-optimized):**
```markdown
# Score System - Death-Based Leaderboard

## Quick Reference
- Score saves ONLY on death (logout = death)
- Kill rewards: 100 base + 25% of victim's score + 10% of gear
- Queen model: Resource deposits give score to base-core owner
- High-score players visible on map (bounty targets)

## Overview
**What:** Death-based scoring with queen ant economy
**Why:** Risk/reward, forces commitment, enables economic warfare
**Key Rule:** Logout = Death (no safe score banking)
```

**Why it works:**
- AI knows system purpose in 2 lines
- Quick Reference = instant context
- Can skip rest if not relevant

---

❌ **BAD (token wasteful):**
```markdown
# Score System

In this document, we will explore the scoring system that has been
designed for the game. The scoring system is an important part of
the gameplay experience and affects how players interact with the
game world. We'll start by discussing the history of scoring systems
in games, then move to our specific implementation...

[500 words later]

...and that's why score saves on death.
```

**Why it's bad:**
- AI must read 500+ tokens to find key info
- No quick scanning
- Verbose without adding value

---

### 2. **Use Dense, Structured Formats**

#### ✅ Tables Over Prose

**GOOD:**
```markdown
| Weapon Type | Consume Time | Trade-off |
|-------------|--------------|-----------|
| Empty Hands | 1s | Fast, safe |
| 1H Weapon | 1s | Fast, vulnerable |
| 2H Weapon | 2s | Slow, very vulnerable |
```
**Tokens:** ~50, **Info density:** High

**BAD:**
```markdown
When you have no weapon equipped, consuming an item takes 1 second, which
is relatively fast and you're not particularly vulnerable. However, if you
have a one-handed weapon equipped, it also takes 1 second but you're more
vulnerable because you can't defend yourself. With a two-handed weapon,
the consumption takes 2 seconds because you need to unequip the weapon
first, then consume, then re-equip it, making you very vulnerable...
```
**Tokens:** ~120, **Info density:** Low

---

#### ✅ Bullet Points Over Paragraphs

**GOOD:**
```markdown
### Death Banking Rules
- Score accumulates during life → currentScore
- Death triggers bank → currentScore added to leaderboardScore
- Logout = Death → instant banking
- Respawn resets currentScore to 0
- Leaderboard ranks by cumulative leaderboardScore
```

**BAD:**
```markdown
The death banking system works by accumulating score throughout the
player's life in a variable called currentScore. When the player dies,
this triggers a banking operation where the currentScore is added to
their leaderboardScore. It's important to note that logging out is
treated the same as dying, so it causes instant banking...
```

---

### 3. **Prioritize Searchable Keywords**

Use consistent terminology that AI can grep for:

✅ **Good keywords:**
- System names: "Attack System", "Score System", "Movement Restrictions"
- States: "Idle", "Attack", "Consume", "Building"
- Rules: "Logout = Death", "One hand slot", "Construction phase"
- Conflicts: "Blocks", "Requires", "Depends on"

❌ **Avoid ambiguity:**
- "The thing that handles attacks" → Use "Attack System"
- "When players stop playing" → Use "Logout" or "Disconnect"
- "The score stuff" → Use "Score System" or "Leaderboard"

---

### 4. **Minimize Code Examples**

Code should ONLY appear when it adds clarity that prose cannot.

#### When Code is Useful (Rare)

✅ **Config structure:**
```typescript
interface AttackPattern {
  id: number;
  damage: number;
  totalDurationMs: number;
  damagePhases: DamagePhase[];
}
```
**Why useful:** Shows data shape clearly, AI understands structure

✅ **State enums:**
```typescript
enum EntityAction {
  Idle = 1,
  Attack = 3,    // Blocks: Consume, Building
  Consume = 4,   // Blocks: Attack, Building
}
```
**Why useful:** Shows mutually exclusive states with inline comments

---

#### When Code is Wasteful (Most of the Time)

❌ **Implementation logic:**
```typescript
function onPlayerDeath(playerId: number) {
  const player = world.getEntity(playerId);
  const currentScore = C_Score.currentScore[player];
  const leaderboardScore = C_Score.leaderboardScore[player];
  C_Score.leaderboardScore[player] = leaderboardScore + currentScore;
  C_Score.currentScore[player] = 0;
  C_Score.totalDeaths[player]++;
}
```
**Why wasteful:**
- AI learns nothing about game design
- Changes frequently (will be outdated)
- Wastes ~80 tokens

**Better alternative:**
```markdown
On death: currentScore → leaderboardScore, then reset to 0
```
**Tokens:** ~15, same information

---

### 5. **Make Relationships Explicit**

AI needs to understand system dependencies quickly.

✅ **GOOD (explicit):**
```markdown
## Integration Points

**Depends On:**
- Entity Action System - Ensures mutual exclusivity with Attack/Building
- Movement Restriction System - Applies slowdown during consumption

**Used By:**
- Health System - Restores HP when food consumed
- Buff System - Applies status effects from items

**Conflicts:**
- Cannot consume while attacking (Entity Action blocks)
- Cannot consume while building (Entity Action blocks)

**Priority:** Consume action has priority 4 (lower than Attack:3)
```

**Why it works:**
- AI immediately knows all connections
- Can trace dependencies without reading other docs
- Conflicts are explicit (avoids bugs)

---

❌ **BAD (implicit):**
```markdown
The consumable system works with other systems to provide item usage.
When you use an item, it interacts with various game systems depending
on the item type. Some items restore health, others give buffs, and
the timing depends on what you're doing...
```

**Why it's bad:**
- No clear list of dependencies
- AI must infer relationships
- No conflict information

---

### 6. **Edge Cases in Q&A Format**

Edge cases are critical for AI to understand boundaries.

✅ **GOOD:**
```markdown
## Edge Cases

**Q: What if player dies while consuming?**
A: Consume cancels, item not consumed, HP not restored

**Q: What if base owner is offline when workers deposit?**
A: Score accumulates in owner's currentScore, banks when they next die

**Q: Can you attack while consuming?**
A: No. Entity Action System blocks (mutual exclusivity)

**Q: What if you logout during construction phase?**
A: Logout = death, building ownership transfers to killer if in combat
```

**Why it works:**
- Scannable Q&A format
- AI learns edge behavior quickly
- Covers common conflicts

---

## 🗂️ Document Types & Focus

### **Design Docs** (`GAME_DESIGN.md`, `SYSTEMS_OVERVIEW.md`)
**AI reads these:** First (to understand vision)
**Focus:** Philosophy, goals, why systems exist
**Code:** None
**Optimize for:** Big-picture understanding

### **System Docs** (`systems/*.md`)
**AI reads these:** When working on features
**Focus:** Game mechanics, player behavior, rules
**Code:** Minimal (configs/enums only)
**Optimize for:** Quick lookup of specific mechanics

### **Architecture Docs** (`architecture/*.md`)
**AI reads these:** When working on performance/implementation
**Focus:** Technical constraints, ECS patterns, optimization
**Code:** More allowed (but still minimal)
**Optimize for:** Understanding technical trade-offs

---

## ✅ Documentation Checklist

Before marking a doc complete, verify AI can answer these in 30 seconds:

- [ ] **What does this system do?** (1-line answer)
- [ ] **Why does it exist?** (Design motivation)
- [ ] **What does it depend on?** (List of systems)
- [ ] **What depends on it?** (List of systems)
- [ ] **What are the conflicts?** (Incompatible states)
- [ ] **What are the edge cases?** (3-5 Q&As)
- [ ] **What are the key numbers?** (Balance targets)

If AI needs to read code to answer any of these → **doc needs improvement**.

---

## 📊 Real Example Comparison

### ❌ BEFORE (Token-Wasteful, 450 tokens)

```markdown
# Consumable System

## Introduction

The consumable system is responsible for handling the consumption of items
by players in the game. This includes food items, water items, and medical
items. The system needs to coordinate with several other systems to ensure
that consumption happens correctly and that the appropriate effects are
applied to the player.

## How It Works

When a player wants to consume an item, they first need to have the item
in their inventory. The system will then check what type of weapon they
have equipped, because this affects the consumption timing. If they don't
have a weapon equipped, the consumption is relatively quick. However, if
they have a two-handed weapon like a bow or a staff, they need to unequip
it first, which makes the consumption slower.

The timing works like this:
- If you have no weapon: 1 second total
  - First, the consumable appears in your hand
  - Then you consume it
  - Then it disappears
- If you have a one-handed weapon: 1 second total
  - The weapon stays in your main hand
  - The consumable appears in your off-hand
  - You consume while holding the weapon
- If you have a two-handed weapon: 2 seconds total
  - First you unequip the weapon (0.5 seconds)
  - Then you consume (1 second)
  - Then you re-equip the weapon (0.5 seconds)

This creates an interesting tactical choice where two-handed weapons deal
more damage but make you more vulnerable when healing...

[More paragraphs continue...]
```

---

### ✅ AFTER (Token-Efficient, 180 tokens)

```markdown
# Consumable System - Context-Aware Item Usage

## Quick Reference
- Food/medical items restore HP/buffs
- Timing depends on equipped weapon (1-2s)
- Cannot consume while attacking or building
- 2H weapons = slower consumption (tactical tradeoff)

## Overview
**What:** Item consumption with weapon-aware timing
**Why:** Create tactical healing tradeoffs (2H = damage but vulnerability)
**Key Rule:** Cannot consume during Attack or Building actions

## Timing by Weapon

| Equipped | Time | Flow |
|----------|------|------|
| Empty Hands | 1s | Show → consume → hide |
| 1H Weapon | 1s | Keep weapon, consume in off-hand |
| 2H Weapon | 2s | Unequip (0.5s) → consume (1s) → re-equip (0.5s) |

**Design Intent:** 2H deals more damage but slower healing = vulnerability

## Integration Points
**Depends On:**
- Entity Action System (blocks during Attack/Building)
- Movement Restriction System (slowdown during consume)

**Used By:**
- Health System (HP restoration)
- Buff System (status effects)

**Conflicts:**
- Blocked by Attack action (priority 3)
- Blocked by Building action (priority 5)

## Edge Cases
**Q: What if player dies while consuming?**
A: Consume cancels, item not consumed, HP not restored

**Q: Can you move while consuming?**
A: Yes, at reduced speed (Movement Restriction System applies 0.7x multiplier)

## Balance Targets
- Food: 1-1.5s (not tedious)
- Medical: 2.5-4s (creates vulnerability)

## See Also
- [Entity Action System](../architecture/ENTITY_ACTION_STATUS_SYSTEM.md)
- [Movement Restrictions](./MOVEMENT_RESTRICTIONS.md)
```

**Token savings:** 270 tokens (~60% reduction)
**Information loss:** None (actually clearer)
**AI scan time:** 15 seconds vs 60 seconds

---

## 🎯 Golden Rules for AI-First Docs

1. **Front-load everything** - Key info in first 100 tokens
2. **Tables > Prose** - Dense, scannable format
3. **Bullets > Paragraphs** - Easy to scan
4. **Explicit > Implicit** - List all dependencies, don't make AI infer
5. **Q&A for edge cases** - Scannable, clear answers
6. **Minimal code** - Only when it adds clarity prose can't
7. **Consistent terms** - Same words for same concepts (searchable)
8. **Cross-reference heavily** - AI can jump to related systems

---

## 🔍 Self-Test: Is Your Doc AI-Optimized?

Ask yourself:

1. **Can AI understand the system purpose in the first 3 lines?**
   - Yes → Good
   - No → Add Quick Reference section

2. **Can AI find all dependencies without reading other docs?**
   - Yes → Good
   - No → Add explicit Integration Points section

3. **If you removed all code blocks, would the doc still be complete?**
   - Yes → Good
   - No → Replace code with prose/tables

4. **Can AI answer "what happens when X during Y?" from edge cases?**
   - Yes → Good
   - No → Add more Q&A edge cases

5. **Is every paragraph necessary, or is it filler?**
   - All necessary → Good
   - Has filler → Cut it ruthlessly

**Goal:** Maximum information density, minimum tokens.

---

## 📝 Template (Copy-Paste for New Docs)

```markdown
# [System Name] - [One-Line Purpose]

## Quick Reference
- [Key rule 1]
- [Key rule 2]
- [Key rule 3]
- [Key rule 4]

## Overview
**What:** [Purpose in 1 sentence]
**Why:** [Design motivation]
**Key Rule:** [Most important constraint]

## Core Mechanics

[Bullet points or table - NO PROSE]

## Integration Points

**Depends On:**
- [System A] - [Why]
- [System B] - [Why]

**Used By:**
- [System C] - [Why]

**Conflicts:**
- [What you can't do and why]

## Edge Cases

**Q: [Common question]?**
A: [Clear answer]

**Q: [Edge case]?**
A: [Clear answer]

## Balance Targets

| Metric | Value | Reasoning |
|--------|-------|-----------|
| [Target] | [Number] | [Why] |

## See Also
- [Related System 1](link)
- [Related System 2](link)
```

---

## 🤖 Final Note: This Doc is AI-Optimized

Notice how this guide itself follows the principles:
- Quick Reference at the top (you knew the purpose in 10 seconds)
- Heavy use of tables, bullets, comparisons
- Minimal prose, maximum density
- Clear examples with token counts
- Explicit structure

**If every doc followed this format, AI could:**
- Scan entire docs folder in ~5,000 tokens
- Understand full system in ~10,000 tokens
- Work on features without repeatedly asking for clarification

**That's the goal.** 🎯
