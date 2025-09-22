# FifteenPuzzleSolver

## 1. Introduction

A command‑line solver for the classic *n×n sliding puzzle* (default shown as 15‑puzzle). The solver reads an initial board from a file and writes the sequence of moves to reach the goal state.

---

## 2. Highlights

* **Search strategy:** Greedy Best‑First Search (GBFS) over puzzle states
* **Heuristic (default):** Composite of Manhattan distance, linear conflict, and misplaced tiles (a.k.a. "walking distance" in this codebase)
* **Pluggable heuristics:** Implemented options include *Manhattan + Linear Conflict*, *Linear Conflict only*, *Misplaced tiles*, and a *Weighted* combination (see below)
* **Deterministic moves output:** Each step lists the moved tile and direction (`L/R/U/D`)

> **Note:** Although `Node` implements `f(n) = g(n) + h(n)`, the current `Solver` prioritizes **`h(n)` only** in the priority queue comparator, so the behavior is **GBFS** (not A\*). See [Switching to A\*](#switching-to-a) to use `f(n)`.

---

## 3. Project Structure

```
src/
 └─ fifteenpuzzle/
     ├─ Node.java     # State representation, neighbors, heuristics
     ├─ Pair.java     # Minimal generic pair utility
     └─ Solver.java   # File I/O, search loop, path reconstruction
```

---

## 4. Problem Definition

Given an `n×n` grid containing tiles `1..n²−1` and a single empty space `0`, find a sequence of moves that transforms an initial configuration into the **goal** configuration:

```
1  2  3  ... n
...          
...     n²-1 0
```

Moves slide a tile **adjacent to the empty space** into that space (up/down/left/right). The solver outputs the moved tile and the direction of its slide.

---

## 5. Build & Run

### Requirements

* Java 17+ (any modern JDK)

### Compile

```bash
javac -d out src/fifteenpuzzle/*.java
```

### Run

```bash
java -cp out fifteenpuzzle.Solver <input_file> <output_file>
```

* `<input_file>`: text file describing the initial board
* `<output_file>`: text file to receive the move sequence

---

## 6. File Formats

### Input format

First line: **board size** `n`. Next `n` lines: tile numbers separated by spaces. Use `0` for the blank.

Example (`4×4`):

```
4
5  1  2  3
9  6  7  4
13 10 11 8
0  14 15 12
```

### Output format

One move per line: `TILE DIRECTION`, where direction is one of:

* `L` (left), `R` (right), `U` (up), `D` (down)

Example:

```
14 U
10 R
6  D
...
```

Each line indicates **which numbered tile moved** and **in which direction it slid** to occupy the blank.

---

## 7. Search Algorithm (Current)

* **Open set:** Java `PriorityQueue<Node>` ordered by **`h(n)` only** (`Node.getHScore()`)
* **Visited:** `Map<Integer, Node>` keyed by `Node.hashCode()` to avoid revisiting states
* **Parent links:** Used to reconstruct the final path
* **Cost tracking:** `g(n)` increments by 1 per move (kept on `Node`), but not used by the queue comparator

> This configuration makes the solver a **Greedy Best‑First Search**: fast in practice but **not guaranteed to find the shortest path**.

---

## 8. Heuristics Implemented

Heuristics estimate distance from a node `s` to the goal `G`.

### 1) Composite "Walking Distance" (default)

```
h(s) = Manhattan(s,G) + LinearConflict(s,G) + MisplacedTiles(s,G)
```

* **Manhattan distance:** Sum of |Δrow| + |Δcol| for each tile
* **Linear conflict:** Adds 2 for each pair of tiles in the same row/column that are reversed relative to their goal order
* **Misplaced tiles:** Count of tiles not already in goal position

### 2) Manhattan + Linear Conflict

```
h(s) = Manhattan(s,G) + LinearConflict(s,G)
```

Defined in `Node.computeMD(...)`.

### 3) Linear Conflict only

```
h(s) = LinearConflict(s,G)
```

Defined in `Node.computeLinearConflict(...)`.

### 4) Misplaced Tiles only

```
h(s) = MisplacedTiles(s,G)
```

Defined in `Node.computeMisplacedTiles(...)`.

### 5) Weighted Composite

```
h(s) = 1.0·Manhattan + 1.5·LinearConflict + 0.1·Misplaced
```

Defined in `Node.computeWeightedHeuristic(...)`.

> You can swap heuristics by changing the calls in `Solver` where `hScore` is set for the start node and neighbors.

---

## 🔁 Switching to A\*

If you want optimal solutions (fewest moves), change the priority queue to order by **`f(n) = g(n) + h(n)`**:

In `Solver`:

```java
// Current (GBFS):
openSet = new PriorityQueue<>(Comparator.comparingInt(Node::getHScore));

// A*: use f(n)
openSet = new PriorityQueue<>(Comparator.comparingInt(Node::getFScore));
```

Make sure you continue setting `g` and `h` for each neighbor:

```java
neighbor.setGScore(current.getGScore() + 1);
neighbor.setHScore(/* choose heuristic here */);
```

With an **admissible** heuristic (e.g., Manhattan + Linear Conflict), A\* will return shortest paths.

---

## 9. Example

Given the input above, run:

```bash
java -cp out fifteenpuzzle.Solver sample4x4.in moves.out
```

`moves.out` will contain the tile‑by‑tile move list to reach the canonical goal state.

---

## 10. Design Notes

* `Node.equals(...)` uses deep array equality on tile grids
* `Node.hashCode()` is consistent with `equals` for use in maps/sets
* `Node.getNeighbors()` generates up to 4 states by sliding the blank
* Move labeling (`L/R/U/D`) is derived from the relative positions of the blank and the moved tile

---

## 11. Testing Ideas

* **Solvability checks:** Add a parity test to reject unsolvable inputs early
* **Regression seeds:** Keep a folder of known puzzles with expected move counts
* **Performance:** Track nodes expanded and wall‑clock times for 3×3, 4×4, 5×5

---

## 12. Known Limitations & Future Work

* **Optimality:** GBFS may find a solution quickly but not the fewest moves
* **Memory:** Large boards can explode in state space; consider IDA\* for 5×5+
* **Heuristics:** Consider implementing pattern databases (e.g., 7‑8‑pattern DB) for stronger guidance
* **Visited structure:** A `HashSet<Node>` could be used directly (with `equals/hashCode`) instead of hashing to `Map<Integer,Node>`

---

## 13. License

Choose a license (e.g., MIT) and add a `LICENSE` file if making the project public.

---

## 14. Acknowledgments

Classic 15‑puzzle literature and community resources on Manhattan distance, linear conflicts, and A\*.
