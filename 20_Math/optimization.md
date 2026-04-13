---
tags: [math, optimization, foundations]
related: [real-analysis, measure-theory, functional-analysis, dynamic-programming, probability, modern-toolkit-references, Principles]
---

# Optimization

This file is the applied counterpart to the foundational math
layer. [[`real-analysis.md`]] built the structural results —
completeness, compactness, continuity, the implicit function
theorem, the contraction mapping theorem. This file puts those
results to work in optimization: when problems have solutions,
what first- and second-order conditions say, how constraints
enter via the Lagrangian and the KKT framework, how duality
relates primal and dual problems, and what the envelope theorem
says about how solutions move when parameters change. A final
section addresses numerical optimization — the gap between
mathematical existence and computational tractability.

The file is shorter than the purely foundational math files
because much of its content is applied rather than theoretical.
Proofs are included for the KKT conditions and the envelope
theorem, where having the argument in hand helps with applying
the theorem correctly. Results whose proofs require machinery
not yet built (infinite-dimensional duality, subdifferential
calculus in Banach spaces) are stated with a pointer forward to
[[`functional-analysis.md`]].

Econometric connections run throughout: OLS is a convex
quadratic problem and has no local-optima problem. MLE is
convex under the exponential family and not in general. GMM is
generically non-convex and has genuine local-optima concerns.
Structural estimation lives in constrained optimization.
Comparative-statics arguments rest on the implicit function
theorem and the envelope theorem. All of this machinery lives
here.

Prerequisite: [[`real-analysis.md`]], particularly §2 (metric
spaces and compactness), §3 (continuity, extreme value
theorem), §4 (differentiation), §5 (inverse and implicit
function theorems), and §6 (contraction mapping). Most
sections do not require [[`measure-theory.md`]] directly, but the
information-matrix discussion in §6.2 uses the dominated
convergence theorem to differentiate under the integral.

---

## 1. Convexity

### 1.1 Convex sets

A set $C \subseteq \mathbb{R}^n$ is *convex* if for any
$x, y \in C$ and any $\lambda \in [0, 1]$, the point
$\lambda x + (1-\lambda) y \in C$. The line segment between
any two points stays inside the set.

Standard examples: any halfspace $\{x : a^\top x \leq b\}$
is convex; any polyhedron (finite intersection of halfspaces)
is convex; the unit ball under any norm is convex; the
positive semidefinite cone is convex; any intersection of
convex sets is convex.

Convexity of the feasible set is load-bearing for global
optimality: if you minimize a convex function over a convex
set and find a local minimum, it is the global minimum. That
is the property that makes linear programming, quadratic
programming, and exponential-family MLE tractable.

### 1.2 Convex functions

A function $f: C \to \mathbb{R}$ on a convex set $C$ is
*convex* if for all $x, y \in C$ and $\lambda \in [0, 1]$:
$$f(\lambda x + (1-\lambda)y) \leq \lambda f(x) + (1-\lambda)f(y).$$
The function lies below or on its chords. $f$ is *strictly
convex* if the inequality is strict for $x \neq y$ and
$\lambda \in (0, 1)$. $f$ is *concave* if $-f$ is convex.

**Equivalent characterizations for differentiable $f$.**
Suppose $f: C \to \mathbb{R}$ is differentiable on the
interior of a convex open set $C$.

- *Tangent-plane characterization:* $f$ is convex if and
  only if $f(y) \geq f(x) + \nabla f(x)^\top (y - x)$ for
  all $x, y \in C$. The function lies above every first-order
  approximation.
- *Hessian characterization:* $f$ is convex if and only if
  $\nabla^2 f(x) \succeq 0$ (positive semidefinite, PSD) at
  every interior point.
- $f$ is strictly convex if $\nabla^2 f(x) \succ 0$ (PD)
  everywhere — the converse fails (e.g., $f(x) = x^4$ is
  strictly convex but $f''(0) = 0$).

**Preservation of convexity.** Convexity is preserved under:
non-negative scaling ($\alpha f$ for $\alpha \geq 0$); sums
($f + g$ when both convex); composition with affine maps
($f(Ax + b)$ is convex when $f$ is convex); and pointwise
supremum ($\sup_\alpha f_\alpha(x)$ over a family of convex
functions). The last rule is the engine behind the dual
representation of convex functions and the convexity of the
maximum-likelihood objective in exponential-family models.

**Sublevel sets.** The sublevel set $\{x \in C : f(x) \leq c\}$
is convex for every $c$ when $f$ is convex. This is relevant
for constructing feasible sets and for checking whether
restriction to a sublevel set preserves the useful structure
of the optimization problem.

### 1.3 Why this matters for econ

The OLS objective $\|y - X\beta\|^2$ has Hessian $2X^\top X$,
which is PSD. Under full column rank of $X$, it is PD and the
objective is strictly convex with a unique global minimum.

The negative log-likelihood of any exponential-family model
(Poisson, logistic, Gaussian, negative binomial) is convex in
the natural parameters. MLE for these models is a convex
problem: any local minimum is the global minimum.

The GMM objective $g(\theta)^\top W g(\theta)$, where
$g(\theta) = n^{-1}\sum_i m(x_i, \theta)$, is convex if and
only if $g(\theta)$ is affine in $\theta$. For linear IV,
$g(\theta) = Z^\top(y - X\theta)/n$ is affine and GMM reduces
to 2SLS — convex. For nonlinear moment conditions, the GMM
objective is generically non-convex. This is the first-order
reason why GMM in structural models has genuine local-optima
problems.

---

## 2. Unconstrained optimization

### 2.1 Existence

The baseline existence result is the extreme value theorem
([[`real-analysis.md`]] §3): a continuous function on a compact
set attains its minimum and maximum. For problems over
unbounded domains, the standard workaround is *coercivity*:
$f(x) \to +\infty$ as $\|x\| \to \infty$. A continuous
coercive function on $\mathbb{R}^n$ attains its minimum
because the sublevel sets are compact. OLS is coercive (the
sum of squares blows up as $\|\beta\| \to \infty$). Many
structural models are not coercive, and verifying coercivity
is often a nontrivial step in a theoretical identification
argument.

### 2.2 First-order conditions

If $f: U \to \mathbb{R}$ is differentiable on an open set
$U \subseteq \mathbb{R}^n$ and $x^*$ is a local minimizer,
then $\nabla f(x^*) = 0$. This is necessary, not sufficient.
The gradient-zero condition picks out all critical points:
local minima, local maxima, and saddle points.

### 2.3 Second-order conditions

At a critical point $x^*$ where $\nabla f(x^*) = 0$:

- $\nabla^2 f(x^*) \succ 0$ $\Rightarrow$ $x^*$ is a strict
  local minimizer (sufficient).
- $\nabla^2 f(x^*) \prec 0$ $\Rightarrow$ $x^*$ is a strict
  local maximizer.
- $\nabla^2 f(x^*)$ indefinite $\Rightarrow$ $x^*$ is a
  saddle point.
- $\nabla^2 f(x^*) \succeq 0$ but not PD $\Rightarrow$
  inconclusive; higher-order conditions needed.

The second-order necessary condition at a local minimizer
is $\nabla^2 f(x^*) \succeq 0$. The sufficient condition is
$\nabla^2 f(x^*) \succ 0$.

### 2.4 Local vs global optima

For a convex function on a convex domain, every local
minimizer is a global minimizer. The practical consequence
in econometrics: OLS and convex MLE have no local-optima
problem — start gradient descent anywhere and converge to
the global solution. GMM, nonlinear least squares, and
structural model estimation (DSGE, BLP, matching models)
generally have local optima. Multiple starting values,
global-search algorithms, or theoretical structure (uniqueness
proofs) is needed before a reported estimate can be called the
global solution.

---

## 3. Constrained optimization

### 3.1 Equality constraints and the Lagrangian

Consider
$$\min_{x \in \mathbb{R}^n} f(x) \quad \text{s.t.} \quad h(x) = 0,$$
where $h: \mathbb{R}^n \to \mathbb{R}^m$, $m < n$. The
*Lagrangian* is
$$\mathcal{L}(x, \mu) = f(x) + \mu^\top h(x),$$
where $\mu \in \mathbb{R}^m$ is the vector of Lagrange
multipliers.

**Theorem (first-order necessary conditions).** Suppose $x^*$
is a local minimizer of $f$ subject to $h(x) = 0$, and that
the Jacobian $Dh(x^*)$ has rank $m$ (the *linear independence
constraint qualification*, LICQ). Then there exists
$\mu^* \in \mathbb{R}^m$ such that
$$\nabla_x \mathcal{L}(x^*, \mu^*) = \nabla f(x^*) + Dh(x^*)^\top \mu^* = 0.$$

Geometrically: at the optimum, the gradient of the objective
is a linear combination of the constraint gradients. There is
no feasible direction that reduces $f$ — any direction
reducing $f$ would violate at least one constraint.

The Lagrange multipliers $\mu^*$ have an economic
interpretation: $\mu_j^*$ is the shadow price of the $j$-th
constraint, the rate at which the optimal value changes when
that constraint is relaxed. This is the envelope theorem
applied to equality constraints; the formal statement is in
§5.

### 3.2 Inequality constraints: KKT conditions

The general constrained problem:
$$\min_{x \in \mathbb{R}^n} f(x) \quad \text{s.t.} \quad g(x) \leq 0, \quad h(x) = 0,$$
where $g: \mathbb{R}^n \to \mathbb{R}^k$ and
$h: \mathbb{R}^n \to \mathbb{R}^m$. The Lagrangian is
$$\mathcal{L}(x, \lambda, \mu) = f(x) + \lambda^\top g(x) + \mu^\top h(x).$$

**Theorem (Karush-Kuhn-Tucker, KKT).** Suppose $x^*$ is a
local minimizer, and that LICQ holds at $x^*$: the gradients
of the active inequality constraints $\{\nabla g_i(x^*) :
g_i(x^*) = 0\}$ together with $\{\nabla h_j(x^*)\}$ are
linearly independent. Then there exist $\lambda^* \in
\mathbb{R}^k$ and $\mu^* \in \mathbb{R}^m$ satisfying:

1. **Stationarity:** $\nabla f(x^*) + \lambda^{*\top} \nabla g(x^*) + \mu^{*\top} \nabla h(x^*) = 0$
2. **Primal feasibility:** $g(x^*) \leq 0$ and $h(x^*) = 0$
3. **Dual feasibility:** $\lambda^* \geq 0$
4. **Complementary slackness:** $\lambda_i^* g_i(x^*) = 0$ for each $i$

**Proof.** If $x^*$ is a local minimizer, no *feasible descent
direction* exists. A direction $d \in \mathbb{R}^n$ is a
feasible descent direction if: $\nabla f(x^*)^\top d < 0$
(descent for $f$); $\nabla g_i(x^*)^\top d \leq 0$ for each
active $i \in \mathcal{A}(x^*) = \{i : g_i(x^*) = 0\}$
(active constraints not immediately violated); and $\nabla
h_j(x^*)^\top d = 0$ for each $j$ (equality constraints
maintained to first order).

By *Farkas' lemma* — a separation theorem for polyhedral
cones — the system
$$\nabla f(x^*)^\top d < 0, \quad \nabla g_i(x^*)^\top d \leq 0 \ (i \in \mathcal{A}), \quad \nabla h_j(x^*)^\top d = 0 \ (\forall j)$$
has no solution if and only if there exist $\lambda_i^* \geq 0$
for $i \in \mathcal{A}$ and $\mu_j^*$ for all $j$ such that
$$\nabla f(x^*) + \sum_{i \in \mathcal{A}} \lambda_i^* \nabla g_i(x^*) + \sum_j \mu_j^* \nabla h_j(x^*) = 0.$$

Setting $\lambda_i^* = 0$ for inactive $i \notin \mathcal{A}$
completes the construction. Complementary slackness holds
automatically: if $g_i(x^*) < 0$ then $i \notin \mathcal{A}$
and $\lambda_i^* = 0$; if $g_i(x^*) = 0$ then $i \in
\mathcal{A}$ and $\lambda_i^* \geq 0$. In both cases
$\lambda_i^* g_i(x^*) = 0$. $\blacksquare$

**Reading complementary slackness.** The condition
$\lambda_i^* g_i(x^*) = 0$ says: either the constraint is
active ($g_i(x^*) = 0$) or its multiplier is zero
($\lambda_i^* = 0$). Inactive constraints are not binding and
carry no shadow price. Active constraints may or may not carry
a positive shadow price — a constraint with $\lambda_i^* = 0$
at the optimum is satisfied with equality but could be
relaxed without changing the optimum value. The genuinely
binding constraints have $\lambda_i^* > 0$.

**Sufficiency under convexity.** KKT is necessary under LICQ.
It is also sufficient for a *global* minimizer when $f$ and
each $g_i$ are convex and the equality constraints are affine.
In a convex program, satisfying KKT certifies global
optimality.

**Constraint qualifications.** LICQ is sufficient but not
necessary for multipliers to exist. Other constraint
qualifications sometimes easier to verify: *Slater's
condition* for convex programs (there exists a strictly
feasible $x$ with $g(x) < 0$) implies LICQ is not needed
— multipliers exist and strong duality holds. *Mangasarian-
Fromovitz* is a less restrictive regularity condition for
non-convex programs. If no constraint qualification holds,
multipliers may not exist or may not be unique, and the
economic interpretation of shadow prices breaks down.

---

## 4. Duality

### 4.1 The Lagrangian dual

For the minimization problem
$$p^* = \min_x f(x) \quad \text{s.t.} \quad g(x) \leq 0, \quad h(x) = 0,$$
the *Lagrangian dual function* is
$$q(\lambda, \mu) = \inf_x \mathcal{L}(x, \lambda, \mu) = \inf_x \left[f(x) + \lambda^\top g(x) + \mu^\top h(x)\right].$$
For any fixed $(\lambda, \mu)$, $q(\lambda, \mu)$ is the
unconstrained infimum of an affine function of $(\lambda, \mu)$.
The dual function is always *concave* (the pointwise infimum
of affine functions is concave, regardless of the structure
of the primal). The *dual problem* is
$$d^* = \max_{\lambda \geq 0,\, \mu} q(\lambda, \mu).$$
Since $q$ is concave, the dual problem is always a concave
maximization — i.e., a convex optimization problem —
regardless of whether the primal is convex.

### 4.2 Weak and strong duality

**Weak duality.** For any primal-feasible $x$ and any
dual-feasible $(\lambda \geq 0, \mu)$:
$$q(\lambda, \mu) \leq \mathcal{L}(x, \lambda, \mu) = f(x) + \underbrace{\lambda^\top g(x)}_{\leq\, 0} + \underbrace{\mu^\top h(x)}_{=\, 0} \leq f(x).$$
Taking the infimum over primal-feasible $x$ and the supremum
over dual-feasible $(\lambda, \mu)$ gives $d^* \leq p^*$. The
dual value is always a lower bound for the primal. The
*duality gap* is $p^* - d^* \geq 0$.

**Strong duality.** Under *Slater's condition* — the primal is
a convex program ($f$ and $g_i$ convex, $h_j$ affine) and
there exists a strictly feasible point with $g(x) < 0$ and
$h(x) = 0$ — the duality gap is zero: $d^* = p^*$. Moreover,
the dual optimum is attained, and any pair $(x^*, \lambda^*,
\mu^*)$ satisfying both primal and dual optimality also
satisfies KKT. Strong duality identifies the KKT conditions
as the joint optimality conditions for the primal and dual.

### 4.3 Economic interpretation

In an economic optimization problem — cost minimization,
utility maximization, social planner — Lagrange multipliers
at the optimum are shadow prices: the marginal value of
relaxing a constraint by one unit. The dual problem prices
the constraints: what does it cost (in units of the
objective) to expand the feasible set?

This is the formal content of the second welfare theorem
(competitive prices decentralize Pareto optima), LP duality
(dual of a cost-minimization LP is a profit-maximization
LP), and the pricing of Arrow-Debreu commodities (which
are the multipliers on state-contingent feasibility
constraints).

---

## 5. The envelope theorem

The envelope theorem answers: how does the optimum value
change when a parameter changes? This is the foundation for
comparative statics.

### 5.1 Setup and statement

Consider a parameterized problem:
$$V(\theta) = \max_{x \in C} f(x, \theta),$$
where $\theta \in \mathbb{R}^k$ is a parameter and
$x^*(\theta)$ is the optimizer at parameter $\theta$.

**Theorem (Envelope theorem, unconstrained).** If $f$ is
differentiable in $\theta$, the unconstrained optimum
$x^*(\theta)$ is unique and differentiable in $\theta$, and
$C$ does not depend on $\theta$, then
$$\frac{dV}{d\theta} = \frac{\partial f(x^*(\theta), \theta)}{\partial \theta}.$$

**Proof.** By the chain rule:
$$\frac{dV}{d\theta} = \frac{\partial f}{\partial x}\bigg|_{x^*(\theta)} \cdot \frac{dx^*}{d\theta} + \frac{\partial f}{\partial \theta}\bigg|_{x^*(\theta)}.$$
At an interior optimum, $\frac{\partial f}{\partial x}\big|_{x^*(\theta)} = 0$ (first-order condition). The
first term vanishes:
$$\frac{dV}{d\theta} = \frac{\partial f}{\partial \theta}\bigg|_{x^*(\theta)}. \qquad \blacksquare$$

The content: the total derivative of the value function
equals the partial derivative of the objective, holding
$x$ fixed at its optimum. The indirect effect through
$x^*(\theta)$ is zero to first order because at an optimum,
a small change in $x$ has no first-order effect on $f$.

### 5.2 Constrained version

For the constrained problem with $h(x, \theta) = 0$ binding:
$$\frac{dV}{d\theta} = \frac{\partial \mathcal{L}}{\partial \theta}\bigg|_{x^*(\theta), \mu^*(\theta)} = \frac{\partial f}{\partial \theta} + \mu^{*\top} \frac{\partial h}{\partial \theta},$$
evaluated at the optimum. The shadow prices $\mu^*$ capture
how the tightening or loosening of constraints (as $\theta$
moves) affects the value.

The proof is the same: apply the chain rule to $V(\theta) =
\mathcal{L}(x^*(\theta), \mu^*(\theta), \theta)$ and use
the stationarity condition $\nabla_x \mathcal{L} = 0$ and
the feasibility condition $h(x^*(\theta), \theta) = 0$ to
kill the indirect terms.

### 5.3 Applications in economics

**Shephard's lemma.** The derivative of the cost function
$C(w, q) = \min_x \{w^\top x : f(x) \geq q\}$ with respect
to $w_i$ equals the conditional input demand $x_i^*(w, q)$.
This is the envelope theorem applied to the cost-minimization
program: the objective $w^\top x$ depends on $w$ only
directly (not through the feasible set), so $\partial C / \partial w_i = x_i^*$ without any indirect terms.

**Roy's identity.** The Marshallian demand for good $i$ is
$$x_i(p, m) = -\frac{\partial V / \partial p_i}{\partial V / \partial m},$$
where $V(p, m)$ is the indirect utility function. This
follows from applying the envelope theorem to the utility-
maximization problem and using the budget constraint as the
equality constraint.

**The score and information matrix.** In MLE, the
first-order condition at the MLE is $\frac{1}{n}\sum_i
\nabla_\theta \log p(x_i; \hat\theta) = 0$. The expected
score is zero at the true $\theta_0$: $E[\nabla_\theta \log
p(X; \theta_0)] = 0$. This follows from differentiating the
identity $\int p(x; \theta)\,dx = 1$ with respect to $\theta$
(an envelope-theorem-style calculation) — the interchange of
differentiation and integration is justified by dominated
convergence ([[`measure-theory.md`]] §6.3).

---

## 6. Econometric applications

### 6.1 OLS

The objective $f(\beta) = \|y - X\beta\|^2 = y^\top y - 2\beta^\top X^\top y + \beta^\top X^\top X \beta$
has Hessian $\nabla^2 f = 2X^\top X \succeq 0$ (PSD). Under
full column rank of $X$, the Hessian is PD, $f$ is strictly
convex, and the unique minimizer is $\hat\beta = (X^\top X)^{-1}X^\top y$. No local optima. No
numerical issues beyond the conditioning of $X^\top X$ (§7.4).

### 6.2 Maximum likelihood estimation

Under the exponential family, the negative log-likelihood is
convex in the natural parameters. Logistic regression,
Poisson regression, linear regression — all convex. MLE is a
global-optima problem computationally.

Outside the exponential family — mixture models, hidden
Markov models, structural models with latent variables — the
likelihood surface is generically non-convex and multimodal.
EM algorithms exploit the complete-data structure (the
complete-data log-likelihood is convex or at least simpler)
to make progress, but convergence to the global maximum is
not guaranteed. Multiple restarts and model-specific theory
are needed.

The *information matrix equality* states that at the true
$\theta_0$:
$$\mathcal{I}(\theta_0) = -E[\nabla^2_\theta \log p(X; \theta_0)] = E\!\left[\nabla_\theta \log p(X; \theta_0) \cdot \nabla_\theta \log p(X; \theta_0)^\top\right].$$
Both the left side (negative expected Hessian) and right
side (outer product of scores) equal the Fisher information.
The equality follows from differentiating the score condition
$E[\nabla_\theta \log p(X; \theta)] = 0$ with respect to
$\theta$ and exchanging differentiation and expectation — the
interchange is dominated convergence.

### 6.3 GMM and its local-optima problem

For linear IV, the moment conditions $g(\theta) = Z^\top(y -
X\theta)/n$ are affine in $\theta$ and the GMM objective is
convex. GMM is equivalent to 2SLS and is computationally
easy.

For nonlinear moment conditions, $Q(\theta) = g(\theta)^\top
W g(\theta)$ is generically non-convex. Local minima are real
and can be numerous. Standard practice: multiple random
starting values, report the lowest objective across starts,
report sensitivity of the estimate to starting values.

The deeper problem is that multiple local minima in GMM can
signal partial identification: the moment conditions are
insufficiently informative to pin down all components of
$\theta$ uniquely. When the global minimum has a small value
of $Q$ but so do several local minima, the parameter is
likely weakly identified. This connects to the
identification-first posture in [[`00_Identity/Principles.md`]]
and the weak-instrument discussion in
[[`10_Methods/Econometrics/instrumental-variables.md`]].

### 6.4 Structural estimation and constrained optimization

**Nested fixed-point problems.** The BLP demand model
requires, for each candidate $\theta$, solving a fixed-point
problem to recover market-share contraction, then matching
those shares to data. The outer loop over $\theta$ is a
non-convex optimization; the inner loop is guaranteed to
converge by the contraction mapping theorem
([[`real-analysis.md`]] §6). This structure — contraction in
the inner loop, gradient-based search in the outer loop —
is common in structural estimation.

**Constrained maximum likelihood.** Equilibrium conditions
imposed as equality constraints enter as $h(x, \theta) = 0$
in the Lagrangian. The FOC combining the score and the
constraint gradients are exactly the Lagrangian first-order
conditions from §3.1. The shadow prices $\mu^*$ are nuisance
parameters being profiled out.

**Calibration and moment matching.** Calibrating a DSGE model
to match moments is GMM with model-generated moments
replacing the estimator. The non-convexity of the model's
solution map makes this a hard global optimization problem.
The standard approach: solve the model on a grid over
$\theta$, then minimize the distance to target moments.
This is computationally intensive but sidesteps gradient-
based optimization in non-smooth or discontinuous models.

---

## 7. Numerical optimization

Mathematical existence and computational tractability are
different questions.

### 7.1 Newton's method

Newton's method updates
$$x_{t+1} = x_t - [\nabla^2 f(x_t)]^{-1} \nabla f(x_t).$$
This minimizes the quadratic Taylor approximation of $f$
around $x_t$. Key properties:

- **Quadratic convergence** near a strict local minimum:
  $\|x_{t+1} - x^*\| = O(\|x_t - x^*\|^2)$. Dramatically
  faster than gradient descent.
- **Requires the Hessian.** Computing and inverting an
  $n \times n$ Hessian is $O(n^3)$ per step. Infeasible for
  large $n$.
- **Can diverge far from $x^*$** if the quadratic
  approximation is poor. Fix: *damped Newton* or *line-search
  Newton*, reducing the step size (backtracking line search)
  until sufficient descent is achieved.
- **Assumes PD Hessian.** If $\nabla^2 f(x_t)$ is indefinite,
  the Newton step may ascend. Fix: *modified Newton*, adding
  $\delta I$ to the Hessian until PD is achieved.

### 7.2 Quasi-Newton methods (BFGS)

The Broyden-Fletcher-Goldfarb-Shanno algorithm maintains an
approximation $H_t \approx \nabla^2 f(x_t)$ updated from
gradient differences — no Hessian evaluation. The update
uses gradient differences $y_t = \nabla f(x_{t+1}) - \nabla
f(x_t)$ and step directions $s_t = x_{t+1} - x_t$:
$$H_{t+1} = H_t - \frac{H_t s_t s_t^\top H_t}{s_t^\top H_t s_t} + \frac{y_t y_t^\top}{y_t^\top s_t}.$$

BFGS has superlinear convergence — faster than gradient
descent, slightly slower than Newton — using only gradient
evaluations. It is the default general-purpose optimizer in
R (`optim(..., method = "BFGS")`) and Python
(`scipy.optimize.minimize(..., method = "BFGS")`). For
large-$n$ problems, *L-BFGS* stores only the last $m$
gradient pairs rather than the full $n \times n$ matrix,
reducing memory from $O(n^2)$ to $O(mn)$.

### 7.3 Gradient descent

Gradient descent: $x_{t+1} = x_t - \alpha_t \nabla f(x_t)$.
Linear convergence for strongly convex problems; much slower
than BFGS. Its advantages are simplicity and scalability. *Stochastic gradient descent* (SGD) replaces the full gradient
with a gradient computed on one observation or a minibatch,
making each step $O(1)$ in $n$ — essential for neural
networks and high-dimensional flexible estimation.

In standard econometric estimation (OLS, convex MLE with
moderate $n$), gradient descent is usually not the right
tool. It becomes relevant for ML-based estimation (DML
nuisance functions, neural-net-based structural models)
where the parameter space is large and Hessian computation
is infeasible.

### 7.4 Common failure modes

**Starting values.** For non-convex objectives, the starting
point determines which local optimum is found. Good practice:
run the optimizer from many random starting points and report
the global best; or use a global search algorithm (simulated
annealing, differential evolution) for a first pass, then
refine with a gradient-based method. Theoretically motivated
starting values (method-of-moments pre-estimates,
concentrated likelihood) often outperform random starts in
structural estimation.

**Scaling.** Optimization algorithms are sensitive to the
scale of variables. If $\theta_1 \in [0, 1]$ and $\theta_2
\in [10^4, 10^6]$, gradient-based methods oscillate because
the gradient is much larger in the $\theta_2$ direction.
Fix: normalize parameters to comparable scale before
optimization. This is the most common overlooked source of
convergence failure in empirical work.

**"Didn't converge" vs "converged to garbage."** A
convergence code of 0 (success) does not mean the solution
is a global minimum or even a proper local minimum. The
algorithm may have stopped at a saddle point (gradient norm
below tolerance but Hessian indefinite), or on hitting the
iteration limit. Standard checks: verify the Hessian is PD
at the reported optimum; verify the gradient is numerically
zero; run from multiple starting points and confirm the same
solution is recovered.

**Numerical vs mathematical equivalence.** Mathematically
equivalent formulations can have wildly different numerical
behavior. The textbook OLS formula $(X^\top X)^{-1}X^\top y$
is correct in exact arithmetic but fragile numerically:
near-multicollinearity in $X$ produces near-singular $X^\top
X$ and the inversion squares the condition number. The
correct numerical implementation is via QR factorization
of $X$ directly — $X = QR$ with $Q$ orthonormal, then
$\hat\beta = R^{-1} Q^\top y$ — which avoids squaring the
condition number. Most statistical libraries (R's `lm`,
Python's `numpy.linalg.lstsq`, LAPACK) do this internally.
The point is to not reimplement OLS by hand using the normal
equations.

The condition number of a matrix $A$ is $\kappa(A) =
\|A\|\|A^{-1}\|$. If $\kappa(X^\top X) \approx 10^{10}$,
then approximately 10 significant digits are lost in the
inversion — catastrophic for double-precision arithmetic
(which has about 15-16 significant digits). Near-
multicollinearity appears as near-singularity in $X^\top X$
and as numerical instability in estimates. The fix is not
to delete variables blindly but to ask whether the
collinearity reflects a near-identification problem.

---

## 8. What this file doesn't cover, and where to find it

**Infinite-dimensional optimization.** The calculus of
variations (minimizing functionals over function spaces,
Euler-Lagrange equations) and infinite-dimensional Lagrangian
duality require functional analysis — Banach and Hilbert
spaces, dual spaces, the Hahn-Banach theorem. These go in
[[`functional-analysis.md`]]. Stochastic control and optimal
stopping also sit here.

**Dynamic programming.** The Bellman equation is a fixed-point
problem (contraction mapping argument from [[`real-analysis.md`]]
§6) combined with an optimization step (the Bellman
maximization). Value function iteration, policy function
iteration, projection methods, the curse of dimensionality,
and the connection to HANK models go in
[[`dynamic-programming.md`]].

**Stochastic optimization.** Objectives of the form
$\min_\theta E[f(x, \theta)]$ underlie MLE and semiparametric
estimation as population problems. The uniform LLN — that
the sample objective converges uniformly to the population
objective — is what licenses replacing expectations with
sample averages and finding the argmin. This is in
[[`probability.md`]].

**Subgradients and non-smooth optimization.** Convex functions
need not be differentiable everywhere ($\ell^1$ norm, lasso
penalty). Subgradient methods, proximal algorithms, and
ADMM handle non-smooth convex objectives. These arise in
lasso, ridge, and regularized estimators. The treatment of
subdifferential calculus in Banach spaces is in
[[`functional-analysis.md`]].

**Integer programming.** Not used in standard econometrics.
Not covered.

---

## Cross-references

- [[`20_Math/real-analysis.md`]] — prerequisite. Compactness and
  the extreme value theorem (§2–3) underlie existence of
  optima; the implicit function theorem (§5) underlies
  comparative statics and the structure of first-order
  conditions; the contraction mapping theorem (§6) underlies
  nested fixed-point problems in structural estimation.
- [[`20_Math/measure-theory.md`]] — the dominated convergence
  theorem, used to justify differentiating under the integral
  in the score and information matrix discussion (§6.2).
- [[`20_Math/functional-analysis.md`]] — The Hahn-Banach theorem
  there (§5.5) is the infinite-dimensional engine behind the
  duality here (§4); the $L^2$ projection and Riesz
  representation underlie the semiparametric efficiency
  framework that grounds the MLE and GMM asymptotic theory.
- [[`20_Math/dynamic-programming.md`]] — Bellman equation as
  a fixed-point-plus-optimization problem; value function
  iteration; HANK model solution.
- [[`20_Math/probability.md`]] — Uniform LLN and consistency of
  MLE and GMM (§§7.1-7.2 there); the score function and
  information matrix in MLE asymptotics; the Cramér-Rao lower
  bound as $I(\theta_0)^{-1}$; stochastic order notation
  for tracking remainder terms in asymptotic proofs.
- [[`10_Methods/Econometrics/instrumental-variables.md`]] — the
  overidentified GMM objective (§3.3 of that file) is a
  constrained optimization problem whose structure is covered
  here in §6.3.
- [[`10_Methods/Econometrics/local-projections.md`]] — HAC and
  Newey-West variance estimators are GMM-adjacent; the LP
  long-horizon bias correction is an MLE-style correction.
- [[`50_Workflows/run-a-regression-properly.md`]] — Stage 4
  (identification) and Stage 7 (estimation) of the regression
  workflow presuppose the GMM and MLE optimization structure
  covered here.
- [[`00_Identity/Principles.md`]] — §5 (estimation) discusses
  when structural optimization matters and when reduced-form
  estimation is preferable; this file provides the
  mathematical backing.

---

*Last updated: April 2026.*