---
tags: [math, probability, foundations, asymptotics]
related: [real-analysis, measure-theory, functional-analysis, optimization, stochastic-processes, modern-toolkit-references, instrumental-variables, local-projections, difference-in-differences]
---

# Probability

This file is built on top of [[`measure-theory.md`]] and
[[`functional-analysis.md`]]. It picks up where those files stop.
[[`measure-theory.md`]] gave the probability space
$(\Omega, \mathcal{F}, P)$, random variables as measurable
functions, expectation as Lebesgue integration, the five modes
of convergence and the hierarchy among them, and the
foundational properties of conditional expectation.
[[`functional-analysis.md`]] gave conditional expectation its
geometric interpretation as $L^2$ projection. This file adds
the limit theorems — the laws of large numbers, the central
limit theorems, the delta method — and the asymptotic notation
and working tools that the econometrics layer runs on.

The file is organized as a working reference for the asymptotic
arguments that appear throughout empirical economics. Every
consistency result is a law of large numbers; every asymptotic
normality result is a central limit theorem; every standard
error derivation is a delta method calculation. Understanding
these theorems at the level where you can read and check a
proof — not just cite them — is what separates an
econometrician who can audit arguments from one who cannot.

Prerequisites: [[`measure-theory.md`]] (all of it; especially §§8-9)
and [[`functional-analysis.md`]] (§§1-3, for $L^p$ completeness
and the projection theorem). [[`real-analysis.md`]] (§4, mean value
theorem) is used in the delta method proof.

The file stops before martingale theory, stochastic processes,
unit root asymptotics, and empirical process theory — all of
which go in [[`stochastic-processes.md`]]. Pointers are in §8.

---

## 1. Characteristic functions

The **characteristic function** of a random variable $X$ is

$$
\varphi_X(t) = E[e^{itX}] = E[\cos(tX)] + i\,E[\sin(tX)],
\quad t \in \mathbb{R}.
$$

This is the Fourier transform of the distribution $P_X$. Unlike
the moment generating function, the characteristic function
always exists: since $|e^{itX}| = 1$, the expectation is
finite regardless of the tail behavior of $P_X$. This
unconditional existence is why characteristic functions are the
right tool for proving the CLT — heavy-tailed distributions are
not excluded.

### 1.1 Basic properties

1. $\varphi_X(0) = 1$.
2. $|\varphi_X(t)| \leq 1$ for all $t$ (bounded).
3. $\varphi_X$ is uniformly continuous on $\mathbb{R}$.
4. $\varphi_{aX+b}(t) = e^{ibt}\,\varphi_X(at)$ (affine
   equivariance).
5. If $X \perp Y$, then $\varphi_{X+Y}(t) = \varphi_X(t)
   \varphi_Y(t)$ (product formula for independent summands).
6. If $E[|X|^k] < \infty$, then $\varphi_X$ is $k$ times
   differentiable and $\varphi_X^{(k)}(0) = i^k E[X^k]$
   (moments from derivatives).

Property 5 is the reason characteristic functions are the right
tool for proving the CLT. The characteristic function of a sum
of independent random variables is the product of the individual
characteristic functions. Products are tractable; distributions
of sums are not.

**The Gaussian characteristic function.** If
$X \sim \mathcal{N}(\mu, \sigma^2)$, then

$$
\varphi_X(t) = \exp\!\left(i\mu t - \frac{\sigma^2 t^2}{2}\right).
$$

Proof: complete the square in $\int e^{itx} \frac{1}{\sigma\sqrt{2\pi}}
e^{-(x-\mu)^2/(2\sigma^2)} dx$ and evaluate the resulting
standard Gaussian integral. For $\mu = 0$, $\sigma^2 = 1$:
$\varphi_X(t) = e^{-t^2/2}$. This is the function the CLT
proof aims to reach.

### 1.2 The inversion theorem

The characteristic function uniquely determines the
distribution.

**Theorem (Inversion).** The map $P_X \mapsto \varphi_X$ is
injective: if $\varphi_X = \varphi_Y$ everywhere, then
$P_X = P_Y$. Moreover, if $\varphi_X \in L^1(\mathbb{R})$,
then $P_X$ has a density

$$
f_X(x) = \frac{1}{2\pi} \int_{-\infty}^{\infty}
e^{-itx} \varphi_X(t) \, dt.
$$

Injectivity is the key fact. It means proving $\varphi_{X_n}(t)
\to \varphi_X(t)$ for all $t$ is sufficient to characterize the
distributional limit — which is how the CLT proof works.

### 1.3 Lévy's continuity theorem

**Theorem (Lévy's continuity theorem).**

(i) If $X_n \xrightarrow{d} X$, then $\varphi_{X_n}(t) \to
\varphi_X(t)$ for all $t \in \mathbb{R}$.

(ii) Conversely, if $\varphi_{X_n}(t) \to \varphi(t)$ for all
$t$ and $\varphi$ is continuous at $0$, then $\varphi$ is the
characteristic function of some distribution $P_X$ and
$X_n \xrightarrow{d} X$.

The converse direction is the workhorse. To prove
$X_n \xrightarrow{d} X$, it suffices to: (a) compute
$\varphi_{X_n}(t)$, (b) show it converges pointwise to
$\varphi(t)$, (c) verify $\varphi$ is continuous at 0, and
(d) identify $\varphi$ as the characteristic function of a
known distribution. This is exactly what the CLT proof does.

The continuity-at-$0$ condition in (ii) rules out degenerate
sequences whose probability mass escapes to $\pm\infty$ — it
is the characteristic-function version of tightness (§6.2).

### 1.4 The Cramér-Wold device

**Theorem (Cramér-Wold).** A sequence of random vectors
$\mathbf{X}_n \in \mathbb{R}^k$ converges in distribution to
$\mathbf{X}$ if and only if $\lambda^\top \mathbf{X}_n
\xrightarrow{d} \lambda^\top \mathbf{X}$ for every
$\lambda \in \mathbb{R}^k$.

**Proof.** ($\Rightarrow$): if $\mathbf{X}_n \xrightarrow{d}
\mathbf{X}$, then the continuous function
$\mathbf{x} \mapsto \lambda^\top \mathbf{x}$ maps this to
$\lambda^\top \mathbf{X}_n \xrightarrow{d} \lambda^\top \mathbf{X}$
by the continuous mapping theorem. ($\Leftarrow$): the
characteristic function of $\mathbf{X}_n$ satisfies
$\varphi_{\mathbf{X}_n}(t) = E[e^{it^\top \mathbf{X}_n}]
= \varphi_{\lambda^\top \mathbf{X}_n}(1)$ with $\lambda = t$.
The scalar convergence hypothesis gives
$\varphi_{\mathbf{X}_n}(t) \to \varphi_{\mathbf{X}}(t)$ for
each $t$. Lévy's theorem then gives $\mathbf{X}_n
\xrightarrow{d} \mathbf{X}$. $\blacksquare$

The Cramér-Wold device reduces multivariate convergence in
distribution to a family of scalar problems. It is how the
multivariate CLT is proved: show every linear combination
satisfies the scalar CLT, then invoke Cramér-Wold.

---

## 2. Laws of large numbers

Let $X_1, X_2, \ldots$ be random variables with common mean
$E[X_1] = \mu$. The laws of large numbers (LLNs) say the
sample average $\bar{X}_n = n^{-1}\sum_{i=1}^n X_i$ converges
to $\mu$. Consistency of every moment-based estimator — OLS,
GMM, MLE — is a consequence of an appropriate LLN.

### 2.1 Weak law of large numbers

**Theorem (Chebyshev WLLN).** If $X_1, \ldots, X_n$ are
pairwise uncorrelated with $E[X_i] = \mu$ and
$\text{Var}(X_i) \leq \sigma^2 < \infty$, then
$\bar{X}_n \xrightarrow{P} \mu$.

**Proof.** $\text{Var}(\bar{X}_n) = n^{-2}\sum_i \text{Var}(X_i)
\leq \sigma^2/n$ (by pairwise uncorrelatedness). Chebyshev's
inequality:

$$
P(|\bar{X}_n - \mu| > \varepsilon) \leq
\frac{\text{Var}(\bar{X}_n)}{\varepsilon^2} \leq
\frac{\sigma^2}{n\varepsilon^2} \to 0. \quad \blacksquare
$$

The proof requires only pairwise uncorrelated observations, not
independence — and finite variance, not finite fourth or higher
moments. This is the right version for OLS under
heteroskedastic but serially uncorrelated errors: the residuals
at different observations are uncorrelated under exogeneity,
which is enough.

**When Chebyshev fails: heavy tails.** If $E[X_i^2] = \infty$,
the Chebyshev bound gives nothing. A WLLN still holds under
the weaker condition $E[|X_i|] < \infty$, proved by
**truncation**: fix $M > 0$ and let $X_i^{(M)} = X_i
\mathbf{1}_{|X_i| \leq M}$. Write $\bar{X}_n - \mu =
(\bar{X}_n - \bar{X}_n^{(M)}) + (\bar{X}_n^{(M)} - \mu^{(M)})
+ (\mu^{(M)} - \mu)$ and control each piece: the first and
third by choosing $M$ large (DCT gives $\mu^{(M)} \to \mu$);
the second by Chebyshev applied to the truncated variables
(which have finite variance for any fixed $M$).

### 2.2 Strong law of large numbers

**Theorem (Kolmogorov SLLN).** If $X_1, X_2, \ldots$ are i.i.d.
with $E[|X_1|] < \infty$, then $\bar{X}_n \xrightarrow{\text{a.s.}} \mu$.

Almost sure convergence is stronger than convergence in
probability: $\bar{X}_n \to \mu$ with probability one — not
merely that the probability of deviation shrinks. The SLLN
is the theorem that justifies "given enough data, the sample
mean will be as close to $\mu$ as you like, and will remain
so."

**Proof sketch (finite variance case, via Borel-Cantelli).**
Assume $E[X_i^2] < \infty$. Consider the subsequence
$n_k = k^2$. By a fourth-moment calculation,
$E[(\bar{X}_{n_k} - \mu)^4] = O(n_k^{-2}) = O(k^{-4})$.
Markov's inequality at the fourth moment:

$$
P(|\bar{X}_{n_k} - \mu| > \varepsilon) \leq
\frac{E[(\bar{X}_{n_k} - \mu)^4]}{\varepsilon^4} = O(k^{-4}).
$$

Since $\sum_{k=1}^\infty O(k^{-4}) < \infty$, the
**Borel-Cantelli lemma** gives $P(|\bar{X}_{n_k} - \mu| >
\varepsilon \text{ i.o.}) = 0$. Thus $\bar{X}_{n_k}
\xrightarrow{\text{a.s.}} \mu$ along the subsequence $k^2$.
For $n$ lying between $n_k = k^2$ and $n_{k+1} = (k+1)^2$,
monotonicity of partial sums controls the difference: if all
the $X_i$ are non-negative, $\bar{X}_{n_k} \leq \bar{X}_n
\leq \bar{X}_{n_{k+1}} \cdot (n_{k+1}/n_k)$, and
$n_{k+1}/n_k \to 1$. The general case uses a truncation
argument analogous to the WLLN. $\blacksquare$

The **Borel-Cantelli lemma**: if $\sum_n P(A_n) < \infty$, then
$P(A_n \text{ i.o.}) = 0$. This follows directly from
$P(\limsup A_n) = P(\bigcap_n \bigcup_{k \geq n} A_k)
\leq \lim_n \sum_{k \geq n} P(A_k) = 0$, using continuity
from above (from [[`measure-theory.md`]] §2). Borel-Cantelli is
the fundamental tool for almost sure results.

### 2.3 LLNs for dependent data

The i.i.d. assumption fails whenever we work with time series,
panel data, or any setting where observations are correlated
across units or time.

**Ergodic theorem (Birkhoff).** For a stationary ergodic
sequence $(X_t)_{t \geq 1}$ with $E[|X_1|] < \infty$:

$$
\frac{1}{T}\sum_{t=1}^T X_t \xrightarrow{\text{a.s.}} E[X_1].
$$

Stationarity ensures a constant mean; ergodicity ensures the
time average equals the ensemble average (no partition of the
sample space into a non-trivial invariant set). This is the
LLN for stationary time series: it justifies sample-moment
estimators in time-series models and underlies the asymptotics
of VAR and LP estimators. The proof belongs in
[[`stochastic-processes.md`]] because it requires the ergodic
decomposition theorem and the martingale convergence theorem.
The result is stated here because it is assumed throughout
the econometrics layer.

**LLN for α-mixing sequences.** A stationary sequence $(X_t)$
is **α-mixing** (strongly mixing) if

$$
\alpha(k) = \sup_{\substack{A \in \mathcal{F}_{-\infty}^0 \\
B \in \mathcal{F}_k^\infty}} |P(A \cap B) - P(A)P(B)| \to 0
\quad \text{as } k \to \infty,
$$

where $\mathcal{F}_a^b$ is the σ-algebra generated by
$(X_t)_{a \leq t \leq b}$. If $\alpha(k) \to 0$ and moment
conditions hold, then $\bar{X}_T \xrightarrow{P} \mu$. Mixing
formalizes "observations far apart in time are approximately
independent." This is the formal condition behind asymptotics
for autoregressive models, panel data estimators, and LP
impulse response functions.

---

## 3. Central limit theorems

### 3.1 Classical CLT (Lindeberg-Lévy)

**Theorem.** Let $X_1, X_2, \ldots$ be i.i.d. with $E[X_i] =
\mu$ and $\text{Var}(X_i) = \sigma^2 \in (0, \infty)$. Then

$$
\sqrt{n}\!\left(\bar{X}_n - \mu\right) \xrightarrow{d}
\mathcal{N}(0, \sigma^2).
$$

**Proof via characteristic functions.** WLOG assume $\mu = 0$
and $\sigma^2 = 1$. Let $S_n = n^{-1/2}\sum_{i=1}^n X_i$.
By independence and property (5) from §1.1:

$$
\varphi_{S_n}(t) = \left[\varphi_{X_1}\!\left(\frac{t}{\sqrt{n}}
\right)\right]^n.
$$

Expand $\varphi_{X_1}$ via property (6). Since $E[X_1] = 0$
and $E[X_1^2] = 1$:

$$
\varphi_{X_1}(s) = 1 + is \cdot 0 - \frac{s^2}{2} \cdot 1
+ o(s^2) = 1 - \frac{s^2}{2} + o(s^2).
$$

Substituting $s = t/\sqrt{n}$:

$$
\varphi_{X_1}\!\left(\frac{t}{\sqrt{n}}\right) =
1 - \frac{t^2}{2n} + o\!\left(\frac{1}{n}\right).
$$

Therefore:

$$
\varphi_{S_n}(t) = \left[1 - \frac{t^2}{2n}
+ o\!\left(\frac{1}{n}\right)\right]^n
\xrightarrow{n \to \infty} e^{-t^2/2},
$$

using $(1 + a_n/n)^n \to e^a$ when $na_n \to a$. Since
$e^{-t^2/2}$ is the characteristic function of $\mathcal{N}(0,1)$
and is continuous at 0, Lévy's continuity theorem gives
$S_n \xrightarrow{d} \mathcal{N}(0,1)$. $\blacksquare$

This proof reveals why the CLT holds. Independence factors the
characteristic function into a product. Each factor at
scale $t/\sqrt{n}$ is approximately $1 - t^2/(2n)$. $n$ such
factors multiply to $e^{-t^2/2}$. The Gaussian is the unique
limit of this procedure because $e^{-t^2/2}$ is the unique
characteristic function with quadratic log — a consequence of
the Gaussian being the maximum-entropy distribution for given
mean and variance.

### 3.2 Lindeberg-Feller CLT

The i.i.d. assumption can be replaced by a condition that
formalizes "no single term dominates the sum."

Consider a **triangular array**: for each $n$, let
$X_{n1}, \ldots, X_{nn}$ be independent (not necessarily
identically distributed) with $E[X_{ni}] = 0$,
$\sigma_{ni}^2 = E[X_{ni}^2]$, and total variance
$s_n^2 = \sum_{i=1}^n \sigma_{ni}^2$.

**Lindeberg condition.** For every $\varepsilon > 0$:

$$
L_n(\varepsilon) := \frac{1}{s_n^2}\sum_{i=1}^n
E\!\left[X_{ni}^2 \,\mathbf{1}_{|X_{ni}| > \varepsilon s_n}
\right] \to 0 \quad \text{as } n \to \infty.
$$

**Theorem (Lindeberg-Feller).** If the Lindeberg condition
holds, then

$$
\frac{1}{s_n}\sum_{i=1}^n X_{ni} \xrightarrow{d} \mathcal{N}(0,1).
$$

Moreover, if additionally $\max_i \sigma_{ni}^2 / s_n^2 \to 0$
(the Feller condition), then the Lindeberg condition is also
*necessary* for the CLT to hold.

The Lindeberg condition says: the contribution to total
variance from any term's tails — beyond $\varepsilon$ times
the total standard deviation — is asymptotically negligible.
The i.i.d. case with finite variance satisfies this
automatically (each term contributes $O(1/n)$ to $s_n^2$,
so the truncation threshold $\varepsilon s_n \sim \varepsilon
\sigma\sqrt{n}$ grows, and finite variance ensures the tail
integral vanishes). The condition can fail for heteroskedastic
designs with growing leverage.

**Why econ cares.** The OLS estimator satisfies
$\sqrt{n}(\hat{\beta} - \beta) = (n^{-1}X^\top X)^{-1}
n^{-1/2}X^\top \varepsilon$, where $n^{-1/2}X^\top \varepsilon
= n^{-1/2}\sum_i x_i \varepsilon_i$ is a sum of independent
but non-identically distributed terms (regressors vary across
$i$). Asymptotic normality of OLS under heteroskedasticity
is the Lindeberg-Feller CLT applied to the score terms
$x_i \varepsilon_i$. When leverage $h_{ii} = x_i^\top
(X^\top X)^{-1} x_i$ concentrates on a few observations —
high-leverage designs — the Lindeberg condition can fail.
This is a concrete mechanism behind finite-sample failures of
asymptotic normal approximations in regression.

### 3.3 Multivariate CLT

**Theorem.** Let $\mathbf{X}_1, \mathbf{X}_2, \ldots$ be i.i.d.
random vectors in $\mathbb{R}^k$ with $E[\mathbf{X}_i] =
\boldsymbol{\mu}$ and covariance matrix $\Sigma$ finite. Then

$$
\sqrt{n}(\bar{\mathbf{X}}_n - \boldsymbol{\mu}) \xrightarrow{d}
\mathcal{N}(\mathbf{0}, \Sigma).
$$

**Proof via Cramér-Wold.** For any $\lambda \in \mathbb{R}^k$,
the scalar sequence $\lambda^\top \mathbf{X}_i$ is i.i.d. with
mean $\lambda^\top \boldsymbol{\mu}$ and variance
$\lambda^\top \Sigma \lambda$. The classical CLT gives
$\sqrt{n}(\lambda^\top \bar{\mathbf{X}}_n - \lambda^\top
\boldsymbol{\mu}) \xrightarrow{d} \mathcal{N}(0,
\lambda^\top \Sigma \lambda)$. Since this holds for all
$\lambda \in \mathbb{R}^k$, the Cramér-Wold device gives
$\sqrt{n}(\bar{\mathbf{X}}_n - \boldsymbol{\mu})
\xrightarrow{d} \mathcal{N}(\mathbf{0}, \Sigma)$. $\blacksquare$

The covariance matrix $\Sigma$ appears as the "meat" of the
sandwich estimator in heteroskedasticity-robust inference. When
the Wald statistic $n(\hat{\boldsymbol{\theta}} -
\boldsymbol{\theta}_0)^\top \hat{V}^{-1}(\hat{\boldsymbol{\theta}}
- \boldsymbol{\theta}_0) \xrightarrow{d} \chi^2_k$, the
$\chi^2_k$ distribution comes from the multivariate CLT and
the continuous mapping theorem applied to $\|\cdot\|_{\hat{V}^{-1}}^2$.

### 3.4 CLTs for dependent data

**Gordin's CLT.** Let $(X_t)_{t \geq 1}$ be stationary and
ergodic with $E[X_t] = 0$ and $E[X_t^2] < \infty$. Gordin's
condition:

$$
\sum_{k=0}^\infty \left(E\!\left[\left(E[X_k \mid
\mathcal{F}_0]\right)^2\right]\right)^{1/2} < \infty,
$$

where $\mathcal{F}_0 = \sigma(X_0, X_{-1}, \ldots)$. Under
this condition:

$$
\frac{1}{\sqrt{T}}\sum_{t=1}^T X_t \xrightarrow{d}
\mathcal{N}(0, \sigma^2_{\text{LR}}),
$$

where the **long-run variance** is

$$
\sigma^2_{\text{LR}} = \sum_{h=-\infty}^{\infty}
\text{Cov}(X_0, X_h) = \gamma_0 + 2\sum_{h=1}^\infty \gamma_h.
$$

**The long-run variance is not $\gamma_0$.** The standard
formula $\hat{\sigma}^2/n$ estimates $\gamma_0/n$, which
underestimates the variance of the sample mean whenever
$\sum_{h \geq 1} \gamma_h > 0$ (positive autocorrelation).
HAC estimators — Newey-West and its variants — estimate the
full long-run variance by a weighted sum of sample
autocovariances:

$$
\hat{\sigma}^2_{\text{NW}} = \hat{\gamma}_0
+ 2\sum_{h=1}^{H_T} k\!\left(\frac{h}{H_T}\right) \hat{\gamma}_h,
$$

where $k(\cdot)$ is a kernel (Bartlett: $k(x) = 1 - |x|$)
and $H_T$ is the bandwidth. The choice of $H_T$ and kernel
involves a bias-variance tradeoff; the Montiel
Olea–Plagborg-Møller correction in [[`local-projections.md`]] is
one modern approach to this problem at long horizons.

**CLT for α-mixing sequences.** Under summable mixing
coefficients — $\sum_{k=1}^\infty \alpha(k)^{1 - 2/p} < \infty$
for some $p > 2$ — and moment condition $E[|X_t|^p] < \infty$,
the CLT holds for $T^{-1/2}\sum_{t=1}^T X_t$ with the
long-run variance. This is the formal condition behind
asymptotic normality of general time-series estimators.

**Near-epoch dependence (NED).** A sequence $(X_t)$ is
**near-epoch dependent** on a process $(\varepsilon_t)$ if

$$
\|X_t - E[X_t \mid \mathcal{F}_{t-m}^{t+m}]\|_2 \leq d_t
\cdot \nu_m,
$$

where $(\nu_m)$ are NED coefficients with $\nu_m \to 0$ and
$(d_t)$ are bounded. NED covers cases where $X_t$ is a
nonlinear function of a mixing base process — e.g., $X_t =
g(\varepsilon_t, \varepsilon_{t-1}, \ldots)$ for a smooth
function $g$ of mixing inputs. CLTs for NED sequences are the
foundation for asymptotics of many structural macro estimators.

**Martingale difference CLT.** If $(X_t, \mathcal{F}_t)$ is a
stationary martingale difference sequence —
$E[X_t \mid \mathcal{F}_{t-1}] = 0$ — with
$E[X_t^2 \mid \mathcal{F}_{t-1}] \to \sigma^2$ in probability
(the conditional variance stabilizes), then

$$
\frac{1}{\sqrt{T}}\sum_{t=1}^T X_t \xrightarrow{d}
\mathcal{N}(0, \sigma^2).
$$

The martingale difference CLT is the right tool for GMM score
functions in time-series settings, where the moment conditions
$g(X_t, \theta_0)$ are martingale differences with respect to
the natural filtration. The proof belongs in
[[`stochastic-processes.md`]]; the result is used throughout
the econometrics layer.

---

## 4. The delta method

The CLT gives asymptotic normality of $\hat{\theta}_n$ for
$\theta_0$. The delta method extends this to $g(\hat{\theta}_n)$
for nonlinear transformations $g$.

### 4.1 Scalar delta method

**Theorem (Delta method).** Suppose
$\sqrt{n}(\hat{\theta}_n - \theta_0) \xrightarrow{d}
\mathcal{N}(0, \sigma^2)$ and $g: \mathbb{R} \to \mathbb{R}$
is differentiable at $\theta_0$. Then:

- If $g'(\theta_0) \neq 0$:
  $$\sqrt{n}(g(\hat{\theta}_n) - g(\theta_0)) \xrightarrow{d}
  \mathcal{N}(0,\, [g'(\theta_0)]^2 \sigma^2).$$
- If $g'(\theta_0) = 0$ and $g''(\theta_0)$ exists and is
  nonzero:
  $$n(g(\hat{\theta}_n) - g(\theta_0)) \xrightarrow{d}
  \frac{g''(\theta_0)}{2}\,\sigma^2\,\chi^2_1.$$

**Proof.** By the mean value theorem (from [[`real-analysis.md`]]
§4):

$$
g(\hat{\theta}_n) - g(\theta_0) =
g'(\tilde{\theta}_n)(\hat{\theta}_n - \theta_0),
$$

where $\tilde{\theta}_n$ lies between $\hat{\theta}_n$ and
$\theta_0$. Since $\hat{\theta}_n \xrightarrow{P} \theta_0$
(implied by the CLT) and $\tilde{\theta}_n$ is squeezed
between them, $\tilde{\theta}_n \xrightarrow{P} \theta_0$.
By continuity of $g'$ at $\theta_0$: $g'(\tilde{\theta}_n)
\xrightarrow{P} g'(\theta_0)$. Therefore:

$$
\sqrt{n}(g(\hat{\theta}_n) - g(\theta_0)) =
g'(\tilde{\theta}_n) \cdot \sqrt{n}(\hat{\theta}_n - \theta_0).
$$

The right side is $g'(\tilde{\theta}_n) \xrightarrow{P}
g'(\theta_0)$ times $\sqrt{n}(\hat{\theta}_n - \theta_0)
\xrightarrow{d} \mathcal{N}(0, \sigma^2)$. Slutsky's theorem
gives the product converges in distribution to
$g'(\theta_0) \cdot \mathcal{N}(0, \sigma^2) =
\mathcal{N}(0, [g'(\theta_0)]^2 \sigma^2)$. $\blacksquare$

**The second-order case $g'(\theta_0) = 0$.** When the first
derivative vanishes, the leading term in the Taylor expansion
is second-order: $g(\hat{\theta}_n) - g(\theta_0) \approx
\frac{g''(\theta_0)}{2}(\hat{\theta}_n - \theta_0)^2$.
Scaling by $n$: $n(\hat{\theta}_n - \theta_0)^2 =
[\sqrt{n}(\hat{\theta}_n - \theta_0)]^2 \xrightarrow{d}
\mathcal{N}(0, \sigma^2)^2 = \sigma^2 \chi^2_1$. The
distributional limit is chi-squared, not Gaussian, and the
convergence rate changes from $\sqrt{n}$ to $n$. This matters
for testing at boundaries: e.g., testing
$\text{H}_0: \sigma^2 = 0$ or testing whether a parameter
lies exactly on the boundary of its natural domain.

### 4.2 Vector delta method

**Theorem.** Suppose $\sqrt{n}(\hat{\boldsymbol{\theta}}_n -
\boldsymbol{\theta}_0) \xrightarrow{d} \mathcal{N}(\mathbf{0},
\Sigma)$ and $g: \mathbb{R}^k \to \mathbb{R}^m$ is
differentiable at $\boldsymbol{\theta}_0$ with Jacobian
$G = \nabla g(\boldsymbol{\theta}_0)$ (an $m \times k$ matrix,
with rows $\nabla g_j(\boldsymbol{\theta}_0)$). Then:

$$
\sqrt{n}(g(\hat{\boldsymbol{\theta}}_n) - g(\boldsymbol{\theta}_0))
\xrightarrow{d} \mathcal{N}(\mathbf{0},\, G\Sigma G^\top).
$$

**Proof.** First-order Taylor expansion at $\boldsymbol{\theta}_0$:

$$
g(\hat{\boldsymbol{\theta}}_n) - g(\boldsymbol{\theta}_0) =
G(\hat{\boldsymbol{\theta}}_n - \boldsymbol{\theta}_0) +
\|\hat{\boldsymbol{\theta}}_n - \boldsymbol{\theta}_0\| \cdot
\mathbf{r}_n,
$$

where $\mathbf{r}_n \to \mathbf{0}$ as $\hat{\boldsymbol{\theta}}_n
\to \boldsymbol{\theta}_0$. Since $\hat{\boldsymbol{\theta}}_n
\xrightarrow{P} \boldsymbol{\theta}_0$ and $\|\hat{\boldsymbol{\theta}}_n
- \boldsymbol{\theta}_0\| = O_P(n^{-1/2})$, the remainder
is $o_P(n^{-1/2})$. Therefore:

$$
\sqrt{n}(g(\hat{\boldsymbol{\theta}}_n) - g(\boldsymbol{\theta}_0))
= G \cdot \sqrt{n}(\hat{\boldsymbol{\theta}}_n -
\boldsymbol{\theta}_0) + o_P(1)
\xrightarrow{d} G \cdot \mathcal{N}(\mathbf{0}, \Sigma) =
\mathcal{N}(\mathbf{0}, G\Sigma G^\top). \quad \blacksquare
$$

The formula $G\Sigma G^\top$ is the **sandwich formula**. It
appears everywhere: OLS with heteroskedastic errors
(White's formula), GMM standard errors, clustered standard
errors, IV standard errors (via the Wald ratio as a function
of covariances), and impulse response function confidence
bands (IRF coefficients are nonlinear functions of VAR
coefficients).

**Standard applications:**

- *Ratio estimator* $\hat{r} = \hat{\mu}_Y / \hat{\mu}_X$:
  $g(\mu_X, \mu_Y) = \mu_Y/\mu_X$,
  $G = (-\mu_Y/\mu_X^2,\; 1/\mu_X)$.
- *Log transform* $g(\theta) = \log\theta$:
  $g'(\theta_0) = 1/\theta_0$. Used for geometric means,
  elasticities, and log-linearized models.
- *Marginal effects* in Probit/Tobit: $\partial P(Y=1|X) /
  \partial X = \phi(X^\top\beta)\cdot\beta$ is a nonlinear
  function of $\hat{\beta}$; asymptotic variance via the
  vector delta method.
- *IV Wald estimator* $\hat{\beta} = \hat{\pi}_{ZY}/
  \hat{\pi}_{ZX}$ (ratio of reduced-form to first-stage
  covariances): the ratio form means the delta method gives
  asymptotic variance $[\hat{\pi}_{ZX}]^{-2}
  \text{Var}(\hat{\pi}_{ZY} - \hat{\beta}\hat{\pi}_{ZX})$,
  which is the foundation of standard IV standard errors.
- *IRF confidence bands* in VAR models: each IRF coefficient
  at horizon $h$ is a polynomial function of the VAR
  companion matrix; delta-method bands propagate estimation
  uncertainty through this mapping.

---

## 5. Asymptotic notation: $o_P$ and $O_P$

Asymptotic proofs are organized around identifying which terms
are negligible and which are not. The stochastic order notation
is the language for this.

### 5.1 Definitions

**$o_P(r_n)$.** $R_n = o_P(r_n)$ if $R_n/r_n \xrightarrow{P}
0$. The most common case: $R_n = o_P(1)$ means
$R_n \xrightarrow{P} 0$.

**$O_P(r_n)$.** $R_n = O_P(r_n)$ if $R_n/r_n$ is **bounded
in probability** (tight): for every $\varepsilon > 0$ there
exist $M < \infty$ and $N$ such that
$P(|R_n/r_n| > M) < \varepsilon$ for all $n \geq N$.
The most common case: $R_n = O_P(1)$ means the sequence
is stochastically bounded.

**Connection to the CLT.** $\bar{X}_n - \mu = O_P(n^{-1/2})$
because $\sqrt{n}(\bar{X}_n - \mu) \xrightarrow{d}
\mathcal{N}(0, \sigma^2)$ — convergence in distribution
implies stochastic boundedness. Any $\sqrt{n}$-consistent
estimator satisfies $\hat{\theta}_n - \theta_0 = O_P(n^{-1/2})$.

### 5.2 Arithmetic rules

$$
o_P(1) + o_P(1) = o_P(1), \qquad
O_P(1) + O_P(1) = O_P(1),
$$
$$
o_P(1) \cdot O_P(1) = o_P(1), \qquad
O_P(r_n) \cdot O_P(s_n) = O_P(r_n s_n),
$$
$$
o_P(r_n) + O_P(s_n) = O_P(s_n) \quad \text{if } r_n / s_n
\to 0.
$$

The most useful rule: a negligible term times a bounded term
remains negligible. Asymptotic proofs typically write the
estimator error as a sum of terms, apply CLT-like results to
the $O_P(n^{-1/2})$ "signal" terms, and discard the
$o_P(n^{-1/2})$ "remainder" terms using these arithmetic rules.

**The standard language:**

- **Consistency**: $\hat{\theta}_n - \theta_0 = o_P(1)$.
- **$\sqrt{n}$-consistency**: $\hat{\theta}_n - \theta_0 =
  O_P(n^{-1/2})$.
- **Asymptotic normality**: $\sqrt{n}(\hat{\theta}_n -
  \theta_0) \xrightarrow{d} \mathcal{N}(0, V)$.

These are three distinct claims in increasing strength. An
estimator can be consistent without being $\sqrt{n}$-consistent
(e.g., some nonparametric estimators achieve only
$n^{-2/5}$ rate). An estimator can be $\sqrt{n}$-consistent
without having a normal limit (e.g., the sample maximum of
uniform observations). All three together — consistency,
$\sqrt{n}$ rate, and normality — are what justify the standard
confidence interval formula $\hat{\theta}_n \pm 1.96
\hat{V}^{1/2}/\sqrt{n}$.

### 5.3 Slutsky and CMT rephrased

**Slutsky's theorem**: if $X_n \xrightarrow{d} X$ and
$Y_n = c + o_P(1)$, then $X_n + Y_n \xrightarrow{d} X + c$
and $X_n Y_n \xrightarrow{d} cX$. The classic use: the OLS
$t$-statistic has numerator $\xrightarrow{d} \mathcal{N}(0,1)$
and denominator $\xrightarrow{P} 1$ (consistent standard
error estimate); Slutsky gives the ratio $\xrightarrow{d}
\mathcal{N}(0,1)$.

**Continuous mapping theorem (CMT)**: if $X_n \xrightarrow{d}
X$ and $g$ is continuous ($P_X$-a.e.), then $g(X_n)
\xrightarrow{d} g(X)$. The delta method is the CMT with
$g$ replaced by its first-order Taylor approximation — the
proof of the delta method in §4.1 shows why the CMT applies
to $g(\hat{\theta}_n)$ once the remainder is $o_P(n^{-1/2})$.

---

## 6. Weak convergence and tightness

The modes of convergence from [[`measure-theory.md`]] §8 are for
sequences of real-valued random variables. Weak convergence of
probability measures on metric spaces is the framework for
functional limit theorems — the setting where the limiting
object is a stochastic process rather than a scalar.

### 6.1 Weak convergence of measures

**Definition.** Probability measures $\mu_n$ on
$(\mathbb{R}^k, \mathcal{B}(\mathbb{R}^k))$ **converge weakly**
to $\mu$ (written $\mu_n \Rightarrow \mu$) if

$$
\int h \, d\mu_n \to \int h \, d\mu
\quad \text{for all bounded continuous } h: \mathbb{R}^k
\to \mathbb{R}.
$$

This is the definition behind convergence in distribution for
random vectors. The **portmanteau lemma** gives equivalent
characterizations: $\mu_n \Rightarrow \mu$ iff $F_{\mu_n}(t)
\to F_\mu(t)$ at all continuity points of $F_\mu$ (the CDF
definition), iff $\limsup_n \mu_n(F) \leq \mu(F)$ for all
closed $F$, iff $\liminf_n \mu_n(G) \geq \mu(G)$ for all
open $G$.

### 6.2 Tightness and Prokhorov's theorem

**Definition.** A family $\{\mu_\alpha\}$ of probability
measures is **tight** if for every $\varepsilon > 0$ there
exists a compact $K \subset \mathbb{R}^k$ with $\mu_\alpha(K)
\geq 1 - \varepsilon$ for all $\alpha$. Tightness says the
mass cannot escape to infinity uniformly over the family.

**Theorem (Prokhorov).** A sequence $\{\mu_n\}$ of probability
measures on $\mathbb{R}^k$ is **relatively compact** in the
weak topology — every subsequence has a weakly convergent
further subsequence — if and only if it is tight.

Prokhorov's theorem is the compactness result for probability
measures. In the CLT proof, once characteristic functions
converge pointwise, tightness follows automatically (the
characteristic function controls tail probability), and
Prokhorov supplies the convergent subsequence; Lévy's theorem
identifies the limit uniquely, making the full sequence
converge.

### 6.3 Functional CLT (Donsker's theorem)

The classical CLT says the normalized sample mean converges in
distribution to a normal. Donsker's theorem says the entire
normalized partial-sum *process* converges as a random element
of a function space.

**Definition.** For $t \in [0,1]$, define the normalized
partial-sum process:

$$
W_n(t) = \frac{1}{\sigma\sqrt{n}} \sum_{i=1}^{\lfloor nt \rfloor}
(X_i - \mu),
$$

linearly interpolated between integer values of $nt$.

**Theorem (Donsker).** Under the hypotheses of the classical
CLT, $W_n \Rightarrow W$ in $C[0,1]$ equipped with the
sup-norm topology, where $W$ is standard Brownian motion.

**Implications.** Functionals of the partial-sum process that
are continuous in the sup-norm inherit convergence in
distribution by the CMT applied in $C[0,1]$:

- **Kolmogorov-Smirnov statistic** $\sup_t |W_n(t)|
  \xrightarrow{d} \sup_t |W(t)|$ — the Kolmogorov distribution.
- **CUSUM tests** for structural breaks: the test statistic is
  $\sup_t W_n(t)^2$, which converges to the distribution of
  the Brownian bridge squared.
- **Cointegration tests** (Johansen, PP, KPSS): limiting
  distributions are functionals of Brownian motion on
  $[0,1]$. These are computable via numerical integration of
  the Brownian bridge distribution.

The formal framework for Donsker's theorem is weak convergence
in metric spaces (Billingsley 1968). The proofs belong in
[[`stochastic-processes.md`]].

---

## 7. Asymptotic foundations of econometrics

This section assembles the limit theorems into the framework
that the econometrics layer assumes throughout.

### 7.1 The standard asymptotic argument

Most econometric estimators have the following structure.
The true parameter $\theta_0$ solves $E[m(X, \theta_0)] = 0$
for some moment function $m$. The estimator $\hat{\theta}_n$
solves $n^{-1}\sum_i m(X_i, \hat{\theta}_n) = 0$ or
minimizes a sample criterion. Under regularity conditions:

$$
\sqrt{n}(\hat{\theta}_n - \theta_0) =
\underbrace{-J_0^{-1}}_{\text{inverse Jacobian}} \cdot
\underbrace{\frac{1}{\sqrt{n}}\sum_{i=1}^n
m(X_i, \theta_0)}_{\text{CLT term}} + o_P(1),
$$

where $J_0 = \partial E[m(X, \theta_0)]/\partial\theta^\top$
is the population Jacobian. The CLT applied to
$n^{-1/2}\sum_i m(X_i, \theta_0)$ gives asymptotic normality
with variance $\Omega_0 = E[m(X, \theta_0)m(X, \theta_0)^\top]$.
By the continuous mapping theorem (linear transformation):

$$
\sqrt{n}(\hat{\theta}_n - \theta_0) \xrightarrow{d}
\mathcal{N}(\mathbf{0},\; J_0^{-1}\Omega_0 J_0^{-\top}).
$$

The matrix $J_0^{-1}\Omega_0 J_0^{-\top}$ is the **sandwich
variance**. For OLS: $J_0 = -E[x_i x_i^\top]$ and
$\Omega_0 = E[\varepsilon_i^2 x_i x_i^\top]$; the sandwich
reduces to the White (1980) heteroskedasticity-robust variance.
For just-identified IV: $J_0 = -E[z_i x_i^\top]$ and the
sandwich is the standard IV asymptotic variance.

The $o_P(1)$ remainder requires a **uniform law of large
numbers** (ULLN) and a mean-value expansion to bound the
error in linearizing $m(X, \hat{\theta}_n)$ around $\theta_0$.

### 7.2 Uniform laws of large numbers

The argument in §7.1 requires controlling the sample moments
$n^{-1}\sum_i m(X_i, \theta)$ uniformly in $\theta$ — not
just pointwise. A pointwise LLN gives convergence for each
fixed $\theta$; a ULLN gives it simultaneously over all $\theta$.

**Theorem (ULLN).** Under regularity conditions (compact
parameter space $\Theta$; $m(x, \cdot)$ continuous on $\Theta$
for each $x$; dominated integrability $E[\sup_\theta
\|m(X, \theta)\|] < \infty$):

$$
\sup_{\theta \in \Theta} \left\|
\frac{1}{n}\sum_{i=1}^n m(X_i, \theta) -
E[m(X, \theta)]\right\| \xrightarrow{P} 0.
$$

The domination condition $E[\sup_\theta \|m(X, \theta)\|] <
\infty$ is the ULLN analogue of the DCT's domination
condition. When it fails — in semiparametric models with
infinite-dimensional nuisance parameters — more sophisticated
arguments from empirical process theory (Vapnik-Chervonenkis
covering number bounds, Donsker class conditions) replace it.
In practice, checking the domination condition is often the
hardest part of a formal consistency proof.

### 7.3 Triangular arrays and non-standard asymptotics

Many econometric problems involve **triangular arrays**:
observation $X_{ni}$ has a distribution that depends on $n$.
Examples:

- **Local alternatives** in power calculations:
  $\mu_n = c/\sqrt{n}$, so the effect shrinks as $n$ grows.
  Power under local alternatives is analyzed using the
  non-centrality parameter of the limiting distribution.
- **Weak instruments** (Staiger-Stock 1997):
  $E[Z_i X_i] = \mu/\sqrt{n}$ for a fixed $c$. The first
  stage has "weak" signal of order $n^{-1/2}$. The IV
  estimator has a non-normal limit in this local-to-zero
  specification — the ratio of two correlated normals.
  Anderson-Rubin inference is robust to this; see
  [[`instrumental-variables.md`]].
- **Bandwidth-indexed estimators**: a kernel estimator with
  bandwidth $h_n \to 0$ is a triangular array. The
  bias-variance tradeoff in bandwidth selection is a
  finite-sample question; the asymptotic theory (including
  the CCT optimal bandwidth in [[`regression-discontinuity.md`]])
  is Lindeberg-Feller applied to weighted sums where the
  weights depend on $n$.

In all these cases, the standard CLT (for a fixed sequence
of i.i.d. random variables) does not apply. The
Lindeberg-Feller CLT for triangular arrays does, but the
limiting distribution may not be Gaussian — if the
signal-to-noise ratio does not stabilize.

### 7.4 When standard asymptotics breaks

The limit theorems above assume the asymptotic distribution
is non-degenerate and the model is correctly specified. Several
situations violate this, and recognizing them is a core skill.

**1. Weak identification.** If the Jacobian $J_0$ is
near-singular — the instrument is weak, the model is
semi-identified, or the parameter appears only in higher-order
terms — then $J_0^{-1}$ in the sandwich formula is large or
ill-defined. The limiting distribution of $\hat{\theta}_n$ may
not be normal; it may have heavier tails, or may not converge
at the $\sqrt{n}$ rate. Confidence sets based on inverting a
test statistic (Anderson-Rubin, CLR) are robust; Wald-based
intervals are not.

**2. Boundary parameters.** If $\theta_0$ lies on the boundary
of the parameter space (e.g., variance component equals zero,
ARMA model has a unit root), the score at $\theta_0$ may not
be zero, the CLT expansion breaks down, and the limiting
distribution is a mix of degenerate mass at the boundary and
a chi-bar distribution. Testing changes fundamentally; the
standard chi-squared critical values are too large.

**3. Non-stationary processes.** If $X_t$ has a unit root,
then $T^{-1}\sum_{t=1}^T X_t = O_P(T^{1/2})$ rather than
$O_P(1)$. Sample moments diverge. Regressions involving
non-stationary series produce spurious results at standard
significance levels. The correct asymptotic framework is
local-to-unity, with limiting distributions expressed as
functionals of Brownian motion — see [[`stochastic-processes.md`]].

**4. Non-standard identification.** When the parameter of
interest is set-identified rather than point-identified, the
standard point estimate and Wald confidence interval make no
sense. Confidence regions for set-identified parameters
(Imbens-Manski bounds, projection-based methods) require a
different asymptotic theory.

**5. Heavy tails.** If $E[|X_i|^2] = \infty$, the sample
mean converges at rate slower than $n^{-1/2}$ and the CLT
does not apply. The limiting distribution of normalized
partial sums is stable with index $\alpha < 2$ (Lévy's
stable laws). Relevant for financial data where extreme events
have non-negligible probability; the Kolmogorov-Smirnov and
$\chi^2$ tests are unreliable.

---

## 8. What this file doesn't cover, and where to find it

**Martingale theory.** Filtrations and adapted processes,
martingale convergence theorems (both a.s. and in $L^2$),
the optional stopping theorem, the Doob decomposition — all
belong in [[`stochastic-processes.md`]]. The martingale difference
CLT (mentioned in §3.4) is the key result for GMM asymptotics
in time series; the proof goes there.

**Stochastic processes.** Brownian motion (existence and
properties), the Wold decomposition (every covariance-stationary
process with a linear component is an infinite-order MA),
ARMA and their state-space representations, unit root
asymptotics (local-to-unity, the Dickey-Fuller distribution),
stochastic integration (Itô calculus) — all in
[[`stochastic-processes.md`]].

**Empirical process theory.** Vapnik-Chervonenkis classes
and covering numbers, Glivenko-Cantelli and Donsker classes,
uniform CLTs in infinite-dimensional function spaces, the
bootstrap as a consequence of the functional CLT — not covered
here. The standard references are Pollard's *Empirical
Processes* and van der Vaart and Wellner's *Weak Convergence
and Empirical Processes*. These tools appear in the asymptotic
theory of DML and semiparametric estimation; [[`double-machine-learning.md`]]
will use them.

**Bayesian asymptotics.** The Bernstein-von Mises theorem
(under regularity, the posterior concentrates around the MLE
with variance equal to the frequentist asymptotic variance)
and its many caveats (fails under model misspecification,
semiparametric models, high dimensions) are not covered here.

**Bootstrap.** The consistency of the nonparametric bootstrap
for the sample mean is a consequence of the functional CLT and
Donsker's theorem (the bootstrap empirical process weakly
converges to the same limit as the original empirical process).
The wild bootstrap, the block bootstrap for time series, and
the pairs bootstrap — all valid for different settings — not
covered here; Horowitz's chapter in the *Handbook of
Econometrics* is the reference.

---

## Cross-references

- [[`20_Math/measure-theory.md`]] — the direct prerequisite.
  Probability spaces, random variables, expectation, the
  five modes of convergence (§8 there), Slutsky's theorem,
  the CMT, conditional expectation (§9 there) — all assumed
  here without re-derivation. This file adds the limit
  theorems that §10 of that file promises.
- [[`20_Math/real-analysis.md`]] — the mean value theorem (§4)
  is used in the delta method proof; Bolzano-Weierstrass
  and compactness (§3) underlie the ULLN; the Borel-Cantelli
  argument in §2.2 uses continuity from above (a consequence
  of countable additivity from [[`measure-theory.md`]] §2).
- [[`20_Math/functional-analysis.md]]` — the $L^2$ projection
  characterization of conditional expectation (§9 there) is
  the geometric picture behind the orthogonality conditions
  that define efficient estimators; the spectral theorem for
  compact self-adjoint operators (§8 there) underlies the
  CLT in function spaces.
- [[`20_Math/optimization.md`]] — the score function
  $\partial \log f(X; \theta)/\partial\theta$ and the
  information matrix $I(\theta) = E[-\partial^2 \log f
  /\partial\theta^2]$ appear in MLE asymptotics; the
  asymptotic variance of MLE is $I(\theta_0)^{-1}$, the
  Cramér-Rao lower bound.
- [[`20_Math/stochastic-processes.md`]] — martingale CLT, Birkhoff
  ergodic theorem (§2.3), Donsker's theorem in full (§6.3),
  unit root asymptotics (§7.4), Itô calculus.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] —
  weak instrument asymptotics (§7.3 here) is the theoretical
  backdrop for why $F > 10$ is insufficient (the local-to-zero
  specification) and why Anderson-Rubin inference is robust.
- [[`10_Methods/Econometrics/local-projections.md`]] — the
  long-run variance formula (§3.4 here) is the theoretical
  foundation for HAC standard errors and the Montiel
  Olea–Plagborg-Møller long-horizon bias correction.
- [[`10_Methods/Econometrics/difference-in-differences.md`]] —
  clustered inference extends the Lindeberg-Feller CLT to
  sums of within-cluster sums; the formal version of
  clustering asymptotics is the framework in §7.1 applied
  to cluster-level scores.

*Last updated: 2026-04-10. Covers characteristic functions
and Lévy's theorem, LLNs (Chebyshev, Kolmogorov, ergodic,
mixing), CLTs (classical, Lindeberg-Feller, multivariate,
dependent-data), the delta method (scalar and vector, with
proofs), $o_P/O_P$ notation, weak convergence and Prokhorov,
Donsker's theorem (stated), and the asymptotic foundations of
the standard econometric argument. Martingale theory,
stochastic processes, and empirical process theory are
deferred to [[`stochastic-processes.md`]].*
