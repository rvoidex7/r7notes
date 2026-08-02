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
To prevent players from pre-calculating optimal paths using a known offline RNG seed, **online sessions** bind the seed to a ledger that only exists *after* the player has already committed to playing — the seed cannot be known, let alone re-rolled, before the on-chain lock lands:

```
[TUI]                  ---> lock_session(player, assets) ---> [Soroban Contract]
                                                                      |
                                                                      v
                                                    records lock_seq = env.ledger().sequence()
                                                                      |
[TUI]  <----------------------- get_lock_seq(player) ----------------┘
  |
  v
Poll briefly until ledger (lock_seq + 1) has closed
  |
  v
[Stellar Horizon API] ---> Fetch ledger hash at sequence = lock_seq + 1
                                 |
                                 v
[SHA-256 Hashing]     ---> Convert hash string to 64-bit seed integer (i64)
                                 |
                                 v
[C# Core Engine]       ---> Seed DeterministicRng(seed) (xoshiro256** + splitmix64)
```

1. **Lock First:** The Rust frontend submits `lock_session` to the Soroban contract *before* any seed is fetched. The contract records `lock_seq`, the ledger sequence at lock time, alongside the locked assets.
2. **Read the Lock Sequence:** The TUI calls the `get_lock_seq` getter to read that sequence back.
3. **Fetch the Next Ledger:** The TUI fetches the ledger at `sequence = lock_seq + 1` — the first ledger closed *after* the lock — from the [Horizon single-ledger endpoint](https://developers.stellar.org/docs/data/apis/horizon/api-reference/resources/ledgers), polling briefly if it hasn't closed yet. Because this ledger postdates the lock transaction, no one — including the developers — can know its hash before the player commits.
4. **Hashing:** The hash string is processed using SHA-256 in Rust:
   ```rust
   let mut hasher = Sha256::new();
   hasher.update(hash.as_bytes());
   let result = hasher.finalize();
   let bytes: [u8; 8] = result[0..8].try_into().unwrap_or([0; 8]);
   let seed = i64::from_le_bytes(bytes);
   ```
5. **Core Seeding:** The generated seed is passed to `CreateGame(seed)` and consumed in full — all 64 bits, no truncation — by the engine's own `DeterministicRng` (xoshiro256** seeded via splitmix64; see `Rng/DeterministicRng.cs`), replacing `System.Random`. Because the seed is unknowable until after the lock transaction lands, the food spawn coordinates, enemy snail directions, and bot paths are impossible to know before the game session initializes.

**Offline mode is unchanged:** without a wallet key, the seed comes from the system clock instead of a ledger hash, no lock is submitted, and the run produces no verifiable proof.

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

At the end of a session, the engine generates a cryptographic summary of the gameplay: **proof format v1**. It serializes the following fields, in order, and hashes them using SHA-256:

```
SHA-256 ( Seed | EngineVersion | ConfigHash | Complete Input Log | "CRAWLCIPHER_PROOF_V1" )
```

- `EngineVersion` is the engine build tag (`GameEngine.EngineVersion`, e.g. `core-0.3.0`), committed so a proof is unambiguously tied to the engine that produced it.
- `ConfigHash` is a SHA-256 over **every** field of the effective `GameConfig` — including the nested `ExpeditionConfig`, `BossWaveConfig`, `ScoringConfig`, `DeathPenaltyConfig` and the `WeaponStats` table — in their declared order, not just grid dimensions. Editing any rule (weapon stats, wave timing, scoring, death penalties, ...) changes this hash.
- There is no secret salt. `"CRAWLCIPHER_PROOF_V1"` is a **public** domain-separation/version tag: under replay verification a secret adds nothing (the engine is open), and a public tag doubles as the proof-format version field so future format changes can't be confused with this one.

This resulting 64-character hex string is the **Session Verification Hash**.

### The Verification Workflow (Optimistic Verification)
This mirrors the fraud-proof philosophy of [optimistic rollups](https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/) — a chain-agnostic *pattern* popularized in the Ethereum ecosystem (accept results by default, punish provable lies). CrawlCipher runs on Stellar; only the idea is borrowed, not the platform.
1. **Submit Proof:** The TUI client submits this hash to the Soroban smart contract via `unlock_session`. The contract persists `{ game_hash, lock_seq, unlock_seq }` for the player in persistent storage before releasing the lock, and exposes it via the `get_proof` getter (one record per player — their last session).
2. **Challenge / Audit:** A validator node or backend oracle can audit the session by pulling the player's logged inputs (which are saved in the session data) and the persisted proof record.
3. **Replay Execution:** The validator runs the C# engine locally using the same seed and applies the inputs tick-by-tick.
4. **Compare Hash:** If the validator's calculated final state hash matches the hash recorded on the blockchain, the game is marked as valid. If they mismatch, a fraud proof is triggered, indicating the player modified memory or forged inputs.
