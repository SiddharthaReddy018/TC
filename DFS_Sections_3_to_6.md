# Graph Traversal — DFS Complete Teaching Notes
### Sections 3 through 6 | From First Principles

---

## Section 3 — DFS: Algorithm, Variables & Complexity

### Core Intuition

BFS explored the graph **level by level** — like ripples on water. DFS does the opposite: it goes **as deep as possible** down one path before backtracking. Think of it like navigating a maze by always taking the first unexplored corridor, and only turning back when you hit a dead end.

The key structural difference:
- BFS uses a **queue** (FIFO) — process vertices in the order you discover them.
- DFS uses a **stack** (LIFO) — the most recently discovered vertex is explored next. In the recursive implementation, the call stack *is* the stack.

---

### Variables DFS Tracks (per vertex)

DFS tracks three things per vertex (see slide "Depth-First Search" — variables):

| Variable | Meaning |
|---|---|
| `color[v]` | white = undiscovered, gray = in progress, black = fully done |
| `d[v]` | **discovery time** — the timestamp when `v` first turns gray |
| `f[v]` | **finish time** — the timestamp when `v` turns black (all its neighbors explored) |
| `π[v]` | predecessor of `v` in the DFS tree |

The global variable `time` is a counter that increments every time a vertex is discovered or finished. Over the entire run, `time` goes from 1 to `2n` (each of the `n` vertices gets one discovery tick and one finish tick).

---

### The Two-Function Structure

DFS is split into two functions. Understanding *why* will save you from confusing yourself later.

**`DFS(G)`** — the outer driver. It loops over every vertex, and for any that are still white, kicks off a fresh DFS from there. This is what makes DFS work on **disconnected graphs** — BFS from a single source would miss unreachable components.

```
DFS(G):
  for each u ∈ V[G]:
    color[u] = white
    π[u] = NIL
  time = 0
  for each u ∈ V[G]:
    if color[u] = white:
      DFS-VISIT(u)
```

**`DFS-VISIT(u)`** — the recursive workhorse. It stamps `u`'s discovery time, turns it gray, recurses on all unvisited neighbors, then stamps the finish time and turns it black.

```
DFS-VISIT(u):
  color[u] = gray
  time = time + 1
  d[u] = time
  for each v ∈ Adj[u]:
    if color[v] = white:
      π[v] = u
      DFS-VISIT(v)          ← recurse deep before coming back
  color[u] = black
  f[u] = time
  time = time + 1
```

Look at the slide diagram for `DFS-VISIT(u)` — it shows `u` calling `DFS-VISIT(v)`, which calls `DFS-VISIT(w)`, and so on. The call stack reflects the current "path" being explored. When `w` finishes, control returns to `v`, then to `u`. This nesting is fundamental — it gives rise to the **Parenthesis Theorem** in Section 4.

---

### Worked Example

Using the graph on the example slide (vertices a, b, c, d, e, f):

DFS starts at `a` (d[a]=1), goes to b (d[b]=2), to c (d[c]=3), to d (d[d]=4).  
`d` has no unvisited neighbors → f[d]=5, turns black.  
Back to `c` → f[c]=6. Back to `b` → f[b]=7. Back to `a` → f[a]=8.  
Now `e` is still white → DFS-VISIT(e): d[e]=9, goes to f: d[f]=10, f[f]=11, f[e]=12.

Final table:

```
Vertex | d[]  | f[]
  a    |  1   |  8
  b    |  2   |  7
  c    |  3   |  6
  d    |  4   |  5
  e    |  9   | 12
  f    | 10   | 11
```

Notice the nesting: a's interval [1,8] contains b's [2,7], which contains c's [3,6], which contains d's [4,5]. This is the parenthesis structure — explained in Section 4.

---

### Time Complexity

From the handwritten notes (page 6 of the second PDF):

**Initialization loop** in `DFS(G)`: touches every vertex once → **O(n)**

**`DFS-VISIT` calls**: each vertex is visited exactly once (once it turns gray, it's never white again, so `DFS-VISIT` is never called on it again). The call itself (excluding the inner loop) takes O(1) per vertex → **O(n) total** for all calls.

**Inner `for` loop across all calls**: when `DFS-VISIT(u)` runs, it iterates over `Adj[u]`. Across *all* calls, the total iterations = sum of all degree sizes = **O(m)** (by the handshaking lemma: Σ deg(v) = 2m for undirected, or m for directed).

**Total: O(n + m)** — same as BFS.

> ⚠️ **Common mistake**: Students often think DFS-VISIT is called multiple times per vertex. It isn't — the `color[v] = white` check prevents this. The `O(m)` cost comes from the adjacency list scans, not from repeated visits.

---

## Section 4 — DFS Properties: Forest, Parenthesis Theorem, White-Path Theorem

### Property 1: The Predecessor Graph Forms a Forest

When DFS finishes, the predecessor pointers `π[v]` form a **DFS forest** — a collection of trees, one per call to `DFS-VISIT` from the outer loop of `DFS(G)`.

Formally: `u = π[v]` if and only if `DFS-VISIT(v)` was called while processing `u`'s adjacency list.

If the graph is connected, this forest is a single tree (the **DFS tree**). If not, you get one tree per connected component (or per strongly connected component, for directed graphs).

---

### Property 2: The Parenthesis Theorem

This is the most important structural theorem about DFS. It tells you exactly how discovery/finish intervals relate to the ancestor-descendant relationship in the DFS forest.

**Theorem**: For any two vertices `u` and `v`, exactly one of these three cases holds:

```
Case 1: Intervals are DISJOINT
  ──[d[u]──f[u]]──────────[d[v]──f[v]]──
  Neither is an ancestor of the other.

Case 2: u's interval is INSIDE v's interval
  ──[d[v]────[d[u]──f[u]]────f[v]]──
  u is a descendant of v.

Case 3: v's interval is INSIDE u's interval
  ──[d[u]────[d[v]──f[v]]────f[u]]──
  v is a descendant of u.
```

**Why no partial overlap?** When DFS discovers `v` while `u` is gray (i.e., `d[u] < d[v]` and `u` is still on the call stack), `v` was reached *through* `u`'s recursion. DFS won't return to finish `u` until the entire sub-call rooted at `v` completes. So `f[v] < f[u]` always — no straddling possible.

**Corollary (Nested Interval = Descendant)**:
`v` is a proper descendant of `u` if and only if:
```
d[u] < d[v] < f[v] < f[u]
```
This is a clean, testable condition. Given a table of d/f values, you can determine the entire ancestor-descendant structure of the DFS forest instantly.

**Proof sketch** (see the handwritten pages 11 and the slide on Case 1/Case 2):

Case d[u] < d[v]:
- Sub-case f[u] < d[v]: intervals are disjoint → neither ancestor of other. ✓
- Sub-case f[u] > d[v]: At the time v was discovered, u was still gray (on the stack). Since v was discovered during u's active period, v is a descendant of u, so f[v] < f[u]. ✓

---

### Property 3: White-Path Theorem

**Theorem**: In a DFS of graph G, vertex `v` is a descendant of `u` in the DFS forest **if and only if** at the moment `d[u]` (when `u` is first discovered), there exists a path from `u` to `v` in G consisting entirely of **white** (undiscovered) vertices.

**Intuition**: The moment you discover `u`, look at what's reachable through white vertices. DFS will greedily consume all of it. Anything reachable through an all-white path will eventually become a descendant of `u` in the DFS tree.

**Why "white path" specifically?** Because DFS only recurses into white vertices. If at time `d[u]`, some vertex `v` on the path to `u` is already gray or black, DFS won't go through it — the path is effectively blocked.

**Proof idea** (from the handwritten notes, pages 14–18):

(⇒) If `v` is a descendant of `u`: Take the path from `u` to `v` in the DFS tree. Every vertex on this path was white at time `d[u]` (by the parenthesis theorem — their discovery times are all > d[u]). So there's a white path. ✓

(⇐) Assume a white path `u → x₁ → x₂ → ... → v` exists at time `d[u]`, but assume for contradiction that `v` is **not** a descendant of `u`. Let `v` be the first vertex on the path that is not a descendant of `u`, and let `w` be its predecessor on the path (so `w` IS a descendant of `u`). Since `(w,v) ∈ E` and `w` is a descendant of `u`:
- `d[u] < d[v] < f[w]` (v was white, then discovered before w finished)
- `f[w] ≤ f[u]` (w is a descendant of u)
- Therefore `d[u] < d[v] < f[u]` → by Parenthesis Theorem, v's interval is inside u's → `v` IS a descendant of `u`. Contradiction. ✓

---

## Section 5 — DFS Edge Classification

### Why Classify Edges?

When DFS runs, it encounters edges. Not every edge becomes part of the DFS tree. Classifying what *kind* of edge an edge is gives deep structural insight into the graph — and it's the foundation for detecting cycles, topological sort, and strongly connected components.

---

### The Four Edge Types

When DFS explores edge `(u, v)`, it checks the color of `v`:

**1. Tree Edge**
Edge `(u, v)` where `v` was **white** when discovered through `u`.
These form the DFS forest itself.
```
u ──(tree edge)──► v   (v was white, π[v] = u)
```

**2. Back Edge**
Edge `(u, v)` where `v` is an **ancestor** of `u` (v is currently **gray**).
These create cycles. In an undirected graph, every non-tree edge is a back edge.
```
u ──(back edge)──► v   (v is gray = still on the stack = ancestor of u)
```

**3. Forward Edge**
Edge `(u, v)` where `v` is a **descendant** of `u`, but NOT through the tree edge (a "shortcut" down). `v` is **black** and `d[u] < d[v]`.
These only appear in **directed** graphs.

**4. Cross Edge**
All other edges — either between different DFS trees, or between vertices in the same tree that have no ancestor-descendant relationship. `v` is **black** and `d[v] < d[u]`.
These also only appear in **directed** graphs.

---

### How to Identify Edge Type During DFS

When you're at vertex `u` and about to explore edge `(u, v)`:

```
if color[v] = white   →  Tree edge
if color[v] = gray    →  Back edge
if color[v] = black:
    if d[u] < d[v]    →  Forward edge
    if d[v] < d[u]    →  Cross edge
```

### Example (from the slide diagram, page 7 of PDF 2)

The graph has edges labeled T (tree), B (back), F (forward), C (cross). Look at the example:
- a→b: T (tree), b→c: T, c→d: T, d→b: B (back — creates a cycle), a→d: F (forward — shortcut from a to d skipping tree path), e→f: T (separate DFS tree starting at e since e was unreachable from a's tree before it was visited).

---

### Key Theorem: Undirected Graphs Have No Forward/Cross Edges

In an undirected graph, DFS only produces **tree edges** and **back edges**. No forward or cross edges.

**Why?** In undirected graphs, `(u,v)` and `(v,u)` are the same edge. If we're processing `(u,v)` and `v` is already black, we would have already processed `(v,u)` earlier — at that point, `u` would have been gray (ancestor of v's processing), so we'd classify it as a back edge from v's perspective. The "cross edge" scenario can't arise.

---

### Back Edges ↔ Cycles

**Theorem**: A directed graph G has a cycle **if and only if** DFS reveals a back edge.

This is extremely useful: to detect a cycle in a directed graph, just run DFS and check if any edge `(u,v)` has `v` currently gray.

---

## Section 6 — DFS: Proof of Correctness & Topological Sort

### DFS Correctness Claim (from your handwritten notes)

**Claim**: If there exists a path from source vertex `s` to vertex `v`, then DFS starting at `s` will visit `v`.

**Proof** (by induction on the path length, as in your notes):

Let the path be `s = v₀ → v₁ → v₂ → ... → vₖ = v`.

**Base case**: `v₀ = s`. DFS starts at `s` and visits it immediately. ✓

**Inductive step**: Assume DFS visits `vᵢ`. We need to show it visits `vᵢ₊₁`.

Since there is an edge `(vᵢ, vᵢ₊₁)`, when DFS processes `vᵢ`'s adjacency list, it will encounter `vᵢ₊₁`.

- If `vᵢ₊₁` is **white**: DFS-VISIT calls it immediately → visited. ✓
- If `vᵢ₊₁` is **gray**: it's currently on the stack, meaning it was already visited. ✓
- If `vᵢ₊₁` is **black**: it was already visited and finished. ✓

In all cases, `vᵢ₊₁` is visited. ✓

**Therefore**: Each vertex is visited at most once (the `color` check ensures this). If DFS(v) is called, it becomes visited (gray), and once visited, it will never be re-visited.

> This also implies: **every vertex is visited at most once**, and the total work done is O(n + m).

---

### Topological Sort

**What it is**: A **topological ordering** of a directed acyclic graph (DAG) is a linear ordering of all its vertices such that for every directed edge `(u, v)`, `u` appears before `v` in the ordering.

Think of it as scheduling: if task A must happen before task B (edge A→B), a topological sort gives you a valid execution order.

**Key fact**: Topological sort is only defined on **DAGs** (graphs with no directed cycles). If there's a cycle, no valid ordering exists.

**Algorithm using DFS** (from your notes):

```
TopologicalSort(G):
  Run DFS(G)
  As each vertex v is FINISHED (turns black), prepend v to a list
  Return the list
```

Equivalently: output vertices in **decreasing order of finish time** `f[v]`.

**Time Complexity**: O(n + m) — just the cost of DFS.

**Why does this work?**

For any edge `(u, v)` in a DAG, we claim `f[v] < f[u]`.

**Proof**: When we explore edge `(u, v)`:
- If `v` is white: `v` becomes a descendant of `u`, so by the parenthesis theorem, `f[v] < f[u]`. ✓
- If `v` is gray: `v` is an ancestor of `u`, meaning there's a path `v → ... → u`. Combined with `u → v`, we have a cycle. But G is a DAG — contradiction. So this case can't happen. ✓
- If `v` is black: `v` is already finished, so `f[v]` is already set and < current time < `f[u]`. ✓

In all valid cases (no back edges in a DAG), `f[v] < f[u]`. So placing vertices in decreasing finish order puts `u` before `v` for every edge `(u,v)`. ✓

---

### Connection: Back Edges and DAGs

The above proof shows the tight link between these concepts:

```
G is a DAG
  ⟺  DFS on G produces NO back edges
  ⟺  Topological Sort is possible
```

Detecting whether a graph is a DAG = running DFS and checking for back edges. If none exist, sort by decreasing finish time and you have a valid topological order.

---

## Big Picture: How Everything Connects

```
DFS(G)
├── Produces a DFS Forest (predecessor pointers π[v])
│   └── One DFS tree per connected component
│
├── Records d[v] and f[v] for every vertex
│   └── Parenthesis Theorem: intervals are either nested or disjoint
│       └── Corollary: d[u] < d[v] < f[v] < f[u] ⟺ v is descendant of u
│
├── White-Path Theorem: descendants = vertices reachable via white paths at d[u]
│
├── Edge Classification (from color of v when (u,v) is explored):
│   ├── Tree   (v white)
│   ├── Back   (v gray)  ← CYCLE INDICATOR in directed graphs
│   ├── Forward (v black, d[u] < d[v])  ← directed graphs only
│   └── Cross  (v black, d[v] < d[u])  ← directed graphs only
│
└── Applications:
    ├── Cycle Detection: DFS reveals back edge ⟺ cycle exists
    └── Topological Sort: sort by decreasing f[v] (only valid if no back edges = DAG)
```

---

## Cheat Sheet

### Key Definitions

| Term | Definition |
|---|---|
| DFS Forest | Predecessor graph formed by π[v] pointers after DFS |
| d[v] | Discovery time (v turns gray) |
| f[v] | Finish time (v turns black) |
| Tree edge | (u,v) where v was white when explored |
| Back edge | (u,v) where v is an ancestor (gray) |
| Forward edge | (u,v) shortcut to a descendant (black, d[u] < d[v]) |
| Cross edge | (u,v) to unrelated vertex (black, d[v] < d[u]) |
| Topological Sort | Linear ordering where u precedes v for every edge (u,v) |

### Complexity

| Operation | Time |
|---|---|
| DFS (full) | O(n + m) |
| Topological Sort | O(n + m) |

### Key Theorems

**Parenthesis Theorem**: Intervals [d[u],f[u]] and [d[v],f[v]] are either completely nested or completely disjoint.

**Nested ↔ Descendant**: `d[u] < d[v] < f[v] < f[u]` iff v is a descendant of u.

**White-Path Theorem**: v is a descendant of u iff at time d[u], there exists an all-white path from u to v in G.

**Cycle detection**: G (directed) has a cycle iff DFS produces a back edge.

**Topo sort correctness**: In a DAG, for any edge (u,v), f[v] < f[u]. So decreasing finish time = valid topological order.

### Common Mistakes

1. **Thinking DFS-VISIT is called more than once per vertex** — it isn't. The white check prevents this.
2. **Assuming undirected graphs have forward/cross edges** — they don't. Only tree and back edges.
3. **Trying to topologically sort a graph with cycles** — undefined. Always check for back edges (cycles) first.
4. **Confusing d[v] in DFS vs d[v] in BFS** — in BFS, d[v] = shortest path distance. In DFS, d[v] = discovery *timestamp*. Completely different meanings.
5. **Forgetting that DFS(G) loops over ALL vertices** — not just from one source. This handles disconnected graphs.

---

## Practice Questions

### Theory

**Q1.** A DFS on a directed graph G produces the following d/f values:

```
Vertex | d[] | f[]
  a    |  1  |  8
  b    |  2  |  5
  c    |  3  |  4
  d    |  6  |  7
  e    |  9  | 10
```

(a) Which vertices are descendants of `a` in the DFS forest?  
(b) Is vertex `e` in the same DFS tree as `a`? Why?  
(c) If edge `(a, d)` exists, what type of edge is it?

**Solution**:

(a) A vertex `v` is a descendant of `a` (d[a]=1, f[a]=8) iff its interval is nested inside [1,8]. That's b (interval [2,5] ⊂ [1,8]), c ([3,4] ⊂ [1,8]), d ([6,7] ⊂ [1,8]). So descendants = {b, c, d}.

(b) No. `e`'s interval [9,10] is completely disjoint from a's [1,8]. DFS started a new tree at `e` after finishing `a`'s tree.

(c) Edge (a,d): `d` is black when `a` explores it (d[a]=1 < d[d]=6 < f[d]=7 < f[a]=8, so d is a descendant of a, and d[a] < d[d]). This makes it a **forward edge**.

---

**Q2.** Prove or disprove: "In an undirected graph, DFS from a single source visits all vertices."

**Solution**: **False**. DFS from a single source only visits all vertices reachable from that source. If the graph is disconnected, vertices in other components are not reached. This is why `DFS(G)` (the outer function) loops over all vertices and starts a new `DFS-VISIT` for any still-white vertex. BFS from a single source has the same limitation.

---

**Q3.** You run DFS on a directed graph and observe a back edge `(u, v)`. Write out explicitly what conditions on d[v], f[v], d[u] must hold.

**Solution**: A back edge `(u, v)` means `v` is gray when `u` is exploring it. Gray means `v` has been discovered but not finished — `v` is currently on the call stack. Since `v` is an ancestor of `u`:
- `d[v] < d[u]` (v was discovered first)
- `f[v]` has NOT been set yet at the time of the edge
- By the parenthesis theorem: `d[v] < d[u] < f[u] < f[v]` — u's interval is nested inside v's, confirming v is an ancestor of u.

---

**Q4.** Given a DAG with topological order. If edge `(u, v)` exists, what can you conclude about `f[u]` and `f[v]`?

**Solution**: In any DFS of a DAG, for every edge `(u, v)`, we have `f[v] < f[u]`. This is the foundation of topological sort — sorting by decreasing `f` values guarantees `u` appears before `v` for every edge. (Proof: v cannot be gray when we explore (u,v), since that would imply a cycle, contradicting DAG. If v is white, v is a descendant so f[v] < f[u]. If v is black, f[v] is already recorded and less than current time < f[u].)

---

**Q5.** Code Tracing. Trace DFS on this graph (adjacency list, edges explored in alphabetical order):

```
Vertices: A, B, C, D
Edges: A→B, A→C, B→D, C→D, D→A
```

Fill in d[], f[], edge types, and determine if the graph has a cycle.

**Solution**:

Start DFS(G): time=0, all white.

DFS-VISIT(A): color[A]=gray, time=1, d[A]=1
  Explore A→B: B is white → Tree edge
  DFS-VISIT(B): color[B]=gray, time=2, d[B]=2
    Explore B→D: D is white → Tree edge
    DFS-VISIT(D): color[D]=gray, time=3, d[D]=3
      Explore D→A: A is **gray** → **Back edge** ← CYCLE DETECTED
      No more neighbors
    color[D]=black, time=4, f[D]=4
  color[B]=black, time=5, f[B]=5
  Explore A→C: C is white → Tree edge
  DFS-VISIT(C): color[C]=gray, time=6, d[C]=6
    Explore C→D: D is **black**, d[C]=6 > d[D]=3 → **Cross edge**
  color[C]=black, time=7, f[C]=7
color[A]=black, time=8, f[A]=8

Final table:
```
Vertex | d[] | f[]
  A    |  1  |  8
  B    |  2  |  5
  C    |  6  |  7
  D    |  3  |  4
```

**Edge types**: A→B: Tree, A→C: Tree, B→D: Tree, D→A: **Back**, C→D: **Cross**

**Cycle?** YES — the back edge D→A reveals the cycle A→B→D→A.
