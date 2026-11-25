# C# Core Engine Code Walkthrough

This document provides a detailed walkthrough of the unmanaged C# simulation engine (`CrawlCipher.Core`), detailing the state flow, physics tick, and FFI boundaries.

---

## 1. Project Structure

The engine lives under `CrawlCipher.Core/`, organized by subsystem (folderized 2026-07-06). The whole simulation is still **one class** — `GameEngine` — split across files with C# `partial class` (a compile-time construct: the compiler merges the parts, so the layout has zero runtime cost):

1.  **`Models/`** — data structures: `Player.cs` (Player, SnakeSegment, WeaponData), `Bullet.cs`, `Snail.cs`, and `Types.cs` (enums, FFI structs, configs, `InputFrame`).
2.  **`GameCore.cs`** — the `GameEngine` root: lifecycle, configuration, input processing, inventory, replay hashing.
3.  **Subsystem partials** — `Physics/GameEngine.Movement.cs`, `Combat/GameEngine.Combat.cs`, `AI/GameEngine.Steering.cs`, `Pathfinding/GameEngine.Pathfinder.cs`.
4.  **`FFI/FFIExports.cs`** — exposes C-compatible pointers and structures to the dynamic linker for Rust interop.

---

## 2. Structural Memory Models (`Models/`)

To pass data across the Rust border without marshalling overhead, entities use two representations: **Managed Classes** (for internal C# simulation) and **Unmanaged Structs** (sequential layouts passed to Rust).

### A. The Player Model
Within the engine, a player is defined as a class:
```csharp
public class Player
{
    public int Id;
    public List<SegmentData> Body;         // List of X, Y positions for segments
    public int Energy;
    public int Score;
    public int Kills;
    public bool IsIdle;                    // True if manual move has not been input
    public bool IsAutopilot;               // True if continuous movement is toggled
    public WeaponType[] BodyWeapons;       // Parallel array mapping equipment to segments
}
```

### B. Segment Data Layout
```csharp
public class SegmentData
{
    public int X;
    public int Y;
    public Direction Dir;
    public string EquippedItemId;          // UUID reference to Backpack item
}
```

---

## 3. Simulation Loop & Physics (`GameCore.cs` + subsystem partials)

The `GameEngine` class orchestrates the gameplay ticks. It manages a state machine defined by `SimulationStateType`:

```
 [Menu] --(StartGame)--> [Running] --(Pause)--> [Paused]
                            |
                     (Head Collision)
                            v
                       [GameOver]
```

### The Update Cycle (`Tick()` Method)
When the Rust TUI calls `Update(gamePtr)`, the engine executes the following sequentially:

1.  **Input Resolution:** Applies pending move commands to shift player directions.
2.  **Snail Simulation:** Bounces snail obstacles off walls and shifts their coordinates based on speed dividers.
3.  **Bullet Propagation:** Advances all bullet projectiles forward by `BulletSpeedMultiplier` cells.
4.  **Collision Pass:**
    - Checks if bullets intersect snake segments or snails (applies damage/severing).
    - Checks if snake heads hit food (triggers growth queue and restores energy).
    - Checks if snake heads hit walls or other snake segments (triggers death or wrap-around).
5.  **Energy Regeneration:** Restores energy based on state (manual idle vs autopilot moving).

---

## 4. Unmanaged Exports & Pinning (`FFI/FFIExports.cs`)

The C# engine is compiled to native code using NativeAOT. It exposes entry points annotated with `[UnmanagedCallersOnly]`.

### FFI Entry Points Registry

The unmanaged functions exposed to the dynamic linker (`libloading`) are mapped as follows:

| Function Symbol | Return Type | Arguments | Description |
|-----------------|-------------|-----------|-------------|
| `CreateGame` | `IntPtr` | Grid dimensions, configuration options | Allocates engine state on heap, returns GCHandle pointer. |
| `DestroyGame` | `void` | `IntPtr gamePtr` | Releases the GCHandle pinning, allowing GC collection. |
| `Update` | `void` | `IntPtr gamePtr` | Advances game state by one simulation tick. |
| `ProcessInput` | `void` | `IntPtr gamePtr, int type, int p1, int p2` | Queues or executes user command. |
| `GetGameState` | `GameState` | `IntPtr gamePtr` | Retrieves global stats (ticks, state type). |
| `GetPlayerState`| `PlayerState`| `IntPtr gamePtr, int playerId` | Retrieves specific player metrics. |
| `GetGridCells` | `int` | `IntPtr gamePtr, IntPtr outCells, int max` | Fills array with active viewport cell data. |

### Heap Pinning Workflow
Because C# is a garbage-collected language, the engine instance must be pinned in memory to prevent the GC from reclaiming or relocating it:

```csharp
[UnmanagedCallersOnly(EntryPoint = "CreateGame")]
public static unsafe IntPtr CreateGame(int width, int height)
{
    // 1. Allocate the C# managed engine instance
    var engine = new GameEngine(width, height);
    
    // 2. Allocate a GCHandle to root the instance
    GCHandle handle = GCHandle.Alloc(engine, GCHandleType.Normal);
    
    // 3. Return the unmanaged raw pointer to Rust
    return GCHandle.ToIntPtr(handle);
}
```

Every subsequent FFI call from Rust passes this `IntPtr` back, allowing C# to resolve the handle target:
```csharp
[UnmanagedCallersOnly(EntryPoint = "Update")]
public static unsafe void Update(IntPtr gamePtr)
{
    GCHandle handle = GCHandle.FromIntPtr(gamePtr);
    var engine = (GameEngine)handle.Target!;
    engine.Tick();
}
```
