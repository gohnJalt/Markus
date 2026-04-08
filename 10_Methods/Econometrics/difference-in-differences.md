---
tags: [econometrics, causal-inference, did, panel]
related: [modern-toolkit-references, run-a-regression-properly, Principles]
---

# Difference-in-Differences

This is the working note. It assumes you know what a DiD is and
why anyone runs one. What it tries to do is give Markus enough
fluency in the post-2018 literature to *reason* about which
estimator fits a given design, rather than reciting names. The
entry in `modern-toolkit-references.md` is a directory; this is
the actual map.

The note is long. Read it linearly the first time. After that,
treat it as a reference: jump to the decision tree, the
negative-weights section, or the estimator-by-estimator breakdown
as needed.

---

## What DiD is actually estimating

The thing to hold in your head before any estimator is the
**estimand**: the average treatment effect on the treated (ATT),
or some weighted average of group-and-time-specific ATTs. DiD is
not estimating "the effect of X on Y." It is estimating a
counterfactual contrast — what happened to the treated, minus
what would have happened to the treated absent treatment, where
the latter is imputed from the untreated.

Two consequences of taking this seriously:

First, the parallel trends assumption is an assumption about
**potential outcomes**, not about pre-treatment data. Pre-trends
tests can falsify the assumption (sometimes) but cannot confirm
it. A flat pre-trend is consistent with violated parallel trends
in the post-period; a non-flat pre-trend is consistent with
parallel trends if the pre-period shock was transitory. The
pre-trend test is a sanity check, not a license. Roth (2022) is
the file to cite when this needs saying out loud — pre-trend
tests have low power against the violations that matter most,
and conditioning on passing them induces a selection problem.

Second, the comparison group is doing real work. "Control" units
are not control in the experimental sense; they are the
imputation device for the missing counterfactual. When the
imputation device is bad — because the controls are on a
different trajectory, because they're contaminated by spillovers,
because they're treated later and you're using them as if they
weren't — the estimand drifts and the estimator returns something
that may not have an interesting interpretation at all.

This is the whole game. Everything that follows is about getting
the imputation right, or being honest about when you can't.

---

## The 2×2 case, and why it's misleading as a default mental model

The textbook DiD is two groups (treated, control) and two periods
(pre, post). The estimator is the difference of differences:

```
ATT = (Y_treated,post - Y_treated,pre) - (Y_control,post - Y_control,pre)
```

Equivalently, run the regression:

```
Y_it = α_i + λ_t + β·D_it + ε_it
```

with unit and time fixed effects, where `D_it` is 1 if unit `i`
is treated in period `t`. Under parallel trends and no
anticipation, β is the ATT.

Three reasons this clean case is misleading as a default mental
model:

1. **Most real designs are staggered.** Treatment is adopted at
   different times by different units — minimum wage changes,
   policy rollouts, court decisions. The 2×2 mental model breaks
   in ways the next section spells out.

2. **Treatment effects are almost never constant.** They evolve
   over time since treatment, vary across units, and may depend
   on calendar time. The 2×2 framing hides this; the modern
   estimators surface it.

3. **The TWFE estimator looks like a generalization of the 2×2
   case but isn't.** It's a weighted average of 2×2 contrasts
   with weights that can be negative, that depend on treatment
   timing, and that have no defensible interpretation as a
   policy-relevant quantity. Goodman-Bacon (2021) is the
   decomposition; it's one of the most consequential papers of
   the decade for applied work.

If you remember nothing else from this section: the 2×2 DiD is
fine for 2×2 designs, and for nothing else by default.

---

## The TWFE problem, in detail

Two-way fixed effects regression on a staggered design produces a
weighted average of all possible 2×2 DiDs you can form from the
data. Goodman-Bacon decomposed this into three groups of
comparisons:

1. **Treated vs. never-treated** — these are clean. The
   never-treated are a valid imputation device.

2. **Earlier-treated vs. later-treated, in periods before the
   later group is treated** — also clean. The later group hasn't
   been treated yet, so it's a valid control.

3. **Later-treated vs. earlier-treated, in periods after the
   earlier group is treated** — this is the problem. The
   "control" group here is the earlier-treated group, *which is
   already treated*, and whose outcome is therefore reflecting
   the dynamic effects of its own treatment. If treatment effects
   grow over time, this comparison subtracts the earlier group's
   accumulated effect from the later group's nascent effect, and
   the resulting "DiD" can be wrong-signed.

The weights on these comparisons are determined by the variance
of treatment in each pair, which depends on group sizes and
treatment timing. There is no economic justification for these
weights. They are an artifact of the regression mechanics.

The practical consequence: when treatment effects are
heterogeneous in time-since-treatment (which they almost always
are), TWFE returns a weighted average where some weights are
negative, the average is not an ATT, and in extreme cases the
sign of the estimated coefficient is opposite the sign of every
underlying effect. This last case is rare in practice but the
fact that it's *possible* should be enough to retire TWFE as the
default for staggered designs.

**Markus's rule**: TWFE on a staggered design without justifying
why it's appropriate is a refusal-to-proceed condition. The
justification needs to be either (a) a homogeneity argument that
holds water, or (b) the design is actually 2×2 or close enough
to be treated as such, or (c) you're using TWFE as a robustness
check against a modern estimator and reporting both. None of
these are common. Most of the time the right move is one of the
modern estimators below.

---

## The modern estimators, briefly

There are five modern estimators worth knowing well. They are
not interchangeable. Each makes a different choice about what
the estimand is and how to construct the comparison group.

### Callaway and Sant'Anna (2021)

The most general framing. Defines group-time average treatment
effects ATT(g,t) — the effect for cohort `g` (units first treated
in period `g`) at calendar time `t` — and estimates each one
separately using a clean comparison group (never-treated units,
or not-yet-treated units, depending on which is available).
Aggregates can then be formed: by length of exposure (event
study), by cohort, by calendar time, or as an overall ATT.

The strength is the explicit estimand. Every quantity has a
clear interpretation; you choose the aggregation; the reader
sees what you did. Doubly-robust extensions (using outcome
regression and propensity scores) handle covariates cleanly.

Implementation: `did` package in R (Callaway and Sant'Anna's
own), `differences` in Python, the CS estimator inside
`pyfixest`. R is the most mature.

When to reach for it: most staggered designs, especially when
you want event-study aggregation and clean cohort-by-cohort
inspection. This is a strong default.

### de Chaisemartin and D'Haultfœuille (2020, 2022, 2024)

A family of estimators. The first paper (`did_multiplegt`)
introduced the diagnostic that quantifies what fraction of the
TWFE estimate comes from negative weights — extremely useful as
a check on legacy results. The later papers extend to dynamic
effects, continuous treatments, and treatments that turn on and
off (which the other estimators mostly don't handle).

The strength is generality, especially for treatments that
aren't simple absorbing-state binary adoption. If the policy
turns off and on, if treatment intensity varies, if you have
multiple treatments — this is the family that handles it.

Implementation: `did_multiplegt_dyn` in Stata and R; the
underlying logic is documented but the implementation is best
trusted to the package authors because the bookkeeping is
fiddly.

When to reach for it: non-absorbing treatments, continuous
treatments, multi-treatment designs, or when you need to
diagnose negative weights in a TWFE specification you've
inherited.

### Sun and Abraham (2021)

Specifically for event-study designs. Shows that the standard
event-study TWFE specification (interacting time-since-treatment
dummies with treatment) suffers from contamination across event
times — the coefficient on event-time `k` is contaminated by
treatment effects at other event times for other cohorts. Their
estimator is an interaction-weighted version that gives a clean
event-study path.

The strength is the focus on event studies as the deliverable.
If your headline figure is going to be an event-study plot, this
is the cleanest path to one.

Implementation: `fixest::sunab()` in R, `eventstudyinteract` in
Stata, `pyfixest` supports it. R is again the most mature.

When to reach for it: when the event study *is* the result, not
just a robustness check.

### Borusyak, Jaravel, and Spiess (2024) — the imputation estimator

Conceptually the cleanest. Estimates unit and time fixed effects
on the untreated observations only, uses these to impute
counterfactuals for the treated observations, and then averages
the residuals. The estimand is transparent (a clearly defined
ATT), the estimator is efficient under homoskedasticity, and
inference is straightforward.

The strength is conceptual clarity and efficiency. If parallel
trends holds and the model is right, this is the
best-performing estimator in the class.

The cost is fragility under violations: imputation methods lean
hard on the model, and if the unit and time fixed effects are
not the right model for the untreated counterfactual, the
imputed values are wrong in ways that other estimators absorb.

Implementation: `did_imputation` in Stata, `didimputation` in
R, also supported in `pyfixest`.

When to reach for it: when you trust the parallel trends
specification and you want the most efficient estimator
available. Pair with sensitivity analysis (see below).

### Gardner (2021) — two-stage DiD

A practical alternative to imputation. First stage: estimate
unit and time fixed effects using only untreated observations.
Second stage: subtract these from all observations and run a
simple regression of the residuals on the treatment indicator.
Equivalent to imputation in many cases but easier to extend to
covariates and cluster-robust inference.

Implementation: `did2s` in R and Stata.

When to reach for it: as a workhorse alternative to Callaway-
Sant'Anna when you want something fast and want to add
covariates flexibly.

---

## A decision tree

This is what Markus walks through when handed a DiD design. It's
short on purpose; the questions are the work.

**Q1: Is the design 2×2, or genuinely close to it?**
Two groups, two periods, sharp adoption. If yes: TWFE is fine,
use the standard estimator with cluster-robust SEs. Skip the
rest.

**Q2: Is treatment staggered?**
Different units treated at different times. If yes: TWFE is
disqualified as the headline. Continue.

**Q3: Is treatment absorbing?**
Once treated, always treated. If no — if treatment turns off and
on, varies in intensity, or is continuous — go to de Chaisemartin
and D'Haultfœuille. The other estimators assume absorption.

**Q4: Is the deliverable an event-study figure?**
If yes, Sun-Abraham or Callaway-Sant'Anna with event-time
aggregation. Sun-Abraham is the more focused tool; Callaway-
Sant'Anna gives you the event study and the cohort breakdown
together.

**Q5: Do you trust the parallel trends specification?**
If yes (and you've done the sensitivity analysis below), the
Borusyak-Jaravel-Spiess imputation estimator is the most
efficient. If you want robustness over efficiency, Callaway-
Sant'Anna with the never-treated as the comparison group.

**Q6: Are there covariates that matter for parallel trends?**
Use the doubly-robust version of Callaway-Sant'Anna, or the
two-stage Gardner estimator with covariates in the first stage.

The default, when in doubt: **Callaway-Sant'Anna with
never-treated as the comparison group, event-study and overall
ATT both reported, covariates handled doubly-robustly if
relevant, sensitivity analysis on parallel trends (next
section), and TWFE reported only as a legacy comparison if at
all.**

---

## Parallel trends, honestly

The parallel trends assumption is that, absent treatment, the
average outcome of the treated would have evolved in parallel
with the average outcome of the controls. It is fundamentally
untestable — you can't observe the counterfactual.

What you *can* do is the following:

**Pre-trends inspection.** Plot the event study. Look for flat
pre-period coefficients. This is necessary but not sufficient.
Roth (2022) documents that pre-trends tests have low power
against economically meaningful violations and that conditioning
on passing them creates selection bias in the reported
post-period effects.

**Sensitivity analysis (Rambachan and Roth, 2023).** This is the
right way to handle pre-trends in 2024 and after. Instead of
binary "pre-trends pass / pre-trends fail," report the range of
post-period treatment effects consistent with various assumptions
about how much the post-period violation could differ from the
observed pre-period violation. The two key parameters:
- The "smoothness" assumption (how much the pre-trend can change
  per period after treatment).
- The "relative magnitudes" assumption (how big the post-period
  violation can be relative to the pre-period violation).
The deliverable is a plot showing how the estimated ATT changes
as these parameters loosen. If the result survives substantial
violations, you have something. If it disappears under modest
violations, you don't.

Implementation: `HonestDiD` in R and Stata, by Roth and
collaborators. Use it. Reporting only the point estimate and
the pre-trends test is a 2018 practice; in 2024 it's a refusal
to do the available work.

**Triple differences (DDD).** If you can find a third dimension
along which treatment doesn't apply but everything else is
shared, you can difference out a common shock. Useful when
parallel trends across the obvious comparison is suspect but
parallel trends-of-trends is more defensible. Olden and Møen
(2022) is the recent treatment.

**Synthetic controls as a complement.** Where the comparison
group is small or obviously non-parallel, synthetic control
methods (Abadie, Diamond, Hainmueller; Doudchenko and Imbens;
Ben-Michael, Feller, and Rothstein) construct a weighted average
of controls to match the pre-treatment trajectory. The synth-DiD
hybrid (Arkhangelsky et al. 2021) is particularly useful: it
applies SC weighting *within* a DiD framework, which gives you
the efficiency of DiD with the comparability of synth.

Markus reaches for synth-DiD when the design has few treated
units and a heterogeneous donor pool, and for HonestDiD whenever
parallel trends is the load-bearing assumption (which is always).

---

## Inference

Three things to get right.

**Clustering.** Cluster at the level of treatment assignment.
Bertrand, Duflo, and Mullainathan (2004) is the original
warning; the practice has filtered through but is still violated
in published work. If treatment is at the state level, cluster
at the state. If at the firm level, cluster at the firm. The
within-unit serial correlation is the thing the clustering is
correcting for.

**Few clusters.** Cluster-robust inference assumes a large
number of clusters. With fewer than ~30, the standard CRSE is
biased toward over-rejection. The correction is wild cluster
bootstrap (Cameron, Gelbach, Miller 2008), implemented in
`fwildclusterboot` in R and `boottest` in Stata. Use it whenever
the number of clusters is small. "Small" here means anything
under about 50; under 20 it's mandatory.

**Permutation / randomization inference.** If the design has
very few treated units (one state, a few firms), conventional
inference doesn't apply. Permutation tests — randomly reassign
treatment across the units that could plausibly have been
treated, recompute the estimator, and use the distribution of
placebo estimates as the null — give you a defensible p-value.
Buchmueller, DiNardo, and Valletta (2011) is a clean example;
the synthetic control literature uses this routinely.

---

## Common ways DiD goes wrong

A short catalog of the failures Markus is on guard against,
because they are the failures that show up in published work and
in submitted drafts.

**TWFE on staggered designs without diagnostics.** Already
covered. The single most common failure mode in legacy work.

**Pre-trends tested, then ignored.** Researcher runs the
event-study plot, eyeballs the pre-period coefficients, declares
parallel trends "satisfied," and reports the post-period
estimate without sensitivity analysis. Insufficient since 2022.

**Comparison group contamination.** The "control" units are
affected by spillovers from treatment — treated firms compete
with control firms, treated regions trade with control regions,
treated workers move to control jobs. The DiD picks up
treatment-minus-spillover, not treatment. Borusyak and Hull
(2023) on the general problem; the solution is usually to
redefine the comparison group to exclude likely-contaminated
units, or to model the spillover explicitly.

**Anticipation.** Units adjust before formal treatment because
they see it coming. Pre-period coefficients pick this up and the
event study plot will show it. The fix is to redefine the
treatment date or to model the anticipation period separately.

**Reverse causality in treatment timing.** Units adopt treatment
*because* their outcomes are evolving in a particular way. The
parallel trends assumption fails by construction. Common in
policy studies where adoption is endogenous to local conditions.
Hard to fix within DiD; usually a sign that you need an IV or
RDD framing.

**The "always-treated" comparison group.** When some units are
already treated at the start of the panel, they cannot serve as
a clean control because their outcome is already reflecting
treatment. Drop them or model them separately. The Callaway-
Sant'Anna framework handles this naturally; ad-hoc TWFE often
doesn't.

**Compositional change in the panel.** Units enter and exit;
the composition of "treated" and "control" shifts over time;
the apparent DiD reflects compositional change, not treatment.
Particularly common with firm panels and worker panels. The
fix is balancing the panel, or being explicit about which
units are in each cell at each time.

**"Heterogeneous treatment effects across calendar time vs.
across event time."** These are different things and the
estimators handle them differently. Calendar-time heterogeneity
(treatment matters more in recessions) is captured by group-
time ATTs aggregated by calendar time. Event-time heterogeneity
(effects build up over years post-treatment) is captured by
event-study aggregation. Don't conflate them, and don't ignore
either.

---

## What to report

When Markus reports a DiD result, the deliverable includes:

1. The estimand, in plain language. "The average effect on
   firms first treated between 2010 and 2015, averaged over the
   first five years post-treatment."
2. The estimator and the comparison group. "Callaway-Sant'Anna
   with never-treated as the comparison; doubly-robust
   adjustment for size and industry."
3. The event-study plot. Pre-period coefficients visible, with
   confidence intervals. Vertical line at treatment.
4. The overall ATT with cluster-robust standard errors (and
   wild bootstrap if cluster count is small).
5. The HonestDiD sensitivity plot. The range of treatment
   effects consistent with violations of parallel trends of
   various magnitudes.
6. A statement of which checks would have changed the
   conclusion if they had failed, and whether they did fail.
   This is the standard from `run-a-regression-properly.md`
   and it applies here especially.
7. If TWFE is being reported as a legacy comparison, the
   Goodman-Bacon decomposition or the de Chaisemartin–
   D'Haultfœuille negative-weights diagnostic, so the reader
   knows what's in the TWFE estimate.

The interpretation paragraph names the estimand explicitly,
locates the result in the existing literature, and is honest
about what the design can and cannot say. "We find an effect of
X on Y" is not interpretation; "the average effect of policy P
on the units that adopted it between 2010 and 2015, over their
first five post-adoption years, is X, with the result robust to
parallel-trends violations up to M times the observed
pre-period variation" is.

---

## Implementation, by language

**R** is the most mature ecosystem for modern DiD. The packages
that matter:
- `did` (Callaway-Sant'Anna) — the canonical implementation
- `fixest::sunab` (Sun-Abraham) — fast, integrated with the
  rest of `fixest`
- `didimputation` (Borusyak-Jaravel-Spiess)
- `did2s` (Gardner two-stage)
- `HonestDiD` (Rambachan-Roth sensitivity)
- `fwildclusterboot` (wild cluster bootstrap)
- `synthdid` (Arkhangelsky et al.)
- `bacondecomp` (Goodman-Bacon decomposition for diagnostics
  on TWFE)

**Stata** has the workhorse implementations:
- `csdid` (Callaway-Sant'Anna)
- `did_multiplegt_dyn` (de Chaisemartin–D'Haultfœuille)
- `eventstudyinteract` (Sun-Abraham)
- `did_imputation` (Borusyak-Jaravel-Spiess)
- `did2s`
- `honestdid`
- `boottest` (wild bootstrap)
- `bacondecomp`

**Python** has been catching up via `pyfixest`, which now
implements Callaway-Sant'Anna, Sun-Abraham, and the imputation
estimator with a `fixest`-like interface. The `differences`
library is a pure-Python alternative for Callaway-Sant'Anna.
For HonestDiD there is no first-class Python implementation as
of 2024; call out to R via `rpy2` or use the Stata version.

Markus's default for new work is R unless there's a reason
otherwise. The implementations are most mature, the
documentation is best, and the package authors are the same
people writing the methodological papers.

---

## What this note doesn't cover, and where to find it

**Continuous treatment intensity.** Callaway, Goodman-Bacon, and
Sant'Anna (2024) is the right reference. The short version: most
of what you know about binary DiD doesn't transfer cleanly, and
the negative weights problem reappears in a new form.

**DiD with multiple treatments.** de Chaisemartin and
D'Haultfœuille have working papers on this; the field is still
moving.

**Spatial spillovers and SUTVA violations.** Butts (2023) and
Borusyak and Hull (2023) are the recent technical treatments.
This deserves its own note eventually.

**Matching plus DiD.** The Imai, Kim, and Wang (2023) framework
for panel matching is worth knowing; it's an alternative to the
weighting-based modern estimators that some applied
researchers find more intuitive.

**Bayesian DiD.** Mostly a niche; not in the standard toolkit
yet. Pang, Liu, and Xu (2022) if you need it.

These extensions get their own notes when a project actually
demands them. Don't speculatively expand this file.

---

## Cross-references

- `00_Identity/Principles.md` — sections on identification and
  on causation. The general posture this note operationalizes.
- `10_Methods/modern-toolkit-references.md` — the directory
  entry that points here. Update it if this file's content
  changes the recommendations.
- `50_Workflows/run-a-regression-properly.md` — the workflow
  this note slots into. The "specify identification, not just
  the equation" stage in the workflow refers to the kind of
  reasoning this note teaches.
- `10_Methods/Econometrics/instrumental-variables.md` — *to be
  written*. The natural next file. The LATE/compliers framing
  there will pick up where this note's discussion of
  treatment-effect heterogeneity leaves off.

---

*Last updated: 2026-04. The DiD literature is still moving;
this note will need revision as the continuous-treatment and
spillover literatures settle. The decision tree and the TWFE
critique are stable.*