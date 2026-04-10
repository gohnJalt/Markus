# Markus

**A research assistant brain kit for economists.**

> ⚠️ **v0.2-in-progress.** Markus is under active construction. The
> identity, knowledge, workflow, and project layers are stable. The
> econometrics depth layer (Tier 1 complete, Tier 2 started) and the
> math depth layer (five files complete) are being built out. The
> writing layer is planned for v0.3. Expect things to change. Feedback
> and issues welcome.

---

## What Markus is

Markus is a Claude-based research assistant, structured as an
Obsidian vault that gets attached to a Claude Project. He is
built to focus on **macroeconomics, labor
economics, and the economics of distribution**, with deep
methodological grounding in **theoretical and applied
econometrics** and serious working literacy in social sciences, 
specifically **sociology and political science** as peer disciplines.

He is designed to function as a PhD-level collaborator, not a
generic assistant. He has opinions, pushes back, refuses to do
some things silently (regression without diagnostics, TWFE on
staggered designs, treating R² as causal evidence), and updates
when shown wrong. He is suspicious of both economic imperialism and 
incentive-blind institutionalism.

The kit is opinionated by design. It is not meant to be a neutral
toolkit; if you fork it, expect to rewrite the identity layer to 
match your own commitments.

---

## How Markus works

Markus runs on Claude via the **Claude Project** feature.

- **Identity files** (`Markus.md`, `Principles.md`) and **core
  knowledge files** (toolkit references, data sources, tools and
  integrations) are attached as **Project knowledge** so they
  load every session.
- The **system prompt** for the Project is set from `Markus.md`.
- **Workflow files** and **templates** live in the vault and are
  invoked explicitly when needed (e.g., *"Run the regression —
  full workflow per `run-a-regression-properly.md`"*).
- **Econometrics and math depth notes** are too long to all live
  as standing context. Markus reaches for them via explicit
  invocation (*"we're doing a shift-share design, load
  `shift-share-instruments.md`"*) or by the researcher pasting
  the relevant file at the start of a session.
- **Continuity across sessions** is handled by a project layer:
  each ongoing project has a `README.md` (the brief), a `log.md`
  (reverse-chronological session log), and a session-start
  ritual that loads them into the conversation. This is the
  workaround for Claude's lack of persistent memory.

The discipline is simple: **if it isn't in the README or the
log, it doesn't exist for the next session.**

---

## Repository structure

```
Markus/
├── CLAUDE.md                                       ← build handoff
├── 00_Identity/
│   ├── Markus.md                                   ← who he is (system prompt)
│   └── Principles.md                               ← how he thinks
├── 10_Methods/
│   ├── modern-toolkit-references.md                ← method-to-tool bridge
│   └── Econometrics/                               ← deep method notes
│       ├── difference-in-differences.md            ← DiD (template setter)
│       ├── instrumental-variables.md               ← IV and 2SLS (Tier 1)
│       ├── shift-share-instruments.md              ← Bartik/shift-share (Tier 1)
│       ├── local-projections.md                    ← LPs and LP-IV (Tier 1)
│       └── regression-discontinuity.md             ← RDD (Tier 2)
├── 20_Math/                                        ← math depth layer
│   ├── real-analysis.md                            ← foundations
│   ├── measure-theory.md                           ← Lebesgue integration
│   ├── optimization.md                             ← convexity, KKT, duality
│   ├── functional-analysis.md                      ← Banach/Hilbert, operators
│   └── linear-algebra.md                           ← projections, decompositions
├── 30_Data/
│   └── AI-accessible-data-sources.md               ← where data comes from
├── 40_Code/
│   └── tools-and-integrations.md                   ← what he reaches for
├── 50_Workflows/
│   ├── run-a-regression-properly.md                ← regression procedure
│   ├── new-dataset-intake.md                       ← data inspection procedure
│   └── replication-audit.md                        ← replication & reconstruction
├── 60_Projects/
│   └── _templates/
│       ├── README.md                               ← project brief template
│       ├── log.md                                  ← session log template
│       └── session-start.md                        ← continuity ritual
└── 70_Writing/                                     ← (planned, v0.3)
```

---

## Current features (v0.2-in-progress)

### Identity layer

- **`Markus.md`** — The master prompt. Names the literatures and
  traditions Markus works in (HANK, the modern DiD revolution,
  the Piketty/Saez/Zucman program, monopsony, varieties of
  capitalism), spells out methodological habits, lists what he
  refuses to do silently, and authorizes him to push back on the
  researcher.
- **`Principles.md`** — Eleven essay-form sections on questions,
  identification, data, models, estimation, causation,
  distribution, the social-science boundary, uncertainty and
  honesty, the practice of research, and the relationship with
  the researcher.

### Knowledge layer

- **`modern-toolkit-references.md`** — A method-to-tool bridge.
  For each method in the modern empirical toolkit (DiD, IV, RDD,
  panel, time series, DML, structural, distributional, labor
  structural, text, spatial), the canonical paper(s), the
  recommended library, what the method assumes, and how it gets
  misused.
- **`AI-accessible-data-sources.md`** — Organized by question
  rather than publisher. Covers macro time series, inequality
  and distribution (the longest section — WID, LIS, OECD IDD,
  the Auten–Splinter dispute, EU-SILC), labor microdata, trade
  and firms, long-run macro, central bank, and policy text. Has
  a Turkish-sources section (TÜİK, CBRT, SGK, BDDK) with
  structural breaks and methodology caveats.
- **`tools-and-integrations.md`** — Curated working knowledge of
  libraries and AI integrations. Built with reference to
  `hanlulong/awesome-ai-for-economists` but independently
  curated: vendor SaaS demoted, gaps for distribution and labor
  filled, and an explicit "do not use by default" section for
  end-to-end AI research agents.

### Workflow layer

- **`run-a-regression-properly.md`** — Nine-stage procedure from
  question restatement through interpretation. Robustness checks
  framed as *"pick checks that would change your mind if they
  failed."* Includes a compressed mode for routine work with
  explicit non-skippable items.
- **`new-dataset-intake.md`** — Eleven-step procedure producing
  a **dataset card** as the deliverable. The card has a "not
  suitable for" section that's more important than the "suitable
  for" section. Includes stop conditions.
- **`replication-audit.md`** — Procedure for auditing empirical
  papers. Two branches: replication (running someone's code) and
  reconstruction (rebuilding from text alone). Both produce a
  written replication report. Explicit about the honesty
  challenges of uncomfortable findings on respected papers.

### Project layer

- **Project brief template**, **session log template**, and
  **session-start ritual** — the continuity mechanism for
  multi-session projects.

### Econometrics depth layer (`10_Methods/Econometrics/`)

Deep, opinionated notes (~3,500–5,000 words each) on methods that
show up in macro/labor/distribution work. Each follows the same
structure: estimand first, naive approach critiqued, modern
estimators covered, decision tree, load-bearing assumption treated
honestly, failure modes, what to report, and implementation by
language (R as default).

- **`difference-in-differences.md`** — The template-setter for
  the layer. Covers the TWFE problem (Goodman-Bacon
  decomposition), the five modern DiD estimators
  (Callaway-Sant'Anna, de Chaisemartin-D'Haultfœuille,
  Sun-Abraham, Borusyak-Jaravel-Spiess, Gardner two-stage), a
  decision tree, parallel-trends sensitivity analysis
  (Rambachan-Roth), and inference.

- **`instrumental-variables.md`** — LATE and compliers as the
  estimand (refusing the constant-effects fiction). Modern
  weak-instrument inference (the F > 10 rule is dead;
  Lee-McCrary-Moreira-Porter, Olea-Pflueger effective F,
  Anderson-Rubin intervals). Judge/examiner designs. Honest
  treatment of overidentification under heterogeneous effects.
  Plausibly-exogenous bounds (Conley-Hansen-Rossi). Includes a
  TS2SLS subsection for two-sample IV — relevant when Y and D
  live in different datasets.

- **`shift-share-instruments.md`** — The Goldsmith-Pinkham-
  Sorkin-Swift (exogenous shares) and Borusyak-Hull-Jaravel
  (exogenous shocks) camps treated as substantively different
  identification stories, not stylistic preferences.
  Adão-Kolesár-Morales inference treated as mandatory. Turkish
  context section on why both stories are harder to defend in
  small-economy settings.

- **`local-projections.md`** — Jordà (2005) and why. The
  Plagborg-Møller–Wolf (2021) equivalence result (LP and VAR
  target the same IRF in population; the choice is a
  finite-sample question). The Montiel Olea–Plagborg-Møller
  (2021) long-horizon bias correction. LP-IV horizon by horizon.
  State-dependent LPs. Panel LPs with Nickell bias. Turkish
  context on single-country vs panel LP trade-offs.

- **`regression-discontinuity.md`** — The CCT
  (Calonico-Cattaneo-Titiunik 2014, *Econometrica*)
  bias-corrected inference procedure as the centerpiece: always
  report the "Robust" CI, not the "Conventional" one. Density
  tests (McCrary 2008; Cattaneo-Jansson-Ma 2020 `rddensity` as
  the modern standard). Fuzzy RDD as local Wald / LATE for
  compliers at the cutoff. "Continuity assumption, honestly."
  Decision tree (Q1–Q6), ten failure modes, seven-item what to
  report.

### Math depth layer (`20_Math/`)

Working mathematical notes (~5,000–8,000 words each) connecting
the math to its uses in economics. Proofs of non-trivial theorems
included inline — the kit is meant to be self-contained, not a
pointer to textbooks.

- **`real-analysis.md`** — Real numbers and completeness, metric
  spaces and topology, continuity, differentiation, the inverse
  and implicit function theorems, the contraction mapping
  theorem, uniform convergence. Proofs of Bolzano-Weierstrass,
  the extreme value theorem, the mean value theorem, Taylor's
  theorem, the implicit function theorem, and the contraction
  mapping theorem.

- **`measure-theory.md`** — Why the Riemann integral breaks.
  Lebesgue measure and measurable functions. The Lebesgue
  integral. Full proofs of the monotone convergence theorem,
  Fatou's lemma, and the dominated convergence theorem. Five
  modes of convergence. Measure-theoretic foundations of
  probability.

- **`optimization.md`** — Convexity, KKT conditions (proved via
  Farkas' lemma), Lagrangian duality (weak and strong, Slater's
  condition), the envelope theorem (proved), Shephard's lemma,
  Roy's identity. Econometric applications (OLS, MLE, GMM,
  structural estimation). Numerical optimization with a detailed
  failure-modes section.

- **`functional-analysis.md`** — Banach and Hilbert spaces.
  $L^p$ completeness (Riesz-Fischer, proved via MCT). The
  projection theorem (proved via parallelogram law; OLS as
  finite-dimensional projection). The Riesz representation
  theorem (proved). The big four theorems (Baire category,
  uniform boundedness, open mapping, closed graph, Hahn-Banach).
  Radon-Nikodym proved via Riesz representation. Compact
  operators and the spectral theorem. Conditional expectation as
  $L^2$ projection.

- **`linear-algebra.md`** — Projection geometry and OLS (hat
  matrix, FWL theorem, leverage). Spectral theorem for real
  symmetric matrices (full proof). PCA and VAR stability. QR
  via Gram-Schmidt (proved). SVD (Eckart-Young, low-rank
  approximation). Moore-Penrose pseudoinverse. Cholesky.
  Numerical linear algebra: condition numbers, the never-invert
  rule, decomposition guide, floating-point pathologies.

---

## Coming in v0.2

### Econometrics depth (Tier 2 and beyond)

- **`panel-methods.md`** — Within transformations, FE vs RE,
  Mundlak/correlated random effects, dynamic panels
  (Arellano-Bond and its fragility), cluster-robust inference,
  bootstrap for few clusters. Full treatment of Nickell bias.
- **`time-series-beyond-lp.md`** — Cointegration, structural
  VARs (recursive, sign restrictions, narrative identification,
  proxy SVAR), state-space and Kalman filtering, unit root
  testing honestly.
- **`structural-labor.md`** — AKM (Abowd-Kramarz-Margolis) and
  the leave-out estimators (Kline-Saggio-Sølvsten 2020) for
  limited mobility bias.
- **`distributional-methods.md`** — Quantile regression, RIF
  (Firpo-Fortin-Lemieux), distributional decompositions
  (DiNardo-Fortin-Lemieux, the Oaxaca-Blinder family),
  distributional treatment effects.
- **`double-machine-learning.md`** — Chernozhukov et al. (2018),
  orthogonality condition, regularization bias, sample
  splitting, when DML is and isn't a license for ML in causal
  inference.

### Math depth (continuing)

- **`probability.md`** — Laws of large numbers, central limit
  theorems (classical through dependent-data extensions), the
  delta method, characteristic functions, the asymptotic
  foundations that underpin econometrics.
- **`stochastic-processes.md`** — Stationarity and ergodicity,
  Markov chains, Brownian motion, Wold decomposition, ARMA,
  long-memory processes.
- **`dynamic-programming.md`** — Principle of optimality,
  Blackwell sufficient conditions, value function iteration,
  curse of dimensionality, heterogeneous-agent models (HANK).

---

## Coming in v0.3

### Literature search

A literature search workflow (`50_Workflows/literature-search.md`)
built as a unit with an accumulating literature notes file
(`80_Literature/notes.md`). Handles tiered access, defends
against AI-search failure modes (hallucinated citations, recency
bias, Anglophone bias), and produces annotated bibliographies,
literature review drafts, or prioritized reading lists.

### Writing skills (`70_Writing/`)

Built with reference to `hanlulong/econ-writing-skill`. Likely
components: paper structure, referee management, tables and
figures, honest interpretation, and a style note on voice.

---

## Design commitments

These thread through every file and are not up for relitigation:

- **Honesty over agreement.** Markus's job is to surface what's
  true, not to soften findings or flatter the researcher.
- **Estimand before estimator.** Name what you're trying to
  estimate, in plain language, before reaching for an estimator.
- **Heterogeneity is the rule, not the exception.** Treatment
  effects vary; methods that hide this are deprecated.
- **Distribution as default, not special topic.** Most econ-AI
  tooling treats distribution as an afterthought; Markus
  doesn't.
- **Adjacent disciplines as peers.** Sociology and political
  science get engaged seriously, not patronized.
- **Turkey as an explicit context.** National data sources and
  institutional context are treated as first-class, not as an
  afterthought to the international apparatus.
- **Skepticism of black-box tools.** End-to-end "AI research
  agents" that hide identification choices are not used by
  default.
- **Stop conditions over silent push-through.** Workflows that
  touch data or external claims include explicit conditions
  under which Markus refuses to proceed and escalates.

---

## How to use it

1. Clone the repo into your Obsidian vault (or anywhere; the
   files are plain markdown).
2. Create a Claude Project on claude.ai.
3. Set the Project's custom instructions from
   `00_Identity/Markus.md`.
4. Attach as Project knowledge:
   - `00_Identity/Principles.md`
   - `10_Methods/modern-toolkit-references.md`
   - `30_Data/AI-accessible-data-sources.md`
   - `40_Code/tools-and-integrations.md`
5. For ongoing research projects, copy the templates from
   `60_Projects/_templates/` into a new project folder and run
   the session-start ritual at the beginning of each working
   session.
6. Invoke workflows and depth notes explicitly when you need
   them: *"Run the regression on this dataset — full workflow
   per `run-a-regression-properly.md`"* or *"We're doing a
   shift-share design — load `shift-share-instruments.md`."*

The kit is opinionated for a specific researcher's fields and
commitments. If your fields are different, **fork it and
rewrite the identity layer first.** Don't try to use Markus as
a generic econ assistant; he isn't one.

---

## Status

**v0.2-in-progress** — Identity, knowledge, workflow, and project
layers are stable. Econometrics depth layer: Tier 1 is complete
(DiD, IV, shift-share, local projections); Tier 2 has begun
(regression discontinuity). Math depth layer: five files complete
(real analysis, measure theory, optimization, functional analysis,
linear algebra). Literature search and writing layers are planned
for v0.3.

Things will change. Files will be revised as the methodological
literatures move (especially DiD, which is still unsettled in its
continuous-treatment and spillover branches). Breaking changes to
file structure are possible before v1.0.

---

## Contributing

Issues and discussion welcome, especially:

- Methodological errors or out-of-date references in the
  econometrics notes.
- Missing data sources, particularly for distribution and for
  national contexts other than Turkey.
- Workflow gaps that show up in real use.
- Disagreements with the design commitments, if argued
  substantively.

Pull requests welcome but please open an issue first for
anything beyond a typo fix — the kit is opinionated and I'd
rather discuss direction before reviewing code.

---

## Acknowledgments

Built with reference to:
- `hanlulong/awesome-ai-for-economists` (tooling layer, curated
  independently).
- `hanlulong/econ-writing-skill` (writing layer, planned).
- The methodological papers cited throughout the econometrics
  notes — especially the post-2018 DiD literature and the
  modern RD inference literature, which made this kit feel
  necessary.

And built in collaboration with Claude, 
which is also what runs it.
