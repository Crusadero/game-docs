# Box2D C++ Migration Guide

## Overview

This document outlines strategies for migrating from TypeScript Box2D (`@box2d/core`) to native C++ Box2D for improved performance when scaling to 200+ concurrent players.

**Current Status:**
- Using `@box2d/core` (pure TypeScript/JavaScript)
- Target: 14 TPS currently, scalable to 20-60 TPS
- Minimal Box2D API usage (CreateBody, DestroyBody, Step, Queries)

---

## Performance Comparison

| Implementation | Physics Step (200 entities) | Static Body Creation (10k) | Overall Speedup |
|----------------|----------------------------|---------------------------|-----------------|
| @box2d/core (JS) | 20-50ms | 500-1000ms | 1x (baseline) |
| box2d-wasm | 6-15ms | 150-300ms | **3-5x** |
| N-API Native | 3-8ms | 50-100ms | **6-10x** |
| C++ Worker Process | 2-6ms | 40-80ms | **8-12x** |

---

## Migration Options

### Option 1: box2d-wasm (Recommended First Step)

**Description:** WebAssembly compiled from C++ Box2D, runs in Node.js V8.

**Installation:**
```bash
npm install box2d-wasm
```

**Implementation:**
```typescript
// Before (TypeScript Box2D)
import { b2World, b2Vec2, b2BodyType } from '@box2d/core';

const world = new b2World(new b2Vec2(0, 0));

// After (WASM Box2D)
import Box2D from 'box2d-wasm';

const box2d = await Box2D();
const world = new box2d.b2World(new box2d.b2Vec2(0, 0));
```

**Pros:**
- ✅ **3-5x performance improvement**
- ✅ Easy integration (minimal code changes)
- ✅ Cross-platform (no compilation needed)
- ✅ Nearly identical API to TypeScript version
- ✅ Still synchronous (blocking)

**Cons:**
- ⚠️ Async initialization (one-time on startup)
- ⚠️ Not as fast as true native
- ⚠️ Still runs in V8 (has some overhead)

**When to Use:**
- Current player count: 100-200
- Tick time: >16ms consistently
- Want quick win without complex build setup

**Migration Effort:** **Low** (1-2 days)

---

### Option 2: N-API Native Addon (Maximum Performance)

**Description:** Direct C++ Box2D bindings using Node.js N-API for zero-overhead synchronous calls.

**Setup:**
```bash
npm install node-addon-api
npm install --save-dev cmake-js node-gyp
```

**Project Structure:**
```
/native
  /src
    physics-world.cpp       # C++ Box2D wrapper
    physics-world.h
  binding.gyp               # Build configuration
  CMakeLists.txt
/src
  /ecs-module
    /systems
      physics-bridge.ts     # TypeScript interface
```

**Implementation:**

**C++ Side (physics-world.cpp):**
```cpp
#include <napi.h>
#include <box2d/box2d.h>
#include <vector>

class PhysicsWorld : public Napi::ObjectWrap<PhysicsWorld> {
private:
  b2World* world;
  std::vector<b2Body*> bodies;

public:
  PhysicsWorld(const Napi::CallbackInfo& info)
    : ObjectWrap(info) {
    world = new b2World(b2Vec2(0.0f, 0.0f));
  }

  ~PhysicsWorld() {
    delete world;
  }

  // Synchronous physics step
  Napi::Value Step(const Napi::CallbackInfo& info) {
    Napi::Env env = info.Env();

    // Input: Array of entity data
    Napi::Array entities = info[0].As<Napi::Array>();

    // Apply forces from game state
    for (uint32_t i = 0; i < entities.Length(); i++) {
      Napi::Object entity = entities.Get(i).As<Napi::Object>();

      int32_t bodyIndex = entity.Get("bodyIndex").ToNumber().Int32Value();
      float dirX = entity.Get("dirX").ToNumber().FloatValue();
      float dirY = entity.Get("dirY").ToNumber().FloatValue();
      float speed = entity.Get("speed").ToNumber().FloatValue();

      b2Body* body = bodies[bodyIndex];

      // Apply impulse
      b2Vec2 velocity(dirX * speed, dirY * speed);
      b2Vec2 impulse = (velocity - body->GetLinearVelocity()) * body->GetMass();
      body->ApplyLinearImpulseToCenter(impulse, true);
    }

    // Step physics
    world->Step(1.0f / 20.0f, 8, 3);

    // Return results
    Napi::Array results = Napi::Array::New(env, entities.Length());
    for (uint32_t i = 0; i < bodies.size(); i++) {
      b2Vec2 pos = bodies[i]->GetPosition();

      Napi::Object result = Napi::Object::New(env);
      result.Set("x", Napi::Number::New(env, pos.x));
      result.Set("y", Napi::Number::New(env, pos.y));
      results.Set(i, result);
    }

    return results;
  }

  Napi::Value CreateBody(const Napi::CallbackInfo& info) {
    Napi::Env env = info.Env();

    Napi::Object config = info[0].As<Napi::Object>();

    b2BodyDef bodyDef;
    bodyDef.type = b2_dynamicBody;
    bodyDef.position.Set(
      config.Get("x").ToNumber().FloatValue(),
      config.Get("y").ToNumber().FloatValue()
    );

    b2Body* body = world->CreateBody(&bodyDef);
    bodies.push_back(body);

    return Napi::Number::New(env, bodies.size() - 1);
  }

  Napi::Value QueryAABB(const Napi::CallbackInfo& info) {
    Napi::Env env = info.Env();

    float minX = info[0].ToNumber().FloatValue();
    float minY = info[1].ToNumber().FloatValue();
    float maxX = info[2].ToNumber().FloatValue();
    float maxY = info[3].ToNumber().FloatValue();

    b2AABB aabb;
    aabb.lowerBound.Set(minX, minY);
    aabb.upperBound.Set(maxX, maxY);

    std::vector<int32_t> foundBodies;

    class QueryCallback : public b2QueryCallback {
    public:
      std::vector<int32_t>& bodies;
      std::vector<b2Body*>& bodyList;

      QueryCallback(std::vector<int32_t>& b, std::vector<b2Body*>& bl)
        : bodies(b), bodyList(bl) {}

      bool ReportFixture(b2Fixture* fixture) override {
        b2Body* body = fixture->GetBody();
        for (size_t i = 0; i < bodyList.size(); i++) {
          if (bodyList[i] == body) {
            bodies.push_back(i);
            break;
          }
        }
        return true;
      }
    };

    QueryCallback callback(foundBodies, bodies);
    world->QueryAABB(&callback, aabb);

    Napi::Array results = Napi::Array::New(env, foundBodies.size());
    for (size_t i = 0; i < foundBodies.size(); i++) {
      results.Set(i, Napi::Number::New(env, foundBodies[i]));
    }

    return results;
  }

  static Napi::Function GetClass(Napi::Env env) {
    return DefineClass(env, "PhysicsWorld", {
      InstanceMethod("step", &PhysicsWorld::Step),
      InstanceMethod("createBody", &PhysicsWorld::CreateBody),
      InstanceMethod("queryAABB", &PhysicsWorld::QueryAABB),
    });
  }
};

Napi::Object Init(Napi::Env env, Napi::Object exports) {
  Napi::String name = Napi::String::New(env, "PhysicsWorld");
  exports.Set(name, PhysicsWorld::GetClass(env));
  return exports;
}

NODE_API_MODULE(physics, Init)
```

**TypeScript Bridge (physics-bridge.ts):**
```typescript
import { PhysicsWorld } from '../../../build/Release/physics.node';

export class PhysicsBridge {
  private static nativeWorld: any = null;

  static initialize(): void {
    this.nativeWorld = new PhysicsWorld();
  }

  static step(entities: Array<{ bodyIndex: number; dirX: number; dirY: number; speed: number }>):
    Array<{ x: number; y: number }> {

    // SYNCHRONOUS call to C++ - blocks until complete
    return this.nativeWorld.step(entities);
  }

  static createBody(x: number, y: number): number {
    return this.nativeWorld.createBody({ x, y });
  }

  static queryAABB(minX: number, minY: number, maxX: number, maxY: number): number[] {
    return this.nativeWorld.queryAABB(minX, minY, maxX, maxY);
  }
}
```

**Usage in NormalWorld:**
```typescript
// normal-world.entity.ts
import { PhysicsBridge } from '../../ecs-module/systems/physics-bridge';

protected update(): void {
  const dynamicEntities = Q_DynamicEntities(this);

  // Prepare input for C++
  const physicsInput = [];
  for (let i = 0; i < dynamicEntities.length; i++) {
    const entityId = dynamicEntities[i];
    const bodyIndex = C_EntityBody.index[entityId];
    const userData = this.bodies[bodyIndex].GetUserData() as DynamicBodyUserData;

    const baseSpeed = C_DynamicEntity.baseSpeed[entityId];
    const speedMultiplier = MobilitySystem.getSpeedMultiplier(entityId);
    const finalSpeed = Math.min(baseSpeed * speedMultiplier, C_DynamicEntity.maxSpeed[entityId]);

    physicsInput.push({
      bodyIndex,
      dirX: userData.directionVector.x,
      dirY: userData.directionVector.y,
      speed: finalSpeed,
    });
  }

  // SYNCHRONOUS C++ call (blocks for ~1-5ms)
  const results = PhysicsBridge.step(physicsInput);

  // Apply results
  for (let i = 0; i < results.length; i++) {
    const entityId = dynamicEntities[i];
    const bodyIndex = C_EntityBody.index[entityId];

    // Update body positions from C++ results
    this.bodies[bodyIndex].SetPosition(results[i].x, results[i].y);
  }

  // Continue with game logic
  AttackableSystem.update(this);
  EntityStatusEffectSystem.update();
}
```

**Build Configuration (binding.gyp):**
```json
{
  "targets": [
    {
      "target_name": "physics",
      "sources": [
        "native/src/physics-world.cpp"
      ],
      "include_dirs": [
        "<!@(node -p \"require('node-addon-api').include\")",
        "native/vendor/box2d/include"
      ],
      "dependencies": [
        "<!(node -p \"require('node-addon-api').gyp\")"
      ],
      "libraries": [
        "../native/vendor/box2d/lib/libbox2d.a"
      ],
      "cflags!": ["-fno-exceptions"],
      "cflags_cc!": ["-fno-exceptions"],
      "defines": ["NAPI_DISABLE_CPP_EXCEPTIONS"]
    }
  ]
}
```

**Pros:**
- ✅ **Maximum performance** (6-10x faster)
- ✅ Truly synchronous (blocking) calls
- ✅ Near-zero overhead (~0.01ms per call)
- ✅ Can use C++ optimizations
- ✅ Direct memory access

**Cons:**
- ❌ Complex build setup
- ❌ Platform-specific compilation (Linux/Windows/Mac)
- ❌ Harder to debug
- ❌ Deployment complexity

**When to Use:**
- Current player count: 200-500
- Need maximum performance
- Have C++ experience on team
- Target multiple platforms

**Migration Effort:** **High** (1-2 weeks)

---

### Option 3: Shared Memory with Atomics (Zero-Copy)

**Description:** Use SharedArrayBuffer for zero-copy data exchange between JS and C++.

**Implementation:**
```typescript
// Allocate shared memory
const ENTITY_SIZE = 8; // 2 floats (x, y) * 4 bytes
const MAX_ENTITIES = 1000;
const sharedBuffer = new SharedArrayBuffer(ENTITY_SIZE * MAX_ENTITIES);
const sharedData = new Float32Array(sharedBuffer);

// Initialize C++ with shared buffer
PhysicsBridge.initializeShared(sharedBuffer);

protected update(): void {
  const entities = Q_DynamicEntities(this);

  // Write directly to shared memory (zero-copy)
  for (let i = 0; i < entities.length; i++) {
    const offset = i * 2;
    sharedData[offset] = directionX;
    sharedData[offset + 1] = directionY;
  }

  // C++ processes data in-place (synchronous)
  PhysicsBridge.stepShared(entities.length);

  // Read results from shared memory (zero-copy)
  for (let i = 0; i < entities.length; i++) {
    const offset = i * 2;
    const newX = sharedData[offset];
    const newY = sharedData[offset + 1];
    // Update positions...
  }
}
```

**Pros:**
- ✅ **Zero-copy** data transfer
- ✅ Fastest for large datasets (500+ entities)
- ✅ Minimal overhead

**Cons:**
- ⚠️ More complex memory management
- ⚠️ Need careful synchronization

**When to Use:**
- 500+ concurrent entities
- Need absolute maximum throughput

**Migration Effort:** **High** (2-3 weeks)

---

### Option 4: Separate C++ Physics Process (Horizontal Scaling)

**Description:** Run Box2D in standalone C++ process, communicate via IPC or Unix sockets.

**Architecture:**
```
┌─────────────────────┐      IPC/Socket      ┌──────────────────┐
│  Node.js Server     │◄────────────────────►│  C++ Physics     │
│  - Game Logic       │   (binary protocol)  │  - Box2D World   │
│  - ECS Systems      │                      │  - Queries       │
│  - Networking       │                      │  - Step          │
└─────────────────────┘                      └──────────────────┘
```

**C++ Physics Server (physics-server.cpp):**
```cpp
#include <box2d/box2d.h>
#include <iostream>
#include <vector>

// Simple binary protocol
struct PhysicsInput {
  uint32_t entityCount;
  struct {
    float dirX, dirY, speed;
  } entities[1000];
};

struct PhysicsOutput {
  uint32_t entityCount;
  struct {
    float x, y;
  } positions[1000];
};

int main() {
  b2World world(b2Vec2(0.0f, 0.0f));
  std::vector<b2Body*> bodies;

  while (true) {
    PhysicsInput input;
    std::cin.read((char*)&input, sizeof(PhysicsInput));

    // Apply forces and step
    for (uint32_t i = 0; i < input.entityCount; i++) {
      // ... apply forces
    }
    world.Step(1.0f / 20.0f, 8, 3);

    // Send results
    PhysicsOutput output;
    output.entityCount = bodies.size();
    for (size_t i = 0; i < bodies.size(); i++) {
      output.positions[i].x = bodies[i]->GetPosition().x;
      output.positions[i].y = bodies[i]->GetPosition().y;
    }

    std::cout.write((char*)&output, sizeof(PhysicsOutput));
    std::cout.flush();
  }
}
```

**Node.js Bridge:**
```typescript
import { spawn } from 'child_process';

export class PhysicsProcessBridge {
  private static process: ChildProcess;

  static initialize(): void {
    this.process = spawn('./physics-server');
  }

  static step(entities: any[]): Promise<any[]> {
    return new Promise((resolve) => {
      // Write input
      const buffer = this.serializeInput(entities);
      this.process.stdin.write(buffer);

      // Read output
      this.process.stdout.once('data', (data) => {
        const results = this.deserializeOutput(data);
        resolve(results);
      });
    });
  }
}
```

**Pros:**
- ✅ Maximum performance
- ✅ Can run on separate CPU cores
- ✅ Horizontal scaling (multiple physics servers)
- ✅ Isolated crashes (physics crash won't kill game server)

**Cons:**
- ❌ Most complex architecture
- ❌ IPC overhead (~0.1-1ms)
- ❌ Harder to debug
- ❌ Need process management

**When to Use:**
- 500+ concurrent players
- Multiple game worlds
- Need horizontal scaling
- Have DevOps infrastructure

**Migration Effort:** **Very High** (3-4 weeks)

---

## Migration Decision Tree

```
Start
  │
  ├─ Need quick win? (< 2 days)
  │   └─► box2d-wasm (Option 1)
  │
  ├─ Need max performance? (200-500 players)
  │   └─► N-API Native (Option 2)
  │
  ├─ Need zero-copy? (500+ entities)
  │   └─► Shared Memory (Option 3)
  │
  └─ Need horizontal scaling? (1000+ players)
      └─► C++ Process (Option 4)
```

---

## Current API Surface (Easy to Migrate)

Our minimal Box2D usage makes migration straightforward:

```typescript
// 1. Body Management
world.b2World.CreateBody(bodyDef)
world.b2World.DestroyBody(body)

// 2. Physics Step
world.b2World.Step(TICK_DELTA, PHYSICS_ITERATIONS)

// 3. Queries
world.b2World.QueryAABB(aabb, callback)
world.b2World.RayCast(point1, point2, callback)

// 4. Position Access
body.GetPosition()
body.SetPosition(x, y)

// 5. Forces
body.ApplyLinearImpulseToCenter(impulse)
```

**That's it!** Only 5 operation types to migrate.

---

## Recommended Migration Path

### Phase 1: Baseline (Current)
- Use `@box2d/core`
- Optimize game logic first
- Profile to identify bottlenecks
- **When:** Initial development

### Phase 2: Easy Win
- Switch to `box2d-wasm`
- Minimal code changes
- 3-5x performance improvement
- **When:** Ticks consistently >16ms, 100-200 players

### Phase 3: Maximum Performance
- Implement N-API native addon
- Full C++ Box2D
- 6-10x performance improvement
- **When:** Need 200-500 players, have C++ expertise

### Phase 4: Horizontal Scaling (Future)
- Separate C++ physics process
- Multiple worlds, load balancing
- **When:** 500+ concurrent players

---

## Performance Targets

| Player Count | Target TPS | Recommended Solution | Expected Tick Time |
|--------------|-----------|---------------------|-------------------|
| 0-100 | 20 TPS | @box2d/core | 30-40ms |
| 100-200 | 20-30 TPS | box2d-wasm | 15-25ms |
| 200-500 | 30-60 TPS | N-API Native | 8-16ms |
| 500+ | 30-60 TPS | C++ Process | 5-12ms |

---

## Build and Deployment

### box2d-wasm
```bash
# Install
npm install box2d-wasm

# Deploy
# No special requirements - works on all platforms
```

### N-API Native
```bash
# Development
npm install node-addon-api
npm install --save-dev cmake-js
npm run build:native

# Production (compile for each platform)
npm run build:native:linux
npm run build:native:windows
npm run build:native:mac

# Deploy
# Include compiled .node files for each platform
```

### C++ Process
```bash
# Build
cd native/physics-server
cmake .
make

# Deploy
# Copy physics-server binary
# Ensure binary is executable
chmod +x physics-server
```

---

## See Also

- [Movement Restrictions](./../systems/MOVEMENT_RESTRICTIONS.md)
- [Entity Action & Status System](./ENTITY_ACTION_STATUS_SYSTEM.md)
- [Attack System](./../systems/ATTACK_SYSTEM.md)
