# Deterministic Bot AI & Steering

Bots in CrawlCipher (cc) act as competitive players navigating the grid, seeking food, avoiding obstacles, and attacking other snakes. To maintain replay determinism, their decision matrix is calculated using static, rule-based heuristics.

---

## 1. Obstacle Avoidance Heuristic

Bots evaluate their immediate neighborhood (in their moving heading direction) to decide when to steer away from collisions.

### Heuristic Steps:
1. **Raycasting:** The bot checks cells along its path at distances of 1, 2, and 3 steps.
2. **Threat Assessment:**
   - If an obstacle (wall, bullet, Snail, or another snake segment) is 1 cell away, a turn is immediately triggered.
   - If an obstacle is detected 2 or 3 cells away, the bot calculates an alternative path.
3. **Determining Turn Direction:**
   - The bot evaluates the 45° left and 45° right directions.
   - It counts the distance to the nearest obstacle in both paths.
   - It chooses the direction that offers the longest collision-free path.
   - If both directions are blocked, it evaluates 90° turns.

---

## 2. Decision Prioritization Matrix

At the start of its tick updates, a bot evaluates its surroundings and prioritizes its goals according to a fixed decision hierarchy:

```
            +---------------------------------------+
            | Is an immediate collision imminent?   | -- Yes -> [ Steer Away / Turn ]
            +---------------------------------------+
                                | No
            +---------------------------------------+
            | Is a player in line of fire & armed?  | -- Yes -> [ Shoot Weapon ]
            +---------------------------------------+
                                | No
            +---------------------------------------+
            | Is food or a Snail nearby?            | -- Yes -> [ Pathfind to Target ]
            +---------------------------------------+
                                | No
            +---------------------------------------+
            |   Keep moving forward (Straight)      |
            +---------------------------------------+
```

---

## 3. Weapon Aiming & Firing Vectors

If a bot acquires a weapon (e.g. Pistol, Rifle, Laser) in its backpack and equips it to a body segment, it will attempt to target the player.

### Perpendicular Target Matching:
Weapons fire perpendicular (90° left or right) to the segment direction. A bot calculates the firing intersection mathematically:
1. The bot loops through its equipped weapon segments.
2. For each weapon segment, it gets the segment's coordinate `(Sx, Sy)` and its direction vector.
3. It projects a firing ray perpendicular to that direction vector.
4. If the player's head or a player segment intersects this ray within the weapon's range:
   - The bot checks if it has at least 3 energy (the cost of firing).
   - If true, it calls `ProcessInput(FireWeapon, segmentIndex)` and fires, dealing damage or stuns to the player.
