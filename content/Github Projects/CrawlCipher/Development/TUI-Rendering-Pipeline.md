# Custom TUI Rendering & Camera Pipeline

Building a fast, responsive grid simulation in a monospace terminal without using a traditional engine requires custom layouts, cell aspect ratio compensations, and wrap-around camera interpolation.

---

## 1. Monospace Aspect Ratio Correction

In terminal emulators, characters are not square. A standard monospace character (e.g. `Font`) is typically twice as tall as it is wide (a 1:2 aspect ratio). If a grid cell is represented by a single character (like `▒` or `@`), the game grid will appear stretched vertically.

### The Double-Character Solution:
To achieve a visually square grid board:
- Every cell on the grid is rendered using **two characters** horizontally instead of one.
- **Examples:**
  - Food: `()`
  - Empty Cell: `  ` (two spaces)
  - Focused Segment: `██`
  - Snails: `@-` or `/@`
  - Strike Destination Preview: `++`

This simple transformation makes the game board look square on standard monospace terminal configurations.

---

## 2. Viewport Culling & FFI Data Minimization

The game board size (default 87x50) can exceed the physical size of the player's terminal window. Copying the entire 4,350-cell grid across the unmanaged FFI boundary every frame is inefficient.

### Pipeline Culling:
1. The Rust TUI calculates its current window size and derives the visible coordinate bounds: `[MinX, MaxX, MinY, MaxY]`.
2. Instead of requesting the entire grid, Rust calls `GetGridCells(gamePtr, bufferPtr, maxCells)` passing only the viewport bounds.
3. The C# Core Engine filters and serializes only the cells within this bounding box into the unmanaged buffer array.
4. Rust loops through this culled buffer and renders it, reducing ABI copying costs.

---

## 3. Toroidal Camera Follow & Interpolation (Lerp)

The camera coordinates follow the focused segment (the snake's head). When the snake wraps around a toroidal border (e.g. moving past X coordinate 86 and appearing at X coordinate 0), a naive camera follow would instantly snap to the other side of the grid, causing visual whiplash.

### Toroidal Camera Lerp:
To provide a smooth visual transition, the camera uses a custom wrap-aware interpolation:
1. **Delta Calculation:** Calculate the shortest distance between the camera's current focus and the snake head's coordinates, taking the map wrap-around into account.
   ```rust
   let mut dx = head_x - camera_x;
   if dx.abs() > grid_width / 2 {
       dx = if dx > 0 { dx - grid_width } else { dx + grid_width };
   }
   ```
2. **Smoothing:** Apply linear interpolation (lerp) to adjust the camera:
   ```rust
   camera_x = (camera_x + dx * lerp_factor) % grid_width;
   ```
3. **Rendering Offset:** Apply this offset when drawing cells on the screen, causing the board to wrap around the screen boundaries smoothly without jumps.
