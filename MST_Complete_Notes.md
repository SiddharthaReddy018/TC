# Minimum Spanning Trees — Complete Notes
> From first principles, with examples, algorithms, proofs, and complexity analysis.

---

## Table of Contents
1. [The MST Problem](#1-the-mst-problem)
2. [Kruskal's Algorithm](#2-kruskals-algorithm)
3. [Prim's Algorithm](#3-prims-algorithm)
4. [Proof of Correctness — The Cut Property](#4-proof-of-correctness--the-cut-property)
5. [Big Picture Summary](#5-big-picture-summary)
6. [Cheat Sheet](#6-cheat-sheet)
7. [Practice Questions](#7-practice-questions)

---

## 1. The MST Problem

### Core Intuition

Imagine you are a telecom company. You need to connect 5 cities with cables so that every city can communicate with every other city. Each cable between two cities has a cost. You want to lay cables such that:

- Every city is reachable from every other city (connectivity).
- The total cable cost is as low as possible (minimality).

You don't need a direct cable between every pair of cities — as long as there is *some* path between them, connectivity holds. The structure that achieves this with the minimum total cost is the **Minimum Spanning Tree**.

---

### Definitions

**Graph G(V, E):**
- V = set of vertices (cities, nodes)
- E = set of edges (connections between pairs of vertices)
- c : E → ℕ — a cost/weight function assigning a positive integer to every edge

**Spanning Tree:**
A subset T ⊆ E such that:
1. G(V, T) is **connected** — every vertex can reach every other vertex.
2. G(V, T) has **no cycles** — there is exactly one path between any two vertices.
3. It uses exactly **|V| - 1 edges**.

**Minimum Spanning Tree (MST):**
A spanning tree T where the total cost `Σ c(e)` for all edges e in T is minimised.

---

### Concrete Example

Consider this graph with 4 vertices A, B, C, D:

```
        A
       /|\
    100/ | \
      /  |  \
     B   |   C
      \  |  /
    50 \ | / (other edges with various costs)
        \|/
         D
```

The slide (page 2) shows a dense graph with nodes A, B, C, D and various edge weights including 100 (A-B) and 50 (A-D). The MST will pick the cheapest edges that still connect everything — it will NOT pick the 100-cost edge if cheaper alternatives exist.

A spanning tree on 4 nodes needs exactly **3 edges** (|V| - 1 = 4 - 1 = 3). We want those 3 edges to have the minimum total weight.

---

### Why Greedy Works Here

For many optimisation problems, greedy (always pick the locally best option) fails globally. But for MST, it works. Intuitively: adding the cheapest safe edge never hurts, because the cheapest edge crossing any partition of the graph *must* be in the MST. This is proven formally by the **Cut Property** (Section 4).

Two classic greedy algorithms solve MST:
- **Kruskal's**: grow the tree by picking the globally cheapest edge that doesn't form a cycle.
- **Prim's**: grow the tree outward from one node, always extending by the cheapest available edge.

---

## 2. Kruskal's Algorithm

### Core Intuition

Think of each vertex as its own isolated island. You have a sorted list of bridges (edges) by cost, cheapest first. You keep building bridges one by one, but with one rule: **never build a bridge that connects two places already connected** (that would form a redundant cycle). Stop when all islands are one connected landmass.

---

### Pseudocode

```
A = ∅                                  // A will hold our MST edges

Add each vertex v to a separate component   // n isolated components

Sort all edges E by weight (ascending)

For each edge (u, v) in sorted order:
    if u and v are NOT in the same component:
        A = A ∪ {(u, v)}               // safe to add — no cycle formed
        Merge the components of u and v
```

**Key idea:** The "same component" check is asking: "are u and v already connected?" If yes, adding this edge would create a cycle — skip it. If no, it's a safe bridge — add it.

---

### Worked Example

Look at the graph on slide 7 (left side). Nodes: a, b, c, d, e, f, g. Edges with weights:
```
a-c: 1,  c-f: 2,  e-g: 3,  c-d: 4,  d-e: 5,  c-g: 6,  f-g: 7,  b-f: 8,  b-c: 9,  a-b: 10,  f-?:11,  c-e: 12
```

**Step-by-step Kruskal's:**

| Step | Edge  | Weight | Same Component? | Action  |
|------|-------|--------|-----------------|---------|
| 1    | a-c   | 1      | No              | ADD ✓   |
| 2    | c-f   | 2      | No              | ADD ✓   |
| 3    | e-g   | 3      | No              | ADD ✓   |
| 4    | c-d   | 4      | No              | ADD ✓   |
| 5    | d-e   | 5      | No              | ADD ✓   |
| 6    | c-g   | 6      | Yes (a-c-f-...) | SKIP ✗  |
| 7    | f-g   | 7      | Yes             | SKIP ✗  |
| 8    | b-f   | 8      | No              | ADD ✓   |
| 9    | b-c   | 9      | Yes             | SKIP ✗  |

We now have 6 edges for 7 nodes — that's |V|-1 = 6. Done!

**MST edges (highlighted in red on the slide):** a-c, c-f, e-g, c-d, d-e, b-f  
**Total cost:** 1+2+3+4+5+8 = **23**

The right side of slide 7 shows this MST drawn cleanly — notice it's a tree (no cycles, all connected).

---

### The Union-Find Data Structure

To efficiently check "are u and v in the same component?" and merge components, we use **Union-Find** (also called Disjoint Set Union).

- `Find(u)` → returns the representative/root of u's component. O(log n).
- `Union(u, v)` → merges the components of u and v. O(log n).

The "same component" check becomes: `Find(u) == Find(v)`.

---

### Time Complexity of Kruskal's

| Step | Cost |
|------|------|
| Initialise n components | O(n) |
| Sort m edges | O(m log m) |
| For each edge: Find + possibly Union (m times, O(log n) each) | O(m log n) |

**Total: O(n + m log m + m log n)**

Since m can be at most n², we have log m ≤ 2 log n, so this simplifies to:

**O(m log n)**

> Note: The slide writes O(n + m log m + m log n). This is correct and simplifies to O(m log n) since log m = O(log n) for simple graphs.

---

## 3. Prim's Algorithm

### Core Intuition

Instead of picking the globally cheapest edge (like Kruskal's), Prim's **grows a single tree outward** from a starting node. At every step, ask: "What's the cheapest edge that connects a node *already in my tree* to a node *not yet in my tree*?" Add that edge and absorb the new node.

Analogy: You're expanding a city. You start from one district. At each step, you build the cheapest road that connects the city to a new suburb. Never jump to a far-away suburb if a nearby cheap one is available.

---

### Key Variables

- `key[v]` — the cheapest edge cost by which v can be attached to the current tree. Initially ∞ for all nodes.
- `π[v]` — the parent of v in the MST (the node it connects to). Initially NIL.
- `Q` — a priority queue (min-heap) of all vertices not yet added to the tree, ordered by `key`.
- `r` — the starting root node; its key is set to 0 so it's extracted first.

---

### Pseudocode

```
For each u in V:
    key[u] = ∞
    π[u] = NIL

key[r] = 0          // Start from root r
Q = V               // All vertices in the queue

While Q is not empty:
    u = ExtractMin(Q)       // Pick the vertex with smallest key
    Add edge (u, π[u]) to MST

    For each neighbour v of u:
        if v ∈ Q  AND  w(u, v) < key[v]:
            π[v] = u
            key[v] = w(u, v)   // Update: cheaper way to reach v found!
```

**Key insight:** When we extract u from Q, we're saying "u is now joined to the tree via the edge (u, π[u]) with cost key[u]." We then look at all of u's neighbours and update their keys if we found a cheaper connection.

---

### Worked Example

Look at slide 14. Graph: nodes a, b, c, d, e. Edges: a-b:1, a-c:2, b-c:3, b-d:1(top), d-e:5, c-e:2, b-e:8.

Start from node **c** (key[c] = 0).

**Initial state:**
```
key:  a=∞, b=∞, c=0, d=∞, e=∞
π:    all NIL
Q:    {a, b, c, d, e}
```

**Round 1:** Extract c (key=0). Neighbours of c: a (cost 2), b (cost 3), e (cost 2).
```
key[a] = 2,  π[a] = c
key[b] = 3,  π[b] = c
key[e] = 2,  π[e] = c
```

**Round 2:** Extract a (key=2, tie with e — say a). Neighbours of a: b (cost 1 < 3 → UPDATE!).
```
key[b] = 1,  π[b] = a    ← improved!
```

**Round 3:** Extract b (key=1). Neighbours of b: d (cost 1).
```
key[d] = 1,  π[d] = b
```

**Round 4:** Extract e (key=2). Neighbours: d (cost via e? not cheaper). No updates.

**Round 5:** Extract d (key=1).

**MST edges (u, π[u]):** (a,c), (b,a), (e,c), (d,b)  
**Total cost:** 2 + 1 + 2 + 1 = **6**

This matches the MST shown on slide 14 highlighted in red. The table on the right of that slide tracks exactly these key[] and π[] values updating over time.

---

### Time Complexity of Prim's

The two expensive operations are:

1. `ExtractMin(Q)` — called **n times** (once per vertex)
2. `key[v]` update (decrease-key) — called at most **m times** (once per edge examined)

#### With a Binary Heap:

| Operation | Cost per call | Total calls | Total cost |
|-----------|---------------|-------------|------------|
| ExtractMin | O(log n) | n | O(n log n) |
| Decrease-Key | O(log n) | m | O(m log n) |

**Total: O((m + n) log n)**

For connected graphs, m ≥ n-1, so this is simply **O(m log n)**.

#### With a Fibonacci Heap:

Fibonacci heap supports decrease-key in **amortised O(1)**.

| Operation | Cost per call | Total calls | Total cost |
|-----------|---------------|-------------|------------|
| ExtractMin | O(log n) amortised | n | O(n log n) |
| Decrease-Key | O(1) amortised | m | O(m) |

**Total: O(m + n log n)**

This is asymptotically better when m is large (dense graphs). In practice, the constant factors of Fibonacci heaps make binary heaps faster for most real inputs.

> **Important note:** Slides 16-18 have "Kruskal's" crossed out and replaced with "Prim's" in handwriting. The complexity analysis on those slides is for **Prim's algorithm**, not Kruskal's. This is a labelling error in the slides — don't be confused.

---

### Kruskal's vs Prim's — Head to Head

| | Kruskal's | Prim's |
|---|---|---|
| **Strategy** | Global: sort all edges, pick cheapest non-cycle edge | Local: grow one tree outward greedily |
| **Data structure** | Union-Find | Priority Queue (heap) |
| **Better for** | Sparse graphs (few edges) | Dense graphs (many edges with Fibonacci heap) |
| **Complexity** | O(m log n) | O(m log n) binary, O(m + n log n) Fibonacci |
| **Both give** | A correct MST | A correct MST |

---

## 4. Proof of Correctness — The Cut Property

### Why Do We Need a Proof?

Greedy doesn't always work. For example, greedy fails for shortest paths in some cases, and for many combinatorial optimisation problems. So why does it work for MST? The answer is the **Cut Property** — a deep theorem that tells us exactly which edges are "safe" to add.

---

### What is a Cut?

A **cut** is a partition of the vertex set V into two non-empty groups: S and V\S (V minus S).

An edge **crosses the cut** if one endpoint is in S and the other is in V\S.

Look at the diagram on slide 21 — S is the left blob, V\S is the right blob. Several edges cross between them. Edge `e` (shown bold/blue) is the minimum weight edge among all crossing edges.

```
         S             V \ S
    ___________      ___________
   |     u     |    |     v     |
   |      \----|----|-e          |
   |            |    |           |
   |   (other   |    |  (other   |
   |   nodes)   |    |   nodes)  |
   |___________|    |___________|

  e = minimum cost edge crossing this cut
```

---

### The Cut Property (Theorem)

**Statement:**  
Assume all edge costs are **distinct** (no ties).  
Let S ⊂ V, S ≠ ∅.  
Let e = (u, v) be the **minimum cost edge** with u ∈ S and v ∈ V\S (i.e., e crosses the cut).  
Then **every MST contains the edge e**.

In plain English: the cheapest edge crossing any cut is *guaranteed* to be in the MST.

---

### Proof (Exchange Argument)

This is a proof by contradiction. Look at slide 23 for the visual.

**Setup:** Suppose for contradiction that some MST T does *not* contain e = (u, v), even though e is the minimum cost edge crossing the cut (S, V\S).

**Step 1:** Since T is a spanning tree, there exists a path P from u to v in T (T connects everything).

**Step 2:** This path P must cross the cut (S, V\S) at least once — it starts in S (at u) and ends in V\S (at v). Call that crossing edge e₁ = (u', v').

**Step 3:** Since e is the *minimum cost* edge crossing the cut, and e₁ also crosses the cut:
```
w(e) < w(e₁)       [e is strictly cheaper, since all costs are distinct]
```

**Step 4:** Construct a new tree T':
```
T' = (T \ {e₁}) ∪ {e}
```
In words: remove e₁ from T, and add e instead.

**Step 5:** Verify T' is still a spanning tree:
- Removing e₁ splits T into two components (one containing S-side, one containing V\S-side).
- Adding e reconnects them (e also crosses the cut).
- So T' is still connected with |V|-1 edges → still a spanning tree. ✓

**Step 6:** Compare costs:
```
w(T') = w(T) - w(e₁) + w(e)  <  w(T)     [because w(e) < w(e₁)]
```

**Contradiction:** T' is a spanning tree with *strictly smaller* total cost than T. But T was assumed to be an MST. This is impossible.

**Conclusion:** Our assumption was wrong. T must contain e. ∎

---

### How the Cut Property Justifies Both Algorithms

**Kruskal's correctness:**  
When Kruskal's adds edge (u, v), let S = the connected component of u in the current partial solution. Then (u, v) is the minimum weight edge crossing the cut (S, V\S) — because Kruskal's processes edges in sorted order and skips edges within the same component. By the Cut Property, (u, v) must be in the MST. So every edge Kruskal's adds is safe.

**Prim's correctness:**  
When Prim's extracts node v and adds it to the tree, let S = the set of nodes already in the tree. The edge (π[v], v) with cost key[v] is the minimum cost edge crossing the cut (S, V\S) — Prim's explicitly maintains this via the key[] values. By the Cut Property, this edge is in the MST.

Look at slide 24 — the left diagram shows Kruskal's: S is the component containing u, the edge (u,v) is the black bold edge being added, and there's a green cheaper edge within S already (which is why those nodes are already merged). The right diagram shows Prim's: S is the growing tree, the green edge is the one being considered, the red edge is rejected.

---

### Important Assumption: Distinct Edge Weights

The proof above requires all edge weights to be **distinct** (stated explicitly on slide 20). If weights can be equal (ties), the Cut Property still holds but the proof requires a small modification — the MST may not be unique, but both algorithms still produce *a* valid MST. In practice, ties are handled by breaking them arbitrarily, and correctness is maintained.

---

## 5. Big Picture Summary

```
PROBLEM: Connect all nodes with minimum total edge weight.

                    GREEDY WORKS because of:
                        CUT PROPERTY
                    (min edge across any cut
                      must be in every MST)
                           /        \
                          /          \
              KRUSKAL'S              PRIM'S
          (global, edge-centric)  (local, node-centric)
          Sort edges, add          Grow tree from root,
          cheapest non-cycle       always absorb cheapest
          edge globally            reachable new node
                |                        |
           Union-Find              Priority Queue
           O(m log n)              O(m log n) binary heap
                                   O(m + n log n) Fib heap
```

Both algorithms use the same underlying correctness guarantee (Cut Property) but implement it differently. Kruskal's thinks in terms of "which edge is globally cheapest and safe?" while Prim's thinks in terms of "which node can I absorb most cheaply from where I am?"

---

## 6. Cheat Sheet

### Key Definitions
| Term | Definition |
|------|-----------|
| Spanning Tree | Subset of edges connecting all vertices with no cycles; has exactly n-1 edges |
| MST | Spanning tree with minimum total edge weight |
| Cut | Partition of V into S and V\S |
| Safe edge | An edge that can be added to the current partial MST without violating optimality |
| key[v] (Prim's) | Cheapest edge cost connecting v to the current tree |
| π[v] | Parent of v in the MST being built |

### Algorithm Summaries

**Kruskal's:**
1. Sort edges by weight.
2. For each edge (cheapest first): if endpoints are in different components, add it.
3. Use Union-Find for component tracking.

**Prim's:**
1. Set key[r]=0, all others ∞. Put all nodes in min-heap Q.
2. Extract min node u. Add edge (u, π[u]) to MST.
3. For each neighbour v: if v ∈ Q and w(u,v) < key[v], update key[v] = w(u,v), π[v] = u.
4. Repeat until Q empty.

### Complexities
| Algorithm | Data Structure | Time Complexity |
|-----------|----------------|-----------------|
| Kruskal's | Union-Find | O(m log n) |
| Prim's | Binary Heap | O((m+n) log n) = O(m log n) |
| Prim's | Fibonacci Heap | O(m + n log n) |

### Common Mistakes
1. **Kruskal's**: forgetting to check for cycles — just sorting and adding all edges gives a graph, not a tree.
2. **Prim's**: confusing `key[v]` with the distance from the root (it's the cheapest *direct edge* to the current tree, not a path length).
3. **Complexity**: the slides mislabel Prim's complexity slides as "Kruskal's" — the O((m+n) log n) and O(m+n log n) complexities are for **Prim's**, not Kruskal's.
4. **Cut Property assumption**: forgetting it requires distinct weights for the strongest form.
5. **MST uniqueness**: with distinct weights, the MST is unique. With ties, there can be multiple valid MSTs.

---

## 7. Practice Questions

### Theory Questions

**Q1.** A graph has 8 vertices. How many edges does any spanning tree of this graph have? Why?

> **Answer:** 7 edges. A spanning tree on n nodes always has exactly n-1 edges. With fewer edges, the graph can't be connected. With more, it must have a cycle.

---

**Q2.** Can Kruskal's algorithm add an edge that creates a cycle? Why or why not?

> **Answer:** No — by design. The algorithm explicitly checks whether the two endpoints are in the same connected component before adding an edge. If they are, the edge is skipped. Two nodes being in the same component means there's already a path between them; adding another edge between them would create a cycle.

---

**Q3.** Does Prim's algorithm depend on the starting node chosen? Will different starting nodes give different MSTs?

> **Answer:** With distinct edge weights, no — the MST is unique regardless of starting node. You'll always get the same set of edges, possibly discovered in a different order. With ties, the MST may differ depending on how ties are broken.

---

**Q4.** Why does the Cut Property require distinct edge weights in its standard formulation?

> **Answer:** The proof uses the fact that w(e) < w(e₁) strictly. If two edges across the cut have equal weight, we can't conclude w(e) < w(e₁). In that case, both could be in different valid MSTs. The property still holds in a weaker form (the min-weight edge is *contained in some* MST), but uniqueness is lost.

---

### Tracing Questions

**Q5.** Trace Kruskal's algorithm on this graph:

```
Vertices: {1, 2, 3, 4}
Edges:
  (1,2): 3
  (1,3): 1
  (2,3): 2
  (2,4): 5
  (3,4): 4
```

> **Answer:**
> 
> Sorted edges: (1,3):1, (2,3):2, (1,2):3, (3,4):4, (2,4):5
> 
> | Edge  | Weight | Components before | Action |
> |-------|--------|-------------------|--------|
> | (1,3) | 1 | {1},{2},{3},{4} → 1≠3 | ADD ✓ → {1,3},{2},{4} |
> | (2,3) | 2 | {1,3},{2},{4} → 2≠3 | ADD ✓ → {1,2,3},{4} |
> | (1,2) | 3 | {1,2,3},{4} → 1=2 (same!) | SKIP ✗ |
> | (3,4) | 4 | {1,2,3},{4} → 3≠4 | ADD ✓ → {1,2,3,4} |
> 
> MST edges: (1,3), (2,3), (3,4)  
> Total cost: 1 + 2 + 4 = **7**

---

**Q6.** Trace Prim's algorithm on the same graph, starting from node 1.

> **Answer:**
> 
> Initial: key = [0, ∞, ∞, ∞], π = [NIL, NIL, NIL, NIL], Q = {1,2,3,4}
> 
> **Extract 1** (key=0). Neighbours: 2 (cost 3), 3 (cost 1).
> - key[2] = 3, π[2] = 1
> - key[3] = 1, π[3] = 1
> 
> **Extract 3** (key=1). Neighbours: 1 (in tree), 2 (cost 2 < 3 → UPDATE), 4 (cost 4).
> - key[2] = 2, π[2] = 3
> - key[4] = 4, π[4] = 3
> 
> **Extract 2** (key=2). Neighbours: 1 (in tree), 3 (in tree), 4 (cost 5 > 4 → no update).
> 
> **Extract 4** (key=4). Q empty.
> 
> MST edges (u, π[u]): (3,1), (2,3), (4,3)  
> Total cost: 1 + 2 + 4 = **7** ✓ Same result as Kruskal's.

---

**Q7.** Apply the Cut Property: In the graph from Q5, define S = {1, 3}. What is the minimum cost edge crossing this cut? Is it in the MST you found?

> **Answer:**  
> Edges crossing cut (S={1,3}, V\S={2,4}):
> - (1,2): cost 3 (1 ∈ S, 2 ∈ V\S)
> - (2,3): cost 2 (3 ∈ S, 2 ∈ V\S)
> - (3,4): cost 4 (3 ∈ S, 4 ∈ V\S)
> 
> Minimum is **(2,3) with cost 2**.  
> Yes — edge (2,3) is in the MST found above. The Cut Property correctly predicted this.

---

**Q8 (Hard).** Suppose you have a graph where the maximum edge weight is W. Describe how Kruskal's total time complexity changes if you use counting sort instead of comparison sort on the edges.

> **Answer:**  
> Counting sort runs in O(m + W) instead of O(m log m). The rest of Kruskal's (Union-Find operations) is O(m log n). 
> 
> Total becomes **O(m + W + m log n)**.  
> 
> If W is small (say W = O(n)), this is O(m log n) — same asymptotic.  
> If W is very large, counting sort is worse than comparison sort.  
> This is useful only when edge weights are small integers (which the problem statement allows: c: E → ℕ).
