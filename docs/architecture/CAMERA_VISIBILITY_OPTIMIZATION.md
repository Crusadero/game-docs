# Camera Visibility Optimization

## Problem Statement

With 200-300 concurrent players, each having their own camera tracking visible entities, the camera visibility system becomes a major performance bottleneck.

**Current Issue:**
- 200 cameras (one per player)
- Each camera needs to track which entities are visible
- Entities entering/leaving camera view need to be detected
- Using Box2D sensors for cameras: **40-60ms per tick**
- Target: **<5ms per tick**

---

## Why Box2D Camera Sensors Are Wrong

### What Box2D Is Designed For:
- Physics simulation (forces, collisions, momentum)
- Continuous collision detection
- Resolving physical contacts

### What We Need:
- Spatial queries ("what entities are in this rectangle?")
- Static visibility checks (camera doesn't interact physically)
- Change detection (entity entered/left view)

### The Mismatch:

Box2D treats 200 camera sensors as 200 additional physics bodies:
```
Every physics step:
├─ 200 camera sensors
├─ 10,000 static bodies (trees, rocks, etc.)
├─ 300 dynamic entities (players, mobs)
└─ Box2D checks ALL against ALL
   ├─ Broad phase: AABB tree traversal
   ├─ Narrow phase: Precise collision checks
   └─ Contact callbacks for enter/leave

Complexity: O(cameras × total_entities)
Result: 200 × 10,300 = ~2,000,000 potential checks per tick
Performance: 40-60ms per tick
```

**This is like using a sledgehammer to crack a nut.**

---

## Solution: Spatial Grid + Dynamic Region Merging

### Core Concepts

#### 1. Spatial Grid
Divide the world into a grid of cells. Each cell tracks which entities are in it.

```
Grid Cell Size: 1000×1000 pixels

World divided into cells:
┌────────┬────────┬────────┐
│ Cell   │ Cell   │ Cell   │
│ [0,0]  │ [1,0]  │ [2,0]  │
│        │        │        │
├────────┼────────┼────────┤
│ Cell   │ Cell   │ Cell   │
│ [0,1]  │ [1,1]  │ [2,1]  │
│        │        │        │
└────────┴────────┴────────┘

Each cell knows: "Which entities are in me?"
Cell [1,1]: {entity_5, entity_23, entity_47}
```

**Benefit:** Finding entities in an area = look up relevant cells (constant time per cell).

#### 2. Dynamic Region Merging
Instead of querying each camera individually, merge overlapping cameras into regions and query once per region.

```
Step 1: Detect Camera Overlap
  Camera 1 bounds: [100, 100] → [2020, 1180]
  Camera 2 bounds: [150, 120] → [2070, 1200]
  → These overlap! Merge into one region.

Step 2: Merge Bounds
  Region A merged bounds: [100, 100] → [2070, 1200]
  (covers both cameras)

Step 3: Query Once
  Query spatial grid for region A → 500 entities

Step 4: Distribute
  Give 500 entities to both cameras
  Each camera filters: "Which are in MY specific bounds?"
```

---

## The Algorithm

### Overview

```
For each game tick:
  1. Detect camera overlaps
  2. Merge overlapping cameras into regions
  3. Query spatial grid once per region
  4. Distribute results to cameras
  5. Cameras do cheap AABB filtering
```

### Detailed Steps

#### Step 1: Detect Camera Overlaps

For 200 cameras, determine which ones have overlapping view rectangles.

**Input:**
```typescript
Camera 1: { minX: 100, minY: 100, maxX: 2020, maxY: 1180 }
Camera 2: { minX: 150, minY: 120, maxX: 2070, maxY: 1200 }
Camera 3: { minX: 5000, minY: 3000, maxX: 7000, maxY: 4080 }
```

**Overlap Check:**
```
Camera A overlaps Camera B if:
  A.minX < B.maxX AND
  A.maxX > B.minX AND
  A.minY < B.maxY AND
  A.maxY > B.minY
```

**Algorithm:** Union-Find (Disjoint Set)
- Efficiently groups overlapping cameras
- O(N × α(N)) ≈ O(N) where N = camera count
- For 200 cameras: ~200 operations

**Output:**
```
Group 1: {camera_1, camera_2, camera_4, ..., camera_180}  // City cluster
Group 2: {camera_181, camera_182, ..., camera_195}        // Forest cluster
Group 3: {camera_196}                                      // Alone
Group 4: {camera_197}                                      // Alone
...
```

#### Step 2: Calculate Merged Bounds

For each group, calculate the bounding rectangle that encompasses all cameras.

```typescript
Group 1 (180 cameras):
  mergedBounds = {
    minX: min(camera_1.minX, camera_2.minX, ..., camera_180.minX),
    maxX: max(camera_1.maxX, camera_2.maxX, ..., camera_180.maxX),
    minY: min(camera_1.minY, camera_2.minY, ..., camera_180.minY),
    maxY: max(camera_1.maxY, camera_2.maxY, ..., camera_180.maxY),
  }

Example result:
  mergedBounds = { minX: 100, minY: 100, maxX: 2500, maxY: 2000 }
```

#### Step 3: Query Spatial Grid

For each merged region, query the spatial grid once.

```typescript
For merged region [100, 100] → [2500, 2000]:

  1. Determine which grid cells overlap this region:
     cellMinX = floor(100 / 1000) = 0
     cellMaxX = floor(2500 / 1000) = 2
     cellMinY = floor(100 / 1000) = 0
     cellMaxY = floor(2000 / 1000) = 2

     Cells to check: [0,0], [1,0], [2,0], [0,1], [1,1], [2,1], [0,2], [1,2], [2,2]
     Total: 9 cells

  2. Collect entities from these cells:
     entities = union of all entities in those 9 cells

  3. Cache result for this region

Result: ~500 entities for this region
```

#### Step 4: Distribute to Cameras

Give the region's entities to all cameras in that region.

```typescript
For each camera in Group 1:
  camera.candidateEntities = region1.entities  // 500 entities
```

#### Step 5: Per-Camera AABB Filtering

Each camera filters the candidate entities to only those in its specific viewport.

```typescript
For camera with bounds [100, 100] → [2020, 1180]:
  visibleEntities = []

  For each entity in candidateEntities (500 total):
    entityPos = getEntityPosition(entity)

    // Simple AABB check (4 comparisons)
    if (entityPos.x >= 100 AND entityPos.x <= 2020 AND
        entityPos.y >= 100 AND entityPos.y <= 1180):
      visibleEntities.add(entity)

Result: ~200 visible entities for this specific camera
```

---

## Spatial Grid Updates

The spatial grid only needs to be updated when entities change cells.

### Event 1: Entity Moved (After Physics Step)

```typescript
After b2World.Step():
  For each dynamic entity:
    oldPosition = entity.previousPosition
    newPosition = entity.currentPosition

    if (oldPosition != newPosition):
      oldCell = getCellForPosition(oldPosition)
      newCell = getCellForPosition(newPosition)

      if (oldCell != newCell):
        // Entity changed cells
        spatialGrid[oldCell].remove(entityId)
        spatialGrid[newCell].add(entityId)
```

**Frequency:** Only when entity crosses cell boundary
**Cost:** O(1) - simple set operations

### Event 2: Entity Created

```typescript
When entity spawns:
  position = entity.position
  cell = getCellForPosition(position)

  spatialGrid[cell].add(entityId)
```

### Event 3: Entity Destroyed

```typescript
When entity removed:
  position = entity.position
  cell = getCellForPosition(position)

  spatialGrid[cell].remove(entityId)
```

---

## Performance Analysis

### Scenario 1: 200 Cameras Clustered (City)

**Setup:**
- 200 cameras all in same area (1000×1000 region)
- All cameras overlap significantly
- ~500 entities in the area

**Processing:**
```
Step 1: Detect overlaps
  - 200 cameras checked
  - All overlap
  - Result: 1 merged group
  Cost: ~200 comparisons = 0.1ms

Step 2: Merge bounds
  - Find min/max of 200 cameras
  Cost: ~200 comparisons = 0.1ms

Step 3: Query spatial grid
  - Query merged bounds: [0, 0] → [3000, 3000]
  - Overlaps ~9 grid cells
  - Returns ~500 entities
  Cost: 9 cell lookups = 0.2ms

Step 4: Distribute
  - Give 500 entities to all 200 cameras
  Cost: Negligible (just references)

Step 5: Filter per camera
  - 200 cameras × 500 entities × AABB check
  - AABB check = 4 comparisons (very fast)
  Cost: 100k comparisons = 1-2ms

Total: ~2.5ms
```

**Comparison:**
- Box2D sensors: 40-60ms
- Improvement: **16-24x faster**

### Scenario 2: 200 Cameras Spread Out

**Setup:**
- 200 cameras distributed across map
- Minimal overlap
- ~50 merged groups (avg 4 cameras per group)
- ~30 entities per camera viewport

**Processing:**
```
Step 1: Detect overlaps
  - 200 cameras checked
  - Found 50 groups
  Cost: ~200 comparisons = 0.1ms

Step 2: Merge bounds (50 groups)
  Cost: ~200 operations = 0.1ms

Step 3: Query spatial grid (50 queries)
  - Each query overlaps ~4 cells
  - Each returns ~30-50 entities
  Cost: 50 queries × 4 cells = 200 lookups = 0.5ms

Step 4: Distribute
  Cost: Negligible

Step 5: Filter per camera
  - 200 cameras × ~40 entities avg
  Cost: 8k comparisons = 0.3ms

Total: ~1ms
```

**Comparison:**
- Box2D sensors: 40-60ms
- Improvement: **40-60x faster**

### Scenario 3: Mixed Distribution

**Setup:**
- 150 cameras in city (1 large cluster)
- 30 cameras in forest (1 medium cluster)
- 20 cameras scattered (20 individual regions)

**Processing:**
```
Total regions: 22
Total queries: 22

City cluster:
  - 1 query → 500 entities
  - 150 cameras × 500 = 75k AABB checks

Forest cluster:
  - 1 query → 100 entities
  - 30 cameras × 100 = 3k AABB checks

Scattered:
  - 20 queries → 20-30 entities each
  - 20 cameras × 25 avg = 500 AABB checks

Total cost: ~3ms
```

---

## When to Use Each Approach

### Notification-Based (Event-Driven)

**Best for:** Sparse player distribution

```
When entity moves:
  - Look up which cameras are watching relevant grid cells
  - Notify only those cameras

When camera moves:
  - Look up which entities are in new grid cells
  - Update camera's visible set
```

**Pros:**
- Only processes changes (very efficient when sparse)
- No wasted work

**Cons:**
- Breaks down when too many cameras in same area (too many notifications)

**Use when:** < 10 cameras per region

### Query-Based with Region Merging (This Solution)

**Best for:** Dense player distribution or mixed scenarios

```
Every tick:
  - Merge overlapping cameras
  - Query once per region
  - Distribute to cameras
```

**Pros:**
- Adapts to any distribution
- Optimal for clustering (1 query for many cameras)
- Predictable performance

**Cons:**
- Slight overhead when very sparse (but still fast)

**Use when:** ≥ 10 cameras per region, or unpredictable distribution

### Hybrid Approach

**Best for:** Production (handles all cases)

```
Classify each region:
  If (cameras_in_region < 10):
    Use notification-based for this region
  Else:
    Use query-based with merging for this region
```

**Pros:**
- Best performance in all scenarios
- Automatically adapts

**Cons:**
- More complex implementation

---

## Implementation Overview

### Core Components

#### 1. SpatialGridManager
```
Responsibilities:
  - Maintain grid of cells → entities
  - Update entity positions (add/remove/move)
  - Query entities in area

API:
  - addEntity(entityId, x, y)
  - removeEntity(entityId)
  - updateEntity(entityId, oldX, oldY, newX, newY)
  - queryArea(minX, minY, maxX, maxY): EntityId[]
```

#### 2. CameraRegionManager
```
Responsibilities:
  - Detect camera overlaps
  - Merge cameras into regions
  - Query spatial grid per region
  - Distribute results to cameras

API:
  - registerCamera(cameraId, bounds)
  - updateCameras(): void
  - getCameraVisibleEntities(cameraId): EntityId[]
```

#### 3. ClientCamera
```
Responsibilities:
  - Track visible entities
  - Detect enter/leave events
  - Send updates to client

API:
  - setVisibleEntities(entities: EntityId[])
  - flush(): void  // Send network updates
```

### Integration Points

#### In NormalWorld.update() (After Physics):
```typescript
// After physics step
for (entity of dynamicEntities) {
  if (entity.positionChanged) {
    SpatialGridManager.updateEntity(entity.id, oldPos, newPos);
  }
}
```

#### In ServerManager.update() (Camera Updates):
```typescript
// Update camera regions
CameraRegionManager.updateCameras();

// Flush camera updates to network
for (client of clients) {
  client.camera.flush();
}
```

#### On Entity Spawn/Remove:
```typescript
// On spawn
SpatialGridManager.addEntity(entityId, x, y);

// On remove
SpatialGridManager.removeEntity(entityId);
```

---

## Configuration

### Recommended Settings

```typescript
// Grid cell size (pixels)
CELL_SIZE = 1000
// Larger cells = fewer cells to check, but less precise
// Smaller cells = more cells to check, but more precise
// 1000px works well for typical viewport sizes (1920×1080)

// Camera overlap threshold for merging
MERGE_OVERLAP_THRESHOLD = 0.5
// 0.0 = cameras must fully overlap to merge
// 1.0 = cameras merge if they touch at all
// 0.5 = cameras merge if they overlap by 50%+ (good default)

// Region query optimization
MAX_CAMERAS_PER_QUERY = 100
// If more than this many cameras in a region, consider splitting
// Prevents one giant query for 200 cameras
```

---

## Performance Targets

| Player Count | Camera Count | Expected Camera Update Time | Approach |
|--------------|--------------|---------------------------|----------|
| 0-50 | 0-50 | < 0.5ms | Simple queries |
| 50-100 | 50-100 | < 1ms | Region merging |
| 100-200 | 100-200 | < 2ms | Region merging |
| 200-300 | 200-300 | < 3ms | Region merging + hybrid |

**Compared to Box2D sensors:** 40-60ms → **15-40x improvement**

---

## Memory Usage

### Spatial Grid
```
Grid cells: map_width / cell_size × map_height / cell_size
For 20,000 × 20,000 world with 1000px cells:
  = 20 × 20 = 400 cells

Each cell: Set of entity IDs
Average entities per cell: ~25
Memory: 400 cells × 25 entities × 8 bytes = ~80KB

Total: Negligible
```

### Camera Regions
```
Per-frame allocation:
  - Region bounds: ~10-50 regions × 32 bytes = ~1KB
  - Entity lists: ~2-5KB (references only)

Total: < 10KB per frame (garbage collected after)
```

**Overall memory overhead:** Minimal (~100KB total)

---

## Edge Cases & Solutions

### Edge Case 1: Camera Spanning Multiple Cells

**Problem:** Large viewport spans many grid cells

**Solution:** Query all overlapping cells (already handled)
```
Camera viewport: 1920×1080
Cell size: 1000×1000
Overlaps: 4-6 cells typically
Cost: 6 cell lookups (still very fast)
```

### Edge Case 2: Entity on Cell Boundary

**Problem:** Entity at x=999 might be in wrong cell

**Solution:** Consistent rounding
```
cellX = floor(x / cellSize)
Always rounds down, deterministic
```

### Edge Case 3: Very Fast-Moving Entity

**Problem:** Entity teleports/moves very fast, skips cells

**Solution:** Not a problem for visibility!
```
updateEntity() only cares about final position
Doesn't matter how entity got there
Grid updated correctly after physics step
```

### Edge Case 4: Camera Moves Every Frame

**Problem:** Camera follows player, moves constantly

**Solution:** Only recompute regions if cameras moved significantly
```
Track last region computation position
Only recompute if camera moved > threshold (e.g., 500px)
Most frames: reuse cached regions
```

---

## Debugging & Monitoring

### Performance Metrics to Track

```typescript
metrics = {
  // Per tick
  cameraUpdateTime: 0,          // Total time for camera system
  regionCount: 0,               // How many merged regions
  queriesPerformed: 0,          // Spatial grid queries
  entitiesChecked: 0,           // Total entities checked

  // Per camera
  avgVisibleEntities: 0,        // Average entities per camera
  maxVisibleEntities: 0,        // Max entities any camera sees

  // Grid stats
  avgEntitiesPerCell: 0,        // Grid density
  maxEntitiesPerCell: 0,        // Hotspot detection
}
```

### Performance Warning Thresholds

```
If cameraUpdateTime > 10ms:
  → WARNING: Camera system slow, investigate

If regionCount < 5 AND cameraCount > 100:
  → All cameras clustered, consider optimizations

If maxEntitiesPerCell > 100:
  → Grid cell too large, consider smaller cells

If avgVisibleEntities > 200:
  → Too many entities visible, consider culling
```

---

## Migration from Box2D Sensors

### Step 1: Add Spatial Grid (Parallel)
- Keep Box2D sensors running
- Add spatial grid alongside
- Compare results (should match)

### Step 2: Switch Camera Queries
- Cameras read from spatial grid instead of Box2D
- Box2D sensors still update (but not used)

### Step 3: Remove Box2D Sensors
- Once validated, remove sensor bodies
- Huge performance win unlocked

### Rollback Plan
- If issues found, toggle flag to use Box2D temporarily
- Fix spatial grid
- Re-enable

---

## See Also

- [Box2D C++ Migration Guide](./BOX2D_CPP_MIGRATION.md) - For further physics optimization
- [Movement Restrictions](./../systems/MOVEMENT_RESTRICTIONS.md) - Mobility system design
- [Entity Action & Status System](./ENTITY_ACTION_STATUS_SYSTEM.md) - Entity state management

---

## Summary

**Problem:** 200 Box2D camera sensors = 40-60ms per tick

**Solution:** Spatial grid + dynamic region merging

**Benefits:**
- ✅ **15-40x faster** (2-4ms per tick)
- ✅ Adapts to player distribution (sparse or dense)
- ✅ Minimal memory overhead (<100KB)
- ✅ Simple to implement
- ✅ Easy to debug and monitor

**Key Innovation:** Merge overlapping cameras before querying - query once, share results.

This approach scales efficiently from 10 to 300+ cameras while maintaining consistent, predictable performance.
