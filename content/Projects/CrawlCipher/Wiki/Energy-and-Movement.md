# Energy & Movement Physics

Movement in CrawlCipher is governed by a strict energy economy. Every turn consumes a portion of your energy pool, making navigation a tactical puzzle.

---

## 1. The Energy Pool

Your energy status is split into two pools:
- **Base Energy (`Energy`):** Your active energy capacity. Max capacity defaults to **7** (configurable in sandbox).
- **Overflow Energy (`BonusEnergy`):** Energy accumulated beyond your maximum capacity (e.g., by consuming snails or foods while at max base energy). Bonus energy is spent first when performing high-cost actions.

---

## 2. Energy Costs of Movement

The energy cost for movement depends entirely on the angle of your turn relative to your current direction:

| Heading Change | Degree | Energy Cost | Practical Implication |
|----------------|--------|-------------|-----------------------|
| **Straight** | 0° | **0** | Free movement. |
| **Slight Turn** | 45° | **2** | Moderately cheap maneuver. |
| **Right Angle** | 90° | **5** | Expensive; consumes most of base energy. |
| **Sharp Turn** | 135° / 180° | **12** | **Impossible** with default max energy (7) unless you have accumulated **Bonus Energy** overflow. |

```
        North (0)
     7      ^      1
      \     |     /
   6 <------+------> 2 East
      /     |     \
     5      v      3
        South (4)
```

Because a 180-degree turn (backtracking) or a sharp 135-degree diagonal turn costs **12 energy**, you cannot execute these maneuvers on a whim. You must plan your trajectory ahead of time, collect food to bank bonus energy, or loop around in a wider radius.

---

## 3. Energy Recovery

Energy can be restored through several actions:
- **Step Recovery:** Moving straight generates a baseline amount of energy (default: `+1` per step).
- **Idle Recovery (Manual Mode Only):** If you remain stationary, you gain energy at the idle rate (default: `+2` per tick) up to your max energy capacity.
- **Consumption:**
  - **Food:** Eating a food pellet restores `+2` base energy and adds score.
  - **Snails:** Eliminating and consuming a Snail segment yields a massive reward (default: `+5` energy). If your base energy is full, this overflows into `BonusEnergy`.

---

## 4. Control Modes

### A. Manual Mode (Tactical)
In Manual mode, the game does not advance automatically. Ticks only pass when you press a key.
- **Behavior:** The game plays like a turn-based board game.
- **Advantage:** Gives you infinite time to calculate your path, count cell distances, and inspect hitboxes.
- **Idle Gain:** Remaining idle in manual mode dynamically charges your energy.

### B. Autopilot Mode (Traditional)
In Autopilot mode, the game advances automatically at a fixed tick rate (e.g. 150ms per step).
- **Behavior:** The snake slithers continuously, matching traditional snake gameplay.
- **Advantage:** Faster gameplay, higher score multipliers, but leaves less time to plan complex circuit alignments.
