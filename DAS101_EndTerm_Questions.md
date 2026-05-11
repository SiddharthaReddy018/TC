# DAS 101 — End-Term Question Bank
### Priority-ordered by predicted appearance probability

**Professor's style signature:** Named schema → concrete scenario → sub-parts at increasing depth. Applied before theoretical. Paired concepts in same question. T/F always with one-line justification. Numericals always show-your-work multi-step.

---

## TIER 1 — CRITICAL (>85% probability)

---

### 1. Transactions & Serializability — 97% | 12–15 marks expected

---

**Q1.** Consider the following schedule **S** involving transactions T1, T2, T3:

```
T1: R(A)
T2: W(A)
T1: W(A)
T3: R(A)
T2: R(B)
T3: W(B)
T1: W(B)
```

**(a)** [2 marks] List ALL conflicting pairs of operations in S. For each pair, state whether it is RW, WR, or WW.

**(b)** [3 marks] Draw the precedence graph for S. Is S conflict-serializable? Write **Yes/No** and justify using the graph.

**(c)** [2 marks] If S is conflict-serializable, give one equivalent serial schedule. If not, explain in one sentence why no serial equivalent exists.

**(d)** [3 marks] For each of the following, write **True/False** with a one-line justification:
- i. Every conflict-serializable schedule is also view-serializable.
- ii. Every serial schedule is conflict-serializable.
- iii. A schedule with no conflicting pairs is always serializable.

---

**Q2.** Consider a bank transfer: T1 deducts ₹500 from account A, then T2 adds ₹500 to account B.

**(a)** [4 marks] For each ACID property, write: (i) what it guarantees in this transaction, and (ii) one concrete example of what goes wrong if that property is violated.

**(b)** [2 marks] T1 deducts ₹500 from A and crashes before T2 runs. After recovery, A is deducted but B is not updated. Which ACID property is violated? Name it and explain in one sentence.

**(c)** [2 marks] Is it possible for a conflict-serializable schedule to still produce incorrect results? Write **Yes/No** and explain in one line.

---

**Q3.** Consider the following schedule S2:

```
T1: R(A)
T2: W(A)
T3: W(A)
T1: W(A)
```

**(a)** [2 marks] Draw the precedence graph. Is S2 conflict-serializable? Write **Yes/No**.

**(b)** [3 marks] Check whether S2 is view-serializable. Show your check for all three conditions: initial reads, final writes, and reads-from relationships. Write **Yes/No** and justify.

**(c)** [1 mark] What does your answer to (a) and (b) together illustrate about the relationship between conflict-serializability and view-serializability?

---

**Q4.** Suppose you have four transactions T1, T2, T3, T4 and the precedence graph has the following edges: T1→T2, T2→T3, T3→T1, T2→T4.

**(a)** [1 mark] Is this schedule conflict-serializable? Write **Yes/No**.

**(b)** [2 marks] Identify the cycle. Which transactions are involved in it?

**(c)** [2 marks] If you were to abort one transaction to make the schedule serializable, which transaction would you choose and why? (Consider minimizing work lost.)

---

### 2. SQL Query Writing — 92% | 12–15 marks expected

**Schema for all SQL questions:**
```
Users(UID, name, age)
Videos(VID, title, length, UID)        -- UID is the uploader
Ratings(UID, VID, rating)              -- rating is 1–5
```

---

**Q1.** [8 marks] Write SQL queries for each of the following:

**(a)** [2 marks] Find the names of users who have rated at least 3 different videos.

**(b)** [2 marks] Find the titles of videos whose average rating is strictly greater than the overall average rating across all ratings in the table.

**(c)** [2 marks] Find names of users who have rated video VID='V3' but have **not** rated video VID='V1'. Use set operations (EXCEPT / MINUS).

**(d)** [2 marks] For each user, display their name, the number of videos they have uploaded, and their average rating received on those videos. Show only users who have uploaded at least 2 videos.

---

**Q2.** [6 marks] Write SQL queries for each of the following:

**(a)** [2 marks] Find pairs of users (U1, U2) who have both rated the same video. Display (U1.name, U2.name, VID). Ensure U1.UID < U2.UID to avoid duplicate pairs.

**(b)** [2 marks] Find names of users who have **never** rated any video that they themselves uploaded. Use NOT EXISTS.

**(c)** [2 marks] Find the title of the video that has been rated by the most number of distinct users. If there is a tie, display all tied videos.

---

**Q3.** [4 marks] For each of the following, write **True/False** with a one-line justification:

- i. `SELECT DISTINCT` and `GROUP BY` always produce the same result on a single-column query.
- ii. A query with `HAVING` can always be rewritten with only `WHERE` and no `GROUP BY`.
- iii. `NATURAL JOIN` automatically joins on all columns with the same name.
- iv. A subquery in the `WHERE` clause that uses `EXISTS` is always equivalent to one using `IN`.

---

**Q4.** Consider this SQL query:

```sql
SELECT U.name, AVG(R.rating)
FROM Users U, Ratings R
WHERE U.UID = R.UID
GROUP BY U.UID, U.name
HAVING AVG(R.rating) > 3.5;
```

**(a)** [2 marks] Write the equivalent Relational Algebra expression for this query (use the γ operator for aggregation).

**(b)** [1 mark] If you remove `U.name` from the GROUP BY clause but keep it in SELECT, what problem arises? Explain in one sentence.

---

### 3. Concurrency Control / 2PL / Locks — 90% | 10–14 marks expected

---

**Q1.** **(a)** [2 marks] Fill in the lock compatibility matrix. Write **Y** (granted) or **N** (must wait):

```
                  S-lock currently held    X-lock currently held
Request S-lock:           ?                        ?
Request X-lock:           ?                        ?
```

**(b)** [4 marks] Consider the following schedule. For each transaction, determine whether it follows the Two-Phase Locking (2PL) protocol. If it does not, identify the **first** violation:

```
T1: lock-S(A), R(A), lock-S(B), R(B), unlock(A), lock-X(C), W(C), unlock(B), unlock(C)
T2: lock-X(B), W(B), unlock(B), lock-S(A), R(A), unlock(A)
T3: lock-S(A), R(A), lock-X(A), W(A), unlock(A)
```

**(c)** [2 marks] Draw the wait-for graph for the schedule in (b). Is there a deadlock? Write **Yes/No** and justify using the graph.

---

**Q2.** **(a)** [3 marks] Explain the difference between basic 2PL and Strict 2PL. Why does basic 2PL alone not prevent cascading rollbacks? Give a concrete two-transaction example.

**(b)** [4 marks] For each of the following, write **True/False** with a one-line justification:
- i. A transaction following 2PL is guaranteed to produce a conflict-serializable schedule.
- ii. Strict 2PL releases shared locks before the transaction commits.
- iii. Intention locks (IS, IX) are used to support multi-granularity locking.
- iv. Deadlocks can be prevented entirely by always acquiring locks in a predefined global order.
- v. A wait-for graph cycle of length 2 always indicates a deadlock.

---

**Q3.** Consider two transactions:

```
T1: lock-X(A), W(A), lock-X(B), W(B), unlock(A), unlock(B)
T2: lock-X(B), W(B), lock-X(A), W(A), unlock(B), unlock(A)
```

**(a)** [2 marks] Does a deadlock occur when T1 and T2 run concurrently? Draw the wait-for graph to justify.

**(b)** [1 mark] Suggest one simple strategy to prevent this specific deadlock without aborting either transaction after it has started.

**(c)** [2 marks] Suppose we apply the **wound-wait** deadlock prevention scheme, with T1 having higher priority (older timestamp). What happens when T2 requests lock-X(A) while T1 holds it? What happens when T1 requests lock-X(B) while T2 holds it?

---

### 4. Recovery / WAL / Redo-Undo — 85% | 8–12 marks expected

---

**Q1.** **(a)** [3 marks] State the two rules of Write-Ahead Logging (WAL). For each rule, explain in one sentence what goes wrong during recovery if that rule is violated.

**(b)** [5 marks] Consider the following log. A checkpoint was taken when T1 was the only active transaction.

```
[T1, begin]
[T1, W(A), old=10, new=25]
[checkpoint: {T1 active}]
[T1, commit]
[T2, begin]
[T2, W(B), old=40, new=60]
[T3, begin]
[T3, W(C), old=5, new=20]
[T2, W(A), old=25, new=70]
[CRASH]
```

- i. [2 marks] Which transactions need to be **redone**? Which need to be **undone**? Justify step by step starting from the checkpoint.
- ii. [2 marks] What is the final value of A, B, and C after full recovery?
- iii. [1 mark] Is it possible for the same transaction to require both redo and undo operations? Write **Yes/No** and explain in one line.

**(c)** [2 marks] Explain how a checkpoint reduces recovery time. What does the recovery algorithm do differently when it finds a checkpoint record in the log?

---

**Q2.** **(a)** [3 marks] For each of the following, write **True/False** with a one-line justification:
- i. WAL guarantees that committed transactions are never lost after a system crash.
- ii. If a transaction has written a log record but not yet flushed the data page to disk, the data is considered safe after a crash.
- iii. Undo logging requires that data pages be flushed to disk before the transaction commits.
- iv. A fuzzy checkpoint allows transactions to continue running while the checkpoint is being written.

**(b)** [3 marks] Explain the difference between deferred updates (redo-only logging) and immediate updates (undo-redo logging). Under what failure scenario does each approach perform recovery differently?

---

**Q3.** Consider this log sequence, with no checkpoint:

```
[T1, begin]
[T2, begin]
[T1, W(X), old=5, new=10]
[T2, W(Y), old=20, new=30]
[T1, W(Y), old=30, new=50]
[T2, commit]
[CRASH]
```

**(a)** [2 marks] Using undo-redo recovery: which transactions are redone, which are undone?

**(b)** [2 marks] What are the final values of X and Y after recovery? Show your working.

**(c)** [1 mark] If T1 had also committed just before the crash (no log record for commit made it to disk), would your answer change? Write **Yes/No** and explain in one line.

---

## TIER 2 — HIGHLY LIKELY (65–85% probability)

---

### 5. SQL DDL + Integrity Constraints — 80% | 3–5 marks expected

---

**Q1.** **(a)** [4 marks] Write SQL DDL to create the following two tables with **all** appropriate constraints:

- `Users(UID, name, age)` — UID is primary key, name cannot be NULL, age must be between 0 and 120.
- `Ratings(UID, VID, rating)` — (UID, VID) together form the primary key, both UID and VID are foreign keys (referencing Users and Videos respectively), rating must be between 1 and 5.

**(b)** [2 marks] What is the difference between `ON DELETE CASCADE` and `ON DELETE RESTRICT` on the Ratings foreign key? Give one concrete scenario where each behaviour is the correct choice.

**(c)** [1 mark] Can you define a CHECK constraint that spans two columns (e.g., ensuring that end_date > start_date)? Write **Yes/No** and explain in one line.

---

**Q2.** The following DDL has a problem:

```sql
CREATE TABLE Ratings (
    UID    INT,
    VID    INT,
    rating INT CHECK (rating > 0),
    PRIMARY KEY (UID),
    FOREIGN KEY (UID) REFERENCES Users(UID)
);
```

**(a)** [2 marks] Identify **two** errors in this DDL. For each, explain why it is wrong and how to fix it.

**(b)** [1 mark] After fixing the errors, if a user is deleted from Users, what happens to their ratings if the foreign key uses `ON DELETE SET NULL`? Is this semantically valid given your primary key definition?

---

### 6. Join Algorithm Cost Numericals — 78% | 8–10 marks expected

---

**Q1.** Suppose relation R has 20,000 tuples stored in **200 pages** and relation S has 8,000 tuples stored in **80 pages**. The buffer pool has **B = 12 pages** available for this join.

**(a)** [3 marks] Compute the I/O cost of **Block Nested Loop Join (BNLJ)** when:
- i. R is the outer relation.
- ii. S is the outer relation.

Show all steps using the formula. Which choice is better and why?

**(b)** [2 marks] What is the minimum buffer size B needed so that the entire inner relation fits in memory during BNLJ? Answer separately for (i) R as inner, (ii) S as inner.

**(c)** [2 marks] Suppose S now has a **clustered B+ tree index** on the join attribute. Which join algorithm becomes most efficient? Give the I/O cost in asymptotic terms (in terms of pages of R and matching tuples per R-tuple from S).

**(d)** [2 marks] For each of the following, write **True/False** with a one-line justification:
- i. Sort-Merge Join is always preferred over Hash Join when the buffer is large.
- ii. If the output of the join must be sorted on the join attribute, Sort-Merge Join has an advantage over Hash Join.

---

**Q2.** **(a)** [2 marks] For Block Nested Loop Join with outer relation of P_R pages, inner of P_S pages, and B buffer pages, write the cost formula. Identify what each term represents.

**(b)** [2 marks] For the same R and S from Q1 (200 and 80 pages, B=12): compute the cost of **Sort-Merge Join**. Assume both relations are initially unsorted. Use the formula: Cost = 2·P_R·⌈log_{B-1}(⌈P_R/B⌉)⌉ + P_R + 2·P_S·⌈log_{B-1}(⌈P_S/B⌉)⌉ + P_S + (P_R + P_S).

**(c)** [1 mark] Under what condition would you prefer Sort-Merge Join over Hash Join even if both have similar I/O cost? Answer in one sentence.

---

### 7. Selectivity Estimation — 65% | 4–6 marks expected

---

**Q1.** Relation R has **50,000 tuples**. Attribute A has 200 distinct values (uniform distribution). Attribute B has 100 distinct values (uniform distribution). Attribute C is boolean with 50% true / 50% false.

**(a)** [3 marks] Estimate the number of tuples returned by each selection (show working):
- i. σ(A = 10)(R)
- ii. σ(A = 10 AND B = 5)(R)
- iii. σ(A = 10 AND C = true)(R)

**(b)** [1 mark] The actual result of σ(A = 10 AND B = 5)(R) is 600 tuples, but your estimate gave 2.5. Which assumption in your estimation is most likely wrong? Explain in one sentence.

**(c)** [2 marks] Suppose A and B are positively correlated (high values of A tend to co-occur with high values of B). Does your independence-based estimate in (a-ii) overestimate or underestimate the actual result? Write one word (overestimate / underestimate) and justify in one sentence.

---

**Q2.** **(a)** [2 marks] Given relations R1 (1000 tuples) and R2 (500 tuples), where the join attribute A has 50 distinct values in R1 and 40 distinct values in R2, estimate the size of R1 ⋈ R2 using the formula: |R1 ⋈ R2| = (|R1| × |R2|) / max(d_A in R1, d_A in R2).

**(b)** [1 mark] What does this formula assume about the data distribution? State one scenario where this assumption breaks badly.

---

## TIER 3 — MODERATELY LIKELY (35–65% probability)

---

### 8. Pipelining vs Materialization — 55% | 3–5 marks expected

---

**Q1.** A query plan (read bottom-up) looks like:

```
Hash Join
  ├── Seq Scan on Videos (filter: length > 60)
  └── Hash
       └── Index Scan on Ratings (filter: rating > 4)
```

**(a)** [2 marks] For each of the following operators, write **P** (can be pipelined) or **PB** (pipeline-breaker), with a one-line reason:
- i. Selection (σ)
- ii. Sort
- iii. Hash Join — build phase
- iv. Hash Join — probe phase
- v. Projection (π)

**(b)** [2 marks] In the plan above, identify which operator forces **materialization** and explain why materialization is unavoidable there.

**(c)** [1 mark] In the demand-driven (pull) execution model, which operator initiates tuple flow — the root or the leaves? Explain in one sentence.

---

**Q2.** **(a)** [2 marks] For each of the following, write **True/False** with a one-line justification:
- i. Pipelining always reduces total I/O cost compared to full materialization.
- ii. A sort operator must read its entire input before producing any output.
- iii. In producer-driven pipelining, operators push tuples downstream without waiting to be asked.
- iv. The iterator model (open/get_next/close) supports both pipelined and materialized execution.

---

### 9. Relational Algebra (Revisit) — 50% | 6–10 marks expected

**Schema:**
```
Employees(EID, name, dept, salary)
Departments(dept, manager_EID, budget)
```

---

**Q1.** [6 marks] Write Relational Algebra expressions for each of the following:

**(a)** [1 mark] Names of employees in the 'CS' department earning more than ₹80,000.

**(b)** [2 marks] Names of departments whose budget is greater than the average budget across all departments. *(Hint: use the γ operator and a cross product / division approach.)*

**(c)** [2 marks] Names of employees who are the manager of their own department (i.e., their EID appears as manager_EID in Departments for their own dept).

**(d)** [1 mark] Draw the RA tree for query (a). Label each node with the operator and relevant condition.

---

**Q2.** Consider this RA expression:
```
π_name(σ_salary > 50000(Employees ⋈ Departments))
```

**(a)** [1 mark] What does this query return in plain English?

**(b)** [2 marks] Rewrite this expression applying predicate pushdown. Show the before and after RA trees.

**(c)** [1 mark] Why does predicate pushdown improve performance? Answer in one sentence.

---

### 10. Normalization (Revisit) — 45% | 5–8 marks expected

---

**Q1.** Consider relation R(A, B, C, D, E) with functional dependencies:
```
AB → C
C  → D
D  → E
E  → A
```

**(a)** [3 marks] Find all candidate keys of R. Show the attribute closures you computed step by step.

**(b)** [2 marks] Is R in BCNF? If not, identify the first violating FD and perform **one** step of BCNF decomposition. State the two resulting relations and their FDs.

**(c)** [2 marks] Is R in 3NF? Write **Yes/No** and justify in two lines. (Recall: in 3NF, for every FD X→Y, either X is a superkey, or Y is a prime attribute.)

---

**Q2.** **(a)** [2 marks] For each of the following decompositions of R(A,B,C) with FD A→B, write whether it is **lossless** and **dependency-preserving**. Write Yes/No for each property:
- i. R1(A, B), R2(A, C)
- ii. R1(A, B), R2(B, C)
- iii. R1(A, C), R2(B, C)

**(b)** [2 marks] Is it always possible to decompose a relation into BCNF while preserving all dependencies? Write **Yes/No** and give a one-line explanation with a concrete example of a case where it fails.

---

### 11. Triggers — 45% | 2–4 marks expected

---

**Q1.** Consider this trigger on the Ratings table:

```sql
CREATE TRIGGER prevent_self_rating
BEFORE INSERT ON Ratings
FOR EACH ROW
BEGIN
  IF (SELECT UID FROM Videos WHERE VID = NEW.VID) = NEW.UID
  THEN SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Cannot rate your own video';
  END IF;
END;
```

**(a)** [1 mark] What is this trigger trying to do? Does it correctly achieve this goal? Write **Yes/No** and explain in one sentence.

**(b)** [2 marks] Identify one edge case where this trigger could fail or behave incorrectly. Describe the scenario and suggest a fix.

**(c)** [1 mark] If you changed `BEFORE INSERT` to `AFTER INSERT`, would this trigger still work correctly? Write **Yes/No** and explain in one sentence.

---

**Q2.** **(a)** [2 marks] A trigger is written that updates a `last_modified` timestamp on the Ratings table whenever a rating is updated. The trigger is defined as `AFTER UPDATE ON Ratings FOR EACH ROW`. A developer notices the trigger fires recursively. Explain why this happens and how to fix it.

**(b)** [1 mark] What is the difference between `FOR EACH ROW` and `FOR EACH STATEMENT` in trigger execution? Give a one-sentence example of when each is appropriate.

---

### 12. Query Plan Reading — 40% | 3–5 marks expected

---

**Q1.** A query plan shows the following (read bottom-up):

```
Merge Join (join: R.A = S.A)
  ├── Sort (on R.A)
  │    └── Seq Scan on R (filter: R.B > 10)
  └── Sort (on S.A)
       └── Index Scan on S (index on S.A)
```

**(a)** [2 marks] In what order are the operations executed? Describe the data flow from leaves to root.

**(b)** [1 mark] The filter `R.B > 10` is applied during the Seq Scan, not after the join. What optimization principle is this? Why is it beneficial?

**(c)** [1 mark] The index scan on S is performed even though a sort is needed afterward. Is it possible that a Seq Scan on S would be cheaper here? Write **Yes/No** and explain in one sentence.

**(d)** [1 mark] Identify which operator(s) in this plan are pipeline-breakers. Justify in one sentence.

---

## TIER 4 — LOW PROBABILITY BUT POSSIBLE (10–30%)

---

### 13. ER Diagram (Revisit) — 25% | 4–6 marks expected

---

**Q1.** Design an ER diagram for a hospital management system with the following requirements:

- Doctors have an employee ID, name, and specialization.
- Patients have a patient ID, name, and date of birth.
- A doctor can treat many patients; a patient can be treated by many doctors.
- Each treatment episode has a date and a diagnosis — these are attributes of the relationship.
- A doctor belongs to exactly one department; a department has many doctors.
- Each department has a unique department name and a head doctor (who is also a doctor in that department).

**(a)** [4 marks] Draw the ER diagram. Show all entities, attributes (underline keys), relationships, and cardinalities.

**(b)** [1 mark] Is the "Treatment" relationship a regular relationship or should it be modelled as an entity? Write your choice and justify in one sentence.

**(c)** [1 mark] The relationship between a Department and its head doctor creates a constraint that cannot be expressed in a basic ER diagram. What type of constraint is this and how would you handle it?

---

### 14. Selinger's Algorithm — 20% | 4–6 marks expected

---

**Q1.** You need to join three relations: R1 (100 pages), R2 (200 pages), R3 (50 pages). Assume:
- All joins use BNLJ with B = 10 buffer pages.
- Selectivity factor (fraction of tuples surviving each join) = 0.01.
- BNLJ cost formula: P_outer + ⌈P_outer/(B−2)⌉ × P_inner.

**(a)** [3 marks] Enumerate all distinct left-deep join orders for these three relations. For each, compute the total I/O cost. Show your working.

**(b)** [1 mark] Which join order is optimal according to Selinger's dynamic programming approach?

**(c)** [2 marks] For each of the following, write **True/False** with a one-line justification:
- i. Selinger's algorithm considers all possible bushy trees, not just left-deep plans.
- ii. The dynamic programming approach avoids recomputing costs for sub-plans.
- iii. Selinger's algorithm always finds the globally optimal join order.

---

### 15. Shadow Paging — 15% | 2–3 marks expected

---

**Q1.** **(a)** [3 marks] For each of the following, write **True/False** with a one-line justification:
- i. Shadow paging requires a Write-Ahead Log for recovery.
- ii. Shadow paging guarantees atomicity without needing undo log records.
- iii. Shadow paging is widely used in modern production RDBMS systems such as PostgreSQL or MySQL.
- iv. In shadow paging, a committed transaction's changes are made visible by a single atomic pointer swap.

**(b)** [1 mark] Give **one concrete reason** why WAL + undo/redo is preferred over shadow paging in production databases. Answer in one sentence.

---

## PROFESSOR'S STYLE QUICK REFERENCE

| Pattern | Example |
|---|---|
| Yes/No + one line | "Is this schedule conflict-serializable? Yes/No, justify in one line." |
| T/F + one line | "True/False: Every serial schedule is conflict-serializable. Justify." |
| Paired concepts | 2PL vs Strict 2PL always in same question |
| Schema anchored | All sub-parts within one question share the same schema |
| Multi-step numericals | Show formula → plug in → simplify → compare |
| "Is it possible" | Tests boundary conditions and exceptions |
| Worst-case / best-case | Appears in cost comparison questions |
| Algorithm tracing | Given a log / schedule / graph → trace step by step |

---

## HIGH-ROI STUDY ORDER (if ≤6 hours remain)

| Hour | Topic | Why |
|---|---|---|
| 1–2 | Transactions: precedence graph + ACID | 97% likely, fully mechanical, fast to learn |
| 2–3 | Concurrency: lock matrix + 2PL + wait-for graph | 90% likely, identical structure to precedence graph |
| 3–4 | Recovery: WAL rules + log trace undo/redo | 85% likely, step-by-step algorithmic |
| 4–5 | SQL: write 8–10 queries on Users/Videos/Ratings | 92% likely, practice makes perfect |
| 5–6 | Join cost: BNLJ formula + one full numerical | 78% likely, one formula to memorize |

**Do not skip under any circumstances:** precedence graph algorithm, lock compatibility matrix, WAL two rules, SQL GROUP BY + HAVING + nested subquery.
