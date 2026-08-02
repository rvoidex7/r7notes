# FAQ — How Does This Actually Work?

Questions a technically-minded visitor tends to ask when they first inspect CrawlCipher — and the honest answers, including what is *not* solved. Short answers here; every deep dive links to the page that owns the topic. For a visual overview, see the [CrawlCipher Mindmap Canvas](./CrawlCipher_Mindmap.canvas) or start at [System Architecture](./Development/Architecture.md).

Answers describing **implemented, verifiable code** are written plainly. Anything that is design-decided but not yet in the engine carries this marker:

> [!note] Planned — not yet in the engine
> The design is settled and the engine will be coded as described, but the code does not exist yet.

---

## Trust & Open Source

### Why should I trust results from a game that runs entirely on the player's machine?

You shouldn't — and the design assumes you won't. CrawlCipher uses a **Proof of Execution** model: the engine is deterministic, every input is logged, and the outcome of a session can be re-computed by anyone from the seed and the input log. Trust is replaced by *replayability*. Full mechanism: [Cryptographic Anti-Cheat & Verification](./Development/Anti-Cheat-Verification.md).

### Isn't keeping the engine source closed just security through obscurity?

Yes, it would be — and we agree with the criticism. The engine follows [Kerckhoffs's principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle): the system must stay secure even when everything except players' keys is public. Nothing in the verification design depends on the code being secret (the one legacy exception, the MVP salt, has been removed — see [the salt question](#whats-actually-inside-the-session-hash) below).

> [!note] Planned — not yet in the engine
> The Core engine source will be fully open-sourced once the proof format no longer contains any secrecy-dependent component. This is a scheduled milestone, not a "maybe".

### If all algorithms are open source, can't cheaters just trial-and-error on their own machine forever until they find the perfect moves?

This is *the* central attack, and it is answered structurally, not by hiding code:

1. **The seed doesn't exist yet when you commit to playing.** A session is locked on-chain first — the contract records the lock-time ledger sequence — and only then is the seed derived from the hash of the [ledger closed right after the lock](./Development/Anti-Cheat-Verification.md#2-dynamic-entropy-via-stellar-ledger). There is nothing to pre-compute against.
2. **Every attempt has a cost.** Re-trying means locking a new session on-chain. "Infinite free retries" don't exist; grinding becomes an economic decision, not a free lunch.
3. What remains possible is searching for good inputs *after* the seed is known, during the session window — a tool-assisted-play problem, not a forgery problem.

> [!note] Planned — not yet in the engine
> One piece of this is not yet built: **interleaved ledger beacons** to shrink the remaining search window further — at fixed intervals the simulation would fold in a fresh ledger hash, so a cheater can never pre-compute the whole run, only search inside one short beacon interval, roughly in real time. (The lock-time ledger sequence and on-chain proof-hash storage described above are already implemented in the contract.)

### What cheats does the system deliberately NOT claim to prevent?

The complete honesty map, in one table:

| Cheat | Status |
|---|---|
| Forging a result, editing memory/score, spawning items | **Blocked** — replay verification rejects any outcome the inputs can't produce |
| Playing with modified rules (or, later, a modified script bundle) | **Blocked** — the config hash pins the exact ruleset; verifiers replay with the canonical one |
| Re-rolling for a lucky seed | **Blocked** — the seed is committed on-chain before play; every retry costs a new lock |
| Submitting someone else's winning replay | **Blocked** — proofs are bound on-chain to the player who locked the session |
| Offline input search (tool-assisted play) | **Bounded** — limited by the session window today; interleaved ledger beacons (planned, see above) would shrink it to roughly real time |
| Live bots / automation producing legitimate inputs | **Not blocked** — a proof shows the inputs are consistent, not that a human produced them |
| Human assistance, multi-accounting, collusion | **Not blocked** — no cryptography anywhere solves these |

The "not blocked" rows are not a CrawlCipher gap: kernel-level anti-cheats, poker sites and chess platforms all fail to *prove* humanity — they estimate it statistically and ban on suspicion. And an assisting program doesn't even need to touch the game's internals: anything that reads the screen and produces keystrokes is invisible to any client-side defense, closed source or not. CrawlCipher refuses to pretend otherwise.

### If bots can't be blocked, doesn't automation just win everything?

Only in a game that rewards what automation is good at. That is a *game-design* problem, and it is treated as a binding design constraint for the flagship game (which is still in its discovery phase — these are directions being designed toward, not shipped features):

- **Reward planning, not execution.** Automation dominates reflex-and-precision play; it matters far less where the win condition is deciding *what* to do (loadouts, routes, trades, territory) rather than executing it frame-perfectly.
- **Zero-input progression.** An expedition/idle-style mechanic under consideration — configure your snake's loadout, close the game, and progress unfolds deterministically while you're away — has, by definition, nothing to automate: there are no inputs during the progression at all.
- **Asynchronous, non-zero-sum multiplayer.** Where one player's perfect run doesn't subtract from another's game, a perfect player is an economy-tuning concern, not a fairness breach.
- **Statistical humanity analysis** remains available as a later soft layer: because replays are deterministic and input logs are tick-exact, the moment information became visible on screen and the moment the player reacted can both be computed *exactly* — precise, reproducible reaction-time profiling that screen-capture-based systems can only approximate. It can flag superhuman play for review; it cannot cryptographically prove anything, and is honestly labeled as such.

---

## Proof of Execution

### What stops classic memory editing (Cheat Engine style)?

Editing memory changes the outcome without changing the input log — so the replayed session no longer matches the submitted result, and verification fails. See [Replay Verification](./Development/Anti-Cheat-Verification.md#4-replay-verification--state-hashing).

### Can someone forge a winning hash without playing at all?

No. The session hash commits to the seed and the *complete* input log. To claim an outcome you must present inputs that actually produce that outcome when replayed by the canonical engine — at which point you have, by definition, "played" the game (possibly via search; see the trial-and-error question above).

### Can a player re-roll until they get a lucky seed?

Not for free. The contract records the lock-time ledger sequence, and the seed is derived from the hash of the ledger closed right after the lock — enforced on-chain via `lock_session`/`get_lock_seq`, not just a client convention — so a re-roll requires a new on-chain lock per attempt.

### Can someone steal and resubmit another player's winning replay?

Sessions are locked per player in the Soroban contract; a proof is only accepted against the session (and player) it was locked for. Replays are not transferable trophies. The contract persists `{ game_hash, lock_seq, unlock_seq }` per player on `unlock_session`, so proof-to-session binding is enforced on-chain, not just assumed.

### What's actually inside the session hash?

**Proof format v1** (current): `SHA-256(seed | engine version | config hash | complete input log | "CRAWLCIPHER_PROOF_V1")`, computed by the engine at session end — details in [Anti-Cheat & Verification](./Development/Anti-Cheat-Verification.md#4-replay-verification--state-hashing). The config hash commits to the *full* effective `GameConfig` (every field, including nested weapon/wave/scoring/death-penalty settings) — not just grid dimensions. There is no private salt anymore: `CRAWLCIPHER_PROOF_V1` is a **public version tag** — under replay verification a secret salt adds nothing, and a public tag doubles as the format version field.

> [!note] Planned — not yet in the engine
> Once modding lands, the config hash will also cover the hash of the loaded Lua script bundle, so every modded ruleset gets its own verifiable proof domain.

### Who verifies proofs, and what happens when one is fraudulent?

The model mirrors the fraud-proof philosophy of [optimistic rollups](https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/): results are accepted by default, and any auditor can replay a session and trigger a fraud proof on mismatch — see [the verification workflow](./Development/Anti-Cheat-Verification.md#4-replay-verification--state-hashing).

> [!note] Planned — not yet in the engine
> The long-term design is a **decentralized, player-operated verification network** with bounties for catching fraud (specified in the internal design docs). That network is *why* the engine must be open source: verifiers re-run the same code everyone can inspect.

### Why optimistic replay instead of zero-knowledge proofs?

Because re-running this simulation is nearly free, and [zk proofs](https://en.wikipedia.org/wiki/Zero-knowledge_proof) are the opposite trade: verification becomes instant and trustless, but *producing* the proof means compiling the entire game logic into an arithmetic circuit, paying minutes of proving time per session, and redoing the circuit on every engine change. When the sim replays in seconds, the expensive factory buys nothing. zk earns its cost where replay is *impossible* — hidden-information games like [Dark Forest](https://zkga.me/) use it to prove facts about state without revealing the state. If CrawlCipher ever adds sealed-information mechanics, zk becomes a candidate for exactly that slice — not for replacing replay.

---

## Determinism

### How can a game session replay bit-identically on a different machine?

Because the simulation is built for it from day one: integer grid state, a fixed tick rate, no wall-clock time inside the sim, a single seeded RNG, and a ban on floating-point math in game logic. Same seed + same inputs ⇒ same final state, on any machine. Details: [Deterministic Physics](./Development/Deterministic-Physics.md).

### Where does in-game randomness come from?

From the Stellar network itself: for online sessions, the hash of the ledger closed right after the on-chain session lock is fetched and reduced to a 64-bit seed (currently via the Horizon API; an RPC migration is on the roadmap). See [Dynamic Entropy via Stellar Ledger](./Development/Anti-Cheat-Verification.md#2-dynamic-entropy-via-stellar-ledger).

### Isn't relying on `System.Random` fragile across .NET versions?

No longer — the engine dropped `System.Random` for its own **`DeterministicRng`** (xoshiro256\*\* seeded via splitmix64, implemented from the public-domain reference algorithms in `Rng/DeterministicRng.cs`), so determinism depends only on CrawlCipher's own code, never on a runtime's RNG implementation. This also fixed the previous 64-bit→32-bit seed truncation: the engine now consumes the full 64-bit seed.

### How do you tell a modified client apart from an unmodified one?

Trick question — we don't, and we don't need to. The client is *never* trusted: only outcomes that the canonical engine reproduces from the logged inputs are accepted. A modified client that produces valid outcomes is indistinguishable from a keyboard; a modified client that produces invalid ones fails replay. Client integrity checks, anti-cheat drivers, kernel modules — none of that is needed or wanted.

### Everything runs on my machine — can't I read hidden information (maphack)?

Yes, you could — which is why **fairness is never allowed to depend on client-side hidden information**. That is an architecture rule, not a hope. Whatever must stay hidden in future designs (e.g. sealed moves between players) belongs on-chain behind [commit–reveal schemes](https://en.wikipedia.org/wiki/Commitment_scheme), not in client memory.

---

## Architecture Choices

### Why is the engine C# but the client Rust — and what is NativeAOT for?

Candidly: C# because it is one of the most familiar game-development languages (the Unity/Godot world), which keeps the door open for binding the engine to other frontends someday. Rust with [ratatui](https://ratatui.rs/) because the goal was a game you can open in any terminal, anywhere, instantly — and a cell-based UI that redraws only what changed is exactly ratatui's home turf. The glue is [.NET NativeAOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/): it compiles the C# engine ahead-of-time into a plain native shared library — no .NET runtime for players to install — which the Rust binary loads directly over a C ABI, in one process. Deep dives: [System Architecture](./Development/Architecture.md) and [Memory & FFI Bridge](./Development/Memory-and-FFI-Bridge.md).

---

## Lua Modding & the Engine Contract

> [!note] Planned — not yet in the engine
> Everything in this section is settled design for the modding milestone; the Lua layer is not yet implemented. It will be coded as described here.

### How can Lua-scripted games and deterministic verification possibly coexist?

By making non-determinism unreachable rather than forbidden: mod scripts run in a **sandbox** with `os`, `io`, file/network access and `math.random` removed. The engine hands scripts a deterministic API instead — `engine.random()` (the engine's seeded PRNG) and `engine.tick` (simulation time). A mod literally cannot observe anything the replay won't also observe.

### Where does hashing happen when the game logic itself is written in Lua?

Never in Lua. Input logging and hashing stay in the engine core, below the script layer; scripts have no hashing API and no access to the input log. What *does* change: the SHA-256 of the exact Lua script bundle becomes part of the proof's config hash — the game's rules are committed into the proof itself.

### If someone mods the rules, how is their session kept apart from the standard game?

The config hash *is* the game's identity. A different script bundle (or tweaked constants) produces a different config hash, which means a different proof domain: verifiers replay it with exactly those scripts, and its results can never be mixed into another ruleset's sessions or leaderboards. Modded play is first-class and verifiable — just unmistakably itself.

### Does a mod author have to implement their own anti-cheat?

No — the rails come from the engine: input logging, hashing, seeding, sandboxing, replay verification all work identically for every game built on it. A mod author has exactly two obligations: use `engine.random()`/`engine.tick` for all randomness and timing (the sandbox enforces this), and never design fairness around client-side hidden information (documented rule; see the maphack question).

### Lua numbers are floating-point — doesn't that break determinism?

It would, if scripts called raw math-library functions (`math.sin` and friends are approximated differently by each platform's math library — see [Floating-Point Restrictions](./Development/Deterministic-Physics.md#1-floating-point-restrictions) for why). The plan is a **deterministic math surface**: the engine exposes helpers for anything risky, *implemented inside the core with its own deterministic algorithms* — not by forwarding to the OS math library, which would just move the same problem one layer down. Integer arithmetic (which modern Lua has natively) is always safe, and grid-and-tick games rarely need more.

---

## The Blockchain Layer

### Why involve a blockchain at all?

For the three things a blockchain is genuinely good at here: **unpredictable public randomness** (ledger-hash seeds that no one — including the developers — can pre-compute), **costly commitments** (session locks that make retry-grinding expensive), and **tamper-proof records** (proof hashes and ownership that outlive any server). Nothing else is on-chain; the game itself runs locally.

### Why Stellar and Soroban instead of Ethereum?

Low, predictable fees and fast finality make per-session locks practical, and [Soroban](https://developers.stellar.org/docs/build/smart-contracts/overview) contracts cover what the design needs. The optimistic fraud-proof *pattern* is borrowed from the Ethereum ecosystem; the platform is not. Component layout: [System Architecture](./Development/Architecture.md).

### Does my gameplay data leave my machine?

The simulation is local-first. What touches the network: the session lock/unlock transactions, the seed fetch, and the submitted proof hash. Input logs are kept for verification purposes — that's the deal that replaces server authority — but frame-by-frame play is not streamed anywhere during the session.

### Can I play offline, without touching the blockchain at all?

Yes — today, not someday. Starting a session without a wallet key launches **offline mode**: the seed comes from the system clock instead of a ledger hash, nothing is locked or submitted on-chain, and the run is casual — it produces no verifiable proof. Connectivity is the price of *verified* sessions only (and beacon mode, once it lands, will require it for the whole run).

### What happens when two players play against the same shared state at the same time?

Sessions themselves never collide — each is locked per player. The interesting case is shared world state (a sector, a tradable asset) that two locally-running sessions both want to update.

> [!note] Planned — not yet in the engine
> The design resolves this with **optimistic concurrency control**: every mutable on-chain record carries a version number, and a submitted result must name the version it started from. First commit wins and bumps the version; a second commit built on the stale version is rejected by the contract (with any escrowed fees refunded), and that client re-syncs and re-runs. The intended rhythm is *frequent, small on-chain writes* — fresher shared state means smaller conflict windows and a feel closer to online multiplayer, which is precisely why a fast, low-fee chain (Stellar) was chosen as the base layer.

### What lives on-chain vs. off-chain?

On-chain today: session locks (including the lock-time ledger sequence, the seed commitment) and asset/ownership records, plus stored proof records (`{ game_hash, lock_seq, unlock_seq }` per player). Off-chain, always: the entire simulation, rendering, input logs, and replay verification runs. The chain stores *commitments*, the world stores *computation* — that split is what keeps fees irrelevant.
