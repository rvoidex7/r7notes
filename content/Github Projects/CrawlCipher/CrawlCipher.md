# CrawlCipher Knowledge Base

Welcome to the documentation for **CrawlCipher**, a full-stack, terminal-based dApp built to demonstrate deterministic state hashing, cryptographic fairness, and Soroban smart contract session locking on the Stellar blockchain.

This knowledge base is divided into two main sections: **Wiki (For Gamers)** and **Development (For Developers)**.

---

## 🎮 [Wiki & Game Mechanics (For Gamers)](./Wiki/Gameplay.md)
Everything about how to play the game, what items do, and the rules of the tactical environment:
- **[Gameplay Overview & Controls](./Wiki/Gameplay.md):** The Chess-and-Snake style movement system.
- **[Energy & Movement Math](./Wiki/Energy-and-Movement.md):** Detailed energy consumption rules, turn cost logic, and overflow mechanics.
- **[Weapons, Modules & Moving Circuit Synergies](./Wiki/Items-and-Synergies.md):** Using weapons (Pistols, Lasers, Rifles) and lining up body segments to trigger 3x3 combos.
- **[Entities & Hazards](./Wiki/Entities.md):** Enemy snails, bullets, bot entities, and the extraction portals.

---

## 🛠️ [Technical Architecture & Development (For Devs)](./Development/Architecture.md)
Details of the engineering decisions, FFI layer, cryptographic verification, and future ideas:
- **[System Architecture](./Development/Architecture.md):** Component interaction between the Rust TUI, C# NativeAOT core, and Soroban contract.
- **[Cryptographic Anti-Cheat & Proof of Execution](./Development/Anti-Cheat-Verification.md):** Deterministic RNG, Horizon entropy extraction, input frame log replay hashes.
- **[Engine Code Walkthrough](./Development/Engine-Code-Walkthrough.md):** Detailed walkthrough of the `Models/` data structures, the partial-class `GameEngine` state machine, and FFI handle pinning.
- **[Memory & FFI Bridge](./Development/Memory-and-FFI-Bridge.md):** Pinned unmanaged handles (`GCHandle`), sequential memory mappings, and byte-level serialization.
- **[Deterministic Physics](./Development/Deterministic-Physics.md):** Floating-point bans, seed-based RNG behavior, and fixed-timestep execution.
- **[The Strike Algorithm Geometry](./Development/Strike-Algorithm-Geometry.md):** Corner segments selection, A* path savings calculation, and collision projection.
- **[Custom TUI & Camera Pipeline](./Development/TUI-Rendering-Pipeline.md):** Viewport culling, monospace aspect ratio square compensation, and toroidal Camera Lerp.
- **[Deterministic Bot AI & Steering](./Development/Bot-AI-Steering.md):** Raycasted obstacle avoidance, decision priority matrix, and perpendicular firing vectors.

---

## ❓ [FAQ — How Does This Actually Work?](./FAQ.md)
The questions a technically-minded visitor asks first — trust, open source, cheating, determinism, Lua modding, the blockchain layer — with honest answers and links into the deep-dive pages. Planned-but-not-yet-coded designs are explicitly marked.

---

## 🗺️ Visual Project Map
For a visual layout of the project, see the [CrawlCipher Mindmap Canvas](./CrawlCipher_Mindmap.canvas).
