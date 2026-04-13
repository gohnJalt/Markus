---
tags: [econometrics, causal-inference, machine-learning, high-dimensional, dml]
related: [modern-toolkit-references, instrumental-variables, difference-in-differences, distributional-methods, run-a-regression-properly, Principles, functional-analysis, linear-algebra, probability]
---

# Double/Debiased Machine Learning

This note assumes you know what a causal parameter is and why
identification is the hard part of estimating one. It assumes
familiarity with the Frisch-Waugh-Lovell theorem from
`linear-algebra.md` (§2.3), with influence functions and
semiparametric efficiency bounds from `functional-analysis.md`
(§9), and with the asymptotic theory in `probability.md`. What
it adds is the treatment of a specific problem: estimating a
low-dimensional causal parameter $\theta$ when the confounding
structure involves many covariates or unknown functional forms,
and naive plug-in of a machine learning estimator for the
confounding produces badly biased inference on $\theta$.

The central message from Chernozhukov, Chetverikov, Demirer,
Duflo, Hansen, Newey, and Robins (2018) — the founding paper,
cited as CCDDHNR throughout — is that two ingredients solve this
problem: **Neyman orthogonality** and **cross-fitting**. Together
they allow machine learning nuisance estimators to be used in
first-stage estimation without corrupting asymptotic inference
on $\theta$. Neither ingredient relaxes the identification
assumption. DML does not replace the need for unconfoundedness,
a valid instrument, or another credible causal design. It solves
the problem of **regularization bias** given identification —
a finite-sample problem, not a causal one.

This is the commitment that threads through the whole note:
**DML is a bias-correction device, not an identification device.**
The moment you reach for DML to solve an identification problem
you don't have a solution to, you are laundering a causal claim
through ML instead of through the identification strategy.
Markus does not do this and will say so.

---

## What DML is actually estimating

Two canonical models.

**The partially linear regression (PLR).** The data-generating
process is:

$$Y = D\theta_0 + g_0(W) + \varepsilon, \qquad
\mathbb{E}[\varepsilon | D, W] = 0$$
$$D = m_0(W) + V, \qquad \mathbb{E}[V | W] = 0$$

where $D$ is the treatment (scalar; possibly continuous),
$W \in \mathbb{R}^p$ is a vector of controls (possibly
high-dimensional), $g_0(W)$ is an unknown nuisance function
(how $W$ affects $Y$), and $m_0(W) = \mathbb{E}[D | W]$ is the
propensity/treatment assignment conditional expectation. The
parameter of interest is $\theta_0$ — the average causal effect
of $D$ on $Y$, conditional on $W$.

The estimand is the familiar partial effect, but now in a setting
where the functional form of the confounding is unknown and $p$
may exceed $n$. This is the high-dimensional analog of the
standard partially linear model from Robinson (1988).

**The interactive regression model (IRM).** For a binary
treatment $D \in \{0, 1\}$:

$$Y = g_0(D, W) + \varepsilon, \qquad
\mathbb{E}[\varepsilon | D, W] = 0$$

Here the full conditional mean $g_0(D, W)$ is nonparametric,
and the estimand is either the average treatment effect
$\text{ATE} = \mathbb{E}[g_0(1,W) - g_0(0,W)]$ or the average
treatment effect on the treated $\text{ATT}$. The IRM is more
general than the PLR (it allows $D$ to interact with $W$ in
determining $Y$) but also more demanding to estimate.

**What these models assume.** Both models assume
**unconfoundedness** (conditional independence of $D$ and
$\varepsilon$ given $W$). In the PLR, this means that all
variables $W$ that simultaneously affect $D$ and $Y$ are
included. In the IRM, the same conditional independence holds:
$Y(d) \perp D \mid W$ for both potential outcomes. This is the
selection-on-observables assumption. DML does not weaken it. If
unconfoundedness fails because an important confounder is
omitted, the DML estimate of $\theta_0$ is biased regardless of
how well-fitted the nuisance functions are.

---

## The regularization bias problem

Why not just estimate $g_0$ and $m_0$ with your favorite ML
method, plug the fitted values into the moment condition, and
proceed? Because regularization introduces bias that is too large
for valid $\sqrt{n}$-inference.

The standard OLS moment condition for $\theta_0$ (if $g_0$ were
known) is:
$$\mathbb{E}[(Y - D\theta_0 - g_0(W))V] = 0$$

Replacing $g_0$ with its ML estimate $\hat g$ introduces error:
$$\sqrt{n}(\hat\theta_\text{naive} - \theta_0) =
\frac{1}{\sqrt{n}} \sum_i V_i \varepsilon_i / \mathbb{E}[V_i^2]
+ \underbrace{\frac{1}{\sqrt{n}} \sum_i V_i
(\hat g(W_i) - g_0(W_i)) / \mathbb{E}[V_i^2]}_{\text{regularization bias}}$$

The regularization bias term is of order $\sqrt{n} \cdot
\|\hat g - g_0\|$. For standard nonparametric ML estimators
converging at rate $n^{-2/(4+p)}$ (in a $p$-dimensional smooth
function class), this term diverges unless $p$ is small relative
to $n$. For LASSO in a sparse high-dimensional model, $\|\hat g
- g_0\|_2 = O_P((s \log p / n)^{1/2})$ where $s$ is sparsity —
still too large for $\sqrt{n}$-convergence of the naive estimator.
The bias swamps the signal.

Two ingredients eliminate the regularization bias. The first
makes the score insensitive to first-order errors in the nuisance.
The second prevents overfitting contamination when the nuisance
is estimated from the same data used for the score.

---

## Neyman orthogonality

A moment condition $\psi(W_i; \theta, \eta)$ is
**Neyman-orthogonal** at the truth $(\theta_0, \eta_0)$ if its
Gateaux derivative with respect to the nuisance $\eta$ vanishes:

$$\partial_\eta \mathbb{E}[\psi(W; \theta_0, \eta_0)]
[\eta - \eta_0] = 0 \quad \text{for all } \eta$$

This is a functional derivative condition — exactly the Gateaux
derivative in $L^2$ developed in `functional-analysis.md` (§7).
The condition says that the expected score $\mathbb{E}[\psi]$,
as a function of the nuisance, is flat at the truth: small errors
in $\eta$ produce second-order, not first-order, errors in the
moment condition.

**Why this eliminates the regularization bias.** When the score
is Neyman-orthogonal, the Taylor expansion of the moment
condition gives:

$$\sqrt{n}(\hat\theta - \theta_0) \approx
\frac{1}{\sqrt{n}} \sum_i \psi(W_i; \theta_0, \eta_0) +
O_P(n^{1/2} \|\hat\eta - \eta_0\|^2)$$

The bias from nuisance error is now of order $n^{1/2} \|\hat\eta
- \eta_0\|^2$ rather than $n^{1/2} \|\hat\eta - \eta_0\|$. If
the nuisance converges at rate $n^{-1/4}$ (the rate achievable
by many ML methods under mild conditions), the squared error is
$n^{-1/2}$, and the bias term is $O_P(1)$ — negligible at the
$\sqrt{n}$-scale needed for inference on $\theta_0$.

**The Neyman-orthogonal score for the PLR.** The score that
achieves orthogonality for $\theta_0$ in the PLR is the
**partialling-out score**:

$$\psi_\text{PLR}(W_i; \theta, \eta) = (Y_i - D_i\theta
- g(W_i))(D_i - m(W_i))$$

This subtracts both the direct confounding effect ($g$ from $Y$)
and the confounding in the treatment equation ($m$ from $D$).
The score is zero at the truth (since $\varepsilon$ and $V$ are
uncorrelated), and its derivative with respect to $g$ and $m$
at the truth vanishes — making it orthogonal. Importantly, this
score has a beautiful interpretation: it is the OLS estimating
equation after **partialling out $W$ from both $Y$ and $D$** via
their respective conditional expectation functions. This is the
nonparametric generalization of the Frisch-Waugh-Lovell theorem
(`linear-algebra.md` §2.3): in the linear case, partialling out
$W$ from $Y$ and $D$ and regressing the residuals gives the same
answer as controlling for $W$ directly; DML generalizes this to
nonparametric residuals.

**The efficient influence function connection.** The
Neyman-orthogonal score for a regular semiparametric problem is
the **efficient influence function** of the target parameter —
the element of the tangent space that achieves the semiparametric
efficiency bound (`functional-analysis.md` §9). DML estimators
that use the EIF as their score are semiparametrically efficient:
they are the best possible estimators of $\theta_0$ in a
semiparametric model, in the same sense that OLS is BLUE under
the Gauss-Markov conditions.

---

## Cross-fitting

Neyman orthogonality eliminates the first-order bias from
nuisance error, but a second problem remains: fitting the
nuisance on the same data used to evaluate the score introduces
**overfitting contamination**. Even if the nuisance estimator is
consistent, the sample correlation between the fitted nuisance
and the score residuals can be non-negligible when both are
computed from the same sample.

**K-fold cross-fitting** solves this by ensuring the nuisance
and the score are evaluated on independent data. The procedure:

1. Partition the data into $K$ folds of roughly equal size.
2. For each fold $k$: estimate $\hat g^{(-k)}$ and $\hat m^{(-k)}$ using all observations *outside* fold $k$.
3. Evaluate the score $\psi(W_i; \theta, \hat\eta^{(-k)})$ for observations $i$ in fold $k$, using the out-of-fold nuisance estimates.
4. Solve for $\hat\theta$ by averaging the score across all observations (using the appropriate out-of-fold nuisance for each).

This is sometimes called the DML2 estimator. The DML1 variant
estimates $\theta$ separately in each fold and averages the
estimates — slightly less efficient but numerically more stable
with small folds.

**Why $K = 5$ or $K = 2$?** With $K = 2$, each fold uses half
the data for nuisance estimation and half for scoring. With $K
= 5$ (the more common default), nuisance is estimated on 80%
of the data for each fold, which improves nuisance estimation
quality at the cost of more computation. For small samples
(say $n < 500$), $K = 5$ is preferred because the nuisance
estimates based on 20% of the data may be poor.

**Cross-fitting and sample size.** Cross-fitting requires enough
data in each fold to estimate the nuisance functions reliably.
With $n = 200$ and $K = 5$, each nuisance is estimated on 160
observations in a potentially high-dimensional setting. The
nuisance quality is directly capped by this within-fold sample
size. Markus's rule: **below $n \approx 300$, DML's theoretical
guarantees are fragile**. The cross-fitted nuisance estimators
may not converge at the rates needed for the orthogonality
argument to dominate. This is directly relevant for Turkish
administrative data, where many subgroup analyses and
sector-level studies operate in small-to-moderate $n$ territory.

---

## The partially linear model in practice

The DML-PLR estimator for $\theta_0$ in $Y = D\theta_0 + g_0(W)
+ \varepsilon$:

1. **Estimate $\hat g$:** regress $Y$ on $W$ using any suitable
   ML method (LASSO, random forest, gradient boosting), using
   out-of-fold predictions from cross-fitting.
2. **Estimate $\hat m$:** regress $D$ on $W$ using the same
   or different ML method, using out-of-fold predictions.
3. **Compute residuals:** $\tilde Y_i = Y_i - \hat g(W_i)$ and
   $\tilde D_i = D_i - \hat m(W_i)$.
4. **Regress $\tilde Y$ on $\tilde D$:** the OLS coefficient is
   $\hat\theta$.

This is literally Frisch-Waugh, with nonparametric residuals.
Under the conditions that $\|\hat g - g_0\|_2 \cdot \|\hat m -
m_0\|_2 = o_P(n^{-1/2})$ (the product of nuisance errors is
smaller than $n^{-1/2}$), $\hat\theta$ is $\sqrt{n}$-consistent,
asymptotically normal, and semiparametrically efficient. The
variance is estimated by the usual OLS sandwich formula on the
residualized regression, or by the `DoubleML` package directly.

**What ML method for the nuisance?** Any method that achieves
the product-rate condition. In high-dimensional sparse settings,
LASSO and post-LASSO are the theoretically cleanest (explicit
rates under restricted eigenvalue conditions). In
lower-dimensional but potentially nonlinear settings, random
forests and gradient boosting are practical choices. Neural
networks can be used when $n$ is large enough. The condition
is not "use the ML method with the lowest cross-validated MSE"
— it is "use a method whose nuisance estimation error satisfies
the rate condition." These are related but not identical.

**Check the nuisance fit.** Report the out-of-sample $R^2$ (or
MSE) for both $\hat g$ and $\hat m$ from the cross-fitted
predictions. A poor nuisance fit means the orthogonality
argument may not dominate in finite samples, and the DML
estimate inherits the bias. A first-stage $R^2$ for $\hat m$
near zero means the ML method is finding no predictable
variation in $D$ from $W$ — which either means $W$ is not
confounding (good news, but then DML is unnecessary) or the
specification is wrong.

---

## The interactive regression model

For binary treatment $D \in \{0, 1\}$, the IRM estimator
targets the ATE or ATT via the **augmented inverse propensity
weighting (AIPW)** / doubly-robust score:

$$\psi_\text{ATE}(W_i; \theta, \eta) =
\hat g(1, W_i) - \hat g(0, W_i)
+ \frac{D_i}{\hat e(W_i)}(Y_i - \hat g(1, W_i))
- \frac{1 - D_i}{1 - \hat e(W_i)}(Y_i - \hat g(0, W_i))
- \theta$$

where $\hat g(d, W) = \mathbb{E}[Y | D=d, W]$ is the conditional
outcome function and $\hat e(W) = P(D=1|W)$ is the propensity
score. The Neyman-orthogonal structure here is doubly robust:
the estimator is consistent if *either* the outcome model or the
propensity score is consistently estimated (not necessarily both).
This is the IRM analog of the doubly-robust Callaway-Sant'Anna
estimator in `difference-in-differences.md`.

The propensity score in the IRM is a nuisance function estimated
via ML classification (LASSO logistic regression, random forest
classifier). The same cross-fitting structure applies: propensity
scores and outcome models are estimated out-of-fold.

**Overlap requirement.** The IRM requires $\hat e(W_i)$ to be
bounded away from 0 and 1 — the same overlap condition as
standard propensity-score methods. ML propensity score estimates
that are very close to 0 or 1 produce extreme inverse propensity
weights and inflated variance. Trimming is standard practice;
the Crump, Hotz, Imbens, and Mitnik (2009) rule or simple
quantile trimming on the estimated propensity are common
implementations.

---

## DML with instruments

When unconfoundedness fails but a valid instrument $Z$ is
available, the DML framework extends to the partially linear
instrumental variable (PLIV) model:

$$Y = D\theta_0 + g_0(W) + \varepsilon$$
$$D = m_0(W, Z) + V, \qquad \mathbb{E}[V \cdot Z | W] = 0$$

The identification assumption shifts from $\mathbb{E}[\varepsilon
| D, W] = 0$ (unconfoundedness) to $\mathbb{E}[\varepsilon \cdot
Z | W] = 0$ (instrument exogeneity). DML handles the case where
$W$ is high-dimensional and/or the reduced forms ($g_0$, $m_0$)
are nonparametric, while the instrument $Z$ provides identification
of $\theta_0$.

The nuisance functions now include both the structural equation
controls and the first-stage relationship $D \sim Z, W$. Cross-
fitting proceeds as in the PLR case. Everything in
`instrumental-variables.md` about LATE, weak instruments, and
the Olea-Pflueger effective F applies to the PLIV first stage —
the ML part does not change the IV identification logic, it
handles the high-dimensional confounding.

---

## Conditional average treatment effects

The PLR and IRM produce *average* treatment effects. When the
research question is about *heterogeneous* effects — how the
treatment effect varies with observable characteristics $X$ —
the DML framework extends to the best linear projection of
$\tau(X) = \mathbb{E}[Y(1) - Y(0) | X]$ onto a user-supplied
basis $\phi(X)$.

The estimating equation uses the Robinson residuals $(\tilde Y,
\tilde D)$ from the PLR step and regresses $\tilde Y / \tilde D$
on $\phi(X)$, weighted by $\tilde D^2$. This is the **CATE via
DML** approach (CCDDHNR §4), and the `EconML` package implements
it as `LinearDML`.

When $\phi(X)$ is a causal forest — an adaptive partition of
the covariate space that estimates $\tau(X)$ nonparametrically
— the resulting estimator is the **generalized random forest** of
Wager and Athey (2018) / Athey, Tibshirani, and Wager (2019).
The `grf` R package implements this. Causal forests use the DML
orthogonal score as the criterion for splitting — the forest
adapts to where heterogeneity in $\tau$ is largest, rather than
where variance in $Y$ is largest. Honest forests (estimated and
evaluated on different subsamples within each leaf) provide
valid inference on $\hat\tau(x)$ point-by-point.

Markus reaches for causal forests when the research question is
explicitly about treatment effect heterogeneity and there is no
prior structure on which $X$ variables are the relevant moderators.
They are a data-driven method and should be treated as exploratory
unless the heterogeneity patterns survive external validation.

---

## The identification assumption, honestly

DML assumes conditional independence: $Y(d) \perp D \mid W$ for
all $d$. This is not something Neyman orthogonality tests,
weakens, or diagnoses. The identification assumption is identical
to what you would need for any selection-on-observables estimator.
DML's advantage is that $W$ can be high-dimensional and the
relationship of $W$ to $D$ and $Y$ can be nonparametric; it is
not that you need *fewer* controls or a *weaker* version of
conditional independence.

**What DML cannot do.** It cannot recover a causal effect when
there is selection on unobservables. It cannot discover which
variables in $W$ are the right controls; the researcher must
specify the conditioning set. It cannot test whether the
instrument in the PLIV model is valid. In all of these dimensions,
DML inherits the standard causal identification constraints.

**Sensitivity analysis for unconfoundedness.** When conditional
independence is the identifying assumption, sensitivity analysis
is the honest companion. The partial $R^2$-based framework of
Cinelli and Hazlett (2020) — originally developed for OLS
omitted-variable bias — adapts to the DML setting: how large
would an unobserved confounder need to be (measured in terms of
partial $R^2$ with $D$ and $Y$) to eliminate the estimated
effect? The `sensemakr` R package implements this. Reporting
DML estimates without a sensitivity analysis is not wrong by
default, but for applications where unconfoundedness is
contested, it is incomplete.

---

## What the nuisance estimators must satisfy

The theoretical guarantee behind DML requires:

$$\|\hat g - g_0\|_2 \cdot \|\hat m - m_0\|_2 = o_P(n^{-1/2})$$

This **product rate condition** is less demanding than requiring
either nuisance to converge at $n^{-1/2}$ individually. If both
converge at rate $n^{-1/4+\epsilon}$, the product is $n^{-1/2+2\epsilon}$,
which is $o_P(n^{-1/2})$ for any $\epsilon > 0$.

**Which ML methods satisfy this?** In a sparse linear model,
LASSO achieves $n^{-1/2+\delta}$ for some $\delta > 0$ under
restricted eigenvalue conditions. Random forests, under smoothness
conditions on $g_0$ and $m_0$, achieve $n^{-\beta/(2\beta+p)}$
for a function in a Hölder class of smoothness $\beta$ in $p$
dimensions — which satisfies the product rate when $\beta$ is
sufficiently large relative to $p$. Deep neural networks can
achieve near-parametric rates in high-dimensional settings under
composition structure, but require enough data to train reliably.

In practice: LASSO (or group LASSO, elastic net) is the default
for high-dimensional sparse settings. Random forest or gradient
boosting (XGBoost, LightGBM) is the default for lower-dimensional
nonlinear settings. The choice should be reported and the
out-of-sample nuisance fit should be diagnosed; a cross-validated
$R^2$ near zero for either nuisance is a warning sign.

**What DML is not suited for.** Very small samples where the
nuisance estimators cannot reach the required rates. Non-smooth
structural breaks in the confounding function. Discrete heavy
support on $D$ where partial effects are not well-defined.

---

## The decision tree

**Q1: Is identification from unconfoundedness (selection on observables)?**
- Yes → DML-PLR or DML-IRM depending on treatment type. Continue.
- No, valid instrument available → PLIV. The IV identification
  machinery from `instrumental-variables.md` applies. DML handles
  the high-dimensional $W$; the instrument provides identification.
- No, and no instrument → DML cannot help. Identification is the
  problem.

**Q2: What is the treatment variable?**
- Continuous treatment, linear effect plausible → PLR. The
  coefficient on $\tilde D$ in the residualized regression is
  $\theta_0$.
- Binary treatment, effect may interact with $W$ → IRM with AIPW
  score. ATE or ATT depending on the estimand.
- Binary treatment, effect is linear in $W$ → PLR or IRM with
  linear outcome model; both work but IRM is more general.

**Q3: ATE, ATT, or CATE?**
- ATE or ATT → PLR (scalar $\theta_0$) or IRM (doubly robust).
- CATE, known effect modifiers → `LinearDML` with specified
  $\phi(X)$. Test whether heterogeneity is statistically
  significant.
- CATE, unknown effect modifiers → causal forest (`grf`). Treat
  as exploratory; validate heterogeneity patterns.

**Q4: Is the sample large enough for cross-fitting?**
- $n > 500$ → standard K = 5 cross-fitting is reliable.
- $200 < n < 500$ → proceed carefully; check nuisance fit quality
  explicitly; consider $K = 2$ for less within-fold overfitting.
- $n < 200$ → DML's theoretical guarantees are fragile. Consider
  whether OLS with carefully selected controls is more appropriate.
  A well-specified OLS model with 10 covariates and $n = 150$ may
  outperform a DML specification where the nuisance ML is
  undertrained.

**The default, when in doubt:** PLR with LASSO for nuisance
functions in sparse high-dimensional settings; random forest
nuisance for lower-dimensional nonlinear settings; always report
nuisance fit quality; always run a sensitivity analysis for
unconfoundedness when the claim is causal; compare to OLS with
the same controls as a sanity check.

---

## Common failure modes

**Using DML to solve an identification problem.** The most
consequential error. DML assumes unconfoundedness; if it fails,
the DML estimate is biased in the same direction as a naive OLS.
The ML nuisance estimation does not diagnose, test, or relax the
selection-on-observables assumption. Presenting a DML estimate
as more credibly causal than an OLS estimate — when both rest on
the same unconfoundedness assumption — is obfuscation.

**Not checking the nuisance fit.** A DML estimate is only as
good as the nuisance estimates. If $\hat m(W_i)$ has an
out-of-sample $R^2$ of 0.03, the residuals $\tilde D$ are nearly
equal to $D$ itself — the DML estimator collapses to an OLS on
the full (unadjusted) data, not on the partialled-out data.
Report the out-of-fold $R^2$ for both nuisance regressions as
part of the standard output.

**Treating the standard error as complete uncertainty.** The
DML standard error reflects sampling uncertainty in $\hat\theta$
given the identification assumption. It does not reflect
uncertainty about whether unconfoundedness holds, whether the
nuisance estimators satisfy the rate conditions, or whether the
functional form of the PLR is correct. Model uncertainty is
additional. The SE should be accompanied by a sensitivity
analysis, not presented as if it were the full picture.

**Using DML when OLS would be fine.** DML is the right tool
when (a) $p$ is large relative to $n$, or (b) the confounding
functional form is genuinely unknown and nonparametric. When
$p = 15$ and $n = 5{,}000$, a well-specified OLS with appropriate
controls will perform at least as well as DML, without the
additional machinery and opacity. DML introduces complexity;
that complexity is only worth introducing when the problem
genuinely requires it.

**Running DML with cross-validated nuisance selection but
reporting a single model.** If the nuisance ML method is
selected by cross-validation (e.g., comparing LASSO, random
forest, and boosting on held-out data), the resulting DML
estimate has an additional "model selection" layer whose
uncertainty is not captured by the standard DML inference.
Report the DML estimate under each nuisance specification and
check that the main conclusion is stable across them.

**Ignoring overlap in the IRM.** Extreme propensity scores
near 0 or 1 produce extreme inverse propensity weights that
inflate variance. Always plot the propensity score distribution;
always trim and report the share of observations trimmed.

**Applying DML to panel data without the appropriate estimator.**
The standard DML is for i.i.d. cross-sectional or i.i.d.-over-
time panel data. With fixed effects and dynamic panel structures,
the cross-fitting interacts with the within-unit correlation in
ways the standard theory does not cover. The `DoubleML` package
has panel extensions; use them rather than adapting the
cross-sectional estimator.

---

## What to report

1. **The estimand.** "The average effect of X on Y, controlling
   for the high-dimensional vector W via a DML-PLR estimator, under
   the unconfoundedness assumption that W contains all relevant
   confounders."
2. **The nuisance specification.** What ML method was used for
   $\hat g$ and $\hat m$; any tuning choices (regularization
   parameter, tree depth, etc.); the cross-fitting structure (K and
   the DML1/DML2 variant).
3. **Nuisance fit quality.** Out-of-fold $R^2$ (or MSE) for both
   $\hat g$ and $\hat m$. If either is poor, flag this explicitly.
4. **The DML estimate and standard error.** With a 95% confidence
   interval based on the asymptotic normal approximation.
5. **Comparison to OLS** with the same or a reduced control set.
   If DML and OLS give similar estimates, that is evidence the
   confounding is handled similarly by both methods. Large
   differences should be explained.
6. **Sensitivity analysis for unconfoundedness.** At minimum,
   a partial $R^2$-based bound on how large an unobserved
   confounder would need to be to eliminate the estimate.
7. **For IRM**: propensity score distribution plot, trimming
   rule used, share of sample trimmed.
8. **For PLIV**: first-stage effective F (Olea-Pflueger) for the
   instrument, by fold if the cross-fitting affects the first-stage
   strength.

---

## Implementation, by language

**Python** is the primary implementation language for DML.

- `DoubleML` (`pip install doubleml`) — the reference
  implementation by CCDDHNR and collaborators. Supports PLR,
  IRM, PLIV, and DiD variants. Uses `sklearn` estimators for
  the nuisance functions; drop in LASSO (`LassoCV`), random
  forest (`RandomForestRegressor/Classifier`), or gradient
  boosting (`LGBMRegressor` from LightGBM). The `.fit()` method
  handles cross-fitting automatically; `.summary` and
  `.confint()` give inference. The cleanest path for standard
  DML applications.
- `EconML` (`pip install econml`, Microsoft Research) — broader
  scope, includes DML (`LinearDML`, `NonParamDML`), causal
  forests, meta-learners, and doubly-robust learners. More
  flexible for CATE estimation. Use when the question involves
  heterogeneous effects and you want the full EconML toolkit.
- For causal forests specifically: `econml.grf` or the R `grf`
  package via `rpy2`.

**R** has solid support:

- `DoubleML` (CRAN, same authors) — the R port of the Python
  package. Uses `mlr3` estimators for the nuisance functions;
  well-maintained. For researchers whose primary workflow is in
  R, this is the path of least resistance.
- `grf` (Wager, Athey, Tibshirani) — the reference
  implementation of generalized random forests and causal
  forests. `causal_forest()` with `tune.parameters = "all"` is
  the default entry point.
- `hdm` (Chernozhukov, Hansen, Spindler) — implements
  LASSO-based selection and post-double-selection inference;
  predates the full DML framework but is widely used for the
  special case of LASSO nuisance functions.

**Stata** has limited DML support. The `pdslasso` package
implements post-double-selection LASSO inference (the linear
special case of DML nuisance with LASSO). Full DML with
nonlinear nuisance functions requires Python or R.

Markus's default: Python `DoubleML` with `LGBMRegressor` nuisance
for moderate-dimensional nonlinear settings; `hdm` in R for
purely sparse linear settings where LASSO nuisance is theoretically
justified. Causal forests via `grf` in R.

---

## What this note doesn't cover, and where to find it

**Distributional DML.** DML applied to distributional functionals
(quantiles, Gini) as the target parameter, rather than the mean.
Chernozhukov, Newey, and Singh (2022). Briefly noted in
`distributional-methods.md`; implemented in `DoubleML`.

**DML for time series and panel data.** The cross-fitting
assumptions need modification when observations are serially
dependent. The `DoubleML` documentation covers panel extensions;
the theoretical treatment is in Chernozhukov et al. (2022 working
papers). This is an active area.

**Causal forests in depth.** The generalized random forest
framework (Wager-Athey 2018, Athey-Tibshirani-Wager 2019) is
treated briefly here; a dedicated note would cover the forest-
building criterion (honest splitting via the DML score), the
local centering, and the bootstrap-of-little-bags inference in
more depth.

**Post-double-selection.** The Belloni-Chernozhukov-Hansen (2014)
LASSO-based variable selection procedure for causal inference in
high-dimensional linear models. This is the linear special case
of DML nuisance, useful when the structural model is linear and
the question is which controls to include. `hdm` in R.

**Regression discontinuity with many covariates.** Applying DML
ideas to RDD with high-dimensional predetermined covariates is
an active area. Currently handled by including covariates in the
local polynomial regression; formal DML extensions are still
developing.

---

## Cross-references

- `10_Methods/Econometrics/instrumental-variables.md` — the PLIV
  model extends IV to the DML setting. The LATE framing, weak
  instrument diagnostics, and identification-honesty from that
  note all apply. DML handles the high-dimensional $W$; the
  instrument provides identification as always.
- `10_Methods/Econometrics/difference-in-differences.md` —
  `DoubleML` implements a DML-DiD estimator that combines
  doubly-robust DiD (Callaway-Sant'Anna) with ML propensity
  scores. The identification machinery is unchanged; DML handles
  high-dimensional baseline controls.
- `10_Methods/Econometrics/distributional-methods.md` — DML
  applied to distributional parameters (Chernozhukov-Newey-Singh
  2022) is noted there as an extension. The RIF regression approach
  in that file is the low-dimensional complement.
- `20_Math/functional-analysis.md` — Neyman-orthogonal scores are
  Gateaux derivatives of the moment condition with respect to the
  nuisance function (§7 there). The semiparametric efficiency bound
  (§9 there) is the squared $L^2$-norm of the efficient influence
  function, which is also the Neyman-orthogonal score for
  semiparametrically efficient DML estimators.
- `20_Math/linear-algebra.md` — the Frisch-Waugh-Lovell theorem
  (§2.3 there) is the linear-algebra foundation for the DML
  partialling-out logic. DML is FWL with nonparametric residuals.
- `20_Math/probability.md` — empirical process theory (Glivenko-
  Cantelli and Donsker classes, uniform LLN) underpins the
  theoretical guarantees for DML's nuisance rate conditions. The
  $o_P/O_P$ notation (§6 there) is the language in which the
  product-rate condition is stated.
- `00_Identity/Principles.md` — the section on identification and
  the section on causation. DML is a prime case where the
  methodological sophistication of the tool can obscure the
  weakness of the identification assumption. The posture from
  Principles — name the identification assumption, bound its
  violation — applies directly.
- `50_Workflows/run-a-regression-properly.md` — DML is a
  regression; Stage 4 (specify identification) still requires
  naming the unconfoundedness assumption and defending it. The
  nuisance-fit diagnostics fit in Stage 5 (diagnostics); the
  sensitivity analysis in Stage 6 (robustness).

---

*Last updated: 2026-04. The core DML framework (CCDDHNR 2018) is
stable. Active areas: DML for panel and time series data, DML for
distributional targets (CNS 2022), integration of causal forests
with DML CATE estimation. The `DoubleML` Python and R packages are
well-maintained and track the methodological developments. The
software defaults here are as of early 2026; check package
documentation for current API details.*
