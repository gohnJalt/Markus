---
tags: [math, calculus, foundations, multivariate]
related: [real-analysis, optimization, functional-analysis, probability, linear-algebra, stochastic-processes, dynamic-programming, modern-toolkit-references, Principles]
---

# Calculus

This file is the multivariable extension layer of the math
library. `real-analysis.md` introduced the total derivative
(Fréchet derivative) for maps $f: \mathbb{R}^n \to \mathbb{R}^m$,
proved the 1-dimensional mean value theorem and Taylor's
theorem, proved the inverse function theorem and the implicit
function theorem via the contraction mapping theorem. Those
results are prerequisites here and are not repeated.

What this file adds: the structured objects that live in
multivariable calculus — gradients, Jacobians, Hessians,
mixed partials — and the machinery that uses them. Section
by section: higher-order partial derivatives and Clairaut's
theorem (§1); the chain rule for compositions of maps
between Euclidean spaces (§2); Taylor's theorem in
$\mathbb{R}^k$ and its role in second-order optimality
conditions and asymptotic expansions (§3); integration in
$\mathbb{R}^n$ and the change of variables formula (§4);
matrix calculus — derivatives of functions of vectors and
matrices, the trace and determinant identities, the score
and information matrix (§5); and calculus of variations —
the Euler-Lagrange equation, the Legendre transformation,
and the bridge to dynamic optimization (§6).

A companion file `differential-equations.md` will cover
the ODE and PDE theory that calculus of variations connects
to — the Hamilton-Jacobi-Bellman equation, phase portraits,
and stability. This file ends at the variational level and
hands off to that file for the PDE treatment.

Prerequisites: `real-analysis.md` (all of §§1-6; especially
§4 for the total derivative, §5 for IFT, §6 for contraction
mapping). `linear-algebra.md` (§§1-3 for the Jacobian as a
matrix, PSD/PD Hessians, and the spectral theorem).
`optimization.md` (§§2-3 for first- and second-order
conditions — §3 here gives the Taylor foundation under those
conditions).

---

## 1. Higher-order partial derivatives and Clairaut's theorem

### 1.1 The Jacobian and Hessian as structured objects

Let $f: U \to \mathbb{R}$ with $U \subseteq \mathbb{R}^n$
open. The **gradient** $\nabla f(x) \in \mathbb{R}^n$ is the
column vector of first partials:
$$\nabla f(x) = \left(\frac{\partial f}{\partial x_1}, \ldots,
\frac{\partial f}{\partial x_n}\right)^\top.$$
When $f$ is $C^1$ (continuous partials), $\nabla f(x)$ is
the vector such that the total derivative satisfies $Df(x)[h]
= \nabla f(x)^\top h$ for all $h \in \mathbb{R}^n$. The
gradient points in the direction of steepest ascent; its
negative points in the direction of steepest descent.

For $f: U \to \mathbb{R}^m$ (a vector-valued map), the
**Jacobian** $Df(x) \in \mathbb{R}^{m \times n}$ has
$(i, j)$ entry $\partial f_i / \partial x_j$. The total
derivative $Df(x)$ is the Jacobian matrix (as established
in `real-analysis.md` §4); the gradient is the special case
$m = 1$, transposed to a column vector by convention.

The **Hessian** $\nabla^2 f(x) \in \mathbb{R}^{n \times n}$
is the matrix of second partial derivatives:
$$[\nabla^2 f(x)]_{ij} = \frac{\partial^2 f}{\partial x_i
\partial x_j}.$$
Clairaut's theorem (§1.2 below) guarantees that this matrix
is symmetric when $f$ is $C^2$. The Hessian is the derivative
of the gradient: $\nabla^2 f(x) = D(\nabla f)(x)$. It plays
the same role for second-order optimality that the gradient
plays for first-order: $\nabla^2 f(x^*) \succ 0$ at a
critical point is the sufficient condition for a strict local
minimum (`optimization.md` §2.3).

**The directional derivative.** For a unit vector $v \in
\mathbb{R}^n$, the directional derivative of $f$ at $x_0$ in
direction $v$ is
$$D_v f(x_0) = \lim_{t \to 0} \frac{f(x_0 + tv) - f(x_0)}{t}.$$
When $f$ is differentiable, $D_v f(x_0) = \nabla f(x_0)^\top v$.
The maximum directional derivative is $\|\nabla f(x_0)\|$,
achieved in the direction $v = \nabla f(x_0) / \|\nabla f(x_0)\|$.

### 1.2 Mixed partials: Clairaut's theorem

The existence of all partial derivatives does not guarantee
that mixed partials commute. Clairaut's theorem gives a
workable sufficient condition.

**Theorem (Clairaut-Schwarz).** Let $f: U \to \mathbb{R}$
with $U \subseteq \mathbb{R}^n$ open. Suppose the mixed
partial derivatives $\partial^2 f / \partial x_i \partial x_j$
and $\partial^2 f / \partial x_j \partial x_i$ both exist
and are continuous in a neighborhood of $x_0 \in U$. Then
$$\frac{\partial^2 f}{\partial x_i \partial x_j}(x_0) =
\frac{\partial^2 f}{\partial x_j \partial x_i}(x_0).$$

**Proof.** Without loss of generality take $i = 1$,
$j = 2$, $n = 2$. Fix $x_0 = (a, b)$ and consider the
difference quotient for small $h, k \neq 0$:
$$\Delta(h, k) = \frac{1}{hk}\bigl[f(a+h, b+k) - f(a+h, b)
- f(a, b+k) + f(a, b)\bigr].$$
Define $\phi(t) = f(t, b+k) - f(t, b)$. Then $\Delta(h,k)
= [\phi(a+h) - \phi(a)]/(hk)$. By the mean value theorem in
the $t$ variable, there exists $\xi$ between $a$ and $a+h$
such that $\phi(a+h) - \phi(a) = h\phi'(\xi) = h[f_1(\xi,
b+k) - f_1(\xi, b)]$. Applying the mean value theorem again
in the second variable, there exists $\eta$ between $b$ and
$b+k$ with $f_1(\xi, b+k) - f_1(\xi, b) = k f_{12}(\xi,
\eta)$. So $\Delta(h,k) = f_{12}(\xi, \eta)$.

As $h, k \to 0$, $(\xi, \eta) \to (a, b)$. By continuity
of $f_{12}$, $\Delta(h,k) \to f_{12}(a, b)$. By a symmetric
argument (defining $\psi(t) = f(a+h, t) - f(a, t)$), the
same $\Delta(h,k) \to f_{21}(a, b)$. So $f_{12}(a, b) =
f_{21}(a, b)$. $\blacksquare$

**Consequence.** If $f$ is $C^2$ (all second-order partials
exist and are continuous), the Hessian is symmetric:
$\nabla^2 f = (\nabla^2 f)^\top$. This is the standing
assumption in all optimization arguments that use the
Hessian. If $f$ is $C^k$, then partial differentiation up
to order $k$ commutes — the order of differentiation is
irrelevant.

### 1.3 Higher-order partial derivatives and $C^k$ functions

A function $f: U \to \mathbb{R}$ is **$C^k$** (k-times
continuously differentiable) if all partial derivatives of
order $\leq k$ exist and are continuous on $U$.
$C^\infty$ (smooth) functions have all partial derivatives
of all orders; $C^\omega$ (analytic) functions are smooth
and equal their Taylor series locally. The hierarchy is
$$C^\omega \subsetneq C^\infty \subsetneq \cdots \subsetneq C^2
\subsetneq C^1 \subsetneq C^0 \subsetneq \text{measurable}.$$

Most functions in economic models are $C^\infty$ on their
domains: polynomials, exponentials, logs, sines and cosines,
and compositions of these. The $C^2$ assumption is the
standard working assumption in optimization (it guarantees
the Hessian exists and is symmetric) and in the implicit
function theorem (the $C^1$ version is stated in
`real-analysis.md`; the $C^k$ version follows by induction
and gives $C^k$ implicitly-defined functions from $C^k$
constraint functions).

---

## 2. The chain rule

### 2.1 Statement and proof

The chain rule for compositions of differentiable maps
between Euclidean spaces is the result that makes "work with
the total derivative" manageable.

**Theorem (Chain rule).** Let $U \subseteq \mathbb{R}^n$
and $V \subseteq \mathbb{R}^m$ be open, let $g: U \to V$
and $f: V \to \mathbb{R}^p$ be differentiable. Then
$h = f \circ g: U \to \mathbb{R}^p$ is differentiable, and
$$Dh(x) = Df(g(x)) \circ Dg(x),$$
i.e., the Jacobian of $h$ at $x$ is the matrix product
$Df(g(x)) \cdot Dg(x)$.

**Proof.** Fix $x_0 \in U$, let $y_0 = g(x_0)$. Since $g$
is differentiable at $x_0$ and $f$ is differentiable at
$y_0$, write:
$$g(x_0 + s) = y_0 + Dg(x_0)s + \|s\|\,\alpha(s),$$
$$f(y_0 + t) = f(y_0) + Df(y_0)t + \|t\|\,\beta(t),$$
where $\alpha(s) \to 0$ as $s \to 0$ and $\beta(t) \to 0$
as $t \to 0$. Substitute the first into the second with
$t = Dg(x_0)s + \|s\|\,\alpha(s)$:
$$h(x_0 + s) = f(y_0) + Df(y_0)\bigl(Dg(x_0)s +
\|s\|\,\alpha(s)\bigr) + \|t\|\,\beta(t).$$
Rearranging:
$$h(x_0+s) - h(x_0) - Df(y_0)Dg(x_0)s =
Df(y_0)\|s\|\,\alpha(s) + \|t\|\,\beta(t).$$
Since $Df(y_0)$ is a bounded linear map and $\alpha(s) \to 0$,
the first term is $o(\|s\|)$. For the second: $\|t\| \leq
\|Dg(x_0)\|\|s\| + \|s\|\|\alpha(s)\| = O(\|s\|)$, and
$\beta(t) \to 0$ as $t = t(s) \to 0$, so $\|t\|\beta(t) =
o(\|s\|)$. Thus the total error is $o(\|s\|)$, confirming
$Dh(x_0) = Df(y_0)Dg(x_0)$. $\blacksquare$

### 2.2 Component form and special cases

In components, if $h = f \circ g$ with $g: \mathbb{R}^n
\to \mathbb{R}^m$ and $f: \mathbb{R}^m \to \mathbb{R}$,
the chain rule gives:
$$\frac{\partial h}{\partial x_j} = \sum_{i=1}^m
\frac{\partial f}{\partial y_i} \cdot
\frac{\partial g_i}{\partial x_j}.$$
This is the familiar multivariable chain rule from elementary
calculus. The matrix product form $Dh = Df \cdot Dg$ is the
coordinate-free statement that makes the pattern clear: each
row of $Df(y)$ (the gradient of a component of $f$) is dotted
with each column of $Dg(x)$ (the directional derivative of
a component of $g$ in a coordinate direction).

**Important special cases:**

- *Scalar $f$, vector $g$:* $\nabla h(x) = Dg(x)^\top
  \nabla f(g(x))$ — the gradient of a composition is the
  pullback of the gradient through the Jacobian transpose.
  This is the form that appears in backpropagation and in
  the score equation $\nabla_\theta \log p(x;\theta) =
  D_\theta g(\theta)^\top \nabla_g \log p(x; g)$ for
  reparameterized models.

- *Path derivative:* if $\gamma: \mathbb{R} \to \mathbb{R}^n$
  is a curve and $f: \mathbb{R}^n \to \mathbb{R}$, then
  $(f \circ \gamma)'(t) = \nabla f(\gamma(t))^\top
  \dot\gamma(t)$. The rate of change of $f$ along a path is
  the dot product of the gradient with the velocity. This is
  the starting point for gradient-flow arguments and for the
  time-derivative of value functions along optimal paths.

- *Leibniz rule:* differentiating $F(t) = \int_a^{b(t)}
  f(x, t)\, dx$ with respect to $t$ gives $F'(t) =
  f(b(t), t) b'(t) + \int_a^{b(t)} \partial f / \partial t\,
  dx$. The first term is the chain rule applied to the moving
  upper limit; the second is differentiation under the
  integral sign (justified by the dominated convergence
  theorem, `measure-theory.md` §5, when $|\partial f /
  \partial t|$ is dominated by an integrable function).

### 2.3 The mean value inequality for vector maps

The 1-dimensional mean value theorem says $f(b) - f(a) =
f'(\xi)(b - a)$ for some $\xi$. For vector-valued $f$, the
exact equality fails in general, but an inequality holds.

**Theorem (Mean value inequality).** Let $f: U \to
\mathbb{R}^m$ be differentiable on a convex open set $U
\subseteq \mathbb{R}^n$. Then for any $x, y \in U$:
$$\|f(y) - f(x)\| \leq \sup_{t \in [0,1]}
\|Df(x + t(y-x))\| \cdot \|y - x\|.$$

**Proof sketch.** Apply the 1-dimensional MVT to $\phi(t)
= f(x + t(y-x)) \cdot e_j$ for each unit vector $e_j$
and assemble. The inequality (rather than equality) arises
because the intermediate point $\xi$ depends on $j$. $\blacksquare$

The mean value inequality is the estimate that converts
"bounded derivative" into "Lipschitz continuity" — used in
the inverse function theorem proof in `real-analysis.md` §5
and in contraction mapping arguments.

---

## 3. Taylor's theorem in $\mathbb{R}^k$

### 3.1 First-order expansion

For $f: U \to \mathbb{R}$ with $U \subseteq \mathbb{R}^n$
open and $f$ differentiable at $x_0$, the **first-order
Taylor approximation** is
$$f(x_0 + h) = f(x_0) + \nabla f(x_0)^\top h + o(\|h\|).$$
This is simply the definition of differentiability written
out, with the remainder $o(\|h\|)$ reflecting that the
gradient gives the best linear approximation. The formula
is exact when $f$ is affine.

### 3.2 Second-order expansion

**Theorem (Second-order Taylor in $\mathbb{R}^n$).** Let
$f: U \to \mathbb{R}$ be $C^2$ in a neighborhood of
$x_0 \in U$. Then for $h$ sufficiently small:
$$f(x_0 + h) = f(x_0) + \nabla f(x_0)^\top h +
\frac{1}{2} h^\top \nabla^2 f(x_0) h + o(\|h\|^2).$$

**Proof.** Apply the 1-dimensional Taylor theorem to
$\phi(t) = f(x_0 + th)$ as a function of $t \in [0, 1]$:
$$\phi(1) = \phi(0) + \phi'(0) + \frac{1}{2}\phi''(0) +
\frac{1}{2}\int_0^1 (1-t)^2 \phi'''(t)\, dt.$$
By the chain rule: $\phi'(t) = \nabla f(x_0 + th)^\top h$
and $\phi'(0) = \nabla f(x_0)^\top h$. Differentiating again:
$\phi''(t) = h^\top \nabla^2 f(x_0 + th) h$, so
$\phi''(0) = h^\top \nabla^2 f(x_0) h$. The remainder
$\frac{1}{2}\int_0^1 (1-t)^2 \phi'''(t)\, dt$ is controlled by
the third derivatives of $f$; if $f$ is $C^3$ in a
neighborhood and $h \to 0$, the remainder is $O(\|h\|^3) =
o(\|h\|^2)$. $\blacksquare$

**The quadratic form $h^\top \nabla^2 f(x_0) h$.** This is
the key object: it measures how curved the surface $f$ is
in the direction $h$. Positive curvature in every direction
($\nabla^2 f(x_0) \succ 0$) means the surface is bowl-shaped
at $x_0$, confirming the second-order sufficient condition
for a local minimum. The spectral theorem for symmetric
matrices (`linear-algebra.md` §3.2) says the eigenvectors
of $\nabla^2 f(x_0)$ are the directions of principal
curvature, and the eigenvalues are the corresponding
curvatures. The condition number of the Hessian determines
how elongated the quadratic surface is — highly elongated
surfaces (large condition number) are the reason gradient
descent can be slow while Newton's method (which accounts
for the curvature shape) is fast.

### 3.3 Applications

**Optimality conditions.** At a critical point $x^*$
($\nabla f(x^*) = 0$), the Taylor expansion reduces to
$f(x^* + h) \approx f(x^*) + \frac{1}{2}h^\top \nabla^2
f(x^*)h$. The sign of this quadratic form determines
whether $x^*$ is a local minimum ($\succ 0$), maximum
($\prec 0$), or saddle (indefinite). This is exactly the
second-order condition in `optimization.md` §2.3; the Taylor
theorem here is the proof that the condition is both
necessary and sufficient at the stated level of generality.

**The delta method.** The vector delta method in `probability.md`
§4.2 uses the first-order Taylor expansion of a smooth
map $g: \mathbb{R}^k \to \mathbb{R}^m$ at the true
parameter $\theta_0$:
$$g(\hat\theta_n) - g(\theta_0) = G(\hat\theta_n - \theta_0)
+ o(\|\hat\theta_n - \theta_0\|),$$
where $G = Dg(\theta_0)$ is the $m \times k$ Jacobian. The
asymptotic variance $G\Sigma G^\top$ (the sandwich formula)
follows by applying the CLT to $\sqrt{n}(\hat\theta_n -
\theta_0)$ and using $o_P(n^{-1/2}) \cdot \sqrt{n} \to 0$.
The second-order delta method ($G = 0$ case in `probability.md`
§4.1) uses the second-order Taylor expansion to get the
chi-squared limit. Taylor's theorem is the mathematical
machine underneath asymptotic approximations throughout
econometrics.

**Comparative statics via Taylor.** In economic models,
an equilibrium condition $F(x^*, p) = 0$ implicitly defines
the equilibrium $x^*$ as a function of parameters $p$.
The implicit function theorem (`real-analysis.md` §5) gives
$dx^*/dp = -(D_x F)^{-1} D_p F$. The second-order expansion
of $F$ around $(x_0^*, p_0)$ gives the second-order
comparative statics — how curvature of the model translates
into second-order effects of parameter changes on
equilibria. This is the tool for welfare analysis in
structural models.

---

## 4. Integration in $\mathbb{R}^n$ and the change of variables formula

### 4.1 Multiple integration

For $f: R \to \mathbb{R}$ on a rectangle $R = [a_1, b_1]
\times \cdots \times [a_n, b_n]$, the Riemann integral over
$R$ is defined via partitions of the rectangle, as in
1D. For a measurable function on a measurable set, the
Lebesgue integral (`measure-theory.md` §4) is the more
general definition. The two agree when $f$ is Riemann-integrable.

The Fubini-Tonelli theorem (`measure-theory.md` §6) handles
the reduction of a multiple integral to iterated integrals:
$$\int_{R} f\, d\lambda^n = \int_{a_n}^{b_n}
\cdots \int_{a_1}^{b_1} f(x_1, \ldots, x_n)\, dx_1 \cdots dx_n,$$
with the order of integration freely interchangeable when
$f \geq 0$ or $\int |f| < \infty$. Fubini is the theorem
that makes computing multivariable integrals by iterating
1D integrals legitimate.

### 4.2 The change of variables formula

The most important theorem in multivariable integration for
applications:

**Theorem (Change of variables / substitution).** Let
$U, V \subseteq \mathbb{R}^n$ be open, $\Phi: U \to V$ a
$C^1$ diffeomorphism (bijection with $C^1$ inverse), and
$f: V \to \mathbb{R}$ integrable. Then:
$$\int_V f(y)\, dy = \int_U f(\Phi(x))
|\det D\Phi(x)|\, dx.$$
The factor $|\det D\Phi(x)|$ is the **Jacobian determinant**
of $\Phi$ at $x$.

**Proof sketch.** The argument proceeds in steps. (i) For
linear $\Phi = A$ (a matrix), the formula becomes
$\int f(Ax)|\det A|\, dx = \int f(y)\, dy$ — this is the
change-of-basis formula for Lebesgue measure, following from
the fact that $|\det A|$ is the volume scaling factor of $A$
(`linear-algebra.md` §5.2 on SVD: the singular values are
the stretch factors in each direction and $|\det A| =
\prod \sigma_i$). (ii) For general $\Phi$, the key
approximation is that $\Phi$ looks like its derivative
$D\Phi(x)$ (a linear map) on small neighborhoods of $x$.
Partition $U$ into small rectangles, approximate $\Phi$ by
its first-order Taylor expansion on each piece, and use
step (i) on each piece; take the mesh to zero. The Jacobian
determinant appears because it is the local volume scaling
factor of $\Phi$. Making this rigorous uses the inverse
function theorem and a covering argument. $\blacksquare$

**Standard applications.**

- *Polar coordinates:* $\Phi(r, \theta) = (r\cos\theta,
  r\sin\theta)$. $D\Phi$ has columns $(\cos\theta,
  \sin\theta)^\top$ and $(-r\sin\theta, r\cos\theta)^\top$,
  so $|\det D\Phi| = r$. Thus $\int_V f(x,y)\, dx\, dy
  = \int_0^\infty \int_0^{2\pi} f(r\cos\theta, r\sin\theta)
  \cdot r\, d\theta\, dr$. The Gaussian integral
  $\int_{-\infty}^\infty e^{-x^2}\, dx = \sqrt{\pi}$ follows
  by squaring it, switching to polar, and integrating.

- *Density of a transformed random variable.* If $X$ has
  density $p_X(x)$ and $Y = g(X)$ with $g$ a
  $C^1$ bijection on the support, then $Y$ has density
  $p_Y(y) = p_X(g^{-1}(y)) \cdot |Dg^{-1}(y)|$. The
  Jacobian determinant is the correction that ensures
  $\int p_Y\, dy = 1$. This formula is the foundation for
  the probability integral transform, normalizing flows in
  machine learning, and distributional analysis of
  transformed estimators.

- *The Gaussian normalizing constant.* Let $X \sim \mathcal{N}(\mu,
  \Sigma)$ with $\Sigma \succ 0$ in $\mathbb{R}^n$. The
  normalizing constant $(2\pi)^{n/2} |\det\Sigma|^{1/2}$
  comes from applying the change of variables with
  $\Phi(z) = \Sigma^{1/2}z + \mu$ (Cholesky factorization,
  `linear-algebra.md` §7) to reduce to the standard
  $\mathcal{N}(0, I)$ integral.

### 4.3 Integration by parts in multiple dimensions

The multivariable integration by parts is **Green's first
identity**: for $C^1$ functions $u, v$ on a bounded open
$\Omega \subseteq \mathbb{R}^n$ with smooth boundary:
$$\int_\Omega u \,\Delta v\, dx = -\int_\Omega
\nabla u \cdot \nabla v\, dx + \int_{\partial\Omega} u
\frac{\partial v}{\partial \nu}\, dS,$$
where $\Delta v = \sum_i \partial^2 v/\partial x_i^2$ is the
Laplacian, $\partial v/\partial\nu$ is the outward normal
derivative, and $dS$ is the surface measure on $\partial\Omega$.
This follows from the divergence theorem: $\int_\Omega
\mathrm{div}(F)\, dx = \int_{\partial\Omega} F \cdot \nu\, dS$,
applied to $F = u\nabla v$.

In calculus of variations (§6), integration by parts of this
form is what converts the variational condition
$\delta \int L\, dt = 0$ into the Euler-Lagrange ODE.

---

## 5. Matrix calculus

Derivatives of functions of matrices and vectors appear
throughout econometrics: the score is the gradient of the
log-likelihood with respect to parameters; the information
matrix is the Hessian; structural estimation requires
gradients of moment conditions with respect to parameters.
This section collects the identities.

### 5.1 Derivatives of scalar functions of vectors

Let $x \in \mathbb{R}^n$, $A \in \mathbb{R}^{m \times n}$
fixed, $b \in \mathbb{R}^n$ fixed. The following gradients
are fundamental:

| Function $f(x)$ | Gradient $\nabla_x f$ |
|---|---|
| $a^\top x$ (linear) | $a$ |
| $x^\top A x$ (quadratic form, $A$ symmetric) | $2Ax$ |
| $x^\top A x$ (quadratic form, $A$ not necessarily symmetric) | $(A + A^\top)x$ |
| $\|Ax - b\|^2$ (OLS objective) | $2A^\top(Ax - b)$ |
| $\log p(x; \theta)$ evaluated at data $x$ | the **score** $s(\theta)$ |

For the OLS objective: $f(\beta) = \|y - X\beta\|^2 =
y^\top y - 2y^\top X\beta + \beta^\top X^\top X\beta$.
Then $\nabla_\beta f = -2X^\top y + 2X^\top X\beta$.
Setting to zero gives the normal equations $X^\top X\beta =
X^\top y$, confirming the OLS formula. The Hessian is
$\nabla^2_\beta f = 2X^\top X$, which is PD when $X$ has
full column rank — confirming global convexity.

**The chain rule for gradients.** If $f(x) = g(Ax + b)$
with $g: \mathbb{R}^m \to \mathbb{R}$ and $A: \mathbb{R}^n
\to \mathbb{R}^m$, then $\nabla_x f = A^\top \nabla_y g(y)
\big|_{y = Ax+b}$. The transpose appears because the
Jacobian of $y \mapsto Ax+b$ is $A$, and the gradient of a
scalar function pulls back through the Jacobian transpose
(§2.2 above).

### 5.2 Derivatives of functions of matrices

Let $A \in \mathbb{R}^{n \times n}$ (varying) and define
scalar-valued functions of $A$. The derivative is taken
with respect to $A$ treated as an $n^2$-dimensional vector,
with the result reshaped into an $n \times n$ matrix.

**Key identities:**

| Function $f(A)$ | $\partial f / \partial A$ |
|---|---|
| $\mathrm{tr}(BA)$ | $B^\top$ |
| $\mathrm{tr}(ABA^\top)$ | $A(B + B^\top)$ |
| $\log \det A$ (for $A \succ 0$) | $A^{-\top} = (A^{-1})^\top$ |
| $\det A$ | $(\mathrm{adj}\, A)^\top = \det(A)\cdot A^{-\top}$ |
| $\mathrm{tr}(A^{-1}B)$ | $-A^{-\top}BA^{-\top}$ |

The most important for econometrics: $\partial \log\det
\Sigma / \partial \Sigma = \Sigma^{-1}$ (for $\Sigma \succ 0$).
This is the derivative that appears in the likelihood of the
multivariate Gaussian: the score with respect to the
covariance matrix involves $\Sigma^{-1}$.

**Proof of $\partial \log\det A / \partial A = A^{-\top}$.**
The derivative of $\det A$ with respect to $A_{ij}$ is the
$(j,i)$ cofactor (expansion along row $i$, column $j$), which
equals $\det(A) \cdot (A^{-1})_{ji}$ by Cramer's rule. So
$\partial\det A/\partial A = \det(A) \cdot A^{-\top}$.
By the chain rule: $\partial \log\det A/\partial A =
(1/\det A) \cdot \det(A) \cdot A^{-\top} = A^{-\top}$.
For symmetric $A = \Sigma$, $\Sigma^{-\top} = \Sigma^{-1}$
since $(\Sigma^{-1})^\top = (\Sigma^\top)^{-1} = \Sigma^{-1}$.
$\blacksquare$

### 5.3 The score and information matrix

For a statistical model with density $p(x; \theta)$ and
log-likelihood $\ell(\theta) = \log p(x; \theta)$:

- The **score** is $s(\theta) = \nabla_\theta \ell(\theta) =
  \nabla_\theta \log p(x; \theta)$.
- The **observed information matrix** is $\hat{\mathcal{I}}(\theta)
  = -\nabla^2_\theta \ell(\theta) = -$ Hessian of the log-likelihood.
- The **Fisher information matrix** is
  $\mathcal{I}(\theta) = E_\theta[-\nabla^2_\theta \log
  p(x;\theta)] = E_\theta[s(\theta)s(\theta)^\top]$,
  where the second equality is the **information matrix
  equality** (the score has zero mean and its outer product
  equals the negative Hessian in expectation).

**Proof of the information matrix equality.** Differentiate
the identity $\int p(x;\theta)\, dx = 1$ with respect to
$\theta$ once to get $\int \nabla_\theta p\, dx = 0$, i.e.,
$E_\theta[s(\theta)] = 0$. Differentiate again: the
dominated convergence theorem (`measure-theory.md` §5)
licenses passing the derivative under the integral when the
score is square-integrable. The derivative of $s(\theta)^\top$
with respect to $\theta$ is the Hessian of $\log p$ plus
the outer product of the score:
$$0 = \int [\nabla^2_\theta \log p + (\nabla_\theta \log p)
(\nabla_\theta \log p)^\top] p\, dx =
E[\nabla^2_\theta \log p] + E[ss^\top],$$
which gives $\mathcal{I}(\theta) = -E[\nabla^2_\theta \log p]
= E[ss^\top]$. $\blacksquare$

The Cramér-Rao bound states that for any unbiased estimator
$\hat\theta$, $\mathrm{Var}(\hat\theta) \succeq
\mathcal{I}(\theta)^{-1}$ (PSD ordering). The bound follows
from the Cauchy-Schwarz inequality in $L^2$ space
(`functional-analysis.md` §2). The MLE achieves this bound
asymptotically — the asymptotic variance of $\sqrt{n}(\hat\theta_{\mathrm{MLE}}
- \theta_0)$ is $\mathcal{I}(\theta_0)^{-1}$.

### 5.4 The vec operator and Kronecker products

For the derivative of a matrix-to-matrix map, the notation
becomes cumbersome without additional structure. The **vec
operator** stacks columns of an $m \times n$ matrix into an
$mn$-vector: $\text{vec}(A) \in \mathbb{R}^{mn}$ with
$[\text{vec}(A)]_{(j-1)m + i} = A_{ij}$.

Key identity: $\text{vec}(AXB) = (B^\top \otimes A)
\text{vec}(X)$, where $\otimes$ denotes the Kronecker product
($A \otimes B$ is the block matrix with $(i,j)$ block
$A_{ij}B$). This identity reduces matrix calculus problems
to standard vector calculus on $\text{vec}(X)$.

**Application: GLS and WLS.** The GLS estimator minimizes
$(y - X\beta)^\top \Omega^{-1}(y - X\beta)$ with respect to
$\beta$. The gradient with respect to $\beta$ is
$-2X^\top \Omega^{-1}(y - X\beta)$. Setting to zero:
$\hat\beta_{\mathrm{GLS}} = (X^\top \Omega^{-1}X)^{-1}
X^\top\Omega^{-1}y$. The Hessian is $2X^\top\Omega^{-1}X$,
PD when $\Omega \succ 0$ and $X$ has full column rank.
Computing this efficiently requires Cholesky of $\Omega$
(`linear-algebra.md` §7), not explicit inversion of $\Omega$.

---

## 6. Calculus of variations

### 6.1 The variational problem

The classical problem of the calculus of variations: among
all $C^2$ curves $x: [t_0, t_1] \to \mathbb{R}^n$ satisfying
fixed boundary conditions $x(t_0) = x_0$ and $x(t_1) = x_1$,
find the one that minimizes (or makes stationary) the
**functional**
$$J[x] = \int_{t_0}^{t_1} L(t, x(t), \dot x(t))\, dt,$$
where $L: \mathbb{R} \times \mathbb{R}^n \times \mathbb{R}^n
\to \mathbb{R}$ is the **Lagrangian**, assumed $C^2$.

The functional $J$ maps curves to real numbers — it is a
function on a function space. The derivative of $J$ with
respect to the curve is what the Euler-Lagrange equation
computes.

**Economic examples.** (i) Optimal consumption growth: the
representative agent maximizes $\int_0^\infty e^{-\rho t}
u(c(t))\, dt$ subject to $\dot k = f(k) - c$; this is a
calculus of variations problem with a state equation
constraint (handled via the Pontryagin approach, §6.4).
(ii) Optimal path length: the curve connecting two points
that minimizes $\int_0^1 \|\dot x(t)\|\, dt$ (the geodesic
problem). (iii) Minimum-variance prediction: among all
linear predictors of a stationary process with given
covariance structure, the one minimizing the integrated
mean-squared prediction error is found by a variational
argument.

### 6.2 The Euler-Lagrange equation

**Theorem (Euler-Lagrange).** Let $x^*$ minimize $J[x] =
\int_{t_0}^{t_1} L(t, x, \dot x)\, dt$ over $C^2$ curves
with fixed endpoints. Then $x^*$ satisfies:
$$\frac{\partial L}{\partial x}(t, x^*, \dot x^*)
- \frac{d}{dt}\frac{\partial L}{\partial \dot x}
(t, x^*, \dot x^*) = 0 \quad \text{for all } t \in (t_0, t_1).$$

**Proof.** Let $x^*$ be a minimizer and $\eta: [t_0, t_1]
\to \mathbb{R}^n$ any $C^2$ variation with $\eta(t_0) =
\eta(t_1) = 0$ (so that $x^* + \varepsilon\eta$ also
satisfies the boundary conditions). Define
$\phi(\varepsilon) = J[x^* + \varepsilon\eta]$. Since $x^*$
is a minimizer, $\phi$ has a minimum at $\varepsilon = 0$,
so $\phi'(0) = 0$.

Differentiate under the integral (justified by smoothness
of $L$):
$$\phi'(\varepsilon) = \int_{t_0}^{t_1} \left[
\frac{\partial L}{\partial x}^\top \eta +
\frac{\partial L}{\partial \dot x}^\top \dot\eta
\right] dt.$$
Set $\varepsilon = 0$ and integrate the second term by parts,
using $\eta(t_0) = \eta(t_1) = 0$ to kill the boundary term:
$$0 = \phi'(0) = \int_{t_0}^{t_1} \left[
\frac{\partial L}{\partial x} - \frac{d}{dt}
\frac{\partial L}{\partial \dot x} \right]^\top \eta\, dt.$$
Since this holds for all admissible variations $\eta$, the
**fundamental lemma of the calculus of variations** (if
$\int f(t)^\top \eta(t)\, dt = 0$ for all $C^\infty$ $\eta$
vanishing at the endpoints, then $f \equiv 0$) implies the
integrand is zero, giving the Euler-Lagrange equation. $\blacksquare$

The Euler-Lagrange equation is an ODE in $x^*(t)$ — a
second-order ODE in general (since it involves $\ddot x^*$
through $\frac{d}{dt}\frac{\partial L}{\partial \dot x}$).
The ODE theory for this equation — existence, uniqueness,
boundary value problems — belongs in `differential-equations.md`.

### 6.3 Transversality conditions

When one or both endpoints are free (not fixed), the
stationarity condition acquires **transversality conditions**
at the free endpoint.

**Free right endpoint ($x(t_1)$ free).** The boundary term
from the integration by parts is
$\left[\frac{\partial L}{\partial \dot x}^\top \eta\right]_{t_0}^{t_1}
= \frac{\partial L}{\partial \dot x}(t_1, x^*(t_1),
\dot x^*(t_1))^\top \eta(t_1)$.
Since $\eta(t_1)$ is now unrestricted, the transversality
condition is
$$\frac{\partial L}{\partial \dot x}(t_1, x^*(t_1),
\dot x^*(t_1)) = 0.$$
In economic terms: at the terminal time, the marginal value
of the state variable is zero (the endpoint carries no
continuation value). This is the **no-Ponzi condition** in
continuous-time growth models — the terminal capital stock
times its shadow price goes to zero.

**Terminal value function.** If the right endpoint is free
but the terminal payoff is $\Psi(x(t_1))$ (a bequest or
scrap value), the problem becomes $\min J[x] - \Psi(x(t_1))$.
The transversality condition becomes $\partial L/\partial
\dot x = \nabla\Psi(x^*(t_1))$ — the costate equals the
marginal bequest value at the terminal time.

### 6.4 The Hamiltonian and Pontryagin's maximum principle

The calculus of variations framework becomes the **optimal
control** framework when there is a state equation
$\dot x = f(t, x, u)$ where $u$ is a control variable (not
necessarily the velocity $\dot x$). The problem is:
$$\min_{u(\cdot)} \int_{t_0}^{t_1} L(t, x(t), u(t))\, dt,
\quad \text{s.t.}\quad \dot x = f(t, x, u),
\quad x(t_0) = x_0.$$
The **Hamiltonian** is
$$H(t, x, u, \lambda) = L(t, x, u) + \lambda^\top f(t, x, u),$$
where $\lambda(t) \in \mathbb{R}^n$ is the **costate variable**
(the continuous-time analogue of the Lagrange multiplier).

**Theorem (Pontryagin's maximum principle, necessary
conditions).** If $(x^*, u^*)$ is optimal, there exists an
absolutely continuous costate $\lambda(t)$ such that:
1. **State equation:** $\dot x^* = \partial H/\partial\lambda = f(t, x^*, u^*)$.
2. **Costate equation:** $\dot\lambda = -\partial H/\partial x =
   -\nabla_x L - (\nabla_x f)^\top \lambda$.
3. **Optimality condition:** $u^*$ minimizes $H(t, x^*, u,
   \lambda)$ over all admissible $u$ at each $t$.
4. **Transversality:** $\lambda(t_1) = 0$ if the terminal
   state is free (or $\lambda(t_1) = \nabla\Psi(x^*(t_1))$
   if there is a terminal payoff).

When $L$ is $C^2$ and $f$ is affine in $u$, condition 3
reduces to $\partial H/\partial u = 0$, and the maximum
principle reduces to the Euler-Lagrange equation. The power
of the maximum principle over the Euler-Lagrange equation
is that it handles non-smooth or constrained controls
(bang-bang control, inequality constraints on $u$) where
the variational argument breaks down.

**Connection to the Bellman equation.** Define the value
function $V(t, x) = \min_{u(\cdot)} \int_t^{t_1}
L(s, x(s), u(s))\, ds$ over paths starting at $(t, x)$.
The **Hamilton-Jacobi-Bellman (HJB) equation** is:
$$-\frac{\partial V}{\partial t} = \min_u\, H\!\left(t, x, u,
\nabla_x V\right) = \min_u\bigl[L(t, x, u) +
(\nabla_x V)^\top f(t, x, u)\bigr].$$
The costate $\lambda = \nabla_x V$ — the shadow price of the
state is the gradient of the value function. The HJB is a
first-order PDE in $V$; existence and uniqueness of its
viscosity solutions, and its connection to the optimal
control problem via the verification theorem, belong in
`differential-equations.md`. The discrete-time analogue —
the Bellman equation $V(x) = \min_u [r(x, u) + \beta
V(f(x,u))]$ — is the centerpiece of `dynamic-programming.md`.

### 6.5 The Legendre transformation

The **Legendre transformation** converts between a function
and its conjugate, and appears in both duality theory
(`optimization.md` §4) and the passage from Lagrangian to
Hamiltonian mechanics.

For a convex $C^1$ function $f: \mathbb{R}^n \to \mathbb{R}$,
the Legendre-Fenchel conjugate (or convex conjugate) is:
$$f^*(p) = \sup_{x \in \mathbb{R}^n} \{p^\top x - f(x)\}.$$
The supremum is attained where $p = \nabla f(x)$ (the
gradient equals the dual variable $p$). If $f$ is strictly
convex and $C^2$, the map $x \mapsto \nabla f(x)$ is
invertible (by the inverse function theorem, since its
Jacobian $\nabla^2 f(x)$ is PD), and
$f^*(p) = p^\top x(p) - f(x(p))$
where $x(p) = (\nabla f)^{-1}(p)$.

**Key properties:**
- $(f^*)^* = f$ when $f$ is closed convex (Fenchel duality).
- Young's inequality: $p^\top x \leq f(x) + f^*(p)$, with
  equality iff $p = \nabla f(x)$ (or $x \in \partial f^*(p)$
  in the subdifferential sense).
- The Lagrangian and Hamiltonian are related by a Legendre
  transform: $H(t, x, p) = L^*_{\dot x}(t, x, p) =
  \sup_v [p^\top v - L(t, x, v)]$, where the sup is over
  velocities $v = \dot x$ and the dual variable is the
  momentum $p = \partial L/\partial \dot x$.

In economics: the expenditure function $e(p, u) =
\min_x \{p^\top x : u(x) \geq u\}$ is the Legendre
transform of the indirect utility function in the prices.
Shephard's lemma ($\nabla_p e = x_h$, the Hicksian demand)
and Roy's identity ($-\nabla_p v / \partial v/\partial m = x_M$,
the Marshallian demand) are both envelope theorem results
(proved in `optimization.md` §5) that follow from the
duality structure.

---

## 7. What this file doesn't cover, and where to find it

**Ordinary and partial differential equations.** The
Euler-Lagrange equation is an ODE; the HJB equation is a
first-order PDE. Existence and uniqueness for ODEs (the
Picard-Lindelöf theorem), phase portraits, stability of
equilibria, linear ODE systems, and the theory of PDEs
(characteristics, boundary value problems, the heat and
wave equations as structural macro tools) all belong in
`differential-equations.md`.

**Differential geometry.** Manifolds, tangent bundles,
exterior calculus (differential forms, Stokes' theorem in
full generality), Lie groups. These appear in structural
macro models that have state spaces with curvature and in
HANK models where the distribution of agents lives on an
infinite-dimensional manifold. Not covered here.

**Stochastic calculus.** The Itô formula is the stochastic
analogue of the chain rule for Brownian motion; the HJB
equation in continuous time is derived using Itô's lemma
rather than the classical chain rule. Both are in
`stochastic-processes.md` §§7-8.

**Subdifferential calculus and non-smooth analysis.** The
calculus of convex functions that are not $C^1$ (kinks,
indicator functions, the $\ell^1$ penalty in LASSO).
Subgradients, the subdifferential, proximal operators.
These arise in modern high-dimensional estimation; the
treatment belongs in `optimization.md` if added, or in a
future `convex-analysis.md`.

---

## Cross-references

- `20_Math/real-analysis.md` — the direct prerequisite.
  The total derivative / Fréchet derivative (§4 there),
  1D Taylor with Lagrange remainder (§4 there), the IFT and
  implicit function theorem (§5 there) — all assumed here.
  Clairaut's theorem (§1.2 here) and the multivariate Taylor
  (§3 here) are the extensions. The contraction mapping
  theorem (§6 there) underlies both the IFT used in §§2-3
  here and the Picard-Lindelöf theorem in `differential-equations.md`.
- `20_Math/optimization.md` — the Taylor expansion (§3 here)
  is the proof behind the second-order conditions (§2.3 there);
  the Legendre transformation (§6.5 here) extends the duality
  section (§4 there); matrix calculus (§5 here) makes the
  score and information matrix (§6.2 there) computable.
- `20_Math/linear-algebra.md` — the Hessian is a symmetric
  matrix; its spectral decomposition (§3.2 there) characterizes
  curvature directions; Cholesky (§7 there) is how GLS
  and Gaussian integrals are computed in practice; the
  Jacobian determinant formula (§4.2 here) involves the
  SVD singular values as volume-scaling factors (§5 there).
- `20_Math/functional-analysis.md` — the Euler-Lagrange
  equation is derived via the projection theorem in function
  space (the fundamental lemma of calculus of variations
  is a density argument); the calculus of variations problem
  is formally an optimization on an infinite-dimensional
  Hilbert space, and the projection machinery of §3 there
  is the abstract setting.
- `20_Math/measure-theory.md` — Fubini-Tonelli (§6 there)
  reduces multiple integration to iterated integrals (§4.1
  here); the dominated convergence theorem (§5 there) justifies
  differentiation under the integral in the Leibniz rule
  (§2.2 here) and in the information matrix equality proof
  (§5.3 here).
- `20_Math/probability.md` — the delta method (§4.2 there)
  is the first-order Taylor expansion applied to estimators
  (§3.3 here); the score and information matrix (§5.3 here)
  are the quantities whose asymptotic behavior the CLT and
  LLN determine (§7 there).
- `20_Math/stochastic-processes.md` — Itô's lemma
  (§7.2 there) is the stochastic chain rule, corresponding
  to the deterministic chain rule (§2 here) but with a
  second-derivative correction from quadratic variation; the
  HJB equation in continuous time (§6.4 here) is derived
  using Itô's lemma.
- `20_Math/dynamic-programming.md` — The HJB equation
  (§6.4 here) is the continuous-time Bellman equation;
  the discrete-time Bellman equation there is the exact
  analogue of the variational problem here. The envelope
  theorem (`optimization.md` §5) applied to the Bellman
  equation gives Benveniste-Scheinkman — the derivative of
  the value function at the optimum.
- `differential-equations.md` — The Euler-Lagrange equation
  and HJB are ODEs and a first-order PDE respectively;
  their rigorous theory (Picard-Lindelöf, phase portraits,
  viscosity solutions) belongs there.
- `10_Methods/Econometrics/regression-discontinuity.md` —
  the Runge's phenomenon critique of global polynomial
  approximation (§4.1 there) is a warning about the behavior
  of the $n$-th order Taylor approximation far from the
  expansion point (§3.3 here).

---

*Last updated: April 2026.*
