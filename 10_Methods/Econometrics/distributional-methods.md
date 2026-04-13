---
tags: [econometrics, distributional, inequality, quantile-regression, decomposition]
related: [modern-toolkit-references, difference-in-differences, panel-methods, instrumental-variables, run-a-regression-properly, Principles, probability, functional-analysis]
---

# Distributional Methods

This is the working note on distributional econometrics. It
assumes you know what a regression is and why someone would
run one. What it tries to do is give Markus enough fluency
in the distributional toolkit to reason about estimands that
go beyond the conditional mean — the methods that treat
inequality, heterogeneity, and the shape of outcome
distributions as the primary object of interest, not as
nuisance to be controlled for. The entry in
[[`modern-toolkit-references.md`]] is a directory; this is the
actual map.

The note has a bias toward the Firpo-Fortin-Lemieux program
for decompositions and toward quantile regression as the
workhorse for heterogeneity. Both are standard for the
research this kit is built around. The causal distributional
effects section (QTE, IVQR) is shorter by design — those
tools import the identification machinery from the IV and DiD
notes rather than rebuilding it here.

Read it linearly the first time. After that, treat it as
a reference: jump to the decision tree, the
RIF-vs-quantile-regression section, or the decomposition
framework as needed.

---

## What distributional methods are actually estimating

The move that distinguishes distributional econometrics from
standard regression is taking seriously that the question is
**not** "what happens to the average outcome when X changes"
but rather something about the *distribution* of outcomes:
what happens to the 10th percentile, to the Gini coefficient,
to the share of income held by the top decile, to the poverty
rate. Mean-regression answers a narrow question, and the narrow
answer can be misleading when the treatment effect is
heterogeneous across the outcome distribution — which it almost
always is.

Two distinct targets appear in the literature, and conflating
them is the most common conceptual error in this space.

**Conditional distributional effects** ask: how does the
conditional distribution of $Y$ given $X=x$ change as $x$
changes? Quantile regression lives here. The estimand is the
conditional quantile function $Q_\tau(Y|X=x)$ — the value $q$
such that $P(Y \le q | X=x) = \tau$ — and how it shifts with
$X$. This is a statement about subgroups: among units with
$X=x$, what happens at the $\tau$-th quantile.

**Unconditional (marginal) distributional effects** ask: how
does the marginal distribution of $Y$ — the distribution you
see if you don't condition on anything — change when $X$
shifts for everyone? RIF regressions live here. The estimand
is a functional of $F_Y$ (a quantile, the variance, the Gini,
a top share, a poverty headcount), and how it responds to
a mean shift in $X$ across the whole population. This is a
statement about aggregate distributions: what happens to the
labor income Gini if the minimum wage rises for every worker.

These are different questions. A policy that compresses
conditional wage distributions within education groups can
widen the unconditional wage distribution if it simultaneously
changes the composition of groups. A regression that shows
zero effect at the conditional median can be consistent with
large effects on the marginal median, if the effect is
heterogeneous across unobserved characteristics correlated
with $X$.

A third estimand in the causal literature is the **quantile
treatment effect on the treated** (QTT) in a DiD setting, or
the **local average quantile treatment effect** (LAQTE) in an
IV setting. These are distributional analogs of the ATT and
LATE. They ask: what is the effect of treatment on the
$\tau$-th quantile of the treated group's counterfactual
outcome distribution? These require the full identification
machinery from the DiD and IV notes, adapted to distribute the
effect across quantiles rather than averaging it.

Naming which estimand you are targeting — and why it answers
the research question — is the first step in any distributional
analysis. The rest of the method follows from that.

---

## Quantile regression

### The estimand and the loss function

Quantile regression was introduced by Koenker and Bassett
(1978) and is now the workhorse for conditional distributional
analysis. The $\tau$-th regression quantile solves:

$$\hat\beta(\tau) = \arg\min_\beta \sum_{i=1}^n \rho_\tau\bigl(Y_i - X_i'\beta\bigr),$$

where $\rho_\tau(u) = u(\tau - \mathbf{1}[u < 0])$ is the
**check function** (also called the tilted absolute loss or
pinball loss). At $\tau = 0.5$ this reduces to least absolute
deviations (LAD), which estimates the conditional median. At
any $\tau$, the minimizer $\hat\beta(\tau)$ gives a linear
approximation to the conditional quantile function
$Q_\tau(Y|X)$.

Three properties worth knowing. First, **equivariance to
monotone transformations**: if $h$ is monotone increasing,
$Q_\tau(h(Y)|X) = h(Q_\tau(Y|X))$. This means you can
interpret QR coefficients in log-outcomes as approximately
log-linear percentage effects without the Jensen-inequality
problem that plagues OLS on log outcomes. Second,
**distributional agnosticism**: no assumption on the
conditional distribution of $Y|X$ is needed beyond the
existence of the $\tau$-th conditional quantile. Heteroskedastic
errors are not a problem — they are the point. Third,
**robustness to outliers in $Y$**: the check function has
bounded influence on the conditional median and is more
resistant than OLS to extreme observations, which matters in
top-coded or skewed income data.

### The quantile process and simultaneous inference

Running QR at a single $\tau$ is rarely sufficient. The full
**conditional quantile process** $\{\hat\beta(\tau): \tau \in
[\tau_L, \tau_U]\}$ is the object of interest, showing how
the covariate effects shift across the outcome distribution.
This process is a random function, and inference should respect
its joint behavior.

Point-by-point inference using the sandwich formula at each
$\tau$ produces marginally valid confidence bands but gives no
guarantee of jointly valid coverage across $\tau$. **Uniform
confidence bands** — based on the asymptotic Brownian bridge
process of the quantile estimator (Koenker and Machado 1999,
Chernozhukov and Fernández-Val 2005) — give joint coverage
over a range of quantiles and are the correct tool when the
interest is in the shape of the quantile process. In practice:
run the uniform bands, and if they tighten and widen
systematically, that heterogeneity is the finding.

### What QR is not

Quantile regression does not estimate quantile treatment
effects in the causal sense without additional assumptions.
$\hat\beta(\tau)$ estimates the partial association between $X$
and the $\tau$-th conditional quantile. Without randomization
or a valid instrument, this is a descriptive quantity: the
gap in the conditional $\tau$-th quantile between two groups
that may differ systematically on unobservables. The causal
interpretation requires the same identification as OLS — and
additional assumptions about rank invariance if you want to
recover individual-level effects.

QR also does not estimate unconditional effects. See §3.

### Standard errors

Three options:

**Sandwich (design-matrix-based).** Consistent under general
conditions; default in most software. For clustered data,
clustered sandwich SEs cluster the score at the appropriate
level (cluster at the level of treatment assignment, same as
OLS).

**Bootstrap.** Straightforward to implement, handles complex
designs, recommended when the analytical formula is suspect.
Pairs bootstrap or block bootstrap for clustered/panel data.

**Rank-based / Koenker-Machado.** Based on the sparsity
estimate (the reciprocal of the conditional density at the
quantile). Useful for uniform bands.

---

## Recentered influence functions

### Motivation: from conditional to unconditional

The key limitation of quantile regression is that it answers
a conditional question. When the distribution of $X$ shifts
in the population — as a minimum wage rise shifts the
distribution of wages — the marginal distribution of $Y$
changes through both a direct effect and a composition effect.
QR captures only the direct effect within groups. The policy
question is often about the total effect on the marginal
distribution.

Firpo, Fortin, and Lemieux (2009) — FFL henceforth — solve
this by constructing a summary statistic for each observation
that, when averaged, recovers the distributional functional of
interest, and then regressing that summary statistic on
covariates. This gives the **unconditional partial effect**
of $X$ on the distributional functional.

### The influence function

The **influence function** of a functional $\nu(F)$ measures
its sensitivity to a perturbation of the distribution at a
single observation:

$$IF(y; \nu, F) = \lim_{\epsilon \to 0} \frac{\nu((1-\epsilon)F + \epsilon\delta_y) - \nu(F)}{\epsilon},$$

where $\delta_y$ is a point mass at $y$. This is the standard
concept from robust statistics (Hampel 1974). For the quantile
functional $\nu(F) = Q_\tau(F) = F^{-1}(\tau)$:

$$IF(y; Q_\tau) = \frac{\tau - \mathbf{1}[y \le Q_\tau]}{f_Y(Q_\tau)},$$

where $f_Y$ is the marginal density of $Y$ evaluated at the
$\tau$-th quantile. For the variance, $\nu(F) = \text{Var}(Y)$:

$$IF(y; \text{Var}) = (y - \mu)^2 - \text{Var}(Y).$$

For the Gini coefficient, the influence function exists and is
more complex but implemented in standard software.

### The recentered influence function and RIF regression

The **recentered influence function** adds the functional back:

$$RIF(y; \nu, F) = \nu(F) + IF(y; \nu, F).$$

The key property is $E[RIF(Y; \nu, F)] = \nu(F)$ — the
mean of the RIF is the functional itself. This follows from
$E[IF(Y; \nu, F)] = 0$, a standard property of influence
functions under regularity.

**RIF regression** runs OLS of $RIF(Y_i; \nu, \hat F)$ on
$X_i$:

$$\hat\gamma = \arg\min_\gamma \sum_i \bigl(RIF(Y_i; \nu, \hat F) - X_i'\gamma\bigr)^2.$$

By iterated expectations, $\gamma$ recovers the **unconditional
partial effect (UPE)** of $X$ on $\nu(F_Y)$: the change in the
distributional statistic when the mean of $X$ increases by one
unit in the whole population, holding the conditional
distribution of $Y|X$ constant. This is the right estimand for
policy questions about how a population-level shift in $X$
changes aggregate inequality.

Three practical notes. First, $\hat F$ is the empirical CDF;
for quantiles, $Q_\tau$ is estimated by sample quantile and
$f_Y(Q_\tau)$ is estimated by kernel density, so the RIF is
a generated regressor — inference should account for this
(bootstrap is safest). Second, the OLS on RIF inherits the
usual concerns: if $E[RIF(Y|X)]$ is non-linear in $X$, the
linear projection is misspecified and recovers a
population-weighted average effect, not a structural parameter.
Third, any distributional functional with a well-defined
influence function works — quantiles, Gini, poverty rates,
top shares, interdecile ratios — which makes RIF regression the
most general tool in this toolkit for summarizing distributional
impacts.

### Comparing QR and RIF regression

|  | Quantile regression | RIF regression |
|--|--|--|
| **Estimand** | Conditional $Q_\tau(Y\|X)$ | UPE on $\nu(F_Y)$ |
| **Composition effect** | Not captured | Captured through population shift |
| **Functional** | Quantiles only | Any functional with IF |
| **Distributional assumption** | None | Kernel density estimate for $f_Y$ |
| **Policy question** | Within-group heterogeneity | Population-level distributional change |

Use QR when the question is about heterogeneity *within* well-defined groups and conditioning on $X$ is the object. Use RIF regression when the question is about aggregate distributional statistics and $X$ shifts the population mean.

---

## Distributional decompositions

### The Oaxaca-Blinder baseline

The Oaxaca (1973) and Blinder (1973) decomposition splits
the mean difference in $Y$ between two groups (A and B) into:

$$\bar Y_A - \bar Y_B = \underbrace{(\bar X_A - \bar X_B)'\hat\beta_B}_{\text{composition}} + \underbrace{\bar X_A'(\hat\beta_A - \hat\beta_B)}_{\text{structure}},$$

where the composition component captures how much of the gap
is explained by differences in observable characteristics, and
the structure component captures differences in coefficients
(including the "unexplained" gap at the intercept). The
decomposition is identified at the mean and requires a linear
outcome model. Standard in labor economics and well-implemented
everywhere, but it answers only a mean question.

The OB detailed decomposition attributes each component of the
composition and structure gap to individual covariates — a
useful diagnostic even when the distributional methods below
are the primary analysis.

### DiNardo-Fortin-Lemieux reweighting

DiNardo, Fortin, and Lemieux (1996) — DFL — extend the
decomposition to the full distribution using **density
reweighting**. The idea: to ask what the wage distribution of
group B would look like if it had group A's characteristics,
reweight the group B density by the propensity-score ratio:

$$\psi(X) = \frac{P(G=A|X)}{P(G=B|X)} \cdot \frac{P(G=B)}{P(G=A)},$$

where $G \in \{A, B\}$ is group membership. The reweighted
distribution of $Y_B$ approximates the counterfactual
"group B outcomes, group A characteristics." Comparing the
actual $F_A$, the counterfactual $F_B^\psi$, and $F_B$ at any
quantile decomposes the distributional difference into
composition and structure components, at every point of the
distribution simultaneously.

DFL is semiparametric (logit or probit for the propensity
score, nonparametric otherwise) and makes no assumption on
the functional form of the outcome equation. Its limitation
is that the "detailed decomposition" — attributing the
composition effect to individual covariates — is not
straightforward without additional structure.

### RIF-based decompositions

Firpo, Fortin, and Lemieux (2018) unify the OB and DFL
approaches using RIF regression. The **RIF decomposition**
for distributional functional $\nu$:

$$\nu(F_A) - \nu(F_B) = \underbrace{(\bar X_A - \bar X_B)'\hat\gamma_B}_{\text{composition}} + \underbrace{\bar X_A'(\hat\gamma_A - \hat\gamma_B)}_{\text{structure}},$$

where $\hat\gamma$ is the RIF regression coefficient on the
$\nu$ functional for each group. This has three advantages
over DFL alone. First, it works for any distributional
functional with a well-defined influence function, not just
quantiles. Second, the detailed decomposition — attribution
to individual covariates — follows directly from the OB
algebra applied to the RIF regression, producing a
covariate-by-covariate breakdown at every quantile or other
functional. Third, it nests OB at the mean (IF of the mean
is $y - \mu$, so RIF regression on the mean is OLS).

The limitation is that the composition effect in the RIF
decomposition uses a linear approximation (the RIF is linear
in $X$ locally); for large distributional differences where
the linear approximation is poor, reweighting first (DFL),
then running the RIF decomposition within the reweighted
sample, gives a more reliable decomposition. This
**reweighting-then-RIF** strategy is the current best
practice (Fortin, Lemieux, and Firpo 2011).

### Quantile counterfactual distributions

Chernozhukov, Fernández-Val, and Melly (2013) — CFM — propose
an alternative based on **distribution regression**: model
$P(Y \le y | X)$ directly via logit or probit at each
threshold $y$, then invert to recover the conditional
quantile function. Counterfactual distributions plug in the
covariate distribution from one group into the CDF estimated
from another, and the resulting distribution is inverted to
get counterfactual quantiles at every $\tau$.

CFM handles discrete, censored, and bounded outcomes more
naturally than RIF regression, because the distribution
regression approach is a proper probability model. The
software is more demanding; `Counterfactual` in Stata
and `cfm` in R implement it. For most applications with
continuous outcomes, FFL is simpler and gives similar results.

---

## Causal distributional effects

### Quantile treatment effects in DiD

Callaway and Li (2019) extend the DiD framework to the full
distributional estimand. The **quantile treatment effect on
the treated (QTT)**:

$$QTT(\tau) = Q_\tau(Y(1)|D=1) - Q_\tau(Y(0)|D=1),$$

which is the difference at the $\tau$-th quantile between
the treated potential outcome and the counterfactual untreated
potential outcome for the treated group.

Identification requires a **distributional parallel trends**
assumption: the entire counterfactual distribution
$F_{Y(0)|D=1}$ has the same distributional trend as the
control group, not just the mean. This is stronger than mean
parallel trends — it rules out heterogeneous trends at
different quantiles — and should be tested via pre-treatment
distributional plots (quantile-by-quantile pre-trend checks)
rather than only mean pre-trend tests.

The estimator uses the control group distribution, reweighted
to match the treated group's covariate distribution, as the
counterfactual. The `drdid` and `csdid` packages in R
implement distributional DiD in the Callaway-Sant'Anna
framework; for a focused QTT, `Counterfactual` in Stata or
direct DFL reweighting within a DiD design is common.

### IV quantile regression

Chernozhukov and Hansen (2005, 2008) extend IV to the
quantile setting. The **structural quantile function** is
defined under a **rank invariance** (or rank similarity)
assumption: a unit's rank in the potential outcome
distribution does not change with treatment. Under rank
invariance and a valid instrument, the IVQR estimator inverts
a grid of QR estimations to find $\beta(\tau)$ such that the
residual is uncorrelated with the instrument at each $\tau$.

The rank invariance assumption is strong. It rules out
selection into treatment based on anticipated individual-level
gains from treatment — precisely the cases where LATE differs
most from ATE. When rank invariance fails, IVQR recovers a
local structural quantile function for compliers, not the
population quantile treatment effect.

The `quantreg` package in R has IV quantile regression
support; Stata's `ivqreg2` (Machado-Santos Silva) implements
an alternative approach. For most applied settings, the
Chernozhukov-Hansen grid-search implementation is preferred.

Abadie (2003) provides an alternative: under monotonicity
(the IV assumption for LATE), the complier's distribution of
$Y(0)$ and $Y(1)$ can be recovered, and quantile effects for
compliers can be estimated via propensity-score reweighting.
This is less parametrically demanding than IVQR but requires
a binary treatment and instrument.

---

## Decision tree

Use this to find the right tool for a distributional question.

**Q1. Is the question about the conditional distribution or
the marginal (population) distribution?**

- **Conditional** (how does the $\tau$-th quantile of $Y|X$
  vary with $X$?) → **Quantile regression** (§2). Go to Q2.
- **Marginal** (how does a population-level statistic — Gini,
  top share, poverty rate — change when $X$ shifts?) →
  **RIF regression** (§3). Go to Q3.
- **Decomposition** (what explains the distributional
  difference between two groups?) → **§4**. Go to Q4.
- **Causal effect on the distribution** with a valid
  identification strategy → **§5**. Go to Q5.

**Q2. (Quantile regression.) How many quantiles?**

- Single $\tau$: run QR at that quantile, report the
  coefficient with clustered sandwich SEs.
- Full process: run QR across a grid of $\tau$, report the
  quantile process plot with uniform confidence bands.
- Test whether the effect is heterogeneous: null is
  $\beta(\tau) = \beta$ for all $\tau$; the Koenker-Bassett
  (1982) $F$-type test or a supremum test over the uniform
  bands.

**Q3. (RIF regression.) What is the functional?**

- Quantile: RIF is the standard formula (§3.2). Use kernel
  density for $f_Y(Q_\tau)$; bootstrap SEs.
- Variance, Gini, poverty rate: use the relevant IF formula.
  `rifreg` (Stata) and `rifr` (R) compute these.
- If the marginal outcome distribution is complex (discrete,
  censored, bounded), consider CFM (§4.4) instead.

**Q4. (Decomposition.) How many covariates?**

- Mean gap, few covariates: **Oaxaca-Blinder** with detailed
  decomposition.
- Full distributional gap, propensity score reliable:
  **DFL reweighting** for the composition component, RIF
  decomposition for the detailed breakdown.
- Large distributional differences or non-linear composition
  effects: **reweight first (DFL), then RIF decomposition**
  on the reweighted sample.

**Q5. (Causal distribution.) What is the design?**

- DiD design, staggered or not: **Callaway-Li QTT** with
  distributional parallel trends assumption (§5.1). Check
  distributional pre-trends, not only mean pre-trends.
- IV design with near-binary treatment and monotonicity:
  **Abadie (2003) complier distributions** via reweighting.
- IV with continuous treatment and rank invariance plausible:
  **IVQR** (Chernozhukov-Hansen 2005).
- When in doubt about rank invariance: report mean LATE
  from the standard IV note and flag that distributional
  effects are not identified without additional assumptions.

---

## The load-bearing assumption, honestly

Distributional methods carry the same identification
assumptions as their mean counterparts, plus one additional
layer that is often quietly adopted.

**For quantile regression.** QR requires the same
exogeneity as OLS: $E[\rho_\tau(Y - X'\beta(\tau)) \cdot X]
= 0$, which holds if the conditional quantile function is
correctly specified and $X$ is exogenous. The *additional*
assumption is that the linear specification adequately
approximates the conditional quantile function at the chosen
$\tau$. Non-linearity in the quantile process that is ignored
produces a weighted average effect, the weights depending on
the covariate distribution — the same misspecification problem
as OLS, but potentially more severe at tails where data are
sparse.

**For RIF regression.** The UPE from RIF regression is the
effect of shifting the *mean* of $X$ in the population by one
unit, holding fixed the conditional distribution of $Y|X$.
This is identified under the same conditions as OLS on the
mean. The *additional* assumption is that the linear
approximation $E[RIF(Y;\nu)|X] \approx X'\gamma$ is adequate.
In the tails — where the marginal distribution is thin and
the influence function is large — the approximation can be
poor. The reweighting-then-RIF approach mitigates this by
restricting the analysis to a region of common support.

**For decompositions.** The DFL reweighting requires that
the propensity score is bounded away from zero and one (common
support) and that the outcome model is correctly specified
within each group. Missing common support is the most
dangerous failure: units with no overlap between groups cannot
be compared, and the decomposition extrapolates. Always plot
propensity score distributions; always restrict to the
region of common support.

**For causal distributional effects.** The distributional
parallel trends assumption for QTT is stronger than mean
parallel trends and is *not* tested by the usual pre-trend
event-study plot (which tests the mean). Rank invariance
for IVQR is untestable and rules out a substantively
important class of treatment effect heterogeneity. Name
these assumptions explicitly; do not adopt them silently.

---

## Common failure modes

**Using quantile regression when the question is about the
marginal distribution.** QR gives the conditional quantile
effect; if the policy shifts the covariate distribution for
everyone (a minimum wage, a transfer program), the relevant
object is the marginal effect — which requires RIF. Calling
the QR coefficient "the effect on the income distribution"
when it is a conditional partial effect is a category error.

**Reporting quantile effects without uniform bands.** Point-
by-point 95% confidence intervals at each $\tau$ give 95%
coverage at each quantile but no joint coverage across the
process. Reporting these as if they characterize uncertainty
about the *shape* of the effect profile is misleading.
Uniform bands (Koenker-Machado, Chernozhukov-Fernández-Val)
are the right tool and are available in standard software.

**Ignoring common support in decompositions.** The DFL
reweighting fails without common support: propensity scores
near zero or one produce extreme weights, inflating variance
and biasing estimates. Always check the overlap; always
trim or reweight to achieve it.

**Treating the "unexplained" component of OB as
discrimination.** The structure component of OB
(coefficient differences) captures *all* unmeasured
differences between groups — discrimination, selection,
unobservables, model misspecification. Calling it
discrimination is a strong causal claim that requires an
argument. Altonji and Blank (1999) is the reference for
why this is hard.

**Applying rank invariance silently for IVQR.** The IVQR
result is a structural quantile treatment effect under rank
invariance. If compliers select into treatment precisely
because they gain more than others — the canonical case for
LATE heterogeneity — rank invariance is violated and the
IVQR estimand is not the population quantile treatment effect
for compliers. State the assumption; defend it or bound the
sensitivity.

**Sparse tails and inflated IF weights.** The influence
function for extreme quantiles (say, $\tau = 0.95$) is
large in absolute value for observations near the quantile,
and the kernel density estimate $f_Y(Q_\tau)$ is noisy.
RIF regression at extreme quantiles has poor finite-sample
properties; the sensitivity to bandwidth choice is high.
Be skeptical of RIF results in the far tails of thin
distributions.

**Non-linear composition effects mis-attributed to
structure.** Standard OB treats the composition effect as
linear in covariates. When the true relationship is non-
linear, the "explained" component absorbs nonlinear
interactions and the "unexplained" component is biased.
The DFL reweighting resolves this for the composition
component; the reweighting-then-RIF strategy ensures that
the structure component is identified on the common-support
subsample with comparable covariate distributions.

---

## What to report

When Markus reports a distributional analysis, the
deliverable includes:

1. **The distributional estimand, in plain language.** "The
   effect of the minimum wage on the 10th percentile of the
   marginal wage distribution" is not the same as "the
   conditional quantile effect of the minimum wage within
   workers of given observable characteristics." Name
   which one you are estimating and why.
2. **The quantile process plot** (for QR or QTT) showing
   coefficients across $\tau$ with uniform confidence bands
   — not point-by-point bands. Mark where the effect is
   statistically different from the OLS estimate or from
   zero.
3. **Common support check** (for decompositions). Plot
   propensity score distributions for both groups. State
   the trimming rule used and what share of observations
   it drops.
4. **Decomposition table** with composition and structure
   components, and the detailed covariate-level breakdown
   for the composition component. Report both the
   aggregate decomposition and the detailed breakdown; the
   detailed breakdown tells you which covariates are doing
   the work.
5. **Kernel bandwidth choice** (for RIF at quantiles).
   Show sensitivity to bandwidth. Results that flip sign
   or magnitude with plausible bandwidth variation should
   be flagged.
6. **Identification assumptions stated.** For distributional
   DiD: distributional parallel trends (not just mean).
   For IVQR: rank invariance. For decompositions: common
   support. Each of these earns its own sentence, not a
   footnote.
7. **The OLS/ATT result alongside the distributional
   result.** The mean effect is always a useful
   benchmark. A distributional analysis that ignores the
   mean is harder to situate in existing work. Show both
   and explain why the distributional result adds to the
   mean result.

---

## Implementation, by language

**R** is the strongest for distributional methods overall.

`quantreg` (Koenker) is the reference for quantile
regression: `rq(y ~ x, tau = 0.5, data = df)` for a
single quantile; `rq(y ~ x, tau = seq(0.1, 0.9, 0.1),
data = df)` for the process. `plot(summary(fit, se = "boot"))`
gives coefficient plots with pointwise bands; uniform
bands require `quantreg::KhmaladzeTest` for the process
or `confint(fit, type = "Huber")` for uniform bands
explicitly.

`rifr` (Rios-Avila) computes RIF regression for quantiles,
the Gini, variance, and poverty rates: `rif(y ~ x, rif = "q50",
data = df)` returns the RIF-regressed object, then `tidy()`
for coefficients. Bootstrap SEs via the `se = "boot"` option.
`rifreg` is the Stata equivalent (Rios-Avila, Carreras).

For OB decompositions: `oaxaca` (R, Hlavac) runs the standard
decomposition with group and pooled coefficient options.
`decompose` in `ddecompose` (Rios-Avila) implements the FFL
distributional decomposition via RIF.

For Callaway-Li QTT: `drdid` and `csdid` handle staggered
DiD; the distributional estimand requires running the
estimator on transformed outcomes (the RIF) rather than $Y$
directly. At the time of this writing, the cleanest path is
DFL reweighting to construct the counterfactual, then
computing the quantile difference.

**Stata.**

`qreg y x` for quantile regression; `sqreg y x, q(.1 .5 .9)`
for simultaneous quantile regression (draws from the same
distribution, enabling cross-quantile tests). `bsqreg` for
bootstrap SEs.

`rifreg` (SSC, Rios-Avila and Carreras) for RIF regression:
`rifreg y x, rif(q50)`. `oaxaca` (Jann, SSC) for OB
decompositions; `oaxaca8` handles nonlinear cases.
`dfl` (SSC) for DiNardo-Fortin-Lemieux reweighting.
`Counterfactual` (CFM, Chernozhukov-Fernández-Val-Melly)
for distribution regression and counterfactual distributions.
`ivqreg2` for IV quantile regression.

**Python.**

`statsmodels.regression.quantile_regression.QuantReg` is
the standard: `QuantReg(y, X).fit(q=0.5)`. Less feature-rich
than R's `quantreg` — uniform bands are not built in; bootstrap
them manually.

`linearmodels` has limited distributional support. The
`dml` package within `DoubleML` handles distributional
treatment effects under DML assumptions but is a different
estimand.

For decompositions, there is no mature Python equivalent of
`oaxaca` or `rifreg`. The standard practice is to call the
R packages via `rpy2` or to hand-code the DFL reweighting
(propensity score via logit, reweight, compute distributional
statistics). Hand-coding DFL is not difficult and gives full
control over the support trimming.

**Markus's default** is R (`quantreg` + `rifr` +
`ddecompose`), with Stata for decompositions when replication
requires it. Python for distributional analysis is a
second-class experience relative to R and Stata.

---

## What this note doesn't cover

**Stochastic dominance tests.** Whether $F_A$ dominates $F_B$
at order one, two, or three — used in welfare comparisons.
Davidson and Duclos (2000), Linton-Maasoumi-Whang (2005).
`dominance` package in R.

**Distributional regression in high-dimensional settings.**
DML applied to distributional parameters (Chernozhukov,
Newey, Singh 2022). `DoubleML` in Python and R.

**Copulas for joint distributions.** When the question
involves the joint distribution of two or more outcomes
(e.g., the joint distribution of wages and hours), copula
methods are needed. Covered separately if the need arises.

**Bayesian distributional methods.** Bayesian quantile
regression (Yu and Moyeed 2001, via the asymmetric Laplace
likelihood) and Bayesian nonparametrics for distributional
estimation. Available in `Stan` and `brms` for those who
want posterior uncertainty over the quantile process.

**AKM and earnings decompositions.** The Abowd-Kramarz-
Margolis (1999) decomposition of earnings into worker and
firm effects, and the leave-out corrected estimators
(Kline-Saggio-Sølvsten 2020). This is a special-purpose
distributional decomposition for linked employer-employee
data; it goes in [[`structural-labor.md`]].

---

## Cross-references

- [[`10_Methods/Econometrics/difference-in-differences.md`]] —
  distributional DiD (Callaway-Li QTT in §5.1) imports the
  ATT estimand framing, the staggered design machinery, and
  the distributional parallel trends assumption as the
  distributional analog of parallel trends.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] —
  IVQR (§5.2) imports IV identification; the rank invariance
  assumption here is the distributional analog of monotonicity
  (LATE) in the IV note; the Abadie (2003) complier
  distribution recovery uses the propensity-score reweighting
  approach from the IV note.
- [[`10_Methods/Econometrics/panel-methods.md`]] — fixed-effects
  quantile regression for panels (Canay 2011) is the panel
  analog of QR; distributional decompositions in panel
  settings require accounting for the panel structure in the
  propensity score model.
- [[`10_Methods/Econometrics/regression-discontinuity.md`]] —
  distributional RD (sharp RD at each quantile of the outcome
  distribution) uses the same local polynomial and bandwidth
  machinery as mean RD; implement via `rdrobust` with the
  transformed outcome $\mathbf{1}[Y \le y]$ for a grid of $y$.
- [[`20_Math/probability.md`]] — the delta method (§5) underlies
  the sandwich inference for QR and the asymptotic theory for
  RIF estimators; the quantile process asymptotics draw on the
  functional CLT (Donsker's theorem, §8).
- [[`20_Math/functional-analysis.md`]] — the influence function
  is a Fréchet derivative in the space of distributions (§7);
  the $L^2$ projection theorem underlies the conditional mean
  framing that OB decomposes.
- [[`30_Data/AI-accessible-data-sources.md`]] — distributional
  analysis requires micro data; WID.world for top shares;
  LIS for comparative micro; TÜİK HBIM and SGK for Turkey.
- [[`50_Workflows/run-a-regression-properly.md`]] — the
  regression workflow governs the estimation step; for
  distributional analysis, Stage 4 (identification) and
  Stage 7 (robustness) should include the distributional-
  specific checks named above (support overlap, uniform
  bands, bandwidth sensitivity).

---

*Last updated: 2026-04-12. Covers quantile regression
through Koenker-Bassett (1978), RIF regression through
Firpo-Fortin-Lemieux (2009, 2018), distributional
decompositions through DiNardo-Fortin-Lemieux (1996) and
Chernozhukov-Fernández-Val-Melly (2013), and causal
distributional effects through Callaway-Li (2019) and
Chernozhukov-Hansen (2005, 2008). AKM and earnings
decompositions deferred to [[`structural-labor.md`]]. High-
dimensional distributional regression deferred to
[[`double-machine-learning.md`]].*
