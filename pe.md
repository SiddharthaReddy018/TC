# Physics-Informed GNN for Polymer Chain Dynamics — 1-Year Execution Plan

**Expanded from the original 12-week plan. Nothing from that plan has been cut — every decision, parameter, formula, and rule from the 12-week version is preserved below. What changed is scope: with ~52 weeks instead of 12, things that were "stretch goals if time permits" are now core deliverables, one-shot checks are now repeated-and-averaged checks, and there is room for a real physics onboarding phase before any code gets written.**

**Team:** 2–3 people, zero code written yet, shared/limited Colab GPU, ~1 year (52 weeks) total.
**Core deliverable (unchanged):** a working baseline GNN and a physics-informed GNN, trained under matched conditions, compared on long-rollout stability of a coarse-grained polymer chain.
**New deliverables enabled by the longer timeline:** a third model family (equivariant / momentum-conserving), a full OOD grid instead of single-axis spot checks, hyperparameter optimization instead of hand-picked settings, uncertainty quantification, a scaling study across chain lengths, and a report written to a standard you could submit somewhere (workshop paper, poster, thesis chapter) rather than just a course deliverable.

---

## 0. How to use this document

This is a **reference document**, not a script to follow top to bottom in one sitting. Suggested first pass:
1. Read §1 (physics study roadmap) and start studying — this can run in parallel with everything else in Month 1.
2. Read §2–§5 (simulator, equations, parameters, dataset schema) once, together as a team, before anyone writes simulation code.
3. Use §9 (the month-by-month plan) as the actual week-to-week checklist.
4. Keep §13 (verification protocol) and §14 (I/O audit) open while building — they're checklists you tick off at each stage, not things you read once.
5. Everything your team already validated in the four prior planning rounds (dynamics choice, parameter table, dataset schema, pseudocode) is preserved intact in §2–§8; treat those as **locked decisions**, not open questions, unless your supervisor tells you otherwise.

A companion file, `recommended_tools_and_integrations.md`, lists things found while researching this plan that are **not** folded into the timeline below — new libraries, papers, and services worth a team discussion before you commit to any of them. Nothing in that file is assumed or required by this plan.

A second companion file, `CLAUDE.md`, is a compact project-knowledge summary you can paste into a fresh chat (with any AI assistant) to restore full context without re-uploading everything.

---

## 1. Physics & math you need to study first (you said you don't know this in depth — start here)

You do **not** need a physics degree to execute this project, but you do need working intuition for five things before writing simulation or loss-function code. Study in this order — each layer assumes the one before it. Budget roughly **3–4 weeks part-time (Month 1)**, done in parallel across the team so no one is blocked.

### 1.1 Statistical mechanics basics (½–1 week)
What you need: what temperature means for a simulated particle system, the Boltzmann distribution, and *equipartition* (why `k_BT` sets an energy scale, and why "temperature" for a system without well-defined velocities has to be estimated differently — this is exactly the overdamped-dynamics subtlety in §2).
- MIT OpenCourseWare, *Statistical Mechanics I* (free, full course): https://ocw.mit.edu/courses/8-333-statistical-mechanics-i-statistical-mechanics-of-particles-fall-2013/
- Wikipedia, *Equipartition theorem* (fast orientation, not a substitute for the course): https://en.wikipedia.org/wiki/Equipartition_theorem

### 1.2 Brownian motion, Langevin dynamics, and why "overdamped" is a real approximation (1 week)
What you need: the physical picture of a particle buffeted by solvent collisions, the difference between the full (underdamped) Langevin equation and the overdamped limit your project uses, and *why* the overdamped limit is valid for a coarse-grained polymer bead (this is the "why this is valid, when it fails" gap your own review docs flagged).
- Wikipedia, *Brownian dynamics* — short, correct, good starting orientation: https://en.wikipedia.org/wiki/Brownian_dynamics
- NSF-hosted review connecting Langevin/Brownian dynamics to coarse-grained simulation practice: https://par.nsf.gov/servlets/purl/10297047
- Hudson, *Coarse-graining and the overdamped Langevin formalism* (Warwick thesis chapter — directly addresses "when does overdamped break down"): https://wrap.warwick.ac.uk/id/eprint/135196/7/WRAP-Coarse-graining-overdamped-Langevin-formalism-Hudson-2020.pdf
- IOP article on Langevin/Brownian dynamics pitched at students: https://iopscience.iop.org/article/10.1088/1361-6404/ac93c9

**Checkpoint before moving on:** you should be able to explain, in your own words, why `γ dr/dt = F(r) + ξ(t)` drops the inertial term `m d²r/dt²`, and name one physical regime (hint: very light/fast-moving beads, or short-timescale ballistic motion) where that would be a bad approximation.

### 1.3 Stochastic differential equations and the Euler–Maruyama method (1 week)
What you need: what it means for a differential equation to have a random forcing term, why you can't just use ordinary Euler integration, and what "strong convergence order" means well enough to understand why your project gets the *better* 1.0 convergence rate (§2 explains why — this is where you confirm the reasoning, not where you first meet the concept).
- Higham, *An Algorithmic Introduction to Numerical Simulation of Stochastic Differential Equations*, SIAM Review 43(3), 2001 — the standard, most-cited accessible reference, built around runnable code: https://doi.org/10.1137/S0036144500378302 (search your university library or Google Scholar for a free PDF mirror if the DOI is paywalled — it is extremely widely mirrored)
- Worked reproduction of Higham's examples with commentary: https://random-walks.org/book/papers/num-sde/num-sde.html

**Checkpoint:** you should be able to write, from memory, the one-line Euler–Maruyama update for `dr = F(r)dt + σ dW`, and say why the noise term scales with `√dt` and not `dt`.

### 1.4 Polymer physics: bead-spring chains, FENE, WCA, and scaling laws (1–1.5 weeks)
What you need: the Kremer–Grest bead-spring picture, what FENE and WCA potentials physically represent, and the standard shape statistics (radius of gyration, end-to-end distance) you'll use for validation in §13.
- Rubinstein & Colby, *Polymer Physics* (Oxford University Press) — the standard textbook for this material, self-contained, assumes only calculus/physics/chemistry: https://global.oup.com/academic/product/polymer-physics-9780198520597 (check your university library for access — this is the book your original report's parameter choices are drawn from)
- Introductory polymer-physics article aimed at physics students (good bridge before tackling the full textbook): https://iopscience.iop.org/article/10.1088/1361-6404/ac93c9
- Macromolecular coarse-graining reference with worked FENE/WCA force derivations: https://phas.ubc.ca/~steve/publication/HadizadehLinhanantaPlotkin_wSuppMat_Macromol11.pdf

**Checkpoint:** derive the FENE force by hand from its potential (this is also Week 2 of the plan below — you're doing it a second time deliberately, once to learn it and once as the project's own validation step). You should be able to say in one sentence why WCA prevents bead overlap and why it's cut off at `2^(1/6)σ`.

### 1.5 Graph neural network fundamentals (1 week, can run in parallel with 1.1–1.4)
What you need: message passing, permutation invariance, and why a GNN is a natural fit for beads-and-bonds data.
- Sánchez-Lengeling et al., *A Gentle Introduction to Graph Neural Networks*, Distill, 2021 — the best first read, interactive, free: https://distill.pub/2021/gnn-intro/
- Companion piece, *Understanding Convolutions on Graphs*: https://distill.pub/2021/understanding-gnns/
- Stanford CS224W, *Machine Learning with Graphs* — free full lecture series + slides, the deeper reference once the Distill articles click: https://cs224w.stanford.edu/ (lecture videos: https://www.youtube.com/playlist?list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn)
- PyTorch Geometric's own "Creating GNNs" tutorial (practical, code-first): https://pytorch-geometric.readthedocs.io/en/latest/notes/create_gnn.html

**Checkpoint:** explain message passing (`m_ij = message(x_i, x_j, e_ij)`, aggregate, update) well enough to whiteboard it without notes.

### 1.6 Equivariance (needed only once you reach the EGNN / momentum-conserving stretch models in Month 9+, but worth a first pass now)
- Satorras, Hoogeboom & Welling, *E(n) Equivariant Graph Neural Networks* (the EGNN paper your original plan already selected as the stretch model): https://arxiv.org/abs/2102.09844
- e3nn documentation, *What is equivariance?* section — the most approachable introduction to E(3)-equivariance used by NequIP/MACE (see companion tools file): https://docs.e3nn.org/

**How to actually study this as a team, given limited time:** don't all three people read everything. Assign §1.1–1.3 (stochastic physics) to whoever owns simulation (Role A in §7), §1.4 (polymer physics) to the same person plus a shared team session, and §1.5 to whoever owns the model (Role B/C). Everyone attends one shared 1-hour session per topic where the owner explains it back to the team — teaching it is the actual comprehension check, and it catches misunderstandings before they turn into bugs in Week 5+.

---

## 2. Simulator decision: HOOMD-blue (unchanged from 12-week plan, version note added)

Your original report left this open pending "GPU availability and implementation complexity." Given your actual situation (pure-Python ML pipeline, small team, shared Colab GPU, nobody's written code yet), HOOMD-blue is the clear call, and a year's runway doesn't change that:

- It's Python-first — your simulator, dataset builder, and PyTorch Geometric pipeline live in one language, no LAMMPS-dump-file parsing step in between.
- Its `hoomd.md.methods.Brownian` integrator directly implements the overdamped Langevin equation you've already chosen as your main equation.
- It installs on Colab in about 2 minutes via `condacolab` + conda-forge, with automatic GPU detection.
- **Practical tip specific to your compute situation:** for a chain of 20–50 beads, HOOMD's GPU acceleration barely matters — GPU speedups in MD only kick in at much larger particle counts. Run data generation on Colab's *CPU* runtime (or a laptop) and save your GPU quota for GNN training. Over a full year this matters even more: you'll generate far more trajectories (see §9), so keeping simulation on CPU is what makes the larger dataset affordable.

**⚠️ Version note (checked August 2026, not in your original report):** HOOMD-blue has moved fast since your last search. It's now at major version 5.x/6.x (your original links pointed at v3.x docs). The core API you need — `hoomd.md.methods.Brownian(filter=..., kT=...)` — has been **stable in name and basic usage across v3 → v6**, so the pseudocode in §8 still applies without changes. But do this before writing any simulator code:
1. Check the current version and install instructions fresh: https://hoomd-blue.readthedocs.io/en/latest/installation.html
2. Skim the migration notes if you ever copy an example from an older tutorial: https://hoomd-blue.readthedocs.io/en/latest/migrating.html (the main breaking change relevant to you: the old `alpha` property on `Brownian`/`Langevin` methods was removed — use `gamma` directly, which is what the parameter table in §4 already assumes).
3. Use the official Glotzer-lab workshop notebook as your Colab install template rather than reconstructing the condacolab incantation from memory — it's kept up to date: https://github.com/glotzerlab/hoomd-workshop
4. **Record the exact HOOMD version, PyTorch version, PyTorch Geometric version, and CUDA version in every trajectory's and every model checkpoint's metadata.** Over a 12-month project, Colab's underlying images and conda-forge packages *will* update under you at least once, and a version drift silently changing simulation output is a real, previously-seen failure mode in long research projects. This is a new requirement added to the dataset schema in §5 — flagged here because HOOMD's own release cadence is the reason it matters.

Fallback (unchanged): if HOOMD install ever breaks in your specific Colab environment, LAMMPS's `fix brownian` (not `fix langevin` + `fix nve`, which is velocity-Verlet and not built for true overdamped dynamics) is the correct alternative.

---

## 3. Your Euler–Maruyama question, for the record (unchanged)

Yes — and you're already using it. Your primary equation, `γ dr/dt = F(r) + ξ(t)`, is a stochastic differential equation, and Euler–Maruyama is its standard first-order integrator. Your own Part 4 pseudocode already implements it.

- Because your noise term doesn't depend on position (**additive noise**), Euler–Maruyama's usual strong convergence order of 0.5 improves to **1.0** for this specific equation — a better numerical guarantee than the "cheap first-order method" reputation suggests. (See §1.3 for the background reading behind this claim.)
- That guarantee is asymptotic (`dt→0`). FENE and WCA are both stiff near their singularities, so you still need the empirical checks in §13 (bond-length histograms, no blow-ups) — theory doesn't replace validation.
- **One correction, kept from the original report:** if you ever use LAMMPS instead, use `fix brownian`, not `fix langevin`+`fix nve` (velocity-Verlet based, not built for the overdamped regime). Also, `fix brownian`'s built-in temperature compute is not physically meaningful there since velocities are undefined in the overdamped limit — estimate temperature from displacement/diffusion statistics instead. This applies whether you use LAMMPS or hand-roll your own overdamped integrator.

---

## 4. Locked parameter table (starting values — validate empirically, then confirm with your supervisor)

These are the standard, textbook Kremer–Grest values, used essentially unchanged across the polymer-physics literature — not guesses. **Unchanged from the 12-week plan**, with one addition (a third chain length, affordable now that you have a year) marked *NEW*.

| Parameter | Symbol | Starting value | Notes |
|---|---|---|---|
| Bead count (main) | N | 30 | Good middle ground: fast, but long enough for meaningful R_g / R_ee statistics |
| Bead count (OOD test) | N | 50 | "Longer chain" generalization test |
| Bead count (scaling study) *NEW* | N | 100, 200 | Only reachable with a year's runway — lets you check R_g ~ N^ν scaling (§9, Month 10) rather than a single two-point comparison |
| FENE spring constant | k | 30 ε/σ² | Standard KG value |
| FENE max extension | R₀ | 1.5 σ | Standard KG value |
| WCA energy scale | ε | 1.0 | Reduced units |
| WCA length scale | σ | 1.0 | Reduced units |
| Resulting equilibrium bond length | — | ≈0.965 σ | Emerges from FENE+WCA combined — **not** an independently set parameter; don't confuse with a "harmonic r₀" |
| Temperature | k_BT | 1.0 ε | Standard reduced units |
| Friction coefficient | γ | 0.5–1.0 m/τ | Start at 1.0; this is also your 4th OOD axis (§9) |
| Timestep | dt | 0.001–0.005 τ | τ = σ√(m/ε). Start conservative, increase only after stability checks pass |
| Burn-in / equilibration | n_burnin | ≥ a few Rouse times (~N² in reduced units) | Confirm via your own R_g/bond-length equilibration plots, don't just trust the formula |
| Save interval | save_every | every 100 integration steps | Adjust once you see autocorrelation times |
| Trajectory count (pilot) | — | 20–30 | Month 1–2, to validate pipeline |
| Trajectory count (production, N=30) | — | 400–600 *(scaled up from 150–250)* | With a year instead of 12 weeks, simulation time stops being the bottleneck — bigger production runs reduce seed-to-seed noise in every downstream metric and let you report tighter confidence intervals. Still CPU-cheap; the constraint is calendar time to validate and process, not compute quota |
| Trajectory count (N=50, 100, 200 OOD/scaling arms) *NEW* | — | 100–150 each | Enough for OOD/scaling statistics without matching the main production count |
| Split | — | 70% / 15% / 15% | By trajectory, never by frame |

If you use a harmonic bond instead of FENE for the "clean first version" your docs mention, then r₀ (equilibrium length) is a real independent parameter — a common starting choice is r₀ = σ = 1.0, k_bond ≈ 100 ε/σ² (much stiffer than ε to keep bonds tight, but small enough for dt=0.001–0.005τ to remain stable).

---

## 5. Dataset schema (unchanged from 12-week plan — this was already thorough — plus new reproducibility fields)

**Per-trajectory metadata (store all of):**
`traj_id`, `integrator_type` ("overdamped_langevin"), `noise_seed`, `friction (γ)`, `target_temperature`, `timestep_size (dt)`, `cutoff_distance`, `simulation_box_size`/`boundary_conditions`, `mass`, `chain_length (N)`, `split_label` ("train"/"val"/"test"), `ood_flag`, `ablation_id`.

**Per-frame fields:**
`timestep_index`, positions, velocities (if stored), forces, bond list, potential energy, kinetic energy, `is_equilibrated` flag, `center_of_mass` position, `end_to_end_vector` (or just the indices of the two end beads).

**Node features:** position, velocity (if used), force (if available), bead index, **normalized chain position (i/N)** — interior vs. end beads behave differently and a raw "local_coordination" count doesn't fully capture *where* along the chain a bead sits — local_coordination (bonded-neighbor count), `bead_type`/`monomer_type` (future-proofing for heterogeneous chains), `chain_index` (future-proofing for multi-chain systems).

**Edge features:** relative vector, distance, `edge_type` (1=bonded, 2=nonbonded-within-cutoff, 0=absent), `cutoff_flag`, `bond_order`/`spring_type` (future-proofing if you ever mix harmonic and FENE).

**Targets:** primary = next displacement. Also store (not necessarily predict) `next_velocities`, `next_forces`, `next_temperature_estimate` — needed later if you add a temperature-consistency loss.

**Normalization:** z-score normalize displacement magnitudes and any other continuous input/target before training anything — skipping this is a common silent cause of a GNN that trains but never actually learns the dynamics. **Store the normalization statistics (mean/std per field) as a versioned artifact alongside the dataset**, not just applied in-memory — you will need the exact same statistics to de-normalize model output during rollout evaluation and again a year later when writing the report, and re-deriving them from a re-shuffled dataset will silently give slightly different numbers.

### 5.1 New reproducibility fields (added for the 1-year scope — not in the original 12-week schema)

A year-long project accumulates far more experiments than a 12-week one, and the failure mode that costs the most time is "which exact code+data+config produced this checkpoint?" Add these to every trajectory and every training run:

**Per-trajectory (software environment):** `hoomd_version`, `python_version`, `numpy_version`, `platform` (CPU/GPU, Colab session type). Rationale: HOOMD's API and RNG implementation can change across minor versions (§2); if trajectory statistics ever look subtly different between two batches, this is the first thing you check.

**Per-trajectory (data provenance):** `sha256_checksum` of the raw trajectory file, `generation_timestamp`, `generating_script_git_commit`. Rationale: lets you detect silent corruption or an accidental re-run with a changed config, and ties every data file back to the exact code that produced it.

**Per-training-run (experiment identity):** `config_file_hash`, `code_git_commit`, `dataset_version_id`, all random seeds used **separately** — `sim_noise_seed` (already covered above), `train_val_test_split_seed`, `model_init_seed`, `dataloader_shuffle_seed`. These are four different seeds; conflating them into "the seed" makes it impossible to later isolate whether a result changed because of data-order or because of weight initialization. Also log `wall_clock_start`, `wall_clock_end`, `gpu_hours_used`, `colab_session_id` — over a year you will want to know where your compute budget actually went (see §9's budget tracking).

**Per-training-run (physics-informed loss curriculum):** if using a loss-weight curriculum (§9, Month 4), log the actual `lambda_bond(epoch)` and `lambda_excl(epoch)` schedule that was used, not just the target end values — curricula get tweaked during debugging and the schedule that actually ran is what needs to be reproducible.

---

## 6. Pipeline architecture — the stages, at a glance

This section gives the map; §13 (verification) and §14 (full I/O audit) give the detail. Every stage below takes something in, does one clearly-scoped job, and writes something out to disk — no stage should silently depend on in-memory state from a previous notebook cell that isn't also saved to a file, because Colab sessions die.

```
[1] Physics design ──► parameter table (§4), config YAML
        │
        ▼
[2] Simulator (HOOMD-blue) ──► raw trajectories (positions, velocities, forces, energies per frame)
        │                       + per-trajectory metadata (§5)
        ▼
[3] Simulator validation ──► pass/fail report + diagnostic plots (§13.1)
        │  (loop back to [2] if this fails — do not proceed past this gate)
        ▼
[4] Dataset engineering ──► graph samples (PyG Data objects) + train/val/test split manifest
        │                    + normalization statistics artifact
        ▼
[5] Baseline GNN training ──► checkpoints + training logs + loss curves
        │
        ▼
[6] Baseline evaluation ──► one-step metrics + rollout curves + steps-until-divergence
        │
        ▼
[7] Physics-informed GNN training ──► checkpoints + logs + per-loss-term curves
        │
        ▼
[8] Controlled comparison ──► comparison tables/plots, statistical significance across seeds
        │
        ▼
[9] Ablations + OOD stress tests ──► ablation matrix results, OOD grid results
        │
        ▼
[10] Stretch models (EGNN / momentum-conserving / equivariant) ──► same evaluation suite, re-run
        │
        ▼
[11] Final consolidation ──► frozen code + re-run of best experiments + final report/paper + repro archive
```

Two rules that apply at every arrow in this diagram (detailed in §13):
1. **Nothing moves to the next stage until the previous stage's validation checks pass.** This is the single most-repeated instruction across your own prior planning docs, and it's worth repeating here because a year-long project makes it *easier*, not harder, to skip — with more calendar time, the temptation to "just start the GNN while the simulator's still a bit off" gets stronger, and it's still wrong.
2. **Every arrow writes a versioned file, not just a Python variable.** If you can't point to the exact file that stage N produced and stage N+1 consumed, you don't have a reproducible pipeline.

---

## 7. Team roles

### 7.1 If 3 people (unchanged core split, expanded responsibilities for the year)
- **A — Simulation & physics:** owns the HOOMD-blue simulator, parameter validation, physics loss math, the physics study roadmap (§1.1–1.4) as team teacher, and the "what should we tell our supervisor" list (§17). *New for the year-long scope:* also owns the scaling-study arm (N=100, 200) and the version/environment-tracking fields in §5.1.
- **B — Data & baseline model:** owns dataset generation pipeline, graph construction, normalization, and the baseline GNN. *New:* owns the hyperparameter-optimization pass (§9, Month 6) and data-efficiency curve experiments (§9, Month 8).
- **C — Training infra & evaluation:** owns the training loop, physics-informed variant, all metrics/plots, ablation bookkeeping, and drafting results sections as they come in. *New:* owns experiment tracking (W&B or equivalent — see tools file), the OOD grid (§9, Month 7), and uncertainty-quantification experiments (§9, Month 9).

All three co-own the ablation matrix, the stretch models in Months 9–11, and report-writing in Month 12.

### 7.2 If 2 people (unchanged split)
Collapse to (A) simulation + data pipeline, (B) model + training + evaluation. Physics-loss design and the ablation matrix become explicitly shared work, done together rather than solo. *New for the year-long scope:* with only two people, be more conservative about which stretch goals in Months 9–11 you actually attempt — a working, thoroughly-validated two-model comparison with a full OOD grid and a written-up report is a *stronger* outcome than three half-finished models. Treat §9's Months 9–11 as a menu, not a checklist, if you're a two-person team.

### 7.3 Working rhythm for a year-long project (new — not needed at 12 weeks, necessary here)
- **Weekly 30-minute sync**, even in slow weeks. The single biggest risk in a year-long student project is not technical difficulty, it's drift — everyone quietly assuming someone else is handling something.
- **Monthly checkpoint against this document.** At the end of each month in §9, spend 30 minutes comparing what actually happened to what was planned, and explicitly update the plan (don't just let it silently diverge — note *why* something moved).
- **Code review before merging anything that touches the physics** (force calculations, loss terms, evaluation metrics). Bugs in these are the ones that produce plausible-looking wrong numbers, which are far more expensive to catch six months later than a crash is to catch immediately.
- **One person owns the "definition of done" for each month's checklist** (rotate this) — someone has to be the one who says "no, we're not moving to Month N+1 yet."

---

## 8. Exact pseudocode (preserved from your original Part 4, unchanged — this was already correct)

This is kept verbatim in structure because it's already correct and your team has already reviewed it. Treat it as the reference implementation skeleton for Months 2–4.

### 8.1 Simulator
```
Input:
  N, T_steps, dt, gamma, T, k_bond, r0, epsilon, sigma, n_burnin, save_every, seed

Initialize random seed
Initialize polymer coordinates r[1..N] as a stretched or random walk chain
Initialize velocities v[1..N] if underdamped is used

for t in 1..T_steps:
    F = zeros_like(r)
    # bonded forces
    for i in 1..N-1:
        r_ij = r[i+1] - r[i]
        F_bond = bond_force(r_ij, k_bond, r0)
        F[i] += F_bond
        F[i+1] -= F_bond
    # excluded-volume forces
    for all nonbonded pairs (i,j) with |i-j| > 1:
        if distance(r[i], r[j]) < cutoff:
            F_rep = wca_force(r[i]-r[j], epsilon, sigma)
            F[i] += F_rep
            F[j] -= F_rep
    # stochastic dynamics update
    if overdamped:
        for i in 1..N:
            noise = sqrt(2*kB*T*dt/gamma) * Normal(0,1)
            r[i] = r[i] + (dt/gamma) * F[i] + noise
    else if underdamped:
        for i in 1..N:
            noise = sqrt(2*gamma*kB*T*dt) * Normal(0,1)
            v[i] = v[i] + dt*(F[i] - gamma*v[i])/m + noise/m
            r[i] = r[i] + dt*v[i]

    if t > n_burnin and t mod save_every == 0:
        save(r, v, F, potential_energy, kinetic_energy, temperature, t, seed,
             hoomd_version, python_version, git_commit)   # <- §5.1 additions
```

### 8.2 Dataset generation
```
Input: M trajectories, simulation settings, equilibration length, save interval

dataset = []
for traj_id in 1..M:
    seed = unique_seed(traj_id)
    trajectory = run_simulator(seed)
    for each saved frame s in trajectory after burn-in:
        current_state = frame[s]
        next_state = frame[s+1]
        graph = build_graph(current_state)
        target = next_state.positions OR next_state.displacement
        sample = {
            "graph": graph, "target": target,
            "metadata": {traj_id, seed, dt, temperature, friction,
                         split_label, ood_flag, ablation_id,          # <- §5 additions
                         sha256_checksum, generation_timestamp}        # <- §5.1 additions
        }
        dataset.append(sample)

split dataset by trajectories: train 70% / val 15% / test 15%
save dataset + normalization_stats.json to versioned processed files
```

### 8.3 Graph construction
```
Input: polymer frame r[1..N]
nodes = []; edges = []
for i in 1..N:
    node_feature = [position r[i], velocity v[i] if available, force F[i] if available,
                    bead_index, i/N, local_coordination, bead_type, chain_index]
    nodes.append(node_feature)
for i in 1..N-1:
    edge_feature = [distance(r[i], r[i+1]), relative_vector, bond_flag=1, edge_type=1]
    edges.append((i, i+1, edge_feature))
for all pairs (i,j) with |i-j| > 1:
    if distance(r[i], r[j]) < neighbor_cutoff:
        edge_feature = [distance(r[i], r[j]), relative_vector, bond_flag=0,
                         edge_type=2, cutoff_flag=1]
        edges.append((i, j, edge_feature))
return graph(nodes, edges)
```

### 8.4 Baseline GNN
```
Input: graph with node features x and edge features e
Output: predicted next positions or displacements
x = node_encoder(x); e = edge_encoder(e)
for layer in 1..L:
    m_ij = message_mlp([x_i, x_j, e_ij]) for each edge (i,j)
    m_i = aggregate(m_ij over incoming edges to node i)
    x_i = update_mlp([x_i, m_i])
pred = decoder(x)
return pred
```

### 8.5 Physics-informed loss
```
pred = GNN(graph)
loss_pred = MSE(pred, target)
loss_bond = sum over bonded pairs of (distance(pred_pos_i, pred_pos_j) - r0)^2
loss_excl = sum over nonbonded pairs of max(0, r_min - distance(pred_pos_i, pred_pos_j))^2
total_loss = loss_pred + lambda_bond(epoch)*loss_bond + lambda_excl(epoch)*loss_excl   # curriculum, §9 Month 4
backprop(total_loss)
```

### 8.6 Training loop
```
Input: train_loader, val_loader, model, optimizer, epochs
best_val = infinity
for epoch in 1..epochs:
    model.train()
    for batch in train_loader:
        batch = add_small_noise(batch) if noise_injection else batch
        pred = model(batch.graph)
        loss = physics_loss(pred, batch.target) if physics_model else MSE(pred, batch.target)
        optimizer.zero_grad(); loss.backward(); optimizer.step()
    model.eval()
    val_loss = sum(evaluation_loss(model(batch.graph), batch.target) for batch in val_loader)
    if val_loss < best_val:
        best_val = val_loss
        save_checkpoint(model, optimizer, epoch, config_hash, git_commit)   # <- §5.1
return best_checkpoint
```

### 8.7 Rollout evaluation
```
Input: test trajectories, trained model, rollout length K
for each test trajectory:
    state = initial frame
    for t in 1..K:
        graph = build_graph(state)
        pred = model(graph)
        state = pred as next input
        compute: one_step_error[t], bond_error[t], excluded_volume_violation[t],
                 radius_of_gyration[t], MSD[t], temperature_proxy[t]
aggregate statistics over all test trajectories; plot metric vs rollout step
```

---

## 9. The 12-month roadmap

Each month below is a direct expansion of the corresponding week(s) in your original 12-week plan — nothing is dropped, everything gets more thorough, and Months 9–11 promote former "stretch goals" to core work. Each month has a **Gate** — the condition that must be true before you move to the next month. Gates are not suggestions; they're the same "do not proceed" discipline your own prior planning docs insisted on for the simulator, applied consistently across the whole year.

**Compute budget tracking (do this from Month 1):** keep a running spreadsheet (or a tab in your experiment tracker) of GPU-hours used per month against a rough annual budget. A reasonable default budget assuming Colab Pro (~$10–50/month, see companion tools file for current pricing) is to spend almost nothing on GPU during Months 1–4 (simulation is CPU-only, per §2) and concentrate GPU spend in Months 5–11 (training). Re-forecast monthly — this is what tells you, in Month 6, whether the ablation/OOD/scaling scope in Months 9–11 is actually affordable or needs trimming.

### Month 1 (Weeks 1–4) — Physics onboarding + environment + project contract
- Run §1's physics study roadmap in parallel across the team (see §7.3 for how to split it).
- One-page "project contract": dynamics regime (overdamped, locked), chain length, target = displacement, exact train/val/test split rule. This is unchanged from the original Week 1 — write it in Week 1, not Week 4, so ambiguity doesn't fester while people are studying.
- Install HOOMD-blue on Colab via `condacolab` (forces an automatic runtime restart — expected, re-run from the next cell after). Use the current version per §2's version note.
- Install PyTorch + PyTorch Geometric.
- Set up a git repo: `data/raw/`, `data/processed/`, `data/splits/`, `models/baseline/`, `models/physics_informed/`, `models/egnn/`, `models/momentum_conserving/`, `logs/`, `figures/`, `reports/drafts/`, `configs/`. (One new folder vs. the original — `models/momentum_conserving/` — because that stretch model is now core; see Month 11.)
- One YAML config file per experiment from day one — don't hardcode parameters in scripts.
- Mount Google Drive for checkpointing; any simulation or training run over ~1 hour needs to checkpoint per-trajectory or per-epoch, not just at the end.
- **New for the year-long scope:** set up an experiment tracker (Weights & Biases free tier or equivalent — see companion tools file) *now*, before you have any results worth tracking. Retrofitting experiment tracking onto 50 already-run experiments in Month 6 is far more painful than starting with it. Also write a one-page `CONTRIBUTING.md` covering: branch naming, when to open a PR vs. push directly, and the code-review rule from §7.3 for anything touching physics.
- **Gate:** HOOMD + PyTorch + PyG all import and run a trivial example on your actual Colab environment; the repo skeleton exists; the project contract is written and everyone agrees on it.

### Month 2 (Weeks 5–8) — Physics deep-dive + simulator implementation begins
- Re-derive the FENE/WCA force expressions by hand, not just copy formulas (this is the "checkpoint" exercise from §1.4 — do it for real here, as a team, on a whiteboard).
- Confirm with your supervisor: harmonic or FENE bond for the first version? Hard-priority on bond length, excluded volume, or both? (This is one of the §17 supervisor flags — resolve it early since it changes code, not just documentation.)
- Write down what "physically reasonable" looks like numerically: expected bond length (~0.965σ for standard FENE params), target T, expected R_g scaling (R_g ~ N^ν with ν ≈ 0.588 for a good solvent / self-avoiding walk — this is the number you'll check the simulator against in Month 3, and check the *scaling study* against in Month 10).
- Begin simulator implementation (§8.1 pseudocode → real HOOMD-blue code): chain initialization, FENE+WCA force objects, `hoomd.md.methods.Brownian` integrator.
- **New:** write unit tests for the force functions in isolation — compute the FENE/WCA force at 3–4 hand-picked bond lengths / separations and compare against your by-hand derivation from earlier in this month, *before* wiring them into the full simulator. This catches sign errors and unit-convention mistakes at the cheapest possible point to fix them.
- **Gate:** force unit tests pass; a single bead-pair (not yet a full chain) simulated under known conditions matches the by-hand expected behavior (e.g., two WCA-repelled beads separate, two FENE-bonded beads oscillate around ~0.965σ).

### Month 3 (Weeks 9–13) — Full simulator + thorough pilot validation (pilot: 20–30 trajectories)
- Complete the simulator: full chain, all forces, `Brownian` integrator, checkpointing.
- Also implement the minimal NumPy Euler–Maruyama version from your original plan — an independent cross-check of HOOMD's output on a tiny system (this catches HOOMD-specific bugs that a HOOMD-only validation would never see).
- Run pilot trajectories. Validate everything from the original Week 3–4 checklist: bond-length histogram centered near expected value, no persistent overlaps, temperature stable near target (estimated from displacement statistics, not raw `compute temp` — see §3), different seeds → different but statistically similar trajectories, R_g and MSD look physically sane.
- **New, made possible by the extra time — do these properly rather than "eyeball the plot":**
  - Compare measured R_g(N) scaling against the theoretical exponent (ν ≈ 0.588) using your pilot data across a couple of chain lengths, not just N=30.
  - Estimate the autocorrelation time of R_g and bond length from the pilot trajectories, and use it to sanity-check (and if needed revise) `save_every` — the original plan flagged "adjust once you see autocorrelation times" but a 12-week timeline rarely leaves room to actually act on that. Now you do.
  - Block-average your pilot statistics to get real error bars on "temperature stays near target" rather than a qualitative plot — this is the first place you establish the seed-to-seed variance habit that §12's metrics checklist requires throughout.
  - Cross-check 2–3 pilot trajectories against the independent NumPy implementation and confirm they agree within expected statistical variation.
- **Do not proceed to graph/GNN work until this passes.** This is still the single most common place these projects quietly go wrong, and it's worth repeating verbatim from the 12-week plan.
- **Gate:** all bullets above pass and are written up (even briefly) in `reports/drafts/simulator_validation.md` with the actual plots — this becomes the first real section of your eventual final report, not just a checkbox.

### Month 4 (Weeks 14–17) — Production dataset engineering + naive baselines
- Scale to production: 400–600 trajectories at N=30 (§4's revised count). Kick off the N=50/100/200 OOD/scaling-arm generation in the background during this month too — it's CPU-only and can run unattended while the team does dataset-engineering work on the N=30 pilot data.
- Convert validated trajectories into graph samples using the schema in §5 (including the §5.1 reproducibility fields).
- Trajectory-level split only. Verify zero leakage (write an automated check, not a manual glance — e.g., assert no `traj_id` appears in more than one of train/val/test).
- Compute and save normalization statistics as a versioned artifact (§5).
- Build the naive baselines now, before any GNN work: **0a. zero-displacement** (predicts no motion) and **0b. global-statistics random draw** (samples a displacement from the training set's marginal distribution). You need these numbers to know if the GNN is doing anything at all — this is unchanged from the original plan and just as important now.
- **Gate:** production dataset (all arms) generated and passing the same validation checks as the pilot (spot-check, don't have to redo every plot on every trajectory); naive baseline numbers computed and recorded; zero split leakage confirmed by an automated test.

### Month 5 (Weeks 18–21) — Baseline GNN implementation + one-step training
- Plain message-passing GNN in PyTorch Geometric: encode → message pass → decode (§8.4). Modest depth/width to start (2–4 layers, hidden dim 64–128).
- Train one-step prediction only. Save best checkpoint by validation loss.
- **New:** before any real training run, do the debugging-order checklist from your own Part 3 doc, which is worth preserving verbatim here because it's exactly right: (1) overfit a tiny dataset, (2) confirm loss decreases, (3) check predictions visually match a short trajectory, (4) check one-step metrics, (5) check rollout metrics. If a model cannot overfit a tiny batch, the implementation is likely wrong — don't skip straight to the full production run.
- Run a small manual sanity sweep (not full HPO yet) over depth ∈ {2,3,4} and hidden dim ∈ {64,128} to pick a sensible starting configuration for Month 6's real search.
- **Gate:** model overfits a tiny batch; full training run completes without NaNs; validation loss curve is sane (decreasing, not wildly oscillating).

### Month 6 (Weeks 22–25) — Baseline rollout evaluation + hyperparameter optimization
- Test-set one-step MSE.
- Autoregressive rollout. Plot bond-length error, R_g, MSD, and **steps-until-divergence** (first NaN/exploded rollout step — a clean, cheap pass/fail number) vs. rollout step, not just final numbers.
- **New, affordable now with a year's compute budget:** run a real hyperparameter optimization pass (Optuna or similar — see companion tools file) over the baseline architecture: depth, hidden dim, learning rate, batch size. **Lock these in as the frozen baseline architecture/hyperparameters before Month 7** — the physics-informed model in Month 7 must use exactly this configuration, unchanged, so that any improvement is attributable to the physics loss terms and not to a better-tuned architecture. This directly strengthens the "fair comparison" rule your own review docs already flagged as essential.
- **Gate:** baseline architecture and hyperparameters are frozen and written into a config file that Month 7's physics-informed model will inherit unmodified; rollout evaluation plots exist for the baseline and are saved as report-ready figures.

### Month 7 (Weeks 26–29) — Physics-informed model
- Identical architecture to baseline (frozen in Month 6) + bond loss + excluded-volume loss (§8.5).
- **Use a loss-weight curriculum:** ramp λ_bond and λ_excl up over the first several epochs rather than fixing them from step 0 — introducing large physics-loss weight before the model can even do one-step prediction well is a common, avoidable cause of unstable training in exactly this kind of setup.
- Same optimizer, LR, batch size, stopping rule as baseline. Store loss terms separately (prediction loss, bond loss, excluded-volume loss, each logged per epoch — not just the summed total).
- Train with **5 seeds** (upgraded from the 12-week plan's "3 seeds is a reasonable floor" — with a year's budget, 5 gives materially tighter confidence intervals for the core comparison in Month 8, and this is exactly the kind of thing that was cut for time at 12 weeks and shouldn't be now).
- **Gate:** physics-informed model trains stably with the curriculum (no loss blow-up); all three loss terms logged separately and behave sensibly (bond/excl losses decrease as the curriculum ramps in); 5-seed runs complete.

### Month 8 (Weeks 30–33) — Controlled comparison + data-efficiency study
- Baseline vs. physics-informed vs. **both naive baselines** on the same test trajectories: one-step metrics, rollout stability, physical-constraint violations, steps-until-divergence, all reported as **mean ± std across the 5 seeds**, not single numbers.
- Run a paired statistical test (e.g., paired t-test or Wilcoxon signed-rank across seeds/trajectories) on the headline comparisons rather than eyeballing whether error bars overlap — a genuinely new rigor step that 12 weeks doesn't leave room for.
- Report honestly if physics-informed helps one metric but hurts another — unchanged from the original plan, and still the most scientifically important sentence in this whole document.
- **New — data-efficiency curve:** re-train both baseline and physics-informed models on 25%/50%/75%/100% of the training trajectories (same architecture, same seeds) and plot test-set rollout stability vs. training-set size. This answers a question reviewers/supervisors will ask ("how much does the physics prior help when data is scarce?") that a 12-week project simply doesn't have time to address, and it's a natural, low-additional-engineering-cost extension once the core comparison pipeline exists.
- **Gate:** core comparison table complete with seed statistics and significance tests; data-efficiency curves plotted; findings written into `reports/drafts/core_comparison.md`.

### Month 9 (Weeks 34–37) — Full ablation matrix + full OOD grid
- Run the ablation matrix in §11 (now including the momentum-conservation row as a serious entry, not an optional afterthought — see Month 11 for the model itself, but its ablation slot is reserved here).
- **New — full OOD *grid* instead of single-axis spot checks:** the original plan tested chain length, timestep, and temperature as separate one-at-a-time perturbations. With a year, run the actual 2×2×2×2-style grid across chain length {30(train), 50, 100}, timestep, temperature, and friction γ (§4's 4th OOD axis) — or at minimum every pairwise combination if full factorial is too expensive. Single-axis tests can hide interaction effects (e.g., a model that's fine for longer chains *and* fine for different timesteps individually, but fails when both change together); a grid is what actually tells you whether the model learned a dynamics family or memorized a regime, which is the exact question your own "why physics-informed learning is needed" framing depends on.
- If GPU hours are tight, prioritize ablation rows and OOD grid cells in the order that isolates the most scientifically important comparisons first (bond loss alone vs. bond+excluded-volume, and chain-length OOD before the rarer combinations).
- **Gate:** ablation matrix fully populated; OOD grid at least pairwise-complete; results written up with the same seed-statistics rigor as Month 8.

### Month 10 (Weeks 38–41) — EGNN (promoted from stretch to core) + scaling study
- EGNN as a third model, run through the *entire* evaluation suite (one-step, rollout, ablation-relevant subset, OOD grid subset) — not a one-off comparison. At 12 weeks this was "optional if time permits"; at 52 weeks it's core, because rotational/translational equivariance is a real, testable hypothesis about *why* physics-informed losses help (or don't), and a year gives you time to actually test it rather than mention it.
- Reference: Satorras, Hoogeboom & Welling, EGNN: https://arxiv.org/abs/2102.09844
- **New — scaling study:** using the N=30/50/100/200 data generated in Month 4, check whether R_g(N) scaling from all three trained models (baseline, physics-informed, EGNN) matches the theoretical Flory exponent (ν≈0.588) better or worse than each other, and how that changes with rollout length. This is a genuinely new scientific result a 12-week project can't reach, and it's a natural centerpiece for a written report or poster.
- **Gate:** EGNN trained, evaluated end-to-end, compared against baseline and physics-informed on the same footing; scaling-study plots exist across all four chain lengths for all three models.

### Month 11 (Weeks 42–45) — Momentum-conserving model (promoted stretch) + uncertainty quantification + interpretability
- **Momentum-conservation loss / architecture**, inspired by Dynami-CAL GraphNet (Sharma & Fink, *Nature Communications* 17:1045, Jan 2026 — https://doi.org/10.1038/s41467-025-67802-5, preprint: https://arxiv.org/abs/2501.07373), which enforces pairwise linear/angular momentum conservation via rotation-equivariant edge-local reference frames. This is cheaper to add than a full EGNN rebuild and directly targets the physical property (Newton's third law / momentum conservation) that your bond+excluded-volume losses don't explicitly enforce. At 12 weeks this was "optional add-on if there's still runway"; here it's a scheduled deliverable.
- **New — uncertainty quantification:** train a small ensemble (3–5 independently-initialized runs) of your best model and report rollout predictions with uncertainty bands, or explore split-conformal prediction on the one-step residuals for a calibrated uncertainty estimate. This turns "the model predicts X" into "the model predicts X ± uncertainty," which is a meaningfully stronger scientific claim and a common reviewer request that a 12-week project has no time to address.
- **New — interpretability pass:** inspect learned edge messages / attention-like weights (if your architecture has them) to see whether the model has implicitly learned to weight bonded vs. nonbonded edges differently, and whether physics-informed training changes this compared to the baseline. Keep this lightweight — a few illustrative plots, not a new research thread.
- **Gate:** momentum-conserving model trained and evaluated end-to-end; at least one UQ method produces calibrated-looking uncertainty estimates; interpretability plots exist for at least the baseline vs. physics-informed comparison.

### Month 12 (Weeks 46–52) — Final consolidation (7 weeks — includes deliberate buffer)
- Freeze code. Re-run best experiments with fixed seeds for the final numbers that go in the report (this is the run whose numbers you actually cite — don't cite numbers from a run whose code has since changed).
- Final plots/tables, failure-case writeup, limitations section (§16 gives you the scope-discipline language to use here almost verbatim).
- Reproducibility packaging: `environment.yml` or `requirements.txt` pinned to exact versions, a top-level `README.md` that lets a new person reproduce the pipeline end-to-end (this is the "new teammate" checklist from your own Part 3 doc — by Month 12 it doubles as your reproducibility statement), and an archive of the exact processed dataset + all final checkpoints.
- Write the full report/paper draft using the structure in §18.
- Prepare a presentation/poster built around the scientific question (does physics-informed help, and how much, and does it scale, and is it interpretable/uncertainty-aware) — not implementation details.
- **Weeks 51–52 are deliberately unscheduled buffer.** Every long project has a "the export broke" or "the figure needs redoing" week near the deadline; the 12-week plan had no slack anywhere, which is exactly the kind of compression a year should remove. Use this time, or don't need it — either is fine.
- **Gate:** everything in §19's final deliverables checklist is checked off.

---
## 10. Model roadmap — what each model is for, in one place

This consolidates the "why this model" reasoning that was scattered across your four prior planning rounds into a single reference table, plus the two new core models the year enables.

| # | Model | Status | Why it exists | Introduced |
|---|---|---|---|---|
| 0a | Zero-displacement | Naive baseline | Proves the GNN learned *something* beyond "chains don't move much" | Month 4 |
| 0b | Global-statistics random draw | Naive baseline | Proves the GNN learned something beyond the marginal displacement distribution | Month 4 |
| 1 | Baseline message-passing GNN | Core | Simplest architecture that respects graph structure; the fair comparison point for everything else | Month 5–6 |
| 2 | Baseline + noise injection | Ablation row | Tests whether GNS-style training-time noise (Sanchez-Gonzalez et al. 2020, see tools file) alone explains rollout robustness, independent of physics losses | Month 9 |
| 3 | Physics-informed (+ bond loss) | Core | Isolates the effect of the bond-length constraint alone | Month 7, ablation row Month 9 |
| 4 | Physics-informed (+ bond + excluded-volume) | Core | The main comparison point against the baseline | Month 7–8 |
| 5 | + momentum-conservation loss | Core (promoted from stretch) | Tests whether explicitly enforcing Newton's-third-law-consistent pairwise momentum exchange (Dynami-CAL-inspired) improves on scalar bond/excluded-volume penalties alone | Month 11 |
| 6 | EGNN variant | Core (promoted from stretch) | Tests whether *architectural* equivariance (rather than a soft loss penalty) is a better way to encode the same physical priors | Month 10 |

**What this project is not** (unchanged from your original plan, worth restating here since the model list has grown): it is not a universal polymer generator, not a replacement for all molecular dynamics, not a claim that the model learns exact physics, and not an argument that any one of models 5/6 is "the" right architecture — it's a controlled study of whether, and how, different ways of injecting physical structure affect long-horizon rollout stability for one coarse-grained polymer chain under overdamped Langevin dynamics. Growing the model list from 5 to 7 rows does not change this framing; keep the claims exactly as narrow as your original Part 3 doc specified (§16 restates this).

---

## 11. Ablation matrix (expanded — momentum-conservation and EGNN are no longer optional rows)

| Model | Noise injection | Bond loss | Excluded-vol loss | Momentum loss | EGNN |
|---|---|---|---|---|---|
| **0a. Zero-displacement (does nothing)** | — | — | — | — | — |
| **0b. Global-statistics random draw** | — | — | — | — | — |
| 1. Baseline GNN | optional | no | no | no | no |
| 2. GNN + noise | yes | no | no | no | no |
| 3. GNN + bond | optional | yes | no | no | no |
| 4. GNN + bond + excl | optional | yes | yes | no | no |
| 5. GNN + bond + excl + momentum | optional | yes | yes | yes | no |
| 6. EGNN variant | optional | yes/no | yes/no | optional | yes |

Rows 0a/0b exist purely so you can say, with a number, that the GNN learned something beyond "chains don't move much" or "displacements look like this on average." Without them you can't actually make that claim — this reasoning is unchanged from the 12-week plan and remains the whole point of including them.

**Execution order if compute is ever tight (unchanged principle from the 12-week plan, restated for the larger matrix):** rows near the top of this table are more essential than rows near the bottom. If Month 9's OOD grid and this ablation matrix are competing for the same GPU budget, prioritize completing rows 0a through 4 and a pairwise-complete OOD grid over completing rows 5–6 exhaustively — a clean 5-row comparison beats a noisy 7-row one.

---

## 12. Full metrics checklist (expanded)

One-step MSE · bond-length error · excluded-volume violation rate · kinetic-energy/temperature consistency (estimated correctly for overdamped dynamics, per §3) · long-horizon rollout stability curves (metric vs. step, not just endpoint) · **steps-until-divergence** · R_g distribution · end-to-end distance · MSD · seed-to-seed variance, reported not eyeballed (mean ± std across **5 seeds** for the core comparison, per Month 7's revision) · paired significance test on headline comparisons (Month 8) · **new for the 1-year scope:** data-efficiency curves (Month 8) · R_g(N) scaling-law fit and its exponent vs. theoretical ν≈0.588 (Month 10) · calibration quality of any uncertainty estimates, e.g. coverage of prediction intervals (Month 11) · GPU-hours and wall-clock cost per experiment (tracked throughout, per §9's budget-tracking note — this isn't a physics metric but it's the number that tells you whether the rest of this checklist is actually affordable).

A model can be accurate for one step and still fail after autoregressive rollout because errors compound — this is the central phenomenon the whole project studies, and no amount of additional scope in Months 9–11 changes that it's the headline result. Don't judge any model by one-step MSE alone, ever, at any point in the year.

---

## 13. Verification & validation protocol — how to check inputs and outputs at every stage

This section answers directly: *how do we check and verify things, at every point, not just at the end?* Each subsection maps to one arrow in §6's pipeline diagram.

### 13.1 Simulator output validation (gate before Month 4)
- **Bond-length histogram**: centered near the theoretical equilibrium value (~0.965σ for standard FENE+WCA); check this holds across seeds, not just one trajectory.
- **No persistent bead overlap**: fraction of frames with any pairwise distance below the WCA cutoff should be ~0 after equilibration.
- **Temperature stability**: estimated from displacement/diffusion statistics (never from a raw kinetic-energy readout in the overdamped limit, per §3), fluctuating around the target within expected statistical bounds — use block-averaging (§9, Month 3) to get an actual error bar, not a visual "looks stable."
- **No numerical blow-ups**: no NaN/Inf positions, no bond lengths diverging.
- **Seed reproducibility**: different seeds produce different but *statistically similar* trajectories — same mean R_g, same mean bond length, different individual paths. Check this with a formal comparison (e.g., overlapping confidence intervals across seed groups), not a glance at two plots.
- **Scaling sanity check** (new, §9 Month 3): R_g(N) trend across whatever chain lengths you've piloted should be in the right direction and roughly the right shape before you trust the full-N=30 production run.
- **Cross-check against the independent NumPy Euler–Maruyama implementation** (§9 Month 3): agreement within expected statistical variation on a small system.

### 13.2 Dataset engineering validation (gate before Month 5)
- **Zero split leakage**: automated assertion that no `traj_id` appears in more than one of train/val/test.
- **Schema completeness**: every field in §5 and §5.1 present and non-null for every sample (write a schema-validation script — don't check by hand).
- **Normalization sanity**: after applying stored normalization stats, training-set displacement magnitudes have mean ≈0, std ≈1; spot-check that de-normalizing recovers the original values exactly (a round-trip test).
- **Graph construction sanity**: for a handful of hand-picked frames, manually verify that bonded edges match the known chain connectivity and that nonbonded edges only appear within the cutoff distance.
- **Checksum verification**: confirm the SHA256 checksums recorded in §5.1 match the actual files on disk before starting any expensive training run — catches silent corruption from interrupted Colab sessions.

### 13.3 Model debugging checklist (gate before trusting any training run, every model, every month)
Unchanged from your own Part 3 doc, because it's already correct — repeat this exact sequence for *every* new model, not just the first one:
1. Overfit a tiny dataset.
2. Confirm loss decreases.
3. Check that predictions visually match a short trajectory.
4. Check one-step metrics.
5. Check rollout metrics.
If a model cannot overfit a tiny batch, the implementation is likely wrong — this applies just as much to the EGNN and momentum-conserving models in Months 10–11 as it did to the baseline in Month 5.

### 13.4 Training run validation (every run, throughout)
- Loss curves logged and inspected, not just the final number — a loss that "decreased" but spiked wildly midway is a different result than a smooth decrease, even if the endpoint is identical.
- For physics-informed models: prediction loss, bond loss, and excluded-volume (and later momentum) loss terms logged *separately* per epoch, and checked that the curriculum (§9 Month 7) is actually ramping as configured.
- Checkpoint selected by validation loss, confirmed to be the actual best epoch (not accidentally the last one, if early stopping wasn't triggered correctly).
- Experiment metadata (§5.1: seeds, git commit, config hash) recorded and retrievable — spot-check monthly that you can actually look up "what produced checkpoint X" from your tracker.

### 13.5 Evaluation output validation (every comparison, throughout)
- Rollout metrics computed identically for every model being compared (same test trajectories, same rollout length, same metric code path) — a subtle but common bug source is accidentally evaluating two models on slightly different data or code.
- Naive baselines (0a/0b) re-checked alongside every new comparison as a sanity floor — if a sophisticated model ever loses to the zero-displacement baseline on any metric, that's a red flag to investigate before reporting the result, not to quietly drop.
- Statistical claims (Month 8 onward) always reported with seed statistics and, where a claim is load-bearing for the report's conclusions, a significance test — an unqualified "physics-informed is better" is not a claim this protocol allows you to make from a single-seed comparison.
- Physical-constraint violation rates (bond error, excluded-volume violation) checked against the *training* distribution's typical values, not just against zero — a small nonzero violation rate close to what the simulator itself produces under thermal noise is expected and healthy, not a bug.

### 13.6 Reproducibility validation (Month 12, but worth a dry run mid-year)
- A team member who did *not* write the original code follows only the `README.md` and reproduces one full result (e.g., the baseline's rollout stability curve) from raw config to final plot. Do this once around Month 6–7 as a dry run (catches documentation gaps while there's still time to fix them) and once for real in Month 12.

---

## 14. Full input/output audit — every stage, and whether anything is still missing

You specifically asked for the *overall* input/output picture and whether anything's missing. Here it is as one table, stage by stage, cross-referencing §5–§8. Read this alongside §6's diagram.

| Stage | Inputs | Outputs | Verified in |
|---|---|---|---|
| Physics design | Literature values, supervisor decisions (harmonic vs FENE, etc.) | Parameter table (§4), project contract, config YAML | Team review, Month 1–2 |
| Simulator | Config YAML (N, dt, γ, T, k_bond, r0/FENE params, ε, σ, n_burnin, save_every, seed) | Per-frame: positions, velocities (if underdamped), forces, bond list, PE, KE, temperature-proxy, `timestep_index`, `center_of_mass`, `end_to_end_vector`, `is_equilibrated`. Per-trajectory: all §5 metadata + §5.1 environment/provenance fields | §13.1 |
| Simulator validation | Raw trajectories | Pass/fail report, diagnostic plots (bond histogram, R_g/MSD/temperature vs. time, scaling check) | §13.1, written to `reports/drafts/simulator_validation.md` |
| Dataset engineering | Validated raw trajectories | Graph samples (PyG `Data` objects) with node/edge features per §5, target displacements, split manifest, normalization-stats artifact | §13.2 |
| Naive baselines | Training-set displacement statistics | Zero-displacement and global-statistics predictors + their metrics on test set | §13.5 |
| Model training (any of models 1–6) | Graph dataset, frozen config (architecture + hyperparameters), optimizer settings, seeds (4 separate seeds per §5.1) | Checkpoints, per-epoch loss curves (all terms separately for physics-informed models), experiment metadata | §13.3, §13.4 |
| Rollout evaluation | Trained checkpoint, test trajectories, rollout length K | Per-step metrics (one-step error, bond error, excluded-volume violation, R_g, MSD, temperature-proxy), steps-until-divergence, aggregate plots | §13.5 |
| Controlled comparison | Evaluation outputs for all models being compared | Comparison tables/plots with seed statistics, significance tests | §13.5 |
| Ablation / OOD grid | Same as above, across the matrix in §11 and the grid in §9 Month 9 | Populated matrix/grid with results | §13.5 |
| Final consolidation | All of the above, frozen | Final report, reproducibility archive, presentation | §13.6, §19 |

### 14.1 Confirming nothing from your prior review rounds was dropped

Your team already ran one explicit pass asking "are there extra input/output variables I should have considered" (this is the content that became your `in_input_output_variables` document). Every field identified there is present in §5 above: `integrator_type`, `noise_seed`, `simulation_box_size`/`boundary_conditions`, `cutoff_distance`, friction/temperature/timestep metadata, `mass`, `center_of_mass`, `end_to_end_vector`, `timestep_index`, `is_equilibrated`, `bead_type`/`monomer_type`, `chain_index`, `local_coordination`, explicit `edge_type`/`cutoff_flag`, `bond_order`/`spring_type`, `next_velocities`/`next_forces`/`next_temperature_estimate`, `split_label`, `ood_flag`, `ablation_id`. Nothing from that review is missing here.

### 14.2 What's newly added in this expansion (not in any prior round — the honest answer to "is anything still missing")

Going through the pipeline again with a full year's engineering discipline in mind (not just the physics/ML content, which your prior rounds already covered thoroughly), five categories of I/O were genuinely absent before and are added in §5.1:

1. **Software/environment version fields** (`hoomd_version`, `python_version`, package versions, platform) — absent from every prior round. This matters uniquely for your project because HOOMD-blue's API has changed release-to-release (§2), and a 1-year timeline makes it likely you'll hit at least one such change mid-project.
2. **Data provenance fields** (checksums, generation timestamps, generating-script git commit) — lets you detect corruption or an accidental stale re-run, which becomes a real risk once you have dozens of dataset versions instead of one.
3. **Disaggregated random seeds** (simulation-noise seed vs. split seed vs. model-init seed vs. dataloader-shuffle seed) — your prior rounds correctly flagged "noise_seed" for the simulator but didn't separate the *other* seeds that affect training reproducibility. All four are now distinct fields.
4. **Experiment identity fields** (config hash, code commit, dataset version ID, wall-clock/GPU-hours) — needed once you have enough experiments (Months 6–11) that "which run produced this number" stops being answerable from memory.
5. **Curriculum-schedule logging** (the actual λ_bond/λ_excl trajectory used, not just the target values) — specific to the physics-informed loss and only matters because §9 Month 7 introduces a curriculum that wasn't in the original single-shot loss-weight design.

If you complete §5, §5.1, and this table's stage-by-stage checklist, the I/O specification is complete for the scope in this document. The one thing that *cannot* be fully specified in advance is whatever fields the stretch models in Months 10–11 (EGNN, momentum-conserving) turn out to need internally (e.g., EGNN's coordinate-update formulation may want an explicit "reference frame" field) — treat those as an expected, normal addition when you reach that model, not a gap in this plan.

---

## 15. Risk register (new — not needed at 12 weeks, necessary at 52)

A 12-week project is short enough that "what could go wrong" is mostly "we run out of time." A year-long project has more, and different, failure modes worth naming in advance.

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| HOOMD-blue (or PyTorch/PyG) releases a breaking API change mid-project | Medium | Medium | Version-pin your environment (§5.1) once the simulator is validated in Month 3; don't casually `pip install --upgrade` mid-year. Re-validate against §13.1 if you ever do upgrade. |
| A team member becomes unavailable for an extended period | Medium | High | §7's role split means each person owns a domain, but the weekly sync (§7.3) and shared physics-teaching sessions (§1.6) mean no single person is the *only* one who understands any given piece. Document decisions in `reports/drafts/`, not just in someone's head. |
| Colab quota/session limits become a bottleneck during heavy training months (7–11) | Medium | Medium | Track GPU-hours monthly (§9); if you're consistently hitting limits, this is the trigger to consider the paid-tier or alternative-compute options in the companion tools file — decide as a team, don't let one person quietly pay out of pocket. |
| Scope creep — Months 9–11's promoted stretch goals expand further ("let's also try MACE/NequIP") | High | Medium | The companion tools file exists specifically to *contain* this impulse — new ideas go there for later discussion, not directly into this plan, until the team explicitly re-scopes. |
| A months-old bug is discovered in the simulator or loss code after downstream results already exist | Low–Medium | High | This is exactly why §13's gates and the Month 3 written validation report exist — the earlier a bug is caught, the less work is invalidated. If one is found late, re-run affected experiments rather than patching results after the fact. |
| The physics-informed model simply doesn't beat the baseline | Medium | Low (scientifically) | This is a valid, reportable outcome — §16 and §18 already commit you to reporting honestly either way. A null or mixed result, well-characterized across the ablation/OOD grid, is a legitimate year-long deliverable. |
| Supervisor requests a scope change mid-year | Medium | Medium | §17's flag list exists to surface exactly the decisions a supervisor should weigh in on — raise these early and often rather than assuming; it's cheaper to redirect in Month 2 than to redo Month 8's work. |

---
## 16. What NOT to do (unchanged principle, restated for a year-long scope)

Don't start EGNN or the momentum-conserving model before the baseline vs. physics-informed comparison (Month 8) is clean. Don't add more than bond + excluded-volume + (later) momentum loss before that core comparison exists. Don't run large hyperparameter sweeps before the pipeline is validated (§13.1–13.2). Don't judge any model by one-step MSE alone — it hides exactly the failure mode this project exists to study. And specific to the extra year of runway: **don't let "we have time" become "we have no deadline discipline."** Every month in §9 has a gate for a reason; a year is long enough to finish this project well, but also long enough to drift indefinitely if no one enforces the gates.

Keep the claims exactly as narrow as your original Part 3 doc specified, even with a richer model list: this is one polymer family, one dynamics regime, one coarse-grained model, one surrogate target, studied through several different ways of adding physical structure to a GNN. That stops overclaiming, and it still applies with 7 model rows instead of 5.

---

## 17. Flag these to your supervisor early, don't decide alone

Unchanged from the 12-week plan, plus two new items that only arise because of the longer timeline:
- Harmonic vs. FENE bond for the first version.
- Whether the revised trajectory counts (§4) and the scaling-study chain lengths (N=100, 200) are worth the added simulation/validation time, or whether they'd rather you spend that time elsewhere.
- Hard-priority physics terms: is excluded-volume as important as bond length, or secondary? Where does the momentum-conservation term (Month 11) rank against them?
- **New:** whether promoting EGNN and the momentum-conserving model from "stretch" to "core" (§9, Months 10–11) matches their expectations for the project's scope, or whether they'd rather you spend that time going deeper on the baseline-vs-physics-informed comparison (more seeds, a finer OOD grid, a proper power analysis) instead of adding model variety.
- **New:** whether they want the final output framed as a course/thesis deliverable, or written toward an external venue (workshop paper, poster submission) — this changes how much polish and related-work coverage Month 12's writing phase needs, and it's worth knowing well before Month 12.

---

## 18. Report writing structure (unchanged order, still correct)

Introduction → Physics background → Related work → Why overdamped dynamics → Dataset generation → Graph construction → Baseline model → Physics-informed model → (New: EGNN and momentum-conserving models) → Training strategy → Evaluation metrics → Results → Ablations → OOD/scaling study (new) → Uncertainty and interpretability (new) → Limitations → Future work.

Report-safe wording, unchanged from your Part 3 doc and just as true with the expanded scope:
- We study a single coarse-grained polymer chain.
- We compare plain and physics-informed GNNs (and, as extensions, EGNN and a momentum-conserving variant).
- We use overdamped Brownian dynamics as the main setting.
- We evaluate one-step accuracy, long-rollout physical stability, out-of-distribution generalization across four physical axes, and (new) data efficiency and scaling behavior.
- We use physics losses to reduce bond and overlap violations, and test whether architectural equivariance or explicit momentum conservation do better than soft loss penalties at the same job.

---

## 19. Final deliverables checklist (Month 12 gate)

- [ ] Frozen, version-pinned codebase with a `README.md` that a new person can follow end-to-end (dry-run this once mid-year per §13.6)
- [ ] `environment.yml`/`requirements.txt` with exact pinned versions (HOOMD, PyTorch, PyG, CUDA)
- [ ] Archived processed dataset (all arms: main N=30, OOD, scaling) with checksums
- [ ] All final checkpoints (baseline, physics-informed, EGNN, momentum-conserving) archived with their config/seed metadata
- [ ] `reports/drafts/simulator_validation.md` (from Month 3)
- [ ] `reports/drafts/core_comparison.md` (from Month 8)
- [ ] Ablation matrix and OOD grid results, fully populated (Month 9)
- [ ] Scaling-study plots (Month 10)
- [ ] Uncertainty and interpretability figures (Month 11)
- [ ] Full report/paper draft following §18's structure
- [ ] Presentation/poster built around the scientific question, not implementation details
- [ ] Limitations and future-work sections written honestly, per §16
- [ ] Companion files (`recommended_tools_and_integrations.md`, `CLAUDE.md`) reviewed and either acted on or explicitly declined as a team

---

## 20. Appendix — consolidated reference links

**HOOMD-blue:** latest docs https://hoomd-blue.readthedocs.io/en/latest/ · installation https://hoomd-blue.readthedocs.io/en/latest/installation.html · migration notes https://hoomd-blue.readthedocs.io/en/latest/migrating.html · Colab workshop template https://github.com/glotzerlab/hoomd-workshop

**PyTorch Geometric:** docs https://pytorch-geometric.readthedocs.io/ · "Creating GNNs" tutorial https://pytorch-geometric.readthedocs.io/en/latest/notes/create_gnn.html · GitHub https://github.com/pyg-team/pytorch_geometric

**Core physics-informed / equivariant GNN references:**
- EGNN — Satorras, Hoogeboom & Welling: https://arxiv.org/abs/2102.09844
- Dynami-CAL GraphNet — Sharma & Fink, *Nature Communications* 17:1045 (2026): https://doi.org/10.1038/s41467-025-67802-5 (preprint: https://arxiv.org/abs/2501.07373)
- Graph Network Simulator (source of the noise-injection training trick) — Sanchez-Gonzalez et al. 2020: https://arxiv.org/abs/2002.09405
- MeshGraphNets — Pfaff et al. 2021: https://arxiv.org/abs/2010.03409
- Multi-scale graph networks for coarse-grained MD (directly on-topic) — Fu et al. 2022: https://arxiv.org/abs/2204.10348

**Physics study resources (see §1 for the full annotated list):**
- MIT OCW Statistical Mechanics I: https://ocw.mit.edu/courses/8-333-statistical-mechanics-i-statistical-mechanics-of-particles-fall-2013/
- Brownian dynamics overview: https://en.wikipedia.org/wiki/Brownian_dynamics
- Coarse-graining/overdamped Langevin (Hudson thesis chapter): https://wrap.warwick.ac.uk/id/eprint/135196/7/WRAP-Coarse-graining-overdamped-Langevin-formalism-Hudson-2020.pdf
- Higham 2001, SDE numerical methods: https://doi.org/10.1137/S0036144500378302
- Rubinstein & Colby, *Polymer Physics*: https://global.oup.com/academic/product/polymer-physics-9780198520597
- "A Gentle Introduction to Graph Neural Networks": https://distill.pub/2021/gnn-intro/
- Stanford CS224W: https://cs224w.stanford.edu/

**Your team's own previously-identified references** (carried over from your four prior planning rounds, still relevant, not re-verified individually in this pass — spot-check any you rely on heavily):
https://iopscience.iop.org/article/10.1088/1361-6404/ac93c9 ·
https://par.nsf.gov/servlets/purl/10297047 ·
https://en.wikipedia.org/wiki/Brownian_dynamics ·
https://pmc.ncbi.nlm.nih.gov/articles/PMC12392447/ ·
https://www.nature.com/articles/s41467-023-43720-2 ·
https://pmc.ncbi.nlm.nih.gov/articles/PMC12641476/ ·
https://phas.ubc.ca/~steve/publication/HadizadehLinhanantaPlotkin_wSuppMat_Macromol11.pdf ·
https://chemrxiv.org/engage/api-gateway/chemrxiv/assets/orp/resource/item/66e3a03bcec5d6c142e9f3f6/original/main.pdf ·
https://pmc.ncbi.nlm.nih.gov/articles/PMC12389585/ ·
https://www.tandfonline.com/doi/full/10.1080/20550340.2025.2547335 ·
https://pubs.aip.org/apl/article/126/5/052901/3333896/Physics-informed-neural-networks-for-phase-field ·
https://pmc.ncbi.nlm.nih.gov/articles/PMC12030369/ ·
https://www.sciencedirect.com/science/article/pii/S0959440X26000205 ·
https://www.sciencedirect.com/science/article/pii/S0893608025010378 ·
https://arxiv.org/pdf/2209.05582.pdf ·
https://link.springer.com/article/10.1007/s10822-024-00578-w ·
https://pubs.acs.org/doi/abs/10.1021/acs.jpclett.5c00217 ·
https://arxiv.org/abs/2112.03383 ·
https://pubs.acs.org/doi/10.1021/acs.jpcb.3c07304 ·
https://docs.deepmodeling.com/projects/deepmd/en/v2.0.0.b4/getting-started.html

For everything found newly in this research pass that is **not** already folded into the plan above — new tools, libraries, and papers your team hasn't discussed yet — see `recommended_tools_and_integrations.md`.
