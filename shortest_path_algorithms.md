# Shortest Path Algorithms — Complete Teaching Guide

---

## Section 1: Shortest Path Foundations

### What is the Shortest Path Problem?

Imagine you're navigating a city. You're at point **s** and want to reach point **t**. There are multiple routes — some short, some long. The **shortest path problem** asks: what is the minimum-cost route?

More formally, given a graph G = (V, E) with edge weights, find a path from source **s** to destination **t** that minimizes the total weight of edges used.

### Unweighted Graphs

When the graph has no weights (or all weights are equal to 1), "shortest" means **fewest edges**. The right tool here is **BFS (Breadth-First Search)** — it explores nodes level by level, guaranteeing the first time it reaches a node, it has done so via the fewest edges.

### Weighted Graphs

When edges have different weights, BFS no longer works — fewer edges doesn't mean smaller total cost. We need smarter algorithms. This document covers three:

| Algorithm | Handles Negative Edges? | Single or All-Pairs? | Complexity |
|---|---|---|---|
| Bellman-Ford | Yes (no negative cycles) | Single-source | O(nm) |
| Dijkstra | No | Single-source | O((n+m) log n) |
| Floyd-Warshall | Yes (no negative cycles) | All-pairs | O(n³) |

---

### Negative Edges and Negative Cycles

A **negative edge** is simply an edge with weight < 0. These are valid in many real-world scenarios (e.g., profit from a trade route).

A **negative cycle** is a cycle whose total weight is negative. This is a problem because you could loop around it forever, decreasing your path cost to −∞. In such a graph, the shortest path is undefined.

```
Example of a negative cycle:
A --(-1)--> B --(-1)--> C --(-1)--> A
Total cycle weight = -3. You can keep looping forever.
```

**Key rule:** As long as there are no negative cycles, shortest paths are well-defined and simple (no repeated vertices).

---

### Optimal Substructure

This is the key property that makes dynamic programming applicable to shortest paths.

**Claim:** Any subpath of a shortest path is itself a shortest path.

**Why?** Suppose the shortest path from s to t passes through an intermediate node w:

```
s ~~~> w ~~~> t
```

The subpath from s to w must be the shortest path from s to w. If there were a shorter path from s to w, we could substitute it in and get a shorter overall path from s to t — contradicting our assumption.

This property is what allows us to break big path problems into smaller ones and solve them recursively.

---

## Section 2: Bellman-Ford Algorithm

### Problem Setup

- Given: Directed weighted graph G, source **s**, destination **t**
- G may have **negative edges** but **no negative cycles**
- Goal: Find the shortest path from s to t

### Core Insight

Since there are no negative cycles, the shortest path is **simple** — it never visits the same vertex twice. A simple path in a graph with n vertices uses at most **n−1 edges**. This bounds the search space.

---

### Dynamic Programming Formulation

We define:

> **OPT(i, v)** = minimum cost of a path from v to t using **at most i edges**

The parameters range over:
- i from 0 to n−1
- v over all vertices V

**The answer we want is OPT(n−1, s)** — the shortest path from s to t using at most n−1 edges.

---

### The Recurrence

Let P be the optimal (shortest) path from v to t using at most i edges. There are two cases:

**Case 1: P uses at most i−1 edges**

The extra budget of i edges wasn't needed. So:

```
OPT(i, v) = OPT(i-1, v)
```

**Case 2: P uses exactly i edges**

Then P starts with some first edge (v → w), followed by a path from w to t using i−1 edges. We don't know which neighbor w is optimal, so we try all of them:

```
OPT(i, v) = min over w ∈ N(v) of [ c(v, w) + OPT(i-1, w) ]
```

where c(v, w) is the cost of edge (v, w) and N(v) is the set of neighbors of v.

**Combining both cases:**

```
OPT(i, v) = min( OPT(i-1, v),  min_{w ∈ N(v)} [ c(v,w) + OPT(i-1, w) ] )
```

This is the full Bellman-Ford recurrence.

---

### Base Cases

```
OPT(0, t) = 0         (already at destination, zero cost)
OPT(0, v) = ∞         (for v ≠ t, can't reach t with 0 edges)
```

---

### Worked Example

Consider this graph (going toward t):

```
s --2--> a --3--> t
s --(-1)--> b --5--> t
```

OPT(0, t) = 0, OPT(0, everything else) = ∞

After i=1:
- OPT(1, a) = c(a,t) + OPT(0,t) = 3
- OPT(1, b) = c(b,t) + OPT(0,t) = 5

After i=2:
- OPT(2, s) = min(
    OPT(1, s),
    c(s,a) + OPT(1,a),    → 2 + 3 = 5
    c(s,b) + OPT(1,b)     → -1 + 5 = 4
  ) = 4

So the shortest path is s → b → t with cost 4.

---

### Correctness

Proved by induction on i, using the **optimal substructure property**: if we correctly know all OPT(i−1, ·) values, the recurrence correctly computes all OPT(i, ·) values.

---

### Running Time

The DP table has **n × n** entries (n values of i, n vertices). For each entry OPT(i, v), we check all neighbors of v — that's deg(v) work. Across all vertices, the total work per round i is:

```
Sum over all v of deg(v) = 2m   (for undirected) or m (directed)
```

There are n rounds. So total time:

```
O(n * m) = O(nm)
```

> **Note:** The slides initially show O(n³), which is a loose upper bound (since m ≤ n²). The tighter and standard correct complexity is **O(nm)**. The slides correct this on the next slide.

---

## Section 3: Dijkstra's Algorithm

### Problem Setup

- Given: Graph G (non-negative edge weights), source s
- Goal: Find shortest paths from s to **all** vertices

Dijkstra solves a harder problem than Bellman-Ford (all destinations instead of one) but requires non-negative edges.

---

### Core Idea: The Greedy Approach

Think of Dijkstra like an expanding ripple in a pond. You start at s. At each step, you commit to the next closest unvisited node — and once committed, you never revise that decision.

This works because with non-negative edges, once you've found the cheapest way to reach a node, no future path can be cheaper (you can't "sneak in" a shortcut with a negative edge later).

---

### The Algorithm

Dijkstra maintains:
- A set **S** of "explored" vertices whose shortest distances are finalized
- A distance label **d(u)** for each u ∈ S (the true shortest distance from s)
- A tentative distance **d'(v)** for each v ∉ S

**Initialization:**
```
S = {s},  d(s) = 0
d'(v) = ∞ for all v ≠ s
```

**Main Loop (while S ≠ V):**

1. For each v ∉ S, compute:
   ```
   d'(v) = min over edges (u,v) with u ∈ S  of  [ d(u) + l(u,v) ]
   ```
   This is the shortest path reachable by going through S and then taking one more edge.

2. Select the node v ∉ S with the **minimum** d'(v)

3. Add v to S, set d(v) = d'(v)

---

### Worked Example

Look at the graph on slide 21. We have vertices s, a, b, c, d with edges:
- s→a: 2, s→b: 1, s→d: 8
- a→c: 1
- b→d: 4
- c→d: 1

**Step-by-step:**

```
Initially:  S={s}, d(s)=0
            d'(a)=2, d'(b)=1, d'(c)=∞, d'(d)=8

Round 1:    Pick b (d'=1). S={s,b}, d(b)=1
            Update: d'(d) = min(8, d(b)+4) = min(8,5) = 5

Round 2:    Pick a (d'=2). S={s,b,a}, d(a)=2
            Update: d'(c) = min(∞, d(a)+1) = 3

Round 3:    Pick c (d'=3). S={s,b,a,c}, d(c)=3
            Update: d'(d) = min(5, d(c)+1) = 4

Round 4:    Pick d (d'=4). S={s,b,a,c,d}, d(d)=4
```

Final distances: d(s)=0, d(b)=1, d(a)=2, d(c)=3, d(d)=4.

---

### Proof of Correctness

**Claim:** When a vertex v is added to S, d(v) is the true shortest distance from s to v.

**Proof by induction on |S|:**

**Base case:** S = {s}, d(s) = 0. Trivially correct.

**Inductive step:** Assume all vertices already in S have correct distances. We pick v ∉ S with minimum d'(v). Suppose for contradiction that d(v) is wrong — there exists a shorter path P' from s to v with weight < d'(v).

P' must leave S at some point. Let (x, y) be the first edge of P' where x ∈ S and y ∉ S. Then:

```
weight(P') ≥ weight(path to x) + weight(x,y)
           ≥ d(x) + w(x,y)         [by inductive hypothesis]
           ≥ d'(y)                  [definition of d']
           ≥ d'(v)                  [v was chosen as minimum]
           = weight(P)
```

(The last inequality uses non-negativity of remaining edges from y to v.) Contradiction — P' cannot be shorter than d'(v). ∎

> **Critical point:** This proof breaks down with negative edges. A negative edge after y could make P' cheaper than d'(v), invalidating the last step. That's why Dijkstra requires non-negative edges.

---

### Running Time

Each iteration of the main loop does two things:
- **ExtractMin:** Find the unvisited vertex with smallest d'. Done **n times** (once per vertex).
- **Update (Relax):** Update d' values for neighbors of the newly added vertex. Done **m times total** across all iterations (each edge is processed once).

| Data Structure | ExtractMin | Update | Total |
|---|---|---|---|
| Array (naive) | O(n) | O(1) | O(n²) |
| Binary Heap | O(log n) | O(log n) | O((n+m) log n) |
| Fibonacci Heap | O(log n) amortized | O(1) amortized | O(n log n + m) |

For **sparse graphs** (m ≈ n), Fibonacci heaps give O(n log n). For **dense graphs** (m ≈ n²), even O(n²) from a simple array can be competitive.

---

### What Happens With Negative Edges?

Dijkstra can give wrong answers. Example:

```
s --2--> a
s --3--> b --(-2)--> a
```

True shortest path to a: s → b → a, cost = 3 + (−2) = 1.

Dijkstra picks a first (d'(a)=2) and finalizes it. It never revisits a. Wrong answer.

Use **Bellman-Ford** when negative edges are present.

---

## Section 4: Floyd-Warshall Algorithm

### Problem Setup

- Given: Directed weighted graph G (can have negative edges, no negative cycles)
- Goal: Find shortest paths between **every pair** of vertices

This is the **All-Pairs Shortest Path (APSP)** problem. You could run Bellman-Ford from every vertex (n times) in O(n²m), or run Dijkstra n times in O(n(n+m) log n). Floyd-Warshall gives a clean O(n³) solution.

---

### Core Idea

Label the vertices 1, 2, ..., n. The key insight: **gradually allow more and more intermediate vertices** on the path.

Define:

> **D[i, j, k]** = weight of the shortest path from i to j, where all **intermediate** vertices (not the endpoints) come from the set {1, 2, ..., k}

- D[i, j, 0] = direct edge weight w(i, j) (no intermediate vertices allowed)
- D[i, j, n] = true shortest path from i to j (all vertices allowed as intermediates)

The final answer table is {D[i, j, n]} for all pairs i, j.

---

### The Recurrence

When we increase k to k, we ask: **does allowing vertex k as an intermediate improve the path from i to j?**

Look at the diagram on slide 34 — the path either passes through k (splitting into i→k and k→j) or it doesn't.

**Case 1: k is not on the shortest i→j path**

The best path still only uses {1, ..., k−1}:
```
D[i, j, k] = D[i, j, k-1]
```

**Case 2: k is on the shortest i→j path**

The path goes i → ... → k → ... → j. Both sub-paths can only use {1, ..., k−1} as intermediates (since k appears only once — no cycles in a shortest simple path):

```
D[i, j, k] = D[i, k, k-1] + D[k, j, k-1]
```

**Combining:**

```
D[i, j, 0] = w(i, j)          (∞ if no direct edge)

D[i, j, k] = min( D[i, j, k-1],  D[i, k, k-1] + D[k, j, k-1] )
```

---

### Why This Works — Intuition

Think of it as progressively building a "highway network." At step k=0, you can only use direct roads. At k=1, you can route through city 1. At k=2, you can also route through city 2. By k=n, every city is available as a waypoint, so you have all optimal paths.

---

### Implementation Sketch

```
// Initialize D[i][j] = w(i,j) for direct edges, ∞ otherwise, 0 on diagonal
for k from 1 to n:
    for i from 1 to n:
        for j from 1 to n:
            D[i][j] = min(D[i][j],  D[i][k] + D[k][j])
```

Note: In practice, we can drop the k dimension and update in-place. The correctness still holds.

---

### Running Time

Three nested loops, each running n times:

```
O(n³)
```

Space: O(n²) for the distance matrix.

---

### Detecting Negative Cycles

After running Floyd-Warshall, check the diagonal: if **D[i, i, n] < 0** for any vertex i, then vertex i is on a negative cycle.

---

## Big Picture: How Everything Connects

```
Shortest Path Problems
│
├── Single-source, unweighted graph
│   └── BFS → O(n + m)
│
├── Single-source, non-negative weights
│   └── Dijkstra → O((n+m) log n) with binary heap
│
├── Single-source, negative edges (no negative cycles)
│   └── Bellman-Ford → O(nm)
│
└── All-pairs shortest path
    ├── Run Bellman-Ford n times → O(n²m)
    ├── Run Dijkstra n times (non-negative) → O(n(n+m) log n)
    └── Floyd-Warshall (DP) → O(n³)
```

All three algorithms in this document rely on **optimal substructure**: sub-paths of shortest paths are themselves shortest paths. Bellman-Ford and Floyd-Warshall exploit this through DP; Dijkstra exploits it greedily.

---

## Cheat Sheet

### Definitions

| Term | Meaning |
|---|---|
| Simple path | A path with no repeated vertices |
| Negative cycle | A cycle with total weight < 0 (makes shortest path undefined) |
| Optimal substructure | Sub-paths of shortest paths are shortest paths |
| OPT(i, v) | Min cost path from v to t using at most i edges (Bellman-Ford) |
| D[i, j, k] | Min cost path from i to j using only {1..k} as intermediates (Floyd-Warshall) |
| d(v) | Finalized shortest distance from s to v (Dijkstra) |
| d'(v) | Tentative shortest distance for unexplored vertex v (Dijkstra) |

### Algorithm Comparison

| Algorithm | Negative Edges? | Negative Cycles? | Scope | Time |
|---|---|---|---|---|
| Bellman-Ford | ✅ Yes | ❌ No | Single-source | O(nm) |
| Dijkstra | ❌ No | N/A | Single-source | O((n+m) log n) |
| Floyd-Warshall | ✅ Yes | ❌ No | All-pairs | O(n³) |

### Key Recurrences

**Bellman-Ford:**
```
OPT(0, t) = 0;  OPT(0, v≠t) = ∞
OPT(i, v) = min( OPT(i-1, v),  min_{w∈N(v)} [c(v,w) + OPT(i-1, w)] )
Answer: OPT(n-1, s)
```

**Dijkstra:**
```
d'(v) = min_{(u,v): u∈S} [d(u) + l(u,v)]
Pick v = argmin d'(v),  set d(v) = d'(v),  add to S
```

**Floyd-Warshall:**
```
D[i,j,0] = w(i,j)
D[i,j,k] = min( D[i,j,k-1],  D[i,k,k-1] + D[k,j,k-1] )
Answer: D[i,j,n] for all pairs
```

### Common Mistakes

1. **Using Dijkstra with negative edges** — it will give wrong answers silently.
2. **Forgetting the base case** in Bellman-Ford: OPT(0, t) = 0, everything else = ∞.
3. **Confusing OPT(i, v)** as path from s to v — it's actually from **v to t** in the standard formulation used here.
4. **Floyd-Warshall on graphs with negative cycles** — the diagonal will go negative; detect this before trusting results.
5. **Thinking O(n³) for Bellman-Ford** — it's O(nm), which is better when the graph is sparse (m << n²).
6. **Misunderstanding D[i,j,k]** — k is the maximum index of allowed *intermediate* vertices, not the number of edges.

---

## Practice Questions

### Theory

**Q1.** Why must the shortest path be simple (no repeated vertices) when there are no negative cycles?

**Answer:** If a path repeated a vertex, it would contain a cycle. Without negative cycles, every cycle has non-negative weight. Removing the cycle gives a path of equal or lower cost — so the shortest path never needs to repeat vertices.

---

**Q2.** Why does Dijkstra fail with negative edges? Give a concrete counterexample.

**Answer:** Dijkstra's correctness relies on the fact that once a vertex is finalized, no future path can be cheaper. With negative edges, a path going through an unvisited vertex with a negative edge could later "undercut" a finalized distance. Counterexample:

```
s --1--> a --(-3)--> t
s --5--> t
```
Dijkstra finalizes d(t)=5 from the direct edge before exploring via a. True shortest: s→a→t = 1 + (−3) = −2.

---

**Q3.** In Floyd-Warshall, why is D[i, k, k-1] used instead of D[i, k, k] in the recurrence?

**Answer:** Because we need the shortest path from i to k using only intermediate vertices from {1..k−1}. If we used D[i, k, k], we'd allow k itself as an intermediate on the path to k — meaning k would appear twice, creating a cycle. Since we assume no negative cycles, this would never help, but using k−1 avoids the logical inconsistency and keeps the sub-problems well-defined.

---

### Tracing

**Q4.** Run Bellman-Ford on this graph (finding shortest path from s to t):

```
Vertices: s, a, b, t
Edges: s→a (cost 1), s→b (cost 4), a→b (cost -2), b→t (cost 2), a→t (cost 6)
```

**Answer:**

Base (i=0): OPT(0,t)=0, all others = ∞

i=1:
- OPT(1,b) = c(b,t) + OPT(0,t) = 2
- OPT(1,a) = min(c(a,t)+0, c(a,b)+∞) = 6
- OPT(1,s) = ∞ (no direct edge to t)

i=2:
- OPT(2,a) = min(OPT(1,a), c(a,b)+OPT(1,b), c(a,t)+OPT(1,t)) = min(6, -2+2, 6) = min(6,0,6) = 0
- OPT(2,s) = min(OPT(1,s), c(s,a)+OPT(1,a), c(s,b)+OPT(1,b)) = min(∞, 1+6, 4+2) = min(7,6) = 6

i=3:
- OPT(3,s) = min(OPT(2,s), c(s,a)+OPT(2,a), c(s,b)+OPT(2,b)) = min(6, 1+0, 4+2) = min(6,1,6) = 1

**Shortest path cost = 1.** Route: s → a → b → t (1 + (−2) + 2 = 1).

---

**Q5.** Run Floyd-Warshall on:

```
Vertices: 1, 2, 3
Edges: 1→2 (weight 3), 2→3 (weight 1), 1→3 (weight 10)
```

**Answer:**

Initial matrix D[i,j,0] (∞ = no direct edge):
```
     1    2    3
1 [  0    3   10 ]
2 [  ∞    0    1 ]
3 [  ∞    ∞    0 ]
```

k=1 (allow vertex 1 as intermediate): No improvement (no edges into 1 from 2 or 3).

k=2 (allow vertex 2):
- D[1,3,2] = min(D[1,3,1], D[1,2,1]+D[2,3,1]) = min(10, 3+1) = **4**

k=3: No further improvements.

Final answer: shortest path from 1→3 is 4 (via 1→2→3).
