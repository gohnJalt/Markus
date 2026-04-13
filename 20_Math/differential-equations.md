---
tags: [math, differential-equations, foundations, dynamic-programming]
related: [real-analysis, calculus, optimization, stochastic-processes, functional-analysis, linear-algebra, dynamic-programming, modern-toolkit-references, Principles]
---

# Differential Equations

This file covers the ODE and PDE theory that the rest of the
math library either uses without proof or explicitly defers.
[[`calculus.md`]] derived the Euler-Lagrange equation and the
Hamilton-Jacobi-Bellman equation as variational conditions —
both are differential equations, and what makes them solvable,
what makes their solutions unique, and when classical solutions
fail are questions that belong here. [[`stochastic-processes.md`]]
introduced the Kolmogorov forward and backward equations as
bridges between SDEs and PDEs. [[`dynamic-programming.md`]] will
use the Bellman operator and its continuous-time version; both
rest on the stability theory developed here.

The file is intentionally smaller than the other math files.
The three threads that matter most for econ work are: ODE
existence and uniqueness via the Picard-Lindelöf theorem, which
is a contraction mapping argument and the right proof to have
on hand (§§1-2); linear systems and phase portrait stability,
which is what equilibrium analysis and VAR dynamics live in
(§§3-4); and the PDE layer of dynamic optimization — the
method of characteristics, the HJB equation, viscosity
solutions, and the Kolmogorov equations for continuous-time
distributional dynamics (§§5-7).

Prerequisites: [[`real-analysis.md`]] (§6, contraction mapping
theorem — the engine of §1 here); [[`calculus.md`]] (§§2-3,
chain rule and Taylor; §§5-6, matrix calculus and the
Hamiltonian/HJB); [[`linear-algebra.md`]] (§§3-4, eigenvalues
and the spectral theorem — used in §§3-4 here);
[[`stochastic-processes.md`]] (§7, Itô calculus — background for
§§6-7 here).

---

## 1. ODEs: existence and uniqueness

### 1.1 The initial value problem

The **initial value problem** (IVP) is:
$$\dot x(t) = f(t, x(t)), \qquad x(t_0) = x_0,$$
where $f: I \times U \to \mathbb{R}^n$ with $I \subseteq
\mathbb{R}$ an interval and $U \subseteq \mathbb{R}^n$ open.
A **solution** is a $C^1$ function $x: J \to U$ (on some
interval $J \subseteq I$ containing $t_0$) satisfying both
the equation and the initial condition.

The key question is: does a solution exist, and if so, is it
unique? The answer depends on how well-behaved $f$ is.

**What can go wrong without good conditions.** (i) *No
solution:* if $f$ is merely measurable (not continuous),
solutions may fail to exist even locally. (ii) *Non-unique
solutions:* $\dot x = |x|^{1/2}$, $x(0) = 0$ has both
$x(t) \equiv 0$ and $x(t) = t^2/4$ as solutions — the right
hand side is continuous but not Lipschitz at $x = 0$, and
uniqueness fails. (iii) *Finite-time blowup:* $\dot x = x^2$,
$x(0) = 1$ has solution $x(t) = 1/(1-t)$, which blows up at
$t = 1$. Local existence does not imply global existence.

### 1.2 The Picard-Lindelöf theorem

**Theorem (Picard-Lindelöf).** Let $f: [t_0 - a, t_0 + a]
\times \overline{B_r(x_0)} \to \mathbb{R}^n$ be continuous,
and suppose $f$ is **Lipschitz in $x$** uniformly in $t$:
there exists $L > 0$ such that
$$\|f(t, x) - f(t, y)\| \leq L\|x - y\|$$
for all $t \in [t_0 - a, t_0 + a]$ and all $x, y \in
\overline{B_r(x_0)}$.

Let $M = \sup\|f\|$ on the domain and $b = \min(a, r/M)$.
Then on $[t_0 - b, t_0 + b]$, the IVP has a **unique
solution**.

**Proof (via Picard iteration, which is a contraction
mapping argument).** Rewrite the IVP as an integral equation:
$$x(t) = x_0 + \int_{t_0}^t f(s, x(s))\, ds.$$
This is equivalent to the IVP whenever $x$ is $C^1$ — it
automatically incorporates the initial condition. Define the
**Picard operator** $T$ on $C([t_0-b, t_0+b], \mathbb{R}^n)$ by
$$(Tx)(t) = x_0 + \int_{t_0}^t f(s, x(s))\, ds.$$
We work on the closed ball $X = \{x \in C([t_0-b,t_0+b],
\mathbb{R}^n) : \|x - x_0\|_\infty \leq r\}$, where
$\|\cdot\|_\infty$ is the sup norm. The space
$(C([t_0-b,t_0+b], \mathbb{R}^n), \|\cdot\|_\infty)$ is
complete ([[`real-analysis.md`]] §7), and $X$ is a closed subset,
hence also complete.

**Step 1: $T$ maps $X$ to $X$.** For $x \in X$:
$$\|(Tx)(t) - x_0\| \leq \int_{t_0}^t \|f(s,x(s))\|\, ds
\leq M|t - t_0| \leq Mb \leq M \cdot \frac{r}{M} = r.$$

**Step 2: $T$ is a contraction on $X$.** For $x, y \in X$:
\begin{align*}
\|(Tx - Ty)(t)\| &\leq \int_{t_0}^t \|f(s,x(s)) - f(s,y(s))\|\,ds \\
&\leq L\int_{t_0}^t \|x(s) - y(s)\|\, ds \\
&\leq L|t - t_0|\,\|x - y\|_\infty \leq Lb\,\|x - y\|_\infty.
\end{align*}
Taking the sup over $t$: $\|Tx - Ty\|_\infty \leq Lb\,\|x-y\|_\infty$.
By choosing $b \leq \min(a, r/M, 1/(2L))$, we get $Lb \leq 1/2 < 1$,
so $T$ is a contraction.

**Step 3: apply the contraction mapping theorem.** The
theorem ([[`real-analysis.md`]] §6) guarantees a unique fixed
point $x^* \in X$: $Tx^* = x^*$. This fixed point is the
unique solution to the integral equation, hence to the IVP.
$\blacksquare$

**The Picard iteration** $x_{n+1} = Tx_n$ (starting from
$x_0(t) \equiv x_0$) converges geometrically to the solution.
This is not just an existence argument — it is a constructive
algorithm. For simple enough $f$, the iteration can be
carried out symbolically. For general $f$, it is the
theoretical basis for Euler's method and its higher-order
variants (Runge-Kutta), which are finite-step truncations of
the Picard iteration.

### 1.3 Gronwall's inequality

The Gronwall inequality is the basic tool for bounding
solutions: it converts "the solution satisfies an integral
inequality" into a pointwise bound.

**Theorem (Gronwall).** Let $u, v, w: [t_0, T] \to [0, \infty)$
be continuous with $v$ nondecreasing, and suppose
$$u(t) \leq v(t) + \int_{t_0}^t w(s) u(s)\, ds.$$
Then
$$u(t) \leq v(t) \exp\!\left(\int_{t_0}^t w(s)\, ds\right).$$

**Proof.** Let $R(t) = \int_{t_0}^t w(s) u(s)\, ds$. Then
$R' = wu \leq w(v + R)$, i.e., $R' - wR \leq wv$.
Multiplying by the integrating factor $e^{-\int_{t_0}^t w}$:
$$(Re^{-\int_{t_0}^t w})' \leq v(t)w(t)e^{-\int_{t_0}^t w}
\leq v(t) w(t).$$
Integrating from $t_0$ to $t$ and using $R(t_0) = 0$:
$$R(t)e^{-\int_{t_0}^t w} \leq v(t)\int_{t_0}^t w(s)\,ds,$$
so $R(t) \leq v(t)(e^{\int_{t_0}^t w} - 1)$. Then
$u(t) \leq v(t) + R(t) \leq v(t) e^{\int_{t_0}^t w}$. $\blacksquare$

**Standard use.** If two solutions $x, y$ of the same IVP
satisfy $\|x(t) - y(t)\| \leq L\int_{t_0}^t \|x(s) - y(s)\|\,ds$
(which follows from the Lipschitz condition), then Gronwall
with $v = 0$ gives $\|x(t) - y(t)\| \leq 0$, reproving
uniqueness. More usefully: if $x, y$ have slightly different
initial conditions $\|x(t_0) - y(t_0)\| = \varepsilon$,
Gronwall gives $\|x(t) - y(t)\| \leq \varepsilon e^{L(t-t_0)}$
— solutions with close initial conditions stay close on
finite intervals. This is **continuous dependence on initial
conditions**. Combined with Picard-Lindelöf, it completes
the well-posedness picture.

---

## 2. Global existence and maximal solutions

The Picard-Lindelöf theorem is local — it gives a solution on
a possibly short interval $[t_0 - b, t_0 + b]$. The solution
may not extend to all of $I$.

**Maximal solution.** Among all solutions to the IVP, there
is a unique **maximal solution** defined on the largest
possible interval $(t_-, t_+)$ with $t_0 \in (t_-, t_+)$.
This follows by taking the union of all solution domains and
checking that the union is itself a valid interval on which
a solution is defined. Uniqueness (from Picard-Lindelöf)
guarantees the pieces patch together.

**When does the maximal solution exist globally?** The
solution blows up at $t_+$ iff $\|x(t)\| \to \infty$ as
$t \to t_+^-$ — it leaves every compact set. So global
existence (on all of $I$) is guaranteed whenever there is a
priori bound on $\|x(t)\|$. The standard sufficient condition:
if $\langle f(t, x), x \rangle \leq \alpha \|x\|^2 + \beta$
for constants $\alpha, \beta$, then Gronwall gives
$\|x(t)\|^2 \leq (\|x_0\|^2 + \beta/\alpha)e^{2\alpha(t-t_0)}$,
which is finite for all finite $t$.

**Economic relevance.** In continuous-time macro models
(Ramsey, AK, HANK), the ODE system for the state and costate
variables typically has global solutions on $[0, \infty)$
under appropriate boundary conditions (transversality at
infinity). Verifying this is part of the characterization of
the optimal path.

---

## 3. Linear ODE systems and the matrix exponential

### 3.1 The matrix exponential

A **linear ODE system** is $\dot x = A(t) x + b(t)$. When
$A$ is constant ($A(t) = A$ for all $t$), the solution has
an explicit form via the **matrix exponential**.

**Definition.** For $A \in \mathbb{R}^{n \times n}$:
$$e^{A} = \sum_{k=0}^\infty \frac{A^k}{k!}.$$
This series converges absolutely under the operator norm
(since $\|A^k\| \leq \|A\|^k$ and $\sum \|A\|^k/k! =
e^{\|A\|} < \infty$), making $e^A$ a well-defined bounded
operator.

**Theorem.** The unique solution to $\dot x = Ax$, $x(0) =
x_0$ is $x(t) = e^{At} x_0$.

**Proof.** Define $x(t) = e^{At}x_0$. Differentiating the
series term by term (justified by uniform convergence on
compact intervals):
$$\frac{d}{dt}e^{At} = \sum_{k=1}^\infty \frac{kA^k t^{k-1}}{k!}
= A\sum_{k=1}^\infty \frac{A^{k-1}t^{k-1}}{(k-1)!}
= A e^{At}.$$
So $\dot x(t) = A e^{At} x_0 = Ax(t)$, and $x(0) = e^0 x_0
= x_0$. Uniqueness is from Picard-Lindelöf. $\blacksquare$

### 3.2 Computing $e^{At}$ via diagonalization

When $A$ is diagonalizable over $\mathbb{R}$ (which holds for
symmetric $A$ by the spectral theorem, [[`linear-algebra.md`]]
§3.2, and for $A$ with distinct eigenvalues in general):
$A = P\Lambda P^{-1}$ with $\Lambda = \mathrm{diag}(\lambda_1,
\ldots, \lambda_n)$. Then
$$e^{At} = P e^{\Lambda t} P^{-1} = P\,\mathrm{diag}(e^{\lambda_1 t},
\ldots, e^{\lambda_n t})\,P^{-1}.$$
Each column of $e^{At}$ is a linear combination of $e^{\lambda_i t}
v_i$ where $v_i$ are the eigenvectors. The long-run behavior
of $x(t) = e^{At}x_0$ is dominated by the eigenvalue with
the largest real part.

For non-diagonalizable $A$ (repeated eigenvalues), the Jordan
normal form $A = PJP^{-1}$ gives $e^{At} = Pe^{Jt}P^{-1}$,
where $e^{Jt}$ for a Jordan block of size $k$ with eigenvalue
$\lambda$ is upper triangular with entries $e^{\lambda t}
t^j / j!$ on the $j$-th superdiagonal.

### 3.3 Variation of parameters

For the inhomogeneous system $\dot x = Ax + b(t)$, $x(0) =
x_0$, the **variation of parameters** (Duhamel's) formula is:
$$x(t) = e^{At}x_0 + \int_0^t e^{A(t-s)} b(s)\, ds.$$
**Proof.** Let $y(t) = e^{-At}x(t)$. Then $\dot y = -Ae^{-At}x
+ e^{-At}\dot x = e^{-At}(-Ax + \dot x) = e^{-At}b(t)$.
Integrating: $y(t) - y(0) = \int_0^t e^{-As}b(s)\, ds$.
Since $y(0) = x_0$, multiplying by $e^{At}$ gives the
formula. $\blacksquare$

The Duhamel formula is the continuous-time analogue of the
recursive formula for linear difference equations. In macro,
it appears whenever one solves for the path of the economy
after a shock: the integral $\int_0^t e^{A(t-s)}b(s)\,ds$
is the cumulated effect of all shocks $b(s)$ through time
$t$, each propagated forward by the system dynamics $e^{A(t-s)}$.

---

## 4. Phase portraits and stability

### 4.1 Equilibria and linearization

For an **autonomous system** $\dot x = f(x)$ (no explicit
time dependence), an **equilibrium** (or fixed point) $x^*$
satisfies $f(x^*) = 0$. The behavior of trajectories near
$x^*$ is determined by the **linearization**:
$$\dot z = Df(x^*) z, \qquad z = x - x^*,$$
where $Df(x^*)$ is the Jacobian of $f$ at $x^*$.

**Theorem (Hartman-Grobman).** If no eigenvalue of $Df(x^*)$
has zero real part (the equilibrium is **hyperbolic**), then
near $x^*$, the nonlinear flow of $\dot x = f(x)$ is
topologically equivalent to the linear flow of $\dot z =
Df(x^*)z$ — there is a homeomorphism from a neighborhood
of $x^*$ to a neighborhood of $0$ that maps orbits of the
nonlinear system to orbits of the linear system, preserving
the direction of time.

Hartman-Grobman says that for hyperbolic equilibria, linear
stability analysis is the whole story qualitatively. The
eigenvalues of $Df(x^*)$ determine the portrait.

### 4.2 Stability concepts

**Definition.** An equilibrium $x^*$ is:
- **Lyapunov stable** if for every $\varepsilon > 0$, there
  exists $\delta > 0$ such that $\|x(0) - x^*\| < \delta$
  implies $\|x(t) - x^*\| < \varepsilon$ for all $t \geq 0$.
- **Asymptotically stable** if it is Lyapunov stable and
  $\|x(t) - x^*\| \to 0$ as $t \to \infty$.
- **Unstable** otherwise.

For the linearization $\dot z = Az$ with $A = Df(x^*)$:
- All eigenvalues of $A$ have negative real part
  $\Rightarrow$ $x^*$ is asymptotically stable.
- Any eigenvalue of $A$ has positive real part
  $\Rightarrow$ $x^*$ is unstable.
- Eigenvalues on the imaginary axis $\Rightarrow$ higher-order
  analysis needed (Hartman-Grobman does not apply).

**Phase portrait classification in $\mathbb{R}^2$.** For
$\dot z = Az$ with $A \in \mathbb{R}^{2\times 2}$,
the eigenvalue pairs $(\lambda_1, \lambda_2)$ determine:

| Eigenvalues | Portrait | Stability |
|---|---|---|
| Both real, same sign ($\lambda_1 < \lambda_2 < 0$) | Stable node | Asymptotically stable |
| Both real, same sign ($0 < \lambda_1 < \lambda_2$) | Unstable node | Unstable |
| Both real, opposite signs ($\lambda_1 < 0 < \lambda_2$) | Saddle point | Unstable |
| Complex $\alpha \pm i\beta$, $\alpha < 0$ | Stable spiral | Asymptotically stable |
| Complex $\alpha \pm i\beta$, $\alpha > 0$ | Unstable spiral | Unstable |
| Purely imaginary $\pm i\beta$ | Center | Lyapunov stable, not asymptotically |

The saddle point is the economically important case:
the stable manifold (trajectories that converge to $x^*$)
is a lower-dimensional subspace, and the unstable manifold
(trajectories that diverge) has positive dimension. In
optimal control problems, the optimal path lies on the
stable manifold — this is the **saddle-path stability** that
Ramsey growth models and New Keynesian models rely on.
Uniqueness of the saddle path (local, near steady state)
follows from the dimension count: the stable manifold has
dimension equal to the number of eigenvalues with negative
real part, and the number of initial conditions to be
determined (free states) must match.

### 4.3 Lyapunov functions

**Theorem (Lyapunov's direct method).** Let $V: U \to [0,
\infty)$ be a $C^1$ function on a neighborhood $U$ of $x^*$
with $V(x^*) = 0$ and $V(x) > 0$ for $x \neq x^*$ (**positive
definite**). If $\dot V(x) = \nabla V(x)^\top f(x) \leq 0$
for all $x \in U$ (**negative semidefinite along orbits**),
then $x^*$ is Lyapunov stable. If moreover $\dot V(x) < 0$
for $x \neq x^*$ (**negative definite**), then $x^*$ is
asymptotically stable.

The time derivative $\dot V(x(t)) = \nabla V(x)^\top f(x)$
is computed without solving the ODE — that is the power of
the method. $V$ plays the role of an energy function; the
condition $\dot V \leq 0$ says energy is nonincreasing along
orbits, so the trajectory cannot escape to higher energy
levels.

**Connection to value functions.** In optimal control, the
value function $V(x)$ is often a Lyapunov function for the
optimally controlled system. When the HJB equation holds and
the optimal control is used, $\dot V(x^*(t)) = -L(t, x^*, u^*)$
along the optimal path — the value function decreases at
the rate of instantaneous loss. This is not coincidental:
Lyapunov's original insight was precisely this connection
between stability and the existence of decreasing "potential"
functions, which Bellman later rediscovered in the dynamic
programming setting.

---

## 5. First-order PDEs and the method of characteristics

### 5.1 First-order PDEs

A first-order PDE in $n+1$ variables $(t, x_1, \ldots, x_n)$
for an unknown function $u: \mathbb{R} \times \mathbb{R}^n
\to \mathbb{R}$ has the form:
$$F(t, x, u, \partial_t u, \nabla_x u) = 0.$$
The HJB equation is of this type:
$$-\partial_t V + H(t, x, \nabla_x V) = 0,$$
where $H$ is the Hamiltonian ([[`calculus.md`]] §6.4). First-order
PDEs differ from higher-order ones (the heat equation, wave
equation) in that their solutions can be constructed by
integrating along characteristic curves, reducing the PDE
to a system of ODEs.

### 5.2 The method of characteristics

The key idea: find curves $(t(s), x(s))$ in the $(t,x)$
space along which the PDE reduces to an ODE. These are the
**characteristic curves**.

For the linear first-order PDE $\partial_t u + a(t, x)
\cdot \nabla_x u = c(t, x) u + d(t, x)$, the characteristics
satisfy:
$$\frac{dt}{ds} = 1, \qquad \frac{dx}{ds} = a(t(s), x(s)).$$
Along a characteristic, $\frac{d}{ds}u(t(s), x(s)) =
\partial_t u + a \cdot \nabla_x u = cu + d$ — an ODE in
$u$ alone. The solution is found by: (1) solve the
characteristic ODE for $(t(s), x(s))$; (2) solve the ODE
for $u$ along the characteristic; (3) stitch together using
initial/boundary data.

**The Hamilton-Jacobi equation** $\partial_t V + H(x, \nabla_x V)
= 0$ has characteristics given by the Hamiltonian system:
$$\dot t = 1, \quad \dot x = \nabla_p H(x, p), \quad
\dot p = -\nabla_x H(x, p), \quad \dot V = p \cdot \nabla_p H - H,$$
where $p = \nabla_x V$ is the costate. These are exactly the
**canonical equations** of the Pontryagin maximum principle
([[`calculus.md`]] §6.4): the state equation $\dot x = \nabla_p H$
and the costate equation $\dot p = -\nabla_x H$. The method
of characteristics for the HJB equation *is* the maximum
principle — both approaches produce the same system of ODEs.
The characteristics are the optimal trajectories.

### 5.3 Shocks and the failure of classical solutions

Along different characteristics, initial data propagate
forward in time. If two characteristics cross, the function
$u$ must take two different values at the crossing point —
a contradiction if $u$ is to be a function. This crossing
produces a **shock**: a point at which the classical solution
breaks down and develops a discontinuity.

For the HJB equation, shocks in the gradient of $V$
correspond to points where two different candidate optimal
paths intersect — the value function has a kink.
This is not a pathology to be avoided but a feature to be
handled: in many optimal stopping and optimal control
problems, the value function is Lipschitz but not $C^1$
everywhere, and classical differentiation breaks down.

---

## 6. Viscosity solutions and the HJB

### 6.1 The viscosity solution concept

**Definition.** A continuous function $u: \Omega \to \mathbb{R}$
is a **viscosity subsolution** of $F(x, u, \nabla u) = 0$ if:
for every $C^1$ test function $\phi$ and every local maximum
point $x^*$ of $u - \phi$,
$$F(x^*, u(x^*), \nabla\phi(x^*)) \leq 0.$$
It is a **viscosity supersolution** if: for every $C^1$
test function $\phi$ and every local minimum point $x^*$ of
$u - \phi$,
$$F(x^*, u(x^*), \nabla\phi(x^*)) \geq 0.$$
It is a **viscosity solution** if it is both.

**Why this definition works.** At a local maximum of $u -
\phi$, the smooth function $\phi$ "touches $u$ from above"
at $x^*$. If $u$ were smooth, we would have $\nabla u(x^*)
= \nabla\phi(x^*)$ — the test function's gradient is a
legitimate proxy for $u$'s gradient at that point. The
definition uses this proxy in place of the (possibly
nonexistent) gradient of $u$. Since we test against all
smooth $\phi$, the definition is both consistent with the
classical notion (when $u$ is $C^1$, both notions agree)
and well-defined for Lipschitz or merely continuous $u$.

### 6.2 The verification theorem

The bridge between the HJB and the optimal control problem:

**Theorem (Verification).** Suppose $V \in C^1$ solves the
HJB equation
$$-\partial_t V(t, x) + H(t, x, \nabla_x V(t,x)) = 0$$
on $[t_0, t_1) \times \mathbb{R}^n$, with terminal condition
$V(t_1, x) = \Psi(x)$. Suppose also that for each $(t, x)$
there exists a measurable $u^*(t, x)$ achieving the minimum
in $H(t, x, p) = \min_u [L(t, x, u) + p^\top f(t, x, u)]$.
Then:
1. For any admissible control $u(\cdot)$ with corresponding
   trajectory $x(\cdot)$:
   $$V(t_0, x_0) \leq \int_{t_0}^{t_1} L(t, x(t), u(t))\,dt
   + \Psi(x(t_1)).$$
2. The control $u^*(t, x(t))$ (the minimizer in $H$)
   achieves equality; hence it is optimal.

**Proof sketch.** Let $x(\cdot)$ be any admissible trajectory.
Apply Itô's lemma (in the deterministic case: the standard
chain rule) to $V(t, x(t))$:
$$\frac{d}{dt}V(t, x(t)) = \partial_t V + \nabla_x V \cdot f(t,x,u).$$
The HJB gives $\partial_t V = H(t, x, \nabla_x V) = \min_{\tilde u}
[L + \nabla_x V \cdot f(t,x,\tilde u)] \leq L(t,x,u) +
\nabla_x V \cdot f(t,x,u)$. So $\frac{d}{dt}V \geq -L(t,x,u)$,
i.e., $L(t,x,u) + \frac{d}{dt}V \geq 0$. Integrating from
$t_0$ to $t_1$ and using the terminal condition
$V(t_1, x(t_1)) = \Psi(x(t_1))$:
$$\int_{t_0}^{t_1} L\,dt + \Psi(x(t_1)) \geq V(t_0, x_0).$$
Equality holds when the minimizer $u^*$ is used, since then
the HJB holds with equality along the path. $\blacksquare$

**Benveniste-Scheinkman.** When the value function is
differentiable at the optimal state $x^*$, the envelope
theorem ([[`optimization.md`]] §5) applied to the Bellman
equation gives $\nabla_x V(t, x^*) = \lambda(t)$ — the
gradient of the value function is the costate (the shadow
price). This connects the PDE approach (HJB) to the ODE
approach (Pontryagin): the costate equation $\dot\lambda =
-\nabla_x H$ and the costate boundary condition
$\lambda(t_1) = \nabla\Psi(x^*(t_1))$ are consequences of
the HJB combined with the verification theorem.

---

## 7. Kolmogorov equations

### 7.1 The setting

Consider a diffusion process $dX_t = \mu(X_t)\,dt +
\sigma(X_t)\,dW_t$ (a stochastic differential equation
as in [[`stochastic-processes.md`]] §7). Kolmogorov's two
equations describe how functions of $X_t$ and how
distributions of $X_t$ evolve over time.

The **infinitesimal generator** of the diffusion is the
operator
$$\mathcal{A}f(x) = \mu(x)^\top \nabla_x f(x) +
\frac{1}{2}\mathrm{tr}\!\bigl(\sigma(x)\sigma(x)^\top
\nabla^2_x f(x)\bigr),$$
which acts on $C^2$ functions $f$. By Itô's formula
([[`stochastic-processes.md`]] §7.2), $df(X_t) = \mathcal{A}f(X_t)\,dt
+ \nabla f(X_t)^\top \sigma(X_t)\,dW_t$, so $\mathcal{A}f$
is the drift of $f(X_t)$. Dynkin's formula states
$E[f(X_T)] = f(X_0) + E\!\left[\int_0^T \mathcal{A}f(X_t)\,dt\right]$
for integrable $f$ and stopping times $T$.

### 7.2 The backward Kolmogorov equation

Let $u(t, x) = E[g(X_T) | X_t = x]$ be the expected value
of a terminal payoff $g(X_T)$ given current state $X_t = x$.
Then $u$ satisfies the **backward Kolmogorov (Feynman-Kac)
equation**:
$$\partial_t u + \mathcal{A} u = 0 \quad \text{on } [0, T)
\times \mathbb{R}^n, \qquad u(T, x) = g(x).$$
This is a parabolic PDE: second-order in $x$ (through the
diffusion term $\frac{1}{2}\mathrm{tr}(\sigma\sigma^\top
\nabla^2 u)$), first-order in $t$. The solution propagates
backward from the terminal condition.

**Why it is called backward.** The equation runs backward
in time from the terminal date $T$ toward the initial date
$0$ — it computes what the expected payoff is, conditional
on the current state, as a function of time remaining. This
is the same direction as the Bellman equation in dynamic
programming ([[`dynamic-programming.md`]]): the value function
is computed from the terminal condition backward.

**The Feynman-Kac formula** is the converse: if $u$ satisfies
the backward equation, then $u(t, x) = E[g(X_T)|X_t = x]$.
This connects PDEs to expectations over stochastic processes
and is the foundation for Monte Carlo methods in option
pricing (compute $E[g(X_T)]$ by simulation rather than
solving the PDE).

### 7.3 The forward Kolmogorov equation (Fokker-Planck)

Let $p(t, x)$ be the density of $X_t$ (assuming it exists).
The **forward Kolmogorov (Fokker-Planck) equation** is:
$$\partial_t p = \mathcal{A}^* p := -\mathrm{div}(\mu p)
+ \frac{1}{2}\sum_{i,j}\partial_{x_i}\partial_{x_j}
[(\sigma\sigma^\top)_{ij} p].$$
Here $\mathcal{A}^*$ is the formal $L^2$ adjoint of
$\mathcal{A}$ (in the sense that $\int f \mathcal{A}^* p\,dx
= \int p \mathcal{A}f\,dx$ for appropriate $f, p$). The
equation propagates forward from an initial density $p(0, x)
= p_0(x)$.

The **stationary density** $p_\infty(x)$ (if it exists)
satisfies $\mathcal{A}^* p_\infty = 0$ — the time derivative
is zero. For the Ornstein-Uhlenbeck process ($\mu(x) =
-\kappa x$, $\sigma$ constant), the stationary density is
Gaussian with mean 0 and variance $\sigma^2/(2\kappa)$
([[`stochastic-processes.md`]] §7.2).

**Application to HANK.** In heterogeneous-agent models
(Bewley-Huggett-Aiyagari, HANK), the distribution of agents
over asset/income states is a density $\mu(t, a, y)$. The
Fokker-Planck equation describes how this distribution
evolves as agents optimize and income shocks hit. In
continuous time (as in the Achdou-Han-Lasry-Lions-Moll
framework), the equilibrium is a coupled system:
- **HJB equation** for the value function $V(t, a, y)$ (how
  much an agent with assets $a$ and income $y$ is worth).
- **Fokker-Planck equation** for the distribution $\mu(t, a, y)$.
- **Market clearing** condition linking the two.

This system is the continuous-time generalization of the
discrete-time Bewley model's value function iteration plus
distribution iteration ([[`stochastic-processes.md`]] §9.3 and
[[`dynamic-programming.md`]]). The mathematical structure —
two PDEs of opposite "time direction" coupled through a
fixed point — is what makes HANK models computationally
demanding.

---

## 8. What this file doesn't cover, and where to find it

**Second-order and higher-order PDEs.** The heat equation
$\partial_t u = \Delta u$ and wave equation $\partial_{tt} u
= c^2 \Delta u$ belong to the parabolic and hyperbolic
classes respectively. Their theory (energy methods, Sobolev
spaces, weak solutions) is not covered here; Evans's *Partial
Differential Equations* is the standard reference.

**Sobolev spaces and weak formulations.** Many PDEs arising
in macro (the HJB with non-smooth data, the Fokker-Planck
on non-compact domains) require solutions in Sobolev spaces
$W^{k,p}$ rather than classical $C^k$ spaces. The functional-
analytic framework is in [[`functional-analysis.md`]]; the PDE
applications are not developed here.

**Numerical methods for ODEs and PDEs.** Euler's method,
Runge-Kutta, finite differences, finite elements — not
covered here. For macro applications, Achdou et al. (2022)
in *Review of Economic Studies* gives the computational
approach to the coupled HJB-Fokker-Planck system; Maliar
and Maliar (2013) covers projection methods for Bellman
equations.

**Delay differential equations.** Solutions of $\dot x(t) =
f(t, x(t), x(t-\tau))$ — relevant for some trade and
information models. Not covered.

**Bifurcation theory.** What happens when the Hartman-Grobman
condition fails — when eigenvalues cross the imaginary axis
and equilibria collide or branch. The Hopf bifurcation (an
equilibrium destabilizes and a limit cycle appears) shows
up in heterogeneous-agent models with frictions. Not covered
here.

---

## Cross-references

- [[`20_Math/real-analysis.md`]] — the contraction mapping
  theorem (§6 there) is the engine of the Picard-Lindelöf
  proof (§1.2 here); completeness of $C([a,b])$ under
  the sup norm (§7 there) is the function space in which
  the Picard iteration lives.
- [[`20_Math/calculus.md`]] — the Euler-Lagrange equation
  (§6.2 there) is the ODE studied in §1 here; the HJB
  (§6.4 there) is the first-order PDE of §§5-6 here; the
  chain rule (§2 there) is used in the verification theorem
  proof; the Legendre transformation (§6.5 there) connects
  the Lagrangian and Hamiltonian formulations.
- [[`20_Math/optimization.md`]] — the envelope theorem (§5
  there) gives Benveniste-Scheinkman (§6.2 here); the
  Pontryagin necessary conditions (restated in [[`calculus.md`]]
  §6.4) and the verification theorem here are two directions
  of the same characterization.
- [[`20_Math/linear-algebra.md`]] — the matrix exponential
  (§3 here) is computed via diagonalization using the
  spectral theorem (§3.2 there); the stability classification
  (§4.2 here) uses eigenvalue real parts; Cholesky of
  $\sigma\sigma^\top$ (§7 there) appears in the Fokker-Planck
  operator $\mathcal{A}^*$.
- [[`20_Math/stochastic-processes.md`]] — Itô's formula (§7.2
  there) is what the backward Kolmogorov equation is a
  consequence of (via Dynkin's formula, §7.1 there); the
  Ornstein-Uhlenbeck process (§7.2 there) is an example
  whose stationary density is found via Fokker-Planck (§7.3
  here); the HJB for optimal stopping (§8 there) is the
  specific form of the general HJB (§6 here).
- [[`20_Math/functional-analysis.md`]] — the Lyapunov function
  method (§4.3 here) is related to the $L^2$ energy
  dissipation argument; viscosity solutions are weak
  solutions in a sense analogous to the weak formulations
  developed via the Riesz representation theorem (§3 there).
- [[`20_Math/dynamic-programming.md`]] — The
  Bellman operator is the discrete-time analogue of the HJB;
  its contraction property (Blackwell sufficient conditions)
  mirrors the Picard-Lindelöf contraction here; the saddle-
  path stability of §4.2 here is the continuous-time version
  of the stable manifold that selects the optimal policy.
- [[`10_Methods/Econometrics/local-projections.md`]] — the VAR
  companion matrix and impulse responses are the discrete-
  time analogue of the linear system $\dot x = Ax$ (§3 here);
  VAR stability (spectral radius $< 1$) corresponds to all
  eigenvalues of $A$ having negative real part (asymptotic
  stability).

---

*Last updated: April 2026.*
