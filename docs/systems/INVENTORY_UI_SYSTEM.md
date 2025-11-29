# Inventory UI System - Button Transfer & Visual Feedback

## Quick Reference

- Button-based transfers: Faster than drag-and-drop for PvP (Click/Ctrl/Ctrl+Shift = 1/10/100 items)
- Split-screen containers: Chest above, player inventory below, health bars always visible
- Radial cooldown display: Non-blocking outline animation on items (item/count visible)
- Category cooldowns: Single cooldown affects multiple items (food, medical)
- Buff icons above status bars: Max 8 visible, debuffs prioritized
- Items on cooldown can still be moved/dropped (not blocked)

---

## Overview

**What:** Fast, button-based inventory system with clear visual feedback for transfers, cooldowns, and status effects, optimized for competitive PvP.

**Why:** Drag-and-drop is slow for PvP, button transfers with modifiers are faster/precise, radial cooldowns don't block item visibility, split-screen keeps health bars visible during looting.

**Key Rule:** Speed and precision over complexity - every UI element must be fast to read and interact with

---

## Core Mechanics

### Transfer System

| Input | Amount | Use Case |
|-------|--------|----------|
| **Click** | 1 item | Precise single-item transfer |
| **Ctrl + Click** | 10 items | Medium stack transfer |
| **Ctrl + Shift + Click** | 100 items | Large stack transfer |
| **Shift + F** | All items | Loot all from container |
| **Shift + R** | All matching | Store all matching items |
| **Shift + Q** | Smart stack | Stack matching items to container |

### Cooldown Types

| Type | Duration | Affects | Color |
|------|----------|---------|-------|
| **Equip** | 500ms-1s | Specific weapon/tool | Orange |
| **Build** | Varies | Same structure type | Yellow |
| **ConsumeFood** | 1-2s | All food items | Green |
| **ConsumeMedical** | 4-10s | All medical items | Blue |
| **ConsumeSpecific** | 30s+ | Specific rare item | Purple |

### Buff Display

| Category | Color | Priority | Max Duration Display |
|----------|-------|----------|---------------------|
| **Positive** (heal, speed, defense) | Green/Blue | Low | Show if >5s |
| **Negative** (bleed, slow, burn) | Red/Orange | High | Always show |
| **Temperature** (warmth, freeze) | Fire/Ice | Medium | Show if >5s |

---

## UI Layout

### Container View (Chest Open)

```
┌──────────────────────────────────────────┐
│      WOODEN CHEST (15/20)                │
│  [↓] [↓] [↓] [↓] [↓]  ← Take buttons    │
│ ┌────┬────┬────┬────┬────┐              │
│ │Wood│Stone│Sword│Axe│[..]│              │
│ │x1000│x1000│ x1│ x1│    │              │
│ └────┴────┴────┴────┴────┘              │
└──────────────────────────────────────────┘
              ╔═══════╗
┌──────────────────────────────────────────┐
│ ┌────┬────┬────┬────┬────┐              │
│ │Wood│Stone│Sword│[..]│[..]│              │
│ │x1000│x1000│ x1│    │    │              │
│ └────┴────┴────┴────┴────┘              │
│  [↑] [↑] [↑] [↑] [↑]  ← Give buttons    │
│      YOUR INVENTORY (8/10)               │
└──────────────────────────────────────────┘

[🛡️][⚡] ❤️ ████░ 🥩 ████░ 💧 ████░ ← Always visible
```

**Design:** Health/status never hidden, container above inventory (natural flow), semi-transparent background (world visible)

---

## Cooldown Display

### Visual Design

**Radial Progress Outline:**
- Item icon fully visible (no overlay)
- Circular outline animates clockwise (100% → 0%)
- Timer text only if >3s remaining
- Item can still be moved/dropped

```
Normal Item:         Item on Cooldown:
┌────────┐           ╔════════╗
│ [Icon] │           ║ [Icon] ║  ← RED outline
│ x1000  │           ║ x1000  ║     (animating)
└────────┘           ╚════════╝
                        2.3s
```

### Category Cooldown Behavior

**Food consumed:**
- Server sends: `{ type: 'COOLDOWN_START', cooldownType: ConsumeFood, durationMs: 1500 }`
- Client shows outline on ALL food items in inventory
- Duration: 1.5s

**Medical item consumed:**
- Server sends: `{ type: 'COOLDOWN_START', cooldownType: ConsumeMedical, durationMs: 4000 }`
- Client shows outline on ALL medical items
- Duration: 4s

**Interaction during cooldown:**
- Click to use → Show error message + shake animation
- Drag to move → Allowed (can reorganize, drop, store)
- Transfer buttons → Allowed (can move to chest)

---

## Buff Icons

### Placement

```
     [🛡️] [⚡] [💊] [🔥]  ← Buff icons (above status bars)
┌──────────────────────────────────────────┐
│ ❤️ ████░  🥩 ████░  💧 ████░  ❄️ ████░ │
└──────────────────────────────────────────┘
```

### Icon Structure

```
┌────┐
│ 🛡️ │  ← Icon (32x32)
│ x3 │  ← Stack count (if >1)
│ 10s│  ← Timer (if >5s)
└────┘
```

### Priority & Overflow

**Max visible:** 8 icons
**Priority order:** Debuffs first, then buffs
**Overflow:** Show "..." if more than 8

**NOT shown as buffs:**
- ❌ Cooldowns (shown on items)
- ❌ Action states (attacking, consuming)
- ❌ Equipment (shown in hotbar)

---

## Integration Points

**Depends On:**
- Inventory System - Item slots, stack counts, transfers
- Cooldown System - Category and item-specific cooldowns
- Status Effect System - Buffs/debuffs for icon display
- Entity Action System - Blocks consume during cooldown

**Used By:**
- Container UI - Chest/storage interaction
- Hotbar UI - Quick access slots
- Crafting UI - Workbench/crafting stations

**Conflicts:**
- None (UI layer)

---

## Edge Cases

**Q: What if chest is full when transferring?**
A: Transfer rejected, show error message, play error sound, shake button

**Q: Can you move items on cooldown?**
A: Yes - can move, drop, store. Cannot use/equip/consume.

**Q: What if more than 8 buffs active?**
A: Show top 8 by priority (debuffs first), "..." for overflow

**Q: What if container closes mid-transfer?**
A: Transfer cancelled, no items moved, return to normal inventory view

**Q: What happens to cooldown if item is dropped?**
A: Cooldown persists with item (server-tracked, not UI state)

**Q: Can you transfer partial stacks?**
A: Yes - use modifiers (1/10/100), clamped to available amount

---

## Visual Feedback

### Transfer Animation

**Item flight:**
1. Create flying icon from source slot
2. Animate to destination (300ms, easeOutQuad)
3. Spawn particle trail behind icon
4. Flash destination slot on arrival
5. Update stack counts with number animation

**Sound feedback:**
- 1 item: soft_pop.wav
- 10 items: chunk.wav
- 100 items: ka_chunk.wav
- Error: error_click.wav

### Button States

| State | Visual | Sound |
|-------|--------|-------|
| **Normal** | Subtle glow, 100% opacity | - |
| **Hover** | Bright glow, scale 1.1x | ui_hover |
| **Click** | Scale 0.9x, sparkle particles | ui_click |
| **Disabled** | Grey, 50% opacity | - |
| **Error** | Red flash, shake animation | error_click |

### Cooldown Ready Feedback

**When cooldown expires:**
- Play sound: cooldown_ready.wav (satisfying chime)
- Outline fades out
- Item slot briefly flashes green

---

## Network Protocol

### Cooldown Packets

```typescript
// Server → Client: Start cooldown
{ type: 'COOLDOWN_START', cooldownType: CooldownType, durationMs: number, itemId?: number }

// Server → Client: Cooldown ended
{ type: 'COOLDOWN_END', cooldownType: CooldownType, itemId?: number }
```

### Transfer Packets

```typescript
// Client → Server: Transfer request
{ type: 'TRANSFER_ITEM', containerId: number, direction: 'to_container' | 'from_container',
  sourceSlotIndex: number, amount: number }

// Server → Client: Transfer result
{ type: 'TRANSFER_RESULT', success: boolean, reason?: string,
  sourceSlot: number, destinationSlot: number, amountTransferred: number }
```

---

## Implementation Notes

### Cooldown Manager (Client)

```typescript
class CooldownManager {
  private activeCooldowns: Map<string, CooldownState> = new Map();

  isOnCooldown(itemId: number): boolean {
    const item = ItemsList[itemId];

    // Check item-specific cooldown
    if (this.activeCooldowns.has(`specific_${itemId}`)) return true;

    // Check category cooldowns
    if (item.isConsumable) {
      if (item.itemSubTypeBitMask & ConsumableSubType.Food) {
        if (this.activeCooldowns.has('category_food')) return true;
      }
      if (item.itemSubTypeBitMask & ConsumableSubType.Medical) {
        if (this.activeCooldowns.has('category_medical')) return true;
      }
    }

    return false;
  }
}
```

### Radial Progress Drawing

```typescript
drawItemSlot(slot) {
  drawSprite(slot.item.icon);
  drawText(slot.amount);

  if (slot.cooldownProgress > 0) {
    drawRadialProgress({
      center: slot.center,
      radius: slot.size / 2,
      progress: slot.cooldownProgress,
      color: slot.cooldownColor,
      thickness: 3,
      startAngle: -90,  // Start at top
      clockwise: true
    });

    if (slot.cooldownRemaining > 3000) {
      const seconds = (slot.cooldownRemaining / 1000).toFixed(1);
      drawText(`${seconds}s`, slot.bottomRight);
    }
  }
}
```

---

## Balance Targets

| Metric | Target | Reasoning |
|--------|--------|-----------|
| Transfer animation | 300ms | Fast enough for PvP, visible feedback |
| Cooldown outline thickness | 3px | Visible but not intrusive |
| Buff icon size | 32x32px | Readable, doesn't block view |
| Max visible buffs | 8 | Enough for most cases, prevents clutter |
| Button hover scale | 1.1x | Noticeable without being distracting |
| Error shake duration | 100ms | Quick feedback, not annoying |

---

## See Also

- [Consumable System](./CONSUMABLE_SYSTEM.md) - Item usage timing, category cooldowns
- [Building System](./BUILDING_SYSTEM.md) - Structure placement cooldowns
- [Attack System](./ATTACK_SYSTEM.md) - Weapon equip cooldowns
- [Item Animation System](./../architecture/ITEM_ANIMATION_SYSTEM.md) - Item visual configs
