# Section 1: BFS — Algorithm, Intuition & Complexity

---

## Core Intuition

Imagine you drop a stone in a pond. Ripples spread outward in concentric circles — first the ring closest to you, then the next ring, then the next. BFS works exactly like that.

You start at a source vertex `s`, visit all neighbors at distance 1 first, then all neighbors at distance 2, and so on. You never go "deep" until you've finished the current "wave."

This is the key contrast with DFS (which you already know from your notes) — BFS is **wide before deep**, DFS is **deep before wide**.

---

## What BFS Does

Look at slide 5. The definition is simple:

> Given a graph G(V,E) and a source vertex `s`, BFS discovers every vertex reachable from `s`.

Two things to note:
- It only discovers vertices **reachable** from `s`. If the graph is disconnected, vertices in other components are never visited.
- It works on both directed and undirected graphs.

---

## The 3-Color System

Look at slide 7. BFS tracks the state of every vertex using three colors:

```
WHITE  → not yet discovered (untouched)
GRAY   → discovered, but neighbors not fully processed yet
BLACK  → discovered, and all neighbors have been seen
```

Think of it this way: gray vertices are the "frontier" — the active wave. Black vertices are done. White vertices are yet to be reached.

The GRAY/BLACK distinction is mostly for theoretical analysis. In practice (like your Image 1 for DFS), you just need visited/not-visited.

---

## The BFS Algorithm

Now look at slides 9 and 10 together. BFS uses a **queue** (FIFO). This is what enforces the wave-by-wave order.

**Initialization (slide 9):**

```
for each vertex u ≠ s:
    color[u] = WHITE
    d[u] = ∞          ← distance from s, unknown yet
    π[u] = NIL         ← no parent yet

color[s] = GRAY        ← source is immediately discovered
d[s] = 0              ← distance to itself is 0
π[s] = NIL
Q = ∅                 ← empty queue
```

**Main loop (slide 10):**

```
Enqueue(Q, s)

while Q ≠ ∅:
    u = Dequeue(Q)             ← take the front vertex
    for v in Adj[u]:           ← look at all neighbors
        if color[v] = WHITE:   ← only process undiscovered vertices
            color[v] = GRAY
            d[v] = d[u] + 1    ← one step further than u
            π[v] = u           ← u is v's parent in BFS tree
            Enqueue(Q, v)
    color[u] = BLACK           ← done with u
```

**Walk through with the diagram on slide 6:**

Starting at `s`, neighbors `a` and `b` are discovered first (distance 1, both go into the queue). Then when `a` is dequeued, its neighbor `c` is discovered (distance 2). The queue shown at the bottom of slide 6 — `s, a, b` — is exactly this frontier in action. Notice `d[c] = 2` written on that slide.

---

## Time Complexity

Slides 11–15 build this up carefully.

**Initialization:** You loop over all vertices once → **O(V)**

**Queue operations:** Every vertex enters and leaves the queue exactly once (because once it's GRAY it's never WHITE again, so it's never re-enqueued) → **O(V)**

**Adjacency list scanning:** When a vertex `u` is dequeued, you scan all its neighbors. Across all vertices, this is the sum of all degrees = 2|E| for undirected, |E| for directed → **O(E)**

**Total: O(V + E)** — also written O(n + m) in your slides.

This is optimal — you can't do better because just reading the graph takes O(V+E).

---

## Common Mistake

> "BFS visits all vertices of the graph."

**Wrong.** BFS only visits vertices reachable from `s`. If the graph has disconnected components, those vertices stay WHITE forever. To visit every vertex in a disconnected graph, you'd need to run BFS from multiple sources (similar to how DFS(G) in your notes loops over all vertices).

---

Ready for **Section 2: BFS — Shortest Path Properties & Correctness Proofs**?
