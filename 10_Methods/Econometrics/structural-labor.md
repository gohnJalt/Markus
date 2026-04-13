---
tags: [econometrics, labor, structural, akm, firm-effects, inequality]
related: [modern-toolkit-references, distributional-methods, panel-methods, instrumental-variables, run-a-regression-properly, Principles, linear-algebra, dynamic-programming]
---

# Structural Labor: AKM and Earnings Decompositions

This is the working note on the Abowd-Kramarz-Margolis (1999)
model and the Kline-Saggio-Sølvsten (2020) leave-out estimator.
It assumes you know panel fixed-effects regression —
`panel-methods.md` covers the geometric foundations (demeaning as
projection, FWL, HDFE) that underlie AKM estimation. What this
note adds is the specific structure of the two-way FE model for
linked employer-employee data, the statistical problem that
makes naive variance decompositions wrong (limited mobility
bias), and the correction that makes them right. It also covers
what AKM does and does not tell you about firm pay premia and
earnings inequality — because the method is frequently invoked
to say more than it can.

The note has a bias toward the variance decomposition application,
which is the dominant use in the inequality literature. The
monopsony and wage-posting connections are treated as motivating
context rather than developed in full; they have their own
literature. Read linearly the first time; jump to the decision
tree and failure modes as needed after that.

---

## What AKM is actually estimating

The AKM model decomposes log earnings into three additive
components:

$$w_{it} = \alpha_i + \psi_{J(i,t)} + X_{it}'\beta + \varepsilon_{it},$$

where $w_{it}$ is log earnings of worker $i$ at time $t$,
$\alpha_i$ is a **worker fixed effect**, $\psi_{J(i,t)}$ is a
**firm fixed effect** for the firm $j = J(i,t)$ that employs
worker $i$ at time $t$, $X_{it}$ are time-varying observables
(experience, tenure), and $\varepsilon_{it}$ is an idiosyncratic
residual.

The estimands are the worker effects $\alpha_i$ and firm effects
$\psi_j$, and the components of the wage variance they generate.

**Worker effects** capture the portable component of a worker's
wages: skills, education, ability, and anything else that travels
with the worker across employers. A high $\alpha_i$ means the
worker commands high wages everywhere they work, conditioning on
the firm they're at and their observables.

**Firm effects** capture the pay premium or penalty of a
particular employer: the amount the firm pays above or below the
market rate for workers of given quality. A high $\psi_j$ means
the firm pays well regardless of which workers it hires. The firm
effect in AKM is the counterpart of firm-specific wage premia in
the search and wage-posting literature.

The variance decomposition of wages follows directly:

$$\text{Var}(w) = \text{Var}(\alpha) + \text{Var}(\psi) + 2\,\text{Cov}(\alpha, \psi) + \text{Var}(X'\beta) + \text{Var}(\varepsilon) + \text{cross-terms}.$$

Three terms drive the empirical literature: $\text{Var}(\alpha)$
measures how much wage inequality comes from worker heterogeneity,
$\text{Var}(\psi)$ measures how much comes from firm
heterogeneity in pay policies, and $\text{Cov}(\alpha, \psi)$
measures assortative matching — whether high-ability workers tend
to work at high-paying firms. These three quantities are the
output of most AKM analyses in the inequality literature, and all
three are biased under naive estimation. The bias and its
correction are the centerpiece of this note.

---

## Identification: connected sets and mobility

### The connected set

AKM is a two-way fixed-effects regression. Identification
requires that worker and firm effects can be separately estimated
— which in turn requires mobility: workers must move between
firms for their effects to be distinguishable from the firms they
work at.

Formally, identification requires that the bipartite graph of
workers and firms — where an edge connects worker $i$ to firm $j$
if $i$ ever worked at $j$ — is **connected**. In a connected
graph, every firm can be reached from every other firm through a
chain of worker moves. Absent connectivity, worker and firm
effects can only be estimated up to a group-specific constant:
firms in different connected components are not identified relative
to each other.

In practice, large administrative panels almost always have a
**largest connected set (LCS)** that contains the great majority
of observations, with a few isolated components of firms linked to
only a single worker. The standard practice is to restrict
estimation to the LCS. Observations not in the LCS are excluded
from the regression, which introduces a sample-selection step
that should be reported.

### Identification of firm effects from movers

Given the connected set, the firm effects are identified by
workers who move between firms. When worker $i$ moves from firm
$j$ to firm $k$, the difference in their wages (net of
time-varying observables and a common time trend) identifies
$\psi_k - \psi_j$. Chaining many such differences across the
connected set identifies all firm effects up to a common
normalization.

The crucial implication: the quality of identification depends
on the number of movers. Firms linked by many movers are
well-identified; firms linked by a single mover (or connected
only through long chains) are poorly identified. This is the
source of the limited mobility bias, the subject of the next
section.

### The exogeneity assumption

The AKM model assumes $E[\varepsilon_{it} | \alpha_i, \psi_{J(i,t)}, X_{it}] = 0$.
In words: workers' mobility across firms is uncorrelated with the
idiosyncratic error term. This is **exogenous mobility** — workers
don't move because of transitory shocks to their wages that are
independent of both their portable skill and the firm's pay
policy.

Exogenous mobility rules out, for instance, workers moving to
new firms precisely when they are having a bad year at their
current firm (negative $\varepsilon_{it}$ precipitating a move).
It also rules out workers with high unobserved match quality
staying at firms longer than others with lower match quality —
because then the residual $\varepsilon_{it}$ is correlated with
tenure at a firm and hence with firm identity $J(i,t)$. The
"load-bearing assumption, honestly" section returns to this.

---

## The limited mobility bias problem

### Where the bias comes from

Suppose we estimate $\hat\alpha_i$ and $\hat\psi_j$ by OLS (the
within-worker-and-firm estimator, equivalent to absorbing two-way
FEs), and then compute the sample variance $\widehat{\text{Var}}(\hat\psi)
= \frac{1}{J}\sum_j \hat\psi_j^2$ as an estimate of
$\text{Var}(\psi)$.

This is biased upward. The OLS estimator overfits: the estimated
firm effect $\hat\psi_j$ absorbs not only the true firm premium
$\psi_j$ but also sampling noise from the movers who identified
it. When only a few workers connect firm $j$ to the rest of the
graph, the sampling noise is large, and $\hat\psi_j$ is noisy.
The variance of $\hat\psi_j$ exceeds the variance of $\psi_j$ by
the noise, so $\widehat{\text{Var}}(\hat\psi)$ is too large.

Andrews et al. (2008) showed this analytically. The bias is:

$$E[\widehat{\text{Var}}(\hat\psi)] = \text{Var}(\psi) + \frac{\sigma^2}{N} \sum_j \ell_{jj},$$

where $\ell_{jj}$ is the sum of **leverages** (diagonal elements
of the hat matrix $P = X(X'X)^{-1}X'$) for observations at firm
$j$, and $\sigma^2 = \text{Var}(\varepsilon)$ is the residual
variance. The second term is the bias: it is large when leverages
are high, which happens precisely when firms have few movers.

The same bias contaminates $\widehat{\text{Var}}(\hat\alpha)$ and
$\widehat{\text{Cov}}(\hat\alpha, \hat\psi)$. The covariance bias
can go in either direction, making it particularly treacherous:
naive AKM can understate assortative matching even when the firm
effect variance is upward-biased.

### Scale of the problem in practice

In a typical administrative panel — the German IAB (Lohnstatistik),
the Portuguese Quadros de Pessoal, Swedish RAMS — the bias in
$\text{Var}(\hat\psi)$ has been estimated at 20-50% of the
estimated value. Studies using uncorrected AKM in thin datasets
(short panels, small countries, restricted samples) can have
bias well above 50%. The correction is not a technical nicety;
it changes the substantive conclusion about how much of wage
inequality is attributable to firms.

---

## The leave-out estimator: Kline, Saggio, and Sølvsten (2020)

### The key idea

Kline, Saggio, and Sølvsten (2020) — KSS henceforth — provide
a bias-corrected estimator of variance components in the AKM
model. The key insight is:

The bias in $\widehat{\text{Var}}(\hat\psi)$ arises because the
same observations used to *estimate* $\hat\psi_j$ are also used
to *evaluate* $\hat\psi_j^2$. If, for each observation $(i,t)$,
we estimated the firm effect using *all other observations except*
$(i,t)$, then the estimated effect and the observation would be
independent, and the variance component would be unbiased.

This is the **leave-one-out** principle applied to a two-way FE
setting.

### The estimator

Define $\hat\psi_{J(i,t)}^{-it}$ as the estimated firm effect
for firm $J(i,t)$ obtained from a regression that leaves out
observation $(i,t)$. The KSS estimator of $\text{Var}(\psi)$ is:

$$\widehat{\text{Var}}_{KSS}(\psi) = \frac{1}{N}\sum_{it} \hat\psi_{J(i,t)} \cdot \hat\psi_{J(i,t)}^{-it}.$$

This is a quadratic form in the estimated effects, where one
instance uses the full-sample estimate and the other uses the
leave-one-out estimate. Under exogenous mobility:

$$E\!\left[\widehat{\text{Var}}_{KSS}(\psi)\right] = \text{Var}(\psi),$$

with no bias regardless of sample size or mobility rates.

### Computing leave-one-out estimates efficiently

Computing $\hat\psi_{J(i,t)}^{-it}$ by literally re-running the
regression $N$ times is infeasible. The key shortcut:

The Sherman-Morrison-Woodbury identity gives the leave-one-out
fitted values without refitting:

$$\hat\psi_{J(i,t)}^{-it} = \hat\psi_{J(i,t)} - \frac{\hat\varepsilon_{it} \cdot \ell_{it}}{1 - \ell_{it}},$$

where $\hat\varepsilon_{it} = w_{it} - \hat\alpha_i - \hat\psi_{J(i,t)} - X_{it}'\hat\beta$
is the OLS residual and $\ell_{it}$ is the leverage of observation
$(i,t)$ in the two-way FE regression.

The formula reduces the KSS estimator to: (1) estimate the full
AKM by OLS, (2) compute the leverages $\ell_{it}$ for all
observations, (3) compute the bias-corrected variance components
as a function of the OLS estimates, residuals, and leverages. No
leave-one-out regressions are actually run.

### Computing the leverages: the hard step

The bottleneck is computing the leverages $\ell_{it} = [P]_{it,it}$
where $P = X(X'X)^{-1}X'$ and $X$ is the design matrix of the
two-way FE model. The design matrix for even a moderate
administrative panel has millions of rows and hundreds of thousands
of columns (one per worker, one per firm), so direct construction
of $P$ is infeasible.

KSS show that the diagonal of $P$ can be estimated by **random
projection**: a randomized approximation based on the Johnson-
Lindenstrauss lemma that estimates $\ell_{it}$ with high precision
using $B$ random projections of the design matrix, at cost
$O(B \cdot \text{nnz}(X))$ where $\text{nnz}(X)$ is the number
of non-zeros. In practice, $B = 200$-$500$ random vectors give
adequate precision; the `LeaveOutTwoWay` package (Matlab) and
`pytwoway` (Python) implement this. The computation is the
dominant cost of the KSS correction — more expensive than fitting
the full AKM, but feasible for administrative panels of reasonable
size.

The connection to `linear-algebra.md`: this is exactly the hat
matrix diagonal problem from §2.4, now at the scale of a large
sparse two-way FE system. The randomized algorithms for computing
$\text{tr}(A)$ and diagonal entries of $A$ for large matrices are
the same tools.

### What KSS corrects

KSS gives unbiased estimators of:
- $\text{Var}(\psi)$ — the variance of firm effects
- $\text{Var}(\alpha)$ — the variance of worker effects
- $\text{Cov}(\alpha, \psi)$ — the covariance (assortative matching)

It also gives standard errors for these variance components, via
a heteroskedasticity-robust variance formula for quadratic forms
in linear estimators. This is non-trivial: the variance of a
variance component is a quartic in the residuals, and the standard
sandwich formula doesn't directly apply. KSS derive the correct
sandwich estimator for this setting.

---

## What AKM does and doesn't tell you about firm pay premia

### What AKM tells you

**The contribution of firms to wage inequality.** $\text{Var}(\psi)$
is a direct measure of how much of wage variance is explained by
employer heterogeneity in pay, net of worker quality. If firms
paid all workers identically (conditional on skill), $\psi_j$
would be constant and $\text{Var}(\psi) = 0$.

**Assortative matching.** $\text{Cov}(\alpha, \psi) > 0$ means
high-skill workers work at high-paying firms — positive
assortativity. This matters for inequality because it means the
gains from working at good firms are concentrated among workers
who would earn well anyway. The rise in $\text{Cov}(\alpha, \psi)$
over time is a candidate explanation for growing wage inequality
even when $\text{Var}(\alpha)$ and $\text{Var}(\psi)$ move little.
Song, Price, Guvenen, Bloom, and von Wachter (2019) find that
rising assortative matching is the main driver of the rise in
between-firm wage inequality in the United States post-1980.

**Portable vs. firm-specific earnings.** A worker's $\alpha_i$
is the component of their wages they take with them when they
move. The ratio $\text{Var}(\alpha)/\text{Var}(w)$ measures how
much of wage inequality is about who workers are, not where they
work.

### What AKM doesn't tell you

**Whether firm effects are rents or competitive returns.** A
firm effect $\psi_j > 0$ means the firm pays above the market
rate for workers of given measured quality. This is consistent
with (a) the firm having market power over workers and sharing
rents (monopsony + rent-sharing), (b) the firm using unobserved
capital or technology that is complementary to worker skill and
the market price reflects this, or (c) compensating differentials
being unmeasured and the "premium" compensating for unpleasant
working conditions. AKM cannot distinguish these.

**Causal effects of firm characteristics.** The firm effect
$\psi_j$ is a residual after absorbing worker heterogeneity and
observables. It is not the causal effect of, say, firm
productivity or industry on wages; it is the conditional mean
wage premium at that firm. Turning firm effects into causal
estimates of specific firm characteristics requires additional
variation — an event study using firm ownership changes, foreign
acquisition, or policy-induced productivity shocks.

**Within-firm wage dispersion.** AKM assumes a single firm
effect: all workers at firm $j$ receive the same firm premium
$\psi_j$. In reality, the within-firm distribution of wages
matters and can be systematically different across firms.
Bonhomme, Lamadon, and Manresa (2019) propose a "grouped"
approach that classifies firms into a small number of wage-policy
types rather than estimating one effect per firm; this allows
within-type dispersion to be characterized but at the cost of
imposing a discrete mixture structure.

**The direction of sorting.** $\text{Cov}(\alpha, \psi) > 0$
is consistent with high-ability workers seeking out high-paying
firms (worker-driven) or high-paying firms recruiting high-ability
workers (firm-driven). The AKM framework is silent on the
mechanism of assortative matching. Understanding direction
requires a structural model of search and matching.

---

## Wage variance decompositions and the inequality literature

The AKM framework connects directly to `distributional-methods.md`
but at a different level of the analysis. Where distributional
methods ask "what is the effect of $X$ on the distribution of
wages in the population," AKM asks "how much of the variance of
wages comes from worker heterogeneity, from firm heterogeneity,
and from the matching between workers and firms."

The empirical findings, summarized:

**Germany** (Card, Heining, and Kline 2013): rising variance
of firm effects between 1985 and 2009 accounts for a substantial
share of rising wage inequality, particularly in the lower half
of the distribution. Assortative matching also increases.

**Portugal** (Card, Cardoso, Heining, and Kline 2018): high-ability
workers earn higher wages primarily because they sort to high-paying
firms, not because they earn more at any given firm type. Firm
effects explain about 20% of wage variance; assortative matching
explains another substantial share.

**United States** (Song, Price, Guvenen, Bloom, and von Wachter 2019):
using matched employer-employee data from SSA records, the rise
in wage inequality is almost entirely between-firm. And most of
the between-firm rise is explained by rising sorting — workers
increasingly concentrated in firms that pay workers of their type
well — rather than rising dispersion of firm effects per se.

These findings matter for the distribution agenda because they
shift attention from within-firm wage setting (which unionization,
minimum wages, and pay equity policies directly address) to
between-firm sorting (which is harder to address directly and more
closely linked to market structure and concentration).

---

## Decision tree

Use this when a project involves linked employer-employee data or
questions about firm pay premia.

**Q1. Is the question about wage variance decomposition or about
a specific firm characteristic?**

- **Variance decomposition** (how much of inequality is firms
  vs. workers vs. sorting?): proceed with AKM + KSS correction.
  Go to Q2.
- **Specific characteristic** (what is the effect of productivity,
  foreign ownership, or market power on wages?): the AKM firm
  effect is not the right estimand. Use an event study or IV
  design with the characteristic as the treatment. The AKM can
  be a robustness check, not the primary specification.

**Q2. Is the data linked employer-employee panel with worker
mobility?**

- **Yes**: AKM applies. Go to Q3.
- **No** (cross-section, employer-only panel, or employee-only
  panel): AKM is not identified. Use OB decomposition
  (`distributional-methods.md`, §4.1) for group-level gaps, or
  industry-level wage premiums from employer-only panel data.

**Q3. How many movers are there per firm?**

- **Many movers** (large administrative panel, long time period):
  the KSS correction is a refinement. Uncorrected AKM is still
  substantially biased; use KSS for any variance component
  analysis.
- **Few movers** (short panel, small country, restricted sample):
  the KSS correction is mandatory. Uncorrected estimates of
  $\text{Var}(\psi)$ can be more than twice the true value. The
  KSS package will also provide the "leave-one-out connected set"
  — a subsample where identification is particularly clean.

**Q4. What is the primary estimand?**

- **Variance components** ($\text{Var}(\alpha)$, $\text{Var}(\psi)$,
  $\text{Cov}(\alpha,\psi)$): use KSS with the full correction
  and the KSS standard errors.
- **Firm-level wage premia for specific firms** (e.g., comparing
  high-HHI to low-HHI markets): the AKM firm effects are the
  right starting object, but translate the comparison into a
  regression of $\hat\psi_j$ on market characteristics, with
  SEs that account for estimation error in the first stage.
- **Worker mobility and wage growth**: the AKM framework gives
  $\hat\alpha_i$ as the portable component; regressing wage
  growth around job moves on $\hat\psi_{new} - \hat\psi_{old}$
  is a standard exercise, with Lachowska, Mas, and Woodbury
  (2022) as a recent reference.

**Q5. Is sorting the central question?**

- **Yes**: Report $\text{Cov}(\alpha, \psi)$ as the primary
  quantity with KSS standard errors. Supplement with a "worker
  quality by firm quartile" table — sort firms into quartiles by
  $\hat\psi_j$ and report the mean $\hat\alpha_i$ within each
  quartile; the gradient shows assortative matching visually.
  Test whether the gradient has changed over time if the
  inequality question is about trends.
- **No**: Variance components alone are sufficient. Report both
  $\text{Var}(\psi)/\text{Var}(w)$ and $\text{Var}(\alpha)/\text{Var}(w)$
  as shares of total wage variance.

**When in doubt**: Use KSS. Report the LCS sample and the share
of observations dropped. Report all three variance components and
their KSS standard errors. Supplement with a worker-quality-by-
firm-quartile table.

---

## The exogenous mobility assumption, honestly

The AKM model requires $E[\varepsilon_{it} | \alpha_i, \psi_{J(i,t)}, X_{it}] = 0$.

This is the load-bearing assumption. What it rules out:

**Transitory wage shocks driving mobility.** If workers move to
new firms when their idiosyncratic productivity at the current
firm is temporarily low (a bad match, a conflict with a manager),
then $\varepsilon_{it}$ before a move is negative and $\varepsilon_{it}$
after a move is positive, but this has nothing to do with the
firm effect. The firm effect the mover *arrives* at is inflated
by the mean reversion in $\varepsilon$. This violates exogenous
mobility.

**Match-specific effects.** If some worker-firm pairs are
especially productive — beyond what the additive $\alpha_i +
\psi_j$ structure captures — then workers sort on their
match-specific component, and $\varepsilon_{it}$ is correlated
with which firm the worker is at. Woodcock (2008) and Bagger and
Lentz (2019) discuss tests for match effects; the finding is that
match effects are typically small but nonzero in high-frequency
data.

**Cyclical mobility patterns.** If high-ability workers are
more likely to move in good aggregate conditions (when firm-to-
firm poaching is more active), and aggregate conditions shift
the mean of $\varepsilon_{it}$, then sorting is correlated with
the error through aggregate time effects. This is typically
handled by including time fixed effects and checking robustness
to time-trend controls.

**Testing exogenous mobility.** The cleanest available test is
Abowd, Lengermann, and McKinney (2003): check whether the mean
residual in the year before and after a job move is zero
("leave-taking residual" and "arrival residual" tests). If workers
who move tend to have positive residuals at origin (suggesting
they are doing well when they leave) or negative residuals at
destination (suggesting the firm looks bad on arrival), exogenous
mobility is suspect.

Lachowska, Mas, and Woodbury (2022) implement a careful version
of this test using Washington state administrative data and find
broadly supportive evidence for exogenous mobility, with modest
evidence of match effects that are too small to overturn the AKM
variance decomposition.

**The honest position**: exogenous mobility is an identifying
assumption, not a testable fact. The tests available are indirect.
Report the pre-move and post-move residual patterns as a
diagnostic. If the data show large systematic patterns in
$\hat\varepsilon$ around job moves, flag this explicitly and
consider whether the Bonhomme-Lamadon-Manresa grouped approach
(which is more robust to match heterogeneity) is more
appropriate.

---

## Common failure modes

**Using uncorrected AKM for variance components.** The bias in
$\widehat{\text{Var}}(\hat\psi)$ from OLS is large and
systematically upward. Reporting variance components from OLS
AKM without acknowledging the bias is a methodological error,
not a conservative simplification. The KSS correction exists,
the software exists, and the computation is feasible. Use it.

**Conflating firm effects with causal estimates of firm
characteristics.** Regressing the AKM firm effects on firm-level
observables (productivity, size, industry) and calling the result
the "causal effect of productivity on wages" is wrong. The firm
effects are reduced-form summaries of firm pay policy; they absorb
all unmeasured differences between firms. A finding that $\hat\psi_j$
correlates with value-added per worker is consistent with a causal
effect, but also with high-TFP firms attracting better workers
than AKM can control for, or with high-TFP firms operating in
tight local labor markets.

**Misinterpreting the "unexplained" as discrimination.** The same
warning from `distributional-methods.md` applies here: the share
of the gender or ethnic wage gap attributable to firm effects
(whether high-wage firms underemploy minority workers) vs. worker
effects (minority workers have lower $\hat\alpha_i$) vs. assortative
matching is informative about mechanisms, but none of the components
can be interpreted as discrimination without additional identification.

**Ignoring the leave-one-out connected set (LOCS).** The KSS
estimator is most reliable in the "leave-one-out connected set" —
the subset of observations where removing any single observation
leaves the firm still connected to the LCS. Firms identified by
a single mover are not in the LOCS; their firm effects are
poorly identified and heavily leveraged ($\ell_{it}$ near 1).
The KSS software constructs the LOCS automatically; estimation
should typically be on the LOCS, with a note on the share of
observations dropped.

**Treating mobility as exogenous without the mobility tests.**
Running AKM without checking the pre-move and post-move residual
patterns is omitting the primary available diagnostic for the
identifying assumption. The tests are simple and add credibility;
skip them only when data restrictions prevent computation.

**Ignoring the normalization.** AKM firm and worker effects are
identified only up to a normalization: if you add a constant to
all $\alpha_i$ and subtract the same constant from all $\psi_j$,
the fit is unchanged. The standard normalization sets the mean
firm effect to zero; the mean worker effect then equals the
grand mean of wages. Comparisons of firm effects across time
periods or countries require consistent normalizations; failure
to normalize consistently produces spurious changes in
$\text{Cov}(\alpha,\psi)$ across samples.

---

## What to report

When Markus reports an AKM analysis, the deliverable includes:

1. **The sample and connected set.** Report the initial sample
   size, the LCS sample, and the LOCS sample (if KSS is used).
   State the share of workers and firms dropped and why. If the
   LOCS restriction is material (drops more than 10% of
   observations), flag this and discuss whether the dropped
   firms are systematically different.
2. **The variance components with KSS standard errors.** Report
   $\text{Var}(\hat\alpha)/\text{Var}(w)$, $\text{Var}(\hat\psi)/\text{Var}(w)$,
   $2\,\text{Cov}(\hat\alpha,\hat\psi)/\text{Var}(w)$, and
   $\text{Var}(\hat\varepsilon)/\text{Var}(w)$, all as shares of
   total wage variance summing to one. Each with the KSS standard
   error. Note the uncorrected OLS values alongside to show the
   magnitude of the bias correction.
3. **The worker-quality-by-firm-quartile table.** Sort firms
   into quartiles by $\hat\psi_j$. Report the mean $\hat\alpha_i$
   within each quartile. This table is the most direct
   visualization of assortative matching.
4. **The mobility residual diagnostic.** Mean residual in the
   year before and after a job move. This is the primary
   diagnostic for exogenous mobility and should be standard.
5. **The exogenous mobility assumption stated and defended.**
   Name the assumption. Report what the residual diagnostic
   shows. If there is evidence against it, note which direction
   the bias goes.
6. **The share of wage gap attributable to each component** (for
   inequality studies). The standard decomposition of
   $\bar w_{group\ A} - \bar w_{group\ B}$ into worker-effect
   differences, firm-effect differences, and matching differences,
   analogous to OB at the mean.

---

## Implementation, by language

**Matlab** is the current primary environment for AKM with the
KSS correction, because the reference implementation lives there.

`LeaveOutTwoWay` (Kline, Saggio, Sølvsten) is the canonical
implementation of the KSS estimator. It computes the LCS and
LOCS, estimates the full AKM by HDFE, computes leverages via
random projections, and returns variance components with standard
errors. The code is documented in the KSS (2020) replication
package. This is the right implementation to use for the variance
decomposition; the Matlab dependency is a cost, but the
correctness of the statistical procedure is more important than
language preference.

**Python** has `pytwoway` (Bonhomme, Lamadon, Manresa and
collaborators), which implements both the standard AKM and the
grouped approach. It also includes a version of the KSS
correction. The package is actively maintained and is the right
Python entry point. Install via `pip install pytwoway`; the
documentation includes worked examples on simulated data that
reproduce the KSS results.

```python
import pytwoway as tw

# Load the data
adata = tw.BipartiteDataFrame(df,
    i_col='worker_id', j_col='firm_id',
    t_col='year', y_col='log_wage')
adata = adata.clean_data()   # constructs LCS

# Estimate AKM + KSS
fe_params = tw.FEControlParams({'featurize': False})
cre_params = tw.CREControlParams({'order': 1})
ak_estimator = tw.AKMEstimator(adata, fe_params=fe_params)
ak_estimator.fit(compute_kss=True)
ak_estimator.summary  # variance components + KSS SEs
```

**R** does not have a maintained KSS implementation. The `lfe`
package (Gaure) handles HDFE estimation and can compute some
leverage-based quantities, but lacks the full KSS bias correction.
For the variance decomposition with KSS correction, the right
path is `pytwoway` in Python or `LeaveOutTwoWay` in Matlab,
optionally calling from R via `reticulate` or `matlab.call`.
For exploratory AKM (without the bias correction), `lfe` or
`fixest` in R can estimate the two-way FE and recover the firm
and worker effects via `getfe()`.

**Stata** has `a2reg` (Ouazad) and `felsdvreg` (Abowd, Creecy,
Kramarz) for basic AKM estimation, but neither implements the
KSS correction. The practical situation is the same as R: use
the Matlab or Python implementation for any variance component
analysis.

**Markus's default**: `pytwoway` for the KSS-corrected variance
decomposition; `fixest` or `pytwoway` for the AKM firm effects
as an input to further analysis (event studies, wage gap
decompositions).

---

## What this note doesn't cover

**Structural wage-posting models.** The Burdett-Mortensen
(1998) model and the Postel-Vinay–Robin (2002) sequential
auctions model interpret firm effects as equilibrium outcomes of
strategic wage-setting in the presence of on-the-job search.
These structural models take AKM as a motivation and go further
by specifying why firm effects arise — market power, bargaining,
and the wage ladder. They are a natural extension of this note
but require their own treatment.

**Monopsony estimation beyond AKM.** Manning's (2003)
"monopsony in motion" approach uses labor supply elasticities
to firms rather than AKM. The employer concentration literature
(Azar, Marinescu, Steinbaum; Benmelech, Bergman, Kim) uses
HHI at the occupation-labor-market level. Both are empirically
productive directions but don't fit the AKM framework directly.

**The Bonhomme-Lamadon-Manresa grouped approach.** BLM (2019)
replace firm-specific effects with a discrete finite mixture of
firm "types," which partially relaxes the AKM additivity
assumption and allows for match heterogeneity within type. This
is a natural robustness check when the exogenous mobility
assumption is suspect. The `pytwoway` implementation includes
BLM.

**Minimum wages and monopsony in Turkish data.** Turkey's 2016
minimum wage increase (roughly 30% in real terms) and subsequent
episodes provide substantial variation for estimating monopsony
power and firm-level wage-setting. This would use the SGK panel
and apply the AKM framework to ask whether minimum wages cause
firm effects to compress or whether workers sort toward higher-
premium firms after the increase. This is a research project,
not covered here.

**Gender and ethnicity gaps decomposed via AKM.** Cardoso,
Guimarães, and Portugal (2016), Hirsch, Schank, and Schnabel
(2010), and others use AKM to decompose gender pay gaps into
worker effects, firm effects, and sorting. The implementation
is standard AKM + KSS on a sample stratified by gender, with a
cross-group decomposition of the wage gap. This is an
application of §5 ("What AKM tells you") rather than a separate
method; it goes in a project note when needed.

---

## Cross-references

- `10_Methods/Econometrics/panel-methods.md` — the two-way
  FE model underlying AKM is a panel fixed-effects regression;
  FWL, HDFE estimation, and the within-estimator geometry
  there are directly applicable; §3 (HDFE and the Mundlak
  representation) and §5 (strict exogeneity) are the relevant
  sections.
- `10_Methods/Econometrics/distributional-methods.md` — the
  AKM variance decomposition is a special-purpose
  distributional decomposition for linked employer-employee
  data; the OB decomposition at the mean (§4.1) and the DFL
  reweighting (§4.2) apply to group wage gaps using AKM
  components as the summary statistics.
- `10_Methods/Econometrics/instrumental-variables.md` — causal
  interpretation of firm effects requires additional variation
  beyond AKM; IV designs using firm ownership changes, industry
  shocks, or minimum wage episodes as instruments for firm
  characteristics import the IV machinery directly.
- `20_Math/linear-algebra.md` — the hat matrix diagonal
  ($\ell_{it}$ leverages) central to the KSS bias correction
  is the projection matrix diagonal discussed in §2.4;
  randomized algorithms for computing the diagonal of large
  sparse matrices are the computational backbone of the KSS
  estimator.
- `20_Math/dynamic-programming.md` — search and matching models
  (Burdett-Mortensen, Postel-Vinay–Robin) that interpret AKM
  firm effects as equilibrium wage-posting outcomes are MDPs;
  the worker's value function satisfies a Bellman equation over
  employment states.
- `30_Data/AI-accessible-data-sources.md` — SGK administrative
  data for Turkey (linked employer-employee panel); German IAB
  Lohnstatistik, Portuguese Quadros de Pessoal, and Swedish
  RAMS are the canonical international datasets for AKM
  research.
- `50_Workflows/run-a-regression-properly.md` — Stage 4
  (identification) for an AKM analysis requires stating the
  exogenous mobility assumption explicitly and reporting the
  mobility residual diagnostic; Stage 7 (robustness) should
  include the LOCS sensitivity check.

---

*Last updated: 2026-04-12. Covers AKM through Abowd-Kramarz-
Margolis (1999) and the Andrews et al. (2008) bias diagnosis;
the KSS leave-out estimator through Kline-Saggio-Sølvsten (2020);
the wage variance literature through Song et al. (2019) and
Card et al. (2018). The Bonhomme-Lamadon-Manresa grouped
approach and structural wage-posting models are deferred.
Minimum wage and monopsony applications for Turkish SGK data
are noted but not developed here.*
