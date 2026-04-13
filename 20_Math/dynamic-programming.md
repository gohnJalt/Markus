---
tags: [math, dynamic-programming, foundations, macro]
related: [real-analysis, optimization, calculus, differential-equations, stochastic-processes, linear-algebra, probability, modern-toolkit-references, Principles]
---

# Dynamic Programming

This file is the applied endpoint of the math layer for macro
and structural economics. The foundational results it rests on
are already in place: the contraction mapping theorem
([[`real-analysis.md`]] §6), the envelope theorem ([[`optimization.md`]]
§5), Markov chain theory ([[`stochastic-processes.md`]] §9), the
HJB equation and verification theorem ([[`differential-equations.md`]]
§§5-6), and the Bellman equation introduced in [[`calculus.md`]]
§6.4. Dynamic programming is what happens when those pieces
are assembled into a coherent theory of sequential decision
making under uncertainty.

The file covers: the principle of optimality and the recursive
structure of dynamic programming problems (§1); the Bellman
operator and the Blackwell sufficient conditions that guarantee
it is a contraction (§2); the theorem of the maximum and the
Benveniste-Scheinkman result on value function differentiability
(§3); value function iteration and policy function iteration as
numerical algorithms (§4); the continuous state space problem,
the curse of dimensionality, and the standard mitigation
strategies (§5); stochastic dynamic programming and the
stationary distribution under the optimal policy (§6); and
heterogeneous-agent models (Bewley-Huggett-Aiyagari and HANK)
as the structural macro application that draws on all of the
above (§7).

Prerequisites: [[`real-analysis.md`]] §6 (contraction mapping
theorem — the engine of §2); [[`optimization.md`]] §§2-5 (FOC,
SOC, envelope theorem); [[`calculus.md`]] §§2-3 (chain rule and
Taylor — used in Benveniste-Scheinkman and perturbation
methods); [[`stochastic-processes.md`]] §9 (Markov chains and
their ergodic theory); [[`differential-equations.md`]] §§5-7
(HJB, verification theorem, Fokker-Planck — the continuous-
time counterparts).

---

## 1. The principle of optimality

### 1.1 Sequential problems and the recursive structure

A **sequential decision problem** has three ingredients: a
**state space** $\mathcal{X}$ (the set of possible states of
the world), a **control space** $\mathcal{U}$ (the set of
available actions), and a **law of motion** $x_{t+1} =
f(x_t, u_t)$ or a stochastic transition kernel $P(x_{t+1}
\mid x_t, u_t)$. At each period, the agent observes the
current state $x_t$, chooses a control $u_t$ subject to a
feasibility constraint $u_t \in \Gamma(x_t)$ (the
**correspondence of feasible actions**), receives payoff
$r(x_t, u_t)$, and the state transitions.

The agent's problem over an infinite horizon with discount
factor $\beta \in (0, 1)$ is:
$$\max_{\{u_t\}_{t=0}^\infty} \sum_{t=0}^\infty \beta^t
r(x_t, u_t), \quad \text{s.t.}\quad x_{t+1} = f(x_t, u_t),
\quad u_t \in \Gamma(x_t), \quad x_0 \text{ given}.$$

**Bellman's principle of optimality** (1957): an optimal
strategy has the property that, whatever the initial state
and initial decision are, the remaining decisions must form
an optimal strategy with respect to the state resulting from
the first decision. In the recursive language: if
$(u_0^*, u_1^*, u_2^*, \ldots)$ is optimal from $x_0$, then
$(u_1^*, u_2^*, \ldots)$ is optimal from $x_1^* = f(x_0, u_0^*)$.

This observation converts the infinite-horizon sequence
problem into a **functional equation** — the Bellman equation:
$$V(x) = \max_{u \in \Gamma(x)} \bigl\{r(x, u) + \beta
V(f(x, u))\bigr\}.$$
The **value function** $V: \mathcal{X} \to \mathbb{R}$ maps
each state to the maximum lifetime payoff achievable from
that state. The **policy function** $g: \mathcal{X} \to
\mathcal{U}$ maps each state to the optimal action.

The power of the principle of optimality is to reduce an
infinite-dimensional optimization (over infinite sequences
of controls) to a fixed-point problem in a function space.
Whether that fixed-point problem is well-posed — whether $V$
exists, is unique, and is computable — is the content of the
next two sections.

### 1.2 Finite horizon problems

In a **$T$-period problem**, the Bellman equation runs
backward from the terminal condition $V_T(x) = W(x)$ (a
terminal value or scrap value):
$$V_t(x) = \max_{u \in \Gamma(x)}\bigl\{r(x, u) + \beta
V_{t+1}(f(x, u))\bigr\}, \quad t = T-1, T-2, \ldots, 0.$$
Existence and uniqueness of $V_t$ at each step is immediate:
given $V_{t+1}$, the right-hand side is a well-posed
optimization problem. The finite-horizon case requires no
functional equation theory — it is just backward induction.

The infinite-horizon case is the harder and more interesting
one. The sequence $(V_T, V_{T-1}, \ldots)$ does not have an
obvious limit as $T \to \infty$; whether it converges and
what it converges to is exactly the content of the Bellman
operator theory.

---

## 2. The Bellman operator and Blackwell's conditions

### 2.1 The Bellman operator

Define the **Bellman operator** $T$ on the space of bounded
functions $B(\mathcal{X})$ (with the sup norm
$\|V\|_\infty = \sup_{x \in \mathcal{X}} |V(x)|$) by:
$$(TV)(x) = \max_{u \in \Gamma(x)}\bigl\{r(x, u) +
\beta V(f(x, u))\bigr\}.$$
A solution to the Bellman equation is a fixed point of $T$:
$V^* = TV^*$.

The space $(B(\mathcal{X}), \|\cdot\|_\infty)$ is a complete
metric space ([[`real-analysis.md`]] §7 — uniform limits of
bounded functions are bounded; uniform Cauchy sequences
converge uniformly). If $T$ is a contraction on this space,
the contraction mapping theorem ([[`real-analysis.md`]] §6)
guarantees: a unique fixed point $V^*$, convergence of
the sequence $T^n V_0 \to V^*$ from any starting $V_0$,
and geometric rate of convergence with constant $\beta$.

The question is: when is $T$ a contraction? The discount
factor $\beta < 1$ suggests it should be, but the $\max$
operation makes verification non-trivial. Blackwell's
conditions give a clean sufficient criterion.

### 2.2 Blackwell's sufficient conditions

**Theorem (Blackwell 1965).** An operator $T: B(\mathcal{X})
\to B(\mathcal{X})$ is a contraction with modulus $\beta$
if it satisfies:

1. **Monotonicity:** $V \leq W$ (pointwise) implies
   $TV \leq TW$ (pointwise).
2. **Discounting:** There exists $\beta \in (0, 1)$ such
   that for any $V \in B(\mathcal{X})$ and constant
   $c \geq 0$: $T(V + c) \leq TV + \beta c$, where
   $(V + c)(x) := V(x) + c$.

**Proof.** Let $V, W \in B(\mathcal{X})$ and let $c =
\|V - W\|_\infty$. Then $V(x) \leq W(x) + c$ for all $x$,
i.e., $V \leq W + c$ pointwise. Applying monotonicity:
$TV \leq T(W + c)$. Applying discounting: $T(W + c) \leq
TW + \beta c$. Together: $TV \leq TW + \beta c$. By
symmetry (applying the same argument with $V$ and $W$
swapped): $TW \leq TV + \beta c$. So $|(TV)(x) - (TW)(x)|
\leq \beta c$ for all $x$, giving $\|TV - TW\|_\infty
\leq \beta \|V - W\|_\infty$. $\blacksquare$

**Verifying Blackwell's conditions for the Bellman operator.**

*Monotonicity:* If $V \leq W$ pointwise, then $V(f(x,u))
\leq W(f(x,u))$ for all $x, u$. Therefore:
$$(TV)(x) = \max_{u \in \Gamma(x)}\{r(x,u) + \beta V(f(x,u))\}
\leq \max_{u \in \Gamma(x)}\{r(x,u) + \beta W(f(x,u))\}
= (TW)(x). \checkmark$$

*Discounting:* $(T(V+c))(x) = \max_{u \in \Gamma(x)}\{r(x,u)
+ \beta(V(f(x,u)) + c)\} = \max_{u \in \Gamma(x)}\{r(x,u)
+ \beta V(f(x,u))\} + \beta c = (TV)(x) + \beta c$. So
$T(V+c) = TV + \beta c \leq TV + \beta c$. $\checkmark$

Both conditions hold, so the Bellman operator is a
contraction with modulus $\beta$. The contraction mapping
theorem ([[`real-analysis.md`]] §6) gives:

1. **Existence and uniqueness of $V^*$**: there is a unique
   bounded function satisfying the Bellman equation.
2. **Convergence of VFI**: $\|T^n V_0 - V^*\|_\infty \leq
   \beta^n \|TV_0 - V_0\|_\infty / (1 - \beta) \to 0$.
3. **Rate**: convergence is geometric with rate $\beta$, so
   100 iterations with $\beta = 0.95$ reduces the error by
   a factor of $0.95^{100} \approx 0.006$.

### 2.3 The stochastic Bellman equation

When the state transition is stochastic — $x_{t+1}$ is drawn
from $P(\cdot \mid x_t, u_t)$ — the Bellman equation becomes:
$$V(x) = \max_{u \in \Gamma(x)}\bigl\{r(x, u) + \beta
E_{x' \sim P(\cdot \mid x, u)}[V(x')]\bigr\}.$$
The Bellman operator is now:
$$(TV)(x) = \max_{u \in \Gamma(x)}\bigl\{r(x, u) + \beta
\int V(x')\, P(dx' \mid x, u)\bigr\}.$$
Blackwell's conditions hold identically: monotonicity because
$V \leq W$ implies $\int V\, dP \leq \int W\, dP$
(monotonicity of the Lebesgue integral,
[[`measure-theory.md`]] §4); discounting because the integral
of a constant is the constant. So the stochastic Bellman
operator is also a contraction with modulus $\beta$, and all
of §2.2 carries over.

---

## 3. Optimal policies: existence, continuity, differentiability

### 3.1 The theorem of the maximum

The Bellman operator argument establishes that $V^*$ exists.
The next question is whether the optimizing control $g(x) =
\arg\max_{u \in \Gamma(x)} \{r(x,u) + \beta V^*(f(x,u))\}$
exists and varies continuously with $x$.

**Theorem of the maximum (Berge 1963).** Let
$\phi: \mathcal{X} \to \mathbb{R}$ be the value function
of $\phi(x) = \max_{u \in \Gamma(x)} h(x, u)$, where
$h: \mathcal{X} \times \mathcal{U} \to \mathbb{R}$ is
continuous and $\Gamma: \mathcal{X} \rightrightarrows
\mathcal{U}$ is a continuous correspondence with nonempty
compact values. Then:
1. $\phi$ is continuous.
2. The **policy correspondence** $G(x) = \arg\max_{u \in
   \Gamma(x)} h(x, u)$ is nonempty, compact-valued, and
   **upper hemicontinuous** (if $x_n \to x$ and $u_n \in
   G(x_n)$, then any limit point of $(u_n)$ lies in $G(x)$).

**Proof sketch.** *Continuity of $\phi$:* Upper semicontinuity
is immediate — $\{x : \phi(x) > c\} = \pi_\mathcal{X}(\{(x,u):
h(x,u) > c, u \in \Gamma(x)\})$ is open by continuity of $h$
and openness of the graph of $\Gamma$. Lower semicontinuity
uses continuity of $\Gamma$ (lower hemicontinuity): for any
$u_0 \in G(x_0)$ and $x_n \to x_0$, lower hemicontinuity of
$\Gamma$ provides $u_n \in \Gamma(x_n)$ with $u_n \to u_0$;
then $\phi(x_n) \geq h(x_n, u_n) \to h(x_0, u_0) = \phi(x_0)$.
*UHC of $G$:* suppose $u_n \in G(x_n)$ and $u_n \to u$.
By compactness of $\Gamma(x)$ and UHC of $\Gamma$, $u \in
\Gamma(x)$. For any $v \in \Gamma(x)$, lower hemicontinuity
gives $v_n \in \Gamma(x_n)$ with $v_n \to v$. Then
$h(x_n, u_n) \geq h(x_n, v_n)$; taking limits, $h(x,u) \geq
h(x,v)$. So $u \in G(x)$. $\blacksquare$

Applied to the Bellman equation: taking $h(x, u) = r(x, u)
+ \beta V^*(f(x, u))$, the theorem of the maximum guarantees
the value function $V^*$ is continuous (if $r$, $f$, and
$\Gamma$ are continuous and compact-valued) and the policy
correspondence is upper hemicontinuous. If $G(x)$ is single-
valued (strict concavity of $h$ in $u$), it is continuous,
and the policy function $g(x)$ is well-defined and continuous.

### 3.2 Benveniste-Scheinkman: differentiability of $V^*$

The contraction mapping theorem guarantees $V^*$ exists and
is continuous. Differentiability is a further property needed
for the envelope theorem, the first-order conditions on the
optimal policy, and perturbation methods.

**Theorem (Benveniste-Scheinkman 1979).** Suppose $V^*$ is
the unique bounded continuous solution of the Bellman equation,
$r$ and $f$ are $C^1$, and the policy function $g$ is
single-valued and continuous. If $V^*$ is concave, then
$V^*$ is differentiable and:
$$\nabla_x V^*(x) = \nabla_x r(x, g(x)) + \beta\,
Df(x, g(x))^\top \nabla_x V^*(f(x, g(x))).$$

**Proof.** The key observation: by the envelope theorem
([[`optimization.md`]] §5), the derivative of $V^*$ with respect
to $x$ at a point where $g(x)$ is optimal does not account
for the change in $g$ (the indirect effect through the
optimizer vanishes at the optimum). Define $h(x) =
r(x, g(x)) + \beta V^*(f(x, g(x)))$. Since $g(x)$ is
feasible but not necessarily optimal when the state is perturbed,
$h(x') \leq V^*(x')$ for $x'$ near $x$, with equality at $x$.
So $V^* - h$ achieves a local minimum of $0$ at $x$, giving
$\nabla(V^* - h)(x) = 0$, i.e., $\nabla V^*(x) = \nabla h(x)$.
Computing $\nabla h$ by the chain rule gives the formula. $\blacksquare$

**The Euler equation.** The FOC for the Bellman equation (when
the optimal $u = g(x)$ is interior) is:
$$\nabla_u r(x, g(x)) + \beta\,\nabla_u f(x, g(x))^\top
\nabla_x V^*(f(x, g(x))) = 0.$$
Substituting the Benveniste-Scheinkman formula for
$\nabla_x V^*$ on the right-hand side and using the law of
motion to connect the next period's state to the current
period gives the **Euler equation** — the standard
first-order condition for discrete-time dynamic programs.
For the Ramsey model: $u'(c_t) = \beta u'(c_{t+1})
(1 + f'(k_{t+1}) - \delta)$ is the Euler equation connecting
today's marginal utility of consumption to tomorrow's.

---

## 4. Value function iteration and policy function iteration

### 4.1 Value function iteration

**Value function iteration (VFI)** is the direct implementation
of the Picard iteration for the Bellman operator: start with
an arbitrary $V_0 \in B(\mathcal{X})$ (usually $V_0 \equiv 0$
or the terminal value) and iterate:
$$V_{n+1} = TV_n.$$
By the contraction mapping theorem, $\|V_n - V^*\|_\infty
\leq \beta^n \|V_1 - V_0\|_\infty / (1 - \beta)$. The error
is halved every $\log(2)/\log(1/\beta)$ iterations.

At each iteration, $TV_n$ requires solving:
$$\max_{u \in \Gamma(x)} \{r(x, u) + \beta V_n(f(x, u))\}$$
at each $x$ — a static optimization problem given the
current guess $V_n$.

**Stopping criterion.** The standard criterion is
$\|V_{n+1} - V_n\|_\infty < \varepsilon(1-\beta)/\beta$,
which guarantees $\|V_n - V^*\|_\infty < \varepsilon$. This
follows from the triangle inequality and the contraction
bound: $\|V_n - V^*\| \leq \|V_n - V_{n+1}\|/(1-\beta)$.

### 4.2 Policy function iteration

**Policy function iteration (Howard improvement)** alternates
between two steps:

1. **Policy evaluation:** given a policy $g_n$, find the
   value function $V^{g_n}$ of that policy by solving the
   linear system $V^{g_n}(x) = r(x, g_n(x)) + \beta
   V^{g_n}(f(x, g_n(x)))$ for all $x$ simultaneously.
2. **Policy improvement:** update the policy by maximizing:
   $g_{n+1}(x) = \arg\max_{u \in \Gamma(x)} \{r(x,u) +
   \beta V^{g_n}(f(x,u))\}$.

**Theorem (Policy improvement).** $V^{g_{n+1}} \geq V^{g_n}$
pointwise, with equality only if $g_n$ is already optimal.

**Proof.** $V^{g_{n+1}}(x) \geq r(x, g_{n+1}(x)) + \beta
V^{g_n}(f(x, g_{n+1}(x))) \geq r(x, g_n(x)) + \beta
V^{g_n}(f(x, g_n(x))) = V^{g_n}(x)$. The first inequality
applies the definition of $V^{g_{n+1}}$ as a fixed point from
below; the second uses the fact that $g_{n+1}$ maximizes over
$g_n$. The argument can be iterated to give $V^{g_{n+1}} \geq
V^{g_n}$; the sequence of value functions is monotone and
bounded, so it converges. $\blacksquare$

**VFI vs PFI in practice.** PFI converges in far fewer
outer iterations than VFI — often 5-20 iterations vs hundreds
for VFI — because the policy evaluation step uses the exact
value of the current policy rather than a one-step approximation.
The cost is that each policy evaluation step requires solving
a linear system (on a discrete grid: a matrix equation of
size $|\mathcal{X}| \times |\mathcal{X}|$, solved by sparse
LU or direct methods). For small-to-medium state spaces, PFI
dominates. For large state spaces where the linear solve
becomes expensive, a hybrid (**modified policy iteration**) —
doing $k$ VFI steps in the evaluation phase rather than full
convergence — is often preferred.

---

## 5. Continuous state spaces and the curse of dimensionality

### 5.1 Discretization and the curse

In practice, $\mathcal{X}$ is typically a continuous set
(e.g., asset holdings $a \in [0, \bar a]$ and income
$y \in \mathcal{Y}$). To implement VFI, the continuous state
space is approximated by a discrete **grid** of $N$ points.
The Bellman operator becomes a finite-dimensional map, and
VFI iterates over the $N$-vector of values.

**The curse of dimensionality** (Bellman 1957): the number
of grid points grows exponentially in the state dimension.
A 1-D problem with $N = 200$ points, a 2-D problem with
$N^2 = 40{,}000$ points, a 3-D problem with $N^3 = 8$ million
points — each dimension adds an order of magnitude. For
state vectors of dimension 5 or more, uniform grids become
computationally infeasible.

### 5.2 Function approximation

The standard alternative to full grids: approximate $V$ by
a parametric family $\hat V(\cdot; \theta)$ and choose
$\theta$ to minimize the Bellman residual or to match $V$
at a set of nodes.

**Piecewise linear interpolation.** On a grid
$\{x_1, \ldots, x_N\}$, define $\hat V$ as the piecewise
linear function through the grid values. The Bellman operator
is applied at the grid points and the result is interpolated
off-grid. Computationally simple but converges slowly
(first-order in the grid spacing) and produces kinks in
the policy function.

**Cubic spline interpolation.** Fit a cubic spline through
the grid values — $C^2$ globally, no kinks. Convergence
is fourth-order in grid spacing. The standard choice for
1-D problems and for smooth value functions. `scipy.interpolate`
and R's `splinefun` implement this. The cost: the spline
requires solving a tridiagonal system at each iteration,
which is $O(N)$.

**Chebyshev polynomials.** Approximate $V \approx \sum_{k=0}^K
\theta_k T_k(x)$ where $T_k$ are Chebyshev polynomials.
The **Chebyshev nodes** $x_k = \cos(\pi(2k-1)/(2K))$ are
the optimal approximation points (they minimize the Runge's
phenomenon that plagues equally-spaced nodes — the same
issue flagged in [[`regression-discontinuity.md`]] §4 for
global polynomial regression). Convergence is exponential
in $K$ for smooth $V$. This is the workhorse of projection
methods for smooth problems.

**Projection methods (collocation).** More generally: choose
a basis $\{\phi_k\}_{k=1}^K$ and coefficients $\theta$
such that the Bellman residual $T\hat V - \hat V = 0$ is
satisfied exactly at $K$ **collocation points**. The
collocation equation is a system of $K$ nonlinear equations
in $K$ unknowns $\theta$, solved iteratively. Chebyshev
collocation on Chebyshev nodes is the canonical form.

### 5.3 Mitigation strategies

**Endogenous grid method (EGM, Carroll 2006).** For problems
where the first-order condition $r_u + \beta V'(f(x,u))
f_u = 0$ can be inverted analytically — i.e., for any
tomorrow's state $x'$, we can solve backward for today's
optimal control $u$ — EGM avoids the inner optimization loop
entirely. Instead of maximizing at each grid point, the
algorithm starts from a grid on next period's state and maps
backward to today's states. The result: VFI with EGM is
orders of magnitude faster than standard VFI for consumption-
savings problems. Most quantitative macro models with a
single asset use EGM.

**Sparse grids.** For $d$-dimensional state spaces, sparse
grids (Smolyak 1963) approximate integrals and functions with
$O(N(\log N)^{d-1})$ points rather than $O(N^d)$ — polynomial
in $N$ with a logarithmic correction, rather than exponential.
The approximation quality is lower than a full tensor-product
grid at the same $N$ but is vastly more tractable for $d \geq 3$.

**Perturbation (linearization) methods.** Linearize the
model around the steady state to get a linear rational
expectations system $x_{t+1} = A x_t + B u_t + C
\varepsilon_{t+1}$. The optimal policy is linear in the
state: $u_t = F x_t$. $F$ is found by solving a discrete
algebraic Riccati equation (a matrix fixed-point problem)
or via the Schur decomposition of the system matrix. The
Dynare software implements this for DSGE models.

*Second-order perturbation* captures precautionary motives
and risk premia by taking a second-order Taylor expansion
of the equilibrium conditions. Third-order and higher-order
perturbation captures larger departures from the steady state.
The connection to Taylor's theorem in [[`calculus.md`]] §3 is
direct: each order of perturbation corresponds to one more
term in the expansion of the policy function around the
steady state.

---

## 6. Stochastic dynamic programming

### 6.1 Markov decision processes

The stochastic counterpart of the deterministic sequential
problem is a **Markov decision process** (MDP): the state
evolves as a controlled Markov chain with transition kernel
$P(dx' \mid x, u)$, and the Bellman equation is as in §2.3.

A **stationary Markov policy** $g: \mathcal{X} \to \mathcal{U}$
induces a Markov chain on $\mathcal{X}$ with transition
kernel $P_g(dx' \mid x) = P(dx' \mid x, g(x))$. Under
conditions that guarantee this chain is ergodic
([[`stochastic-processes.md`]] §9.2), it has a unique stationary
distribution $\pi_g$ — the long-run distribution of states
under the policy $g$.

The ergodic theorem for Markov chains
([[`stochastic-processes.md`]] §9.2) then gives:
$$\frac{1}{T}\sum_{t=0}^{T-1} r(x_t, g(x_t)) \xrightarrow{a.s.}
\int r(x, g(x))\,\pi_g(dx).$$
The time-average payoff under policy $g$ converges to the
integral of the per-period payoff against the stationary
distribution. This is the link between the discounted
Bellman equation (which is about lifetime utility) and the
long-run average behavior of the economy.

### 6.2 The distribution of the state under the optimal policy

Let $g^*$ be the optimal policy. The stationary distribution
$\pi^*$ induced by $P_{g^*}(dx' \mid x) = P(dx' \mid x, g^*(x))$
satisfies:
$$\pi^*(A) = \int_\mathcal{X} P_{g^*}(A \mid x)\,\pi^*(dx)
\quad \text{for all measurable } A.$$
This is the discrete-time analogue of the stationary
Fokker-Planck equation ([[`differential-equations.md`]] §7.3).
Computing $\pi^*$ is needed for calibration, for matching
moments in the data (the stationary distribution of earnings,
wealth, employment status, etc.), and for equilibrium in
heterogeneous-agent models.

In practice: on a discrete grid, $\pi^*$ is the left
eigenvector corresponding to eigenvalue 1 of the transition
matrix induced by the optimal policy. Off-grid, the
**Young (2010) method** tracks the distribution by
propagating a histogram forward under the policy function,
using the same grid and interpolation scheme as VFI.

---

## 7. Heterogeneous-agent models and HANK

### 7.1 The Aiyagari/Bewley structure

The baseline heterogeneous-agent model
(Bewley 1986, Huggett 1993, Aiyagari 1994) has a unit
measure of ex-ante identical agents, each facing
idiosyncratic income risk that is not insurable.

**Individual problem.** Each agent solves:
$$V(a, y) = \max_{c \geq 0,\, a' \geq \underline a}
\{u(c) + \beta E[V(a', y') \mid y]\}$$
subject to the budget constraint $c + a' = (1+r)a + y$,
where $a$ is asset holdings, $y$ is idiosyncratic income
(a Markov chain), $r$ is the interest rate, and $\underline a$
is a borrowing limit.

This is a two-state Bellman equation ($x = (a, y)$,
$u = a'$) of the type analyzed in §§1-3. The policy
functions $c^*(a, y)$ and $a'^*(a,y)$ are the objects
of interest.

**Aggregate equilibrium.** Let $\mu(da, dy)$ be the
joint distribution of agents over $(a, y)$ states. In
a stationary equilibrium:
1. $(V, a'^*, c^*)$ solve the individual Bellman equation
   given prices $(r, w)$.
2. $\mu$ is stationary: $\mu = \mathcal{L}_{a'^*} \mu$,
   where $\mathcal{L}_{a'^*}$ is the operator that maps
   today's distribution to tomorrow's under the optimal policy.
3. Markets clear: $\int a\,\mu(da, dy) = K$ (aggregate
   capital equals aggregate savings), which pins down $r$
   and $w$ via the firm's FOC.

Computing the stationary equilibrium requires solving
the fixed-point problem in prices: given $(r, w)$, solve
the Bellman equation, compute the stationary distribution,
check market clearing, and update prices. The outer loop
is a root-finding problem in a low-dimensional price vector.

### 7.2 The HANK structure

**HANK (Heterogeneous-Agent New Keynesian)** models add
aggregate shocks and nominal rigidities to the Bewley
structure. The key difference: individual decisions now
depend on the entire distribution $\mu$ of agents as a state
variable (because $\mu$ determines aggregate demand, which
feeds back into income and prices).

The state space is therefore infinite-dimensional: the
individual's state is $(a, y)$ as before, but the aggregate
state includes the distribution $\mu$ itself. A full
recursive equilibrium has value functions and policy
functions that depend on $\mu$ — a fixed-point problem in
function spaces, not just finite-dimensional vectors.

**The Krusell-Smith (1998) approximation** handles this by
replacing the full distribution $\mu$ with a small set of
its moments (typically just the mean, i.e., aggregate
capital $K$). Agents are assumed to forecast future prices
using a log-linear rule in $K$; this rule is verified to
be accurate in simulations ("approximate aggregation" holds
when the distribution is close to a family parameterized by
its mean). The approximation is excellent for first-order
dynamics but breaks down for higher-order effects and for
models with strong distributional dynamics.

**The continuous-time approach (Achdou-Han-Lasry-Lions-Moll
2022).** In continuous time, the equilibrium is characterized
by the coupled HJB-Fokker-Planck system described in
[[`differential-equations.md`]] §7.3:
$$-\partial_t V + \rho V = r(a,y) + \mu^*(a,y) \partial_a V
+ \lambda(y)[\bar V(a) - V(a,y)] + \frac{\sigma^2}{2}
\partial_{aa} V,$$
$$\partial_t \mu = \mathcal{A}^* \mu,$$
plus market clearing. The finite-difference method
(upwind scheme) on the asset-income grid solves this system
efficiently. The PDE approach is more tractable analytically
and numerically than the discrete-time Krusell-Smith approach
for models with rich individual dynamics (earnings risk,
portfolio choice, life cycle).

**What HANK adds to RANK.** The monetary transmission
mechanism in a Representative-Agent NK (RANK) model works
primarily through the substitution effect: lower interest
rates incentivize borrowing and consumption today. In HANK,
the income effect (via redistribution of interest income
across net debtors and net creditors) and the indirect
general-equilibrium effect (via employment and wages) can
dominate, reversing or amplifying the representative-agent
prediction. The size of these effects depends on the shape
of the distribution $\mu$ — which is why getting the
distributional dynamics right is not a refinement but a
first-order concern for monetary policy analysis.

---

## 8. What this file doesn't cover, and where to find it

**Continuous-time dynamic programming in full.** The HJB
equation, the verification theorem, and Dynkin's formula
are covered in [[`differential-equations.md`]] §§5-7. The
connection between the discrete-time Bellman equation
(this file) and the continuous-time HJB (that file) is the
limit $\Delta t \to 0$ of the Bellman equation.

**Stochastic control with jumps.** When the state process
has jump components (Poisson arrivals of job loss, death,
technology shocks), the HJB acquires a jump term and the
Fokker-Planck acquires a source/sink term. These models
are common in heterogeneous-agent labor search models.
Not covered here; Pham's *Continuous-time Stochastic Control
and Optimization* is the standard reference.

**Reinforcement learning.** Model-free DP: Q-learning,
policy gradients, actor-critic methods. These are
approximation algorithms for MDPs when the transition kernel
is unknown or the state space is too large for explicit VFI.
The Bellman equation is the theoretical foundation, but the
computational methods (neural network function approximators,
experience replay) are outside this file's scope.

**Optimal stopping.** The secretary problem, American options,
search models with optimal stopping. A special case of DP
where the control at each date is binary (stop or continue).
The value function has a "stopping region" and a "continuation
region" separated by a free boundary. Not covered here; Peskir
and Shiryaev's *Optimal Stopping and Free-Boundary Problems*
is the reference.

---

## Cross-references

- [[`20_Math/real-analysis.md`]] — the contraction mapping
  theorem (§6 there) is the engine of §2 here; completeness
  of $B(\mathcal{X})$ under the sup norm (uniform convergence,
  §7 there) is the function space in which the Bellman
  operator acts.
- [[`20_Math/optimization.md`]] — the envelope theorem (§5
  there) is the proof mechanism behind the Benveniste-
  Scheinkman result (§3.2 here); the convexity and FOC/SOC
  machinery (§§1-3 there) governs the inner maximization
  in the Bellman equation.
- [[`20_Math/calculus.md`]] — the chain rule (§2 there) is
  used in the Benveniste-Scheinkman formula; the Taylor
  expansion (§3 there) is the foundation for perturbation
  methods (§5.3 here); the Hamiltonian and Bellman equation
  connection (§6.4 there) is the continuous-time limit of
  the discrete-time Bellman equation here.
- [[`20_Math/differential-equations.md`]] — the continuous-time
  HJB equation (§6 there) is the limit of the discrete-time
  Bellman equation as $\Delta t \to 0$; the verification
  theorem (§6.2 there) is the PDE analogue of the policy
  improvement theorem (§4.2 here); the Fokker-Planck equation
  (§7.3 there) governs the distribution dynamics in HANK
  (§7.2 here).
- [[`20_Math/stochastic-processes.md`]] — Markov chains and
  their stationary distributions (§9 there) are the building
  blocks of MDPs (§6 here); the ergodic theorem for Markov
  chains (§9.2 there) connects the lifetime-utility Bellman
  equation to the long-run average behavior; the HANK model
  (§7.2 here) is the heterogeneous-agent application
  mentioned in §9.3 there.
- [[`20_Math/linear-algebra.md`]] — the Schur decomposition
  and eigenvalue calculations (§§3-4 there) underlie the
  perturbation method's Riccati equation; the stationary
  distribution as the unit-eigenvector of the transition
  matrix (§6.2 here) is an eigenvalue computation; sparse
  LU for policy evaluation (§4.2 here) uses the
  decomposition guide from §8.3 there.
- [[`20_Math/probability.md`]] — the ULLN (§7.2 there)
  underlies the consistency of simulation-based estimators
  of the stationary distribution; the ergodic theorem for
  Markov chains (§9.2, [[`stochastic-processes.md`]]) is the
  time-average form of the LLN.
- [[`10_Methods/Econometrics/structural-labor.md`]] — Search and
  matching models are MDPs: the worker's value function satisfies
  a Bellman equation over employment states; the Mortensen-
  Pissarides model is a continuous-time
  optimal stopping and matching problem.
- [[`10_Methods/Econometrics/double-machine-learning.md`]] —
  Structural estimation of dynamic models often proceeds by
  matching simulated moments from the model's stationary
  distribution to data moments; the DML approach can handle
  nuisance parameters in these moment conditions.

---

*Last updated: April 2026.*
