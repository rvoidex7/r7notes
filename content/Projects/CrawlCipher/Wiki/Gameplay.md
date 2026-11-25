# Gameplay Overview

> [!warning] Prototype / Sandbox Build
> The *final* CrawlCipher game — the mode with a designed goal, progression, and polished fun loop — is still being designed. What this Wiki documents is the current **experimental sandbox build**: a testbed exercising the engine's movement, energy, combat, and extraction systems. Everything below reflects implemented behavior, but the game built on top of these mechanics may change significantly.

CrawlCipher's current build is a tactical, grid-based, turn-calculated snake simulation. It rewards spatial planning, resource management, and geometric positioning over raw reaction speed.

---

## The Core Loop (current build)

1. **Lock & Load:** Equip your items and initiate the blockchain session lock.
2. **Deterministic Spawn:** The game board (grid width, grid height, walls, food, enemies) is initialized deterministically based on the transaction seed.
3. **Navigate & Survive:** Slither through the grid, avoiding walls, enemy bullets, and body self-collisions.
4. **Acquire Energy & Score:** Eat food and eliminate enemy snails to maintain your energy level and increase your score multiplier.
5. **Circuit Activation:** Bend and loop your body segments to activate overlapping weapon hitboxes (the **Moving Circuit** mechanic) to destroy obstacles and enemies.
6. **Extract:** When the wave count or time objective is complete, the Exit Portal activates. Navigate the snake's head to the portal and survive the countdown to extract.
7. **Unlock & Record:** Submit the final session hash to release your locked blockchain assets and record your new scores.

Curious how steps 2 and 7 make cheating cryptographically detectable? See [[Anti-Cheat-Verification|Anti-Cheat & Verification]] and [[Deterministic-Physics|Deterministic Physics]].

---

## UI Layout (Terminal Panel)

The Terminal interface displays several distinct panels (illustrative layout):

```
+-------------------------------------------------------------+
| V0.2.0 | PILOT: RVOIDEX7                    [ARROWS] MOVE   |
+------------------------------------+------------------------+
|                                    |                        |
|                                    |   MISSION STATUS       |
|                                    |   Wave: 3/5            |
|             GAME GRID              |   Multiplier: x1.45    |
|                                    |   Score: 2450          |
|         (Snake Movement)           |                        |
|                                    |------------------------|
|                                    |   PILOT STATUS         |
|                                    |   Energy: [|||||..] 5/7|
|                                    |   Bonus:  [||...]   2  |
|                                    |                        |
+------------------------------------+------------------------+
| BACKPACK: [Pistol (10)] [Laser (2)] [Amplifier]             |
+-------------------------------------------------------------+
```

## Controls

| Key | Action |
|---|---|
| **Arrow Keys / `W` `S` `D`** | Change the snake's heading. Press two directions in the same tick for diagonals (e.g. Up+Right = NorthEast). `A` is reserved for focus control, not movement. In *Manual* mode, each step only advances when the engine ticks with your queued input. |
| **`A` / `Z`** | Move the segment focus cursor toward the head / toward the tail. |
| **`Space`** | Fire the weapon mounted on the focused segment. |
| **`F`** | Execute a Strike (A* dash). |
| **`X` / `C`** | Attach the selected backpack item to the focused segment — left / right side. |
| **`I`** | Toggle the Backpack & Inventory overlay (`E`/`Enter` equips, `U` unequips, `Esc`/`I` closes). |
| **`M`** | Toggle Manual / Autopilot movement mode. |
| **`P` / `Esc`** | Pause / return toward menu. |
| **`R`** | Restart the session. |
| **`Ctrl+Q`** | Quit. |
