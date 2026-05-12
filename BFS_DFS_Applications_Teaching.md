# Applications of BFS and DFS
### A Complete Teaching Guide — From Zero to Exam-Ready

---

## Before You Begin: What You Need to Know

This document assumes you know what a **directed graph** is (nodes connected by one-way arrows), and you roughly know that **DFS (Depth-First Search)** explores as deep as possible before backtracking. If you know those two things, you're ready.

One DFS detail you **must** understand before anything else:

> When DFS visits a vertex `u`, it records two timestamps:
> - `d[u]` — **discovery time** (when DFS first touches `u`, it turns **gray**)
> - `f[u]` — **finish time** (when DFS is completely done with `u` and all its descendants, it turns **black**)

These timestamps are the engine behind everything in this document.

---

---

# SECTION 1: Topological Sort

---

## 1.1 The Problem — Build Intuition First

Imagine you're a university student planning your courses. Some courses have prerequisites:

```
DSA ──→ DAA ──→ NLP
         ↑
Probability ──→ ML ──→ NLP
                 ↓
Optimization    FSL
         ↑
Linear Algebra
```

You **cannot** take DAA before DSA. You **cannot** take ML before Probability and Linear Algebra. But you need to choose an order to take all your courses.

This is **exactly** what Topological Sort solves:

> Given a set of tasks with dependencies, find a **linear ordering** of all tasks such that every dependency is respected.

The slide (page 3) shows this exact example with your actual CS courses: DSA → DAA → NLP, and Probability/Optimization/Linear Algebra → ML → NLP/FSL.

---

## 1.2 Formal Definition

A **topological sort** of a directed graph G(V, E) is a linear ordering of all vertices such that:

> If G contains an edge **(u, v)**, then **u appears before v** in the ordering.

**Critical constraint — it only works on a DAG:**

A **DAG** is a **Directed Acyclic Graph** — a directed graph with **no cycles**.

Why no cycles? If A depends on B and B depends on A, there's no valid ordering. You're stuck in a loop. Topological sort only makes sense when there's a clear "direction" to the dependencies.

---

## 1.3 The Algorithm

The algorithm is beautifully simple. It's just DFS with one extra step:

```
TOPOLOGICAL-SORT(G):
1. Call DFS(G) — compute finish time f[v] for every vertex v
2. As each vertex FINISHES (turns black), prepend it to a linked list
3. Return the linked list
```

**Why does prepending to the front work?**

Think about it: the vertex that finishes **last** has the highest `f[v]`. It has no unfinished dependencies — it's the one everything else depends on. It should come **first** in the ordering.

By prepending each finished vertex, the final list is automatically sorted in **decreasing order of finish time**, which is exactly the topological order.

---

## 1.4 Walkthrough Example

Look at the diagram on slide 6. The graph is:

```
a ──→ b ──→ d
      ↓   ↙ ↓
      c ──→ e ──→ f
```

DFS starts at `a`, goes deep: a → b → d → c → e → f

As vertices finish, they get prepended:

```
f finishes first   → list: [f]
e finishes next    → list: [e, f]
c finishes         → list: [c, e, f]
d finishes         → list: [d, c, e, f]
b finishes         → list: [b, d, c, e, f]
a finishes last    → list: [a, b, d, c, e, f]
```

Final topological order: **a → b → d → c → e → f**

The slide also shows the DFS tree on the right: a straight chain a→b→c→e→f with d branching off b. This is what the DFS forest looks like for this graph.

---

## 1.5 How to Detect if a Topological Sort Even Exists (Cycle Detection)

**Theorem:** A directed graph G is acyclic **if and only if** DFS of G produces **no back edges**.

A **back edge** is an edge (u, v) where v is an **ancestor** of u in the DFS tree — meaning DFS found a path from v down to u, and then u is trying to go back to v. That's a cycle.

**Proof (both directions):**

**Direction 1 — Back edge ⟹ Cycle:**
- Suppose DFS finds a back edge (u, v), meaning v is an ancestor of u in the DFS tree.
- There's already a path v →...→ u through the DFS tree.
- The back edge (u, v) adds an edge from u back to v.
- Together: v →...→ u → v. That's a **cycle**.

**Direction 2 — Cycle ⟹ Back edge** (from slide 8):
- Assume there's a directed cycle C in G.
- Run any DFS. Let `v` be the **first vertex** in cycle C to be discovered (it turns gray first).
- At the moment `v` is discovered (`d[v]`), every other vertex in the cycle is still **white** (not yet visited).
- So there's a white path from `v` all the way around the cycle to the vertex `u` where (u, v) is the last edge of the cycle.
- By the **White Path Theorem**: if there's a white path from v to u at time `d[v]`, then u becomes a descendant of v in the DFS tree.
- This means when DFS processes edge (u, v), v is already gray (an ancestor of u).
- Therefore (u, v) is a **back edge**.

**Practical implication:** While running DFS for topological sort, if you ever encounter an edge to a **gray** vertex (one that's currently being explored), you've found a cycle — topological sort is impossible.

---

## 1.6 Correctness of the Algorithm

**Claim:** For every edge (u, v) in the graph, `f[u] > f[v]`.

**Proof:**
When DFS explores edge (u, v):
- v cannot be **gray** (that would be a back edge = cycle, and we assumed it's a DAG)
- If v is **white**: v becomes a descendant of u, so v finishes before u → `f[v] < f[u]` ✓
- If v is **black**: v already finished, so `f[v] < f[u]` ✓

In both cases, u finishes after v. Since we prepend to the list as things finish, u appears before v in the output. This is exactly what we need.

---

## 1.7 Complexity

- DFS runs in **O(V + E)**
- Prepending to a linked list is **O(1)**
- **Total: O(V + E)** — linear in the size of the graph

---

## 1.8 Common Mistakes

| Mistake | Correction |
|---|---|
| Thinking topological sort is unique | It's **not** unique. Multiple valid orderings usually exist. |
| Trying to use it on a cyclic graph | Only works on DAGs. Always check for cycles first. |
| Confusing with sorting by discovery time | Sort by **finish time** (decreasing), not discovery time. |
| Appending instead of prepending | You must **prepend** each finished vertex, or reverse at the end. |

---

---

# SECTION 2: Strongly Connected Components (SCC)

---

## 2.1 The Problem — Build Intuition First

In a directed graph, just because you can get from A to B doesn't mean you can get back from B to A.

Think of a city's one-way street network. Some neighborhoods are "closed loops" — you can drive from any intersection in the neighborhood to any other, all within the neighborhood. Other streets only let you leave, never return.

A **Strongly Connected Component (SCC)** is exactly that "closed loop" neighborhood: a maximal group of vertices where you can get from any vertex to any other vertex.

**The problem:** Given a directed graph, find ALL such groups.

---

## 2.2 Formal Definition

A **Strongly Connected Component** of directed graph G(V, E) is a **maximal** set of vertices C ⊆ V such that:

> For every pair of vertices u and v in C, u is reachable from v **and** v is reachable from u.

"**Maximal**" means you can't add any more vertices to C and still have everyone reachable from everyone. You've found the biggest possible such group.

---

## 2.3 Example — Reading the Diagram

Look at slide 12. The graph has vertices {a, b, c, d, e, f}. The SCCs are:

```
SCC 1: {a, b, c, d}   ← everyone can reach everyone here
SCC 2: {e}            ← e is alone; no one from SCC1 can return from e
SCC 3: {f}            ← f is a dead-end
```

Slide 14 shows a richer example with 8 vertices {a,b,c,d,e,f,g,h} grouped into three SCCs:
- **V1** = {a, b, c} (with internal cycles)
- **V2** = {d, e} (with internal cycles)
- **V3** = {f, g, h} (with internal cycles)

The **component graph G_scc** on the right of slide 14 shows how the SCCs relate to each other: V1 → V2, V1 → V3. This component graph is always a **DAG** (we'll prove why shortly).

---

## 2.4 The Component Graph G_scc

Given G(V, E), we can build a **condensed** graph G_scc:

```
V_scc = { V1, V2, ..., Vk }   ← one node per SCC

Edge (Vi, Vj) ∈ E_scc  iff  there exist ui ∈ Vi, uj ∈ Vj
                              such that (ui, uj) ∈ E
```

Translation: There's an edge between two SCC-nodes if any original edge crosses from one SCC to the other.

**Key property: G_scc is always a DAG.**

Why? If G_scc had a cycle between two SCCs (say Vi → Vj → Vi), then every vertex in Vi could reach every vertex in Vj and vice versa — but then they would all be in the **same** SCC, contradiction. So no cycle can exist between distinct SCCs.

---

## 2.5 The Algorithm (Kosaraju's Algorithm)

This is the elegant 3-step algorithm shown on slide 16:

```
SCC(G):
Step 1: Call DFS(G) — compute finish time f[u] for every vertex u

Step 2: Compute G^T (the transpose of G)
        G^T has the same vertices but all edges reversed:
        E^T = { (u,v) | (v,u) ∈ E }

Step 3: Call DFS(G^T), exploring vertices in DECREASING order of f[u]
        (using the finish times from Step 1)

Step 4: Each tree in the DFS forest of Step 3 is one SCC
```

---

## 2.6 Why Does This Work? — The Core Intuition

**Why reverse the graph?**

Think about the component graph G_scc. Suppose SCCs are ordered C1, C2, ..., Ck in topological order (so all edges go left to right: C1 → C2 → ... → Ck).

From Step 1 (DFS on G), the SCC with the highest finish time is **C1** (it's first in topological order — it "dominates" everything).

Now if you run DFS from C1 on the **original** G, you'd explore C1 AND all SCCs reachable from it (C2, C3, ...). You can't isolate C1.

But on **G^T** (reversed graph), all edges between SCCs flip direction: C1 ← C2 ← ... ← Ck. Now starting DFS from C1 on G^T, you **cannot** escape C1 — no edges lead out of C1 in G^T. You only explore C1's own vertices.

That's the magic: the reversed graph traps you inside each SCC.

---

## 2.7 The Key Lemma (Slide 20–21)

This is the formal backbone. We define:

> **f(C)** = max{ f[u] | u ∈ C }  — the finish time of an SCC = the latest finish time of any vertex in it.

**Lemma 1 (Original graph G):**
If there's an edge (u, v) ∈ E with u ∈ C and v ∈ C' (two different SCCs), then:
> **f(C) > f(C')**

Meaning: the SCC you're leaving from always has a higher finish time than the SCC you're going into. This matches topological order of the component graph.

**Lemma 2 (Transposed graph G^T):**
If there's an edge (u, v) ∈ E^T with u ∈ C and v ∈ C', then:
> **f(C) < f(C')**

This is just Lemma 1 applied to the reversed graph. In G^T, edges flip, so the ordering of finish times flips too.

**Why Lemma 2 is the key to correctness:**

In Step 3, we start DFS on G^T from the vertex with the **highest** f[u] globally. That vertex belongs to the SCC C with the **highest** f(C). By Lemma 2, all edges out of C in G^T go to SCCs with even lower f values — but since we start from the maximum, those SCCs' vertices haven't been picked as roots yet in this DFS. So the DFS from C on G^T **stays within C**. ✓

Then we pick the next unvisited vertex (highest remaining f[u]), which is the root of the next SCC — and the same argument applies.

---

## 2.8 Correctness Proof (Induction — Slide 22–23)

The proof on slides 22–23 is by **induction on the number of trees in the DFS forest** of G^T.

**Base case:** 0 trees — trivially correct.

**Inductive step:**
- Assume the first k trees correctly correspond to k SCCs.
- Consider the (k+1)-th tree. Let u be its root, and C be the SCC containing u.
- When u is chosen (highest remaining f[u]), every vertex in C is still **white** (not yet visited).
  - Why? Because all previously completed trees belong to SCCs with *lower* f values by Lemma 2 — they are different SCCs.
- By the **White Path Theorem**: since there's a path from u to every vertex in C (they're in the same SCC), and all of C is white at time d[u], all of C's vertices become descendants of u in this DFS tree.
- Can the tree include vertices **outside** C? No — by Lemma 2, any edge out of C in G^T goes to an SCC with *lower* f value, and those vertices are either already completed (black) or belong to other SCCs. They won't be pulled into u's tree.

Therefore, u's tree = exactly C. ✓

---

## 2.9 Walkthrough Example (Slide 15)

Graph has 8 vertices. Step 1 DFS on G gives finish order (bottom to top of the stack shown):
```
h, f, g, e, d, c, b, a   (a has highest finish time)
```

Step 2: Reverse all edges → G^T

Step 3: DFS on G^T starting from `a` (highest f):
- From a, can reach b, c (staying within V1 = {a,b,c}) → **Tree 1 = SCC V1**
- Next: start from `d` (next highest f among unvisited)
  - From d, can reach e → **Tree 2 = SCC V2 = {d, e}**  
- Next: start from `g` (or f or h, next highest)
  - Reaches f, h → **Tree 3 = SCC V3 = {f, g, h}**

Three trees = three SCCs. Done.

---

## 2.10 Complexity

- Step 1: DFS(G) → **O(V + E)**
- Step 2: Build G^T → **O(V + E)**
- Step 3: DFS(G^T) → **O(V + E)**
- **Total: O(V + E)** — linear time!

---

## 2.11 Common Mistakes

| Mistake | Correction |
|---|---|
| Running DFS on G^T in arbitrary order | MUST use decreasing finish times from Step 1 |
| Forgetting that G_scc is a DAG | This property is crucial — SCCs can never have cycles between them |
| Confusing "reachable" with "mutually reachable" | SCC requires reachability **in both directions** |
| Thinking a single node can't be an SCC | It can — any node with no mutual reachability is its own SCC |
| Running Step 1 on G^T instead of G | Step 1 is always on the original G; Step 3 is on G^T |

---

---

# BIG PICTURE: How Everything Connects

```
DFS + finish times
        │
        ├──→ Topological Sort
        │    (prepend on finish, valid only on DAGs)
        │    (back edges = cycle = no topo sort possible)
        │
        └──→ SCC (Kosaraju's)
             Step 1: DFS on G → finish times
             Step 2: Reverse graph → G^T
             Step 3: DFS on G^T in decreasing f order
             (each DFS tree = one SCC)
             (G_scc is always a DAG → has a topo sort)
```

Both algorithms are **O(V + E)** and both are powered by the **finish time** property of DFS.

The SCC algorithm secretly uses topological sort logic internally — the decreasing finish time order in Step 3 is exactly the topological order of G_scc.

---

---

# CHEAT SHEET

## Topological Sort
| Property | Value |
|---|---|
| Input | DAG G(V, E) |
| Output | Linear ordering respecting all edges |
| Algorithm | DFS; prepend vertex to list when it finishes |
| Key insight | Higher finish time = appears earlier in order |
| Complexity | O(V + E) |
| Uniqueness | Not unique in general |
| Cycle detection | Back edge found during DFS ⟺ cycle exists |

## SCC (Kosaraju's)
| Property | Value |
|---|---|
| Input | Directed graph G(V, E) |
| Output | All strongly connected components |
| Step 1 | DFS(G) → compute f[u] for all u |
| Step 2 | Build G^T (reverse all edges) |
| Step 3 | DFS(G^T) in decreasing f[u] order |
| Result | Each DFS tree in Step 3 = one SCC |
| Complexity | O(V + E) |
| G_scc | Always a DAG |

## Key Definitions
- **DAG**: Directed Acyclic Graph — no directed cycles
- **Back edge**: Edge (u,v) where v is a gray ancestor of u during DFS → indicates a cycle
- **f(C)**: finish time of an SCC = max finish time of any vertex in it
- **G^T**: transpose of G — same vertices, all edges reversed
- **White Path Theorem**: In DFS, v is a descendant of u iff at time d[u], there exists a white path u→...→v

## Critical Lemmas
- Edge (u,v) ∈ E, u ∈ C, v ∈ C' (different SCCs) → **f(C) > f(C')**
- Edge (u,v) ∈ E^T, u ∈ C, v ∈ C' (different SCCs) → **f(C) < f(C')**

---

---

# PRACTICE QUESTIONS

## Theory

**Q1.** Can a graph have more than one valid topological ordering? Give an example or prove it's always unique.

**Q2.** True or False: If DFS of an undirected graph yields no back edges, the graph is acyclic. Is this the same condition as for directed graphs?

**Q3.** Why must the component graph G_scc always be a DAG? Prove it in 3 sentences.

**Q4.** In Kosaraju's algorithm, why do we run the second DFS on G^T instead of G?

**Q5.** What happens to Kosaraju's algorithm if you run Step 3 in **increasing** order of finish time instead of decreasing?

---

## Tracing Problems

**Q6.** Run topological sort on this graph:
```
A → C
A → B
B → D
C → D
D → E
```
What is one valid topological ordering? What are all valid orderings?

**Q7.** Find all SCCs in this graph:
```
Vertices: {1, 2, 3, 4, 5}
Edges: 1→2, 2→3, 3→1, 2→4, 4→5, 5→4
```

---

## Worked Solutions

**A1.** No, topological ordering is generally NOT unique. Example:
```
A → C
B → C
```
Both `A, B, C` and `B, A, C` are valid. Only a completely linear chain (one path through all nodes) gives a unique ordering.

**A2.** False — and this is a subtle difference. In an **undirected** graph, any non-tree edge is a back edge (since edges are bidirectional). What you're looking for in undirected graphs is the absence of cross edges during DFS. In **directed** graphs, back edges specifically (to gray/ancestor nodes) signal cycles. The condition differs.

**A3.** Suppose G_scc had a cycle between SCCs: C1 → C2 → ... → C1. Then every vertex in C1 can reach every vertex in C2 (through the cycle), and every vertex in C2 can reach every vertex in C1. But then C1 and C2 would be the same SCC — contradiction with them being distinct. Therefore G_scc is acyclic, i.e., a DAG.

**A4.** On the original G, starting DFS from the highest-f vertex would also explore all reachable SCCs beyond it — you can't isolate one SCC. On G^T, all inter-SCC edges are reversed. The SCC with the highest finish time has no outgoing edges in G^T (they all point back in), so DFS stays trapped within just that one SCC.

**A5.** You'd start from the SCC with the **lowest** finish time — the "last" SCC in topological order, the one no other SCC points to. Starting DFS there on G^T would pull in vertices from many other SCCs (since in G^T, edges from "later" SCCs point back toward "earlier" ones). The algorithm breaks completely.

**A6.**
DFS finishes in order: E, D, B, C, A (one possible order — depends on DFS choices).
Prepend each: [A, C, B, D, E] or [A, B, C, D, E]
Both are valid topological orderings.
All valid orderings: A must come first, E must come last, B and C are interchangeable.
Valid orderings: `A,B,C,D,E` and `A,C,B,D,E`.

**A7.**
- Edge (3→1) and path 1→2→3 forms a cycle → **SCC1 = {1, 2, 3}**
- Edge (5→4) and (4→5) forms a cycle → **SCC2 = {4, 5}**
- No mutual reachability between these two groups.
- **Final SCCs: {1,2,3} and {4,5}**
- Component graph: {1,2,3} → {4,5} (because of edge 2→4)
