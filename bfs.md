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

# Section 2: BFS — Shortest Path Properties & Correctness Proofs

---

## Core Intuition

Here's the key claim about BFS:

> The value `d[v]` that BFS computes is not just *some* distance — it is the **shortest** possible distance from `s` to `v`.

Why should this be true? Because BFS processes vertices in order of increasing distance. It finishes all distance-1 vertices before touching distance-2 vertices. So the first time BFS reaches a vertex, it *must* be via the shortest path. It literally cannot reach a vertex "too early" through a longer path.

---

## The δ Notation

Slide 18 introduces notation you'll see in all the proofs:

> **δ(u, v)** = the true shortest path distance from u to v in the graph

So the claim becomes: after BFS finishes, **d[v] = δ(s, v)** for every reachable vertex v.

The slide also gives a simple but important lemma:

> If (u, v) is an edge, then **δ(s, v) ≤ δ(s, u) + 1**

This is just the triangle inequality — the shortest path to v can't be longer than going through u and then taking the edge (u,v). Look at the small diagram on slide 18: s reaches u via some path, then one more hop gets to v.

---

## The BFS Tree

Slide 8 mentions this, and slide 28 states it clearly.

The **π[v]** values recorded during BFS form a tree rooted at s — the **BFS tree**. The path from s to any vertex v in this tree, traced via π pointers, is a shortest path in the original graph.

```
         s
        / \
       a   b
      /     \
     c       d
```

So if you want the actual shortest path (not just the distance), you follow π[v] → π[π[v]] → ... → s backwards.

---

## Proving d[v] = δ(s,v): Two-Part Strategy

The full proof on slides 19–30 splits into two inequalities:

```
d[v] ≥ δ(s,v)    ... BFS never underestimates
d[v] ≤ δ(s,v)    ... BFS never overestimates
```

Together these force d[v] = δ(s,v).

---

## Part 1: d[v] ≥ δ(s,v)

**Intuition:** BFS can only assign distances by following edges. It can never "teleport" to a vertex and assign it a distance smaller than the true shortest path.

Look at slide 19. The proof is by induction on the number of Enqueue operations.

**Base case:** The first enqueue is s itself. d[s] = 0 = δ(s,s). ✓

**Inductive step:** When vertex v is discovered through neighbor u:
```
d[v] = d[u] + 1
     ≥ δ(s,u) + 1    ← by inductive hypothesis on u
     ≥ δ(s,v)        ← by the triangle inequality lemma
```

That last step uses the fact that since (u,v) is an edge, δ(s,v) ≤ δ(s,u)+1.

---

## The Queue Ordering Lemma

Before proving the other direction, the slides prove a critical structural fact about BFS. Look at slide 21.

**Lemma:** At any point during BFS, if the queue contains {v₁, v₂, ..., vᵣ} where v₁ is the head, then:
```
d[v₁] ≤ d[v₂] ≤ ... ≤ d[vᵣ]    (non-decreasing order)
d[vᵣ] ≤ d[v₁] + 1               (tail is at most 1 more than head)
```

**Intuition:** The queue always holds at most two consecutive "levels" of the BFS wave. You never have a distance-5 vertex sitting ahead of a distance-3 vertex. The wave is orderly.

Look at slide 24 for the proof sketch. It handles two cases:

**Case 1 — Dequeue:** When v₁ is removed, the queue becomes {v₂,...,vᵣ}. We need d[vᵣ] ≤ d[v₂]+1. By hypothesis d[vᵣ] ≤ d[v₁]+1 ≤ d[v₂]+1. ✓

**Case 2 — Enqueue:** A new vertex vᵣ₊₁ is added, discovered through some vertex u that was just dequeued. So d[vᵣ₊₁] = d[u]+1. Since u was dequeued before v₁ was enqueued, d[u] ≤ d[v₁], giving d[vᵣ₊₁] ≤ d[v₁]+1. ✓

The **corollary** on slide 26 follows immediately: if vᵢ was enqueued before vⱼ, then d[vᵢ] ≤ d[vⱼ]. Earlier enqueued = smaller or equal distance. This is just saying the wave is ordered.

---

## Part 2: d[v] ≤ δ(s,v) — The Main Theorem

Slide 28 states the full theorem:

> BFS discovers every reachable vertex, and upon termination d[v] = δ(s,v).

The proof on slides 29–30 is by contradiction.

**Assume not true:** There exists some vertex v where d[v] > δ(s,v). Pick v to be the closest such vertex to s.

Let u be the vertex just before v on the true shortest path s→...→u→v. So:
```
δ(s,v) = δ(s,u) + 1
```

Since v was chosen as the closest "bad" vertex and u is even closer, u must be correctly handled:
```
d[u] = δ(s,u)
```

Therefore:
```
d[v] > δ(s,v) = δ(s,u) + 1 = d[u] + 1
```

Now when BFS dequeued u, it looked at all of u's neighbors including v. What was v's color at that moment? Three cases on slide 29:

- **v was WHITE:** Then BFS set d[v] = d[u]+1. But we just said d[v] > d[u]+1. Contradiction. ✗
- **v was BLACK:** v was already processed, so d[v] ≤ d[u] (by the queue ordering corollary, since v was dequeued before u was). But d[u]+1 > d[v] means d[v] < d[u]+1, i.e., d[v] ≤ d[u]. This means δ(s,v) ≤ d[v] ≤ d[u] = δ(s,u), contradicting δ(s,v) = δ(s,u)+1. ✗
- **v was GRAY:** v was discovered through some other vertex w ≠ u. By the queue ordering lemma, since w was enqueued before u, d[w] ≤ d[u], so d[v] = d[w]+1 ≤ d[u]+1. Contradicts d[v] > d[u]+1. ✗

All cases give a contradiction. So the assumption was wrong, and d[v] = δ(s,v). ✓

---

## Common Mistakes

**1.** Confusing δ (true shortest path) with d (what BFS computes). The whole point of the proof is showing they are equal.

**2.** Thinking the queue ordering lemma is trivial. It's actually the key structural fact that makes the contradiction argument in Part 2 work.

**3.** Forgetting that d[v] = δ(s,v) only holds for **reachable** vertices. Unreachable vertices keep d[v] = ∞, which correctly equals δ(s,v) = ∞.

---

Ready for **Section 3: DFS — Algorithm, Variables & Complexity**?
(This is where your Image 1 comes in directly.)
