# CrawlCipher Knowledge Base

Welcome to the documentation for **CrawlCipher**, a full-stack, terminal-based dApp built to demonstrate deterministic state hashing, cryptographic fairness, and Soroban smart contract session locking on the Stellar blockchain.

This knowledge base is divided into two main sections: **Wiki (For Gamers)** and **Development (For Developers)**.

---

## 🎮 [[Gameplay|Wiki & Game Mechanics (For Gamers)]]
Everything about how to play the game, what items do, and the rules of the tactical environment:
- **[[Gameplay|Gameplay Overview & Controls]]:** The Chess-and-Snake style movement system.
- **[[Energy-and-Movement|Energy & Movement Math]]:** Detailed energy consumption rules, turn cost logic, and overflow mechanics.
- **[[Items-and-Synergies|Weapons, Modules & Moving Circuit Synergies]]:** Using weapons (Pistols, Lasers, Rifles) and lining up body segments to trigger 3x3 combos.
- **[[Entities|Entities & Hazards]]:** Enemy snails, bullets, bot entities, and the extraction portals.

---

## 🛠️ [[Architecture|Technical Architecture & Development (For Devs)]]
Details of the engineering decisions, FFI layer, cryptographic verification, and future ideas:
- **[[Architecture|System Architecture]]:** Component interaction between the Rust TUI, C# NativeAOT core, and Soroban contract.
- **[[Anti-Cheat-Verification|Cryptographic Anti-Cheat & Proof of Execution]]:** Deterministic RNG, Horizon entropy extraction, input frame log replay hashes.
- **[[Engine-Code-Walkthrough|Engine Code Walkthrough]]:** Detailed walkthrough of the `Models/` data structures, the partial-class `GameEngine` state machine, and FFI handle pinning.
- **[[Memory-and-FFI-Bridge|Memory & FFI Bridge]]:** Pinned unmanaged handles (`GCHandle`), sequential memory mappings, and byte-level serialization.
- **[[Deterministic-Physics|Deterministic Physics]]:** Floating-point bans, seed-based RNG behavior, and fixed-timestep execution.
- **[[Strike-Algorithm-Geometry|The Strike Algorithm Geometry]]:** Corner segments selection, A* path savings calculation, and collision projection.
- **[[TUI-Rendering-Pipeline|Custom TUI & Camera Pipeline]]:** Viewport culling, monospace aspect ratio square compensation, and toroidal Camera Lerp.
- **[[Bot-AI-Steering|Deterministic Bot AI & Steering]]:** Raycasted obstacle avoidance, decision priority matrix, and perpendicular firing vectors.

---

## ❓ [[FAQ|FAQ — How Does This Actually Work?]]
The questions a technically-minded visitor asks first — trust, open source, cheating, determinism, Lua modding, the blockchain layer — with honest answers and links into the deep-dive pages. Planned-but-not-yet-coded designs are explicitly marked.

---

## 🗺️ Visual Project Map
For a visual layout of the project, see the [[CrawlCipher_Mindmap.canvas|CrawlCipher Mindmap Canvas]].
