---
tags: [econometrics, panel-data, fixed-effects, dynamic-panels]
related: [modern-toolkit-references, difference-in-differences, local-projections, instrumental-variables, run-a-regression-properly, Principles, linear-algebra, dynamic-programming]
---

# Panel Methods

This is the working note on panel data. It assumes you know
what a fixed-effects regression is and can run one. What it
tries to do is give Markus enough fluency in the panel
econometrics literature to *reason* about what a panel
regression is actually identifying, when fixed effects are
right and when they are not, and how to handle the problems
that arise when the model is dynamic — particularly the
Nickell bias that the local projections note
([[`local-projections.md`]]) forwards here. The entry in
[[`modern-toolkit-references.md`]] is a directory; this is the
actual map.

Read it linearly the first time. After that, treat it as a
reference: jump to the decision tree, the Nickell bias
section, or the cluster inference section as needed.

---

## What panel regressions are actually estimating

The thing to hold before any panel estimator is the
**estimand** — and in panel data the estimand question is
sharper than it first appears, because the same coefficient
$\beta$ can be identified from two fundamentally different
sources of variation.

Consider the panel model:
$$Y_{it} = \alpha_i + X_{it}'\beta + \varepsilon_{it},
\quad i = 1,\ldots,N,\quad t = 1,\ldots,T.$$

The parameter $\beta$ can be recovered from:

- **Within-unit variation**: how $Y_{it}$ and $X_{it}$ move
  together *within* unit $i$ over time, after netting out
  the unit-specific level $\alpha_i$. This is the
  **within estimator** (FE, fixed effects), and it identifies
  $\beta$ from deviations from individual means.
- **Between-unit variation**: how $\bar Y_i$ and $\bar X_i$
  differ *across* units. This is the **between estimator**,
  and it is essentially a cross-sectional regression on
  unit averages.
- **A GLS combination of both**: the **random effects (RE)
  estimator**, which is efficient when the within and between
  estimates are estimating the same $\beta$.

These are not different estimators of the same quantity. They
are estimators of *potentially different* quantities. The
within estimator recovers $\partial E[Y_{it}|X_{it}, \alpha_i]
/ \partial X_{it}$ — the effect of a within-unit change in
$X$, holding the unit's baseline fixed. The between estimator
recovers a cross-unit comparison that is contaminated by
anything that differs systematically across units and correlates
with $X$ — selection, sorting, omitted unit-level confounders.
Unless the within and between effects are the same, and unless
you have a theoretical reason to believe they are, the between
estimator is an observational comparison with all the problems
that entails.

**The first question in any panel analysis is not "FE or RE."
It is "what variation is my question about?"** If the question
is about within-unit dynamics — how a policy change affects a
firm's outcomes over time — FE is the natural target. If the
question is about cross-unit differences — why some countries
grow faster than others — between variation is what you need,
and FE literally throws it away. Choosing FE when the question
is cross-sectional is not conservative; it is misguided.

---

## The within estimator: geometry and implementation

### Demeaning as projection

The within estimator demeans all variables by their unit mean:
$$\tilde Y_{it} = Y_{it} - \bar Y_i, \qquad
\tilde X_{it} = X_{it} - \bar X_i,$$
and runs OLS of $\tilde Y$ on $\tilde X$. The Frisch-Waugh-
Lovell theorem ([[`linear-algebra.md`]] §2.3) guarantees this
gives the same coefficient as including unit dummies
explicitly: the within estimator is OLS after projecting out
the unit-fixed-effect subspace.

The projection matrix that implements demeaning is
$M_D = I - D(D'D)^{-1}D'$, where $D$ is the $NT \times N$
matrix of unit indicator columns. This is the
$M_{X_1}$ of FWL, with $X_1 = D$. The coefficient on $X_2$
(the regressors of interest) from the full regression with
unit dummies equals the coefficient from regressing
$M_D Y$ on $M_D X_2$ — residualizing both on the unit
dummies removes between-unit variation.

**High-dimensional fixed effects.** When both unit and time
fixed effects are included — the two-way FE model
$Y_{it} = \alpha_i + \lambda_t + X_{it}'\beta + \varepsilon_{it}$
— demeaning requires an iterative alternating-projections
algorithm (the "within within within" algorithm). The
`fixest` package in R and `pyfixest` in Python implement
this efficiently via the HDFE algorithm; direct inclusion of
dummies is computationally prohibitive for large $N$. For
three-way or higher-order FE (industry × region × year),
the same alternating-projections logic extends, and `fixest`
handles it.

### The Mundlak representation

Mundlak (1978) offers an equivalent formulation of the FE
estimator that is useful for testing. Write the unit effect
as a projection onto the unit's time-average of $X$:
$$\alpha_i = \bar X_i'\gamma + u_i, \qquad E[u_i|\bar X_i] = 0.$$
Substituting into the panel model:
$$Y_{it} = \bar X_i'\gamma + X_{it}'\beta + u_i + \varepsilon_{it}.$$
The Mundlak regression — including the unit-mean $\bar X_i$
as an additional regressor alongside $X_{it}$ — recovers the
within estimator $\hat\beta_{FE}$ as the coefficient on
$X_{it}$, and $\hat\gamma$ measures the between-within
difference. The Mundlak representation makes a key point
explicit: FE is equivalent to RE with a specific auxiliary
regression included, and $\hat\gamma \neq 0$ is evidence
of unit-effect endogeneity (i.e., the RE assumption fails).

---

## Random effects and the Hausman test

### The RE assumption and when it fails

The random effects estimator is GLS of $Y_{it}$ on $X_{it}$,
treating $\alpha_i$ as a mean-zero random variable uncorrelated
with $X_{it}$:
$$E[\alpha_i | X_{i1}, \ldots, X_{iT}] = 0.$$
Under this assumption, RE is consistent and efficient (it
uses both within and between variation). Under violation,
RE is inconsistent — the cross-unit correlation between
$\alpha_i$ and $X_{it}$ biases the coefficient.

In practice, RE is almost never defensible for observational
data. The unit effect $\alpha_i$ captures everything
stable about unit $i$ — ability, geography, management
quality, historical institutions — and virtually any
regressor of interest correlates with some stable unit
characteristic. Wages correlate with ability. Output
correlates with geography. Policy adoption correlates with
pre-existing institutional strength. The RE assumption says
none of this is true; it is almost always wrong.

RE is defensible in two settings: (i) designed experiments
where units are randomly assigned, so $\alpha_i \perp X_{it}$
by construction; and (ii) multi-level models where the unit
effect represents genuine sampling variation from a
population of units (schools, hospitals) and the goal is
prediction or partial pooling rather than causal inference.
For causal questions with observational data, default to FE.

### The Hausman test: what it tests and why it is overrated

The Hausman (1978) test compares $\hat\beta_{FE}$ and
$\hat\beta_{RE}$. Under $H_0: E[\alpha_i|X_{it}] = 0$, both
are consistent and the RE estimator is efficient, so the
difference $\hat\beta_{FE} - \hat\beta_{RE}$ should be small.
Under $H_1$, only FE is consistent, so the difference is
large. The test statistic $(\hat\beta_{FE} - \hat\beta_{RE})'
[\mathrm{Var}(\hat\beta_{FE}) - \mathrm{Var}(\hat\beta_{RE})]^{-1}
(\hat\beta_{FE} - \hat\beta_{RE}) \xrightarrow{d} \chi^2_k$
under $H_0$.

**Why it is overrated:**

First, it tests a joint null — that the RE assumption holds
*and* that the model is correctly specified. A rejection can
come from RE assumption failure, heteroskedasticity,
autocorrelation, or functional form misspecification.
Acceptance does not imply RE is appropriate; it could just
mean the test has low power in your sample.

Second, the conventional Hausman statistic requires
homoskedastic errors for its standard-error formula. The
"robust Hausman" (Wooldridge 2010, via the Mundlak
regression) is the correct version: add $\bar X_i$ to the
RE specification and test $H_0: \gamma = 0$ with cluster-
robust standard errors. This is equivalent but more
transparent and more honest about inference.

Third, in large $N$ samples the test almost always rejects
because it has power against even tiny violations of the
RE assumption that have negligible practical consequence for
$\hat\beta_{FE}$. A rejection is not a license to stop
thinking about the source of bias; it is a prompt to examine
it.

**The right approach:** use the Mundlak regression to
characterize the between-within difference $\hat\gamma$ and
ask whether it is substantively significant and theoretically
interpretable, not just whether a test rejects. If
$\hat\gamma$ is large and has a plausible interpretation
(workers with higher ability sort into higher-paying jobs,
so ability contaminates the between estimate of returns to
education), the FE design is justified on those grounds,
not on a test statistic.

### Correlated random effects (Chamberlain 1982)

Chamberlain's CRE model generalizes Mundlak by projecting
$\alpha_i$ on the full sequence of $X_{it}$, not just the
mean:
$$\alpha_i = X_{i1}'\gamma_1 + \cdots + X_{iT}'\gamma_T
+ u_i.$$
This nests the Mundlak specification. In a balanced panel,
Chamberlain's model is equivalent to the FE estimator when
the $\gamma_t$ are unrestricted, but it allows partial
restrictions on how the unit effect correlates with $X$ at
different time periods. CRE also generalizes to nonlinear
models (logit, probit, Tobit) where fixed effects are
inconsistent due to the incidental parameters problem.
Wooldridge (2019) is the standard reference for CRE in
nonlinear panels.

---

## Nickell bias in dynamic panels

### The source of the bias

Add a lagged dependent variable to the panel model:
$$Y_{it} = \rho Y_{i,t-1} + X_{it}'\beta + \alpha_i
+ \varepsilon_{it}, \qquad |\rho| < 1.$$
The FE estimator demeans this regression. The demeaned
lagged dependent variable is
$\tilde Y_{i,t-1} = Y_{i,t-1} - \bar Y_i^{(-1)}$, where
$\bar Y_i^{(-1)} = (T-1)^{-1}\sum_{s=1}^{T-1} Y_{is}$ is
the mean of the lagged series. The fundamental problem:
$\bar Y_i^{(-1)}$ is a function of all $Y_{i,1}, \ldots,
Y_{i,T-1}$, which includes $\varepsilon_{i,1}, \ldots,
\varepsilon_{i,T-1}$. Meanwhile, the demeaned error is
$\tilde\varepsilon_{it} = \varepsilon_{it} - \bar\varepsilon_i$,
which also contains $\varepsilon_{i,1}, \ldots, \varepsilon_{i,T-1}$.
So $\tilde Y_{i,t-1}$ and $\tilde\varepsilon_{it}$ are
correlated even if the original $\varepsilon_{it}$ is
i.i.d. — the within transformation induces endogeneity where
there was none.

Nickell (1981) derived the leading-order bias in $\hat\rho_{FE}$:
$$\mathrm{plim}(\hat\rho_{FE}) - \rho \approx
-\frac{1 + \rho}{T - 1}\bigl(1 + O(T^{-1})\bigr).$$
For $\rho = 0.8$ and $T = 5$ (a short panel), the bias is
approximately $-1.8/4 \approx -0.45$ — the FE estimate of
persistence is roughly halved. For $T = 40$ (a long cross-
country panel), the bias is roughly $-0.045$, which may
be tolerable depending on the application. **The Nickell
bias shrinks with $T$ but never disappears in fixed-$T$
asymptotics.**

The bias propagates to $\hat\beta$ through the correlation
between $Y_{i,t-1}$ and $X_{it}$ — if the controls and the
lagged outcome are correlated, the bias contaminates the
whole coefficient vector, not just $\hat\rho$.

### First differencing

First-differencing eliminates $\alpha_i$:
$$\Delta Y_{it} = \rho \Delta Y_{i,t-1} + \Delta X_{it}'\beta
+ \Delta\varepsilon_{it}.$$
But $\Delta Y_{i,t-1} = Y_{i,t-1} - Y_{i,t-2}$ contains
$Y_{i,t-1}$, which contains $\varepsilon_{i,t-1}$; and
$\Delta\varepsilon_{it} = \varepsilon_{it} - \varepsilon_{i,t-1}$
also contains $\varepsilon_{i,t-1}$. So $\Delta Y_{i,t-1}$
and $\Delta\varepsilon_{it}$ are still correlated. First-
differencing removes the fixed effect but does not solve the
endogeneity of the lagged dependent variable.

What it does do: under the assumption that $\varepsilon_{it}$
is serially uncorrelated, $Y_{i,t-2}, Y_{i,t-3}, \ldots$
are valid instruments for $\Delta Y_{i,t-1}$ — they are
correlated with $\Delta Y_{i,t-1}$ (through $Y_{i,t-1}$)
but uncorrelated with $\Delta\varepsilon_{it}$ (because
$\varepsilon_{it}$ is i.i.d.). This is the Arellano-Bond
insight.

### Arellano-Bond (1991)

**The estimator.** Instrument $\Delta Y_{i,t-1}$ with
$\{Y_{i,1}, \ldots, Y_{i,t-2}\}$ — all lags of $Y$ dated
$t-2$ and earlier are valid instruments for the period-$t$
first-differenced equation, under the assumption that
$\varepsilon_{it}$ is serially uncorrelated.

The instrument set grows with $t$: for $t=3$, use
$Y_{i1}$; for $t=4$, use $Y_{i1}, Y_{i2}$; for $t=T$,
use $Y_{i1}, \ldots, Y_{i,T-2}$. The total instrument count
is $T(T-1)/2$ for the lagged-$Y$ instruments alone, which
grows quadratically in $T$. Arellano-Bond uses these in a
GMM framework — the moment conditions are
$E[Y_{i,t-s} \Delta\varepsilon_{it}] = 0$ for $s \geq 2$.

The two-step AB estimator uses the optimal GMM weight
matrix, which depends on $\Delta\varepsilon_{it}$, estimated
from the first-step residuals. Two-step is asymptotically
more efficient but the standard errors require finite-sample
correction (Windmeijer 2005) — without the correction they
are severely downward biased.

**Validity conditions.** The AB instruments require: (i)
no serial correlation in $\varepsilon_{it}$ (test with the
AR(1) and AR(2) tests on first-differenced residuals — AR(1)
will always be present and is expected; AR(2) should not
be); (ii) instrument validity (Sargan/Hansen J-test, but
see below).

### Blundell-Bond (1998) and instrument proliferation

**Blundell-Bond** augments AB with a "levels" equation:
use $\Delta Y_{i,t-1}$ as an instrument for $Y_{i,t-1}$ in
the levels regression, under the additional assumption that
the initial conditions $Y_{i1}$ are uncorrelated with the
unit effect $\alpha_i$ (the "mean stationarity" assumption).
System GMM stacks the differenced and levels equations and
uses both sets of instruments simultaneously.

BB is particularly useful when $\rho$ is close to 1 —
lagged levels are weak instruments for first differences
when the series is highly persistent — and in applications
with large $N$, small $T$ where the AB instrument set is
limited.

**The fragility problem.** Both AB and BB are vulnerable to
instrument proliferation. With many instruments:

- The Sargan/Hansen J-test loses power — it almost never
  rejects even invalid instruments when the instrument
  count approaches $N$. Roodman (2009) documents this and
  recommends collapsing the instrument matrix (using one
  instrument per lag rather than one per lag per period) to
  limit the count. As a rule of thumb: instrument count
  should be less than $N$, ideally substantially less.
- The two-step efficient weight matrix is estimated from
  the first-step residuals. With many instruments, this
  matrix is large and potentially near-singular, making
  the two-step estimator unstable. This is the GMM many-
  instruments bias ([[`instrumental-variables.md`]] §4.2).
- The mean stationarity assumption in BB is untestable and
  often implausible in growth regressions (where initial
  conditions are precisely what drives long-run levels).

**Practical discipline for AB/BB:**

1. Limit the lag depth used as instruments (e.g., lags 2-4
   only, not all available lags).
2. Collapse the instrument matrix.
3. Report both the instrument count and the number of groups
   — if instruments ≥ groups, interpret with extreme caution.
4. Use the Windmeijer-corrected two-step standard errors.
5. Take the Hansen J-test seriously when instruments are
   few; be skeptical of non-rejection when instruments
   are many.
6. Report the AR(2) test on differenced residuals. A
   significant AR(2) means the identification assumption
   (no serial correlation in $\varepsilon_{it}$) fails.

### Alternatives to AB/BB

When $T$ is large enough (say $T \geq 20$), the Nickell bias
is small relative to sampling variance and FE may be
acceptable with a bias correction. The **Hahn-Kuersteiner
(2002)** and **Dhaene-Jochmans (2015)** analytical bias
corrections remove the leading term of the Nickell bias
without the IV step. For non-linear dynamic models and for
models where the mean stationarity assumption is implausible,
these corrections are often more reliable than system GMM.

For very long panels with large $N$, **interactive fixed
effects** (Bai 2009) models the error as $\alpha_i' f_t +
\varepsilon_{it}$ — the unit effect is a linear function of
unobserved common factors $f_t$ that vary over time. This
handles cross-sectional dependence and allows unit effects
to have time-varying loadings, at the cost of a more complex
estimation (iterative OLS-PCA).

---

## Cluster-robust inference in panels

### The baseline problem

Panel error terms are serially correlated within units —
this is the whole motivation for fixed effects in the first
place. OLS standard errors that ignore this are too small.
The panel version of the Bertrand-Duflo-Mullainathan (2004)
warning: cluster at the unit level to account for arbitrary
serial correlation.

The **cluster-robust variance estimator** (CRVE) is:
$$\hat V_{CR} = (X'X)^{-1}\!\left(\sum_{i=1}^N X_i'
\hat\varepsilon_i \hat\varepsilon_i' X_i\right)(X'X)^{-1},$$
where $X_i$ and $\hat\varepsilon_i$ are the data and
residuals for unit $i$. This allows arbitrary
heteroskedasticity and serial correlation within units,
with no parametric structure assumed.

**Cluster level.** Cluster at the unit level when the unit
is the source of serial dependence (standard panels). If
treatment is assigned at a higher level (state, industry),
cluster at that level — within-state correlation in
treatment timing makes state-level clustering necessary even
when units are individuals.

**Two-way clustering.** When there is both unit-level and
time-level dependence — firms correlated within years because
of common macro shocks — cluster on both dimensions
simultaneously (Cameron-Gelbach-Miller 2011 multi-way
clustering). The two-way CRVE adds the unit-clustered and
time-clustered variance matrices and subtracts the unit×time
clustered one.

### The few-clusters problem

The CRVE has favorable large-$G$ properties (where $G$ is
the number of clusters). With small $G$, the variance matrix
is estimated from few "meat" matrices and the small-sample
t-distribution approximation is poor — standard errors are
biased downward and tests over-reject.

Rules of thumb: with $G < 30$, exercise caution; with
$G < 20$, corrections are necessary; with $G < 10$, the CRVE
should not be used without corrections.

**Wild cluster bootstrap (Cameron-Gelbach-Miller 2008).**
Re-weight entire cluster residuals by $\pm 1$ (Rademacher
weights) or by draws from a distribution with mean 0 and
variance 1 (Webb weights for $G < 10$). The bootstrap
distribution of the test statistic approximates the null
distribution without relying on normal asymptotics. Implemented
in `fwildclusterboot` (R) and `boottest` (Stata); `wildboottest`
in Python. Use it whenever $G < 50$ as a check; as the
primary method when $G < 30$.

**Cluster-adjusted critical values.** The CR-$t$ distribution
(Imbens-Kolesár 2016) uses a degrees-of-freedom correction
that accounts for the effective number of informative clusters.
This is the correct approach when cluster sizes are unequal
— the effective number of clusters is lower than the count
when a few large clusters dominate.

### Cross-sectional dependence

In panels of countries, regions, or industries, units are
correlated because of common shocks (global business cycles,
sector-wide demand shocks). Clustering on units does not
handle this — it assumes units are independent in the
between-cluster dimension.

**Driscoll-Kraay (1998)** standard errors are HAC-consistent
for panels with cross-sectional dependence, assuming the
dependence decays with "economic distance." They account for
arbitrary heteroskedasticity, serial correlation, and cross-
sectional correlation up to a bandwidth in the time dimension.
Implemented in `sandwich::vcovSCC` (R) and `xtscc` (Stata).

**Pesaran's CD test** (2004) tests for cross-sectional
dependence in panel residuals; a significant CD statistic
means Driscoll-Kraay or stronger corrections are needed.

**Common correlated effects (CCE, Pesaran 2006)** goes
further: augment the regression with cross-sectional averages
of $Y$ and $X$ as additional regressors. This controls for
unobserved common factors that create dependence, rather than
just correcting the standard errors after the fact. The CCE
estimator is consistent even when the number of common
factors is unknown. For panels of countries with common
global shocks, CCE is the right tool.

---

## Decision tree

**Q1. Is the question about within-unit or cross-unit
variation?**
→ Within: fixed effects is the natural target; proceed to Q2.
→ Cross-unit: pooled OLS or between estimator; be explicit
that this is a cross-sectional comparison.

**Q2. Does the model include a lagged dependent variable?**
→ No: go to Q4.
→ Yes: the model is dynamic; go to Q3.

**Q3. How large is T?**
→ $T \geq 30$: Nickell bias is approximately
$-(1+\rho)/(T-1) \approx 3\text{--}6\%$ of $\rho$ —
usually tolerable. Use FE with analytical bias correction
(Hahn-Kuersteiner). Go to Q4 for inference.
→ $T < 30$: Nickell bias is material. Use Arellano-Bond
(if $N$ large, $T$ small) or Blundell-Bond (if $\rho$ near
1). Limit instrument count. Windmeijer-correct two-step SEs.
Test AR(2) in first differences. Report Hansen J carefully.

**Q4. Is the random effects assumption defensible?**
→ Designed experiment with random assignment: RE may be
efficient; run the Mundlak regression to check.
→ Observational data: default to FE. Use Mundlak to
characterize the between-within difference.
→ Nonlinear model (logit, probit): use CRE (Chamberlain-
Mundlak); note FE is inconsistent for short $T$ due to
incidental parameters.

**Q5. How many clusters?**
→ $G \geq 50$: standard CRVE, cluster at unit level (or
treatment assignment level if higher).
→ $20 \leq G < 50$: CRVE plus wild cluster bootstrap as
a robustness check.
→ $G < 20$: wild cluster bootstrap (Webb weights); report
both CRVE and bootstrap; note the small-$G$ caveat explicitly.
→ $G < 10$: CRVE is unreliable; permutation inference or
randomization inference.

**Q6. Is there cross-sectional dependence?**
→ Run Pesaran's CD test. If significant: Driscoll-Kraay SEs
(mechanical fix) or CCE (structural fix). Report which you
used and why.

**When in doubt:** fixed effects, cluster at the unit level,
wild bootstrap if $G < 30$. Dynamic model? Arellano-Bond
with collapsed instruments and Windmeijer correction. Report
instrument count and Hansen J.

---

## The load-bearing assumption, honestly

The FE estimator rests on **strict exogeneity**:
$$E[\varepsilon_{it} | X_{i1}, \ldots, X_{iT}, \alpha_i] = 0
\quad \text{for all } t.$$

This is considerably stronger than contemporaneous exogeneity
($E[\varepsilon_{it}|X_{it}, \alpha_i] = 0$). Strict
exogeneity rules out:

- **Feedback from past outcomes to current regressors.**
  If $Y_{i,t-1}$ affects $X_{it}$ (a firm's past performance
  affects this period's investment; a country's past inflation
  affects this period's monetary policy), then $\varepsilon_{i,t-1}
  \in Y_{i,t-1}$ and $X_{it}$ are correlated, violating
  strict exogeneity. This is the Nickell bias setup.

- **Anticipation.** If units adjust $X_{it}$ in anticipation
  of a future shock that affects $Y_{it+s}$, then $X_{it}$
  correlates with $\varepsilon_{i,t+s}$, violating strict
  exogeneity in the forward direction.

- **Bad controls.** If a control variable $X_{it}$ is itself
  a consequence of the treatment being studied (a "post-
  treatment variable"), conditioning on it reopens back-door
  paths and creates bias.

Strict exogeneity can be partially tested. Add one-period
leads $X_{i,t+1}$ to the regression: if FE is consistent
under strict exogeneity, the coefficient on the lead should
be zero (Wooldridge 2010, p. 286). A significant lead
coefficient is evidence of either strict exogeneity failure
(feedback from $Y$ to future $X$) or anticipation effects.

The assumption is not testable in full — it involves all
lags and leads — but the lead test is a useful diagnostic.
The honest approach: state the strict exogeneity assumption
explicitly, report the lead test, and discuss which feedback
channels are plausible given the institutional context.

---

## Common failure modes

**Running TWFE on a dynamic model without checking.** Adding
a lagged dependent variable to a two-way FE regression
without addressing the Nickell bias. The bias is large for
short panels and affects every coefficient, not just the
autoregressive parameter. The fix is Arellano-Bond or a
bias correction; ignoring it is not defensible.

**Instrument proliferation in system GMM.** Using all
available lags as instruments in BB, reporting a non-
significant Hansen J as validation, and presenting the
result as causal. With instrument count near $N$, the
J-test has essentially no power. Report the instrument count.
Collapse the instrument matrix. Sensitivity-check by varying
the lag depth.

**Clustering at the wrong level.** Clustering on units when
treatment is assigned at the state or industry level. The
within-state correlation in treatment is the dominant source
of dependence; unit-level clustering misses it and produces
standard errors that are too small. The right level is the
coarsest level at which treatment status varies.

**Using the Hausman test as a license for RE.** Failing to
reject the Hausman null and then proceeding with RE as if the
assumption were validated. The test has low power in small
samples and high power against trivial violations in large
samples. Run the Mundlak regression and examine $\hat\gamma$
on substantive grounds.

**Ignoring cross-sectional dependence in macro panels.**
Running a panel LP or panel regression on countries without
testing for and addressing cross-sectional dependence. Common
global factors contaminate standard errors and can bias
coefficients when they interact with dynamic specifications.
Always run Pesaran's CD test; if it rejects, apply CCE or
Driscoll-Kraay.

**Treating $R^2$ from within regression as a model fit
measure.** The within $R^2$ describes variation explained
in demeaned $Y$, which is only the within-unit variation.
A high within $R^2$ does not mean the model fits the data
well overall. Report both within and overall $R^2$ and be
explicit about which you are interpreting.

---

## What to report

1. **The estimand** — within-unit or cross-unit, what the
   fixed effects are absorbing, and what variation is left.

2. **Specification** — which fixed effects are included
   (unit, time, unit × time), and what is partialled out.
   In dynamic models, state whether the lag structure is
   AR(1) or higher-order.

3. **The dynamic model treatment** — if lagged $Y$ is
   included: which estimator (FE with bias correction,
   Arellano-Bond, Blundell-Bond), the instrument count vs.
   $N$, the AR(1) and AR(2) test statistics, and the Hansen
   J p-value.

4. **Inference method** — CRVE clustering level, whether
   wild cluster bootstrap was used, and the number of
   clusters.

5. **Cross-sectional dependence diagnostics** — Pesaran CD
   test statistic and whether Driscoll-Kraay or CCE was
   applied.

6. **The Mundlak/lead test** — as a diagnostic of the strict
   exogeneity assumption. Even if FE is the maintained
   specification, reporting this signals awareness of the
   assumption.

7. **Robustness** — vary the lag structure in dynamic
   models; vary the set of time controls; report the
   between estimator alongside FE to show the between-within
   gap and make the identifying variation explicit.

---

## Implementation

**R (default).**

`fixest` is the standard for static FE in R. `feols(y ~ x |
unit + year, data = df, cluster = ~unit)` runs two-way FE
with unit-clustered standard errors. `feols(..., vcov =
"twoway")` for two-way clustering. Handles high-dimensional
FE efficiently via the HDFE algorithm. The `etable()` function
produces publication-quality output.

`fwildclusterboot` wraps `fixest` for wild cluster bootstrap:
`boottest(feols_model, clustid = "unit", param = "x", B = 9999)`.
This is the go-to for small-$G$ inference.

For dynamic panels, `plm` provides Arellano-Bond via
`pgmm()`: `pgmm(y ~ lag(y,1) + x | lag(y,2:4), data = df,
effect = "twoways", model = "twosteps", transformation = "d")`.
The `robust = TRUE` option applies the Windmeijer correction.
`pgmm` is adequate but slower and less flexible than the Stata
alternatives for complex instrument structures.

`panelr` handles within-between models and Mundlak
specifications cleanly. `sandwich::vcovDC()` gives
Driscoll-Kraay standard errors; `pcse::pcse()` gives
panel-corrected SEs.

**Stata.**

`reghdfe` for static HDFE. `xtdpd` and `xtabond2` for
dynamic panels — `xtabond2` (Roodman 2009) is the gold
standard for GMM panels, with explicit options for collapsing
instruments (`collapse`), lag depth, and Windmeijer
correction. `xtscc` for Driscoll-Kraay. `boottest` for wild
cluster bootstrap.

**Python.**

`pyfixest` mirrors `fixest` syntax: `feols("y ~ x | unit +
year", data = df, vcov = {"CRV1": "unit"})`. Wild bootstrap
is supported via `wildboottest` integration. For dynamic
panels, the R/Stata tooling is more mature; Python users
typically call the R `pgmm` or `xtabond2` equivalents through
`rpy2` or run the estimation in R and import results.

---

## What this note doesn't cover

**Synthetic control.** When there is a single treated unit
or a small treated group, synthetic control methods
(Abadie-Diamond-Hainmueller, synth-DiD) replace the panel
FE estimator. These belong in [[`synthetic-control.md`]] (Tier
4 if needed).

**Interactive fixed effects and factor models.** The Bai
(2009) framework with latent common factors and unit-specific
loadings. Relevant for large macro panels where the factor
structure is the object of interest, not just a nuisance.

**Nonlinear dynamic panels.** Logit and probit with lagged
dependent variables have a double incidental parameters
problem (unit effects and the initial conditions problem).
The CRE approach (Wooldridge 2010) handles this via the
conditional distribution of $Y_{i0}$ given $X_{i1}, \ldots,
X_{iT}$. Not covered here.

**Quantile regression for panels.** Fixed-effects quantile
regression (Canay 2011, Rosen-Powell 2012) with within
estimators at quantiles other than the mean. Connects to
[[`distributional-methods.md`]] (Tier 3).

---

## Cross-references

- [[`10_Methods/Econometrics/difference-in-differences.md]]` —
  the two-way FE estimator (DiD note §2 and passim) is a
  panel FE estimator; the Nickell bias is not a concern in
  DiD because the lagged dependent variable is typically not
  included, but the cluster inference section of DiD note
  maps directly to §6 here.
- [[`10_Methods/Econometrics/local-projections.md`]] — the panel
  LP section defers the Nickell bias discussion here; the
  cross-sectional dependence and heterogeneous-response
  issues in panel LPs (Pesaran-Smith mean-group, Pesaran CCE)
  are the same issues in §6 here.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] —
  Arellano-Bond uses GMM moment conditions; the instrument
  proliferation problem in §4 here is the same as in
  the IV note's many-instruments discussion; Windmeijer's
  correction for two-step GMM SEs belongs to the same
  family as the IV note's weak-instrument literature.
- [[`20_Math/linear-algebra.md`]] — the within transformation
  is an orthogonal projection (§2.1 there); the FWL theorem
  (§2.3 there) is the algebraic foundation of partialling
  out fixed effects; the HDFE algorithm is an alternating-
  projections algorithm.
- [[`20_Math/dynamic-programming.md`]] — dynamic panel models
  are reduced-form analogs of the dynamic programming
  models in that file; the persistence parameter $\rho$ in
  a dynamic panel is the analog of the discount factor in
  the Bellman equation.
- [[`50_Workflows/run-a-regression-properly.md`]] — Stage 4
  (identification) of the regression workflow applies
  directly: in panel regressions, the identification comes
  from within-unit variation, and strict exogeneity is the
  assumption to state and defend.

---

*Last updated: April 2026.*
