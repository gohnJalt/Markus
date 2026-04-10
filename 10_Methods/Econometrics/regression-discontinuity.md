---
tags: [econometrics, causal-inference, rdd, regression-discontinuity]
related: [modern-toolkit-references, instrumental-variables, run-a-regression-properly, Principles, linear-algebra]
---

# Regression Discontinuity Design

This is the working note. It assumes you know what an RDD is
and why anyone runs one: there is a cutoff in a running
variable, crossing the cutoff determines or strongly influences
treatment, and the discontinuity in outcomes at the cutoff
identifies a causal effect. What this note tries to do is give
Markus enough fluency in the modern RD literature — centered on
Cattaneo, Idrobo, and Titiunik's Cambridge Elements volumes as
the canonical reference — to *reason* about whether a given RD
design is credible, what it is actually estimating, and which
of the post-2014 tools apply. The entry in
`modern-toolkit-references.md` is a directory; this is the
actual map.

The centerpiece is the Calonico-Cattaneo-Titiunik (2014,
*Econometrica*) bandwidth-selection and bias-corrected
inference procedure. The conventional textbook approach —
choose a bandwidth by judgment, fit a polynomial, report
standard errors — is wrong in a precise way that CCT fixes. Get
this right before anything else.

---

## What RD is actually estimating

The thing to hold in your head before any estimator is the
**estimand**. RDD is not estimating "the effect of crossing the
threshold on Y." It is estimating a **local average treatment
effect at the cutoff**: the average effect of treatment for
units whose running variable value equals the cutoff $c$.

Formally, let $X$ be the running variable, $c$ the cutoff, $D$
the treatment indicator (1 if $X \geq c$ in the sharp case),
and $Y(0), Y(1)$ potential outcomes. The sharp RD estimand is:

$$\tau_{SRD} = E[Y(1) - Y(0) \mid X = c] = \lim_{x \downarrow c} E[Y \mid X = x] - \lim_{x \uparrow c} E[Y \mid X = x].$$

This is a *limit*: the effect for the unit with running
variable exactly at the cutoff. It is identified by continuity
of the potential outcome regression functions — $E[Y(0) \mid X
= x]$ and $E[Y(1) \mid X = x]$ are continuous at $c$ — which
means that in the absence of treatment the outcome would not
jump at $c$. Hahn, Todd, and van der Klaauw (2001) is the
formal identification paper.

Three consequences of taking this seriously.

First, the estimand is **local**. It applies to units at the
margin — those with running variable values near $c$. Units far
from the cutoff are used to estimate the regression functions
(to extrapolate to $c$), but they are not in the target
population. Claiming the RD estimate generalizes to the broad
population requires an assumption — typically that treatment
effect heterogeneity in the running variable is limited — that
should be stated explicitly and defended, not assumed silently.

Second, what the estimand is *not*. It is not the ATE in the
population, the ATT, or the LATE in the IV sense. In a fuzzy
RD (§3 below), the estimand does become a LATE — but for
compliers at the cutoff, a further restriction. In the sharp
case, there are no compliers to worry about: everyone at or
above the cutoff is treated. But "at the cutoff" is doing all
the restriction work.

Third, the running variable is a selection mechanism. Units do
not cross the cutoff randomly — they have their own values of
$X$ for reasons correlated with their outcomes. The
identification claim is not random assignment; it is that the
*precise value* of $X$ near $c$ is as good as random, because
agents cannot precisely control which side of $c$ they fall on.
The density test (§4) is the principal diagnostic for whether
this claim is plausible.

---

## The naive approach and why it's wrong

The textbook RD procedure runs a regression of $Y$ on $D$, $X$,
and $X \times D$ in a window around the cutoff:

$$Y_i = \alpha + \tau D_i + \beta_1 (X_i - c) + \beta_2 D_i (X_i - c) + \varepsilon_i,$$

using all observations with $|X_i - c| \leq h$ for some
bandwidth $h$. The coefficient $\tau$ is the RD estimate.

Two problems with this as a default procedure.

**Global polynomials.** An older tradition uses the full sample
and fits a high-degree polynomial in $X$:

$$Y_i = \sum_{k=0}^p \alpha_k X_i^k + \tau D_i + \sum_{k=1}^p \beta_k D_i X_i^k + \varepsilon_i.$$

Gelman and Imbens (2019, *JBES*) document the failure modes:
high-degree global polynomials exhibit Runge's phenomenon near
the boundary, producing large-amplitude oscillations that
generate spurious discontinuities. The estimate of $\tau$ is
sensitive to polynomial degree in a way that has no principled
resolution. Do not use global polynomial regression for RDD.
The canonical approach is **local polynomial** — fit within a
bandwidth $h$ of the cutoff, using a low-order polynomial
(local linear or local quadratic is standard).

**Discretionary bandwidth.** Without a principled procedure for
choosing $h$, the bandwidth becomes a specification choice that
the researcher can — consciously or not — tune toward a
preferred result. The garden of forking paths problem is acute
here: different reasonable bandwidths can give qualitatively
different estimates, and reporting only one creates the illusion
of precision.

Both failures are fixable: local polynomial + principled
bandwidth selection. The next section is about that fix.

---

## Bandwidth selection and bias-corrected inference (CCT)

This is the centerpiece of the modern RD literature. The
Calonico-Cattaneo-Titiunik (2014, *Econometrica*) paper gives
the standard and `rdrobust` is its implementation.

### The bias problem

The local linear estimator at bandwidth $h$ has a bias that
depends on $h$. A smaller bandwidth reduces bias (fewer
observations away from $c$ pull the fit) but increases variance
(fewer observations). The **MSE-optimal bandwidth** balances
these:

$$h^*_{MSE} = C_{MSE} \cdot n^{-1/5},$$

where $C_{MSE}$ depends on the curvature of the regression
function near $c$ (estimated from the data). The
Imbens-Kalyanaraman (2012, *ReStud*) bandwidth was an early
implementation of this idea.

The problem: at $h^*_{MSE}$, the bias is of the same order as
the standard deviation. Standard confidence intervals centered
at the estimate do not cover the true parameter at the nominal
rate. The bias is not negligible at the optimal bandwidth, and
pretending it is — as the naive procedure does — produces
under-covering confidence intervals.

### The CCT solution: robust bias correction

Calonico, Cattaneo, and Titiunik (2014) solve the problem in
two steps.

**Step 1: Estimate and subtract the bias.** Using a larger
secondary bandwidth $b > h$, fit a higher-order polynomial
(local quadratic on each side) and use it to estimate the bias
of the local linear estimator at $h$. Call this $\hat B$.
Subtract it from the original estimate: $\hat\tau_{bc} =
\hat\tau - \hat B$. This is the **bias-corrected** estimate.

**Step 2: Use robust standard errors.** The bias-corrected
estimate has additional variance from the bias estimation step.
Standard errors that ignore this additional term understate the
true uncertainty. The **robust** (RBC) standard errors account
for it. The valid confidence interval is:

$$\hat\tau_{bc} \pm 1.96 \cdot \widehat{se}_{rb}.$$

The key output from `rdrobust` is the row labelled **"Robust"**
in the results table — not the "Conventional" row. The
Conventional row gives the naive estimate with standard errors
that do not account for bias; the Robust row gives the
bias-corrected estimate with valid coverage. This distinction
matters and is regularly missed in applied work.

### Bandwidth selection: the CCT procedure

`rdrobust` implements the CCT bandwidth selector by default.
The bandwidth $h$ is chosen to minimize the MSE of the
bias-corrected estimator (not the uncorrected one — this is a
subtlety of the 2014 paper's update relative to the IK
procedure). A pilot bandwidth is used to estimate the local
second derivatives (curvature) that determine the optimal $h$
and $b$.

**Practical advice:**
- Use the CCT default. Do not hand-pick the bandwidth.
- Report robustness to half and double the CCT bandwidth in the
  paper. If the estimate changes substantially with bandwidth,
  that is a diagnostic: the smoothness assumption is not well-
  supported near the cutoff.
- The triangular kernel (weighting observations by distance from
  the cutoff) is the default in `rdrobust` and is MSE-optimal
  for the point estimator. The rectangular (uniform) kernel is
  an alternative for robustness checks.

### Covariate adjustment

Adding predetermined covariates to the local polynomial
regression can sharply reduce variance without introducing bias
(under continuity of the covariates). Calonico, Cattaneo,
Farrell, and Titiunik (2019, *ReStat*) formalize this:
add covariates to both sides of the local regression. The
`covs` option in `rdrobust` implements this. Use it when
predetermined covariates are available — the efficiency gain
can be large.

---

## Fuzzy RDD

In the **fuzzy** design, crossing the cutoff increases the
probability of treatment but does not determine it:

$$\Pr(D = 1 \mid X = x) \begin{cases} \uparrow & \text{at } x = c \end{cases} \quad \text{but } \Pr(D=1|X \geq c) < 1 \text{ or } \Pr(D=1|X < c) > 0.$$

The fuzzy RD estimand is the ratio of the jump in outcome to
the jump in treatment probability:

$$\tau_{FRD} = \frac{\lim_{x \downarrow c} E[Y|X=x] - \lim_{x \uparrow c} E[Y|X=x]}{\lim_{x \downarrow c} E[D|X=x] - \lim_{x \uparrow c} E[D|X=x]}.$$

This is a local Wald ratio. Under monotonicity — no unit moves
from treated to untreated when crossing the cutoff — $\tau_{FRD}$
is a LATE for the **compliers at the cutoff**: units who are
induced to take treatment by their running variable crossing $c$.
This is an even more local estimand than the sharp case: it
applies to compliers (not all units) at the cutoff (not the
full population).

**The first stage must be strong.** The denominator of the Wald
ratio must be nonzero and reasonably large. Measure the
first-stage jump: if $\Pr(D=1|X \geq c) - \Pr(D=1|X<c)$ at
the cutoff is less than 0.1, the design is weak in a sense
analogous to weak instruments (`instrumental-variables.md` §4).
The RD analog of the first-stage F-statistic is the $t$-
statistic on the first-stage discontinuity, computed by
`rdrobust` when the `fuzzy` option is used. Treat a jump below
10% as a warning that the design may not deliver a credible
estimate.

**Implementation.** Pass the treatment variable to the `fuzzy`
argument of `rdrobust`. The package runs the reduced form and
first stage using local polynomial estimation with the CCT
bandwidth, then computes the ratio and the RBC standard errors
for the ratio. Report both the reduced form and first stage
estimates alongside the fuzzy RD estimate.

---

## Density tests and manipulation

If agents can precisely manipulate their running variable to
land just above or below the cutoff, the continuity assumption
fails: the composition of units on each side of $c$ is not
comparable. The density test is the principal diagnostic.

### The logic

If manipulation is present — agents bunch just above $c$ to get
treatment, or just below to avoid it — the density of the
running variable will have a discontinuity at $c$: there will
be more mass on the "preferred" side than a smooth density
would predict. The density test asks: is the density of $X$
continuous at $c$?

This is a necessary but not sufficient condition for no
manipulation. If the test fails, manipulation is likely. If the
test passes, manipulation is not ruled out — agents may
manipulate in ways that preserve density continuity (e.g.,
symmetric bunching, or corruption that replaces one unit with
another of similar $X$ on the preferred side). The test is a
diagnostic, not a license.

### McCrary (2008) and rddensity (CJM 2020)

**McCrary (2008, *JoE*)** is the original test. It divides the
running variable into bins, fits a piecewise linear density on
each side, and tests for a jump at $c$.

**Cattaneo, Jansson, and Ma (2020, *JASA*)** — the `rddensity`
package — is the modern replacement. It uses local polynomial
density estimation rather than a histogram, with bias-corrected
inference analogous to `rdrobust`. It has better size control
and power properties than the McCrary test and is the current
standard.

**Visuals matter.** Plot the density histogram of the running
variable centered on the cutoff. Heaping — sharp spikes just
above or below $c$ — is visible by eye and should be reported
even if the formal test is marginal. Heaping is common with
integer-valued running variables (test scores, ages in months,
school enrollment cutoffs) and often requires separate attention
(§7 below).

---

## Covariate balance and placebo tests

The continuity assumption has two main observable implications
beyond the density test.

**Covariate balance.** Predetermined covariates — characteristics
determined before the running variable was realized — should
show no discontinuity at $c$. Run `rdrobust` separately for
each baseline covariate as the outcome. A statistically
significant jump in a predetermined covariate is evidence
against continuity. Report this as a balance table: one row per
covariate, with the RD estimate, RBC standard error, and
p-value.

**Placebo outcomes.** If outcomes determined before the cutoff
was crossed can be identified, they should show no discontinuity.
In school-based designs, pre-enrollment test scores are a
placebo. In employment designs, wages in the year before the
policy are a placebo.

A failed covariate balance check is more damning than a failed
density test. Density continuity can hold despite selective
entry and exit; covariate imbalance directly contradicts the
claim that units on each side of the cutoff are comparable.

---

## The continuity assumption, honestly

RDD rests on one untestable assumption: the **continuity of
potential outcomes** at the cutoff. Specifically:

$$E[Y(0) \mid X = x] \text{ is continuous at } x = c.$$

This says that in the absence of treatment, the mean outcome
would not jump at $c$. This is the assumption that makes the
left-limit ($x \uparrow c$) a valid counterfactual for the
right-limit ($x \downarrow c$).

This assumption is **untestable** in the same way that the
exclusion restriction in IV is untestable. The density test and
covariate balance checks test *observable implications* of
continuity, but they cannot confirm the assumption itself.
Continuity can fail while the density is smooth — if, for
example, officials selectively admit units with high potential
outcomes above the cutoff while drawing them from the same
$X$-distribution.

The honest characterization of the design is: "We assume that
units just above and just below the cutoff are comparable in
potential outcomes. We present evidence consistent with this —
the density is smooth, predetermined covariates are balanced —
but these tests do not confirm the assumption." Researchers who
claim the RD "proves" continuity have confused a testable
implication with the assumption itself.

The **internal validity** argument for RD is local and design-
based: near the cutoff, selection on $X$ is approximately
random because agents cannot precisely control $X$ near the
boundary. This argument is strongest when: (1) $X$ is
determined by factors partly outside the agent's control
(weather, test timing, bureaucratic rounding), (2) agents do
not know the exact threshold in advance, and (3) the prize from
crossing the threshold is not so large that it induces
maximal effort to manipulate $X$.

---

## Decision tree

**Q1. Is treatment deterministic at the cutoff?**
- Yes (everyone above $c$ treated, nobody below) → **Sharp RD**
- No (probability jumps but not to 0/1) → **Fuzzy RD**

**Q2. (Fuzzy only) How large is the first-stage jump?**
- $\Delta\Pr(D=1) \geq 0.2$ at the cutoff → proceed
- $0.1 \leq \Delta\Pr(D=1) < 0.2$ → weak; report with caution
- $< 0.1$ → design is likely too weak to credibly estimate; reconsider framing as IV

**Q3. Does the density test pass?**
- Pass (`rddensity` p-value $> 0.10$) → proceed to covariate balance
- Fail → manipulation is likely; report this prominently; results should be treated with high skepticism; consider whether the design can be salvaged by restricting to units that could not have manipulated (e.g., exclude observations in a donut hole around the cutoff)

**Q4. Are predetermined covariates balanced?**
- All balanced → evidence consistent with continuity
- One or two marginally significant → expected under multiple testing; use Bonferroni or Holm corrections
- Several significant → continuity likely violated; results are fragile

**Q5. Is there a single cutoff or multiple?**
- Single cutoff → standard analysis
- Multiple cutoffs → consider pooling by normalizing the running variable as $X - c_k$ for each cutoff $c_k$; report heterogeneity across cutoffs if the samples are large enough

**Q6. Is the running variable continuous or discrete?**
- Continuous (or fine-grained) → standard analysis
- Discrete (integer test scores, age in months, etc.) → see §7 failure modes; consider whether the design supports RD assumptions at all

**Default when in doubt:** `rdrobust(y, x)` with the default CCT bandwidth, triangular kernel, and report the Robust confidence interval. Add `covs = baseline_covariates` if available. Run `rddensity(x)` and `rdplot(y, x)`. Do not report the Conventional CI as the main result.

---

## Common failure modes

**Reporting the Conventional CI at the MSE-optimal bandwidth.**
The most common technical error. The Conventional confidence
interval from `rdrobust` does not have valid coverage at the
MSE-optimal bandwidth. Report the Robust (RBC) CI. Always.

**Using a global polynomial.** Still appears in published work,
especially in papers following older templates. Local linear is
the correct default; local quadratic is a robustness check.
Global polynomials are not a robustness check for local linear;
they are a different and worse method.

**Discretionary bandwidth.** Choosing the bandwidth "by looking
at the data" without a principled procedure is the RD analog of
choosing the comparison group by looking at the pre-trends. Use
CCT. Report sensitivity to half and double the bandwidth in a
figure or table.

**Skipping the density test.** Inexplicable in post-2008 work.
Run `rddensity(x)` (or at minimum the McCrary test) and report
it. If you do not report the density test, a sophisticated
reviewer will ask why.

**Conflating local and global effects.** The RD estimate applies
to units at the margin. Policy implications that require
knowing the treatment effect for units far from the cutoff
(e.g., "this program works and should be expanded to a non-
marginal population") require additional assumptions. State them.

**Integer running variables and heaping.** When $X$ takes
integer values (exact test scores, age in months, school-grade
cutoffs), several problems arise: (1) there may be bunching at
round numbers unrelated to the cutoff; (2) the local polynomial
approximation assumes $X$ is continuous, which fails at the
support level; (3) standard errors need to account for the
discrete support. The CCT procedure handles some of these
through its bias correction, but visible spikes at round numbers
near the cutoff should be investigated and reported. The
"donut hole" approach — excluding a small window exactly around
the cutoff — can help diagnose whether the effect is driven by
heaping.

**Endogenous cutoff.** If the cutoff value $c$ was set in
response to the outcome variable — a budget threshold chosen
to match past performance, a test score cutoff set to achieve a
target pass rate — the running variable is correlated with
potential outcomes at $c$ by construction, and continuity fails.
Always investigate the history of the threshold.

**Spillovers and contamination.** Units just below the cutoff
may be affected by the treatment of units just above it —
through market competition, peer effects, or resource
reallocation. In this case, the control group is not clean and
the RD estimate mixes the treatment effect with the spillover.
Report any known mechanisms through which spillovers could
occur.

**Over-precision at small sample sizes.** When the bandwidth
window contains fewer than 50-100 observations on each side, the
local polynomial fit is extrapolating heavily. The RBC
confidence interval will be wide, but point estimates can still
be misleading in their apparent specificity. Report the
effective sample size (number of observations within the
bandwidth on each side) and flag when it is small.

---

## What to report

**1. The RD plot.** Use `rdplot(y, x)` — it constructs the
bin-mean scatter on each side of the cutoff with the appropriate
number of bins determined by an integrated MSE criterion. Do not
manually adjust the bin width to make the discontinuity look
larger or smaller. The plot should make the discontinuity (if
any) visible without requiring imagination.

**2. The density plot and test.** A histogram of the running
variable with the cutoff marked. The `rddensity` output: the
estimated density on each side at the cutoff, the difference,
and the p-value for the test of continuity.

**3. Covariate balance table.** One row per predetermined
covariate. RD estimate, RBC standard error, p-value. No stars
(per the convention in `run-a-regression-properly.md`);
confidence intervals instead.

**4. The main RD estimate.** The bias-corrected estimate
$\hat\tau_{bc}$ with the Robust confidence interval. The
bandwidth $h$ (CCT). The effective sample size on each side of
the cutoff within $h$.

**5. Bandwidth robustness.** A table or plot of estimates at
$h/2$, $h$, and $2h$. If the estimate is stable across
bandwidths, say so. If it changes materially, report the range
and note that the result is bandwidth-sensitive.

**6. (Fuzzy only) First stage and reduced form.** Separately
report the jump in $\Pr(D=1|X=x)$ at $c$ (first stage) and the
jump in $E[Y|X=x]$ at $c$ (reduced form), each with RBC CIs.
The fuzzy RD estimate is their ratio. Report all three.

**7. Interpretation.** Name the estimand explicitly: "We
estimate the average effect of [treatment] for units at the
margin of [cutoff criterion], which in [year/context] includes
units with [description of the marginal unit]." State what the
design cannot say about units far from the cutoff. Report
whether the continuity assumption is consistent with the
diagnostic evidence presented, and whether any of the checks
came close to failing.

---

## Implementation, by language

**R** is the canonical home for RD estimation. All packages
are authored by Calonico, Cattaneo, Farrell, Titiunik, and
collaborators — the same group writing the methodology papers.

- `rdrobust` — the main estimation package. `rdrobust(y, x)`
  for sharp RD; `rdrobust(y, x, fuzzy = d)` for fuzzy RD;
  `rdrobust(y, x, covs = covariates)` with covariate adjustment.
  Always read the **Robust** row of the output.
- `rddensity` — the CJM density test and density plot.
  `rddensity(x)` for the test; `rdplotdensity(rddensity(x), x)`
  for the visual.
- `rdplot` — the canonical RD scatter plot with MSE-optimal
  bins. `rdplot(y, x)`.
- `rdlocrand` — randomization inference for RD, for small
  windows where asymptotics are unreliable (Cattaneo, Frandsen,
  Titiunik 2015).
- `rdmulti` — multi-cutoff and multi-score RD (Cattaneo, Keele,
  Titiunik et al.).

Markus's default for all RD work is `rdrobust` + `rddensity` +
`rdplot` in R.

**Stata** has the same suite with the same names (`rdrobust`,
`rddensity`, `rdplot`). The implementations are maintained in
parallel and produce identical results. The Stata code and
accompanying documentation in Cattaneo, Idrobo, and Titiunik's
*A Practical Introduction to Regression Discontinuity Designs*
(Cambridge Elements) is the best step-by-step reference.

**Python** has a `rdrobust` port that replicates the R output
for the core estimator. Less mature than R or Stata for the
auxiliary tools (`rddensity`, `rdmulti`). For serious RD work
in Python, call R via `rpy2` for the full suite.

---

## What this note doesn't cover, and where to find it

**Kink designs.** In a regression kink design (RKD), the
treatment level (not indicator) has a kink at the cutoff. The
estimand is the ratio of the kink in the outcome to the kink
in the treatment schedule. Card, Lee, Pei, and Weber (2015,
*Econometrica*) is the canonical reference. The identification
assumptions are stronger than in the level-discontinuity case.
`rdrobust` handles kinks via the `deriv` option.

**Geographic and boundary RD.** When the "cutoff" is a line on
a map (administrative boundary, electoral district, school
catchment area), the running variable is two-dimensional. Keele
and Titiunik (2015) and Cattaneo et al. on geographic RD.
The key challenges: which direction is "the running variable,"
how to define the bandwidth in two dimensions, and how to
handle boundary non-smoothness.

**Multi-score RD.** When treatment is determined by multiple
running variables simultaneously (a student must score above
$c_1$ on exam 1 AND above $c_2$ on exam 2), the design
involves an intersection of cutoffs. The estimand and
estimation approach differ from single-running-variable RD.

**Dynamic treatment and time-of-treatment variation.** If units
cross the cutoff at different times and treatment effects
evolve, the single-period RD estimand aggregates across
potentially heterogeneous effects. The connection to event-study
and DiD methods (`difference-in-differences.md`) is active
research.

**RD with interference.** Standard RD assumes SUTVA — the
outcome of unit $i$ depends only on unit $i$'s treatment. When
treated units affect nearby untreated units (through markets,
peer effects, or physical proximity), SUTVA fails and the RD
estimate mixes treatment and spillover effects.

**Extrapolation beyond the cutoff.** Extrapolating the RD
estimate to policy contexts that affect units far from the
margin requires either a structural model of treatment effect
heterogeneity or explicit assumptions about how effects vary
across the running variable distribution. Neither is given
automatically by the RD design.

---

## Cross-references

- `10_Methods/modern-toolkit-references.md` — the directory
  entry for RDD with canonical papers and the RD-adjacent
  methods (IV, DiD).
- `10_Methods/Econometrics/instrumental-variables.md` — fuzzy
  RD is local Wald estimation; the first-stage strength
  considerations and LATE interpretation there apply here at
  the cutoff. The weak-instruments discussion (§4 there) is
  the closest analog to the weak-first-stage problem in fuzzy RD.
- `10_Methods/Econometrics/difference-in-differences.md` — RD
  and DiD are both design-based strategies; the question of
  which is appropriate is usually answered by the data
  structure (panel + staggered treatment → DiD; running
  variable + threshold → RD).
- `20_Math/linear-algebra.md` — the local polynomial
  regression underlying `rdrobust` is weighted least squares;
  the hat matrix (§2.4 there) is how leverage and the bias
  estimate are computed.
- `50_Workflows/run-a-regression-properly.md` — the
  robustness-check convention (report checks that would change
  your mind if they failed) applies directly to bandwidth
  sensitivity; the no-stars convention applies to the balance
  table and main results.
- `00_Identity/Principles.md` — the estimand-first posture
  (§2 there) and the honesty about assumptions (§9 there)
  apply throughout; the "design over estimator" stance (§3
  there) is what the CCT procedure implements.

---

*Last updated: April 2026.*
