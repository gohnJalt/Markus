---
tags: [econometrics, time-series, non-stationary, cointegration, unit-roots, macro]
related: [modern-toolkit-references, var-identification, local-projections, stochastic-processes, run-a-regression-properly, Principles]
---

# Non-Stationary Time Series and Cointegration

This note assumes familiarity with stationary time series — ARMA
processes, VARs, and the concept of a unit root as a statistical
object. The mathematical foundations are in `stochastic-processes.md`:
the Dickey-Fuller distribution as a functional of Brownian motion
(§8.3 there), the Wold decomposition theorem (§4), and the relationship
between integrated and stationary processes. What this note adds is
the applied and inferential layer: how to test for unit roots honestly,
what cointegration means and how to model it, and how to reason about
long-run equilibrium relationships when the data are non-stationary.

This is also the note where spurious regression lives as an active
concern. Non-stationary series correlate with each other by
construction. Every applied economist working with macro time series
needs to have this worry active throughout, which is why unit root
testing and cointegration testing belong at the start of any levels
regression in macro, not as an afterthought.

---

## Why non-stationarity matters for empirical work

A stationary series fluctuates around a fixed mean; its shocks are
transitory. A unit-root process (I(1)) accumulates shocks; each
innovation permanently shifts the level of the series. The
distinction is not merely statistical — it determines what kind of
economic question the data can answer and which regression methods
are valid.

If two I(1) series — say, consumption and income — share a common
stochastic trend, they are **cointegrated**: there exists a linear
combination of the two that is stationary, an equilibrium relationship
the series cannot permanently deviate from. This is the foundation
for error correction models: the ECM describes both short-run dynamics
and the speed at which deviations from the long-run equilibrium are
corrected.

The practical implications are direct:

- A levels regression of one I(1) variable on another I(1) variable,
  without establishing cointegration, is potentially **spurious**
  (Granger and Newbold 1974): high $R^2$, significant $t$-statistics,
  no structural content.
- Standard $t$- and $F$-statistics are not normally distributed when
  regressors are I(1) under the null of no cointegration — the null
  distribution is non-standard and must be tabulated.
- If cointegration holds, the Granger Representation Theorem mandates
  the ECM form. Running a VAR in differences discards the long-run
  dynamics and misspecifies the model.

The working sequence in applied macro: test for unit roots → test for
cointegration if variables appear I(1) → if cointegrated, specify an
ECM or VECM → if not cointegrated, difference the series before
running regressions or a VAR.

---

## Unit roots: the statistical theory

The DF distribution is non-standard because, under the unit root null,
the OLS estimator of the AR(1) coefficient converges at rate $T$
rather than $\sqrt{T}$, to a Brownian-motion functional. For the
demeaned case:

$$T(\hat\rho - 1) \xrightarrow{d} \frac{\int_0^1 W(r)\, dW(r)}{\int_0^1 W(r)^2\, dr}$$

The numerator is a stochastic integral and is not normal — it has a
negative mean (the OLS estimator is biased toward stationarity) and a
long left tail. Critical values must come from this distribution;
using $t$-table critical values for the DF statistic is wrong.

**The deterministic component matters.** The DF distribution changes
depending on whether a constant, a trend, or neither is included in
the test regression. The three cases — pure random walk (no
deterministics), random walk with drift (constant only), and random
walk around a deterministic trend (constant and trend) — have
different critical values. The researcher must choose based on the
visual behavior of the series. Getting this wrong biases the test.

**The power problem.** Unit root tests have low power against
near-unit-root stationary alternatives. An AR(1) with $\rho = 0.95$
is hard to distinguish from $\rho = 1$ in samples of 40–80 years at
quarterly frequency — the typical range for macro data. "Fail to
reject the unit root null" is not strong evidence of a unit root; it
may reflect low power against a persistent but stationary process.
Markus holds this in mind throughout: integration order is a
maintained hypothesis, not a testable fact.

---

## Unit root tests in practice

**The Augmented Dickey-Fuller (ADF) test.** Augments the DF
regression with lagged differences to account for serial correlation:

$$\Delta y_t = \mu + \gamma y_{t-1} + \sum_{j=1}^p \delta_j \Delta y_{t-j} + \epsilon_t$$

Test $H_0: \gamma = 0$ (unit root). Lag order $p$ is selected by
information criteria (AIC/BIC applied to this regression); the
Ng-Perron (2001) data-dependent rule is the more careful modern
approach. Too few lags inflate size; too many lose power.

**The Phillips-Perron (PP) test.** Adjusts for serial correlation
non-parametrically rather than by augmentation. Has known severe size
distortions in small samples with heteroskedastic errors. Markus does
not treat PP as a primary test; it is useful when ADF and PP agree,
and should be treated skeptically when they disagree.

**The KPSS test.** Kwiatkowski, Phillips, Schmidt, and Shin (1992)
reverses the null: $H_0$: stationary against $H_a$: unit root. KPSS
complements ADF. Failing to reject both ADF and KPSS simultaneously
is the uncomfortable case — the data are consistent with both I(0)
and I(1). Markus reports both tests, notes whether they agree, and
treats disagreement as a signal of near-unit-root behavior requiring
explicit robustness checks.

**The Elliott-Rothenberg-Stock efficient test (DF-GLS).** Designed to
have the best asymptotic power among feasible unit root tests for a
given local alternative. Achieves this through GLS detrending before
the DF regression, removing the bias in the standard DF from estimated
deterministic components. Better power than ADF against near-unit-root
alternatives. This is Markus's preferred primary test alongside KPSS.

**Structural breaks and unit roots.** Perron (1989) showed
dramatically that allowing for a structural break in the deterministic
component transforms most macroeconomic series from non-stationary to
stationary. A stationary-around-a-break process mimics a unit root to
standard tests, and the tests cannot distinguish them without allowing
for the break. Zivot and Andrews (1992) estimate the break date
endogenously and test for a unit root conditionally. When the series
contains a visually obvious level shift or trend break, apply
Zivot-Andrews before drawing conclusions.

**Turkish data.** Turkish macroeconomic series have multiple documented
structural breaks: the 2001 banking crisis (level break in most
financial series), the 2005 currency reform (the redenomination of
the lira, visible in any nominal series), and the 2018–2022 monetary
policy regime change (large shifts in the real interest rate and the
exchange rate-inflation relationship). Standard unit root tests on
Turkish series without accounting for these breaks will produce
misleading results. Always apply Zivot-Andrews on Turkish macro data
and plot the series before drawing unit root conclusions.

---

## Spurious regression

Granger and Newbold (1974) demonstrated that regressing one
independent random walk on another produces significant coefficients
and high $R^2$ purely from trending, without any structural
relationship. Phillips (1986) provided the theory: the $t$-statistic
for a coefficient in a regression of one I(1) series on another
diverges in distribution as $T \to \infty$ under the null of no
cointegration. The regression is spurious.

The practical warning: any levels regression of a macroeconomic
variable on another should be treated as potentially spurious until
cointegration is established. "These two series trend together" is
not evidence of a relationship — it is the null hypothesis for
spurious regression. Cointegration testing is the correct inference
procedure; declaring significance from standard $t$-statistics in a
levels regression of I(1) variables is not.

---

## Cointegration: the concept and the representation theorem

Two series $y_{1t}$ and $y_{2t}$ are cointegrated if both are I(1)
but $y_{1t} - b \cdot y_{2t}$ is I(0) for some scalar $b$. The
cointegrating vector $(1, -b)'$ defines the long-run equilibrium
relationship; any deviation from $y_{1t} - b y_{2t} = 0$ is
transitory.

The economic interpretation: $y_{1t}$ and $y_{2t}$ share a common
stochastic trend — the same permanent shock drives both. Long-run
money demand (money supply and nominal GDP cointegrated with
elasticity 1), purchasing power parity (nominal exchange rate and
relative price levels), and fiscal sustainability (government debt
and the primary balance) are canonical examples where economic
theory predicts a cointegrating relationship.

**Granger Representation Theorem** (Engle and Granger 1987). If
$y_t = (y_{1t}, y_{2t})'$ is cointegrated with cointegrating vector
$\beta$, then the system has an error correction model (ECM)
representation:

$$\Delta y_{1t} = \alpha_1 (y_{1,t-1} - b\, y_{2,t-1}) +
\text{short-run dynamics} + \epsilon_{1t}$$
$$\Delta y_{2t} = \alpha_2 (y_{1,t-1} - b\, y_{2,t-1}) +
\text{short-run dynamics} + \epsilon_{2t}$$

The error correction term $y_{1,t-1} - b\, y_{2,t-1}$ is stationary
by the cointegration assumption. The adjustment coefficients
$\alpha_1$ and $\alpha_2$ govern how fast each series corrects toward
equilibrium. If $\alpha_1 = 0$, $y_{1t}$ does not adjust — it is
weakly exogenous with respect to the long-run relationship.

This theorem mandates the ECM form: for any cointegrated system, the
ECM is the correct dynamic specification. Discarding the error
correction term by working entirely in differences is not a
conservative choice — it is a misspecification that throws away the
long-run dynamics and biases the short-run estimates.

---

## The Engle-Granger two-step procedure

The simplest operational approach to cointegration.

**Step 1.** Regress $y_{1t}$ on $y_{2t}$ (and any deterministic terms)
in levels:
$$y_{1t} = \mu + b\, y_{2t} + e_t$$

The OLS estimator $\hat b$ is **super-consistent**: it converges at
rate $T$ rather than $\sqrt{T}$. This means the cointegrating
coefficient is estimated very precisely in large samples, even without
instruments.

**Step 2.** Test whether the residuals $\hat e_t = y_{1t} - \hat\mu -
\hat b\, y_{2t}$ are I(0) using ADF or DF-GLS. Critical values are
non-standard because the residuals are estimated, not observed —
standard DF tables are not applicable. Use Engle-Granger (1987),
MacKinnon (1991/2010), or the critical values from the `urca` package.

If the null of a unit root in the residuals is rejected, the Granger
Representation Theorem applies and the ECM can be estimated in a
second stage.

**Weaknesses.** Engle-Granger tests for only a single cointegrating
vector, which is a restriction when $n > 2$ (multiple cointegrating
relationships are possible). The two-step procedure propagates
estimation error from step 1 into step 2, affecting the critical
values. Inference on the cointegrating vector itself is non-standard
in small samples. For any system with more than two variables, the
Johansen procedure is the better approach.

---

## The Johansen procedure

Johansen (1988, 1991) develops a maximum likelihood framework for
estimating and testing cointegration in a system of $n$ I(1)
variables, explicitly allowing for multiple cointegrating vectors.

**The VECM.** Johansen's procedure estimates the Vector Error
Correction Model:

$$\Delta y_t = \Pi y_{t-1} + \Gamma_1 \Delta y_{t-1} + \cdots +
\Gamma_{p-1} \Delta y_{t-p+1} + \mu + \epsilon_t$$

The long-run information is in $\Pi = \alpha\beta'$, where $\beta$ is
the $n \times r$ matrix of cointegrating vectors, $\alpha$ is the
$n \times r$ matrix of adjustment coefficients, and $r$ is the
cointegrating rank. If $r = 0$, no cointegration; if $r = n$, all
variables are stationary; the economically interesting cases are
$0 < r < n$.

**Rank tests.** Two test statistics:

- *Trace statistic*: tests $H_0: r \leq r_0$ against $H_a: r > r_0$.
  Start with $r_0 = 0$ and increase until the null is not rejected.
- *Maximum eigenvalue statistic*: tests $H_0: r = r_0$ against
  $H_a: r = r_0 + 1$.

Both have non-standard distributions (functionals of Brownian motion
depending on $n - r_0$) whose critical values depend on the
deterministic terms included in the VECM. The researcher must specify
whether the constant and trend are restricted (inside the cointegrating
space) or unrestricted (outside it) — the economic interpretation
differs and the critical values differ.

**Small-sample distortion.** The trace statistics are size-distorted
upward in small samples: too many cointegrating vectors tend to be
detected. For $T < 80$ quarterly observations — common for emerging
market data and for Turkey — apply the Reinsel-Ahn (1992) small-sample
correction (multiplying the trace statistic by $(T - np)/T$) or use
bootstrap critical values. Reporting uncorrected Johansen results with
$T = 50$ and claiming three cointegrating vectors is not reliable.

**Identifying structural cointegrating vectors.** With $r > 1$
cointegrating vectors, the matrix $\beta$ is not unique — any linear
combination of cointegrating vectors is also cointegrating. Identifying
individual vectors as economically interpretable relationships requires
restrictions, parallel to the identification problem in SVARs. Economic
theory typically motivates these: long-run money demand, PPP, and
fiscal sustainability are three relationships that can be separately
identified by restricting entries of $\beta$. King, Plosser, Stock,
and Watson (1991) is the canonical reference for structural
identification of common stochastic trends.

**Weak exogeneity.** If $\alpha_i = 0$ (the $i$-th row of $\alpha$ is
zero), variable $i$ does not adjust to deviations from the
cointegrating relationship — it is weakly exogenous with respect to
the long-run parameters. Testable via a likelihood ratio test on
$\alpha$. Establishing weak exogeneity simplifies the system: the
exogenous variable can be conditioned on, and the remaining variables
modeled as a partial ECM. Importantly: weak exogeneity with respect
to the long-run parameters does not imply Granger non-causality in
the short run.

---

## Bounds testing: the ARDL approach

Pesaran, Shin, and Smith (2001) developed bounds testing for
cointegration in an Autoregressive Distributed Lag (ARDL) framework.
The key advantage: it does not require the regressors to have a known
and uniform integration order. The test is valid whether the
regressors are I(0) or I(1) — or a mixture.

**The setup.** For a bivariate case:

$$\Delta y_t = c + \theta_1 y_{t-1} + \theta_2 x_{t-1} +
\sum_{j=0}^q \phi_j \Delta x_{t-j} +
\sum_{j=1}^p \psi_j \Delta y_{t-j} + \epsilon_t$$

The null of no long-run relationship is $\theta_1 = \theta_2 = 0$.
The $F$-statistic for this joint test has a non-standard distribution
depending on the integration order of $x_t$. Pesaran et al. tabulate
lower bounds (assuming $x_t$ is I(0)) and upper bounds (assuming I(1))
for the critical values: if $F$ exceeds the upper bound, a long-run
relationship is established regardless of integration order; below the
lower bound, no relationship; between them, the result is inconclusive.

**Why bounds testing matters for Turkish data.** Turkish macro series
frequently have uncertain integration orders — structural breaks and
high volatility make unit root tests uninformative or contradictory.
ARDL bounds testing avoids the pre-testing problem: it tests for a
long-run relationship without requiring certainty about integration
orders. For short and volatile samples, ARDL is often the more
defensible approach than Johansen, which assumes known I(1) variables.
This is one case where the standard methodology for large
developed-economy datasets genuinely needs to be adjusted for the
Turkish context.

---

## Fractional integration

Some macro series (financial series in particular) lie between I(0)
and I(1): their autocorrelations decay slowly but not at unit-root
pace. Fractionally integrated processes $I(d)$ with $0 < d < 1$
cover this middle ground. Semi-parametric estimators (Geweke-Porter-
Hudak, Exact Local Whittle, Shimotsu-Phillips) estimate $d$
non-parametrically. The econometric treatment of fractionally
integrated series in cointegrating regressions is more involved than
the standard I(0)/I(1) framework and is mostly specialist territory.
Flag when it might be relevant — particularly for high-frequency or
financial macro data — and seek specialist guidance before applying
fractional cointegration models.

---

## The decision tree

**Q1: Are the series plausibly I(1)?**
- Plot each series. Does it look like a random walk (variance growing
  with time, no visible mean-reversion)?
- Apply DF-GLS and ADF (with and without trend), KPSS (stationarity
  null), and Zivot-Andrews if structural breaks are visible.
- Both DF-GLS and ADF fail to reject + KPSS does not reject →
  ambiguous; treat as I(1) for safety and run robustness in
  differences.
- Both DF-GLS and ADF reject the unit root → stationary; proceed with
  a levels regression or levels VAR as appropriate.
- Turkish data: always apply Zivot-Andrews; assume breaks at 2001 and
  2018 at minimum.

**Q2: Bivariate or system?**
- Bivariate ($n = 2$), at most one cointegrating relationship expected
  → Engle-Granger two-step. Simple; super-consistent long-run
  coefficient; correct non-standard critical values for the residual
  ADF.
- System ($n > 2$), possibly multiple cointegrating relationships →
  Johansen VECM with likelihood ratio rank tests.

**Q3: Are integration orders known with confidence?**
- All I(1) with confidence → Johansen preferred.
- Uncertain (some variables may be I(0) or near I(0)) → ARDL bounds
  testing. Avoids the pre-testing problem.

**Q4: Is $T$ small?**
- $T > 100$ quarterly observations → Johansen is reliable.
- $T < 80$ quarterly observations → Apply Reinsel-Ahn small-sample
  correction or bootstrap critical values. Consider ARDL as an
  alternative.

**Q5: Does economic theory predict specific cointegrating vectors?**
- Yes → Restrict $\beta$ in the Johansen VECM and test via likelihood
  ratio.
- No → Identify from data; impose just-identifying normalizations.

**The default, when in doubt:** test with DF-GLS + KPSS + Zivot-
Andrews; if I(1), apply Johansen for a system or Engle-Granger for a
bivariate case; use ARDL when integration orders are uncertain or
$T$ is small; report the full diagnostic table.

---

## The integration order assumption, honestly

The I(1) assumption is a maintained hypothesis, not a testable fact.
Unit root tests cannot distinguish a near-unit-root stationary process
from a unit root process with sample sizes typical of macro work. The
consequences of getting it wrong are asymmetric and both bad:

- Treating I(0) series as I(1) and over-differencing wastes long-run
  information and eliminates the ECM dynamics.
- Treating I(1) series as I(0) and running regressions in levels
  risks spurious inference with invalid standard errors.
- Using the wrong cointegrating rank misspecifies the dynamic model
  in a way that persists in all downstream analysis.

The honest treatment: report the full battery of unit root tests;
acknowledge when results are ambiguous; run the main analysis in both
the ECM specification (if cointegration is suspected) and the
differences specification (if uncertain); report when the conclusions
differ. The analyst's job is not to resolve the ambiguity — unit root
testing cannot do this — but to be transparent about it and to check
whether the conclusions of interest are robust to the maintained
hypothesis.

For Turkish data with documented structural breaks, ambiguity in
integration order is nearly universal. Markus treats integration order
as explicitly maintained, runs both specifications, and flags
disagreement rather than choosing the result that fits the narrative.

---

## Common failure modes

**Running levels regressions without testing for cointegration.** The
foundational error. Standard errors are invalid, $t$-statistics are
non-pivotal, and $R^2$ is meaningless as a fit measure. Remains common
in published work involving macro time series.

**Using ADF alone.** ADF has low power and is sensitive to lag
selection. DF-GLS is uniformly more powerful; KPSS provides a
complementary reversed null. Reporting only ADF results as if unit
root testing is settled is insufficient.

**Ignoring structural breaks.** Particularly first-order for Turkish
and other emerging-market data. The Perron (1989) critique is not
technical fine print; it is an empirical concern for any dataset with
known regime changes.

**Johansen in small samples without corrections.** Trace statistics
are size-distorted upward when $T$ is small. Finding more
cointegrating vectors than economic theory predicts in a small
Turkish sample is likely a small-sample artifact, not a discovery.
Apply the Reinsel-Ahn correction or bootstrap critical values.

**Treating the cointegrating vector as structural without
identification.** The cointegrating vector is identified by
normalization convention. With multiple cointegrating vectors,
structural identification requires economic restrictions on $\beta$.
Labeling a reduced-form cointegrating vector "the long-run money demand
elasticity" without imposing the identifying restrictions for that
economic relationship is not warranted.

**Confusing weak exogeneity and causal exogeneity.** A variable with
$\alpha_i = 0$ does not adjust to the long-run relationship — it is
weakly exogenous with respect to the long-run parameters. This says
nothing about whether it Granger-causes the other variables in the
short run, or whether it is exogenous in a causal sense for a
structural question. The concepts are distinct; conflating them is a
recurring error in applied cointegration work.

**Differencing away the long-run relationship.** Running a VAR in
first differences when the variables are cointegrated drops the error
correction term. This is not a conservative specification choice; it
is misspecification that biases both the short-run dynamics and any
forecast horizon beyond the near term.

---

## What to report

1. **Unit root test results.** DF-GLS and ADF (with lag-selection
   procedure stated and deterministic terms specified), KPSS, and
   Zivot-Andrews for each series. Note structural breaks and how they
   were handled.
2. **Cointegration test results.** Johansen trace and max-eigenvalue
   statistics with Reinsel-Ahn correction if $T$ is small; the
   cointegrating rank chosen and the basis for the choice; the
   deterministic terms specification.
3. **The estimated cointegrating vector(s).** With the normalization
   and any structural identifying restrictions stated explicitly.
4. **The ECM/VECM specification.** The adjustment coefficients $\alpha$
   with standard errors; a likelihood ratio test for weak exogeneity
   where relevant; the lag structure for the short-run dynamics.
5. **Robustness to integration order.** How do the main conclusions
   change if the series are treated as I(0) (differences specification)
   vs I(1) with cointegration (ECM specification)?
6. **For Turkish data.** Explicit treatment of structural breaks;
   rationale for the sample period and any truncation.

---

## Implementation, by language

**R:**

- `urca` — the canonical package for unit root and cointegration
  analysis. ADF, KPSS, DF-GLS (Elliott-Rothenberg-Stock), Zivot-
  Andrews, and the full Johansen VECM (trace and max-eigenvalue
  tests, `ca.jo()`; ECM estimation via `cajorls()`). The standard
  implementation.
- `tseries::adf.test` — simpler ADF interface; less flexible than
  `urca`.
- `vars::VAR` and the `vec2var()` conversion — bridge from the Johansen
  cointegration test in `urca` to the VECM structure in `vars`.
- `ardl` — ARDL estimation and Pesaran-Shin-Smith bounds testing.
  Check the current version for the bounds-test critical values table.
- `tsDyn` — VECM with bootstrapped critical values for the rank test;
  useful for small-sample corrections.

**Stata:**

- `dfuller`, `dfgls`, `kpss`, `zandrews` — unit root tests. All widely
  available; `dfgls` and `zandrews` are user-written but standard.
- `vecrank` and `johansen` — Johansen cointegration tests; `vecrank`
  is Stata's built-in version; `johansen` is more flexible.
- `ardl` (Kripfganz and Schneider) — the most complete ARDL/bounds
  testing implementation currently available in any language. Preferred
  over the R `ardl` package for production work with bounds testing.
- `vec` — VECM estimation after the rank test.

**Python:**

- `statsmodels.tsa.stattools`: `adfuller`, `kpss`, `zivot_andrews`
  (unit root tests); `coint` (Engle-Granger bivariate cointegration).
- `statsmodels.tsa.vector_ar.vecm`: VECM estimation with the Johansen
  rank test.
- Bounds testing has limited Python support; Stata is preferred for
  ARDL bounds work.

Markus's defaults: R with `urca` for the full Johansen procedure;
Stata with the Kripfganz-Schneider `ardl` package for ARDL bounds
testing where integration orders are uncertain. Python is adequate for
unit root testing but the VECM and ARDL machinery is more mature in R
and Stata.

---

## What this note doesn't cover, and where to find it

**Structural VAR identification in the presence of unit roots.** When
the variables are cointegrated, the correct structural model is a VECM
with identification of structural shocks. The identification content
— Cholesky, sign restrictions, proxy SVARs — is in
`var-identification.md`. This note provides the non-stationarity layer;
that note provides the identification layer. They are companions.

**LP-IV and impulse responses with I(1) data.** LPs can be run on
levels of I(1) series when the shock variable provides identification;
the shock variation controls for common stochastic trends. See
`local-projections.md` for the methodological discussion.

**Panel cointegration.** Kao, Pedroni, and Westerlund tests for
cointegration in panels of time series. Relevant for cross-country
macro studies; not covered here. Worth its own note when a project
demands it.

**Fractional cointegration.** Multivariate long-memory processes and
fractionally cointegrated systems. Specialist territory; flagged above.

**Forecasting non-stationary series.** Beveridge-Nelson decompositions,
state-space trend-cycle models for forecasting. Not covered in depth
here; see `stochastic-processes.md` for mathematical foundations and
`tools-and-integrations.md` for forecasting tools.

---

## Cross-references

- `10_Methods/Econometrics/var-identification.md` — structural
  identification of shocks in a VAR or VECM. When variables are
  cointegrated, the VECM is the correct specification; the
  identification content of that note applies to the VECM system.
- `10_Methods/Econometrics/local-projections.md` — LPs as an
  alternative to VARs for impulse response estimation. LPs do not
  require knowledge of integration order, which is a practical
  advantage over VECMs in ambiguous settings.
- `20_Math/stochastic-processes.md` — the Dickey-Fuller distribution
  as a functional of Brownian motion (§8.3), unit roots, cointegration,
  and the Wold decomposition that underlies the ECM representation.
  Read those sections before working with the distributional results
  for unit root tests cited here.
- `30_Data/AI-accessible-data-sources.md` — the Turkish data sources
  (TÜİK, CBRT, SGK, BDDK) where structural breaks and integration
  ambiguities are most severe. The methodology notes there on
  structural breaks are the context for applied unit root testing on
  Turkish series.
- `00_Identity/Principles.md` — the section on uncertainty and
  honesty. Treating integration order as a maintained hypothesis
  rather than an established fact is a direct application of that
  section's commitments.
- `50_Workflows/run-a-regression-properly.md` — the unit root and
  cointegration pre-testing step belongs in Stage 1 (question and
  setup) of the regression workflow, before any specification choices
  are made for a macro levels regression.

---

*Last updated: 2026-04. The Johansen procedure and the Engle-Granger
two-step have been stable for decades. The ARDL bounds testing
approach (Pesaran-Shin-Smith 2001) has gained practical traction for
datasets with uncertain integration orders; the Kripfganz-Schneider
Stata implementation has made it accessible. The structural break
literature — Zivot-Andrews and the multiple break extensions — is
particularly important for emerging-market applications. Turkish data
applications remain a recurring methodological concern; the notes in
`AI-accessible-data-sources.md` are the practical companion.*
