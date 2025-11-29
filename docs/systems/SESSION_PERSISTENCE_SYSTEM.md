# Session Persistence System - Logout = Death

## Quick Reference

- Logout = Death (no character state between sessions)
- No safe logout mechanic
- Corpse remains 5 minutes (loot able)
- Score banks on logout (death banking)
- Forces commitment to current life
- Prevents logout-to-escape exploits

---

## Overview

**What:** Logging out is treated identically to death - character dies, loot drops, score banks to leaderboard.

**Why:** Prevent logout-to-escape, force commitment to current run, eliminate safe score banking, create tension for high-score runs, ensure persistent world (no disappearing).

**Key Rule:** No persistent character state between sessions = fresh spawn every login

---

## Core Mechanics

### Logout Flow

**When player disconnects (any reason):**

1. Treat as normal death
2. Process death (score banking, leaderboard update)
3. Drop loot (everything in inventory)
4. Create corpse (5-minute duration, lootable)
5. Player removed from world

**On next login:**
- Spawn as fresh character (no items, default stats)
- Previous score saved to leaderboard (permanent)
- Must re-gear from scratch

---

## Integration with Score System

### Score Banking

**On logout/disconnect:**
- currentScore → leaderboardScore (permanent)
- Best single life updated if record
- Total deaths incremented
- Respawn with currentScore = 0

**Same as combat death:**
- No difference between logout and death
- Score saved either way
- Cannot "preserve" high score by logging out

---

## Corpse Mechanics

### Loot Persistence

**Corpse properties:**
- Duration: 5 minutes
- Lootable: Yes (anyone can take)
- Despawn: After 5 min OR when empty
- Location: Where player logged out

**Why 5 minutes:**
- Long enough for player to reconnect (crash/disconnect)
- Short enough to prevent clutter
- Gives looters time to find and claim

**Loot contents:**
- All inventory items
- All equipped items
- Resources
- Everything except leaderboard score (that's saved)

---

## Edge Cases

**Q: What if I crash/disconnect accidentally?**
A: Same as logout = death. Corpse remains 5 min, can reconnect and try to recover loot

**Q: Can I combat-log (logout during fight)?**
A: No benefit. You die, they get kill credit + loot your corpse

**Q: What if server crashes?**
A: Implementation choice - either all players "logout" (fair) or rollback (preserves state)

**Q: Can I logout in safe zone to avoid death?**
A: No safe zones for logout. Logout = death everywhere

**Q: What happens to my base if I logout?**
A: Base persists (buildings remain). Only character dies

**Q: Can I quickly logout/login to teleport?**
A: No. Spawn at random/fixed spawn point, not where you logged out

---

## Comparison to Other Models

### Traditional MMO (Persistent Character)
❌ Logout = safe (can preserve high-score run)
❌ Logout-to-escape works (unfair in PvP)
❌ Character state persists

### Session-Based (This System)
✅ Logout = death (no safe preserve)
✅ Logout-to-escape = you die anyway
✅ Fresh character each session

---

## Design Intent

### Forces Commitment
- High-score run? Can't logout to "save" it
- Being chased? Can't logout to escape
- Must commit to current life

### Creates Tension
- Players with 40k+ current score feel pressure
- Risk banking now (intentional death) vs continuing for more
- Leaderboard climbs require sustained play

### Prevents Exploits
- No combat logging
- No logout-to-escape
- No score preservation exploits

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Corpse duration | 5 min | Crash recovery possible, doesn't clutter world |
| Score banking | On every logout | No safe preserve, forces commitment |
| Spawn location | Random/fixed | Can't use logout for teleport |

---

## Implementation Notes

### On Disconnect Event

```typescript
function onPlayerDisconnect(playerId: number) {
  // 1. Process as death
  processPlayerDeath(playerId, null) // No killer

  // 2. Drop loot
  dropInventoryAsCorpse(playerId, {
    duration: 300000, // 5 min
    lootable: true
  })

  // 3. Save score to leaderboard (already done in processPlayerDeath)

  // 4. Remove player entity
  removePlayerEntity(playerId)
}
```

### On Login

```typescript
function onPlayerLogin(playerId: number) {
  // 1. Spawn fresh character
  spawnPlayer(playerId, {
    items: [], // Empty inventory
    equipped: null, // No equipped items
    hp: 100, // Full HP
    hunger: 100,
    thirst: 100
  })

  // 2. Leaderboard score intact (persistent)
  loadPlayerScore(playerId) // Loads leaderboardScore, resets currentScore to 0
}
```

---

## Integration Points

**Depends On:**
- Score System - Death banking, leaderboard updates
- Health System - Death processing
- Inventory System - Loot dropping
- Spawn System - Fresh character spawn

**Used By:**
- None (terminal system)

**Conflicts:**
- None (works with all systems)

---

## See Also

- [SCORE_SYSTEM.md](./SCORE_SYSTEM.md) - Death banking, score persistence
- [GAME_DESIGN.md](./../GAME_DESIGN.md) - Design philosophy (no safe logout)
