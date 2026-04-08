# Markus

**A research assistant brain kit for economists.**

> ⚠️ **Alpha.** Markus is under active construction. The identity,
> methodology, and workflow layers are stable; the econometrics,
> math, and writing layers are being built out. Expect things to
> change. Feedback and issues welcome.

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
│       └── difference-in-differences.md            ← (in progress)
├── 20_Math/                                        ← (planned)
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
└── 70_Writing/                                     ← (planned)
```

---

## Current features (v0.1-alpha)

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

### Econometrics depth (in progress)

- **`Econometrics/difference-in-differences.md`** — Deep note on
  the post-2018 DiD literature. Covers the TWFE problem
  (Goodman-Bacon decomposition), the five modern estimators
  (Callaway-Sant'Anna, de Chaisemartin-D'Haultfœuille,
  Sun-Abraham, Borusyak-Jaravel-Spiess, Gardner two-stage), a
  decision tree for choosing between them, parallel-trends
  sensitivity analysis (Rambachan-Roth), inference, common
  failure modes, and implementation by language.

---

## Coming soon

The next three layers, in order:

### Econometrics depth (`10_Methods/Econometrics/`)

Deep, opinionated notes on the methods that show up in
macro/labor/distribution work. Each note will be longer and more
opinionated than its directory entry in
`modern-toolkit-references.md`, with decision trees, common
failure modes, and implementation guidance.

**Planned next:**
- **Instrumental variables and 2SLS** — modern weak-instrument
  inference, shift-share / Bartik (the Borusyak-Hull-Jaravel
  and Goldsmith-Pinkham-Sorkin-Swift camps), judge/examiner
  designs, the LATE/compliers framing.
- **Local projections** — Jordà (2005), the LP-vs-VAR debate,
  Plagborg-Møller and Wolf (2021), and LP-IV.
- **Regression discontinuity** — Cattaneo, Idrobo, and Titiunik
  treatment, bandwidth selection, density tests, fuzzy RDD.
- **Panel methods** — fixed effects, modern cluster-robust
  inference, Mundlak, correlated random effects.
- **Time series** — beyond local projections: cointegration,
  structural VARs, identification by sign restrictions.
- **Structural labor** — AKM and the leave-out estimators.

### Math depth (`20_Math/`)

Working mathematical notes that connect the math to its uses in
economics — not textbook substitutes. Planned order:
optimization → linear algebra → probability → dynamic
programming. Written at the level of a serious applied
econometrician.

### Writing skills (`70_Writing/`)

Built with reference to `hanlulong/econ-writing-skill`. Likely
components: paper structure, referee management, tables and
figures, honest interpretation, and a style note on voice.
Curated rather than copied, with the same critical posture
applied to the source material as the tools file.

### And then

- Method-specific workflow files (literature review, paper
  draft, conference presentation, referee report).
- A literature notes file — running one-paragraph entries on
  papers actively shaping the researcher's thinking.
- Context packs in `99_Context-packs/` — ready-to-paste bundles
  for common task types.
- Consolidation of refusals/guardrails if the implicit ones
  scattered through the kit need a single home.

These get built as gaps appear in real use, not speculatively.

---

## Design commitments

These thread through every file. They are not up for
relitigation:

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
6. Invoke workflows explicitly when you need them:
   *"Run the regression on this dataset — full workflow per
   `run-a-regression-properly.md`."*

The kit is opinionated for a specific researcher's fields and
commitments. If your fields are different, **fork it and
rewrite the identity layer first.** Don't try to use Markus as
a generic econ assistant; he isn't one.

---

## Status

**v0.1-alpha** — Identity, knowledge, workflow, and project
layers are stable. Econometrics depth is being built (DiD note
drafted; IV, local projections, and others next). Math and
writing layers are planned but not started.

Things will change. Files will be revised as the
methodological literatures move (especially DiD, which is still
unsettled in its continuous-treatment and spillover branches).
Breaking changes to file structure are possible during alpha.

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
## License

MIT License

Copyright (c) 2026 Kaan Akkaş

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


---

## Acknowledgments

Built with reference to:
- `hanlulong/awesome-ai-for-economists` (tooling layer, curated
  independently).
- `hanlulong/econ-writing-skill` (writing layer, planned).
- The methodological papers cited throughout the econometrics
  notes — especially the post-2018 DiD literature, which made
  this kit feel necessary.

And built in collaboration with Claude, 
which is also what runs it.
