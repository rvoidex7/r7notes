# System Architecture

CrawlCipher is designed as a hybrid dApp split into three decoupled components: an interactive **Rust TUI** wrapper, a deterministic **C# NativeAOT** engine, and a **Soroban Smart Contract** on the Stellar blockchain.

---

## Architecture Diagram

The overall structure and interface bridges are diagrammed below:

```mermaid
graph TD
    %% Frontend Group
    subgraph Rust TUI (crawlcipher)
        TUI[Terminal GUI - Ratatui]
        Loader[Dynamic Library Loader - libloading]
        StellarAPI[Stellar Interface - stellar-cli & reqwest]
    end

    %% Engine Group
    subgraph C# Core Engine (CrawlCipher.Core.so)
        FFI[FFI Export Layer]
        Engine[Game Engine - Grid & Snake State]
        RNG[Deterministic RNG]
        Logger[Input Log recorder]
    end

    %% Blockchain Group
    subgraph Stellar Testnet Blockchain
        Horizon[Stellar Horizon API]
        Contract[Soroban Contract - session-lock]
    end

    %% Interactions
    TUI -->|Loads| Loader
    Loader -->|Invokes C ABI| FFI
    FFI -->|Updates & Inputs| Engine
    Engine -->|Reads| RNG
    Engine -->|Logs| Logger
    
    TUI -->|Fetches Latest Block Hash| Horizon
    Horizon -->|Seed Entropy| TUI
    TUI -->|Initializes Engine with Seed| Loader
    
    TUI -->|Lock Loadout / Submit Proof| StellarAPI
    StellarAPI -->|Invokes Contract| Contract
```

---

## 1. Subsystems

### A. Terminal Interface (Rust / Ratatui)
The frontend terminal layout is built in Rust using the `ratatui` crate. It coordinates:
- **UI & Input:** Capture inputs (keyboard/mouse) via `crossterm` raw mode, map mouse click coordinates to grid coordinates, and render status bars, game boards, menus, and overlays.
- **Dynamic Link Loading:** Instead of compile-time linking, the frontend dynamically loads `libCrawlCipher.Core.so` (Linux) or `CrawlCipher.Core.dll` (Windows) using the `libloading` crate. This allows swapping core logic without recompiling the UI binary.
- **Blockchain Connectivity:** Interacts with the Stellar network by calling Horizon REST APIs directly (for account stats and block hashes) and spawning the `stellar` CLI tool to interact with Soroban contracts.

### B. Simulation Core (C# / .NET 8.0 NativeAOT)
The underlying simulation state and logic are written in C# (`CrawlCipher.Core`).
- **Native AOT:** Compiled using Microsoft's .NET NativeAOT toolchain. This publishes the managed C# library as a standard, self-contained unmanaged shared library (DLL/SO) with zero .NET runtime dependency requirements on the host machine.
- **Deterministic Loop:** Tracks coordinates, snakes, entities, grids, and tick counts. To remain 100% replayable and fair, it forbids calling `DateTime.Now` or system timers, relying solely on a seed-linked `System.Random` generator.
- **C-ABI Export Layer:** Exposes unmanaged entry points via `[UnmanagedCallersOnly]` so the Rust frontend can map struct layouts sequentially in memory and invoke actions.

### C. Soroban Session Lock Contract (Rust / WASM)
A smart contract deployed on the Stellar testnet (`smart-contracts/session-lock`):
- **Asset Lock (`lock_session`):** Locks a list of game items (identified by asset IDs) for a user address to prevent trading or double-spending during an active game session.
- **Release and Prove (`unlock_session`):** Releases the kilit (lock) when the client submits the final cryptographic session verification hash.

---

## 2. The FFI Bridge (C-ABI)

The Rust TUI interacts with the C# core through a thin foreign function interface (FFI) layer. Data layouts must match exactly across the compiler boundary:

### Struct Alignments
- **C# representation (`[StructLayout(LayoutKind.Sequential)]`)** ensures variables are arranged sequentially in memory.
- **Rust representation (`#[repr(C)]`)** maps these byte arrays exactly.

### Example FFI Export (C#)
```csharp
[UnmanagedCallersOnly(EntryPoint = "ProcessInput")]
public static unsafe void ProcessInput(IntPtr gamePtr, int inputType, int param1, int param2)
{
    var handle = GCHandle.FromIntPtr(gamePtr);
    var engine = (GameEngine)handle.Target!;
    engine.ProcessInput(inputType, param1, param2);
}
```

### Example FFI Loader (Rust)
```rust
type FnProcessInput = unsafe extern "C" fn(*mut c_void, i32, i32, i32);
// ...
let process_input_fn: Symbol<FnProcessInput> = lib.get(b"ProcessInput").expect("Missing ProcessInput");
```
