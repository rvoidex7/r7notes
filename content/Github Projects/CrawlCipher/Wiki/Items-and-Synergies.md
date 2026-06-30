# Weapons, Modules & Moving Circuit Synergies

Instead of raw stats, CrawlCipher's inventory system focuses on geometric rules and spatial interactions. You can attach weapons and modules to specific body segments of your snake and trigger combos by wrapping your body around itself.

---

## 1. Item & Weapon Registry

Items are classified into three types: **Weapons**, **Modules**, and **Consumables**.

### Weapons
Weapons fire projectiles along the line of sight of the segment they are attached to:

| Weapon Type | Base Ammo | Range (Cells) | Description |
|-------------|-----------|---------------|-------------|
| **Pistol** | 12 | 10 | Standard projectile weapon. Reliable, medium range. |
| **Rifle** | 30 | 20 | Rapid-fire projectile weapon. Long range. |
| **Laser** | 5 | 30 | Instant hitscan beam. Pierces through multiple entities. |

### Modules
Modules do not fire on their own but modify the behavior of neighboring cells and weapons:

| Module Type | Active Range | Description |
|-------------|--------------|-------------|
| **Amplifier** | 3x3 Grid | Multiplies the damage of any projectile passing through its area. |
| **Prism** | 3x3 Grid | Bends the trajectory of passing bullets or lasers by 45 degrees (1 step in the 8-directional layout). |
| **Collector**| 5x5 Grid | Automatically absorbs food and energy pellets that enter its etki alanı (area of influence, radius 2) without the snake's head needing to touch them. |

---

## 2. Hitbox Geometry

- Standard module and weapon etki alanları (areas of effect) are represented as **3x3 cell boxes** (radius 1) centered on the equipped segment, with the exception of the **Collector** module which spans a **5x5 cell box** (radius 2).
- An interaction triggers only when another entity or weapon line-of-fire physically enters or intersects this grid.

```
   3x3 Hitbox Grid:
   +---+---+---+
   | . | . | . |
   +---+---+---+
   | . | S | . |  <-- S = Equipped Snake Segment
   +---+---+---+
   | . | . | . |
   +---+---+---+
```

---

## 3. The Moving Circuit Synergy

The signature mechanic of CrawlCipher is the **Moving Circuit**. 

Because your snake is moving and bending, the relative positions of your body segments are constantly changing. When you curve the snake so that equipped segments run parallel or adjacent to each other, their 3x3 hitboxes overlap. This completes a "circuit," transforming your weapons:

```
        [Segment 1: Laser]  ---> Shoots straight forward
               |
               v (Intersects 3x3 area)
        [Segment 8: Prism]  ---> Bends Laser beam 45 degrees
               |
               v (Intersects 3x3 area)
        [Segment 7: Amp]    ---> Amplifies beam power
```

### Example Synergies
- **Laser-Prism Cornering:** By curving the tail so a Prism segment sits 1 block ahead and 1 block to the side of a Laser segment, you can fire a laser beam that bends 45 degrees to hit diagonal targets.
- **Super Box Combo:** If you loop the snake into a tight 2x2 grid spiral, 4 body segments compress together. If these segments hold a **Laser**, **Amplifier**, and **Prism**, their overlapping fields create a "Super Box" that fires high-damage, splitting laser beams in multiple directions simultaneously.
- **Geomertic Limitation:** Because the snake must keep moving to survive, these perfect circuit alignments are temporary. You must timing-align your curls, fire your combo, and then unwrap to avoid self-collision.
