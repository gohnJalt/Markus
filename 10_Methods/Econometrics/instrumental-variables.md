---
tags: [econometrics, causal-inference, iv, 2sls, weak-instruments]
related: [modern-toolkit-references, difference-in-differences, run-a-regression-properly, Principles]
---

# Instrumental Variables

This is the working note. It assumes you know the 2SLS mechanics
— what the first stage is, what the reduced form is, why the
2SLS coefficient is the reduced-form coefficient divided by the
first-stage coefficient in the just-identified case. What it
tries to do is give Markus enough fluency in the modern IV
literature to *reason* about whether a given IV design is
credible, what its estimand actually is, and which of the
post-2019 weak-instrument tools apply, rather than reciting
names. The entry in [[`modern-toolkit-references.md`]] is a
directory; this is the actual map.

The note is long. Read it linearly the first time. After that,
treat it as a reference: jump to the decision tree, the
weak-instruments section, or the judge-design section as needed.

---

## What IV is actually estimating

The thing to hold in your head before any estimator is the
**estimand**. IV is not estimating "the causal effect of X on
Y." In the heterogeneous-treatment-effects world — which is the
world we live in — IV is estimating a **Local Average Treatment
Effect**: the average effect of the treatment on the subset of
units whose treatment status is *moved by the instrument*. These
units are the compliers, and the LATE is their average effect,
not the population's.

This is Imbens and Angrist (1994), and it is the single most
important thing to internalize about modern IV. In the constant-
effects world the 2SLS coefficient can be called "the" causal
effect because there is only one. In the heterogeneous-effects
world there are many, and the instrument selects which one you
get. Two different instruments for the same endogenous variable
can give two different estimates, both correct, both targeting
different compliers.

Three consequences of taking this seriously.

First, the exclusion restriction is an assumption about
**potential outcomes**, not about correlations in the data. It
says that the instrument affects the outcome only through the
endogenous variable, for every unit. It is fundamentally
untestable. Overidentification tests (Sargan, Hansen J) test
whether multiple instruments give the *same* 2SLS estimate, not
whether any of them are valid — under heterogeneous effects they
are testing an equal-LATE restriction that is neither necessary
for validity nor sufficient for it. Treat them as diagnostics of
*consistency across instruments*, not as validity tests. This
point gets lost in practice and Markus should not let it slide.

Second, the monotonicity assumption is doing real work. It says
that no unit is a defier — nobody is pushed *into* treatment by
an instrument that pushes others *out*. Without monotonicity,
the 2SLS coefficient is a weighted average of treatment effects
with possibly-negative weights, and the LATE interpretation
collapses. Monotonicity is often defended casually. It shouldn't
be.

Third, the compliers are almost never the population of
interest. If the question is "what is the effect of college on
earnings," and the instrument is distance to the nearest
college, the compliers are the marginal students for whom
proximity tips the decision. The effect for those students is
*a* causal effect, but it is not the effect on the average
student, nor on the student a policy would target. The
interpretation paragraph has to say this in plain language.

Everything that follows is about getting the LATE right, or
being honest about when the design does not identify what the
paper claims it does.

---

## The just-identified case, and why it's the right default

The textbook IV is one endogenous regressor, one instrument. The
2SLS estimator is:

```
β_IV = Cov(Y, Z) / Cov(D, Z)
```

where Y is the outcome, D the endogenous regressor, Z the
instrument. Equivalently: the reduced-form coefficient from
regressing Y on Z, divided by the first-stage coefficient from
regressing D on Z.

Under heterogeneous effects, this is the LATE. Under constant
effects, it is *the* effect, full stop. The two interpretations
coincide when they coincide.

The modern literature has pushed hard toward **just-identified
designs** as the default, for three reasons. First, the
interpretation is cleaner: one instrument, one compliant
subpopulation, one LATE to argue about. Second, the weak-
instrument inference is well-developed and conservative in the
just-identified case (more below). Third, overidentification
tests in heterogeneous-effects settings do not test what people
think they test, so the alleged benefit of overidentification is
partly illusory.

Angrist and Pischke's *Mastering 'Metrics* chapter on IV makes
the case for just-identified designs explicitly. Markus's
default posture: if you can tell the story with one instrument,
tell it with one instrument. Multiple instruments appear in
Bartik-style settings, in judge designs with many judges, and in
structural work where the overidentifying restrictions are the
point — but these are specific cases with their own reasons, not
the default.

---

## The weak-instruments problem, in 2024

This is where the file earns its keep. The weak-instruments
literature has moved substantially since 2019, and applied work
has not caught up. If you remember one thing from this note,
remember that **the F > 10 rule is dead** for anything except
rough diagnostic screening. The people who wrote it said so.

### The old framework

Stock and Yogo (2005) gave critical values for the first-stage F
statistic based on how much 2SLS bias you were willing to
tolerate as a fraction of OLS bias, or on how much size
distortion you were willing to tolerate in the conventional
t-test. The rule of thumb "F > 10" came from the 10% bias
critical value in the single-instrument, single-endogenous-
regressor case under i.i.d. errors. It was always a rough cutoff
for a specific definition of "weak" under specific assumptions,
and the assumptions matter.

### What broke it

Three things.

**One.** Lee, McCrary, Moreira, and Porter (2022) showed that if
you use the conventional t-test for inference after 2SLS, and
you want the nominal 5% level to actually be 5%, the first-stage
F statistic needs to be about **104.7**, not 10. The intuition
is that weak first stages make the 2SLS distribution heavy-
tailed, and conventional confidence intervals are too narrow by
a factor that depends on the first-stage strength. At F = 10,
the true coverage of a nominal 95% interval is something like
85%. This is not a subtle correction.

**Two.** The Stock-Yogo critical values were derived under
homoskedastic, non-clustered errors. In practice almost nobody
has i.i.d. errors. Olea and Pflueger (2013) developed a
heteroskedasticity-and-autocorrelation-robust effective F
statistic that can be computed with arbitrary error structures,
and showed that the standard F-stat reported by most packages is
not the right quantity under realistic error structures. The
Olea-Pflueger effective F is what Markus should report in
practice.

**Three.** Anderson-Rubin confidence intervals are fully robust
to weak instruments, and have been since Anderson and Rubin
(1949). They do not depend on the first-stage strength. The
reason they fell out of fashion was computational — they require
inverting a test — and the reason they are back is that
computation is cheap now. Andrews, Stock, and Sun (2019) review
the modern weak-instrument inference landscape and recommend
reporting AR intervals alongside conventional ones as a routine
practice.

### What Markus reports

For any IV result, Markus's default deliverable includes:

1. The Olea-Pflueger effective F statistic, with the relevant
   critical value for the chosen size. If the F is below the
   Lee-McCrary-Moreira-Porter threshold for t-test inference,
   Markus says so explicitly.
2. Anderson-Rubin confidence intervals, alongside conventional
   2SLS confidence intervals. If they differ meaningfully, the
   AR interval is the honest one.
3. The first-stage coefficient and its standard error, not just
   the F.
4. The reduced-form effect of the instrument on the outcome.
   This is the thing identification ultimately rests on, and if
   the reduced form is a mess the IV result cannot be better
   than the reduced form.

"The first-stage F was 14" is not a defense; in 2024 it is a
confession that you have not done the relevant inference. Report
F = 14, report the AR interval, and let the reader see the
width.

### The Montiel-Olea-Pflueger procedure, operationally

For a single endogenous regressor, the recipe is:

1. Compute the Olea-Pflueger effective F, which is the
   heteroskedasticity-robust analogue of the first-stage F.
2. Compare it to the critical value for your chosen worst-case
   bias tolerance (usually 10%, sometimes 5%).
3. If it exceeds the critical value, conventional 2SLS inference
   is approximately valid.
4. If it does not, report AR intervals as the primary result.
   Do not hide the weakness by selectively reporting only
   conventional intervals.

The `ivreg2` package in Stata reports the Olea-Pflueger F.
`fixest` in R reports it via `feols(... | 0 | D ~ Z)` with
appropriate options. `linearmodels` in Python has it as the
default weak-instrument diagnostic. There is no excuse for
reporting the conventional F in 2024.

---

## From "find an instrument" to design-based IV

The older IV tradition treated instrument-finding as a creative
exercise: come up with a story, find a variable that fits the
story, run 2SLS. This produced a lot of clever papers and a lot
of work that hasn't aged well, because the exclusion restriction
was defended by storytelling rather than by design.

The modern tradition is **design-based IV**: instruments derived
from the structure of how treatment is assigned, where the
exclusion restriction is defensible because of how the
assignment mechanism actually works, not because of a story
about why the variable should not affect the outcome directly.

The canonical examples:

- **Random assignment of cases to judges.** Judges differ in
  sentencing severity; the assignment is effectively random
  within a court-by-time cell. The judge's historical severity
  rate is an instrument for whether a particular defendant is
  incarcerated. Exclusion is defensible because the judge does
  not affect the defendant's post-release outcomes through any
  channel other than the sentence (up to monotonicity and some
  auxiliary concerns).
- **Lottery-based admissions.** Charter school admission
  lotteries, Section 8 housing vouchers, military draft lotteries.
  The lottery is the instrument and the exclusion restriction is
  defensible by design.
- **Shift-share instruments** (Bartik), where the exogeneity of
  either the shares or the shocks is the identifying
  assumption. These get their own file
  ([[`shift-share-instruments.md`]]) because the two camps
  (Goldsmith-Pinkham-Sorkin-Swift and Borusyak-Hull-Jaravel)
  give different interpretations and the choice matters.

The rule Markus follows: if the defense of the exclusion
restriction comes down to "it seems implausible that Z would
affect Y except through D," the design is not design-based and
the paper is selling a story. If the defense comes down to "here
is the mechanism by which Z was assigned, and that mechanism
does not touch Y through any other channel," the design has
teeth.

This is the same move the DiD literature made when it shifted
from parallel-trends-as-assumption to parallel-trends-as-
something-you-probe-with-Rambachan-Roth sensitivity. The field
is learning to treat untestable assumptions as things you
characterize and bound rather than things you assert and move
on from.

---

## Judge and examiner designs

This literature deserves its own section because it is where a
lot of the serious applied IV work of the last decade lives, and
because it has its own failure modes.

The structure: cases are randomly assigned to judges (or
caseworkers, examiners, loan officers, patent examiners,
immigration judges). Judges differ in how they decide — one
judge incarcerates 60% of defendants with a given profile,
another incarcerates 40%. The leave-one-out mean of a judge's
decision rate becomes an instrument for the individual
decision, and you can estimate the effect of the decision on
downstream outcomes (re-offense, employment, earnings).

Kling (2006) was the early application. Doyle (2007, 2008) on
foster care, Aizer and Doyle (2015) on juvenile incarceration,
Dobbie, Goldin, and Yang (2018) on pretrial detention, Dobbie
and Song (2015) on bankruptcy are the canonical papers. The
design has also been used for patent examiners (Sampat and
Williams 2019), disability judges (Maestas, Mullen, and Strand
2013), and caseworkers (Bhuller, Dahl, Løken, and Mogstad 2020).

**What can go wrong.**

*Monotonicity.* The LATE interpretation requires that the
instrument moves every unit in the same direction. In judge
designs, this means: no defendant is treated *more* harshly by
a lenient judge than by a strict one. This is a strong claim.
Judges may be strict on some dimensions and lenient on others
("strict on drug crimes, lenient on property crimes"), and the
single-index severity rate then violates monotonicity.
Frandsen, Lefgren, and Leslie (2023) provide a testable
implication of monotonicity that should be reported in any judge-
design paper. Markus considers this test mandatory for this
class of designs.

*First-stage strength within cell.* Judge assignment is random
within a fairly narrow cell (court × time × case type). The
first stage is the leave-one-out judge severity predicting
individual decisions within those cells, and it can be weak if
the cells are small and the severity signal is noisy. Report
the effective F within the relevant strata.

*The "judge" may not actually be the decision-maker.*
Caseworker, prosecutor, or clerk influence can mean that the
nominally random judge assignment is less exclusive than it
looks. The exclusion restriction needs to survive that.

*Complier characterization.* Who is moved by a severe vs.
lenient judge? Usually the marginal cases — defendants on the
edge between incarceration and release. The LATE is for them,
not for all defendants, and certainly not for the hardened
career criminals whom every judge would convict or the clearly
innocent whom every judge would release. Abadie's (2002,
2003) LATE-complier-characterization machinery is the right
way to describe this formally.

The judge-design literature is strong *because* it takes these
concerns seriously. Papers that ignore monotonicity tests or
that hand-wave about compliers are not at the frontier anymore.

---

## Overidentification tests, honestly

When you have more instruments than endogenous regressors, the
system is overidentified and you can run Sargan (Hansen J under
heteroskedasticity) tests of the overidentifying restrictions.
Under constant treatment effects, failure to reject is evidence
that the instruments are jointly valid.

Under heterogeneous effects, this is not what the test does.
Each instrument identifies its own LATE, for its own compliers.
Two valid instruments can produce different 2SLS estimates
because their compliers differ, and the overidentification test
will reject even though both instruments are valid. Conversely,
two invalid instruments can produce similar biased estimates and
the test will fail to reject.

The honest reading: the Hansen J test is a test of whether the
instruments give *similar* estimates, not whether they are
*valid*. Failure to reject is consistent with validity; it is
also consistent with "both instruments have similar biases" or
"both instruments have similar compliers." Rejection is
consistent with invalidity; it is also consistent with "valid
instruments, different compliers."

Markus reports the J test when it is expected (referees will
ask) but interprets it correctly. The correct interpretation in
an interpretation paragraph reads something like: "The Hansen J
test does not reject (p = 0.3), which is consistent with the
instruments identifying similar LATEs — either because they
move similar compliers or because both are valid. This test
does not rule out common bias."

This is what Angrist and Pischke mean when they recommend
reporting one instrument at a time under heterogeneous effects
and being explicit about which compliers are being moved.

---

## Two-sample IV, when Y and D live in different datasets

Insert this section into [[`instrumental-variables.md`]] between
"Overidentification tests, honestly" and "The decision tree."

---

## Two-sample IV, when Y and D live in different datasets

The standard 2SLS setup assumes that Y, D, and Z all live in
the same dataset, measured on the same units. This is often
not the case. The outcome of interest may be in one dataset
(administrative earnings records), the endogenous regressor in
another (a survey of schooling or training), and the instrument
available in both. When the units can be linked, you merge and
proceed normally. When they cannot — because privacy rules
forbid it, because the two sources use incompatible identifiers,
because one is a sample and the other a register — **two-sample
IV** is the right tool.

The basic idea goes back to Angrist (1990) on the Vietnam draft
and Angrist and Krueger (1992) on quarter-of-birth and
schooling. Inoue and Solon (2010) is the modern reference and
the paper that settled the inference machinery.

### TSIV vs. TS2SLS

There are two variants and the distinction matters.

**Two-sample IV (TSIV)**, the original Angrist-Krueger version,
estimates the reduced-form and first-stage moments separately in
the two samples and divides them. This is consistent but
inefficient, and the finite-sample behavior is rough.

**Two-sample 2SLS (TS2SLS)**, the version Inoue and Solon
recommend and the version Markus reaches for by default,
estimates the first stage (D on Z) in Sample A, uses the
estimated first-stage coefficients to predict D̂ for every
observation in Sample B, and then regresses Y on D̂ in Sample B.
It is asymptotically more efficient than TSIV and behaves better
in finite samples. When the literature says "two-sample IV" in
2024, it usually means TS2SLS.

The intuition is standard 2SLS — predict the endogenous
regressor from the instrument, use the prediction in the second
stage — except the prediction comes from a model estimated on a
different sample.

### What it assumes, beyond standard IV

Everything the standard IV assumptions require (relevance,
exclusion, monotonicity) still applies. On top of that, TS2SLS
adds one critical assumption:

**The conditional distribution of D given Z must be the same in
both samples.** This is the assumption that lets the first-stage
model estimated in Sample A carry over to Sample B. It is not
automatic. It fails when:

- The two samples come from different time periods and the
  underlying relationship between Z and D has changed.
- The sampling frames are different in ways that correlate with
  the D-Z relationship (one is urban-biased, the other national).
- The instrument Z is measured differently in the two samples
  (e.g. different levels of aggregation, different question
  wording).
- The populations differ on characteristics that moderate the
  first stage.

The honest practice is to **overlap the samples where possible
and compare the estimated first stage in the overlap region**.
If the two samples both cover some common subset of units or
cells (same years, same geographies, same demographic strata),
estimate the first stage on that overlap in each sample
separately and check whether the coefficients agree. If they
don't, TS2SLS is on shaky ground and the assumption is doing
more work than it should.

### Inference, and why naive standard errors are wrong

The TS2SLS point estimate is straightforward. The standard
errors are not. The naive approach — run the second-stage
regression in Sample B and read off the usual 2SLS standard
errors — treats the first-stage predictions as if they were
data, ignoring the fact that they were estimated with
uncertainty on a different sample. The resulting standard errors
are too small, sometimes substantially.

Inoue and Solon (2010) derive the correct asymptotic variance,
which has two components: the second-stage sampling variation
in Sample B, and the first-stage sampling variation from Sample
A, propagated through the prediction step. The correct variance
is larger than the naive variance, and the gap grows with the
imprecision of the first stage.

The practical implication: **reporting naive standard errors in
a TS2SLS paper is wrong**, full stop. Use the Inoue-Solon
formula, a properly specified GMM that stacks the moment
conditions from both samples, or a bootstrap that resamples from
both source samples jointly. Papers that predate Inoue-Solon
(2010) commonly use the naive SEs and their inference should be
re-examined before the results are cited.

### Weak instruments in TS2SLS

The modern weak-instrument machinery — Olea-Pflueger effective
F, Lee-McCrary-Moreira-Porter thresholds, Anderson-Rubin
intervals — was developed for the single-sample case. It
extends to TS2SLS in spirit but not cleanly in software. The
Olea-Pflueger F can be computed on the first-stage sample, but
the LMMP threshold was derived under the assumption that the
second stage uses the same data as the first; applying it to a
TS2SLS context is an approximation whose accuracy depends on
the relative sizes of the two samples.

The conservative move: compute the effective F on Sample A,
apply the standard LMMP threshold as a rough guide, and
**bootstrap the AR-style inference across both samples** rather
than relying on asymptotic weak-instrument-robust intervals that
were derived for a different setting. The bootstrap is
expensive but honest.

### Implementation

This is one place where the software is genuinely behind the
methodology.

- **Stata** has `tsivreg` (Conroy) and various user-written
  commands, but nothing that implements Inoue-Solon inference
  out of the box. Most practitioners bootstrap manually.
- **R** has no dedicated TS2SLS package as of 2024. The
  standard move is to code the estimator directly — estimate
  the first stage with `feols` or `lm`, use `predict` to
  generate D̂ for the second sample, run the second stage with
  `feols`, and bootstrap the whole thing with `boot` or
  `fwildclusterboot`-style resampling across both source
  samples.
- **Python** is in the same state — `linearmodels` does not
  have native TS2SLS. Code it directly, bootstrap for inference.

The absence of a canonical package is itself a reason to be
careful: the implementations floating around in replication
files vary in whether they implement the Inoue-Solon correction
or the naive SEs. Always check.

### The Turkish context

TS2SLS is more relevant to Turkish applied work than to US work
because the linkage problem is more severe. TÜİK microdata,
SGK administrative records, CBRT firm-level data, and
ministry-specific registers frequently cannot be matched at the
unit level. When the question calls for combining (say) SGK
wage records with TÜİK household-level outcomes, the honest
choices are (a) restrict to questions where a single source
suffices, (b) use TS2SLS with an instrument that exists in
both, or (c) give up on unit-level causal inference and work at
the cell level. Option (b) exists and is legitimate, and the
assumption about the common conditional distribution of D given
Z is the load-bearing thing to defend — often possible when the
two datasets cover the same years and use compatible
definitions, much harder across methodology changes like the
2014 SGK revisions or the TÜİK Household Labour Force Survey
redesigns.

When Markus runs TS2SLS on Turkish data, the overlap check is
not optional. Find a period where both sources cover the same
population, re-estimate the first stage on each, and put the
comparison in the paper.

### What to report (in addition to the standard IV deliverables)

On top of the items in the "What to report" list below, a
TS2SLS paper should include:

- A clear statement of which sample each moment comes from.
- The first-stage coefficient estimated in Sample A, with its
  standard error.
- An overlap check on the first stage if any overlap exists,
  with the first-stage coefficients estimated separately on
  whatever common subset both samples share.
- Standard errors computed via the Inoue-Solon formula, a
  stacked-moment GMM, or a bootstrap across both samples.
  Naive second-stage standard errors are not acceptable.
- An explicit discussion of why the conditional distribution of
  D given Z is plausibly the same in both samples, naming the
  specific reasons it might fail in this application.

---

**Insertion note:** This section goes between "Overidentification
tests, honestly" and "The decision tree" in the IV file. No
other sections need edits. The decision tree's Q2 should have a
new bullet added: *"Y and D in different datasets → two-sample
2SLS; see the TS2SLS section above."*

---

## The decision tree

Markus uses this to choose an IV strategy for a given question:

**Q1: Do you have a genuinely design-based source of variation,
or are you searching for an instrument?**
If the latter, stop. The question is whether the endogeneity
can be addressed at all in your data, not whether you can find
a Z that correlates with D. An IV with a storytelling defense
is worse than an honest OLS with an acknowledged omitted-
variables problem, because the IV launders the uncertainty.

**Q2: What assignment mechanism generates the variation?**
- Random lottery → use it directly, compliers are defined by
  the lottery, this is the cleanest case.
- Random assignment to decision-makers (judges, examiners) →
  judge-design IV with leave-one-out severity; run Frandsen-
  Lefgren-Leslie monotonicity tests.
- Exogenous shocks interacted with shares of exposure → shift-
  share design; go to [[`shift-share-instruments.md`]] because the
  identification argument is subtle.
- Policy discontinuity in eligibility → consider whether RDD
  is actually the right tool; IV with a policy instrument is
  sometimes used but often RDD is cleaner.
- Institutional quirk that affects some units and not others →
  interrogate the exclusion restriction hard. These are the
  cases that don't age well.

**Q3: How strong is the first stage?**
Compute the Olea-Pflueger effective F. If it clears the Lee-
McCrary-Moreira-Porter threshold for your chosen inference
target, conventional 2SLS inference is approximately valid.
If not, report AR intervals as primary and conventional as
supplementary.

**Q4: Do you have one instrument or many?**
- One → just-identified, report AR intervals, done on the
  inference side.
- Many → ask whether overidentification adds to the argument
  or just adds instruments. If the multiple instruments have
  distinct identifying stories (e.g. multiple judges), the
  overidentification is incidental and each instrument should
  be reported separately at least once. If they are combined
  into a single index (Bartik), the question is how to
  interpret the index.

**Q5: Are you defending monotonicity?**
If the treatment is non-binary, or the instrument is a
single-index summary of a multidimensional decision, monotonicity
is not automatic. Run the Frandsen-Lefgren-Leslie test where
applicable, characterize the compliers, and do not rely on the
LATE interpretation if the test rejects.

**Q6: Who are the compliers, and are they the population you
want to talk about?**
This is the interpretation question. Use the Abadie
complier-characterization machinery to describe them on
observables. If the compliers are not the population of
interest, the LATE is still a causal effect — just not the one
the paper's title implies. Say so in the interpretation
paragraph.

**The default, when in doubt:** just-identified design-based IV,
Olea-Pflueger effective F reported, Anderson-Rubin confidence
intervals reported alongside conventional ones, monotonicity
discussed explicitly, compliers characterized, interpretation
paragraph that names the estimand in plain language and does
not overclaim generalization.

---

## Exclusion restriction, honestly

The exclusion restriction is untestable. You can't observe
potential outcomes under alternative instrument values. What
you *can* do is characterize the ways it could fail and probe
them:

**Spell out the channels.** List every channel through which Z
could plausibly affect Y other than through D. For each
channel, say what you would need to believe for it to be
negligible. "The instrument is distance to college; exclusion
requires that distance not affect earnings except through
college attendance. Channels that could violate this include
(a) labor market characteristics of rural areas, (b) migration
selection, (c) access to other post-secondary alternatives. I
address (a) by including county fixed effects, (b) by
restricting to non-movers, (c) by noting that community college
access is uncorrelated with distance to four-year institutions
in my sample."

**Conduct reduced-form checks on plausible violation
channels.** If the instrument should not affect a
pre-determined outcome, check it. Van Kippersluis and Rietveld
(2018) call this a "plausibly exogenous" test. If Z predicts
something it shouldn't, the exclusion restriction is in
trouble.

**Conley-Hansen-Rossi (2012) plausibly exogenous bounds.**
Allow the exclusion restriction to hold only approximately —
parameterize the direct effect of Z on Y as within some range
[−δ, δ] — and trace out the 2SLS estimates. If the result
survives for plausible δ, you have something. If it only
survives at δ = 0, the exclusion restriction is doing all the
work.

The move to make is not to declare the exclusion restriction
valid; it is to characterize how much violation the result
survives. This is exactly parallel to what Rambachan-Roth do
for parallel trends in DiD, and the posture is the same: treat
the load-bearing assumption as something you bound rather than
something you assert.

---

## Common failure modes

**"F > 10, so we're fine."** No. Report the Olea-Pflueger F
and the LMMP threshold, or report AR intervals. This is the
single most common modern IV error.

**Defending exclusion with a story instead of a design.** If the
identification depends on the reader finding the story
plausible, the reader will — until they don't, and the paper
won't age well.

**Interpreting LATE as ATE or ATT silently.** Papers that use
distance-to-college as an instrument and then write "the effect
of college on earnings is X" are selling the LATE as something
it isn't. The honest version names the compliers.

**Using overidentification tests as validity tests.** They
aren't. Report them, interpret them honestly, don't let them
substitute for an identification argument.

**Monotonicity violations in judge designs.** The Frandsen-
Lefgren-Leslie test exists; run it. If it rejects, the LATE
interpretation is in trouble and the paper needs to either
restrict the sample or acknowledge the failure.

**Ignoring complier characterization.** Abadie (2002, 2003)
gave us the tools to describe compliers; not using them is a
choice to keep the interpretation vague.

**Proliferating instruments in GMM for weakly-identified
settings.** In dynamic panel and other GMM contexts, more
instruments means more overidentification and also more finite-
sample bias. The Roodman critique of instrument proliferation
in dynamic panels (see the [[`panel-methods.md`]] note) applies
across GMM-based IV settings.

**Ignoring the reduced form.** The reduced-form effect of Z on
Y is the thing you are actually estimating; the 2SLS is just
rescaling it by the first stage. If the reduced form is
insignificant, the IV result cannot be significant in any
honest sense, regardless of what the 2SLS standard error says.

---

## What to report

When Markus reports an IV result, the deliverable includes:

1. **The estimand in plain language.** "The local average
   treatment effect on compliers — defendants on the margin
   between detention and release — of pretrial detention on
   subsequent re-offense."
2. **The assignment mechanism.** How the instrument was
   generated in the world, not a story about why it correlates
   with D.
3. **The reduced form.** The effect of Z on Y, with standard
   errors, separately from the 2SLS.
4. **The first stage.** The effect of Z on D, with the Olea-
   Pflueger effective F and the comparison to the relevant
   weak-instrument critical value.
5. **Anderson-Rubin confidence intervals**, alongside
   conventional 2SLS CIs. If they differ, the AR interval is
   primary.
6. **Complier characterization.** Abadie-style description of
   who the compliers are on observables.
7. **A defense of the exclusion restriction** that lists
   channels and addresses each, not a sentence asserting
   plausibility. Conley-Hansen-Rossi bounds where feasible.
8. **Monotonicity discussion**, and the Frandsen-Lefgren-Leslie
   test where applicable.
9. **A statement of which checks would have changed the
   conclusion if they had failed**, per the regression workflow,
   and whether they did fail.

The interpretation paragraph names the estimand explicitly,
locates the result in the literature, and is honest about the
compliers. "We find an effect of X on Y" is not interpretation;
"the LATE of pretrial detention on re-offense, for defendants on
the margin of the detention decision in the Harris County
system between 2010 and 2015, is X, with the result surviving
exclusion-restriction violations up to [Conley-Hansen-Rossi
δ-bound]" is.

---

## Implementation, by language

**R** is a strong ecosystem for modern IV, especially for
design-based work. The packages that matter:

- `fixest::feols` — fast 2SLS with high-dimensional fixed
  effects, Olea-Pflueger diagnostics, clean IV syntax. The
  Markus default for most panel IV work.
- `ivreg` (package, not the old `AER::ivreg`) — modern IV with
  proper weak-instrument inference from Kleiber and Zeileis.
- `ivmodel` — Anderson-Rubin and related weak-instrument-robust
  inference in the just-identified and overidentified cases.
- `ShiftShareSE` — Adão-Kolesár-Morales inference for
  shift-share designs (also used in [[`shift-share-instruments.md`]]).
- `sensemakr` and related — sensitivity-to-unobservables tools
  that extend to IV contexts.

**Stata** has the workhorse implementations, and for judge-
design and applied IV work is still where a lot of the field
lives:

- `ivreg2` / `ivreghdfe` — the reference implementations,
  with the Olea-Pflueger F and most weak-instrument
  diagnostics. `ivreghdfe` adds high-dimensional FE support.
- `weakiv` — comprehensive weak-instrument-robust inference
  (AR, CLR, LM, K tests), by Finlay and Magnusson.
- `rdrobust` is for RDD, not IV, but shows up adjacent to IV
  in fuzzy designs.
- `xtabond2` for dynamic panel GMM (see [[`panel-methods.md`]]).

**Python** has caught up substantially:

- `linearmodels.iv` — the cleanest Python IV implementation
  (Sheppard), with `IV2SLS`, `IVGMM`, weak-instrument
  diagnostics, and AR intervals built in. Markus's Python
  default.
- `pyfixest` — high-dimensional FE 2SLS with a `fixest`-like
  interface; supports the Olea-Pflueger F.
- `econml` — IV variants for ML-assisted estimation, useful
  when the first stage is nonparametric.

For Anderson-Rubin intervals specifically, R and Stata are ahead
of Python; for routine 2SLS with high-dimensional fixed effects,
the three ecosystems are comparable and the choice is about
which language the rest of the project is in.

Markus's default for new IV work is R (`fixest`) unless there is
a reason otherwise, same as for DiD. The implementation
machinery is most mature, the weak-instrument diagnostics are
built in, and the package authors are the methodologists.

---

## What this note doesn't cover, and where to find it

**Shift-share / Bartik instruments.** The two camps (Goldsmith-
Pinkham-Sorkin-Swift 2020 and Borusyak-Hull-Jaravel 2022) give
different identifying assumptions, and the choice between them
matters enough to deserve its own file. See
[[`shift-share-instruments.md`]].

**IV in dynamic panels and GMM more broadly.** Arellano-Bond,
Blundell-Bond, and the instrument-proliferation problem belong
in [[`panel-methods.md`]] because the issues are panel-specific.

**Fuzzy regression discontinuity.** FRD is IV-in-RDD-clothing,
with the discontinuity as the instrument. The weak-instrument
concerns transfer, but the bandwidth and bias-correction
machinery is RDD-specific. See [[`regression-discontinuity.md`]].

**IV with machine-learning first stages.** Double/debiased ML
in the IV setting (Chernozhukov et al. 2018) handles
high-dimensional controls and nonparametric first stages. This
is a DML topic (see the DML section in
[[`modern-toolkit-references.md`]]) rather than an IV topic in the
classical sense.

**Nonlinear IV.** Control function approaches for nonlinear
models, special regressors, limited dependent variables with
endogeneity — these are specialized and deserve their own
treatment if a project demands it.

**Identification at infinity and Heckman selection.** These are
more about selection bias than IV per se, but the mechanics
touch. A selection note may appear in v0.3 if it earns its
place.

These extensions get their own notes when a project actually
demands them. Don't speculatively expand this file.

---

## Cross-references

- [[`00_Identity/Principles.md`]] — sections on identification and
  on causation. The general posture this note operationalizes,
  especially the line that "every identification strategy makes
  assumptions you cannot test" and the warning against "method
  laundering."
- [[`10_Methods/modern-toolkit-references.md`]] — the directory
  entry that points here. Update it if this file's content
  changes the recommendations.
- [[`10_Methods/Econometrics/difference-in-differences.md`]] — the
  LATE/compliers framing here picks up from the DiD note's
  discussion of treatment-effect heterogeneity. The two notes
  share the posture that estimators target specific estimands
  under specific assumptions, and that the assumptions must be
  probed rather than asserted.
- [[`10_Methods/Econometrics/shift-share-instruments.md`]] — the 
  natural next file, building on the exclusion-restriction and 
  exogenous-shock framing from this note.
- [[`50_Workflows/run-a-regression-properly.md`]] — the workflow
  this note slots into. Stage 4 ("specify identification, not
  just the equation") refers to the kind of reasoning this
  note teaches, and Stage 1 (naming the estimand) is where the
  LATE/compliers discussion belongs in a real analysis.

---

*Last updated: 2026-04. The weak-instrument literature is still
moving (the LMMP threshold was 2022, the Montiel-Olea-Pflueger
framework is under active extension), but the core message —
F > 10 is dead, report AR intervals, just-identified is the
default — is stable. The judge-design literature is maturing
but the core references and the monotonicity critique are
settled.*