# Cryptographic Anti-Cheat & Verification

CrawlCipher uses a **Proof of Execution (PoE)** model to prevent cheating in a completely local terminal simulation without relying on continuous server authority. This document explains the mathematical and logical structures that make this possible.

---

## 1. The Threat Model
In local-first Web3 games, the client runs on the user's local hardware. If the game merely sends the final score or won items to the blockchain, a user can easily intercept the network request or modify game memory (e.g. Cheat Engine) to forge wins.
CrawlCipher addresses this through two primary mechanisms:
1. **Preventing Simulation Pre-Calculation:** Ensuring the player cannot pre-compute optimal moves before starting.
2. **Deterministic Session Auditing:** Forcing the player to submit a full log of their moves, which is then replayed to verify the legitimacy of the outcome.

---

## 2. Dynamic Entropy via Stellar Ledger
To prevent players from pre-calculating optimal paths using a known offline RNG seed, the initial game state is tied to active network activity on the Stellar blockchain:

```
[Stellar Horizon API] ---> Fetch Latest Ledger Hash (e.g. 0x3d7a...f4a)
                                 |
                                 v
[SHA-256 Hashing]     ---> Convert Hash string to 64-bit seed integer (i64)
                                 |
                                 v
[C# Core Engine]       ---> Seed Engine's Random(seed) instance
```

1. **Horizon Request:** The Rust frontend fetches the latest ledger data from:
   `https://horizon-testnet.stellar.org/ledgers?order=desc&limit=1`
2. **Hashing:** The hash string is processed using SHA-256 in Rust:
   ```rust
   let mut hasher = Sha256::new();
   hasher.update(hash.as_bytes());
   let result = hasher.finalize();
   let bytes: [u8; 8] = result[0..8].try_into().unwrap_or([0; 8]);
   let seed = i64::from_le_bytes(bytes);
   ```
3. **Core Seeding:** The generated seed is passed to `CreateGame(seed)`. Because `System.Random` in C# is seeded with this unpredictable value, the food spawn coordinates, enemy snail directions, and bot paths are impossible to know before the game session initializes.

---

## 3. Deterministic Input Logging

During the session, the engine keeps a strict record of every single input frame. An input frame (`InputFrame`) records exactly what key/action was triggered and at what simulation tick:

```csharp
public struct InputFrame
{
    public long Tick;      // The current frame index
    public int InputType;  // KeyCode or action type
    public int Param1;     // Coordinates or details
    public int Param2;     // Extra parameters (e.g. Weapon slots)
}
```

Because the simulation state updates at a fixed tick rate (e.g. 10 ticks per second) and all state transitions are pure, deterministic functions of the previous state and current inputs, **replaying the input log against the initial seed will recreate the exact final state of the game, every single time.**

---

## 4. Replay Verification & State Hashing

At the end of a session, the engine generates a cryptographic summary of the gameplay. It serializes the following inputs and hashes them using SHA-256:

```
SHA-256 ( Seed + GameConfig + Complete Input Log + Private Verification Salt )
```

This resulting 64-character hex string is the **Session Verification Hash**.

### The Verification Workflow (Optimistic Verification)
1. **Submit Proof:** The TUI client submits this hash to the Soroban smart contract via `unlock_session`.
2. **Challenge / Audit:** A validator node or backend oracle can audit the session by pulling the player's logged inputs (which are saved in the session data).
3. **Replay Execution:** The validator runs the C# engine locally using the same seed and applies the inputs tick-by-tick.
4. **Compare Hash:** If the validator's calculated final state hash matches the hash recorded on the blockchain, the game is marked as valid. If they mismatch, a fraud proof is triggered, indicating the player modified memory or forged inputs.
