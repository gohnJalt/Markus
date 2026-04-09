# CLAUDE.md — Markus build handoff

This document is the handoff for any new Claude session that picks
up the work of building Markus, the personalized research assistant
brain kit. It captures who Markus is, what's been built, what the
design choices have been, and what comes next. A fresh instance
should be able to read this file and continue construction without
losing the thread of what came before.

If you are Claude reading this in a new chat: read the whole file
before doing anything else. The choices made earlier in the build
were deliberate and the consistency of Markus's character matters.
Don't propose redesigns of decisions that have already been made
without checking the rationale first.

---

## What Markus is

Markus is a Claude-based research assistant, structured as an
Obsidian-vault "brain kit," built for an economist whose fields are
**macroeconomics, labor economics, and the economics of distribution**,
with deep methodological grounding in **theoretical and applied
econometrics** and serious working literacy in **sociology and
political science** as peer disciplines.

The researcher works in Turkey, which means national data sources
(TÜİK, CBRT, SGK, BDDK) and the Turkish institutional context
matter alongside the standard international apparatus.

Markus is meant to function as a PhD-level collaborator, not a
generic assistant. He has opinions, pushes back, refuses to do
some things silently (regression without diagnostics, TWFE on
staggered designs, treating R² as causal evidence), and updates
when shown wrong. He treats sociology and political science as
peer disciplines rather than soft cousins. He is suspicious of
both economic imperialism and incentive-blind institutionalism.

The kit runs on Claude via the Claude Project feature. Identity
and knowledge files are attached as Project knowledge so they
load every session. Workflow files and templates are referenced
when invoked. Continuity across sessions is handled through a
project layer with READMEs, logs, and a session-start ritual.

The kit is currently at **v0.1-alpha**. A public README exists
for the GitHub repo and is the right entry point for anyone
encountering the kit for the first time.

---

## What's been built

The kit currently has twelve substantive files plus a CLAUDE.md
and a public README, across five layers. The structure inside
the Obsidian vault:

```
Markus/
├── CLAUDE.md                                       ← this file
├── README.md                                       ← public-facing
├── 00_Identity/
│   ├── Markus.md                                   ← who he is
│   └── Principles.md                               ← how he thinks
├── 10_Methods/
│   ├── modern-toolkit-references.md                ← method-to-tool bridge
│   └── Econometrics/
│       └── difference-in-differences.md            ← deep DiD note
├── 20_Math/                                        ← (planned, v0.2)
├── 30_Data/
│   └── AI-accessible-data-sources.md               ← where data comes from
├── 40_Code/
│   └── tools-and-integrations.md                   ← what he reaches for
├── 50_Workflows/
│   ├── run-a-regression-properly.md                ← regression procedure
│   ├── new-dataset-intake.md                       ← data inspection procedure
│   └── replication-audit.md                        ← replication AND reconstruction
├── 60_Projects/
│   └── _templates/
│       ├── README.md                               ← project brief template
│       ├── log.md                                  ← session log template
│       └── session-start.md                        ← continuity ritual
└── 70_Writing/                                     ← (planned, v0.3)
```

### Identity layer

**`00_Identity/Markus.md`** — The master prompt.
It states who Markus is, names the literatures and traditions he
works in (HANK, the modern DiD revolution, Piketty/Saez/Zucman
program, monopsony, varieties of capitalism), spells out his
methodological habits (estimand vs estimator, units sacred,
heterogeneity is the rule, mechanisms matter, distribution as
default not special topic), specifies the work format for
analysis tasks, lists the things he refuses to do silently, and
authorizes him to push back on the researcher and to engage
seriously with sociology and political science. This file is
the constitution.

**`00_Identity/Principles.md`** — The long-form companion to the
system prompt. Eleven sections covering questions, identification,
data, models, estimation, causation, distribution, the social-
science boundary, uncertainty and honesty, the practice of
research, and the relationship with the researcher. Written as
an essay rather than a checklist because principles followed
mechanically miss their own point. This is the case law to the
system prompt's constitution. Markus reaches for it when a
principle from the system prompt needs unpacking.

### Knowledge layer

**`10_Methods/modern-toolkit-references.md`** — A reference
file (not a reading) that bridges methods to tools. For each
method in the modern empirical toolkit (DiD, IV, RDD, panel,
time series, DML, structural, distributional, labor structural,
text, spatial), it lists the canonical paper(s), the
recommended library implementation, what the method assumes in
one or two sentences, and the most common ways it gets misused.
Particularly thorough on the post-2018 DiD literature (Goodman-
Bacon, Callaway-Sant'Anna, de Chaisemartin-D'Haultfœuille, Sun-
Abraham, Borusyak-Jaravel-Spiess) because that's where the
field has moved most. The file is meant to be consulted, not
read top to bottom.

**`10_Methods/Econometrics/difference-in-differences.md`** — The
first deep econometrics note, drafted in the v0.1→v0.2
transition. ~3,600 words. Covers the estimand framing, the 2×2
case, the TWFE problem in detail (Goodman-Bacon decomposition),
the five modern estimators (Callaway-Sant'Anna, de Chaisemartin-
D'Haultfœuille, Sun-Abraham, Borusyak-Jaravel-Spiess, Gardner
two-stage), a decision tree for choosing between them, parallel
trends sensitivity analysis (Rambachan-Roth as the standard),
inference (clustering, few clusters, permutation), common
failure modes, what to report, and implementation by language
(R as the default). This file is the **template** for the rest
of the econometrics folder: estimand-first framing, decision
tree, common failure modes, implementation by language,
cross-references to workflows and principles. Subsequent
econometrics notes should match its grain (~3,500-5,000 words)
and structure.

**`30_Data/AI-accessible-data-sources.md`** — The data companion
to the tools file. Organized by question (macro time series,
inequality and distribution, labor microdata, trade and firms,
long-run macro, central bank, policy text) rather than by
publisher. For each source: what it covers, its known quirks,
how to access it (API, MCP, bulk, restricted), what it is and
isn't appropriate for. Distribution gets the longest section
because the awesome-list world doesn't cover it well — WID, LIS,
the OECD IDD, the Auten-Splinter dispute, EU-SILC, national
sources. Includes a starter section on Turkish sources (TÜİK,
CBRT, SGK, BDDK) with the structural breaks, the inflation
context, and the contested-methodology caveats. Explicit about
what Markus *cannot* fetch (administrative microdata, restricted
sources) and why.

**`40_Code/tools-and-integrations.md`** — Markus's working
knowledge of libraries and AI integrations relevant to empirical
economics. Built with reference to the `hanlulong/awesome-ai-for-
economists` repo but curated rather than copied: high-leverage
items kept (MCP servers, modern causal inference libraries,
forecasting tools, document processing), vendor SaaS demoted, and
an explicit "do not use by default" section for end-to-end "AI
research agents" because they hide identification choices Markus
doesn't trust to a black box. Includes pyfixest, fixest,
sequence-jacobian, lpirfs, Quarto, and other things the original
list missed for these specific fields.

### Workflow layer

**`50_Workflows/run-a-regression-properly.md`** — The procedure
Markus follows when asked to run a regression. Nine stages from
"understand what's being asked" through "interpret carefully."
Stage 0 is question restatement; Stage 1 is naming the estimand
in plain language; Stage 4 ("specify identification, not just the
equation") is the longest because that's where the difference
between a generic stats assistant and an econometrics collaborator
lives. Includes a compressed mode for routine work, with an
explicit list of things Markus doesn't skip even in compressed
mode (units check, no TWFE on staggered, no stars, no calling
correlations causal). Robustness checks are framed as "pick
checks that would change your mind if they failed," which is the
opposite of standard practice and the file's strongest commitment.

**`50_Workflows/new-dataset-intake.md`** — The procedure Markus
runs the first time he sees a dataset. Eleven steps from
provenance through joins, with the deliverable being a **dataset
card** (a short markdown document saved alongside the data,
capturing what the next user needs to know). The card has a
"not suitable for" section that's more important than the
"suitable for" section, because misuse warnings are what
prevent downstream errors. Step 6 (units, frequencies,
deflators) is the highest-value single step and is non-skippable
even in compressed mode. Includes explicit stop conditions —
when Markus should refuse to proceed and surface a problem to
the researcher rather than pushing through.

**`50_Workflows/replication-audit.md`** — The procedure for
auditing someone else's empirical paper. Originally written
assuming a code package exists, then extended with a major
section ("When there is no replication package") that handles
the more common case: reconstruction from text alone. The
distinction between *replication* (running someone's code) and
*reconstruction* (rebuilding from the text) matters because the
standards of success, the failure modes, and what the audit
licenses are all different. The file is explicit about when a
reconstruction should be abandoned. Both branches feed into a
written replication report as the deliverable, and the file
discusses the specific honesty challenges that replications
raise (uncomfortable findings about respected papers).

### Project layer

**`60_Projects/_templates/README.md`** — Template for a project
brief, copied (not transcluded) into each new project folder.
Has sections for question, estimand, contribution, audience,
data, identification strategy, methods, current status, open
questions, and a "Decisions made" section that serves as the
project's institutional memory against contradictory choices in
later sessions.

**`60_Projects/_templates/log.md`** — Template for the running
session log. Reverse chronological (newest first) so the most
recent state is always at the top, which is what the session
start ritual most needs to load. Each entry has a tight format:
what worked on, what changed, what found, what's open, next
session.

**`60_Projects/_templates/session-start.md`** — The continuity
ritual. Three variants: starting a session on an existing
project (paste README + recent log entries + relevant dataset
cards, then have Markus confirm orientation by summarizing
back), starting a new project (the first session produces the
README as its deliverable), and ending a session (Markus drafts
the log entry, researcher reviews and appends). The file is
explicit that Claude has no persistent memory between sessions
and that this ritual is the workaround. The discipline: if it
isn't in the README or the log, it doesn't exist for the next
session.

### Public-facing

**`README.md`** — The public README for the GitHub repo. ~1,600
words. Opens with the alpha warning, explains what Markus is and
how he works, shows the repo structure (with planned folders
marked), lists current features by layer, lists coming features
under v0.2 and v0.3, restates the design commitments, and gives
a six-step setup for would-be users. Explicit that the kit is
opinionated for a specific researcher and that forks should
rewrite the identity layer first. Includes placeholders for
license and contact info.

---

## Design choices that hold across the kit

These are the recurring decisions that should not be relitigated
in later files unless we have a specific reason to revisit them.

**Voice and tone.** Markus's files are written as essays where
they are essays and as references where they are references.
Identity and Principles are essays. Tools, Data sources, and
Modern toolkit references are references. Workflows are
procedures. Templates are forms. The format matches the use,
not a uniform house style. The prose is opinionated, willing
to exclude things, and explicit about why — because neutral
documentation is less useful than curated documentation when
the goal is to give Markus things to *reach for*.

**The "compressed mode" pattern.** The two long workflow files
(regression and intake) end with explicit compressed-mode
sections that name what can be skipped for routine work and what
can never be skipped even when moving fast. This pattern keeps
the workflows usable in practice. Future workflow files should
follow it.

**Stop conditions.** Both data intake and replication audit
include explicit stop conditions — situations where Markus
should refuse to proceed and escalate to the researcher rather
than push through. This is a deliberate design choice and
should be standard for any workflow that touches data or
external claims.

**Honesty over agreement.** The system prompt, principles file,
and especially the replication audit file are explicit that
Markus's job is to surface what's true, not to soften findings
or flatter the researcher. This is the most important value
commitment in the kit and it threads through every file. New
files should reinforce it, not erode it.

**Adjacent disciplines as peers.** Sociology and political
science are treated as peer disciplines throughout, not as
soft cousins. The system prompt names this explicitly; the
principles file has a section on it; the regression workflow
tells Markus to bring in adjacent-discipline framings in
interpretation. This is one of the things that makes Markus
*this researcher's* assistant rather than a generic econ tool.

**Distribution is not a special topic.** The data file gives
distribution its longest section. The principles file makes
"distribution is the default" a methodological commitment.
The toolkit references file has a full inequality section.
This is deliberate: most econ-AI tooling treats distribution
as an afterthought, and the kit corrects for that.

**Turkey as an explicit context.** The data sources file has
a Turkish section with the specific quirks (TÜİK methodology
disputes, 2005 lira redenomination structural break, the
inflation context that makes real-vs-nominal nearly mandatory,
SGK as the serious labor source). New files should add to this
where relevant rather than treating Turkey as an afterthought.

**Skepticism of black-box tools.** End-to-end "AI research
agents" that hide identification choices are explicitly listed
as things Markus does not use by default. Aggregator MCP servers
are flagged as convenient for exploration but not the source of
record. This skepticism is consistent with the principles file's
commitments around identification and data.

**Reference to the awesome-list repo.** The kit was built with
reference to `hanlulong/awesome-ai-for-economists` for the
tooling section, but the curation is independent: vendor self-
promotion in the original was demoted, gaps for distribution
and labor were filled, and several tools that mattered for these
fields (pyfixest, sequence-jacobian, the modern DiD ecosystem)
were added that weren't in the original.

**The DiD note as template.** The DiD file establishes the
shape of an econometrics depth note: ~3,500-5,000 words,
estimand-first framing, walks from the simple case through the
problems with naive approaches through the modern estimators,
includes a decision tree, a parallel-trends-equivalent honesty
section (whatever the load-bearing assumption is), inference
discussion, common failure modes, what to report, and
implementation by language with R as the default. Subsequent
econometrics notes should match this grain and structure
without rigidly copying it.

---

## Where Markus lives operationally

The intended setup is as a Claude Project. The identity files
(`Markus.md` and `Principles.md`) and the three knowledge files
(`modern-toolkit-references.md`, `AI-accessible-data-sources.md`,
`tools-and-integrations.md`) are attached as Project knowledge
so they load every session. The system prompt also goes into the
Project's custom instructions.

The workflow files and templates live in the Obsidian vault and
are referenced when needed. The most powerful invocation pattern
is explicit reference: "Run the regression — full workflow per
`run-a-regression-properly.md`, not compressed" tells Markus to
walk the nine stages explicitly rather than defaulting to a
quicker mode.

For ongoing projects, the session start ritual loads the
relevant project README, recent log entries, and dataset cards
into the conversation at the start of every working session.
This is the workaround for Claude's lack of persistent memory
between sessions, and it's the discipline that makes the
project layer actually work.

As the econometrics and math libraries grow in v0.2, individual
deep notes from those folders will be loaded into context on
demand rather than as Project knowledge — they're too long to
all live as standing context. Markus reaches for them when a
discussion calls for them.

---

## v0.2 — Econometrics depth and Math depth

v0.2 is about depth: two libraries built out in parallel
(econometrics and math), both substantial, both following the
pattern the DiD note established. v0.2 ships when both
libraries reach a workable critical mass — not when every
planned file is written, but when the core methods and the core
mathematical foundations are in place.

### Econometrics library (`10_Methods/Econometrics/`)

Each file matches the DiD note's grain (~3,500-5,000 words,
estimand-first, decision tree, failure modes, implementation
by language). Each file is *deeper* than its directory entry
in `modern-toolkit-references.md` — that file is a directory;
this folder is the actual library.

**Tier 1 — current project priorities, build in this order:**

1. **`instrumental-variables.md`** — IV and 2SLS. Assumes 2SLS
   mechanics (this is not a textbook). Focused on the modern
   weak-instrument literature (Andrews-Stock-Sun, the AR test
   as default, retiring the F>10 rule, the heteroskedasticity-
   robust first-stage F from Olea-Pflueger), the LATE/compliers
   framing, overidentification tests honestly, and the move
   from "find an instrument" to "design-based IV." The
   judge/examiner-design literature gets a section. Cross-
   references the DiD note's discussion of treatment-effect
   heterogeneity since the LATE story picks up there.

2. **`shift-share-instruments.md`** — Bartik instruments, as a
   standalone file rather than folded into IV. The two camps:
   the exposure-design framing (Borusyak-Hull-Jaravel 2022) and
   the shares-design framing (Goldsmith-Pinkham-Sorkin-Swift
   2020). When each is appropriate, what each assumes, the
   inference machinery for each, and the recent applications.
   Cross-references the IV file heavily.

3. **`local-projections.md`** — Jordà (2005), the LP-vs-VAR
   debate, Plagborg-Møller and Wolf (2021) showing they
   estimate the same IRFs in population, LP-IV (Stock-Watson
   2018, Ramey 2016), bias and inference issues at long
   horizons (Montiel Olea and Plagborg-Møller 2021), smooth
   local projections (Barnichon-Brownlees), panel LP. This is
   the file the researcher's current project most directly
   needs and the natural endpoint of Tier 1.

**Tier 2 — core methods every applied econometrician needs:**

4. **`regression-discontinuity.md`** — Cattaneo, Idrobo, and
   Titiunik as the canonical modern reference. Bandwidth
   selection (Imbens-Kalyanaraman, Calonico-Cattaneo-Titiunik),
   bias-corrected inference, density tests (McCrary, Cattaneo-
   Jansson-Ma), fuzzy RDD, kink designs, multi-cutoff and
   multi-score, and the "RDD is not a magic bullet" failure
   modes.

5. **`panel-methods.md`** — Beyond what's implicit in DiD.
   Within transformations, between estimators, random vs. fixed
   effects (and why the Hausman test is overrated), Mundlak and
   correlated random effects, dynamic panels (Arellano-Bond,
   Blundell-Bond and their fragility), modern cluster-robust
   inference, the bootstrap for few clusters.

6. **`time-series-beyond-lp.md`** — Cointegration (Johansen,
   the Engle-Granger two-step, modern critiques), structural
   VARs (recursive, sign restrictions à la Uhlig, narrative
   identification, external instruments / proxy SVAR),
   state-space and Kalman filtering at a working level, unit
   root testing honestly (the power problem). May split into
   "VAR identification" and "non-stationary time series" if it
   grows past 6k words.

**Tier 3 — fields-specific depth:**

7. **`structural-labor.md`** — AKM (Abowd, Kramarz, Margolis)
   and the leave-out estimators (Kline, Saggio, Sølvsten 2020)
   for limited mobility bias. Connected sets, identification of
   firm and worker effects, the recent literature on what AKM
   does and doesn't tell you about firm pay premia. Connects
   directly to the researcher's distribution work.

8. **`distributional-methods.md`** — Quantile regression
   (Koenker, Chernozhukov), recentered influence functions
   (Firpo-Fortin-Lemieux), distributional decompositions
   (DiNardo-Fortin-Lemieux, the Oaxaca-Blinder family
   generalized), and the modern distributional treatment
   effects literature (Athey-Imbens, Callaway-Li). This is the
   file that operationalizes "distribution as default."

9. **`double-machine-learning.md`** — Chernozhukov et al.
   (2018), the conditions under which it works, sample
   splitting, when to reach for it instead of conventional
   methods, and the failure modes. The orthogonality condition
   is doing real work; regularization bias is real; DML is not
   a license to use ML for causal inference without thinking.

**Tier 4 — additions worth considering, build as gaps appear:**

10. **`synthetic-control.md`** — Abadie-Diamond-Hainmueller,
    Doudchenko-Imbens, Ben-Michael-Feller-Rothstein augmented
    SC, synth-DiD (Arkhangelsky et al.). The DiD note mentions
    this in passing; it deserves its own treatment because the
    inference story is genuinely different.
11. **`spatial-and-network.md`** — When SUTVA fails. Spatial
    autoregressive models, the Manski reflection problem,
    network peer effects (Bramoullé-Djebbari-Fortin), and the
    recent design-based literature on spillovers (Aronow-Samii,
    Vazquez-Bare).
12. **`bayesian-macro.md`** — BVARs (Litterman, the Minnesota
    prior, modern hierarchical priors), Bayesian model
    averaging, MCMC at a working level, the SMC sampler.
    Useful for the macro side but not urgent.

### Math library (`20_Math/`)

Working notes for an applied econometrician, not textbook
substitutes. The level should be "Markus reasons fluently
about why a method works, not Markus rederives the proofs."
The audience is Markus himself, mid-conversation, when he
needs to ground a methodological choice.

**Design choice for the math library (resolved):** these are
*thinner notes that link to canonical sources for the heavy
proofs*, not self-contained textbook chapters. The point is
the connecting work between math and its uses in economics.
Where Markus needs the proof he points to Hansen, Sargent-
Stachurski, MWG, or whoever the canonical source is. This
keeps the math library aligned with the kit's stance and
avoids competing with textbooks Markus can't replace.

**Order, build in this sequence:**

1. **`optimization.md`** — Most directly useful for estimation.
   Convex optimization (the basics: convexity, KKT, duality,
   why OLS and MLE are convex problems and why GMM usually
   isn't), Newton and quasi-Newton methods, the difference
   between local and global optima and when it matters,
   constrained optimization for structural estimation, and a
   section on numerical optimization gotchas (starting values,
   scaling, the difference between "didn't converge" and
   "converged to garbage"). Then dynamic optimization at an
   applied level: Bellman equations, value function iteration,
   policy function iteration, the envelope theorem as it
   actually gets used.

2. **`linear-algebra.md`** — For high-dimensional methods and
   for the geometric intuition behind regression. Vector
   spaces and projections (regression as projection onto a
   subspace), matrix decompositions (QR, SVD, Cholesky and
   when each is the right tool), eigenvalues and eigenvectors
   as they show up in PCA and VAR stability, the Moore-Penrose
   pseudoinverse, and a section on numerical linear algebra
   (conditioning, why you don't invert matrices in code, the
   difference between solving and inverting).

3. **`probability-and-measure.md`** — For inference. Probability
   spaces at the level needed (sigma-algebras as "the events
   you can talk about," not as a foundational exercise),
   random variables and distributions, modes of convergence
   (in probability, in distribution, almost sure, in L²) and
   why they matter for asymptotics, laws of large numbers,
   central limit theorems (the classical one and the ones that
   actually get used: Lindeberg-Feller, the martingale CLT, the
   CLT for dependent data), and the delta method honestly. A
   section on conditional expectation as projection ties this
   back to the linear algebra file.

4. **`calculus.md`** — Refresher with an econometric bent.
   Multivariable calculus, the implicit function theorem (which
   shows up everywhere in comparative statics and in
   identification arguments), Taylor expansions and how they
   ground asymptotic arguments, the envelope theorem (cross-
   reference to optimization), and a section on the calculus
   of variations and optimal control as they show up in dynamic
   problems.

5. **`stochastic-processes.md`** — For time series and for
   structural macro. Stationarity and ergodicity, Markov chains
   at a working level, Brownian motion and the basics of
   stochastic calculus (enough to read a continuous-time macro
   paper without panicking), the Wold decomposition, ARMA
   processes and their state-space representations, and a
   section on long-memory and unit-root processes.

6. **`dynamic-programming.md`** — Picking up where the
   optimization file left off. The principle of optimality,
   contraction mapping arguments, the Blackwell sufficient
   conditions, value function iteration with continuous state
   spaces (interpolation, projection methods, collocation), the
   curse of dimensionality and the methods that mitigate it
   (sparse grids, deep learning approximations à la Maliar-
   Maliar-Winant), and a section on heterogeneous-agent models
   that connects to HANK.

**Tier 2 / additions worth considering:**

7. **`functional-analysis.md`** — Hilbert spaces (because
   that's what L² is), operators (because that's what
   conditional expectation is), and the basics of nonparametric
   estimation viewed through this lens. Useful for understanding
   sieve estimation, kernel methods, and the recent ML-
   econometrics interface.
8. **`information-theory.md`** — Entropy, KL divergence, mutual
   information, and how they show up in model selection, in the
   EM algorithm, and in ML methods Markus might encounter.
9. **`numerical-methods.md`** — A stand-alone file on the
   numerical issues that bite applied work: floating-point
   arithmetic, conditioning, why your standard errors changed
   when you reordered the variables, and the gap between
   mathematical and numerical equivalence. Could be folded into
   the linear algebra file but probably deserves its own.

### Recommended sequence for v0.2

The two libraries get interleaved rather than built sequentially.
Econometrics Tier 1 leads because it serves the researcher's
current project; the math library starts in parallel once Tier
1 is underway.

1. Econometrics: `instrumental-variables.md`
2. Econometrics: `shift-share-instruments.md`
3. Econometrics: `local-projections.md`
4. Math: `optimization.md` *(begin interleaving)*
5. Econometrics: `regression-discontinuity.md`
6. Math: `linear-algebra.md`
7. Econometrics: `panel-methods.md`
8. Math: `probability-and-measure.md`
9. Econometrics: `time-series-beyond-lp.md`
10. Math: `calculus.md`
11. Econometrics: `structural-labor.md`
12. Math: `stochastic-processes.md`
13. Econometrics: `distributional-methods.md`
14. Econometrics: `double-machine-learning.md`
15. Math: `dynamic-programming.md`

**→ v0.2 release.**

Tier 4 econometrics files and Tier 2 math files build as gaps
appear, not speculatively. The release is when the listed core
is in place, not when every possible file exists.

---

## v0.3 — Capability layers

v0.3 is about capability rather than knowledge: two new things
Markus can *do* — run trustworthy literature searches and write
in econ conventions and the researcher's voice.

### Literature search and literature notes (built as a unit)

Built together as a unit because the workflow generates entries
and the entries become context for future workflows. Building
the workflow without the storage layer means searches don't
accumulate and the kit doesn't get smarter over time.

**`50_Workflows/literature-search.md`** — The procedure Markus
follows when asked to search the literature. The workflow is
parameterized over **constraints** that the researcher specifies
at the start of each session and that lock for the duration:

- **Scope constraints** — time window, paper types (working
  papers, journal articles, books), specific outlets.
- **Quality constraints** — journal tier, predatory-journal
  exclusion, working paper handling.
- **Disciplinary constraints** — pure econ vs. with sociology
  vs. with political science, which sub-branches included.
- **Methodological constraints** — design-based only,
  structural only, theoretical only, etc.
- **Linguistic constraints** — English plus Turkish where
  TÜİK and CBRT working papers are relevant.

The workflow handles **tiered access** explicitly because
Markus's reach varies by source:

- **Tier A — direct retrieval.** Open repositories Markus can
  search and fetch directly: NBER, SSRN, RePEc/IDEAS, arXiv
  econ, CEPR DPs, IZA DPs, central bank working papers, Google
  Scholar (with throttling caveats).
- **Tier B — researcher-mediated retrieval.** Institutional
  library access (JSTOR, ScienceDirect, Wiley, Project MUSE)
  and specialized databases (EconLit, Web of Science, Scopus).
  Markus identifies what to retrieve and explains why; the
  researcher fetches; Markus reads what gets uploaded.
- **Tier C — feeders.** Other AI-native search tools (Elicit,
  Consensus, Scite, Undermind, Perplexity research mode) used
  for retrieval only, never for summary. Markus does the
  reading.

The workflow supports multiple **deliverables**, specified at
the start along with constraints:

- **Annotated bibliography** — list of papers with one-paragraph
  entries, for orienting in a new literature.
- **Literature review draft** — synthesized prose that situates
  papers in relation to each other, for use as a paper section.
- **Reading list with priority ordering** — papers ranked by
  essentialness, for "what should I read this week."
- **Citation network map** — who cites whom, where the field
  forks, for understanding intellectual lineage.
- **Targeted answer to a specific question** — for when the
  search is in service of a specific argument.

The workflow defends explicitly against the standard failure
modes of AI literature search:

- **Hallucinated citations** — every cited paper must be
  retrievable; Markus reads the actual abstract before
  including it; uncertain claims about content are flagged.
- **Citation laundering** — claims about "the literature finds"
  require multiple traceable sources; Markus distinguishes
  "X claims Y" from "the literature establishes Y."
- **Recency bias** — explicit step for finding the foundational
  papers, not just recent ones.
- **Anglophone bias** — explicit prompt to consider Turkish,
  French, German, Spanish-language work where relevant.
- **Methodological monoculture** — explicit "have the adjacent
  disciplines worked on this?" step, building on the kit's
  commitment to sociology and political science as peers.

The workflow ends with a **logging step** that adds the papers
encountered during the search to the literature notes file
below. This is the part that makes the kit accumulate knowledge
across sessions.

**`80_Literature/notes.md`** — The accumulating literature notes
file. One-paragraph entries on papers that have passed through
the literature workflow, organized by field (macro, labor,
distribution, methods, sociology of inequality, political
economy of welfare states, Turkey-specific). Each entry has a
tight format: citation, one-sentence claim, one-sentence
methodological note, one-sentence relevance to the researcher's
work, tags. Entries get added by the workflow; Markus can query
the file to orient any future discussion.

The file lives in its own folder because it's neither a method
nor a workflow nor a project — it's a personal corpus that
spans all of these.

### Writing skills (`70_Writing/`)

Built with reference to `hanlulong/econ-writing-skill` but
curated independently. Same critical posture as the tools file:
some of the 50+ source guides will be more useful than others;
some are dated; some are aimed at audiences this researcher
isn't.

**Likely files:**

- **`paper-structure.md`** — Intro/body/conclusion conventions
  in econ. The intro as a contract with the reader. The
  five-paragraph intro structure (motivation, gap, contribution,
  preview, roadmap) and when to deviate from it. Section
  conventions in the body. Conclusion as something other than
  a summary.
- **`referee-management.md`** — Responding to referees,
  requesting revisions, the referee report as a genre (both
  reading them and writing them). The reply letter as a genre
  with its own conventions. When to push back on a referee and
  how.
- **`tables-and-figures.md`** — The design rules and why. What
  goes in a table vs. a figure. The Schwabish/Cleveland
  conventions. Stars are out, confidence intervals are in.
  The figure-first paper as an organizing principle.
- **`honest-interpretation.md`** — Cross-references
  `Principles.md` heavily. How to write results sections that
  don't oversell, don't undersell, and don't launder
  uncertainty. The difference between "consistent with" and
  "establishes." Hedging that means something vs. hedging that
  doesn't.
- **`voice.md`** — The distinction between the researcher's own
  voice (for their own work, for academic outputs) and Markus's
  collaboration voice (for internal drafts, for working notes).
  These are different and the file makes the difference
  explicit.

**Conditional additions, build if the researcher actually does
these things:**

- **`turkish-language-writing.md`** — If the researcher writes
  in Turkish for some outputs.
- **`policy-writing.md`** — If the researcher writes for policy
  audiences. Different genre conventions, different audience,
  different rules about hedging and uncertainty.
- **`rebuttal-letters-and-grants.md`** — Rebuttal letters and
  grant applications as their own genres.
- **`presentations.md`** — The slide deck as a genre, very
  different from a paper. Conference presentations vs. seminar
  presentations vs. job talks.

### Sequence for v0.3

1. `literature-search.md` and `80_Literature/notes.md` built
   together as a unit.
2. Writing skills layer in the order above (paper-structure
   → referee-management → tables-and-figures → honest-
   interpretation → voice).
3. Conditional writing files as the researcher confirms which
   apply.

**→ v0.3 release.**

---

## Beyond v0.3

After the three substantive layers, the remaining build items:

- **Method-specific workflow files** beyond the three already
  built. Candidates: literature review workflow (largely
  satisfied by v0.3 above), paper draft workflow, conference
  presentation workflow, referee report workflow.
- **Context packs** in `99_Context-packs/` — ready-to-paste
  bundles for common task types, made with Obsidian
  transclusion of files already written.
- **A `refusals-guardrails.md`** in the identity folder if
  the implicit refusals scattered through the system prompt
  and workflow files turn out to need consolidation.

These are lower priority and can be built as gaps appear in
real use, rather than speculatively.

---

## How to work on this kit

A few notes on the working pattern that's served us well so
far, in case the next session is with a fresh Claude:

**The researcher leads, Claude drafts.** The researcher names
what to build next, often with specific framing. Claude drafts
the file in full. After the file content, Claude writes notes
on the design choices made — what the file is doing, what
tradeoffs were weighed, what to consider editing, how to use it
with Claude. These notes are not optional; they're how the
researcher knows why a file looks the way it does. Researcher requires
to understand what you are doing, and makes changes when it is needed,
which is regularly updated to /gohnJalt/Markus, and sometimes shared with
Claude to give context.

**Files are long but tightly structured.** A typical workflow
file is 2000-4000 words. A typical reference file is similar
or longer. The DiD note is ~3,600 words and is the template
for the econometrics folder. The length is justified by the
file being the durable artifact — Markus consults these
repeatedly, so investment in clarity pays back. But every
section should earn its place. Padding is worse than absence.

**The tone is opinionated and willing to disagree.** Earlier
files set commitments (heterogeneity is the rule, distribution
is the default, the modern DiD literature is the standard,
TWFE on staggered is deprecated except in specific cases).
Later files should reinforce these, not contradict them. If a
new file genuinely needs to revisit a commitment, surface the
reconsideration explicitly rather than quietly breaking
consistency.

**Reference to the existing files matters.** Every new file
should know what came before and link to it. The regression
workflow refers to the principles file, which refers to the
system prompt, which refers to the tools file. The DiD note
refers to the workflow and the principles. This cross-
referencing is what turns the kit into a single coherent thing
rather than a collection of separate documents.

**The researcher will ask for what's next at the end of each
file.** Standard pattern: at the end of drafting a file, Claude
recommends one or two natural next-builds with a one-sentence
rationale for each, and asks the researcher to choose. This
keeps momentum without prescribing a fixed sequence. The v0.2
sequence above is the current default but the researcher may
deviate.

**Don't write the code or run analyses unless asked.** This
build is about constructing the kit, not using it. Markus
will use the kit on real research with the researcher; the
build sessions are about the kit itself. If the researcher
asks for an example or a draft of how Markus would handle a
specific task, that's different — but don't volunteer
analyses during build sessions.

---

## Where we paused

We finished v0.1-alpha with the eleven original files plus the
first econometrics depth note (`difference-in-differences.md`)
and the public README. The researcher then planned out v0.2 and
v0.3 in detail (this document reflects that plan) and asked for
the CLAUDE.md handoff to be updated before drafting the next
file.

The next build item is **`10_Methods/Econometrics/instrumental-
variables.md`**, the first file in econometrics Tier 1. It
should follow the DiD note's template: ~3,500-5,000 words,
estimand-first framing (LATE/compliers), assume 2SLS mechanics
rather than rederiving them, decision tree, modern weak-
instrument literature as the centerpiece, judge/examiner-design
section, common failure modes, what to report, implementation
by language. Cross-reference the DiD note where the treatment-
effect heterogeneity discussion picks up.

When the next session starts, the researcher will likely paste
this file at the top, name which next item to build (probably
IV), and ask for a draft. The right move is to read this file
in full first, confirm understanding of where things stand, and
then start drafting the requested file in the established voice
and format.

---

## A short version, for the impatient

Markus is a Claude-based research assistant for an economist
working in macro, labor, and distribution, with strong
methodological commitments and serious literacy in adjacent
social sciences. He's built as an Obsidian vault that gets
attached to a Claude Project. **v0.1-alpha** is shipped:
identity, knowledge, workflow, and project layers stable, plus
the first econometrics depth note (DiD) and a public README.
**v0.2** is econometrics depth and math depth, built in
parallel, starting with IV/Bartik/local projections to serve
the researcher's current project, with math interleaved
beginning with optimization. **v0.3** is the literature search
workflow (built as a unit with an accumulating literature notes
file) and the writing skills layer. Tone is opinionated, files
are long but structured, the DiD note is the template for
econometrics depth, and honesty over agreement is the central
value. Read the existing files before drafting new ones. Ask
the researcher what's next at the end of each draft. Don't run
analyses during build sessions.

That's the kit, and that's where we are.