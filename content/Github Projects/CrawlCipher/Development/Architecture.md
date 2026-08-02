# System Architecture

CrawlCipher is designed as a hybrid dApp split into three decoupled components: an interactive **Rust TUI** wrapper, a deterministic **C# NativeAOT** engine, and a **Soroban Smart Contract** on the Stellar blockchain.

---

## Architecture Diagram

The overall structure and interface bridges are diagrammed below:

```
+-----------------------------------------------------------------------+
|                       RUST TUI (crawlcipher)                          |
|  +-----------------------+     +------------------+     +----------+  |
|  |  Terminal GUI         |     |  Dynamic Loader  |     | Stellar  |  |
|  |  (Ratatui/Crossterm)  |===> |  (libloading)    |     | API & CLI|  |
|  +-----------+-----------+     +--------+---------+     +----+-----+  |
+--------------|--------------------------|-------------------|---------+
               |                          |                   |
               | (Reads Block Hash)       | (Invokes C-ABI)   | (Invokes contract)
               v                          v                   v
+--------------|--------------------------|-------------------|---------+
|              |                          |                   |         |
|  +-----------v-----------+     +--------v---------+     +---v------+  |
|  |  Deterministic Seed   |     |  C-ABI FFI Layer |     | Soroban  |  |
|  |  Entropy Loader       |     |  (FFIExports)    |     | Contract |  |
|  +-----------+-----------+     +--------+---------+     +----------+  |
|              |                          |                             |
|              v                          v                             |
|  +-----------------------+     +--------+---------+                   |
|  |  Deterministic RNG    |===> |  Game Core       |                   |
|  |  (System.Random)      |     |  (Grid & Snake)  |                   |
|  +-----------------------+     +--------+---------+                   |
|                                         |                             |
|                                         v                             |
|                                +------------------+                   |
|                                |  Input Log       |                   |
|                                |  (Replay Record) |                   |
|                                +------------------+                   |
|                   C# CORE ENGINE (NativeAOT Shared Library)           |
+-----------------------------------------------------------------------+
```

---

## 1. Subsystems

### A. Terminal Interface (Rust / Ratatui)
The frontend terminal layout is built in Rust using the [`ratatui`](https://ratatui.rs/) crate. It coordinates:
- **UI & Input:** Capture inputs (keyboard/mouse) via [`crossterm`](https://crates.io/crates/crossterm) raw mode, map mouse click coordinates to grid coordinates, and render status bars, game boards, menus, and overlays.
- **Dynamic Link Loading:** Instead of compile-time linking, the frontend dynamically loads `libCrawlCipher.Core.so` (Linux) or `CrawlCipher.Core.dll` (Windows) using the [`libloading`](https://crates.io/crates/libloading) crate. This allows swapping core logic without recompiling the UI binary.
- **Blockchain Connectivity:** Interacts with the Stellar network by calling [Horizon REST APIs](https://developers.stellar.org/docs/data/apis/horizon) directly (for account stats and block hashes) and spawning the [`stellar` CLI](https://developers.stellar.org/docs/tools/cli) tool to interact with Soroban contracts.

> [!note] Planned — not yet in the client
> Stellar has deprecated Horizon in favor of [Stellar RPC](https://developers.stellar.org/docs/data/apis/migrate-from-horizon-to-rpc); migrating the client's REST calls is tracked internally.

### B. Simulation Core (C# / .NET 8.0 NativeAOT)
The underlying simulation state and logic are written in C# (`CrawlCipher.Core`).
- **Native AOT:** Compiled using Microsoft's [.NET NativeAOT](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/) toolchain. This publishes the managed C# library as a standard, self-contained unmanaged shared library (DLL/SO) with zero .NET runtime dependency requirements on the host machine.
- **Deterministic Loop:** Tracks coordinates, snakes, entities, grids, and tick counts. To remain 100% replayable and fair, it forbids calling `DateTime.Now` or system timers, relying solely on a seed-linked `System.Random` generator.
- **C-ABI Export Layer:** Exposes unmanaged entry points via `[UnmanagedCallersOnly]` so the Rust frontend can map struct layouts sequentially in memory and invoke actions.

### C. Soroban Session Lock Contract (Rust / WASM)
A [Soroban](https://developers.stellar.org/docs/build/smart-contracts/overview) smart contract deployed on the Stellar testnet (`smart-contracts/session-lock`):
- **Asset Lock (`lock_session`):** Locks a list of game items (identified by asset IDs) for a user address to prevent trading or double-spending during an active game session.
- **Release and Prove (`unlock_session`):** Releases the lock when the client submits the final cryptographic session verification hash.

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
