---
tags: [econometrics, causal-inference, iv, shift-share, bartik]
related: [modern-toolkit-references, instrumental-variables, difference-in-differences, run-a-regression-properly, Principles]
---

# Shift-Share Instruments

This is the working note. It assumes you've read
[[`instrumental-variables.md`]] — the design-based IV posture, the
LATE framing, the modern weak-instrument machinery, and the
exclusion-restriction-honesty section all carry over and are
not rederived here. What this file does is the part that is
specific to shift-share designs: the two identifying stories
(the shares story and the shocks story), when each applies,
how to do inference under each, and why the choice between them
is not a stylistic preference but a substantive claim about
what the research design is.

Shift-share deserves its own file because the literature
reorganized itself between roughly 2018 and 2022 in a way that
applied practice has partly caught up to and partly hasn't.
Before the reorganization, shift-share was "the Bartik
instrument," a clever construction whose validity was
sometimes defended casually. After the reorganization, it is
two different identification strategies that happen to produce
similar-looking estimators, with different assumptions,
different diagnostics, and different inference machinery. The
choice of which story you are telling matters enough that
Markus will not let a draft slide without naming it.

---

## What shift-share is, operationally

The canonical shift-share instrument has the form:

```
Z_i = Σ_k s_{ik} · g_k
```

where `i` indexes units (commuting zones, regions, counties,
countries), `k` indexes sectors (industries, occupations,
product categories, origin countries), `s_{ik}` is unit `i`'s
baseline share of exposure to sector `k`, and `g_k` is a
sector-level shock. The unit-level instrument is the share-
weighted sum of sector shocks, and it is used to instrument for
the unit's treatment variable (employment change, import
exposure, immigration inflow, and so on).

The name comes from the decomposition in the shift-share
accounting literature (where the national growth rate is
decomposed into an industry mix effect, a national shift, and a
residual), but the econometric object is just a constructed
instrument that combines a cross-sectional exposure measure
with a time-series shock.

The classic applications set what the researcher is after:

- **Bartik (1991):** local labor demand, where shares are
  industry employment shares in a baseline year and shocks
  are national industry employment growth rates. The
  instrument is a prediction of what local employment growth
  would have been if each industry locally grew at its
  national rate.
- **Autor, Dorn, and Hanson (2013):** the "China shock,"
  where shares are industry employment shares and shocks are
  Chinese exports to other rich countries (as a proxy for
  Chinese supply-side shocks).
- **Card (2001):** immigration enclaves, where shares are the
  historical distribution of immigrants from a given origin
  across US cities and shocks are aggregate origin-country
  inflows.
- **Nunn and Qian (2014):** the product-level shift-share in
  the historical trade literature.

These papers look structurally similar. Their identification
arguments, read carefully, are not the same, and the post-2018
literature is largely the process of making that clear.

---

## The two stories, and why the distinction is the whole game

### Story 1: exogenous shares (Goldsmith-Pinkham, Sorkin, and Swift 2020)

Goldsmith-Pinkham, Sorkin, and Swift (GPSS) showed that a
shift-share instrument is numerically equivalent to a
generalized-method-of-moments IV that uses the *shares* as
instruments, with the shocks acting as weights. The identifying
content, in this interpretation, lives in the shares.

What this means: if you are relying on the GPSS interpretation,
you are asserting that the baseline shares `s_{ik}` are
exogenous to the unit-level outcome, conditional on controls.
In the Bartik example, you are saying that a commuting zone's
1980 industry mix is not correlated with unobservables that
drive subsequent employment outcomes, after controlling for
whatever you have controlled for. This is a claim about the
*cross-sectional distribution of exposure*, not about the
time-series shocks.

The GPSS machinery gives you three things:

1. A **"Rotemberg weights" decomposition**: each share-based
   instrument gets a weight in the overall 2SLS estimate. The
   top-weighted shares tell you where the identification is
   actually coming from. This is diagnostically valuable — a
   Bartik instrument whose entire weight sits on one or two
   industries is effectively a case study of those industries,
   and the researcher should know that.
2. An **exogeneity test logic**: the shares that carry the
   weight are the ones whose exogeneity you have to defend.
   "My shares are exogenous" is too vague; "the employment
   shares in electrical machinery and textiles in 1980 are
   exogenous to post-1980 employment outcomes conditional on
   region and demographic controls" is the actual claim.
3. A **pre-trends logic**: if the shares are exogenous, they
   should not predict pre-period outcomes. This becomes a
   testable diagnostic, analogous to the pre-trends check in
   DiD, and Markus expects to see it.

When to reach for the GPSS story: when the shares plausibly
reflect long-standing structural features of the unit that are
exogenous to the shock period, when the shocks are relatively
few and large, and when the researcher can tell a credible
story about why the high-Rotemberg-weight shares are not
correlated with unobservables.

### Story 2: exogenous shocks (Borusyak, Hull, and Jaravel 2022)

Borusyak, Hull, and Jaravel (BHJ) showed that a shift-share
instrument is also equivalent to an IV that uses the
*sector-level shocks* directly, at the sector level, with the
unit-level specification acting as an aggregation device. The
identifying content, in this interpretation, lives in the
shocks.

What this means: if you are relying on the BHJ interpretation,
you are asserting that the sector shocks `g_k` are as good as
randomly assigned across sectors, conditional on sector-level
controls. You are essentially running a sector-level IV where
each sector is a unit and the shock is the source of
variation, and the aggregation to the unit level is just a
weighting device that reflects which units were exposed to
which sectors.

The BHJ machinery gives you:

1. **Equivalence with a sector-level regression**: the
   shift-share 2SLS estimate can be recovered by running the
   regression at the sector level, with shocks as the
   instrument and a particular weighting scheme. This is not
   just a curiosity — it reframes the whole question. If you
   would not believe the sector-level regression is
   identified, you should not believe the shift-share is
   identified.
2. **A different set of diagnostics**: the number of
   "effective sectors" matters for inference, and BHJ give a
   concrete formula for computing it. An instrument built
   from 300 sectors where three sectors drive everything has
   effectively three sectors for inference purposes.
3. **A different defense of exclusion**: you defend it at the
   sector level. "Chinese export growth in sector `k` to third
   countries is not correlated with US outcomes in sector `k`
   except through the channel of US-China trade" is a
   sector-level claim that can be probed directly.

When to reach for the BHJ story: when you have many shocks,
when the shocks are plausibly exogenous at the sector level
(from a genuinely external source or a specific identification
argument), and when the shares are less defensible as
structural features — e.g. when they are outcome-like and
you'd rather not assert their exogeneity.

### The Turkish context

The Turkish applied-work angle: when shift-share is used on
Turkish data (typically with regional employment data from
TÜİK or SGK, interacted with either national or global sector
shocks), the choice of story matters because both stories have
weaknesses in the Turkish setting that don't show up in US
work. Shares based on 2000s NUTS-2 regional employment are
contaminated by internal migration patterns that were driven
by political and security events (the 1999 earthquake,
southeast migration) — so GPSS exogeneity is hard to defend
without explicit controls for migration history. Shocks based
on Turkish national growth are problematic because the
national economy is small enough that unit-level outcomes
feed back into national sector growth — so BHJ requires either
an external shock (global commodity prices, foreign demand)
or an instrument for the Turkish shocks themselves. Both
routes exist and both require the kind of explicit
justification that the modern shift-share literature demands.

### The two stories are not substitutes

You do not get to pick whichever story is more convenient. The
honest move is to **state which story the identification rests
on, and defend the assumption that story requires**. "The
shares are exogenous and also the shocks are exogenous" is not
a stronger claim than one of them alone — it is usually just
hand-waving. The GPSS and BHJ papers both note that in most
applications one story is more defensible than the other, and
the researcher should pick the defensible one and commit to it.

Markus's rule: **name the story before running the estimator**.
In the interpretation paragraph, name it again. A draft that
runs a shift-share IV and reports a result without saying which
story the identification is appealing to is doing something
the modern literature no longer allows.

---

## The decision tree

**Q1: Where does the variation that identifies your effect
come from — the cross-sectional exposure pattern or the
time-series sector shocks?**

- Cross-sectional exposure plausibly exogenous, shocks possibly
  not → GPSS story, share-based identification, Rotemberg
  weights diagnostic.
- Shocks plausibly exogenous (external, well-identified),
  shares possibly endogenous → BHJ story, shock-based
  identification, sector-level reframing.
- Neither clearly → the instrument may not be identified. Do
  not push forward by combining weak versions of both stories.
- Both clearly → rare, but report both diagnostic stacks and
  show robustness across stories.

**Q2: Under the GPSS story, where do the Rotemberg weights
concentrate?**
Compute them. If the top 5 sectors carry most of the weight,
those are the sectors whose exogeneity you are actually
defending. Write the defense for those specific sectors. A
Bartik instrument whose weight concentrates on one industry is,
in effect, a single-industry case study and should be argued
as such.

**Q3: Under the BHJ story, what is the effective number of
shocks?**
Compute it. BHJ give a formula (it's a function of the
Herfindahl of the aggregated exposure weights). If the
effective number of shocks is small, your degrees of freedom
for sector-level inference are small even if the nominal
sector count is large. This matters for how you compute
standard errors.

**Q4: What is the first stage, and what does its weak-
instrument diagnostic look like?**
Everything in [[`instrumental-variables.md`]] on weak-instrument
inference applies: report the Olea-Pflueger effective F, check
against the LMMP threshold, report Anderson-Rubin intervals
where appropriate. The shift-share literature has not
consistently caught up to the weak-instrument literature;
Markus has.

**Q5: Is the inference at the right level?**
This is the shift-share-specific question. Standard clustered
standard errors at the unit level are not adequate under
either story. GPSS inference requires clustering at the share
level; BHJ requires clustering at the shock level (often
sectors). Adão, Kolesár, and Morales (2019) showed that
naive SEs can be badly wrong and the `ShiftShareSE` package
implements the correction.

**Q6: Are you making a causal claim, a descriptive claim, or
a prediction?**
Shift-share instruments are used for all three. The identifying
assumptions matter most for the causal interpretation. A
"Bartik projection" used to describe expected local employment
growth under no-shift-in-composition is a descriptive tool and
does not require the exogeneity assumptions to be defensible.
Don't sneak a descriptive use into a causal claim by reporting
causal-style tables.

**The default, when in doubt:** pick one story (whichever is
more defensible in the specific application), compute the
diagnostics the story requires (Rotemberg weights for GPSS,
effective number of shocks for BHJ), use Adão-Kolesár-Morales
inference, report the first-stage diagnostics from the modern
IV literature, and write an interpretation paragraph that
names the story and the LATE.

---

## Inference, in detail

This is the section that catches the most published shift-share
papers off guard. Standard OLS or unit-level-clustered 2SLS
standard errors are typically wrong for shift-share instruments,
and the correct inference depends on the story.

### The problem

Adão, Kolesár, and Morales (2019) showed that shift-share
regressions have a specific form of cross-unit correlation:
units with similar exposure shares receive correlated
instruments (because they are built from the same sector
shocks), and this correlation is not absorbed by standard
clustering at the unit level. Two commuting zones with similar
industry mixes get correlated Bartik instruments even if they
are geographically distant, so geographic clustering does not
help.

The practical consequence: naive t-statistics in shift-share
regressions are often off by a factor of 2 or more. Published
results with "t = 3" have sometimes been re-evaluated at "t =
1.4" when AKM inference is applied. This is a big deal and it
is not handled by tweaking the cluster level.

### The correction

Adão, Kolesár, and Morales propose inference that accounts for
the correlation induced by shared sector exposure. The idea is
to construct standard errors that reflect the true
dimensionality of the sector-level variation. Operationally,
this is done via the `ShiftShareSE` package in R and Stata,
which takes the shares and shocks and returns corrected
standard errors.

The BHJ framework implies a related correction — if the
identification is sector-level, inference should be at the
sector level, not at the unit level. Clustering at the sector
level (in the sector-level equivalent regression) gives the
same kind of correction in a different formalism. Borusyak and
Hull (2023) extend the framework to cases where the exposure
weights themselves need bias correction.

### What Markus reports

Any shift-share result Markus reports includes, at minimum:

1. **Naive standard errors** (for transparency, so the reader
   can see the delta).
2. **AKM or BHJ-equivalent standard errors** as the primary
   inference.
3. **The weak-instrument diagnostics** from
   [[`instrumental-variables.md`]] (Olea-Pflueger F, AR intervals
   where relevant).
4. **The effective degrees of freedom** (Rotemberg-weight
   concentration or effective number of shocks, depending on
   the story).

Reporting only naive SEs in 2024 is not acceptable. The
correction has been available and replicable for five years.

---

## Common failure modes

**Not picking a story.** The most common modern failure: the
paper runs a Bartik regression, defends the "instrument"
vaguely as "pre-determined," and does not commit to GPSS or
BHJ. This is no longer acceptable. The modern literature
requires a stated identification story.

**GPSS without Rotemberg weights.** If you are using the shares
as identification, you owe the reader the Rotemberg weights.
Hiding which sectors carry the identification is equivalent to
hiding the identification.

**BHJ without sector-level reasoning.** If you are using the
shocks as identification, the sector-level version of the
regression should make sense and should be reported. If it
doesn't make sense at the sector level, the unit-level
aggregation is not rescuing it.

**Naive standard errors.** Covered above. In 2024 this is a
mistake the reviewer should catch and increasingly will.

**Treating shift-share as a general exogeneity trick.** Shift-
share instruments are not a way to launder OLS into causal
inference. They are IV estimators whose validity depends on
specific assumptions, and those assumptions can fail. A paper
that reaches for Bartik because nothing else worked is a paper
that hasn't yet admitted the question may not be identified in
its data.

**Not updating for the ADH China-shock critiques.** The
Autor-Dorn-Hanson China shock is the most famous shift-share
application and has been heavily re-examined. Borusyak, Hull,
and Jaravel revisit it with BHJ methods; others have raised
concerns about the specific sectors driving the GPSS-weighted
estimates. A paper using ADH-style methods should engage with
this literature, not cite the original as a settled application.

**Outcome-based shares.** If the shares are measured after
treatment begins or depend on outcomes, they are not
instruments. This sounds obvious; the failure mode is using
"baseline" shares measured at t=0 when treatment effects
operate through the shares and t=0 is post-treatment for some
units. The shares have to be genuinely predetermined.

**Assuming monotonicity.** The LATE interpretation from
[[`instrumental-variables.md`]] carries over: the shift-share IV
identifies an average effect over compliers (units whose
treatment is moved by the instrument). Monotonicity at the
unit level is often plausible (more exposed units get more
treatment) but not automatic, and under heterogeneous
treatment effects the interpretation is still a LATE, not a
population ATE.

---

## What to report

For a shift-share IV result, the deliverable includes
everything on the standard IV list from
[[`instrumental-variables.md`]], plus:

1. **The identification story**, named explicitly: GPSS
   (exogenous shares) or BHJ (exogenous shocks).
2. **The specific defense of the exogeneity assumption for the
   chosen story.** Under GPSS, this means defending the
   high-Rotemberg-weight shares. Under BHJ, this means
   defending the sector-level exogeneity of the shocks.
3. **The Rotemberg weights table** (if GPSS), showing which
   sectors carry the identification. Top 5 weights at minimum.
4. **The effective number of shocks or sectors** (if BHJ),
   showing the true dimensionality of the variation.
5. **AKM or BHJ-corrected standard errors** as primary, naive
   SEs reported for comparison.
6. **Weak-instrument diagnostics** (Olea-Pflueger F, AR
   intervals if the first stage is marginal).
7. **A reduced-form check**: does the instrument predict
   pre-period outcomes? Under GPSS this is a test of share
   exogeneity; under BHJ it is a test of shock exogeneity; in
   either case it is cheap and informative.
8. **Complier characterization** at whatever level the story
   allows — which units or sectors are being moved by the
   instrument.

The interpretation paragraph names the story, names the LATE,
and does not overclaim. "The effect of local labor demand
shocks on employment, identified by variation in national
sector shocks weighted by 1980 industry shares, is X, for the
commuting zones whose industry mixes put them on the margin of
exposure. This interpretation rests on the exogeneity of 1980
shares conditional on 1970 region characteristics, defended in
Section Y." — this is the level of specificity the modern
literature expects.

---

## Implementation, by language

**R** is where the shift-share methodology lives most fully:

- `ShiftShareSE` (Kolesár et al.) — the reference
  implementation of AKM inference. Handles both the BHJ-style
  sector-level correction and the AKM-style share-based
  correction.
- `fixest::feols` for the point estimation, since shift-share
  2SLS is just 2SLS with a constructed instrument.
- `bartik.weight` (package by Paul Goldsmith-Pinkham's group)
  computes Rotemberg weights for GPSS diagnostics.
- For the sector-level BHJ reformulation, any IV package works
  — the reformulation is computational, not methodological.

**Stata** has `bartik_weight` (Goldsmith-Pinkham), the
`ShiftShareSE` port, and standard IV tools (`ivreg2`,
`ivreghdfe`). The ecosystem is slightly less integrated than R
but everything needed is available.

**Python** lags here. `linearmodels.iv` handles the 2SLS
computation but there is no first-class implementation of AKM
inference or Rotemberg weights as of 2024. The practical path
is to compute point estimates in Python and call out to R via
`rpy2` for the shift-share-specific inference, or to do the
entire analysis in R.

Markus's default for shift-share work is R, unambiguously. The
methodology and the software are tightly coupled here and R is
where the methodologists have invested.

---

## What this note doesn't cover, and where to find it

**Continuous treatment shift-share.** Recent work by Borusyak
and Hull extends to continuous-exposure settings; the issues
overlap with the continuous-treatment DiD literature noted in
[[`difference-in-differences.md`]].

**Shift-share in spatial equilibrium models.** When shift-share
is used as the empirical counterpart to a quantitative
spatial model (Ahlfeldt-Redding-Sturm-Wolf, Caliendo-Parro
style), the identification question interacts with the model
structure. This belongs in a structural note, not here.

**Recentered shift-share (Borusyak and Hull, 2023).** The
formal treatment of how to center the instrument when the
exposure weights themselves have expected components. This is
an important extension for settings where part of the
variation in exposure is predictable and should be netted out.
A project that genuinely needs it should bring it in;
otherwise, standard shift-share covers most applied needs.

**Shift-share in macro (IV-based LP, external instruments for
SVARs).** The shift-share logic sometimes shows up as an
external instrument in structural VAR or LP-IV work. That
belongs in [[`local-projections.md`]] and later in a SVAR note,
not here.

These extensions get their own notes or sections when a project
demands them. Don't speculatively expand this file.

---

## Cross-references

- [[`00_Identity/Principles.md`]] — sections on identification and
  on causation. The general posture this note operationalizes,
  especially the warning against method laundering and the
  requirement that every identification strategy be defended
  on assumptions you can characterize.
- [[`10_Methods/modern-toolkit-references.md`]] — the directory
  entry that points here. Update it if this file's content
  changes the recommendations.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] — the
  parent note. The LATE framing, the weak-instrument machinery,
  the exclusion-restriction honesty section, and the design-
  based IV posture all carry over. Read that file before this
  one if you haven't.
- [[`10_Methods/Econometrics/difference-in-differences.md`]] — the
  parallel-trends-honesty section is the structural sibling of
  this file's "pick a story and defend it" posture. Same move,
  different setting.
[[`10_Methods/Econometrics/local-projections.md`]] — shift-share 
  instruments sometimes enter macroeconomic LP-IV settings as 
  the external instrument, and that intersection will be covered there.
- [[`50_Workflows/run-a-regression-properly.md`]] — Stage 4
  ("specify identification, not just the equation") is where
  the GPSS-vs-BHJ choice belongs in a real analysis.

---

*Last updated: 2026-04. The shift-share literature reorganized
itself between roughly 2018 and 2022, and the post-2022
landscape is stable on the core points: pick a story, report
the story-specific diagnostics, use AKM inference. Extensions
to continuous treatment and recentering are still developing
and the note will need revision as they settle.*