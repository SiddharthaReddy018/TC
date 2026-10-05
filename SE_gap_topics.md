# SE Study Guide — Part 2: Remaining Topics

Covers everything flagged as missing from `SE_midsem_study_guide.md`. Sections 1-5 are worked examples straight from `requirement-modelling-modelling-methods.pdf` (depth gaps). Section 6 is the statechart "Semantics" topic list — those slides are **title-only, no diagrams in the deck**, so section 6 is my own explanation of the standard Harel-statechart semantics these titles refer to, not slide content. Section 7 is a minor activity note from `SE_annotated.pdf`.

---

## 1. Use Case Diagram — Notation + Worked Examples

Notation:

```
  Actor            System boundary box
  (stick figure)   +------------------------+
    O              |                         |
   /|\   ---------  (  Use Case Oval  )       |
   / \              |                         |
                    +------------------------+
```

- Actor: outside the box, a stick figure, labelled with a role (not a person's name).
- Use case: an oval inside the system boundary box, named as a verb phrase ("Raise Complaint").
- Association line: actor to use case, plain line, no arrowhead.
- `<<extend>>`: a dashed arrow from the extending use case to the base use case, meaning "this use case optionally adds behaviour to the base, under a condition." Example: "Through Mobile App" extends "Raise Complaint" — raising a complaint can optionally happen via that channel.
- `<<include>>` (standard UML, not shown in this deck but worth knowing): a dashed arrow from base to included use case, meaning "the base use case always invokes this one." Different from extend: include is mandatory/always, extend is optional/conditional.

### Example 1 — Plumber (simple)

```
            +-------------------------+
            |                         |
  Plumber --( Enter Service Report )  |
    O       |                         |
   /|\      |                         |
   / \   ---( Accept Service Request )|
            |                         |
            +-------------------------+
```

### Example 2 — Service Manager added

```
  Service   +-------------------------+
  Manager --(   Assign Plumber     )  |
    O       |                         |
   /|\   ---(  Process Escalation  )  |
   / \      +-------------------------+
```

### Example 3 — Customer, with `<<extend>>`

```
                        +----------------------------+
                        |                             |
                        |   ( Through Mobile App )    |
                        |         \  extend           |
                        |          v                  |
  Customer -----------> (   Raise Complaint    ) <----( Through Web App )
    O                   |          ^                  |
   /|\                  |         / extend             |
   / \                  |   ( Through Phone Call )     |
                        |                              |
                        |     ( Respond To Service )   |
                        |        ^            ^        |
                        |   extend|       extend|       |
                        |    ( Accept )     ( Reject )  |
                        +------------------------------+
```

Reading it: "Raise Complaint" can happen through three channels (mobile/web/phone), each modelled as a use case that extends the base. "Respond To Service" similarly splits into "Accept" and "Reject" outcomes.

**How to use this on an exam**: when a scenario names an actor and some optional/channel-specific variants of one action, draw the base use case once and extend it — don't draw three separate unconnected use cases for the same action.

---

## 2. Class Diagram — Worked Example (Academic System)

This is the deck's running example. Two class trees, linked by an association.

### Person hierarchy

```
                 Person
                 (1..N) author (1..N)
                    |------------------------ Publication --(0..N)--1--Platform
                    |  (filled circle = inheritance split)
        +-----------+-----------+
        |           |           |
     Faculty     Student     External

  Faculty --supervisor(1) : student(0..N)--> Student     [one faculty supervises many students]
  Faculty --cosupervisor(0..N) : student(0..N)--> Student [many-to-many co-supervision]

  Student --student(0..N) : 1--> Programme   [many students, one programme each]

        Programme
           |
   +----+----+----+----+
   |    |    |    |    |
 iMTech PhD MTech MS/R MScDT
```

### Publication / Platform hierarchy

```
  Publication --(0..N)----1-- Platform

        Platform
           |
   +-------+-------+
   |       |        |
Conference Journal Workshop

  Conference --conference(1) : workshop(0..N)--> Workshop
  (a conference can have 0..N co-located workshops)
```

Reading the whole thing: a `Person` is either `Faculty`, `Student`, or `External` (inheritance — the filled dot is the deck's notation for an inheritance split, same idea as the hollow triangle used elsewhere). A `Person` authors 1..N `Publication`s and a `Publication` has 1..N authors. Each `Publication` is on exactly one `Platform`, and a `Platform` is a `Conference`, `Journal`, or `Workshop`. A `Student` belongs to one `Programme`, and `Programme` splits into the degree types.

### Class detail box — Course

```
+------------------+
|      Course       |
+------------------+
| Name              |
| Code              |
| Instructor        |
| Offerings         |
| Specialisation    |
+------------------+
| offer()           |
| revoke()          |
+------------------+
```

A plain example of a fully specified class: attributes above the line, operations below. Note `Offerings` and `Specialisation` are themselves attributes here (not broken into separate classes) — a reminder that not every noun in a description needs its own class; only promote a noun to a class if it has its own identity, attributes, or relationships.

**Exam takeaway**: this example is the template for Q1-style questions — draw the inheritance split first, then attach associations with roles and multiplicities on each end, then add one fully-specified class box if asked for "class details".

---

## 3. State Diagram — Ticket Lifecycle (separate from the washing machine example)

A second, much simpler worked example from the same deck:

```
                 raised by          Plumber
                 customer           assigned
  +-----------+  -------->  +--------+  -------->  +----------+  ------>  +----------+
  | NOT RAISED|              |  RAISED |              | ASSIGNED |           | RESOLVED |
  +-----------+              +--------+  <--------  +----------+           +----------+
                                   ^        unsatisfactory
                                   |________________________|
                                       (back to RAISED)
```

- `NOT RAISED -> RAISED`: triggered by "raised by customer".
- `RAISED -> ASSIGNED`: triggered by "Plumber assigned".
- `ASSIGNED -> RESOLVED`: triggered by "satisfactory" outcome (implied).
- `ASSIGNED -> RAISED`: triggered by "unsatisfactory" — the ticket gets reopened/reassigned instead of closing.

This is a flat (non-hierarchical, non-concurrent) state diagram — good contrast with the washing machine's hierarchical+concurrent one. If an exam question gives a simple linear process with one "send it back" loop, this is the pattern: don't force hierarchy or concurrency where the system doesn't have it.

---

## 4. Sequence Diagram — Complaint Resolution (actual worked example)

Participants: **Customer**, **Plumber**, **Can-Do-IT** (the system/company).

### Version 1 (simple path)

```
Customer         Plumber         Can-Do-IT
   |--raiseComplaint()-------------->|
   |                 |<--assign()----|
   |                 |--accept()---->|
   |<--visit()-------|               |
   |<--examineIssue()|               |
   |<--sendQuotation()---------------|
   |--accept()----------------------->|   (generates invoice)
   |<--solicitFeedback()--------------|
   |--recordService()---------------->|
   |--accept()------------------------>|
   |<--raiseInvoice()------------------|
   |--Pay()---------------------------->|
```

### Version 2 (adds a reject/reschedule branch)

Same flow, but after the quotation/visit step the deck adds an `alt`-style branch for when the customer does not simply accept:

```
   ...--accept()----------------->|  --> Generate invoice
alt [customer rejects / wants reschedule]
   |--reject()-------------------->|
   |<--reschedule()-----------------|
   |<--solicit[plumber] approval----|
   |<--suggestAppointment()---------|
   |--accept()--------------------->|
   |<--inform()----------------------|
end
   |<--visit()----------------------|
   |--recordService()--------------->|
   |--solicitFeedback()------------->|
   |--accept()------------------------>|
   |<--raiseInvoice()------------------|
   |--Pay()------------------------>|
```

This is the deck's own answer to "what happens if the customer doesn't accept the quotation" — which is almost exactly midsem Q2(b)(ii). The pattern to copy: wrap the rejection path in an `alt`, and show the system looping back to a new `suggestAppointment()` / re-accept cycle rather than just dead-ending.

**Exam takeaway**: your Q2 answer should structure the "customer doesn't accept" cases exactly like this — an `alt` fragment that reschedules/re-offers rather than silently failing.

---

## 5. DFD Level 1 — Decomposition Example

Level 0 (context diagram, single process):

```
 User  --PIN, service request-->  +------------------+   --Cash-->  Cash
 ATM card --user data-->          |  Cash Withdrawal  |
                                   +------------------+
```

Level 1 (the single process "Cash Withdrawal" is broken into sub-processes):

```
 User  --PIN, service request-->
 ATM card --user data------------>  +------------+     +----------------------+     +---------------+
                                     | Verify Data| --> | Check Cash           | --> | Dispense Cash |--Cash-->
                                     +------------+     | Availability         |     +---------------+
                                                         +----------------------+
```

Rule being demonstrated: a Level 1 diagram never introduces new external entities or new overall inputs/outputs — it only opens up the single Level 0 process into an internal pipeline of sub-processes, each still following "data must be transformed, not just moved." Here: verify the PIN/account, then check the account has enough balance, then physically dispense the cash.

**Exam takeaway**: if a DFD question asks for more than one level, draw Level 0 first (one bubble, all external entities), then redraw just that one bubble as 2-4 sub-processes with data flowing between them in order — don't add new entities at Level 1.

---

## 6. Statechart "Semantics" — Not in the Slides, Explained From General Knowledge

The deck has six slides titled only: Basic Transition, Interlevel Transition, Conflicting Transitions – Simple, Conflicting Transitions – Advanced, Unreachable State, Unescapable State. **No diagrams or body text exist on any of these six pages** — they're section headers with nothing under them, most likely meant to be drawn live on a whiteboard in class. Since you haven't been to class, here's what each term means in standard Harel-statechart semantics (the formalism this course is using, based on the washing machine example):

### 6.1 Basic Transition
The simplest kind: `State A --event[guard]/action--> State B`. It fires when the object is in A, the named event occurs, and the guard (a boolean condition in `[ ]`) is true. The object then leaves A (running A's `exit` action if any), runs the transition's own action, enters B (running B's `entry` action if any). If there's no guard, it's implicitly always true.

### 6.2 Interlevel Transition
A transition that crosses the boundary of a composite (hierarchical) state — it starts inside one composite state and ends outside it, or vice versa, rather than staying within one level. Example from the washing machine: `pause` from any substate of WASHING straight to PAUSE (which sits outside the WASHING composite) is an interlevel transition — it doesn't matter whether you were in SOAK, AGITATE, RINSE, or SPIN, the transition exits the whole composite at once. This is a key feature of statecharts vs plain finite-state machines: you can write one transition out of a composite instead of repeating it from every substate.

### 6.3 Conflicting Transitions — Simple
Two transitions leave the *same* state on the *same* event (and overlapping/no guards), so it's ambiguous which one should fire. Example: `IDLE --start--> WASHING` and `IDLE --start--> PAUSE` both defined with no distinguishing guard. This is a modelling bug (nondeterminism) — fix it by adding mutually exclusive guards (`[door_closed]` vs `[door_open]`) or by removing the duplicate.

### 6.4 Conflicting Transitions — Advanced
The conflict is between transitions at *different* levels of the hierarchy rather than literally the same state. Example: an outer transition `ON --error--> OFF` (defined at the top level, fires from anywhere inside ON) conflicts with an inner transition `WASHING --error--> PAUSED` (defined only inside the WASHING substate) when both are enabled by the same event. The standard resolution rule (used in UML/Harel statecharts) is: **the more specific (inner/deeper) transition has priority** over the more general (outer) one — the system assumes you meant the specific handling you wrote for that particular substate, and only falls back to the outer transition when no inner one applies.

### 6.5 Unreachable State
A state with no path leading to it from the initial state, no matter what sequence of events occurs — so the system can never actually be in it. This is always a design bug (dead code, essentially). Example: if you define a `SUPER_RINSE` state but no transition anywhere in the diagram actually targets it, it's unreachable; typically caused by forgetting to wire up a transition into a state after adding it.

### 6.6 Unescapable State ("trap" state)
A state with no outgoing transition at all (or none that are ever enabled), so once entered, the system is stuck there forever. This is a bug *unless* the state is deliberately meant to be a final/terminal state (e.g., `RESOLVED` or `COMPLETED` legitimately having no way out). The usual fix is to check whether every non-final state has at least one event that can fire out of it, including error/reset events — a common real cause is an error state with no "reset" transition back to normal operation.

**Why this matters for the midsem-style Q3**: part (f) of the midsem's statechart question ("how will you modify the machine to handle error conditions e.g. door open, no water supply, rotor jammed?") is directly testing 6.3-6.6. A good answer: add one `ERROR` state, with an interlevel transition into it from anywhere inside `ON` (6.2) guarded by the specific fault condition so there's no ambiguity about which fault triggered it (avoiding 6.3/6.4), make sure every substate that can enter `ON` has that path to `ERROR` so it's reachable (avoiding 6.5), and give `ERROR` an explicit `reset` transition back to `IDLE` so it isn't a trap (avoiding 6.6).

---

## 7. Minor Activity Note (SE_annotated.pdf)

A class-activity slide not otherwise mentioned: teams of 10, with roles split as Business Analyst (x2), Technical Architect (x4), UI/UX Experts (x4). This is a role-assignment exercise for a group project, not exam content — included here only for completeness.

---

## Where This Leaves You

Combined with `SE_midsem_study_guide.md`, every diagram and bullet point across all 8 syllabus PDFs is now covered, except:
- The `statechart_annotated.pdf` "Activity: Microwave Oven" slide (no content given — it's a live in-class exercise).
- The two blank "Hostel Management System" class-activity slides (also no content given).

Both are activity prompts with nothing on the page to teach — there's nothing missing to add for them.
