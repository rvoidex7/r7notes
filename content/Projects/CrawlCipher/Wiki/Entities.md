# Entities & Hazards

The grid is populated by various active entities that can help or hinder your survival.

---

## 1. Snails (`@`)

Snails are slow-moving obstacles that crawl across the grid.
- **Visual Representation:** Rendered in the terminal as `@-`, `@/`, `@|`, or `@\` indicating their movement direction.
- **Speed:** Snails move slower than snakes, determined by the `SnailSpeedDivisor` configuration (e.g., if set to 2, they move once every 2 ticks).
- **Interactions:**
  - **Collision:** Crashing your snake's head into a Snail causes a collision, dealing damage/stun.
  - **Elimination:** You can destroy Snails using weapons (bullets or lasers).
  - **Reward:** Destroying a snail drops energy-rich residue. Consuming it yields a high score bonus and `+5` energy (which can overflow into `BonusEnergy`).

---

## 2. Projectiles (Bullets)

Bullets are spawned by fired weapons.
- **Speed:** Bullets travel much faster than normal movement, governed by the `BulletSpeedMultiplier` (e.g., moving 3 cells per tick).
- **Lifetime:** Bullets live for a limited number of ticks (`MaxTicks` depending on the weapon range) before disappearing.
- **Collision:** Bullets deal damage to snake segments, bots, and snails. A hit to a snake segment can disable equipped weapons or reduce durability.

---

## 3. Bots

Bots are AI-controlled snakes that compete in the same arena.
- **AI Behavior:** Bots navigate the grid searching for food and avoiding walls.
- **Aggression:** If a bot possesses weapons, it will attempt to line up shots and fire at the player.
- **Elimination:** Bots can be destroyed by cutting off their paths (forcing them to crash into walls or your body) or shooting them.
- **Scoring:** Eliminating a bot grants a base score (default 100) plus additional points proportional to the length of the destroyed bot.

---

## 4. Exit Portal

The extraction portal is your gateway to ending a session successfully and recording your blockchain proof:
- **Trigger:** Activates when the target wave or match duration is reached.
- **Activation:** Renders as an active portal at random coordinates on the grid.
- **Extraction Countdown:** Once your snake's head enters the portal, an extraction countdown begins. You must survive within the portal area for the duration of the countdown to complete the mission and trigger the `unlock_session` transaction.
