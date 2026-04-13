---
tags: [econometrics, time-series, macro, local-projections, impulse-response]
related: [modern-toolkit-references, instrumental-variables, difference-in-differences, run-a-regression-properly, Principles]
---

# Local Projections

This is the working note. It assumes you know what an impulse
response function is and why anyone estimates one. What it
tries to do is give Markus enough fluency in the Jordà-style
local projections literature — and the LP-vs-VAR debate that
has shaped it since 2021 — to *reason* about which estimator
fits a given macro question, rather than defaulting to either
the VAR or the LP out of habit. The entry in
`modern-toolkit-references.md` is a directory; this is the
actual map.

The note is long. Read it linearly the first time. After
that, treat it as a reference: jump to the LP-vs-VAR section,
the long-horizon bias discussion, or the LP-IV section as
needed.

This file is the endpoint of Tier 1 of the econometrics
library. It draws on the IV note (`instrumental-variables.md`)
for the LP-IV section and on the DiD note
(`difference-in-differences.md`) for the heterogeneous-effects
framing that shows up in state-dependent LPs.

---

## What an LP is actually estimating

Before any estimator, the estimand. A local projection
estimates, for each horizon `h` from 0 out to some maximum
horizon `H`, the expected change in the outcome `h` periods
ahead in response to a unit shock to the variable of interest
today, controlling for a set of pre-shock conditioning
variables:

```
y_{t+h} = α_h + β_h · x_t + γ_h' · W_t + u_{t,h}    for h = 0, 1, ..., H
```

The sequence `{β_h}` as a function of `h` is the impulse
response function. Each horizon is a separate regression. The
IRF is a collection of partial regression coefficients, one
per horizon, each estimated independently.

Two things to hold in your head.

First, the LP β_h is a **conditional expectation of a future
outcome given the shock today**, not a parameter of a deep
model. Under the identification assumption that `x_t` is
orthogonal to the error `u_{t,h}` conditional on `W_t`, the LP
estimates the *causal* impulse response. Under weaker
assumptions (no orthogonality, just prediction), it estimates
the best linear predictor of `y_{t+h}` given `x_t` and
controls, which is descriptive rather than causal. The
distinction matters and should be stated explicitly in any
interpretation paragraph.

Second, **each horizon is its own regression**, with its own
sample, its own standard error, and its own residual. This is
LP's distinguishing feature relative to VARs: a VAR estimates
a joint dynamic model and derives IRFs as a nonlinear function
of the VAR parameters; an LP estimates each horizon's
coefficient directly from the data, without going through a
joint model.

Everything that follows is about what that design choice buys
you and what it costs.

---

## Why LPs exist: the Jordà argument

Òscar Jordà (2005) introduced local projections as an
alternative to VAR-based impulse response estimation. His
argument was methodological: VARs impose a parametric dynamic
structure (the VAR lag polynomial), and if that structure is
wrong, the IRFs you derive from it are biased, potentially
badly so at long horizons where the compounding of
misspecification accumulates. LPs avoid this by estimating
each horizon directly, so misspecification at horizon 1 does
not bias the estimate at horizon 10.

The cost, as Jordà acknowledged, is efficiency. If the VAR is
correctly specified, the VAR uses the cross-horizon
restrictions to estimate the IRF with smaller standard errors.
The LP discards those restrictions and pays for its robustness
with wider confidence bands.

This was the trade-off for roughly fifteen years: VARs for
efficiency, LPs for robustness. Applied macro split along this
line, with the monetary-policy literature (Romer and Romer,
Christiano-Eichenbaum-Evans, and the narrative-identification
tradition) increasingly reaching for LPs once the shocks were
in hand, while the structural VAR literature continued
refining identification schemes.

Then Plagborg-Møller and Wolf (2021) changed the framing.

---

## The Plagborg-Møller–Wolf equivalence

Plagborg-Møller and Wolf (2021) showed that LPs and VARs
estimate the *same* impulse response function in population —
as the sample size goes to infinity, the LP and the VAR
converge to the same IRF, provided both are consistently
estimated from the same underlying data-generating process. In
other words, the old dichotomy of "LPs for robustness, VARs
for efficiency" was half wrong. In population, they give the
same answer.

What this reframes:

**The difference between LPs and VARs is not about what they
target.** Both target the same population IRF. The difference
is about finite-sample bias-variance trade-offs, and about
which misspecifications each is robust to.

**VAR misspecification still matters.** If the VAR's lag
structure is wrong, the VAR estimate will be biased in finite
samples even though it would converge to the truth under the
true DGP. Plagborg-Møller and Wolf do not rescue misspecified
VARs; they show that correctly-specified VARs and LPs are
asymptotically equivalent.

**LPs are not automatically the more robust choice.** If the
LP's control set `W_t` is insufficient to identify the shock,
the LP is biased in the same way the VAR would be. LP
robustness comes from flexibility in lag structure, not from
some magical identification property.

**The choice between LP and VAR becomes a finite-sample
question.** Given the data you have, which estimator has
better properties? This depends on the true lag length, the
sample size, the noise structure, and the horizon of interest.
Plagborg-Møller and Wolf give concrete guidance: VARs tend to
have lower variance at short horizons, LPs at long horizons;
the crossover depends on how close the parametric VAR
assumption is to the truth.

The post-2021 consensus: **LPs and VARs are complements, not
substitutes, and the right move is often to report both.** If
they agree, the finding is robust to the LP/VAR choice. If
they disagree, the disagreement is informative — it tells you
something about where in the horizon-structure the estimators
differ, and the reader can decide which to trust based on
which misspecification they fear more.

Markus's rule: **do not choose between LP and VAR on aesthetic
grounds**. The choice is a finite-sample judgment and should
be justified in terms of the specific application.

---

## Long-horizon bias and inference: Montiel Olea and Plagborg-Møller

The second major correction to naive LP practice comes from
Montiel Olea and Plagborg-Møller (2021), who showed that the
standard way of doing inference on LPs at long horizons is
biased, and that the bias can be large.

The problem is specific. Standard LP inference uses
heteroskedasticity-and-autocorrelation-consistent (HAC)
standard errors — Newey-West or similar — to handle the
serial correlation in the error `u_{t,h}` that arises because
the outcome `y_{t+h}` overlaps across adjacent observations.
This is right as far as it goes. What Montiel Olea and
Plagborg-Møller showed is that HAC inference is *still* biased
at long horizons, because the finite-sample distribution of
the LP coefficient is non-normal in a way that HAC does not
correct.

Concretely: at horizon `h = 20`, a nominal 95% confidence
interval based on HAC standard errors can have true coverage
well below 90% in sample sizes typical of applied macro work.
The bias gets worse the longer the horizon and the smaller the
sample.

Their fix is a lag-augmented LP estimator with a specific bias
correction, implemented in the `localprojections` family of
packages (and manually implementable as an OLS regression with
extra lags of the shock on the right-hand side). The details
matter: the number of augmentation lags, the specific bias
correction, and the inference procedure are all spelled out in
the paper and the replication code.

Markus's rule: **for horizons beyond roughly 8–12 periods in
standard macro applications, use lag-augmented LPs and the
Montiel Olea–Plagborg-Møller inference procedure**. Reporting
standard HAC-based LPs at long horizons without the correction
is, in 2024, an unforced error.

This is the LP analogue of the "F > 10 is dead" point in the
IV file: the old workhorse inference procedure is no longer
good enough, the correction exists and is replicable, and
continuing to use the old version is a refusal to do the
available work.

---

## LP-IV: local projections with instruments

LP-IV is the LP framework applied to causal identification via
an instrument for the shock. It has become one of the central
tools in modern empirical macro, especially in the monetary-
policy and fiscal-policy literatures where clean shocks are
rare and instruments do most of the identifying work.

The setup. You have a shock variable `x_t` that you suspect
is endogenous — it responds to the outcome or to common
drivers of the outcome. You have an instrument `z_t` that is
plausibly exogenous to `y_{t+h}` except through `x_t`. The
LP-IV estimator is just the LP with `z_t` replacing `x_t` in
the right-hand side:

```
y_{t+h} = α_h + β_h · z_t + γ_h' · W_t + u_{t,h}
```

Under the exclusion restriction (the instrument affects the
outcome only through the shock) and relevance (the instrument
is correlated with the shock conditional on controls), `β_h`
is the causal impulse response of `y` to a unit shock of `x`,
scaled by the first-stage relationship between `z` and `x`.

Canonical applications:

- **Monetary policy**: Romer and Romer (2004) identification
  of monetary shocks from FOMC documents; high-frequency
  identification around FOMC announcements (Gürkaynak, Sack,
  and Swanson 2005; Gertler and Karadi 2015; Miranda-Agrippino
  and Ricco 2021).
- **Fiscal policy**: Ramey (2011, 2016) narrative identification
  of military spending shocks; Mertens and Ravn (2013) on tax
  shocks.
- **Oil and commodity shocks**: Känzig (2021) on OPEC
  announcements.

Everything in `instrumental-variables.md` about the IV
machinery applies here, with three extensions specific to LP-
IV.

**One: weak instruments at every horizon.** Because each
horizon is a separate regression, the first-stage strength
matters at each horizon, not just on average. Stock and Watson
(2018) discuss this in detail. The Olea-Pflueger effective F
should be computed and reported for the relevant first stage,
and the Anderson-Rubin machinery extends to LP-IV but requires
care because the inference is now horizon-by-horizon.

**Two: the LATE framing still applies, horizon by horizon.**
The LP-IV at horizon `h` estimates the effect of an
instrument-induced shock on the outcome at horizon `h`, for
the subset of periods whose shocks are moved by the
instrument. This is a LATE, in the usual Imbens-Angrist sense,
not a population average effect of "a monetary policy shock."
The compliers in LP-IV are the periods during which the
instrument moves the policy variable, and those periods may
be systematically different from the full sample. For
high-frequency monetary shock identification, the compliers
are FOMC announcement windows — not a random sample of
monetary history.

**Three: the external-instrument SVAR is the LP-IV's sibling,
not its rival.** Stock and Watson (2018) and Plagborg-Møller
and Wolf (2021) discuss the connection. In population, a
proxy-SVAR and an LP-IV using the same instrument identify
the same impulse response. In finite samples they can differ,
for the same lag-structure-robustness reasons as the general
LP-vs-VAR comparison. Report both when feasible.

Ramey (2016) is the canonical survey that walks through these
issues with concrete applications and should be the first
reference for any serious LP-IV project. Stock and Watson
(2018) is the methodological complement.

---

## State-dependent LPs

One of LP's genuine advantages over VARs is that it extends
cleanly to state-dependent impulse responses: you can estimate
different IRFs in different regimes (recessions vs
expansions, high- vs low-debt states, zero-lower-bound
periods vs normal times) by interacting the shock with an
indicator for the state.

The Auerbach-Gorodnichenko (2012, 2013) papers are the
canonical applications — they estimate different fiscal
multipliers in recessions and expansions using a smooth
transition between regimes and LP estimation. Ramey and
Zubairy (2018) pushed back on the specific magnitudes with a
longer historical sample and more careful specifications,
which is itself a useful methodological discussion about how
sensitive state-dependent LP estimates are to sample and
controls.

The setup. Define a state indicator `S_t ∈ [0, 1]` (or a
binary indicator, or a multinomial). Interact it with the
shock:

```
y_{t+h} = α_h + β_h^L · (1 - S_t) · x_t + β_h^H · S_t · x_t 
        + γ_h' · W_t + u_{t,h}
```

Now you have two IRF sequences, `{β_h^L}` and `{β_h^H}`, one
per state. The difference between them is the state
dependence, and it can be tested formally.

What to watch for.

**The state indicator is an endogenous object.** It is
measured at time `t`, before the shock, but it may be
correlated with unobservables that drive both the shock
response and the state assignment. A recession indicator is
not randomly assigned. This is the core critique in the Ramey-
Zubairy pushback: state-dependent multipliers can be
confounded by the business-cycle context that defines the
states.

**Sample sizes within states can be small.** If there are 20
recessions in the sample and each lasts 6 quarters, the
"recession" subsample is 120 observations and the
state-dependent coefficient is identified off those. LP
inference at long horizons on a subsample of 120 observations
is exactly where the Montiel Olea–Plagborg-Møller bias
problem is worst.

**The smooth-transition specification is not neutral.** The
Auerbach-Gorodnichenko smooth-transition uses a logistic
function of the unemployment rate. The specific functional
form and the smoothness parameter matter and should be
probed. Robustness to alternative state definitions is not
optional.

**State dependence is not the same as heterogeneous effects
in DiD.** The DiD literature's heterogeneity discussion
(see `difference-in-differences.md`) is about treatment
effects that vary across units with different baseline
characteristics. State-dependent LPs are about responses that
vary across aggregate states of the macroeconomy. Both are
forms of heterogeneity, but they live in different
econometric structures and should not be conflated in
interpretation.

The rule: state-dependent LPs are a powerful tool and they
are also a tool that invites overfitting and overclaiming.
Markus reports them when they are the point of the paper,
with explicit discussion of sample size, robustness to state
definition, and engagement with the Ramey-Zubairy critique.

---

## Panel local projections

LPs extend to panel data in the obvious way: replace the
single time series with a panel, add unit fixed effects, and
estimate horizon-by-horizon panel regressions. This has
become standard in cross-country macro, where the sample of
countries provides the unit dimension.

Key issues specific to panel LPs.

**Nickell bias.** Panel regressions with fixed effects and
lagged dependent variables suffer from Nickell bias, and
panel LPs inherit this because they typically include lagged
outcome controls. The bias shrinks with `T`, but in typical
cross-country samples where `T` is 40–60 years, it is
non-trivial. The standard fix — differencing out the unit
effects and instrumenting with further lags — creates its
own problems (see the Arellano-Bond discussion in
`panel-methods.md`).

**Cross-sectional dependence.** Countries are not
independent. Shocks to the US transmit to other countries,
and the LP error terms are cross-sectionally correlated.
Driscoll-Kraay standard errors handle this mechanically but
they do not fix the bias from cross-sectional dependence
interacting with dynamic specifications. Pesaran (2006)
common correlated effects and the Chudik-Pesaran extensions
are the more serious fix.

**Heterogeneity across units.** Different countries respond
differently to shocks. A panel LP with a single `β_h`
imposes a homogeneous response, which is almost certainly
wrong. Mean-group estimators (Pesaran and Smith 1995) allow
heterogeneity at the cost of efficiency. The right choice
depends on whether the question is "what is the average
response" or "how does the response vary across countries,"
and these are different questions with different estimators.

**The Turkish angle.** For a country like Turkey, which is a
single unit, panel LPs across emerging markets are one route
to borrowing identification from similar countries. The
caveat is that "similar" is doing a lot of work — Turkey's
monetary and political context since 2018 is distinctive
enough that pooling with other emerging markets can obscure
rather than illuminate. The cleaner move is often a
single-country LP with narrative identification of shocks,
at the cost of small-sample precision.

---

## Smooth local projections

Barnichon and Brownlees (2019) introduced smooth local
projections: penalized LP estimation that imposes smoothness
on the IRF across horizons via a shrinkage prior. This sits
between the unrestricted LP (which can have wiggly IRFs at
long horizons because each coefficient is estimated
independently) and the VAR (which imposes a tight parametric
structure).

The motivation is practical: LP IRFs can look visually
unstable at long horizons, with horizon-to-horizon jumps that
are probably noise rather than signal. Smooth LPs penalize
second differences in the IRF across horizons, producing
visually cleaner IRFs with smaller standard errors at the
cost of a smoothness assumption.

When to reach for it: when the unrestricted LP produces an
IRF that is hard to interpret because of horizon-to-horizon
noise, and when the smoothness assumption is defensible. The
fear: smoothness can hide genuine dynamics (sharp turning
points in the response, non-monotonic paths) and the
researcher should probe whether the smoothed IRF differs
substantively from the unrestricted one in ways that matter
for the conclusion.

Not every LP paper needs smooth LPs. The unrestricted version
is the default; smooth is a specific tool for specific
problems.

---

## The decision tree

Markus uses this to choose an LP-based strategy for a given
question:

**Q1: Is the question about impulse responses to a shock, or
about something else?**
LPs are a tool for estimating IRFs. If the question is about
a level relationship, a trend, a cointegration structure, or
a distributional outcome, LP is not the right tool. Go to the
relevant section of `modern-toolkit-references.md`.

**Q2: Do you have a credible shock, or do you need an
instrument for one?**
- Clean exogenous shock → standard LP, with the shock
  variable on the right-hand side. Identification comes from
  the shock's design.
- Endogenous shock, good instrument → LP-IV. Everything in
  `instrumental-variables.md` applies, plus the horizon-by-
  horizon considerations above.
- No clean shock, no instrument → the question is not
  identified as a causal IRF in this data. An LP can still
  be run for descriptive purposes (conditional forecasting,
  dynamic correlations) but must not be interpreted
  causally.

**Q3: What is the relevant horizon?**
- Short horizon (1–8 periods in quarterly data) → standard
  LP, HAC inference, no bias correction needed.
- Long horizon (beyond roughly 8–12 periods) → lag-augmented
  LP with Montiel Olea–Plagborg-Møller inference. This is
  where HAC-based inference goes wrong.
- Very long horizon (beyond 20 periods in quarterly data) →
  be skeptical of any IRF estimate at all. The Montiel
  Olea–Plagborg-Møller correction helps but the fundamental
  signal-to-noise problem at very long horizons is
  unavoidable.

**Q4: Do you want to compare to a VAR?**
Per Plagborg-Møller and Wolf (2021), LPs and VARs target the
same population object, so comparing them is cheap and
informative. The default move is to report both, especially
at horizons where they might differ. If they agree, the
finding is robust. If they disagree, you have something to
explain.

**Q5: Is the response plausibly state-dependent?**
- No, or no strong prior → homogeneous LP.
- Yes, and the states are well-defined → state-dependent LP
  with explicit discussion of the state definition, sample
  size per state, and robustness to alternative state
  measures. Engage with Ramey-Zubairy if the application is
  fiscal.

**Q6: Is this a panel or a single time series?**
- Single → standard LP, with the choices above.
- Panel → panel LP with fixed effects, Driscoll-Kraay
  standard errors at minimum, mean-group estimation if
  heterogeneity matters for the question. Be explicit about
  Nickell bias if `T` is modest.

**The default, when in doubt:** single-country or panel LP
depending on the data, lag-augmented estimation with
Montiel Olea–Plagborg-Møller inference for horizons beyond
8–12, standard LP for short horizons, LP-IV when the shock
is endogenous and a good instrument exists, state dependence
only when the question demands it, comparison to a VAR
reported as a robustness check.

---

## Common failure modes

**Using standard HAC inference at long horizons.** The
single most common modern LP error. Reporting LP IRFs at
horizon 20 with Newey-West standard errors in 2024 is the LP
equivalent of reporting F = 14 in an IV paper. The Montiel
Olea–Plagborg-Møller correction is available and replicable.
Use it.

**Interpreting LP-IV as an average policy effect.** The LATE
framing applies. An LP-IV on high-frequency monetary shocks
identifies the effect of monetary shocks *during FOMC
announcement windows*, not the effect of "monetary policy"
in general. The interpretation paragraph should name the
compliers.

**Ignoring first-stage weakness in LP-IV.** The first stage
can be strong on average and weak at some horizons, or vice
versa. Report the Olea-Pflueger effective F for the relevant
first stage, and use Anderson-Rubin-style weak-instrument-
robust inference if it matters.

**Treating LP and VAR as a methodological choice rather than
a finite-sample one.** Post-Plagborg-Møller-Wolf, the choice
is about misspecification robustness in finite samples, not
about what you are estimating. If the two disagree, the
disagreement is a finding, not an embarrassment.

**State-dependent LPs without engaging the pushback.**
Auerbach-Gorodnichenko is not the final word. Ramey-Zubairy
showed the magnitudes are sensitive to sample and
specification. A state-dependent LP paper that does not
engage with this is a paper that has not done its homework.

**Over-smoothing with smooth LPs.** Smooth LPs can hide
dynamics. If the unrestricted LP has a turning point at
horizon 5 and the smoothed version does not, that is
information about your smoothness prior, not just a cleaner
picture.

**Panel LPs ignoring cross-sectional dependence.** Country
shocks transmit. Driscoll-Kraay is a mechanical fix;
common-correlated-effects is the more serious treatment. Do
one of them; do not ignore the problem.

**Forgetting that LP is still a regression.** Everything in
`50_Workflows/run-a-regression-properly.md` applies: name the
estimand, specify identification, discuss what variation the
estimator is using, choose the right inference. LP does not
escape the workflow just because it is a macro tool.

---

## What to report

When Markus reports an LP-based result, the deliverable
includes:

1. **The estimand in plain language.** "The impulse response
   of the unemployment rate to a contractionary monetary
   shock at horizons 0 through 20 quarters, identified by
   high-frequency surprises in federal funds futures around
   FOMC announcements."
2. **The identification**, spelled out. For a standard LP,
   what makes the shock exogenous. For LP-IV, the instrument
   and the exclusion restriction, with the same honesty
   standards as `instrumental-variables.md`.
3. **The LP specification**, including the lag length, the
   control set, and whether lag augmentation is being used.
   If not lag-augmented, a defense of why not (usually: short
   horizons only).
4. **The IRF plot.** Point estimates across horizons with
   confidence bands. Clean axes, horizons on the x-axis, a
   zero line, and the response units on the y-axis. No stars,
   per `Principles.md`.
5. **Inference**, computed correctly for the horizon. Standard
   HAC for short horizons, Montiel Olea–Plagborg-Møller for
   long horizons. State which is being used and why.
6. **A comparison to a VAR** at the same horizons, reported
   as a robustness check, with discussion of any disagreement.
7. **For LP-IV**: first-stage diagnostics (Olea-Pflueger F,
   AR intervals if the F is marginal), horizon-by-horizon if
   the strength varies.
8. **For state-dependent LPs**: the state definition and its
   justification, the sample size per state, robustness to
   alternative state measures, engagement with the relevant
   pushback literature.
9. **A statement of which checks would have changed the
   conclusion**, per the regression workflow.

The interpretation paragraph names the estimand explicitly,
locates the result in the existing literature, and is honest
about what the design can and cannot say. "A monetary policy
shock reduces unemployment by X" is not interpretation;
"the LP-IV estimate, identified by high-frequency FOMC
surprises over 1994–2019, implies that a one-standard-deviation
contractionary shock raises unemployment by X percentage
points at peak (quarter `h`), with the estimate based on
Montiel Olea–Plagborg-Møller lag-augmented inference and
robust to a comparable proxy-SVAR specification" is.

---

## Implementation, by language

**R** is the mature ecosystem for LPs, especially for the
post-2021 methodological developments.

- `lpirfs` (Adämmer) — the reference implementation. Handles
  standard LPs, LP-IV, state-dependent LPs, panel LPs, and
  most of the visualization needed. The natural starting
  point for most LP work.
- `fixest::feols` — for custom LP specifications and for
  efficient implementation with high-dimensional fixed
  effects (useful in panel LPs with many countries and long
  time series). LP is just a sequence of OLS regressions,
  and `fixest` is the fastest way to run them.
- For the Montiel Olea–Plagborg-Møller lag-augmented
  inference, the authors provide replication code from the
  paper. It is not yet a standalone package in the
  `lpirfs` style as of the most recent versions I have
  knowledge of — check whether an update has integrated it
  before rolling your own.
- For smooth LPs, Barnichon and Brownlees provide
  replication code; a standalone R implementation is
  available but less widely used than `lpirfs`.

**Stata** has solid LP support:

- `lpirf` and related user-written commands for standard LP
  estimation.
- Custom LP specifications are easy to roll by hand with
  `regress` inside a loop over horizons.
- The Montiel Olea–Plagborg-Møller bias correction requires
  either custom coding or calling out to R.

**Python** is adequate but less mature than R for LPs
specifically.

- `localprojections` is a package but less feature-rich than
  `lpirfs`.
- `pyfixest` handles the regression side efficiently.
- For most applications Markus implements LPs directly in
  Python as a loop of OLS regressions with appropriate
  standard errors. The method is simple enough that this is
  not a burden.
- `statsmodels` has the underlying regression and HAC
  inference machinery but nothing LP-specific.

Markus's default for new LP work is R, following `lpirfs` for
the standard cases and calling out to the Montiel
Olea–Plagborg-Møller replication code for long-horizon
inference. The R ecosystem is where the methodology lives.
Python is fine for quick exploratory work but the
publication-quality inference machinery is in R.

---

## What this note doesn't cover, and where to find it

**Structural VARs in detail.** Sign restrictions, recursive
identification, proxy SVARs beyond their connection to
LP-IV. These are in `var-identification.md`.

**Narrative identification of shocks.** Romer-Romer, Ramey,
Mertens-Ravn. The narrative side of shock identification is
worth its own treatment and is mostly field-specific. A
`narrative-shocks.md` note in v0.3 if it earns its place.

**Nonlinear and asymmetric LPs.** Threshold models,
regime-switching with endogenous transitions, asymmetric
responses to positive vs negative shocks. Covered in
passing above in the state-dependent section but a deeper
treatment belongs elsewhere.

**Frequency-domain methods.** Spectral analysis, wavelets,
frequency-specific impulse responses. These are real tools
for certain questions and are not covered here at all.

**Time-varying-parameter VARs.** Primiceri (2005) and the
literature that followed. When the question is about how
the IRF itself evolves over time, TVP-VAR is one approach;
rolling-window LPs are another; neither is covered in
depth here.

**Forecasting applications.** LPs are sometimes used for
conditional forecasting rather than impulse-response
estimation. The machinery is the same; the interpretation
is different. See the forecasting section of
`tools-and-integrations.md` and the
`modern-toolkit-references.md` entry on time series ML for
the broader forecasting landscape.

These extensions get their own treatment when a project
actually demands them. Don't speculatively expand this file.

---

## Cross-references

- `00_Identity/Principles.md` — sections on identification,
  causation, and uncertainty. The general posture this note
  operationalizes, especially the commitment that "every
  identification strategy makes assumptions you cannot test"
  and the refusal to treat long-horizon IRFs as more certain
  than the data support.
- `10_Methods/modern-toolkit-references.md` — the directory
  entry that points here. Update it if this file's content
  changes the recommendations.
- `10_Methods/Econometrics/instrumental-variables.md` — the
  parent note for the LP-IV section. The LATE framing, the
  weak-instrument machinery, and the exclusion-restriction
  honesty all carry over and are not rederived here. Read
  that file first if you are approaching LP-IV.
- `10_Methods/Econometrics/difference-in-differences.md` —
  the heterogeneous-effects framing in the state-dependent
  LP section is the sibling of the treatment-effect-
  heterogeneity discussion in DiD. Different settings, same
  underlying posture: heterogeneity is the rule, not the
  exception.
- `10_Methods/Econometrics/shift-share-instruments.md` —
  shift-share instruments can enter LP-IV as the external
  instrument when the shock is constructed from sectoral
  variation. The identification-story discipline from the
  shift-share note applies when this is being done.
- `50_Workflows/run-a-regression-properly.md` — LPs are
  still regressions. Stage 4 (specify identification, not
  just the equation) and Stage 1 (name the estimand) apply.

---

*Last updated: 2026-04. The LP literature has consolidated
substantially since Plagborg-Møller and Wolf (2021) and
Montiel Olea and Plagborg-Møller (2021); the core points —
LPs and VARs target the same object, HAC at long horizons is
biased, lag augmentation is the fix — are stable. The state-
dependent and panel LP literatures are more active and will
need revision as they develop. The LP-IV section will need
updating as the integration of high-frequency monetary
identification and LP machinery continues to evolve.*