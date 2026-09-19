# Software Engineering: Fast Study Guide

Grouped by concept. Sections 1-3 are foundations, 4-8 are the modelling techniques (5-7 are what the midsem tests), 9 is design intro, 10 is project planning (not in the slides), 11 has worked answers to the four midsem questions.

Midsem coverage: Q1 class diagram (15), Q2 sequence diagram (25), Q3 statechart (15), Q4 project planning / critical path (15) = 70 marks.

---

## 1. Software Engineering Big Picture

Why SE exists: real projects have limited time, money, people and expertise. Teams are multi-person, distributed and changing. Software is often business- or safety-critical. There is also competition, regulation and sustainability.

SDLC phases and the core problem at each:

```
1 Requirement Engineering : requirements are ill-understood and changing
2 Design                  : solution choices unknown
3 Construction / Coding   : implementation choices unknown
4 Testing (V&V)           : when to test? when to stop? how to get test inputs?
                            how to decide pass/fail?
5 Maintenance
```

- Verification: "are we building the product right?" (against the spec)
- Validation: "are we building the right product?" (against the user's need)

Course admin: theory is mid 30, end 40, attendance 10, assignments 20 (classwork + homework). Lab is project 50 and attendance 50. References: Pressman, Sommerville.

---

## 2. Requirements Engineering (RE)

RE answers "what is to be done?". It is the basis for the contract (SLA, QoS), planning (budget, HR, timelines, tech stack) and architecture.

### Two kinds of requirements

- **Functional (FR)**: the features. If unmet, you get correctness or completeness problems. Example: "user can add a task".
- **Non-functional (NFR)**: quality attributes:
  - -ilities: availability, interoperability, adaptability, flexibility, configurability, dependability, reliability, stability, scalability, elasticity, modularity
  - resource usage: CPU, memory, power, human effort
  - affordability (price)
  - performance: latency, response time, throughput
  - real-time constraints
  - safety, security
  - fairness, accessibility, inclusivity
  - sustainability (SDG goals)
  - regulatory compliance

### RE process (Pressman), 7 steps

1. **Inception**: understand the problem, who wants it, what solution, and how well customer and developer communicate. Ask who is behind the request, who will use it, and the economic benefit.
2. **Elicitation**: meetings with a facilitator, an agenda and a definition mechanism. Work products: statement of need and feasibility, scope, stakeholder list, technical environment, requirements list, usage scenarios, prototypes.
3. **Elaboration**: build the analysis model (data, function, behaviour).
4. **Negotiation**: find each stakeholder's "win conditions"; aim for win-win.
5. **Specification**: a written document, a set of models, a formal mathematical spec, use cases, or a prototype.
6. **Validation**: review for errors, missing information, inconsistencies, conflicting or unrealistic requirements.
7. **Requirements management**.

Validation checklist (short form): consistent with the objective? right abstraction level? necessary? bounded and unambiguous? has a source (attribution)? conflicts? achievable? testable?

---

## 3. Properties of Good Requirements (the "3 Cs" + more)

The 3 Cs: **Correctness, Completeness, Consistency**.
Full list: clear, concise, simple, precise, unambiguous, consistent, complete.

```
Ambiguous   : "Response should be prompt."
Unambiguous : "Under <network conditions>, respond within <N> ms."

Inconsistent (navigation): event e in Screen A -> go to C;
    Screen B is inside A; event e in Screen B -> go to D. Which one fires?
Inconsistent (physics): torque 0<=t<=x -> acc=a1; torque t>=x -> acc=a2.
    At t=x both apply. (Overlap.)

Incomplete : 3 user types A,B,C; 4 features F1-F4; only says A gets F1,F2.
Complete   : A: F1,F2 / B: F2,F3 / C: only F3   (and F4 is stated too)
```

Completeness and ambiguity are related: a missing case is a form of ambiguity.

---

## 4. Analysis Model = The Modelling Toolkit

Four kinds of elements, each with its diagram:

```
Scenario-based   -> Use cases / use-case diagram
Class-based      -> Class diagram      (structure)
Behavioural      -> State diagram      (states over time)
                    Sequence diagram   (interaction over time)
Flow-oriented    -> Data Flow Diagram
```

**Use case**: a scenario from an actor's viewpoint. It answers:
- Who are the primary and secondary actors, and what is the goal?
- What preconditions hold?
- What are the main tasks?
- What extensions and variations exist?
- What information does the actor exchange with the system?
- What unexpected changes must be reported?

---

## 5. Class Diagrams (midsem Q1, 15 marks)

Class box:

```
+------------------+
|  ClassName       |   italic name = abstract
+------------------+
| - attribute:type |   + public  - private  # protected
+------------------+
| + operation()    |
+------------------+
```

Relationships (know the symbols):

```
Association      A ----------- B        "uses/knows", plain line, optionally with a name and roles
Aggregation      A <>--------- B        hollow diamond: whole-part, parts can live alone
Composition      A <#>-------- B        filled diamond: whole-part, parts die with the whole
Inheritance      Child ---|> Parent     hollow triangle at parent (is-a)
Dependency       A - - - -> B           dashed: temporary use
Realization      Class - - -|> Interface
```

Multiplicity is written at each end: `1`, `0..1`, `0..N` (or `0..*`), `1..N`.

How to find classes: take the description and underline the nouns (candidate classes/attributes) and verbs (operations/relationships). Discard synonyms and things that are only attributes.

Rules of thumb:
- "has multiple X, and X can't exist without it" means composition (Order to OrderItem).
- "various kinds/modes of X" means inheritance (Payment to CreditCard and UPI).
- "browses/places/refers to" means association.

---

## 6. Sequence Diagrams (midsem Q2, 25 marks, the biggest question)

Shows how objects interact over time.

```
 Customer        System        Technician        <- participants (top boxes)
    |               |               |             <- lifelines (dashed, vertical)
    |--raise()----->|               |             <- sync message (solid, filled head)
    |               |--assign()---->|
    |               |<--accept()----|             <- return / reply (dashed)
    |               |---[]          |             <- self message
```

Elements:
- Activation bar: a thin rectangle on the lifeline while an object is executing.
- Messages: sync (solid, filled arrow), async (open arrow), return (dashed), self-call.
- Time flows downward.
- Combined fragments, which is how you show the variations:
  - `alt`: if/else (with [guard] conditions)
  - `opt`: optional, runs only if the guard is true
  - `loop`: repeat
  - `par`: parallel
  - `break`: exit on an exceptional condition
  - `ref`: refer to another diagram

Approach for a long narrative: list the actors first, then walk through the text sentence by sentence and turn each into a message. Put every "if the customer does not accept..." into an `alt` fragment.

---

## 7. State Diagrams / Statecharts (midsem Q3, 15 marks)

Model of behaviour: which states an object can be in and what events move it between them.

### Basics

- State: a rounded box.
- Transition: an arrow labelled `event [guard] / action`.
- Initial state: a filled dot. Final state: a bullseye.
- Entry/exit/do actions: inside a state box, written as `entry / action`, `exit / action`.

### Statechart (Harel) extensions

```
Hierarchy (XOR / composite)  a state contains substates; you are in exactly one substate
                             ON contains IDLE, WASHING, PAUSE
Concurrency (AND / orthogonal regions)
                             a state with several regions separated by dashed lines,
                             ALL regions are active at the same time
Interlevel transition        an arrow that crosses a composite boundary (from a substate to outside)
History (H)                  re-enter a composite at the substate you left (Pause -> resume)
Synchronisation              one region's event triggers a transition in another region
                             (broadcast events, or guard like [in(WASHING)])
```

### Semantics questions the deck lists

- **Basic transition**: fires when its event occurs and the guard is true.
- **Conflicting transitions (simple)**: two transitions from the same state on the same event. This is nondeterminism, so avoid it.
- **Conflicting transitions (advanced)**: one at an outer level and one at an inner level. The inner (more specific) transition usually has priority.
- **Unreachable state**: no path leads to it from the initial state. This is a design bug.
- **Unescapable state**: no way out (a trap or dead-end state). This is a bug unless it is deliberately a final state.

### The washing machine model from the slides

```
TOP:   OFF <----power----> ON

ON has two concurrent regions:
 Washing Process Control:
    IDLE -> WASHING [SOAK -> AGITATE -> RINSE -> SPIN] -> (COMPLETED) -> IDLE
    any point in WASHING: pause -> PAUSE; pause again -> back via history (H)
 Water Management:
    CLOSED --(soaking started OR rinsing started)--> FILLING
    FILLING --filled--> CLOSED
    CLOSED --(soaking completed OR rinsing completed)--> DRAINING
    DRAINING --drain_complete--> CLOSED
```

Synchronisation between the regions:
- Soaking or rinsing must wait until "filled" (the target level).
- Draining only starts when washing or rinsing completes.

---

## 8. Data Flow Diagrams (not in the midsem, but in the syllabus)

A DFD shows how data is transformed as it moves through the system. Every system is input -> transformation -> output.

Elements:
- **External entity**: a producer or consumer of data (person, sensor, other system), drawn as a rectangle. Data must originate somewhere and go somewhere.
- **Process (data transformer)**: a circle or rounded box that turns input into output (compute tax, format report).
- **Data store**: two parallel lines, data kept for later (Sensor Data store).
- **Data flow**: a labelled arrow.

Levels:
- **Level 0 (context diagram)**: one process for the whole system, plus the external entities. Show no procedural logic.
- **Level 1 and below**: break the single process into sub-processes, with data stores.

Method: do a grammatical parse (nouns are entities or stores, verbs are processes), find the external entities, then draw level 0, then refine.

Example (ATM level 0): User sends the ATM card, PIN and a service request to the Cash Withdrawal process. It reads user data and gives back cash.

Rules: a process must have an input and an output; entities and stores never connect directly to each other.

---

## 9. Software Design Intro

Design is deciding how the system will be built, before coding. Desirable characteristics (from *Code Complete*):

```
1 Minimal complexity      5 Reusability
2 Ease of maintenance     6 Portability
3 Loose coupling          7 Standard techniques
4 Extensibility
```

- **Coupling**: how dependent modules are on each other (want low).
- **Cohesion**: how focused a module is (want high).

Class activity to remember: a To-Do list app (add, view, mark complete, remove; a task has a title and status). You documented the language and stack, UI, portability, 4-5 code aspects tied to the characteristics above, and the data structure (e.g., a list of Task objects).

---

## 10. Project Planning: Critical Path (midsem Q4, 15 marks)

No slide covers this. The method is simple.

Draw the activity-on-node graph: each task is a node, with an arrow from each predecessor to its task. Then:

```
FORWARD PASS  (earliest times)
  ES = max(EF of all predecessors)     (ES = 0 if no predecessor)
  EF = ES + duration
  Project length = largest EF

BACKWARD PASS (latest times)
  LF = min(LS of all successors)       (LF = project length for last tasks)
  LS = LF - duration

Slack = LS - ES     Critical path = tasks with slack 0 (longest path)
```

Related terms:
- **Gantt chart**: a bar-per-task timeline.
- **PERT**: the same idea, with duration = (optimistic + 4 x likely + pessimistic) / 6.

---

## 11. Worked Answers to the Midsem

### Q4 (fully solved)

```
Edges: A->B, A->C, B->D, B->E, D->E, C->F, E->G, F->G, G->H, H->I, H->J

Task  dur  ES  EF | LS  LF  slack
A      5    0   5 |  0   5   0  *
B      4    5   9 |  5   9   0  *
C      3    5   8 | 12  15   7
D      4    9  13 |  9  13   0  *
E      8   13  21 | 13  21   0  *
F      6    8  14 | 15  21   7
G      3   21  24 | 21  24   0  *
H      5   24  29 | 24  29   0  *
I      2   29  31 | 30  32   1
J      3   29  32 | 29  32   0  *

Critical path: A -> B -> D -> E -> G -> H -> J = 32 days (unique).
The path A-C-F-G-H-J is 25 days, so C and F have 7 days of slack.
```

### Q1

```
Classes: Customer, Address, Catalogue, Product, Order, OrderItem,
         Payment (abstract), CreditCard, UPI
Inheritance : CreditCard, UPI ---|> Payment
Composition : Order <#> 1 ---- 1..* OrderItem
              Customer <#> 1 ---- 1..* Address (or plain association, but say why)
Aggregation : Catalogue <> 1 ---- 0..* Product
Association : Customer 1 ---- 0..* Order ("places")
              OrderItem 0..* ---- 1 Product ("refers to")
              Order 1 ---- 0..1 Payment ("paid by")
              Customer ---- Catalogue ("browses")
Operations  : Customer.browseCatalogue(), placeOrder()
              Order.calculateTotal()
              OrderItem.getSubtotal()
              Payment.processPayment()   (overridden in CreditCard/UPI)
```

### Q2 (participants and message flow; then draw it)

```
Participants: Customer, System, Technician
(add Warehouse, Supplier/Dealer, Payment for part (b))

1 Customer -> System : raiseComplaint()
2 System : autoAssign(category, expertise, proximity, availability)
3 System -> Technician : assignJob()
4 Technician -> System : accept()
5 Technician -> Customer : visit(); examineIssue()
6 Technician -> System : submitQuotation()
7 System -> Customer : shareQuotation()
8 alt [accepted]  Customer -> System : accept() ; System : generateInvoice()
  else [rejected] System -> Technician : cancel/close job     (b.ii)
9 Technician : executeService(); submitDetails()
10 System -> Customer : requestFeedback()
11 alt [accepted] Customer -> System : accept(); System -> Customer : shareInvoice()
   else [not satisfactory] System -> Technician : reopen/rework job   (b.iii)
12 Customer -> System : makePayment()  -> System : markComplete()
   opt/alt [delay] System -> Customer : reminder; loop until paid; late fee/escalate   (b.iv)

(b.i) Spare parts: alt on source
   [technician has part]   -> proceed
   [company warehouse]     -> System reserves part, price added to invoice
   [authorised supplier / market]
        alt [customer buys directly] Customer pays dealer, gives receipt
        alt [technician pays cash]   cost added to invoice for reimbursement
        alt [technician creates credit note] Technician -> Dealer : creditNote(company)
                                             Company settles with Dealer later
```

### Q3 (order to draw it, mapped to the mark scheme)

```
(a) 5  Hierarchy: ON contains Washing Process Control; WASHING contains SOAK, AGITATE, RINSE, SPIN
(b) 2  Two concurrent regions inside ON (dashed line): Washing Process Control | Water Management
(c) 2  Entry/exit: entry/openInletValve in FILLING, exit/closeInletValve;
       entry/openDrain in DRAINING; entry/startMotor in AGITATE, exit/stopMotor
(d) 3  Labels: power, start, soaking_done, agitating_done, rinsing_done, spin_done, pause,
       resume, filled, drain_complete; each transition as event[guard]/action
(e) 2  Sync: WASHING/RINSING waits for "filled"; DRAINING only after washing/rinsing completes
       (guard [in(FILLING)] or broadcast event)
(f) 1  Errors: add an ERROR state (door_open, no_water, rotor_jammed) reachable from any
       substate of ON; it stops the motor and closes the valves, and a reset event returns
       to IDLE. Door open can pause via history.
```

---

## Exam Tips

- Q2 is 25 of 70 marks. Draw the main flow cleanly first, then the fragments for the variations.
- Q3 is almost the class example, so it is the cheapest set of marks.
- In Q1, state the multiplicities explicitly and add a one-line reason for each composition versus association choice.
- In Q4, show your ES/EF table. Method marks count even if you slip on a number.
