# Full Dry Run — `DIV(x+1, 2y)`

Let's go very slowly and trace the machine exactly as if we were executing it by hand.

---

## 1. First understand the encoding

The machine uses **unary encoding**:

$$
n \longrightarrow 1^{n+1}
$$

So:

| Mathematical number | Tape encoding |
| ------------------: | ------------- |
|                   0 | `1`           |
|                   1 | `11`          |
|                   2 | `111`         |
|                   3 | `1111`        |
|                   4 | `11111`       |
|                   5 | `111111`      |
|                   6 | `1111111`     |

Notice the important point:

> Number \(n\) is represented by **\(n+1\) ones**, not \(n\) ones.

---

# Example 1: \(x=1,\ y=3\)

We want:

$$
DIV(x+1,2y)
$$

Substitute:

$$
x+1=1+1=2
$$

and

$$
2y=2(3)=6
$$

Therefore we are checking:

$$
DIV(2,6)
$$

Since

$$
2\mid6
$$

the answer should be **TRUE / ACCEPT**.

---

# PART A — Initial tape

The standard input encoding is:

$$
1^{x+1}\Delta1^{y+1}
$$

For \(x=1\):

$$
x+1=2
$$

so the first block is:

```text
11
```

For \(y=3\):

$$
y+1=4
$$

so the second block is:

```text
1111
```

Therefore the starting tape is:

```text
11 ∆ 1111
```

or visually:

```text
[1][1][∆][1][1][1][1]
 ↑
HEAD
```

The head starts at the leftmost cell.

---

# PART B — SETUP

The setup has to transform the input into:

```text
1^(x+2) ∆ 1^(2y+1)
```

Why?

Because we want:

* left block to encode \(D=x+1\)
* right block to encode \(N=2y\)

Remember:

$$
\text{encoding of }D = 1^{D+1}
$$

Since

$$
D=x+1
$$

we need:

$$
D+1=x+2
$$

ones.

Similarly:

$$
N=2y
$$

requires:

$$
N+1=2y+1
$$

ones.

---

## Setup Step 1 — Increase the D-block

Current tape:

```text
11 ∆ 1111
```

We currently have **2 ones** in the left block.

But \(D=2\) needs its standard encoding:

$$
1^{D+1}=1^3
$$

So we need **3 ones**.

The machine writes one additional `1` immediately to the **left** of the block.

Therefore:

```text
111 ∆ 1111
```

Now the left block contains:

```text
111
```

That's 3 ones.

And:

$$
3=D+1
$$

so it correctly represents:

$$
D=2
$$

### Important!

The leftmost `1` is the **spare encoding symbol**.

We can think of the block as:

```text
1 | 1 1
↑   ↑ ↑
spare actual D-units
```

So:

```text
111
```

represents:

$$
D=2
$$

not \(D=3\).

---

# Setup Step 2 — Double the y-block

The current right block is:

```text
1111
```

This is the encoding of \(y=3\), because:

$$
3+1=4
$$

But we need:

$$
N=2y=6
$$

The encoding of 6 requires:

$$
6+1=7
$$

ones.

So we need to transform the right block into:

```text
1111111
```

That is 7 ones.

The doubling procedure is conceptually:

```text
1111 ∆ 1111
```

Two copies of the original block.

Then the separator `∆` between them becomes a `1`.

So:

```text
1111 1 1111
```

which contains:

$$
4+1+4=9
$$

ones.

Then two ones are deleted:

$$
9-2=7
$$

giving:

```text
1111111
```

Therefore the tape after setup is:

```text
111 ∆ 1111111
```

This represents:

```text
D = 2
N = 6
```

---

# PART C — Step (i): Put the sentinel S

The pseudocode says:

> Find the right end of the N-block and replace its last `1` with `S`.

Why?

Because the machine needs to know:

> "I have reached the end of the N-block."

So the last `1` is changed to `S`.

Before:

```text
111 ∆ 1111111
```

After:

```text
111 ∆ 111111S
```

Let's separate the meanings:

```text
111 | ∆ | 111111 | S
 ↑       ↑          ↑
 D       N          boundary
```

There are **six real N-units**:

```text
111111
```

and the final `S` is the sentinel.

So:

```text
111 ∆ 111111 S
```

means:

$$
D=2,\qquad N=6
$$

---

# Step (i) — Return to the left

The machine now moves the head back to the leftmost cell.

Tape:

```text
111 ∆ 111111 S
↑
HEAD
```

Now the setup is finished.

---

# PART D — Step (ii): Check the remainder

This is a **very important step**.

The machine moves right:

```text
111 ∆ 111111 S
```

It passes:

```text
111
```

then:

```text
∆
```

and reaches the N-block.

The first surviving N-symbol is:

```text
1
```

So the machine sees:

```text
1
```

not `S`.

Therefore:

> N is not finished yet.

So we cannot accept.

The machine goes back to the beginning of the D-block and starts a subtraction round.

---

# ROUND 1

We want to subtract:

$$
D=2
$$

from:

$$
N=6
$$

So we need to consume **2 N symbols**.

---

## Step (iii) — First D-unit

The D-block is:

```text
111
```

Remember:

```text
1 | 11
↑   ↑
spare D-units
```

The first/leftmost cell is the spare and is not touched.

The machine reaches the first actual D-unit:

```text
1
```

It changes it to `X`.

So:

```text
111
```

becomes:

```text
1XX
```

Actually, at this moment only the first actual unit has been marked:

```text
1X1
```

So:

```text
Before:  1 1 1
             ↑
           D-cell

After:   1 X 1
```

Why `X`?

Because this D-unit has been used **temporarily** in the current subtraction round.

We cannot erase it because we'll need it again in the next round.

---

# Step (iv) — Find an N-unit

Now the machine moves right past the D-block and separator.

It reaches:

```text
111111S
```

The first available N-unit is:

```text
1
```

So it changes that `1` to `∆`.

Before:

```text
111111S
```

After:

```text
∆11111S
```

We have consumed one N-unit.

So:

$$
6\rightarrow5
$$

---

# Step (v) — Return to the D-block

The machine moves left until it finds the `X` it just created.

Tape:

```text
1X1 ∆ ∆11111S
```

It finds:

```text
X
```

Then it moves one position to the right.

That takes it to the next D-cell:

```text
1 X 1
    ↑
   HEAD
```

---

# Step (iii) — Second D-unit

The current D-cell is:

```text
1
```

So it hasn't been used yet.

The machine changes:

```text
1 → X
```

Therefore:

```text
1X1
```

becomes:

```text
1XX
```

---

# Step (iv) — Consume another N-unit

The machine goes right to the N-block.

Current N portion:

```text
∆11111S
```

The first surviving `1` is found.

It is erased:

```text
∆∆1111S
```

So one more N-unit has been consumed.

Now:

$$
5\rightarrow4
$$

Therefore after Round 1:

$$
6-2=4
$$

---

# Step (v) — Return to D

The machine finds the second `X`.

Tape:

```text
1XX ∆ ∆∆1111S
```

All actual D-units have now been marked:

```text
1 X X
  ↑ ↑
  both D-units used
```

The machine moves to the next D position.

But there is no more actual D-unit.

It reaches:

```text
∆
```

---

# Step (iii) — D-block finished for this round

The pseudocode says:

> If current symbol is `∆`, every non-sentinel D-cell has been marked `X`.

Exactly what happened.

Current D-block:

```text
1XX
```

So the machine knows:

> I have used all 2 units of D.

Therefore Round 1 is complete.

---

# Restore the D-block

The `X`s were temporary.

So:

```text
1XX
```

is restored to:

```text
111
```

Why?

Because we need to use \(D=2\) again.

This is the key reason we **marked** D with `X` rather than erasing it.

---

# Check N again

Now the machine moves to the N-block.

Current tape conceptually:

```text
111 ∆ ∆∆1111S
```

The surviving N symbols are:

```text
1111
```

There are 4 left.

The machine sees:

```text
1
```

not `S`.

Therefore:

> N still has something left.

So start another subtraction round.

---

# ROUND 2

Current amount of N:

$$
4
$$

D:

$$
2
$$

We need:

$$
4-2=2
$$

---

## Step (iii) — First D-unit

D-block:

```text
111
```

First actual D-unit:

```text
1
```

Change it to `X`.

```text
1X1
```

---

## Step (iv) — Consume N-unit

Remaining N:

```text
1111S
```

Consume first one:

```text
∆111S
```

So:

$$
4\rightarrow3
$$

---

## Step (v)

Return to the `X`.

Move one position right.

We reach the second actual D-unit.

---

## Step (iii) — Second D-unit

Change:

```text
1 → X
```

So:

```text
1XX
```

---

## Step (iv) — Consume another N-unit

Remaining N:

```text
∆111S
```

Consume one:

```text
∆∆11S
```

Now:

$$
3\rightarrow2
$$

So Round 2 has consumed exactly 2 N-units.

---

# Finish Round 2

All D-units are now marked:

```text
1XX
```

Therefore restore:

```text
1XX → 111
```

The N-block now has:

```text
11S
```

So there are:

$$
2
$$

N-units remaining.

---

# ROUND 3

Again step (ii) checks the remainder.

It sees:

```text
1
```

rather than `S`.

Therefore:

> N is not empty.

Start another round.

---

## First D-unit

```text
111
```

becomes:

```text
1X1
```

Consume one N-unit:

```text
11S → ∆1S
```

So:

$$
2\rightarrow1
$$

---

## Second D-unit

Mark it:

```text
1X1 → 1XX
```

Consume the last N-unit:

```text
∆1S → ∆∆S
```

Now:

$$
1\rightarrow0
$$

---

# Finish Round 3

All D-units have been marked:

```text
1XX
```

Restore:

```text
111
```

Now look at the N-block.

It is:

```text
∆∆S
```

The **first surviving symbol** is:

```text
S
```

---

# Step (ii) — The crucial ACCEPT

The machine checks:

> Is the first surviving N-symbol `S`?

Yes.

```text
S
↑
HEAD
```

Therefore:

### ACCEPT

Why?

Because we consumed the N-block in complete groups of D.

The sequence was:

```text
6
↓ subtract 2
4
↓ subtract 2
2
↓ subtract 2
0
```

There was never a partial round.

Therefore:

$$
6=2+2+2
$$

and hence:

$$
2\mid6
$$

---

# Step (vi) — Write TRUE

The machine erases the entire tape.

Then it writes:

```text
11
```

Why `11`?

Because:

$$
1\rightarrow1^{1+1}=11
$$

And `1` represents **TRUE**.

So final tape:

```text
11
```

### Final answer:

$$
\boxed{DIV(2,6)=TRUE}
$$

---

# Complete Example 1 in One Table

| Stage    | N remaining | D-block | Action             |
| -------- | ----------: | ------- | ------------------ |
| Setup    |           6 | `111`   | Create \(D=2,N=6\) |
| Sentinel |           6 | `111`   | `111 ∆ 111111S`    |
| Round 1  |       6 → 5 | `1X1`   | First D-unit used  |
| Round 1  |       5 → 4 | `1XX`   | Second D-unit used |
| Restore  |           4 | `111`   | Reset D            |
| Round 2  |       4 → 3 | `1X1`   | First D-unit used  |
| Round 2  |       3 → 2 | `1XX`   | Second D-unit used |
| Restore  |           2 | `111`   | Reset D            |
| Round 3  |       2 → 1 | `1X1`   | First D-unit used  |
| Round 3  |       1 → 0 | `1XX`   | Second D-unit used |
| Restore  |           0 | `111`   | Reset D            |
| Check    |           0 | `111`   | Find `S`           |
| Final    |           — | `11`    | **ACCEPT / TRUE**  |

---

# Example 2: \(x=3,\ y=3\)

Now let's do the **REJECT** case in exactly the same way.

We calculate:

$$
D=x+1=3+1=4
$$

and:

$$
N=2y=2(3)=6
$$

So we're checking:

$$
DIV(4,6)
$$

Obviously:

$$
4\nmid6
$$

So we expect **FALSE / REJECT**.

---

# PART A — Initial tape

For \(x=3\):

$$
x+1=4
$$

so input encoding is:

```text
1111
```

For \(y=3\):

$$
y+1=4
$$

so:

```text
1111
```

Initial tape:

```text
1111 ∆ 1111
```

---

# PART B — Setup

We need:

$$
D=x+1=4
$$

The encoding of 4 requires:

$$
4+1=5
$$

ones.

So add one `1` to the left:

```text
11111 ∆ 1111
```

The left block contains 5 ones.

We interpret it as:

```text
1 | 1111
↑     ↑
spare actual D=4 units
```

---

# Double the y-block

Two copies:

```text
1111 ∆ 1111
```

Turn the middle separator into `1`:

```text
1111 1 1111
```

There are:

$$
4+1+4=9
$$

ones.

Delete two:

$$
9-2=7
$$

So we obtain:

```text
1111111
```

Thus:

```text
11111 ∆ 1111111
```

represents:

$$
D=4,\quad N=6
$$

---

# Step (i) — Sentinel

Replace the last N `1` with `S`.

```text
11111 ∆ 111111S
```

There are six real N-units:

```text
111111
```

plus:

```text
S
```

---

# Step (ii) — First remainder check

The machine goes to the N-block.

It sees:

```text
1
```

not `S`.

Therefore:

> N is not empty.

Begin Round 1.

---

# ROUND 1

We need to consume:

$$
D=4
$$

N currently contains:

$$
6
$$

So we need four successful matches.

---

## First D-unit

D:

```text
1 | 1111
```

Mark first actual D-unit:

```text
1X111
```

Consume one N:

```text
111111S
↓
∆11111S
```

So:

$$
6\rightarrow5
$$

---

## Second D-unit

Mark second:

```text
1XX111
```

Consume another N:

```text
∆∆1111S
```

So:

$$
5\rightarrow4
$$

---

## Third D-unit

Mark:

```text
1XXX11
```

Consume N:

```text
∆∆∆111S
```

So:

$$
4\rightarrow3
$$

---

## Fourth D-unit

Mark:

```text
1XXXX
```

Consume N:

```text
∆∆∆∆11S
```

So:

$$
3\rightarrow2
$$

---

# Finish Round 1

All four actual D-units have now been marked:

```text
1XXXX
```

Restore them:

```text
1XXXX → 11111
```

The remaining N is:

```text
11S
```

Therefore:

$$
N_{\text{remaining}}=2
$$

---

# Step (ii) — Check again

The machine checks the first surviving N-symbol.

It finds:

```text
1
```

not `S`.

So it starts Round 2.

---

# ROUND 2

This is where the important thing happens.

We need:

$$
D=4
$$

N has only:

$$
2
$$

units remaining.

So we can consume two, but then N will run out **before we finish using all four D-units**.

Let's see it happen.

---

## First D-unit

D:

```text
11111
```

Actual D-units:

```text
1 1 1 1
```

Mark first:

```text
1X111
```

N:

```text
11S
```

Consume first:

```text
∆1S
```

N remaining:

$$
2\rightarrow1
$$

---

## Second D-unit

Mark second:

```text
1XX11
```

N:

```text
∆1S
```

Consume the last real N-unit:

```text
∆∆S
```

Now:

$$
1\rightarrow0
$$

---

# Third D-unit — THE CRITICAL MOMENT

The machine has NOT finished the D-block.

Remember:

$$
D=4
$$

So it still needs:

```text
D-unit 3
D-unit 4
```

It marks the third D-unit:

```text
1XXX11
```

Now it goes to the N-block looking for another surviving `1`.

But N looks like:

```text
∆∆S
```

There are **no real 1s left**.

The first surviving symbol is:

```text
S
```

---

# Step (iv) — REJECT

The pseudocode says:

> If the machine finds `S` while it is in the middle of a D-round, REJECT.

And that is exactly what happened.

Why?

Because we needed:

$$
4
$$

N-units to complete one subtraction of \(D=4\), but only:

$$
2
$$

were available.

So mathematically:

$$
6-4=2
$$

and then:

$$
2<4
$$

There isn't enough N left to subtract another complete 4.

Therefore 6 is **not** divisible by 4.

---

# Step (vii) — Write FALSE

The machine erases the entire tape.

Then writes:

```text
1
```

Why?

Because:

```text
1
```

is the standard unary encoding of:

$$
0
$$

And `0` is being used to represent **FALSE**.

Therefore:

```text
Final tape = 1
```

and:

$$
\boxed{DIV(4,6)=FALSE}
$$

---

# Complete Example 2 in One Table

| Stage      | N remaining | D status | What happens                      |
| ---------- | ----------: | -------- | --------------------------------- |
| Setup      |           6 | `11111`  | Create \(D=4,N=6\)                |
| Sentinel   |           6 | `11111`  | Add `S`                           |
| Round 1    |       6 → 5 | `1X111`  | D-unit 1                          |
| Round 1    |       5 → 4 | `1XX11`  | D-unit 2                          |
| Round 1    |       4 → 3 | `1XXX1`  | D-unit 3                          |
| Round 1    |       3 → 2 | `1XXXX`  | D-unit 4                          |
| Restore    |           2 | `11111`  | Reset D                           |
| Round 2    |       2 → 1 | `1X111`  | D-unit 1                          |
| Round 2    |       1 → 0 | `1XX11`  | D-unit 2                          |
| Round 2    |           0 | `1XXX1`  | D-unit 3 attempted                |
| **REJECT** |           0 | `1XXX1`  | Finds `S` before D-round finishes |
| Final      |           — | —        | `1` = FALSE                       |

---

# Now the most important part: WHY `X` and `S`?

This is very likely to be an **exam question**.

There are two different kinds of "used" information.

### `X` = temporarily used D-unit

Suppose:

```text
1XXXX
```

The four `X`s mean:

> I have used all four D-units **in this current round**.

After the round:

```text
1XXXX → 11111
```

So `X` is temporary.

---

### `S` = permanent boundary of N

Suppose:

```text
∆∆11S
```

The `∆` symbols mean N-units have already been consumed.

The `S` means:

> There are no more real N-units after this point.

`S` is never changed back into `1`.

So:

| Symbol | Meaning                      | Temporary? |
| ------ | ---------------------------- | ---------- |
| `1`    | unused unit                  | —          |
| `X`    | D-unit used in current round | **Yes**    |
| `∆`    | blank / consumed N-cell      | **No**     |
| `S`    | end of N-block               | **No**     |

---

# The two situations involving `S`

This is another **very important exam distinction**.

## Situation 1 — Find `S` during Step (ii)

Suppose the machine checks N **before starting a new round**:

```text
∆∆∆∆S
```

It sees `S`.

That means:

> N is completely consumed.

Therefore:

$$
\boxed{\text{ACCEPT}}
$$

Example:

```text
D = 2
N = 6

6 → 4 → 2 → 0
```

Perfect division.

---

## Situation 2 — Find `S` during Step (iv)

Now suppose we're in the middle of a D-round.

For example:

```text
D = 4
N remaining = 2
```

We have:

```text
D-unit 1 → consumed N
D-unit 2 → consumed N
D-unit 3 → need N
```

But N is already:

```text
∆∆S
```

So we find `S`.

This means:

> N ran out before one complete D-group could be matched.

Therefore:

$$
\boxed{\text{REJECT}}
$$

---

# Why Step (ii) MUST come before Step (iii)

This is the clever \(y=0\) case.

Suppose:

$$
y=0
$$

Then:

$$
N=2y=0
$$

The encoding of 0 is:

```text
1
```

During setup, that final spare `1` becomes the sentinel:

```text
S
```

So the tape becomes something like:

```text
D-block ∆ S
```

Now Step (ii) checks N **before marking any D-unit**.

It sees:

```text
S
```

immediately.

Therefore:

$$
\boxed{\text{ACCEPT}}
$$

And that's mathematically correct because:

$$
D\mid0
$$

for every positive \(D\).

For example:

$$
2\mid0
$$

because:

$$
0=2\times0
$$

So there are **zero subtraction rounds**.

This is why the order:

```text
(ii) Check N
     ↓
(iii) Start D-round
```

is important.

If the machine tried to mark D first, it could incorrectly treat \(N=0\) as a failed subtraction.

---

# The entire algorithm in simple English

Forget the formal TM language for a moment.

The algorithm is basically doing this:

```text
Take D and N.

WHILE N is not zero:

    Try to subtract D from N.

    For every 1 in D:
        consume one 1 from N.

    If N runs out before all D's 1s are matched:
        REJECT

    Restore D so it can be reused.

If N becomes exactly zero:
    ACCEPT
```

That is literally **repeated subtraction**.

---

# Visualizing the two examples

## Example 1

$$
D=2,\quad N=6
$$

Think of N as six balls:

```text
● ● ● ● ● ●
```

Take groups of 2:

```text
● ● | ● ● | ● ●
```

Exactly three complete groups.

Therefore:

$$
6=2\times3
$$

→ **ACCEPT**

---

## Example 2

$$
D=4,\quad N=6
$$

Take groups of 4:

```text
● ● ● ● | ● ●
```

The first group works.

But the remaining two are not enough to make another group of four:

```text
● ●
```

Therefore:

$$
6=4+2
$$

There is a remainder of 2.

→ **REJECT**

---

# One subtle point about the tape

When you see:

```text
111
```

in the D-block for Example 1, **do not say "D=3."**

Instead say:

```text
1 | 11
↑   ↑
spare D-units
```

The first `1` is the encoding overhead.

Therefore:

$$
3-1=2
$$

so:

$$
D=2
$$

Likewise, when you see:

```text
11111
```

for Example 2:

```text
1 | 1111
```

there are 5 ones, but:

$$
5-1=4
$$

so:

$$
D=4
$$

This distinction is essential.

---

# Final exam-style summary

For

$$
DIV(x+1,2y)
$$

the machine first transforms

$$
1^{x+1}\Delta1^{y+1}
$$

into

$$
1^{x+2}\Delta1^{2y+1}.
$$

The left block encodes

$$
D=x+1
$$

and the right block encodes

$$
N=2y.
$$

The last symbol of N is changed to `S`, giving the machine a permanent boundary.

Then the machine repeatedly:

1. **Checks N first.**

   * If it sees `S`, N is exactly exhausted → **ACCEPT**.
2. Marks each real D-unit with `X`.
3. For every marked D-unit, consumes one real N-unit.
4. Restores all `X`s to `1`s after a complete round.
5. If N reaches `S` **before the current D-round is complete**, → **REJECT**.
6. On ACCEPT, writes `11` = TRUE.
7. On REJECT, writes `1` = FALSE.

So:

$$
\boxed{x=1,y=3\Rightarrow DIV(2,6)=TRUE}
$$

and

$$
\boxed{x=3,y=3\Rightarrow DIV(4,6)=FALSE}
$$

The core idea to remember for the exam is:

> **`X` lets D be reused; `S` tells us that N has run out. Complete D-round + N exactly empty = ACCEPT. N runs out halfway through D-round = REJECT.**
