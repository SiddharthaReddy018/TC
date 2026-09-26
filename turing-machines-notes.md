# Turing Machines for Computation — Exam Notes
*(Prof. Shrisha Rao, IIIT Bangalore, 2026-09-10)*

---

## PART 0 — Ground Rules / Conventions

Before any machine makes sense, fix three conventions used throughout:

1. **Arrow labels** are read as: `oldSymbol newSymbol moveDirection`
   Example: `0 Δ R` on an arrow from state P to state Q means: "while in state P, if you read a 0, overwrite it with Δ (blank), move the head one cell Right, and go to state Q." This is just δ(state, symbol) = (newState, newSymbol, direction), written on the arrow.

2. **Accepting states** are drawn as double circles.

3. **Erasing vs. marking.** For the harder languages here (balanced brackets, `0^n1^n2^n`), the machine does **not** erase symbols to record "already handled." Instead it overwrites the symbol with a *marker* (like X, Y, Z), added to the tape alphabet Γ (not part of input alphabet Σ).

   **Why:** if you erase (write Δ) in the *middle* of the string, you punch a blank hole there. Since "blank = outside the input" is the standing convention, an interior hole gets mistaken for the edge of the string. A marker avoids this — the cell stays non-blank, tape stays contiguous, but the machine still knows "this position is used up."

   **Contrast:** in `{0^n1^2n}`, erasing IS safe, because all erasures happen only at the two *ends* of the shrinking string — never the middle. So Γ = {0,1,Δ}, no markers needed.

4. **"Tape is blank outside the input"** — this is why a machine can always find the edges of its own input: scanning far enough left or right always lands on Δ.

---

## PART 1 — TM for `{0^n 1^(2n) | n ≥ 0}`

**Intuition:** generalizes the classic `{0^n1^n}` matcher. Every 0 needs *two* 1's, so: erase one 0 from the left, then erase the *last two* 1's from the right, repeat. If the tape empties out cleanly, accept.

**Why erase-only works:** all action happens at the two extreme ends of what's left — interior is only scanned through, never touched. No hole problem. Γ = {0,1,Δ}.

**Pseudocode:**
```
(i)    If Δ -> accept. If 0 -> write Δ, move R. Else reject.
(ii)   Move R until Δ, then move one step L.
(iii)  If 1 -> write Δ, move L. Else reject.
(iv)   If 1 -> write Δ, move L. Else reject.
(v)    Move L until Δ, then move one step R.
(vi)   Goto (i).
```
In English: (i) kill one 0 from the left (or accept if nothing's left). (ii) walk to the right edge. (iii)-(iv) kill two 1's from the right edge (two steps, one per 1 removed). (v) walk back to the left edge. (vi) repeat.

**State diagram (6 states: A start, F accept):**
```
A --0,Δ,R--> B            (erase a 0, step right)
B --0,0,R / 1,1,R--> B     (self-loop: skip over remaining 0s/1s)
B --Δ,Δ,L--> C             (hit right blank, step back left)
C --1,Δ,L--> D             (erase rightmost 1)
D --1,Δ,L--> E             (erase 2nd rightmost 1)
E --0,0,L / 1,1,L--> E     (self-loop: skip left over remainder)
E --Δ,Δ,R--> A             (hit left blank, step back right — restart cycle)
A --Δ,Δ,S--> F             (nothing left to erase, tape blank: ACCEPT)
```

### Worked example 1: `"001111"` (0²1⁴ — should ACCEPT, since 2n=4)

Tape cells 0..5: `0 0 1 1 1 1`

| step | state | head | read | action | tape after |
|---|---|---|---|---|---|
| 1 | A | 0 | 0 | erase→Δ, R | Δ0 1 1 1 1 |
| 2-6 | B | 1..5 | 0/1 | self-loop R | (same) |
| 7 | B | 6 | Δ | →C, L | head=5 |
| 8 | C | 5 | 1 | erase→Δ, →D, L | Δ0 1 1 1 Δ |
| 9 | D | 4 | 1 | erase→Δ, →E, L | Δ0 1 1 Δ Δ |
| 10-12 | E | 3,2,1 | 1/1/0 | self-loop L | (same) |
| 13 | E | 0 | Δ | →A, R | head=1 |
| 14 | A | 1 | 0 | erase→Δ, R | Δ Δ 1 1 Δ Δ |
| 15-16 | B | 2,3 | 1,1 | self-loop R | |
| 17 | B | 4 | Δ | →C, L | head=3 |
| 18 | C | 3 | 1 | erase→Δ, →D, L | Δ Δ 1 Δ Δ Δ |
| 19 | D | 2 | 1 | erase→Δ, →E, L | Δ Δ Δ Δ Δ Δ |
| 20 | E | 1 | Δ | →A, R | head=2 |
| 21 | A | 2 | Δ | **ACCEPT** | |

Correct — `001111` = 0²1⁴ fits the pattern.

### Worked example 2: `"00011"` (0³1² — should REJECT, needs 6 ones not 2)

Trace shows the machine erasing 0's from the left, erasing pairs of 1's from the right, until eventually a `0` is found where a `1` was expected in state C → **REJECT**. Correct: 3 zeros needs 6 ones, only 2 are present.

**Complexity:** each cycle scans the remaining tape twice (O(length)); O(n) cycles → **O(n²)** total steps.

---

## PART 2 — TM for Balanced Brackets over `{ (, ) }`

**Intuition:** a string is balanced exactly when you can repeatedly cancel a `)` against the nearest unmatched `(` to its left, ending with nothing left over. Core loop: find leftmost unmatched `)`, find nearest unmatched `(` to its left, mark **both** with X, repeat from the left edge.

**Why marking is required:** a matched pair like the inner `()` in `(()())` sits in the *middle* of the string. Erasing it would punch a hole there, and the machine would wrongly think the string ends early. So Γ = {(, ), X, Δ}.

**Pseudocode (head starts at left end):**
```
(i)   No unpaired "(" seen yet.
        Δ -> accept.
        X -> R, repeat (i).
        ( -> R, goto (ii).
        ) -> reject.
(ii)  An unpaired "(" has been seen (scanning right for its partner).
        X or ( -> R, repeat (ii).
        Δ -> reject (no ")" ever showed up).
        ) -> write X, L, goto (iii).
(iii) Move left over X until "(" is found. Write X there. Move one step left.
(iv)  Move left over X and ( until Δ. Move one step right. Goto (i).
```

**State diagram (5 states: A start, E accept):**
```
A --(,(,R--> B                (found unpaired "(", start hunting right)
A --X,X,R--> A                (self-loop, skip already-matched)
A --Δ,Δ,S--> E                (ACCEPT — nothing left, tape all matched/blank)
A --),?,?--> reject
B --X,X,R / (,(,R--> B        (self-loop, keep hunting right for partner ")")
B --),X,L--> C                (found partner ")", mark it X, step left)
B --Δ,Δ,?--> reject           (ran off the end — no partner)
C --X,X,L--> C                (self-loop, skip already-matched, hunting left)
C --(,X,L--> D                (found the "(" partner: mark X, step left)
D --X,X,L / (,(,L--> D        (self-loop: sweep back to start)
D --Δ,Δ,R--> A                (hit left edge, step right, restart cycle)
```

### Worked example 1: `"(()())"` — should ACCEPT

Full trace (3 rounds) confirms the machine progressively marks every bracket with X, and on the final pass state A skips over all X's and hits Δ → **ACCEPT**. Correct.

### Worked example 2: `"())((" ` — should REJECT

Trace: first `(` at cell 0 pairs with `)` at cell 1 (both become X). Then state A skips the X's and hits the **second** `)` at cell 2 with no unpaired `(` pending → **REJECT** per step (i). Correct — this string has an unmatched `)`.

**Complexity:** each round is an O(length) sweep; O(n) rounds (n = matched pairs) → **O(n²)** total.

---

## PART 3 — TM for `{0^n 1^n 2^n | n ≥ 0}`

**Why this needs a TM, not a PDA:** this language is **not context-free** (classic pumping-lemma result). A PDA's stack can compare *two* quantities (push on block 1, pop on block 2) but by the time the third block arrives, the stack info from block 1 is already consumed — there's no way to compare three things at once with one LIFO stack. A TM has no such limit: it can rescan the tape as many times as needed. **This is the headline demonstration that TMs are strictly more powerful than PDAs.**

**What must be checked:**
1. **Shape check** — string must look like `0*1*2*` (correct order, no stray symbols).
2. **Count check** — the three blocks must be equal length.

Two solutions differ in *how* they interleave the count check.

### 3a. First Solution: Two Passes (reduction)

**Idea:** Phase 1 matches 0's against 2's (erase a 0 from the left, mark the corresponding 2 with X, left to right). For a valid string this leaves `Δ...Δ 1^n X^n` on the tape. Phase 2 is then *exactly* the `{0^n1^n}` machine, renamed: "1" plays the role of "0", "X" plays the role of "1".

Γ = {0,1,2,X,Δ} — only **1 extra symbol**, because Phase 2 reuses erase-based matching (safe again, since it's back to a two-block end-matching problem).

**Pseudocode:**
```
Phase 1 (match 0's against 2's):
(i)   Δ->accept. 1->goto(v). 0->write Δ, R. else reject.
(ii)  Move R over 0,1,X until 2; write X, L. Reject if Δ read first.
(iii) Move L until Δ, then R.
(iv)  Goto (i).
Phase 2 (match 1's against X's, exactly like {0^n1^n}):
(v)   Δ->accept. 1->write Δ, R. else reject.
(vi)  Move R until Δ, then L.
(vii) X->write Δ, L. else reject.
(viii)Move L until Δ, then R.
(ix)  Goto (v).
```

**States:** A,B,C do phase 1; D,E,F,G do phase 2; H = accept.
```
A --0,Δ,R--> B              (erase a 0)
B --0,0,R/1,1,R--> B        (scan right, skipping 0's/1's/X's)
B --2,X,L--> C              (found a 2, mark X, step left)
C --0,0,L/1,1,L/X,X,L--> C  (scan back left to start)
C --Δ,Δ,R--> A              (restart phase 1)
A --1,1,S--> D              (no 0's left → enter phase 2)
D --1,Δ,R--> E              (erase a 1)
E --1,1,R/X,X,R--> E        (scan right)
E --Δ,Δ,L--> F              (hit right edge)
F --X,Δ,L--> G              (erase matching X)
G --1,1,L/X,X,L--> G        (scan back left)
G --Δ,Δ,R--> D              (restart phase 2)
D --Δ,Δ,S--> H              (ACCEPT)
```

### Worked example: `"001122"` (n=2, ACCEPT)

Full step trace (2 rounds of phase 1, pairing 0's with 2's; 2 rounds of phase 2, pairing 1's with X's) ends with the tape entirely Δ and machine in state D reading Δ → **ACCEPT**.

### Worked example: `"0011122"` (0²1³2² — REJECT)

Phase 1 pairs 2 zeros with 2 twos successfully (counts match there). Phase 2 then tries to pair 3 ones against only 2 X's — the surplus 1 finds Δ instead of an X partner in step (vii) → **REJECT**. Confirms correct detection of a mismatched *middle* block even when outer blocks match.

**States/symbols needed:** 8 states, 1 extra tape symbol.

### 3b. Second Solution: One Sweep Marks One of Each

**Idea:** instead of two passes, one sweep marks one 0, then one 1, then one 2 — all in a single left-to-right pass — then returns to the start and repeats. Nothing is ever erased.

Γ = {0,1,2,X,Y,Z,Δ} — **3 extra symbols** (X for 0's, Y for 1's, Z for 2's), since three counters are tracked simultaneously.

**Pseudocode:**
```
(i)   Start of a sweep. Δ->accept. Y-> (no unmarked 0's left) goto(v). 0->write X, R. else reject.
(ii)  Move R over 0,Y until 1; write Y, R. Reject on anything else.
(iii) Move R over 1,Z until 2; write Z, L. Reject on anything else.
(iv)  Move L over 0,1,Y,Z until X; R, goto (i).
(v)   Move R over Y, then over Z. Δ->accept, else reject.
```

**States:** A,B,C mark 0→X, 1→Y, 2→Z in sequence within one sweep; D returns left to find the next 0; F,G handle the final check; H = accept.
```
A --0,X,R--> B          (mark a 0)
B --0,0,R/Y,Y,R--> B    (skip to find next unmarked 1)
B --1,Y,R--> C          (mark a 1)
C --1,1,R/Z,Z,R--> C    (skip to find next unmarked 2)
C --2,Z,L--> D          (mark a 2, sweep back left)
D --0,0,L/1,1,L/Y,Y,L/Z,Z,L--> D
D --X,X,R--> A          (found start of X-block, next sweep)
A --Y,Y,R--> F          (0-block exhausted — verify the rest)
F --Y,Y,R--> F
F --Z,Z,R--> G
G --Z,Z,R--> G
G --Δ,Δ,S--> H          (ACCEPT)
```

**Correctness invariant:** after `k` complete sweeps on a valid string, the tape reads
```
X^k 0^(n-k) Y^k 1^(n-k) Z^k 2^(n-k)
```
and the head sits on the leftmost non-X cell. That one cell decides everything:
- **0** → more matching to do, sweep again.
- **Y** → 0's ran out exactly at k=n; check the rest is Y's-then-Z's-then-blank.
- anything else → shape/order/count violation → **reject**.

### Worked example: `"012"` (n=1, ACCEPT)

After 1 sweep the tape is exactly `X Y Z` — matching the invariant formula with n=1, k=1. State A then reads `Y`, switches to the verification phase, skips over Y then Z, hits Δ → **ACCEPT**.

**States/symbols needed:** 7 states, 3 extra tape symbols.

**Trade-off (explicitly stated in the lecture):** First solution needs 1 extra symbol and 8 states; second needs 3 extra symbols and 7 states. Fewer states ↔ more alphabet symbols — no free lunch.

---

## PART 4 — Turing Machines as Calculators

**Language acceptors vs. calculators:** so far every machine reads a string and outputs one bit (accept/reject). A TM can instead be a **calculator**: reads numbers as input, and when it halts, the tape holds the numeric **answer**.

**Church-Turing Thesis:** any function computable by *any* mechanical procedure whatsoever (any algorithm, on any device) can be computed by *some* Turing machine. This is a **thesis**, not a theorem — there's no formal definition of "any mechanical procedure" to prove it against — but it has never been contradicted in ~90 years, and it's why TMs are treated as the standard definition of "computable."

---

## PART 5 — Giving Numerical Inputs / Unary Alphabet

- Numbers are written on the tape before the machine starts, head at the leftmost non-blank cell.
- **Multiple inputs** (e.g. two numbers to add) are written one after another, **separated by a single Δ**. This Δ is meaningful structural data here — not "outside the input."
- **Unary alphabet:** only one symbol, `1`, used to represent numbers (keeps arithmetic machines simple — no carries, no place value).

**Encoding rule (important, easy to misremember):**
```
0        -> "1"          (1^1)
1        -> "11"         (1^2)
n (n>1)  -> 1^(n+1)       (n+1 copies of "1")
```
General rule: natural number `n` ↔ tape-string `1^(n+1)`. To recover the number from a tape-string of `k` ones: **n = k − 1**.

**Why the "+1" offset:** it guarantees the number 0 still gets a *non-empty* tape string (a lone "1"), so a totally blank tape never has to double as "the number zero" — blank always unambiguously means "no input here / end of input."

---

## PART 6 — TM to Add 1 (simplest calculator)

**Problem:** tape holds `1^k` (k>0); output should be `1^(k+1)` (append one more 1).

**"Simplest possible TM calculation":** walk to the right end of the input, write one more 1, halt. No matching, no back-and-forth.

**Construction:**
```
S0 (start): self-loop on 1,1,R   (walk right over existing 1's)
S0 --Δ,1,R--> H                  (write one more 1 at the blank, halt)
```

If the tape held `1^(n+1)` (representing n), after this machine it holds `1^(n+2)` (representing n+1). This literally computes **successor**.

---

## PART 7 — More Realistic Addition (m + n)

**Setup:** m written as `1^(m+1)`, n written as `1^(n+1)`, separated by one Δ.
```
Starting tape:  1^(m+1) Δ 1^(n+1)
Desired tape:   1^(m+n+1)
```

**Key trick:** `1^(m+1) Δ 1^(n+1)` has, if you just erase the middle Δ, a total of `(m+1)+(n+1) = m+n+2` ones — exactly **two too many**. So:
1. Overwrite the middle Δ with a `1` (joins the blocks; total = m+n+2 ones).
2. Delete exactly **two** ones from *either* end (doesn't matter which — unary numbers only encode a count, not position) → `1^(m+n+1)`.

**Worked example (from the slide):** m=2, n=2.
```
Start:  111 Δ 111        (3+1+3 = seven ones after step 1: 1111111)
Step 1: 1111111
Step 2: delete two -> 11111   (five ones)
Check: five ones -> number 5-1 = 4. And 2+2=4. Correct!
```

**Why "delete from either end" is safe:** unary numbers encode only a *count* — every `1` is interchangeable. Contrast with the bracket-matching / block-counting machines above, where *which* symbol you touch mattered a great deal (front vs. back, marked vs. unmarked).

**Extending to 3+ numbers:** "handled in the same way, sequentially" — add the first two to get a partial sum, then add the third to that partial sum, and so on (function composition).

---

## PART 8 — Multiplication by 2 (multiplication as repeated addition)

**Problem:** given m on the tape (as `1^(m+1)`), compute 2m.

**Key idea:** `2m = m + m`. Reuse the addition machine from Part 7 rather than building multiplication from scratch. The only new step needed: **duplicate** the number on the tape with a Δ gap.

```
Start:  1^(m+1)
Step 1 (duplicate): 1^(m+1) Δ 1^(m+1)
Step 2: run the Part-7 addition machine on this (shape "1^(a+1) Δ 1^(b+1)" with a=b=m)
Result: 1^(m+m+1) = 1^(2m+1)  -> correctly represents 2m
```

**Big conceptual point:** once you have a successor machine and an addition machine, you get multiplication (at least by a small constant, via repeated addition/duplication) essentially "for free" by *composing* machines, rather than designing new state diagrams from zero. Same compositional idea as the Church-Turing thesis discussion — complex computable functions are built from simpler computable building blocks.

---

## BIG PICTURE

The deck has two "modes" of Turing machine:

**Mode 1 — TM as language acceptor** (first ~18 slides):
- `{0^n1^2n}` → simple erase-based two-block matching, no marker needed.
- Balanced brackets → needs a **marker** (X), because matching happens in the interior, not just at the ends.
- `{0^n1^n2^n}` → needs matching between **three** blocks at once. Solved two ways:
  - (a) reduce to a problem already solved twice in a row (erase-based, fewer extra symbols, more states).
  - (b) interleave all three counts in one sweep (marker-based, more extra symbols, fewer states).
  - Culminates in demonstrating a language that is **not context-free** but **is** Turing-decidable — TMs are strictly stronger than PDAs.

**Mode 2 — TM as calculator** (last ~7 slides):
- Represent numbers in unary (`1^(n+1)`).
- Successor (add 1) is the trivial base case.
- Addition reuses the "fill gap + trim two" trick — exploiting that unary numbers only care about **count**, not position (a direct callback to the "markers vs. erasing, does position matter" theme from Mode 1).
- Multiplication by 2 reuses addition (duplicate, then add) — showing TMs **compose** like subroutines.
- All justified by the Church-Turing thesis: since a TM can express any computable function, it's expected that ordinary arithmetic falls out of a few simple machines wired together.

**Throughline:** every machine in this document — whether deciding a language or computing a function — uses the same primitive toolbox: read a symbol, optionally overwrite it (erase to Δ, or write a marker), move one cell L/R (or Stay), change state. Nothing more powerful is ever used — exactly the point of the Church-Turing thesis: this simple toolbox is (by the thesis) as powerful as any computing device could ever be.

---

## CHEAT SHEET

**Conventions:** arrow label = `oldSymbol newSymbol direction` (S=stay). Double circle = accept. Erase only safe at the *ends* of the surviving string; interior matches need a marker symbol (in Γ, not Σ) to keep "blank = outside input" true everywhere.

| Language | Technique | Γ | States |
|---|---|---|---|
| `{0^n1^2n}` | erase 1 zero (left) + erase last 2 ones (right), repeat | {0,1,Δ} | 6 (A start, F accept) |
| Balanced brackets | mark leftmost unmatched `)` and nearest unmatched `(` with X, repeat | {(,),X,Δ} | 5 (A start, E accept) |
| `{0^n1^n2^n}` (2-pass) | phase1: match 0↔2 (erase 0, mark 2 as X); phase2: reuse `{0^n1^n}` on 1's vs X's | {0,1,2,X,Δ} | 8 |
| `{0^n1^n2^n}` (1-sweep) | each sweep marks one 0→X, one 1→Y, one 2→Z; invariant Xᵏ0ⁿ⁻ᵏYᵏ1ⁿ⁻ᵏZᵏ2ⁿ⁻ᵏ | {0,1,2,X,Y,Z,Δ} | 7 |

- `{0^n1^n2^n}` is **not context-free** (PDA can compare only 2 quantities via one stack) but **is** TM-decidable.
- **Church-Turing thesis:** any function computable by any mechanical procedure is computable by some TM (unprovable thesis, never contradicted).
- **Unary encoding:** number n on tape = `1^(n+1)` (the "+1" keeps zero visually distinct from blank/no-input). Multiple inputs separated by one Δ.
- **Add 1:** walk right to blank, write one more 1, halt.
- **Add m+n:** tape `1^(m+1) Δ 1^(n+1)` → fill middle Δ with 1 (now m+n+2 ones) → delete any 2 ones from either end → `1^(m+n+1)`. Extends to k numbers by repeating pairwise.
- **Multiply by 2:** duplicate m with a Δ gap → `1^(m+1) Δ 1^(m+1)` → reuse addition machine → `2m`.

### Common exam mistakes
1. Using erasure where a marker is required (balanced brackets, or "1's vs X's" phase) — destroys the "used interior" vs. "outside input" boundary.
2. Forgetting the "+1" offset: a tape of k ones represents the number **k−1**, not k.
3. Forgetting a PDA *cannot* do `{0^n1^n2^n}` (mixing it up with `{0^n1^n}`, which a PDA handles fine).
4. Miscounting states/symbols when comparing the two `{0^n1^n2^n}` designs: **8 states/1 extra symbol** (2-pass) vs. **7 states/3 extra symbols** (1-sweep).
5. Misreading the arrow-label order (`old new direction`) — flips the entire trace of a simulation.

---

## PRACTICE SECTION

### Theory Questions

**Q1.** Why can't erasure be used to mark a matched bracket pair inside a string like `(()())`, but erasure IS safe for the `{0^n1^2n}` machine?

*A1.* In `{0^n1^2n}`, every erasure happens strictly at the current two **ends** of the surviving string, so the blank region only grows inward from the outside — never creating a gap that could be mistaken for "outside the input." In balanced brackets, a matched pair (like the inner `()` in `(()())`) can sit in the **middle** of the string. Erasing it there leaves a blank hole surrounded by non-blank symbols. Since "blank = outside the input" is the machine's only way to detect the edge, such a hole would be wrongly read as the string's end, corrupting all further scans. A marker (X) keeps every cell non-blank until the string truly ends, preserving that invariant everywhere.

**Q2.** Explain, in plain English, why `{0^n1^n2^n}` cannot be accepted by a PDA, but can be accepted by a TM.

*A2.* A PDA has one unbounded memory resource — its stack — accessed strictly LIFO. Checking `{0^n1^n2^n}` requires comparing **three** quantities. A stack can compare two (push on 0's, pop on 1's) — but by the time the third block (2's) arrives, the stack information from the first block is already consumed by the second comparison, with no way to reuse it. A TM has no such restriction: its tape can be scanned left and right arbitrarily many times, so it can do a 0-vs-2 comparison and a separate 1-vs-(leftover) comparison, exactly as both solutions in this document do.

**Q3.** What is the Church-Turing thesis, and why is it a "thesis" rather than a theorem?

*A3.* It states that any function computable by any mechanical/algorithmic procedure whatsoever can be computed by some Turing machine. It's a thesis (not a theorem) because "any mechanical procedure" is an informal notion, not a rigorously defined mathematical object — there's nothing to formally prove the claim against. It's accepted because every proposed model of "effective computation" ever studied (recursive functions, lambda calculus, register machines, real programming languages) has been shown exactly as powerful as Turing machines, with no counterexample found in ~90 years.

### Tracing Questions

**Q4.** Trace the `{0^n1^2n}` machine on input `"011"` step by step. Accept or reject?

*Work:* Tape `0 1 1`. A erases the 0 (→Δ), B scans right to the blank, C erases the rightmost 1, D erases the 2nd rightmost 1, E scans back left, A reads the resulting Δ and **ACCEPTS**. Correct: `011` = 0¹1², and 2×1=2. ✓.

**Q5.** Trace the second-sweep `{0^n1^n2^n}` machine on input `"012"` (n=1). Show the tape after 1 sweep and state what happens next.

*Work:* Start `0 1 2`. A marks the 0→X (`X 1 2`), B marks the 1→Y (`X Y 2`), C marks the 2→Z (`X Y Z`), D scans left, finds X, restarts at A. After 1 sweep: tape = `X Y Z` — matches the invariant `Xᵏ0ⁿ⁻ᵏYᵏ1ⁿ⁻ᵏZᵏ2ⁿ⁻ᵏ` with n=1,k=1. State A now reads `Y` (not `0`) → switches to verification phase (v): skip over Y, skip over Z, hit Δ → **ACCEPT**. Correct: `012` = 0¹1¹2¹.

