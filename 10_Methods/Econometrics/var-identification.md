---
tags: [econometrics, time-series, macro, var, svar, identification]
related: [modern-toolkit-references, local-projections, non-stationary-time-series, stochastic-processes, instrumental-variables, run-a-regression-properly, Principles]
---

# VAR Identification and Structural Shocks

This note assumes you know what a reduced-form VAR is — a system of
equations where each variable is regressed on lags of all variables.
The mathematical foundations live in [[`stochastic-processes.md`]]: the
Wold decomposition, companion-matrix stability, state-space
representations of ARMA systems. The LP-vs-VAR equivalence debate
lives in [[`local-projections.md`]]. What this note addresses is the
*identification problem*: a reduced-form VAR describes the statistical
structure of the data but cannot, on its own, say anything about the
causal effect of a structural shock — a monetary policy surprise, an
oil supply disruption, a technology innovation — on the economy.
Getting from the reduced form to structural causal claims requires
restrictions, and those restrictions are the content of structural VAR
identification.

This is a note about structural identification, not about VARs as a
forecasting tool. If the goal is forecasting, the reduced-form VAR (or
its regularized Bayesian variants) is the right tool and the
identification content here is irrelevant. If the goal is causal
impulse responses — how does output respond to a monetary shock over
eight quarters — you need what follows.

---

## What a structural VAR is actually estimating

The estimand is the **structural impulse response function**: the
expected path of variable $y_j$ over horizons $h = 0, 1, \ldots, H$
in response to a unit structural shock to equation $i$, where
"structural" means an orthogonal, economically interpretable
innovation — a genuine monetary policy surprise, not a composite of
contemporaneous shocks that happens to correlate with interest rate
movements.

The reduced-form VAR is:

$$y_t = A_1 y_{t-1} + \cdots + A_p y_{t-p} + u_t, \qquad
\mathbb{E}[u_t u_t'] = \Sigma_u$$

The innovations $u_t$ are reduced-form residuals. They are correlated
across equations because contemporaneous variables affect each other
within the period. A monetary shock does not arrive in isolation; it
is mixed with contemporaneous output and price movements that feed
back into the rate decision. The reduced form cannot decompose these.

The structural form is:

$$B_0 y_t = B_1 y_{t-1} + \cdots + B_p y_{t-p} + \varepsilon_t,
\qquad \mathbb{E}[\varepsilon_t \varepsilon_t'] = I$$

where $B_0$ captures contemporaneous relationships and $\varepsilon_t$
are orthogonal structural shocks with unit variance. The structural
moving-average representation is:

$$y_t = \sum_{h=0}^{\infty} \Theta_h \varepsilon_{t-h}, \qquad
\Theta_h = \Phi_h B_0^{-1}$$

where $\Phi_h$ are the reduced-form moving-average coefficients. The
entry $[\Theta_h]_{ji}$ is the response of variable $j$ at horizon $h$
to structural shock $i$ — the structural IRF.

The identification gap: $B_0^{-1}$ must satisfy
$B_0^{-1}(B_0^{-1})' = \Sigma_u$, a system of $n(n+1)/2$ equations
in $n^2$ unknowns. The gap is $n(n-1)/2$ free parameters: 3
restrictions needed for a 3-variable system, 15 for a 6-variable
system. All SVAR identification methods are ways to supply those
missing restrictions.

---

## Recursive (Cholesky) identification

The simplest scheme restricts $B_0$ to be lower triangular: variable 1
has no contemporaneous dependence on any other variable; variable 2
depends on variable 1 but nothing else; variable 3 depends on 1 and 2;
and so on. Under this restriction $B_0^{-1}$ is the Cholesky factor of
$\Sigma_u$, uniquely determined.

The economic interpretation: recursive identification imposes a
**Wold-causal ordering** — a strict contemporaneous hierarchy among
the variables. The variable ordered first is the most exogenous;
the variable ordered last responds contemporaneously to all others.

**Classical applications.** Christiano, Eichenbaum, and Evans (1999)
ordered the policy rate between slow-moving variables (output, prices,
commodity prices) and fast-moving financial variables (money, exchange
rate). The economic premise: the Fed observes output and prices within
the period but output and prices do not respond to monetary policy
within the same month. Blanchard and Quah (1989) imposed a long-run
recursive restriction instead: demand shocks have no long-run effect
on output. The identification logic is the same; the restriction
applies at horizon $h \to \infty$ rather than $h = 0$.

**When it is defensible.** The ordering story must be plausible at the
data frequency. For monthly or quarterly data, some contemporaneous
exclusion restrictions can be argued from institutional timing — the
Fed meets on scheduled dates; price indices are released with a lag.
For annual data, ordering restrictions have no real timing
justification and Cholesky identification is harder to defend.

**The ordering problem.** Cholesky identification is not
ordering-invariant. A different ordering of the same variables gives a
different $B_0$ and different IRFs. If different orderings of the same
variables give substantively different IRFs for the shock of interest,
the identification scheme is doing real work that should be argued on
economic grounds, not accepted as a software default.

**What Cholesky does not allow you to test.** The lower-triangular
restriction is exactly-identifying — it supplies precisely the $n(n-1)/2$
restrictions needed, leaving no testable over-identifying restrictions.
It cannot be verified against the data. The assumption must be argued
or it is not an identification strategy; it is a default.

---

## Sign restrictions

Sign restrictions (Uhlig 2005; extended by Arias, Rubio-Waggoner, and
Zha 2018) identify structural shocks by imposing the expected sign of
their impulse responses rather than parametric zero restrictions. A
contractionary monetary policy shock is identified as a shock that
raises the interest rate and does not lower prices or raise output on
impact and over some horizon $h = 0, 1, \ldots, K$.

The identifying assumption is qualitative rather than quantitative: we
know the direction of the response but not its magnitude. This is
typically more defensible than the specific parametric restrictions
Cholesky imposes.

**Set identification.** Sign restrictions are set-identifying rather
than point-identifying. For any set of signs that holds, a range of
structural models is consistent with the data and the restrictions.
The reported "impulse responses" are either the median of the
posterior distribution over this set (Uhlig's original algorithm) or
the bounds of the identified set (the Arias-Rubio-Waggoner-Zha
approach). These are not the same thing and the distinction matters.
The Arias-Rubio-Waggoner-Zha framework has the correct Bayesian
interpretation; it is the preferred implementation for modern work.

**The Fry-Pagan critique.** Fry and Pagan (2011) showed that the
"median response" — the most widely reported summary — is not
necessarily achievable by any single structural model. The median of
the response at horizon 2 and the median at horizon 5 may come from
different models, and the composite picture has no structural
interpretation. Their solution: the **median target** model —
the model closest to the median across all horizons. This correction
is standard practice in rigorous applied work; reporting only the
horizon-by-horizon median without it is an unforced error.

**Narrative sign restrictions.** Antolín-Díaz and Rubio-Ramírez (2018)
extended the framework to include restrictions on structural shocks at
specific historical dates: the monetary shock in October 1979 (the
Volcker disinflation) must be contractionary. These "narrative sign
restrictions" sharpen identification substantially without imposing
exact parametric values, and they are the cleanest way to bring
institutional knowledge into a sign-restrictions SVAR.

**How many restrictions, over how many horizons.** Restricting
on-impact signs only typically leaves the identified set large.
Extending restrictions to horizons 1 through $K$ shrinks the set but
imposes more. The researcher must argue what horizon is economically
defensible and should report how results change as the restriction
horizon changes. Shrinking the identified set is not the same as
improving identification — it is trading substantive assumptions for
apparent precision.

---

## Narrative identification

Narrative identification (Romer and Romer 2004; Ramey 2011, 2016)
constructs a time series of structural shocks directly from historical
records — Federal Reserve meeting minutes, congressional budget
documents, news coverage of oil-supply disruptions — rather than
identifying them econometrically from the VAR residuals.

The resulting shock series is then used in one of two ways:
- As the shock variable in an LP or LP-IV (the more common modern
  approach, described in [[`local-projections.md`]]).
- As an external instrument for a proxy SVAR (described below).

**What narrative identification buys.** It bypasses the identification-
by-restriction problem: if the structural shock series can be
constructed directly from institutional records, parametric
restrictions on $B_0$ are not needed. The quality of identification
depends on the quality of the historical reading, not on parametric
assumptions about contemporaneous causal structure.

**What it costs.** Narrative shocks are typically sparse — one
observation per FOMC meeting, one per major tax bill, one per OPEC
announcement. The series may be contaminated by anticipation effects:
the Fed pre-announces; Congress signals budget intentions; oil markets
incorporate news before the formal announcement. The Ramey-Romer
dispute over fiscal multipliers is in part a dispute over whether
narrative military-spending shocks capture genuinely unanticipated
changes or merely the formal announcement of decisions long since
priced in.

**Romer-Romer (2004) monetary shocks** are constructed by subtracting
the Fed's internal Greenbook forecast from the observed rate change to
isolate the surprise component. Miranda-Agrippino and Ricco (2021)
showed that part of the Romer-Romer "shock" reflects the market
updating its beliefs about economic conditions based on what the Fed's
action signals about the Greenbook forecast — the information channel.
Their correction removes this component and produces cleaner monetary
surprises. For monetary applications, engage with this correction
rather than treating Romer-Romer (2004) as the final word.

---

## Proxy SVARs (SVAR-IV)

Proxy SVARs — independently developed by Mertens and Ravn (2013) and
Stock and Watson (2012/2018) — use an external instrument to identify
one or more structural shocks from the reduced-form VAR without
imposing parametric restrictions on the full $B_0$ matrix.

**The setup.** You have reduced-form residuals $u_t$. You have a proxy
$m_t$ — a narrative shock series, a high-frequency financial surprise,
or another exogenous series — that satisfies:

$$\mathbb{E}[m_t \varepsilon_{1t}] \neq 0 \qquad \text{(relevance)}$$
$$\mathbb{E}[m_t \varepsilon_{jt}] = 0, \quad j \neq 1 \qquad
\text{(exogeneity)}$$

Under these conditions $m_t$ can recover $b_1$ — the column of
$B_0^{-1}$ corresponding to the target shock — via IV, without
restricting the rest of $B_0$.

**The IV logic.** Projecting the first reduced-form residual $u_{1t}$
on the proxy gives the first-stage relationship. The instrumented
component of $u_{1t}$ is the proxy-SVAR estimate of the structural
shock. The structural IRFs are then recovered from the relationship
between $b_1$ and the reduced-form moving-average coefficients.

**Connection to LP-IV.** Stock and Watson (2018) and
Plagborg-Møller and Wolf (2021) showed that the proxy SVAR and the
LP-IV using the same instrument are asymptotically equivalent — they
identify the same structural IRF in population. In finite samples they
differ for the usual LP-vs-VAR reasons (the VAR imposes lag structure;
the LP does not). The preferred modern practice: report both and
discuss any disagreement. Agreement is a robustness result. Disagreement
is a finding worth explaining.

**The LATE framing applies.** Just as in LP-IV (see
[[`local-projections.md`]]), the proxy SVAR estimates the structural IRF
for periods during which the proxy moves the target shock. This is a
LATE in the Imbens-Angrist sense. For high-frequency monetary
identification, the compliers are FOMC announcement windows; the
estimated IRF is the response to monetary surprises as observed in
those windows, not the response to "monetary policy" in general.

**Weak proxy.** The first-stage $F$-statistic for projecting $u_{1t}$
on $m_t$ is the standard weak-instruments diagnostic. The same
discipline as in [[`instrumental-variables.md`]] applies: effective
$F < 10$ (Olea-Pflueger criterion) is a problem, and Anderson-Rubin
weak-instrument-robust confidence sets are the fix. Reporting proxy
SVAR results with a weak proxy and standard SEs is the SVAR analogue
of reporting IV results with $F = 6$.

**Partial identification with multiple proxies.** With $k$ proxies,
$k$ structural shocks can be identified while remaining agnostic about
the rest of $B_0$. This partial identification is one of the proxy
SVAR's main practical strengths for applied macro work, where
researchers often want to say something about two or three named shocks
without committing to a full structural model.

---

## State-space representations and the Kalman filter

Many macro models can be written in state-space form, and VARs are a
special case. The state-space representation separates the measurement
equation (what we observe) from the transition equation (how the
underlying state evolves):

$$y_t = Z \alpha_t + \epsilon_t \qquad \text{(measurement)}$$
$$\alpha_{t+1} = T \alpha_t + R \eta_t \qquad \text{(transition)}$$

where $\alpha_t$ is the unobserved state vector. A VAR($p$) is the
special case with $Z = [I, 0, \ldots, 0]$, $T$ the companion matrix,
and no measurement error. The state-space form unifies VARs, VECMs,
unobserved-components models, time-varying-parameter VARs, and dynamic
factor models — all of which share the Kalman filter as the estimation
engine.

**The Kalman filter.** For a linear-Gaussian state-space model, the
Kalman filter computes the optimal (minimum-mean-square-error) linear
estimate of $\alpha_t$ given $y_1, \ldots, y_t$ via prediction and
update steps:

$$\hat\alpha_{t|t-1} = T \hat\alpha_{t-1|t-1}$$
$$\hat\alpha_{t|t} = \hat\alpha_{t|t-1} + K_t (y_t - Z \hat\alpha_{t|t-1})$$

where $K_t = P_{t|t-1} Z' F_t^{-1}$ is the Kalman gain and
$F_t = Z P_{t|t-1} Z' + H$ is the innovation variance. The **Kalman
smoother** runs the filter backward to compute $\hat\alpha_{t|T}$ —
the optimal state estimate given the full sample — which is needed for
historical decompositions and for estimating TVP-VARs.

**Applications in macro:**

- *Output gap estimation.* The HP filter is a special case; more
  structural trend-cycle decompositions (Watson 1986; Clark 1987) use
  a state-space model with separate stochastic trend and cycle
  equations. The gap is identified by structure (the assumed dynamics
  of the cycle), not from data alone — a fact that should be
  acknowledged in any policy-relevant discussion of the output gap.

- *Time-varying-parameter VARs.* Primiceri (2005) allows all VAR
  coefficients and the error covariance to evolve as random walks.
  The posterior over states is computed via the Carter-Kohn algorithm
  (a Kalman smoother variant for the coefficients; a different
  algorithm for the covariances). Use when the research question is
  whether monetary policy transmission has changed over time — not as
  a default specification, because the parameter-count is large and
  identification requires strong priors.

- *Factor-augmented VARs.* Stock and Watson (2002) dynamic factor
  model is a state-space model with a large measurement equation. The
  Kalman filter recovers the latent factors. FAVAR (Bernanke, Boivin,
  and Eliasz 2005) adds these factors to a structural VAR to use
  information from hundreds of series without overparameterizing the
  structural model.

---

## The decision tree

**Q1: Is the goal causal impulse responses or forecasting?**
- Forecasting only → reduced-form VAR or Bayesian VAR with Minnesota
  prior. No structural identification needed. Exit.
- Causal IRFs → identification required. Continue.

**Q2: Do you have an external instrument (proxy)?**
- Yes, instrument strong (effective $F > 10$) → proxy SVAR. This is
  the most credible identification when a good proxy exists. Also run
  LP-IV with the same instrument and compare.
- Yes, instrument weak → Anderson-Rubin weak-instrument-robust
  confidence sets. If the instrument is too weak to be useful,
  rethink the strategy before reporting.
- No → Continue.

**Q3: Do you have prior information about contemporaneous restrictions?**
- Yes, credible Wold-causal ordering → Cholesky. Argue the ordering
  explicitly; probe alternative orderings as robustness.
- Yes, but only sign information → Sign restrictions with
  Arias-Rubio-Waggoner-Zha implementation; apply Fry-Pagan
  median-target correction; use narrative restrictions at historically
  identifiable dates if available.
- Only weak priors → Rethink. An SVAR without credible identification
  restrictions is a reduced-form VAR with a structural label.

**Q4: Are VAR parameters stable over the full sample?**
- Yes → Standard SVAR.
- No, and regime change is the research question → TVP-VAR (Primiceri
  2005). Computationally demanding; not a robustness check.

**The default, when in doubt:** proxy SVAR when a credible instrument
exists; sign restrictions with narrative sharpening otherwise; always
compare to LP-IV when using an external instrument. Cholesky only when
the causal ordering can be defended at the data frequency and the
argument is written down.

---

## The identification assumption, honestly

The load-bearing assumption varies by scheme. The honest treatment of
each:

**Cholesky.** The Wold-causal ordering: no contemporaneous causation
from later-ordered to earlier-ordered variables within the period. This
is exactly-identifying and untestable. The honest approach: run the
SVAR under alternative orderings and report whether IRFs for the shock
of interest are stable. If they are not, the ordering is doing real
work that the researcher is responsible for defending, not a neutral
default.

**Sign restrictions.** The identifying assumption is that the sign
pattern specified holds over the restriction horizon. Economically
motivated but untestable. The honest approach: report the width of the
identified set, not just a point summary; apply the Fry-Pagan
correction; check how results change as the restriction horizon varies.
Wide identified sets mean weak identification — that is information,
not an embarrassment to hide.

**Proxy SVARs.** Two conditions: relevance (testable via first-stage
$F$) and exogeneity of the proxy with respect to other structural
shocks (untestable). The honest approach: describe the proxy's
construction and what makes it plausibly exogenous; acknowledge what
contamination would violate exogeneity; apply Conley-Hansen-Rossi style
partial-exogeneity bounds from [[`instrumental-variables.md`]] where
relevant. The proxy is not an escape from the identification problem; it
relocates it.

**General rule.** Identification schemes in SVARs should be argued,
not assumed. The identifying restriction is a substantive claim about
the world. The researcher, not the econometric software, is responsible
for defending it.

---

## Common failure modes

**Imposing Cholesky without arguing the ordering.** Using the
"standard" textbook ordering, or copying a prior paper's ordering
without rechecking its economic logic for the current application.
The ordering is a substantive assumption; if it can't be argued, it
should not be imposed.

**Reporting the median sign-restricted response as a point estimate.**
Median responses from sign-restricted SVARs summarize an identified
set; they are not structural parameters. The identified set width is
information and should be reported. Confidence bands that collapse
to the median without showing the set width misrepresent the
identification uncertainty.

**Ignoring the Fry-Pagan critique.** Reporting a median-response IRF
constructed from different models at different horizons is standard
practice but not correct practice. Apply the median-target correction.

**Treating the proxy as identification-free.** The exogeneity condition
for the proxy is as untestable as any other identifying restriction.
The proxy shifts the identification argument from parametric
restrictions on $B_0$ to substantive claims about what the proxy
measures. Neither is free.

**Weak proxies without robust inference.** The F = 6 error has a
direct SVAR analogue. If the proxy first-stage is weak, standard
confidence sets are invalid. Anderson-Rubin intervals are the fix;
use them.

**Not comparing proxy SVAR to LP-IV.** When the same instrument is
available, the LP-IV and proxy SVAR identify the same population
object. Disagreement in finite samples is informative; not doing the
comparison is an unforced omission.

**Ignoring parameter instability in long samples.** A SVAR pooling
1960–2023 data pools across the Bretton-Woods era, the Great
Moderation, the zero-lower-bound period, and the post-2020 inflation
episode. If monetary policy transmission has changed — and there is
substantial evidence it has — pooling obscures structural change.
Rolling-window or TVP-VAR provides a useful check for any application
spanning multiple monetary regimes.

---

## What to report

1. **The estimand.** "The impulse response of real GDP to a
   contractionary monetary shock over 20 quarters, identified by
   high-frequency surprises in federal funds futures around FOMC
   announcements as a proxy."
2. **The identification scheme**, including all restrictions imposed,
   the proxy if applicable, and the economic argument for each
   restriction.
3. **IRF plots.** Point estimates with 68% and 90% confidence bands.
   For sign-restricted SVARs: the identified set bounds alongside the
   median-target response.
4. **First-stage diagnostic** for proxy SVARs: Olea-Pflueger effective
   $F$; Anderson-Rubin confidence sets if $F$ is marginal.
5. **Robustness to identification.** For Cholesky: alternative
   orderings. For sign restrictions: alternative restriction horizons
   and the width of the identified set. For proxy SVARs: comparison to
   LP-IV and partial-exogeneity relaxation.
6. **Comparison to LP-IV** whenever an external instrument is used.
7. **Parameter stability check** for samples spanning multiple regimes.
8. **A statement of which robustness checks would have changed the
   conclusion**, per [[`run-a-regression-properly.md`]].

---

## Implementation, by language

**R** is the most complete environment for modern SVAR work:

- `vars` — standard reduced-form VAR estimation, Cholesky SVAR, lag
  selection, stability checks. The workhorse starting point.
- `VARsignR` — Arias-Rubio-Waggoner-Zha sign restrictions with the
  correct posterior. The methodologically preferred implementation for
  sign-restriction work.
- `svars` — sign restrictions and some extensions; useful but less
  methodologically current than `VARsignR`.
- Proxy SVARs: no single canonical package; implement directly in R
  using OLS with the proxy as the instrument, following the
  Stock-Watson (2018) replication code. The procedure is simple enough
  that direct implementation is not burdensome.
- `dlm` and `KFAS` — flexible state-space models and Kalman
  filter/smoother; `dlm` is more flexible for custom specifications.
- `bvarsv` — Primiceri (2005) TVP-VAR.

**Stata:**

- `var`, `svar` — built-in; supports Cholesky and parametric short-run
  and long-run restrictions.
- Sign restrictions require user-written code or calling out to R.
- `sspace` for state-space models.

**Python:**

- `statsmodels.tsa.vector_ar` — VAR estimation and Cholesky SVAR.
- Sign-restricted SVARs require custom implementation.
- `statsmodels.tsa.statespace` for state-space models; reasonably
  mature.

Markus's default for SVAR work is R, using `VARsignR` for sign
restrictions and direct implementation for proxy SVARs based on
replication code from the relevant papers. Python is adequate for
reduced-form VAR work but the SVAR-specific packages are less mature.

---

## What this note doesn't cover, and where to find it

**LP-vs-VAR equivalence and LP-IV.** The Plagborg-Møller–Wolf (2021)
result and the LP side of the proxy comparison are in
[[`local-projections.md`]].

**Non-stationary VARs and cointegration.** When the variables are I(1)
and cointegrated, the correct specification is a VECM (Vector Error
Correction Model), not a VAR in levels. Unit root testing, the
Engle-Granger procedure, the Johansen procedure, and the VECM
structure are in [[`non-stationary-time-series.md`]]. The identification
content of this note applies to VECMs as well, but the non-stationarity
layer must be handled first.

**FAVAR and large-data factor models.** Bernanke-Boivin-Eliasz (2005)
and the broader literature on using factor models to incorporate
large-N information into structural VARs. Worth its own note when a
project demands it.

**Frequency-domain methods.** Spectral decompositions,
frequency-specific variance decompositions, band-pass filtered VARs.
Not covered here.

---

## Cross-references

- [[`10_Methods/Econometrics/local-projections.md`]] — the LP side of
  the proxy SVAR comparison. Plagborg-Møller–Wolf equivalence: the
  proxy SVAR and LP-IV using the same instrument identify the same
  structural IRF in population. The LATE framing and weak-instrument
  machinery from the LP-IV discussion apply directly here.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] — the full IV
  machinery (effective $F$, Anderson-Rubin intervals,
  Conley-Hansen-Rossi plausibly-exogenous bounds) applies when the
  proxy is the instrument. Import that machinery; do not rederive it.
- [[`10_Methods/Econometrics/non-stationary-time-series.md`]] — the VECM
  is the correct VAR specification for cointegrated I(1) variables.
  Read that file before specifying any VAR in levels with trending
  macro series.
- [[`20_Math/stochastic-processes.md`]] — reduced-form VAR, stationarity,
  Wold decomposition, companion-matrix stability, state-space and
  ARMA representations. The structural identification layer built here
  rests on those foundations.
- [[`00_Identity/Principles.md`]] — sections on identification and
  uncertainty. The general posture: identification assumptions must be
  argued, their robustness explored, their limits acknowledged.
- [[`50_Workflows/run-a-regression-properly.md`]] — SVARs are multivariate
  regressions and the identification stage of the workflow applies.
  Stage 4 (specify identification, not just the equation) is where
  the content of this note lives in practice.

---

*Last updated: 2026-04. The SVAR literature has consolidated around
proxy SVARs and the Arias-Rubio-Waggoner-Zha sign-restrictions
framework as the two main modern approaches. The connection between
proxy SVARs and LP-IV — formally established by Stock-Watson (2018)
and Plagborg-Møller–Wolf (2021) — is now the organizing framework for
the external-instruments literature. The TVP-VAR literature continues
to develop but the Primiceri (2005) framework remains the reference
point.*
