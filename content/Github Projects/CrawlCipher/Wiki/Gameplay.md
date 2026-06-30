# Gameplay Overview

CrawlCipher is a tactical, grid-based, turn-calculated action game described as a mix of **Chess** and **Snake**. It values spatial planning, resource management, and geometric positioning over raw reaction speeds.

---

## The Core Loop

1. **Lock & Load:** Equip your items and initiate the blockchain session lock.
2. **Deterministic Spawn:** The game board (grid width, grid height, walls, food, enemies) is initialized deterministically based on the transaction seed.
3. **Navigate & Surive:** Slither through the grid, avoiding walls, enemy bullets, and body self-collisions.
4. **Acquire Energy & Score:** Eat food and eliminate enemy snails to maintain your energy level and increase your score multiplier.
5. **Circuit Activation:** Bend and loop your body segments to activate overlapping weapon hitboxes (the **Moving Circuit** mechanic) to destroy obstacles and enemies.
6. **Extract:** When the wave count or time objective is complete, the Exit Portal activates. Navigate the snake's head to the portal and survive the countdown to extract.
7. **Unlock & Record:** Submit the final session hash to release your locked blockchain assets and record your new scores.

---

## UI Layout (Terminal Panel)

The Terminal interface displays several distinct panels:

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

### Controls
* **Arrow Keys / WASD:** Change the snake's heading direction. In *Manual* mode, each keypress advances the game by one tick (step).
* **Enter / Space:** Dash or fire active weapons.
* **I / B:** Toggle Backpack & Inventory UI overlay to equip/unequip modules.
* **Esc:** Pause game / return to menu.
