# The Geometry of the Strike (A* Dash)

The **Strike** is CrawlCipher's core tactical maneuver. It allows the snake to straighten out loops in its body to project its head forward in a sudden dash.

---

## 1. Algorithmic Overview

A Strike is not a simple teleport; it is a mathematical redistribution of the snake's segments. The engine calculates how much distance can be "saved" by straightening a curved portion of the body.

```
       Curved Body (Loops):                       Straightened Path (Dash):
        +---+---+                                  
        | S | S |                                  
    +---+---+---+                                  +---+---+---+---+---+---+
    | S | S |                                      | S | S | S | S | S | H |  <-- H = Projected Head
    +---+---+                                      +---+---+---+---+---+---+
    | H |                                          
    +---+                                          
```

---

## 2. Step-by-Step Execution

### Step 1: Selecting the Corner Segments
The player triggers a Strike using a focused segment. The engine evaluates the segment index range:
- `StartIndex`: Typically index `0` (the head).
- `EndIndex`: The segment index selected by the player's focus cursor.

### Step 2: Running the A* Pathfinding
The engine calculates the shortest path between the coordinates of `EndIndex` and `StartIndex` on the grid.
- **Cost Map:** The snake's own body segments are normally treated as obstacles. However, during a Strike, the segments *between* the StartIndex and EndIndex are temporarily ignored by the pathfinding grid since they are the ones being collapsed.
- **Path Calculation:** The A* algorithm returns the minimum number of cells required to connect the two points: `PathLength`.

### Step 3: Savings Calculation
The engine compares the current number of body segments in that loop against the new straightened path:
```csharp
int currentSegmentCount = EndIndex - StartIndex;
int savings = currentSegmentCount - PathLength;
```
If `savings` is greater than 0, a Strike is valid. The player has successfully created a loop that can be collapsed to project the head forward.

### Step 4: Head Projection & Collision Checks
Before executing the dash, the engine must verify safety:
1. The head's destination is calculated by moving it forward in the current heading direction by the `savings` distance.
2. The engine performs a raycast sweep along this projection path.
3. If any obstacle (a wall in non-wrap mode, another snake's body, or a snail) lies on the path, the Strike is aborted to prevent suicide.

### Step 5: Body Readjustment
If safe, the engine:
1. Moves the head coordinates directly to the destination.
2. Collapses the body loops between `StartIndex` and `EndIndex`, turning them into a straight line connecting the head to the trailing tail.
3. Consumes energy (releasing the dash).
