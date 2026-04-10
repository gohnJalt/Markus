---
tags: [math, functional-analysis, foundations]
related: [real-analysis, measure-theory, optimization, linear-algebra, probability, stochastic-processes, modern-toolkit-references, Principles]
---

# Functional Analysis

This file builds the functional-analytic infrastructure that
the rest of the math layer needs and that shows up throughout
econometrics theory and structural macro. The central objects
are infinite-dimensional vector spaces — function spaces —
equipped with norms and inner products, and the bounded linear
maps between them.

The results that matter most for econ work are: the $L^p$
spaces (which formalize the moment conditions that appear
throughout econometrics and the spaces that convergence
theorems live in), the projection theorem in Hilbert space
(which is the correct formulation of what OLS and conditional
expectation are doing), the Riesz representation theorem (the
dual of a Hilbert space is itself — the engine behind the
Radon-Nikodym proof), the big four theorems (particularly the
uniform boundedness principle and the Hahn-Banach theorem,
which underpin asymptotic theory and infinite-dimensional
duality), the spectral theorem for compact self-adjoint
operators (PCA in function spaces, covariance operators, HAC
theory), and the formal definition of conditional expectation
as $L^2$ projection.

The file covers: normed spaces and Banach spaces, $L^p$ spaces
and their completeness (Riesz-Fischer), Hilbert spaces and the
projection theorem, the Riesz representation theorem, bounded
linear operators, the big four theorems (Baire category,
uniform boundedness, open mapping, closed graph, Hahn-Banach),
dual spaces and weak convergence, the Radon-Nikodym theorem
proved via Riesz (deferred from `measure-theory.md`), compact
operators and the spectral theorem, and conditional expectation
as $L^2$ projection.

Prerequisites: `real-analysis.md` (metric spaces, compactness,
completeness) and `measure-theory.md` ($L^p$ functions, modes
of convergence, measure-theoretic probability foundations).
`optimization.md` is useful background for the duality
sections but not formally required.

This file is long enough that it may eventually split into
`banach-spaces.md` and `hilbert-spaces-and-operators.md`.
The natural split point is after §5 (the big four theorems) —
§§1-5 are Banach-space material, §§6-9 are Hilbert-space and
operator material. For now they stay in one file. If and when
the split happens, the cross-references in sibling files point
to whichever file each result ends up in.

---

## 1. Normed spaces and Banach spaces

### 1.1 Normed spaces

A *normed vector space* $(X, \|\cdot\|)$ is a vector space $X$
over $\mathbb{R}$ equipped with a norm $\|\cdot\|: X \to [0, \infty)$
satisfying:
1. $\|x\| = 0$ if and only if $x = 0$,
2. $\|\alpha x\| = |\alpha| \|x\|$ for all scalars $\alpha$,
3. $\|x + y\| \leq \|x\| + \|y\|$ (triangle inequality).

Every normed space is a metric space under $d(x, y) = \|x - y\|$,
so all of the metric-space machinery from `real-analysis.md`
(convergence, Cauchy sequences, completeness, compactness)
applies directly.

Standard examples: $\mathbb{R}^n$ under any of the $\ell^p$
norms $\|x\|_p = (\sum |x_i|^p)^{1/p}$ or the sup-norm
$\|x\|_\infty = \max_i |x_i|$; the space $C([a,b])$ of
continuous functions under the sup-norm $\|f\|_\infty =
\sup_{t \in [a,b]} |f(t)|$; the $L^p$ spaces (§2 below).

A key distinction from finite dimensions: in $\mathbb{R}^n$,
all norms are equivalent (they induce the same topology and
the same notion of convergence). In infinite dimensions,
different norms are generally inequivalent and generate
genuinely different topologies. Convergence in one norm need
not imply convergence in another.

### 1.2 Banach spaces

A *Banach space* is a complete normed space: every Cauchy
sequence converges to a limit inside the space. Completeness
is to normed spaces what the least-upper-bound property is to
$\mathbb{R}$ — the fundamental regularity condition that makes
analysis work.

Key examples: any $\mathbb{R}^n$ with any norm; $C([a,b])$
under the sup-norm; the $L^p$ spaces (proved in §2); the space
$\ell^p$ of sequences $(x_n)$ with $\sum |x_n|^p < \infty$.

Non-example: the space of polynomials on $[0,1]$ under the
sup-norm is not Banach, because Cauchy sequences of
polynomials can converge to continuous functions that are not
polynomials.

**Series in Banach spaces.** A series $\sum_n x_n$ in a
Banach space converges if its partial sums form a Cauchy
sequence. The series converges *absolutely* if $\sum_n \|x_n\|
< \infty$. In a Banach space, absolute convergence implies
convergence. This is used in the Neumann series for operator
inverses (§4.2) and in the proof of $L^p$ completeness (§2.2).

---

## 2. $L^p$ spaces

### 2.1 Definition and basic inequalities

Fix a measure space $(\Omega, \mathcal{F}, \mu)$. For $1 \leq
p < \infty$, $L^p(\mu)$ consists of (equivalence classes of)
measurable functions with
$$\|f\|_p = \left(\int |f|^p \, d\mu\right)^{1/p} < \infty.$$
For $p = \infty$: $L^\infty(\mu)$ consists of essentially
bounded functions with
$$\|f\|_\infty = \inf\{M \geq 0 : |f| \leq M \text{ a.e.}\}.$$
Functions differing on null sets are identified.

**Hölder's inequality.** For conjugate exponents $p, q$ with
$1/p + 1/q = 1$:
$$\int |fg| \, d\mu \leq \|f\|_p \|g\|_q.$$
For $p = q = 2$ this is the Cauchy-Schwarz inequality. The
proof uses Young's inequality $ab \leq a^p/p + b^q/q$ (for
$a, b \geq 0$).

**Minkowski's inequality.** For $p \geq 1$:
$$\|f + g\|_p \leq \|f\|_p + \|g\|_p.$$
This is the triangle inequality for the $L^p$ norm. The proof
applies Hölder to $|f+g|^p \leq |f||f+g|^{p-1} +
|g||f+g|^{p-1}$.

### 2.2 Completeness: the Riesz-Fischer theorem

**Theorem (Riesz-Fischer).** For $1 \leq p \leq \infty$,
$L^p(\mu)$ is a Banach space.

**Proof (for $1 \leq p < \infty$).** Let $(f_n)$ be Cauchy in
$L^p$. Extract a subsequence $(f_{n_k})$ with $\|f_{n_{k+1}}
- f_{n_k}\|_p \leq 2^{-k}$. Define the partial sums
$$g_N = \sum_{k=1}^N |f_{n_{k+1}} - f_{n_k}|.$$
By Minkowski's inequality, $\|g_N\|_p \leq \sum_{k=1}^N 2^{-k}
\leq 1$. The sequence $g_N$ is nonneg and increasing, with
$L^p$ norms bounded above. By the monotone convergence theorem
(`measure-theory.md` §6.1), $g := \lim g_N$ satisfies $\|g\|_p
\leq 1$. In particular, $g$ is finite $\mu$-a.e.

Where $g < \infty$, the telescoping series
$$f_{n_1}(x) + \sum_{k=1}^\infty (f_{n_{k+1}}(x) - f_{n_k}(x))$$
converges absolutely, so $\lim_k f_{n_k}(x) =: f(x)$ exists
for a.e. $x$. The function $f$ is measurable.

For large $k$: $|f_{n_k} - f| \leq 2g$ a.e., so by the
dominated convergence theorem (with dominating function
$(2g)^p \in L^1$), $\|f_{n_k} - f\|_p \to 0$. Since the
original sequence $(f_n)$ is Cauchy and a subsequence converges
to $f$ in $L^p$, the full sequence $f_n \to f$ in $L^p$.
$\blacksquare$

**Why this proof matters.** The proof reveals the structure
of $L^p$ completeness: it is the monotone convergence theorem
applied to the tails of a series. The MCT is what converts
a.e. convergence of partial sums into $L^p$ convergence of the
series — and hence into completeness. This is the first place
where the measure-theory tools and the functional-analytic
goals mesh visibly.

### 2.3 Moment conditions and $L^p$ inclusions

For a probability measure ($\mu(\Omega) = 1$): if $p > q \geq 1$,
then $L^p \subseteq L^q$ (by Jensen's inequality), and
$\|f\|_q \leq \|f\|_p$. Higher moments are stronger conditions.

The practical content: the "finite second moments" condition
in econometrics says $E[X^2] < \infty$, i.e., $X \in L^2$.
The LLN under Kolmogorov requires $X \in L^1$. The CLT
requires $X \in L^2$. The "bounded support" assumption that
appears in many proofs puts $X \in L^\infty$ — stronger than
needed, but it implies uniformly bounded tails and simplifies
the domination conditions in DCT-based arguments.

---

## 3. Hilbert spaces

### 3.1 Inner products

An *inner product space* is a vector space $X$ with a map
$\langle \cdot, \cdot \rangle: X \times X \to \mathbb{R}$
satisfying:
1. $\langle x, x \rangle \geq 0$ with equality iff $x = 0$,
2. $\langle x, y \rangle = \langle y, x \rangle$ (symmetry),
3. $\langle \alpha x + \beta y, z \rangle = \alpha \langle x, z \rangle + \beta \langle y, z \rangle$ (linearity in the first argument).

The inner product induces a norm $\|x\| = \langle x, x
\rangle^{1/2}$, and the Cauchy-Schwarz inequality holds:
$|\langle x, y \rangle| \leq \|x\| \|y\|$.

A *Hilbert space* is an inner product space that is complete
under the induced norm — a Banach space with an inner product.

Key examples:
- $\mathbb{R}^n$ with $\langle x, y \rangle = x^\top y$.
- $L^2(\mu)$ with $\langle f, g \rangle = \int fg \, d\mu$.
  By Cauchy-Schwarz (Hölder with $p = q = 2$), the integral
  is finite for $f, g \in L^2$. This is the central example
  for all the applications below.
- $\ell^2$ — square-summable sequences — with $\langle
  (x_n), (y_n) \rangle = \sum_n x_n y_n$.

### 3.2 The projection theorem

**Theorem (Projection onto a closed convex set).** Let $H$ be
a Hilbert space and $C \subseteq H$ a nonempty closed convex
set. For each $x \in H$, there exists a unique $P_C x \in C$
minimizing $\|x - y\|$ over $y \in C$.

**Proof.** Let $d = \inf_{y \in C} \|x - y\|$. Pick $(y_n)
\subseteq C$ with $\|x - y_n\| \to d$. The *parallelogram law*
holds in any inner product space:
$$\|u + v\|^2 + \|u - v\|^2 = 2\|u\|^2 + 2\|v\|^2.$$
Apply it with $u = x - y_n$ and $v = x - y_m$:
$$\|y_n - y_m\|^2 = 2\|x - y_n\|^2 + 2\|x - y_m\|^2 - 4\left\|x - \tfrac{y_n + y_m}{2}\right\|^2.$$
Since $C$ is convex, $(y_n + y_m)/2 \in C$, so $\|x -
(y_n + y_m)/2\| \geq d$. As $n, m \to \infty$, the right
side is at most $2d^2 + 2d^2 - 4d^2 = 0$. So $(y_n)$ is
Cauchy; it converges (by completeness) to some $P_C x \in C$
(by closedness) with $\|x - P_C x\| = d$.

Uniqueness: if $y_1, y_2 \in C$ both achieve distance $d$,
applying the parallelogram law gives $\|y_1 - y_2\|^2 \leq 0$,
so $y_1 = y_2$. $\blacksquare$

**The linear case.** If $M \subseteq H$ is a closed linear
subspace (a convex set), the minimizer $P_M x$ satisfies the
*orthogonality condition*: $x - P_M x \perp M$, meaning
$\langle x - P_M x, y \rangle = 0$ for all $y \in M$.

This is OLS. Let $H = \mathbb{R}^n$ and $M = \mathrm{col}(X)$
(the column space of the design matrix, a closed subspace
since $\mathbb{R}^n$ is finite-dimensional). The OLS fitted
values $\hat y = X\hat\beta = P_M y$ are the projection of $y$
onto $M$. The residual $y - \hat y$ is orthogonal to every
column of $X$ — this is the normal equations $X^\top(y -
X\hat\beta) = 0$. OLS is not a computational technique; it
is a geometric projection, and all of its properties (the
Gauss-Markov theorem, the partition of variance, the
geometry of added variable plots) follow from the projection
theorem.

In infinite dimensions: the conditional expectation $E[Y | X]$
is the projection of $Y \in L^2$ onto the closed subspace
$L^2(\sigma(X))$ of $\sigma(X)$-measurable square-integrable
functions (§9). The linear projection $L(Y | X) = \alpha +
\beta X$ is the projection onto the much smaller subspace of
affine functions of $X$. The difference between OLS and
nonparametric regression is the choice of subspace.

### 3.3 Orthonormal bases and Parseval's identity

A set $\{e_n\}_{n \geq 1}$ in $H$ is *orthonormal* if
$\langle e_m, e_n \rangle = \delta_{mn}$. It is a *Hilbert
basis* (complete orthonormal system) if $\overline{\mathrm{span}\{e_n\}} = H$ (the closed linear span is all of $H$).

**Bessel's inequality.** For any $x \in H$ and any orthonormal
set $\{e_n\}$: $\sum_n |\langle x, e_n \rangle|^2 \leq \|x\|^2$.

**Parseval's identity.** If $\{e_n\}$ is a Hilbert basis:
$$\sum_{n=1}^\infty |\langle x, e_n \rangle|^2 = \|x\|^2 \quad \text{and} \quad x = \sum_{n=1}^\infty \langle x, e_n \rangle e_n.$$

The coefficients $\langle x, e_n \rangle$ are the generalized
Fourier coefficients of $x$. Parseval's identity says the map
$x \mapsto (\langle x, e_n \rangle)_{n \geq 1}$ is an
isometric isomorphism from $H$ to $\ell^2$. Every separable
Hilbert space is isometrically isomorphic to $\ell^2$.

For $L^2([0, 2\pi])$, the trigonometric system $\{e^{int}/\sqrt{2\pi}\}_{n \in \mathbb{Z}}$ is a Hilbert basis, and Parseval's identity is the classical statement that the $\ell^2$ norm of the Fourier coefficients equals the $L^2$ norm of the function.

### 3.4 Riesz representation theorem (Hilbert space version)

**Theorem.** Let $H$ be a Hilbert space and $\ell: H \to
\mathbb{R}$ a bounded linear functional. Then there exists a
unique $z \in H$ such that $\ell(x) = \langle x, z \rangle$
for all $x \in H$, and $\|\ell\|_{H^*} = \|z\|_H$.

**Proof.** If $\ell = 0$, take $z = 0$. Otherwise,
$\ker(\ell) = \{x : \ell(x) = 0\}$ is a proper closed
subspace. Pick $w \notin \ker(\ell)$ and let $v = w -
P_{\ker(\ell)} w \neq 0$; then $v \perp \ker(\ell)$. For any
$x \in H$, the element
$$x - \frac{\ell(x)}{\ell(v)} v$$
lies in $\ker(\ell)$ (direct verification: $\ell$ of this
element is $\ell(x) - \frac{\ell(x)}{\ell(v)}\ell(v) = 0$),
so it is perpendicular to $v$:
$$0 = \left\langle x - \frac{\ell(x)}{\ell(v)} v,\, v \right\rangle = \langle x, v \rangle - \frac{\ell(x)}{\ell(v)} \|v\|^2.$$
Solving: $\ell(x) = \langle x, z \rangle$ where $z =
\frac{\ell(v)}{\|v\|^2} v$. Uniqueness follows from: if
$\langle x, z - z' \rangle = 0$ for all $x$, take $x =
z - z'$ to get $\|z - z'\|^2 = 0$. $\blacksquare$

**Content.** The dual space of a Hilbert space is the Hilbert
space itself. Every bounded linear functional is an inner
product with a fixed element. Hilbert spaces are *self-dual*.

This is the engine behind the Radon-Nikodym proof in §7 and
behind the semiparametric efficiency bound: the influence
function of an efficient estimator is characterized as an
element of the Hilbert space of mean-zero square-integrable
functions, and the Riesz representation theorem guarantees
it exists.

---

## 4. Bounded linear operators

### 4.1 Operator norms and the space $\mathcal{B}(X, Y)$

A map $T: X \to Y$ between normed spaces is *linear* if
$T(\alpha x + \beta y) = \alpha Tx + \beta Ty$. It is *bounded*
if $\|T\| := \sup_{\|x\| \leq 1} \|Tx\|_Y < \infty$. For
linear maps, bounded is equivalent to continuous.

The space $\mathcal{B}(X, Y)$ of all bounded linear operators
is a normed space under the operator norm; when $Y$ is Banach,
$\mathcal{B}(X, Y)$ is Banach. Composition satisfies
$\|ST\| \leq \|S\|\|T\|$.

**The adjoint.** For $T \in \mathcal{B}(H, K)$ between Hilbert
spaces, the *adjoint* $T^* \in \mathcal{B}(K, H)$ is the
unique operator satisfying $\langle Tx, y \rangle_K = \langle
x, T^*y \rangle_H$ for all $x, y$. Existence follows from
the Riesz representation theorem: the map $x \mapsto \langle
Tx, y \rangle$ is a bounded linear functional on $H$, so it
equals $\langle x, T^*y \rangle$ for a unique $T^*y \in H$.
Key identities: $\|T^*\| = \|T\|$ and $\|T^*T\| = \|T\|^2$.

An operator $T \in \mathcal{B}(H)$ is *self-adjoint* if $T^*
= T$. Covariance operators, the Hessian of a quadratic form
(viewed as an operator on $\mathbb{R}^n$), and kernel integral
operators with symmetric kernels are self-adjoint.

### 4.2 The Neumann series

**Theorem.** If $T \in \mathcal{B}(X)$ with $\|T\| < 1$, then
$I - T$ is invertible in $\mathcal{B}(X)$ with
$$(I - T)^{-1} = \sum_{n=0}^\infty T^n,$$
convergent in operator norm, and $\|(I-T)^{-1}\| \leq
1/(1 - \|T\|)$.

**Proof.** Absolute convergence: $\sum_{n=0}^\infty \|T^n\|
\leq \sum_{n=0}^\infty \|T\|^n = 1/(1-\|T\|) < \infty$.
Since $\mathcal{B}(X)$ is Banach (when $X$ is), absolute
convergence implies convergence. The identity $(I - T)
\sum_{n=0}^N T^n = I - T^{N+1} \to I$ as $N \to \infty$
(since $\|T^{N+1}\| \leq \|T\|^{N+1} \to 0$). $\blacksquare$

Applications: perturbation theory (invertibility persists under
small perturbations — if $T$ is invertible and $\|S - T\| <
\|T^{-1}\|^{-1}$, then $S$ is invertible); the finite-sample
approximation of the Beveridge-Nelson decomposition in time
series; and the contraction mapping argument from
`real-analysis.md` §6 (which is the Neumann series for the
fixed-point iterate).

---

## 5. The big four theorems

These four results — and the Baire category theorem that
underlies three of them — are the structural facts of
functional analysis. Their content is genuinely about
infinite-dimensional completeness; they have no finite-
dimensional analog because every linear map on $\mathbb{R}^n$
is automatically bounded and the pathologies they rule out
simply do not arise.

### 5.1 Baire category theorem (prerequisite)

**Theorem.** A complete metric space cannot be written as a
countable union of nowhere-dense sets.

A set is *nowhere dense* if its closure has empty interior.
"Countable union of nowhere-dense sets" is the definition of a
*meager* (or first-category) set. The theorem says complete
metric spaces are non-meager — they are "large" in this
topological sense.

**Proof.** Suppose $X = \bigcup_{n=1}^\infty A_n$ with each
$\bar A_n$ nowhere dense. We construct a Cauchy sequence
whose limit avoids every $A_n$.

Start with any open ball $B_0$. Since $\bar A_1$ has empty
interior, $B_0 \not\subseteq \bar A_1$, so there is an open
ball $B_1 \subseteq B_0 \setminus \bar A_1$ with radius
$r_1 \leq 1/2$. Since $\bar A_2$ has empty interior, there
is an open ball $B_2 \subseteq B_1 \setminus \bar A_2$ with
radius $r_2 \leq 1/4$. Continue: pick $B_{n+1} \subseteq
B_n \setminus \bar A_{n+1}$ with $r_{n+1} \leq 2^{-(n+1)}$.

The centers of $(B_n)$ form a Cauchy sequence (they are in
$B_N$ for all $n \geq N$, and $\mathrm{diam}(B_N) \to 0$).
By completeness, the centers converge to some $x \in X$.
Since $B_n$ is closed and $x \in B_n$ for all large $n$,
we have $x \notin A_n$ for any $n$ — contradicting
$X = \bigcup A_n$. $\blacksquare$

### 5.2 Uniform boundedness principle (Banach-Steinhaus)

**Theorem.** Let $X$ be a Banach space, $Y$ a normed space,
and $\{T_\alpha\} \subseteq \mathcal{B}(X, Y)$ a family of
bounded operators. If $\sup_\alpha \|T_\alpha x\| < \infty$
for every $x \in X$ (pointwise boundedness), then
$\sup_\alpha \|T_\alpha\| < \infty$ (uniform boundedness).

**Proof.** For $n \geq 1$, let
$$A_n = \{x \in X : \|T_\alpha x\| \leq n \text{ for all } \alpha\} = \bigcap_\alpha T_\alpha^{-1}(\bar B_Y(0,n)).$$
Each $A_n$ is closed (intersection of closed sets). By
hypothesis, $X = \bigcup_n A_n$. By the Baire category
theorem, some $A_N$ has nonempty interior: there exist
$x_0 \in X$ and $r > 0$ with $\bar B(x_0, r) \subseteq A_N$.

For any $x$ with $\|x\| \leq r$, the point $x_0 + x$ lies
in $\bar B(x_0, r) \subseteq A_N$, so $\|T_\alpha(x_0 + x)\|
\leq N$ for all $\alpha$. By linearity:
$$\|T_\alpha x\| \leq \|T_\alpha(x_0 + x)\| + \|T_\alpha x_0\| \leq N + N = 2N.$$
Scaling: $\|T_\alpha(x/r)\| \leq 2N/r$ for all unit vectors,
so $\|T_\alpha\| \leq 2N/r$ for all $\alpha$. $\blacksquare$

**Why this matters.** Pointwise boundedness (each $x$ is
handled) implies uniform boundedness (a single constant works
everywhere). In asymptotic econometrics: if a sequence of
functionals $\ell_n$ (e.g., empirical process functionals)
is pointwise bounded, it is uniformly bounded. The uniform
boundedness principle is what licenses replacing a pointwise
bound with the stronger uniform bound that rigorous limit
theorems require. It also implies: a sequence of operators
that converges pointwise must converge to a bounded operator
(the Banach-Steinhaus corollary).

### 5.3 Open mapping theorem

**Theorem.** Let $X$ and $Y$ be Banach spaces and $T \in
\mathcal{B}(X, Y)$ be surjective. Then $T$ is an open map:
the image of every open set is open.

**Corollary (Bounded inverse theorem).** If $T \in
\mathcal{B}(X, Y)$ is bijective, then $T^{-1} \in
\mathcal{B}(Y, X)$.

**Proof sketch.** Surjectivity means $Y = \bigcup_{n \geq 1}
\overline{T(nB_X)}$ where $B_X$ is the open unit ball of $X$.
By Baire, some $\overline{T(NB_X)}$ has nonempty interior,
containing an open ball $B_Y(y_0, \varepsilon)$. One shows
(using completeness) that $T(B_X)$ itself contains an open
ball around 0, and then $T$ maps open sets to sets with
nonempty interior. The bounded inverse theorem follows because
$T^{-1}$ maps bounded sets to bounded sets (the preimage of
a bounded set is bounded). $\blacksquare$

**Applications.** The bounded inverse theorem says there is no
"one-way" continuous bijection between Banach spaces. In
structural identification: if the mapping from structural
parameters $\theta$ to reduced-form observables is a
continuous bijection between Banach spaces, the inverse
(identification mapping) is also continuous.

### 5.4 Closed graph theorem

**Theorem.** Let $X$ and $Y$ be Banach spaces and
$T: X \to Y$ linear. Then $T$ is bounded if and only if its
graph $\Gamma(T) = \{(x, Tx) : x \in X\}$ is closed in
$X \times Y$.

**Proof sketch.** Forward direction: if $T$ is bounded and
$(x_n, Tx_n) \to (x, y)$, then $Tx_n \to Tx$ by continuity,
so $y = Tx$ and $(x, y) \in \Gamma(T)$. Converse: if
$\Gamma(T)$ is closed, then $X \times Y / \Gamma(T)$ and the
projection maps satisfy the hypotheses of the open mapping
theorem, giving boundedness of $T$. $\blacksquare$

**Applications.** The closed graph theorem lets you verify
boundedness by verifying that "whenever a sequence converges
and the images converge, the images converge to what they
should." It is used in proofs that specific operators (like
the composition operator in function spaces) are bounded.

### 5.5 Hahn-Banach theorem

**Theorem (real version).** Let $X$ be a real normed space,
$M \subseteq X$ a subspace, and $\ell: M \to \mathbb{R}$ a
bounded linear functional. Then there exists a bounded linear
extension $\tilde\ell: X \to \mathbb{R}$ with $\|\tilde\ell\|
= \|\ell\|$.

**Proof sketch.** The proof proceeds by Zorn's lemma. Consider
all bounded linear extensions of $\ell$ to subspaces of $X$
(with the same norm bound), ordered by inclusion of domains.
Every chain has an upper bound (define the functional on the
union of domains, which is a subspace). A maximal element must
be defined on all of $X$: if it were defined only on a proper
subspace $M'$, pick any $x \notin M'$ and show the functional
extends to $M' + \mathbb{R}x$ while preserving the norm bound
— a one-dimensional extension argument using the norm
condition. $\blacksquare$

**Geometric corollary (separation theorem).** If $C \subseteq X$
is a nonempty closed convex set and $x_0 \notin C$, there
exists a bounded linear functional $\ell$ and a constant $c$
such that $\ell(x_0) > c \geq \ell(x)$ for all $x \in C$.
Closed convex sets can be separated from exterior points by
hyperplanes. This is the geometric heart of duality theory in
optimization (`optimization.md` §4): the Lagrangian dual
function lower-bounds the primal via this separation.

**Applications.** Hahn-Banach is the non-constructive workhorse
of functional analysis:
- Ensures the dual space $X^*$ is "large enough" to separate
  points of $X$: if $x \neq y$, there exists $\ell \in X^*$
  with $\ell(x) \neq \ell(y)$. Without this, the dual space
  would be too small to characterize elements of $X$.
- Underpins infinite-dimensional linear programming duality
  (the extension from finite LP duality).
- The moment problem: a measure is characterized by its
  moments via Hahn-Banach applied to the space of bounded
  continuous functions.

---

## 6. Dual spaces and weak convergence

### 6.1 The dual space

For a normed space $X$, the *dual space* $X^*$ is the Banach
space of all bounded linear functionals on $X$, with the
operator norm $\|\ell\|_{X^*} = \sup_{\|x\| \leq 1} |\ell(x)|$.

Key duality results:
- For $1 < p < \infty$: $(L^p)^* \cong L^q$ where $1/p + 1/q
  = 1$. Every bounded linear functional on $L^p$ is of the
  form $f \mapsto \int fg \, d\mu$ for a unique $g \in L^q$.
  This is proved using Radon-Nikodym (§7) applied to the
  signed measure $\nu(A) = \ell(\mathbf{1}_A)$.
- $(L^1)^* \cong L^\infty$ under σ-finiteness.
- $(L^\infty)^* \supsetneq L^1$: the dual of $L^\infty$ is
  strictly larger, including finitely additive measures
  ("charges"). This is why $L^\infty$ is harder to work with
  — its dual escapes the function-space setting.
- $(L^2)^* \cong L^2$: the Riesz representation theorem from
  §3.4 applied to $L^2$.
- $(\ell^p)^* \cong \ell^q$ for $1 \leq p < \infty$.

The *double dual* $X^{**}$ contains $X$ as a canonical
subspace via $x \mapsto [\ell \mapsto \ell(x)]$. If $X =
X^{**}$, then $X$ is *reflexive*. All $L^p$ for $1 < p <
\infty$ are reflexive; $L^1$ and $L^\infty$ are not.

### 6.2 Weak convergence

*Strong convergence*: $x_n \to x$ means $\|x_n - x\| \to 0$.

*Weak convergence*: $x_n \rightharpoonup x$ means $\ell(x_n)
\to \ell(x)$ for every $\ell \in X^*$.

Strong implies weak; the converse fails in infinite dimensions.
The standard counterexample: the sequence of indicator
functions $(\mathbf{1}_{[n, n+1]})$ in $L^2(\mathbb{R})$ has
$\|\mathbf{1}_{[n,n+1]}\|_2 = 1$ but converges weakly to 0.

**Banach-Alaoglu theorem.** The closed unit ball of $X^*$ is
weak-* compact (compact in the topology where $\ell_n
\overset{*}{\rightharpoonup} \ell$ means $\ell_n(x) \to \ell(x)$
for all $x \in X$). In reflexive spaces, the unit ball is also
weakly compact: every bounded sequence has a weakly convergent
subsequence.

**Why weak convergence matters.** In nonparametric estimation,
the parameter of interest is an element of a function space
(e.g., a density, a regression function, a distribution
function). Consistency proofs establish weak convergence of
the estimator in that function space. The empirical process
literature (uniform law of large numbers, functional CLT)
works in $\ell^\infty(\mathcal{F})$ — the space of bounded
functionals on a class $\mathcal{F}$ — and weak convergence
there corresponds to convergence of the finite-dimensional
distributions plus asymptotic tightness. The distinction
between weak convergence and strong convergence in function
spaces corresponds to convergence in distribution versus
convergence in probability at the scalar level.

---

## 7. The Radon-Nikodym theorem via Riesz

This is the proof deferred from `measure-theory.md` §9.4. The
Hilbert space machinery makes it clean.

**Theorem (Radon-Nikodym).** Let $(\Omega, \mathcal{F}, \mu)$
be a σ-finite measure space and $\nu$ a σ-finite measure on
$(\Omega, \mathcal{F})$ that is *absolutely continuous* with
respect to $\mu$ ($\nu \ll \mu$: $\nu(A) = 0$ whenever
$\mu(A) = 0$). Then there exists a nonneg measurable function
$f$ such that
$$\nu(A) = \int_A f \, d\mu \quad \text{for all } A \in \mathcal{F}.$$
The function $f = d\nu/d\mu$ (the Radon-Nikodym derivative)
is $\mu$-a.e. unique.

**Proof.** Handle the finite case first ($\mu(\Omega), \nu(\Omega) < \infty$).

Let $\lambda = \mu + \nu$. For $g \in L^2(\lambda)$, define
$$\ell(g) = \int g \, d\nu.$$
This is a bounded linear functional on $L^2(\lambda)$: by
Cauchy-Schwarz, $|\ell(g)| \leq \nu(\Omega)^{1/2}
\|g\|_{L^2(\lambda)}$. By the Riesz representation theorem
(§3.4), there exists $h \in L^2(\lambda)$ such that
$$\int g \, d\nu = \langle g, h \rangle_{L^2(\lambda)} = \int gh \, d\lambda = \int gh \, d\mu + \int gh \, d\nu$$
for all $g \in L^2(\lambda)$. Rearranging:
$$\int g(1 - h) \, d\nu = \int gh \, d\mu \quad \text{for all } g \in L^2(\lambda). \tag{$*$}$$

We show $0 \leq h < 1$ $\lambda$-a.e. Taking $g =
\mathbf{1}_{\{h \geq 1\}}$ in $(*)$: $\int_{\{h \geq 1\}}
(1-h) \, d\nu = \int_{\{h \geq 1\}} h \, d\mu \geq 0$. But
$(1-h) \leq 0$ on this set, so the left side is $\leq 0$.
Thus $\nu(\{h \geq 1\}) = 0$ and $\mu(\{h \geq 1\}) = 0$,
hence $\lambda(\{h \geq 1\}) = 0$. Similarly, taking $g =
\mathbf{1}_{\{h < 0\}}$ shows $\lambda(\{h < 0\}) = 0$.
So $0 \leq h < 1$ $\lambda$-a.e.

On the set $\{h < 1\}$ (which has full $\lambda$-measure),
divide $(*)$ by $(1 - h)$ and set $g = f_0/(1-h)$ for
nonneg $f_0$: one recovers $\int f_0 \, d\nu = \int f_0 \cdot
\frac{h}{1-h} \, d\mu$. The function $f = h/(1-h)$ is nonneg
and measurable, and satisfies $d\nu = f \, d\mu$.

The σ-finite case: write $\Omega = \bigsqcup_k \Omega_k$
with $\mu(\Omega_k) < \infty$ and $\nu(\Omega_k) < \infty$.
Apply the finite case on each $\Omega_k$ to get $f_k$, and
set $f = \sum_k f_k \mathbf{1}_{\Omega_k}$. $\blacksquare$

**Applications.**

*Likelihood ratios.* If $P$ and $Q$ are probability measures
with $P \ll Q$, then $f = dP/dQ$ is the likelihood ratio. The
Neyman-Pearson lemma, the information inequality, and the
construction of locally minimax tests all use this.

*Conditional expectation.* The conditional expectation
$E[X | \mathcal{G}]$ is the Radon-Nikodym derivative of the
signed measure $\nu(G) = E[X \cdot \mathbf{1}_G]$ (for $G
\in \mathcal{G}$) with respect to $P|_\mathcal{G}$. This is
the formal definition when $X$ is not necessarily
$\mathcal{G}$-measurable (§9 below).

*Change of measure.* Girsanov's theorem expresses the
Radon-Nikodym derivative of a probability measure under a
change of drift in a diffusion, and is the engine behind
risk-neutral pricing in continuous-time finance.

---

## 8. Compact operators and the spectral theorem

### 8.1 Compact operators

An operator $T \in \mathcal{B}(X, Y)$ is *compact* if it maps
bounded sets to precompact sets (sets with compact closure).
Equivalently: every bounded sequence $(x_n)$ has a subsequence
$(x_{n_k})$ such that $(Tx_{n_k})$ converges.

Compact operators are "almost finite-rank." In infinite
dimensions, the identity operator is not compact (the image
of the unit ball — the unit ball itself — is not compact).
Compact operators "compress" the space.

Key examples:
- Any finite-rank operator is compact.
- *Hilbert-Schmidt operators*: $Tf(x) = \int K(x,y) f(y)\,
  d\mu(y)$ with $\iint |K(x,y)|^2 \, d\mu \, d\mu < \infty$
  is compact on $L^2(\mu)$. Sample covariance operators of
  random processes in $L^2$ are Hilbert-Schmidt.
- An operator is compact if and only if it is the norm-limit
  of a sequence of finite-rank operators.

### 8.2 Spectral theorem for compact self-adjoint operators

**Theorem.** Let $H$ be a separable Hilbert space and $T \in
\mathcal{B}(H)$ compact and self-adjoint. Then:

1. All eigenvalues of $T$ are real.
2. There are at most countably many eigenvalues $\lambda_1,
   \lambda_2, \ldots$, each nonzero, accumulating only at 0.
3. Eigenvectors for distinct eigenvalues are orthogonal.
4. The eigenvectors $\{e_n\}$ can be chosen orthonormal and
   form a Hilbert basis for $\overline{\mathrm{range}(T)}$.
5. The spectral expansion: $Tx = \sum_n \lambda_n \langle
   x, e_n \rangle e_n$, convergent in $H$.

**Proof sketch.** Self-adjoint implies real eigenvalues: if
$Tx = \lambda x$ then $\lambda \|x\|^2 = \langle Tx, x
\rangle = \langle x, Tx \rangle = \lambda \|x\|^2$, so
$\lambda \in \mathbb{R}$.

For the first eigenvalue: $\|T\| = \sup_{\|x\|=1} |\langle Tx,
x \rangle|$ (a consequence of self-adjointness). Let $(x_n)$
be a maximizing sequence with $|\langle Tx_n, x_n \rangle|
\to \|T\|$. By compactness, a subsequence $(Tx_{n_k})$
converges; one shows the subsequence $(x_{n_k})$ also
converges to an eigenvector $e_1$ with eigenvalue
$\lambda_1 = \|T\|$ or $-\|T\|$.

Restrict to $e_1^\perp$ (which $T$ maps to itself, by
self-adjointness) and iterate. The iteration produces a
sequence of eigenvalues with $|\lambda_n| \to 0$ (because
the eigenvectors $e_n$ are orthonormal, so $Te_n = \lambda_n
e_n \rightharpoonup 0$ weakly, and $\|T e_n\| = |\lambda_n|
\to 0$ by compactness — compact operators map weakly
convergent sequences to strongly convergent ones).
$\blacksquare$

**Applications in econometrics.**

*Functional PCA.* When observations are random functions in
$L^2([0,1])$ (functional data), the covariance operator
$\mathcal{K}f = E[\langle X, f \rangle X]$ is a compact
self-adjoint operator. Its eigenfunctions are the functional
principal components; its eigenvalues are the variances along
each direction. The spectral expansion of $\mathcal{K}$
gives the Karhunen-Loève representation of the random process.

*HAC and long-run variance.* The long-run covariance matrix
$\Sigma = \sum_{h=-\infty}^\infty \Gamma(h)$ (where $\Gamma$
is the autocovariance function) can be viewed as the spectral
density operator of the time series evaluated at frequency
zero. Consistent HAC estimation is consistent estimation of
this operator — or, in the scalar case, of $\Sigma$ as a
matrix-valued function of the data.

*Kernel methods and RKHS.* In DML and related methods, kernel
regression uses a reproducing kernel Hilbert space (RKHS),
where the kernel integral operator $T_K f(x) = \int K(x,y)
f(y) \, d\mu(y)$ is compact and self-adjoint when $K$ is
symmetric. The spectral expansion of $T_K$ provides the
natural basis for regularized estimation.

---

## 9. Conditional expectation as $L^2$ projection

Bringing together §3 (Hilbert space and projection) and §7
(Radon-Nikodym).

**Formal definition.** Let $(\Omega, \mathcal{F}, P)$ be a
probability space, $X \in L^1(P)$, and $\mathcal{G} \subseteq
\mathcal{F}$ a sub-σ-algebra. The *conditional expectation*
$E[X | \mathcal{G}]$ is the (a.s. unique) $\mathcal{G}$-measurable
function satisfying
$$\int_G E[X | \mathcal{G}] \, dP = \int_G X \, dP \quad \forall G \in \mathcal{G}.$$

**Existence via Radon-Nikodym.** The set function $\nu(G) =
\int_G X^+ \, dP$ is a finite measure on $(\Omega, \mathcal{G})$,
absolutely continuous with respect to $P|_\mathcal{G}$. By
Radon-Nikodym (§7), its derivative $d\nu/d(P|_\mathcal{G})$
exists and is $\mathcal{G}$-measurable. Do the same for $X^-$
and subtract. The result is $E[X | \mathcal{G}]$.

**The $L^2$ projection characterization.** When $X \in L^2(P)$:
the space $L^2(\Omega, \mathcal{G}, P)$ of $\mathcal{G}$-measurable
square-integrable functions is a closed subspace of
$L^2(\Omega, \mathcal{F}, P)$. The conditional expectation
$E[X | \mathcal{G}]$ is the orthogonal projection $P_{L^2(\mathcal{G})} X$:
the unique element of $L^2(\mathcal{G})$ minimizing
$E[(X - Y)^2]$ over all $\mathcal{G}$-measurable $Y \in L^2$.

**Proof.** The defining property $\int_G E[X|\mathcal{G}] \, dP
= \int_G X \, dP$ for all $G \in \mathcal{G}$ is exactly
$\langle E[X|\mathcal{G}], \mathbf{1}_G \rangle_{L^2} = \langle X,
\mathbf{1}_G \rangle_{L^2}$, i.e., $X - E[X|\mathcal{G}] \perp
\mathbf{1}_G$ for all $G \in \mathcal{G}$. Since the indicator
functions $\{\mathbf{1}_G : G \in \mathcal{G}\}$ are dense in
$L^2(\mathcal{G})$, this says $X - E[X|\mathcal{G}] \perp
L^2(\mathcal{G})$ — exactly the orthogonality condition for
the projection. $\blacksquare$

**The hierarchy of projections.** This gives a unified
picture:

- *OLS*: projection of $Y$ onto $\mathrm{span}\{1, X_1, \ldots, X_k\}$
  — a finite-dimensional subspace of $L^2$.
- *Linear projection* $L(Y | X)$: projection onto
  $\overline{\{a + bX : a, b \in \mathbb{R}\}}$, the closure
  of linear functions in $L^2$. Equals OLS when $X$ is
  the only regressor.
- *Conditional expectation* $E[Y | X]$: projection onto
  $L^2(\sigma(X))$, the (infinite-dimensional) subspace of
  all $\sigma(X)$-measurable square-integrable functions.
  This is nonparametric regression.
- *Semiparametric efficient estimator*: characterized as a
  projection (in the Hilbert space of influence functions)
  onto the tangent space of the model — a subspace defined
  by the model's score equations. The Riesz representation
  theorem guarantees this projection exists and gives the
  efficient influence function.

The efficiency bound in semiparametric models — the
variance floor that no regular estimator can beat — is the
squared norm of the efficient influence function in $L^2$.
The geometry of the $L^2$ space is what the semiparametric
efficiency theory is doing.

---

## 10. What this file doesn't cover, and where to find it

**Unbounded operators.** Differential operators (the Laplacian,
Schrödinger operator, Sturm-Liouville problems) are unbounded
on $L^2$ but still have a spectral theory via the Cayley
transform and the theory of self-adjoint extensions. Relevant
for mathematical physics; not standard in econometrics.

**Semigroup theory.** Operator semigroups are the continuous-
time analog of matrix exponentiation and appear in the rigorous
theory of diffusion processes and abstract Cauchy problems.
Relevant if the researcher works with continuous-time dynamic
models; not covered here.

**Distribution theory (Schwartz distributions).** The space
of tempered distributions is the dual of the Schwartz space
of rapidly decaying smooth functions. It is the natural setting
for Fourier transforms without integrability restrictions and
for weak solutions of PDEs. Not standard in econometrics.

**Sobolev spaces.** Function spaces with weak derivatives in
$L^p$, used in the theory of PDEs and in nonparametric
estimation rate theory (e.g., optimal rates for estimating
functions in Sobolev balls). A possible future addition if the
distributional methods or nonparametric work demands it.

**LLN and CLT in Banach spaces.** The laws of large numbers
and central limit theorems for Banach-space-valued random
variables — including the functional CLT (Donsker's theorem)
in $\ell^\infty(\mathcal{F})$ — are in `probability.md` and
`stochastic-processes.md`.

**Optimal transport.** The Wasserstein distance and optimal
transport maps use the Kantorovich duality, whose proof uses
Hahn-Banach applied to the space of continuous functions.
A possible future addition to the math layer.

---

## Cross-references

- `20_Math/real-analysis.md` — foundational prerequisite.
  Metric spaces, Cauchy completeness, and compactness (§§2-3
  there) are the prerequisites for Banach spaces (§§1-2 here);
  the contraction mapping theorem (§6 there) is the special
  case of the Neumann series (§4.2 here) for the Banach space
  of bounded functions.
- `20_Math/measure-theory.md` — the $L^p$ spaces here (§2)
  are the function spaces defined there. The convergence
  theorems there (MCT, DCT) appear in the Riesz-Fischer proof
  (§2.2). The Radon-Nikodym theorem is stated without proof in
  `measure-theory.md` §9.4 and is proved here in §7.
- `20_Math/optimization.md` — the Hahn-Banach theorem (§5.5
  here) is the infinite-dimensional engine behind the duality
  theory in `optimization.md` §4; the separation theorem here
  generalizes the Farkas-lemma argument used there for KKT.
- `20_Math/linear-algebra.md` — The finite-dimensional
  companion: spectral theorem for real symmetric matrices
  (specialization of the compact self-adjoint theorem),
  QR (Gram-Schmidt), SVD, pseudoinverse, Cholesky, and
  numerical linear algebra. The projection geometry of §3
  here (OLS as $L^2$ projection) is given its matrix form
  there.
- `20_Math/probability.md` — *to be written*. Builds directly
  on §9 here (conditional expectation as $L^2$ projection) for
  its treatment of the conditional expectation in probability
  theory, and on §6 (weak convergence) for convergence in
  distribution.
- `20_Math/stochastic-processes.md` — *to be written*.
  Martingale theory uses the $L^2$ projection characterization
  of conditional expectation (§9 here); the quadratic variation
  of a martingale lives in $L^1$.
- `20_Math/dynamic-programming.md` — *to be written*. The
  Bellman operator acts on the Banach space of bounded functions
  (sup-norm); the Neumann series (§4.2 here) and the
  contraction mapping theorem (`real-analysis.md` §6) jointly
  establish existence and uniqueness of the value function.
- `10_Methods/Econometrics/distributional-methods.md` —
  *to be written*. Recentered influence functions are elements
  of $L^2$; the semiparametric efficiency bound is the squared
  $L^2$-norm of the efficient influence function (§9 here).
- `10_Methods/Econometrics/double-machine-learning.md` —
  *to be written*. Neyman-orthogonal scores are defined via the
  Gateaux derivative in the direction of the nuisance function
  — a functional derivative in $L^2$ whose analysis uses the
  projection and Riesz representation structure of §3 here.
- `10_Methods/modern-toolkit-references.md` — the asymptotic
  theory entries (empirical process theory, uniform LLN,
  functional CLT) depend on the Banach-space LLN/CLT framework
  that this file sets up.
- `00_Identity/Principles.md` — §5 (estimation) and §6
  (causation) are grounded in the geometric view of OLS and
  conditional expectation developed in §§3 and 9 here.

---

*Last updated: April 2026.*
