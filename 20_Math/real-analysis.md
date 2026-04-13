---
tags: [math, real-analysis, foundations]
related: [modern-toolkit-references, Principles, optimization, measure-theory, functional-analysis]
---

# Real Analysis

This is the foundational file of the math layer. It covers the
part of real analysis that shows up in econ work: the
structural results about $\mathbb{R}$ and metric spaces that
underlie optimization, dynamic programming, comparative
statics, and the convergence arguments Markus runs into when
reading econometrics theory or checking a proof.

The file is a reference, not a textbook. It is organized by
mathematical structure but anchored throughout by why each
result matters for econ work: compactness is why optimization
problems have solutions, the implicit function theorem is why
comparative statics works, the contraction mapping theorem is
why dynamic programming terminates. Every section earns its
place by pointing to the econ work that depends on it.

Proofs are included for essentially all the non-trivial
theorems. This is a deliberate choice: Markus reaches for this
file not just to look up statements but to reproduce
arguments, and having the proofs in the file rather than
forwarding to an external text keeps the kit self-contained.
The proofs are compact — enough to verify the argument works,
not a pedagogical unfolding. For a linear first reading of
real analysis, Rudin's *Principles of Mathematical Analysis*
or Abbott's *Understanding Analysis* are the standard starting
points; this file assumes you have read one of them or an
equivalent, and serves as the working reference afterward.

The file stops before measure theory — Lebesgue integration,
the convergence theorems (dominated, monotone), Fubini, and
the measure-theoretic treatment of probability all go in
`measure-theory.md`. It also stops before functional analysis
— Banach spaces, Hilbert spaces, operator theory go in
`functional-analysis.md`. This file is the prerequisite for
both.

---

## 1. The real numbers and completeness

The real numbers $\mathbb{R}$ are the unique (up to
isomorphism) complete ordered field. "Complete" here is the
**least-upper-bound property**: every nonempty subset of
$\mathbb{R}$ that is bounded above has a supremum in
$\mathbb{R}$. This is the axiom that distinguishes
$\mathbb{R}$ from $\mathbb{Q}$ and it is the load-bearing
assumption for everything that follows.

Equivalent formulations of completeness that show up in
practice:

- **Every Cauchy sequence in $\mathbb{R}$ converges** (Cauchy
  completeness).
- **Every bounded monotone sequence in $\mathbb{R}$ converges**
  (monotone convergence for sequences).
- **Nested closed bounded intervals have nonempty
  intersection** (nested interval property).
- **Every bounded sequence has a convergent subsequence**
  (Bolzano-Weierstrass).

These are equivalent in the sense that, given the ordered
field axioms, any one implies the others. Different textbooks
take different ones as the axiom; the choice is a matter of
exposition, not mathematics.

**The supremum and infimum.** For a nonempty set
$S \subseteq \mathbb{R}$ bounded above, $\sup S$ is the least
upper bound: the unique number $s^*$ such that (i) $s^* \geq x$
for all $x \in S$, and (ii) for any $\varepsilon > 0$, there
exists $x \in S$ with $x > s^* - \varepsilon$. The second
condition is the one that matters in arguments — it says that
the supremum is approachable from within the set, even if it
is not attained.

The infimum is defined symmetrically as the greatest lower
bound. If $S$ is unbounded above, we write $\sup S = +\infty$
as a convention; the set has no supremum in $\mathbb{R}$ but
we treat $+\infty$ as a formal symbol. Likewise for
$\inf S = -\infty$.

**Sequences and limits.** A sequence $(x_n)_{n \geq 1}$ in
$\mathbb{R}$ converges to $L \in \mathbb{R}$ if: for every
$\varepsilon > 0$, there exists $N \in \mathbb{N}$ such that
$|x_n - L| < \varepsilon$ for all $n \geq N$. We write
$x_n \to L$ or $\lim_{n \to \infty} x_n = L$.

Basic facts Markus assumes without reproof: limits are
unique; sums, products, and (nonzero-denominator) quotients
of convergent sequences converge to the corresponding
operations on the limits; $|x_n - L| \to 0$ iff $x_n \to L$.

**Cauchy sequences.** A sequence $(x_n)$ is Cauchy if for
every $\varepsilon > 0$, there exists $N$ such that
$|x_n - x_m| < \varepsilon$ for all $n, m \geq N$. The
concept captures "the terms get close to each other without
needing to name a limit in advance." In $\mathbb{R}$, Cauchy
sequences are exactly the convergent sequences — this is
Cauchy completeness, and it is equivalent to the least-upper-
bound property.

### Proof: Bolzano-Weierstrass theorem

**Theorem.** Every bounded sequence in $\mathbb{R}$ has a
convergent subsequence.

**Proof.** Let $(x_n)$ be a bounded sequence, so there exist
$a, b$ with $a \leq x_n \leq b$ for all $n$. Set $I_0 = [a, b]$.

Bisect $I_0$ into two closed subintervals of equal length. At
least one of them contains infinitely many terms of the
sequence (by pigeonhole); call it $I_1$. Pick
$x_{n_1} \in I_1$.

Bisect $I_1$ similarly. At least one of the two halves
contains infinitely many terms of $(x_n)$ with index greater
than $n_1$; call it $I_2$, and pick $x_{n_2} \in I_2$ with
$n_2 > n_1$.

Continuing inductively, we obtain a nested sequence of
closed intervals $I_0 \supseteq I_1 \supseteq I_2 \supseteq
\cdots$, with length$(I_k) = (b-a)/2^k$, and a subsequence
$(x_{n_k})$ with $x_{n_k} \in I_k$ for each $k$.

By the nested interval property (which follows from
completeness), $\bigcap_{k=0}^\infty I_k$ is nonempty and
(since the lengths shrink to zero) contains exactly one point,
call it $L$. For any $\varepsilon > 0$, choose $K$ such that
$(b-a)/2^K < \varepsilon$. Then for all $k \geq K$, both
$x_{n_k}$ and $L$ lie in $I_k$, so
$|x_{n_k} - L| \leq (b-a)/2^k < \varepsilon$. Thus
$x_{n_k} \to L$. $\blacksquare$

This proof is the model for a whole family of arguments in
analysis: bisect, pick a subsequence, use completeness. Worth
internalizing for that reason.

---

## 2. Metric spaces and topology

Most of real analysis lives on $\mathbb{R}$ or $\mathbb{R}^n$,
but the right level of generality for the structural results
is **metric spaces**. This matters because the spaces that
show up in econ work — strategy spaces in game theory, state
spaces in dynamic programming, function spaces in
econometrics — are often metric spaces that are not
$\mathbb{R}^n$, and the theorems generalize to them without
modification.

A **metric space** is a pair $(X, d)$ where $X$ is a set and
$d: X \times X \to \mathbb{R}$ satisfies, for all
$x, y, z \in X$:

1. $d(x, y) \geq 0$, with equality iff $x = y$ (positivity).
2. $d(x, y) = d(y, x)$ (symmetry).
3. $d(x, z) \leq d(x, y) + d(y, z)$ (triangle inequality).

Standard examples: $\mathbb{R}^n$ with the Euclidean metric
$d(x, y) = \|x - y\|_2$; any set with the discrete metric
($d(x,y) = 1$ if $x \neq y$, else $0$); the space of
continuous functions on a compact interval with the sup metric
$d(f, g) = \sup_t |f(t) - g(t)|$.

**Convergence in a metric space.** A sequence $(x_n)$ in $X$
converges to $x \in X$ if $d(x_n, x) \to 0$ as $n \to
\infty$. The ε-δ definition generalizes with $d$ replacing
absolute value. Limits are unique when they exist.

**Open and closed sets.** For $x \in X$ and $r > 0$, the
**open ball** is $B_r(x) = \{y \in X : d(x, y) < r\}$. A set
$U \subseteq X$ is **open** if for every $x \in U$ there
exists $r > 0$ with $B_r(x) \subseteq U$. A set is **closed**
if its complement is open.

Equivalently: $F$ is closed iff every convergent sequence in
$F$ has its limit in $F$. This is often the useful
characterization.

**Compactness.** A set $K \subseteq X$ is **sequentially
compact** if every sequence in $K$ has a subsequence that
converges to a point in $K$. It is **(covering) compact** if
every open cover of $K$ has a finite subcover. In general
topological spaces these can differ; in metric spaces they
coincide.

### Proof: sequential compactness equals covering compactness in metric spaces

**Theorem.** Let $(X, d)$ be a metric space and $K \subseteq X$.
Then $K$ is sequentially compact iff $K$ is (covering) compact.

**Proof sketch.** ($\Rightarrow$) Suppose $K$ is sequentially
compact. We first show $K$ is **totally bounded**: for every
$\varepsilon > 0$, $K$ can be covered by finitely many balls of
radius $\varepsilon$. If not, there exists $\varepsilon > 0$
such that no finite collection of $\varepsilon$-balls covers
$K$, and we can inductively pick a sequence $x_1, x_2, \ldots$
in $K$ with $d(x_i, x_j) \geq \varepsilon$ for $i \neq j$. This
sequence has no convergent subsequence, contradicting
sequential compactness.

Next we show that every open cover has a **Lebesgue number**:
a $\delta > 0$ such that every subset of $K$ of diameter
$< \delta$ is contained in some cover element. Suppose not:
then for each $n$ there is a subset $A_n$ of diameter $< 1/n$
not contained in any cover element. Pick $x_n \in A_n$; by
sequential compactness, some subsequence $x_{n_k}$ converges
to $x \in K$. Since $x$ is in some open cover element $U$,
there is $r > 0$ with $B_r(x) \subseteq U$. For $k$ large,
$x_{n_k} \in B_{r/2}(x)$ and diameter$(A_{n_k}) < r/2$, so
$A_{n_k} \subseteq B_r(x) \subseteq U$, contradiction.

Now combine: total boundedness gives a finite
$\delta/2$-cover by balls of radius $\delta/2$, each of which
has diameter $< \delta$ and so is contained in some cover
element. Picking one cover element for each ball gives a
finite subcover.

($\Leftarrow$) Suppose $K$ is covering compact and let $(x_n)$
be a sequence in $K$. Suppose for contradiction no
subsequence converges in $K$. Then for each $x \in K$, there
is an open ball $B_{r_x}(x)$ containing only finitely many
terms of the sequence. The collection $\{B_{r_x}(x)\}$ covers
$K$, so has a finite subcover, but each element contains only
finitely many terms — contradiction, since the sequence has
infinitely many terms. $\blacksquare$

The proof is worth seeing because it exhibits the two main
tools of metric topology: total boundedness and Lebesgue
numbers. Both generalize and both show up again.

**The Heine-Borel theorem** is the special case that matters
most in practice: **a subset of $\mathbb{R}^n$ is compact iff
it is closed and bounded**. The forward direction is
straightforward (closed comes from the limit characterization
of compactness, bounded because unbounded sequences fail
sequential compactness). The converse uses Bolzano-Weierstrass
applied coordinate by coordinate.

**Completeness of metric spaces.** A metric space $(X, d)$ is
**complete** if every Cauchy sequence in $X$ converges to a
point in $X$. $\mathbb{R}$ and $\mathbb{R}^n$ are complete.
The space of continuous functions on a compact interval with
the sup metric is complete — this fact will be load-bearing
when we get to the contraction mapping theorem. Not every
metric space is complete: $\mathbb{Q}$ with its usual metric
is not.

Compactness implies completeness: a Cauchy sequence in a
compact set has a convergent subsequence (by sequential
compactness), and a Cauchy sequence with a convergent
subsequence converges to the same limit. The converse fails —
$\mathbb{R}$ is complete but not compact.

**Connectedness.** A set $E \subseteq X$ is **connected** if
it cannot be written as a disjoint union of two nonempty open
sets (relative to $E$). The connected subsets of $\mathbb{R}$
are exactly the intervals. Connectedness matters in econ work
mostly through the intermediate value theorem and through the
use of connected strategy spaces in game theory; it does not
need heavy treatment here.

---

## 3. Continuity

A function $f: X \to Y$ between metric spaces is **continuous
at $x_0 \in X$** if: for every $\varepsilon > 0$, there
exists $\delta > 0$ such that $d_Y(f(x), f(x_0)) < \varepsilon$
whenever $d_X(x, x_0) < \delta$. The function is
**continuous** if it is continuous at every point of its
domain.

Equivalent formulations:

- **Sequential.** $f$ is continuous at $x_0$ iff for every
  sequence $x_n \to x_0$ in $X$, $f(x_n) \to f(x_0)$ in $Y$.
- **Topological.** $f$ is continuous iff the preimage of
  every open set in $Y$ is open in $X$. Equivalently,
  preimage of closed is closed.

The sequential characterization is usually the most useful in
proofs; the topological one is the most general.

**Uniform continuity.** $f$ is **uniformly continuous** on
$X$ if: for every $\varepsilon > 0$, there exists $\delta > 0$
such that $d_Y(f(x), f(y)) < \varepsilon$ whenever
$d_X(x, y) < \delta$. The difference from ordinary continuity
is that $\delta$ does not depend on the base point. Uniform
continuity is a global property; ordinary continuity is
pointwise.

Example of the distinction: $f(x) = x^2$ on $\mathbb{R}$ is
continuous but not uniformly continuous (as $x$ grows,
$\delta$ must shrink to control $|x^2 - y^2|$ for fixed
$\varepsilon$). On any bounded interval it is uniformly
continuous.

### Proof: continuous functions on compact sets are uniformly continuous

**Theorem.** Let $(X, d_X)$ and $(Y, d_Y)$ be metric spaces,
let $K \subseteq X$ be compact, and let $f: K \to Y$ be
continuous. Then $f$ is uniformly continuous on $K$.

**Proof.** Suppose for contradiction that $f$ is not uniformly
continuous. Then there exists $\varepsilon > 0$ such that for
every $n \in \mathbb{N}$, there exist $x_n, y_n \in K$ with
$d_X(x_n, y_n) < 1/n$ but
$d_Y(f(x_n), f(y_n)) \geq \varepsilon$.

By sequential compactness of $K$, the sequence $(x_n)$ has a
subsequence $x_{n_k} \to x \in K$. Since
$d_X(x_{n_k}, y_{n_k}) < 1/n_k \to 0$, we have $y_{n_k} \to x$
as well. By continuity of $f$ at $x$, both $f(x_{n_k}) \to
f(x)$ and $f(y_{n_k}) \to f(x)$, so $d_Y(f(x_{n_k}),
f(y_{n_k})) \to 0$, contradicting
$d_Y(f(x_{n_k}), f(y_{n_k})) \geq \varepsilon$. $\blacksquare$

### Proof: extreme value theorem

**Theorem.** Let $K$ be a nonempty compact metric space and
$f: K \to \mathbb{R}$ continuous. Then $f$ attains its
supremum and infimum on $K$.

**Proof.** We prove attainment of the supremum; the infimum
case is symmetric. Let $M = \sup_{x \in K} f(x)$, allowing
the possibility $M = +\infty$.

By the definition of supremum, there exists a sequence
$(x_n)$ in $K$ with $f(x_n) \to M$. By sequential
compactness, some subsequence $x_{n_k}$ converges to a point
$x^* \in K$. By continuity of $f$, $f(x_{n_k}) \to f(x^*)$.
Since the full sequence $f(x_n) \to M$, the subsequence
$f(x_{n_k}) \to M$ as well, so $f(x^*) = M$. In particular,
$M$ is finite and is attained at $x^*$. $\blacksquare$

**This is the theorem that makes optimization possible.**
Every "the problem has a solution" argument in static
optimization traces back to this. The hypotheses matter:
compactness (bounded and closed in $\mathbb{R}^n$), continuity
of the objective. Drop either and the existence of a
maximizer is not guaranteed. Much of what optimization theory
does is relax the continuity assumption (to upper
semi-continuity) and the compactness assumption (to
coercivity and some closedness) while preserving the
existence conclusion. The extreme value theorem is the
prototype and the intuition everything else refines.

### Proof: intermediate value theorem

**Theorem.** Let $f: [a, b] \to \mathbb{R}$ be continuous and
suppose $f(a) < c < f(b)$. Then there exists $x \in (a, b)$
with $f(x) = c$.

**Proof.** Let $S = \{x \in [a, b] : f(x) < c\}$. Then
$a \in S$ (since $f(a) < c$), so $S$ is nonempty, and $S$ is
bounded above by $b$. Let $x^* = \sup S$; since $[a, b]$ is
closed, $x^* \in [a, b]$.

We claim $f(x^*) = c$. Suppose $f(x^*) < c$. By continuity,
there exists $\delta > 0$ such that $f(x) < c$ for all
$x \in [x^*, x^* + \delta) \cap [a, b]$. But then $x^* + \delta/2
\in S$ (assuming $x^* + \delta/2 \leq b$), contradicting
$x^* = \sup S$. If $x^* = b$, the same argument shows
$f(b) < c$, contradicting the hypothesis.

Suppose instead $f(x^*) > c$. By continuity, there exists
$\delta > 0$ such that $f(x) > c$ for all
$x \in (x^* - \delta, x^*]$. Then no point in this interval
is in $S$, so $\sup S \leq x^* - \delta < x^*$, contradiction.

Therefore $f(x^*) = c$. Since $f(a) < c = f(x^*)$, $x^* > a$;
since $f(x^*) = c < f(b)$, $x^* < b$. $\blacksquare$

---

## 4. Differentiation

Let $f: U \to \mathbb{R}$ where $U \subseteq \mathbb{R}$ is
open. The derivative of $f$ at $x_0 \in U$ is

$$
f'(x_0) = \lim_{h \to 0} \frac{f(x_0 + h) - f(x_0)}{h}
$$

when the limit exists. In higher dimensions, $f: U \to
\mathbb{R}^m$ with $U \subseteq \mathbb{R}^n$ is
**differentiable at $x_0$** if there exists a linear map
$A: \mathbb{R}^n \to \mathbb{R}^m$ such that

$$
\lim_{h \to 0} \frac{\|f(x_0 + h) - f(x_0) - Ah\|}{\|h\|} = 0.
$$

When it exists, $A$ is unique and is called the
**derivative** or **Jacobian** of $f$ at $x_0$, denoted
$Df(x_0)$ or $f'(x_0)$ depending on context. The idea to
hold: the derivative is the best linear approximation to $f$
near $x_0$. Everything about differentiation is about
reducing nonlinear problems to linear ones locally.

**Partial derivatives** versus **differentiability.** If $f$
is differentiable at $x_0$, all partial derivatives exist and
the Jacobian matrix is
$[Df(x_0)]_{ij} = \partial f_i / \partial x_j$. The converse
fails: a function can have all partial derivatives at a
point without being differentiable there. However, if the
partial derivatives exist and are continuous in a
neighborhood of $x_0$, then $f$ is differentiable at $x_0$.
The usual setting in econ is "continuously differentiable,"
i.e. $C^1$, which gives differentiability and lets us
compute with partials.

### Proof: mean value theorem

**Theorem.** Let $f: [a, b] \to \mathbb{R}$ be continuous on
$[a, b]$ and differentiable on $(a, b)$. Then there exists
$c \in (a, b)$ such that

$$
f'(c) = \frac{f(b) - f(a)}{b - a}.
$$

**Proof.** Define $g: [a, b] \to \mathbb{R}$ by

$$
g(x) = f(x) - \frac{f(b) - f(a)}{b - a} (x - a).
$$

Then $g$ is continuous on $[a, b]$, differentiable on
$(a, b)$, and $g(a) = g(b) = f(a)$. By the extreme value
theorem, $g$ attains a maximum and a minimum on $[a, b]$. If
both are attained at the endpoints, then $g$ is constant
(since $g(a) = g(b)$), so $g'(x) = 0$ everywhere in
$(a, b)$, and we are done. Otherwise, at least one extremum
is attained at some interior point $c \in (a, b)$, where
$g'(c) = 0$ by the interior extremum criterion (Fermat's
theorem, an elementary consequence of the definition of
derivative).

Computing: $g'(x) = f'(x) - (f(b) - f(a))/(b - a)$. Setting
$g'(c) = 0$ gives the conclusion. $\blacksquare$

The mean value theorem is the bridge between "local"
information (the derivative) and "global" information
(function values at different points). Every "if the
derivative is bounded, then the function is Lipschitz"
argument uses it. It is also the engine behind Taylor's
theorem.

### Proof: Taylor's theorem with Lagrange remainder

**Theorem.** Let $f: [a, b] \to \mathbb{R}$ be $n$ times
differentiable on $[a, b]$ and suppose $f^{(n)}$ is
continuous on $[a, b]$ and $f^{(n+1)}$ exists on $(a, b)$.
Fix $x_0 \in [a, b]$. Then for every $x \in [a, b]$ there
exists $\xi$ strictly between $x_0$ and $x$ such that

$$
f(x) = \sum_{k=0}^{n} \frac{f^{(k)}(x_0)}{k!} (x - x_0)^k
     + \frac{f^{(n+1)}(\xi)}{(n+1)!} (x - x_0)^{n+1}.
$$

**Proof.** Fix $x \neq x_0$ (the case $x = x_0$ is trivial).
Define

$$
P(t) = \sum_{k=0}^{n} \frac{f^{(k)}(t)}{k!} (x - t)^k
$$

and let $M$ be the unique constant such that

$$
f(x) = P(x_0) + M (x - x_0)^{n+1}.
$$

(The equation determines $M$ because $x \neq x_0$.)

Define $g(t) = f(x) - P(t) - M (x - t)^{n+1}$. Then $g(x) =
f(x) - P(x) - 0 = 0$, and by choice of $M$, $g(x_0) = 0$. By
the mean value theorem (in its Rolle's theorem form — the
case $f(a) = f(b)$), there exists $\xi$ strictly between
$x_0$ and $x$ with $g'(\xi) = 0$.

Compute $g'(t)$ term by term. The derivative of $P(t)$
telescopes: writing out $P(t)$ and differentiating, all
terms cancel except the last, giving
$P'(t) = f^{(n+1)}(t) (x - t)^n / n!$. The derivative of
$M(x - t)^{n+1}$ is $-M(n+1)(x - t)^n$. So

$$
g'(t) = -\frac{f^{(n+1)}(t)}{n!}(x - t)^n + M(n+1)(x - t)^n.
$$

Setting $g'(\xi) = 0$ and noting $x - \xi \neq 0$, we solve
for $M$:

$$
M = \frac{f^{(n+1)}(\xi)}{(n+1)!}.
$$

Substituting back into $f(x) = P(x_0) + M(x - x_0)^{n+1}$
gives the theorem. $\blacksquare$

**Why this proof matters.** Economists use Taylor
expansions constantly, and the most common error is to drop
the remainder term without checking whether the error is
actually negligible. The proof shows the remainder is a
genuine term, bounded by the size of $f^{(n+1)}$ in a
neighborhood, and should not be dropped on faith. "First-
order Taylor approximation" is shorthand for "the remainder
is $o(x - x_0)$ as $x \to x_0$" — a statement about what
happens *near* $x_0$. When economists use Taylor expansions
far from the expansion point, the remainder can dominate
and the approximation is useless. Having the proof on hand
makes the limits explicit.

---

## 5. The inverse function theorem and the implicit function theorem

These two results are the bridge between differentiation and
comparative statics. The implicit function theorem is, in
most applied settings, the more important: it is the result
that justifies differentiating through an equilibrium
condition to get comparative statics, and it is the engine
behind most "small change in a parameter produces a small
change in the equilibrium" arguments in economic theory.

### Inverse function theorem

**Theorem.** Let $U \subseteq \mathbb{R}^n$ be open and
$f: U \to \mathbb{R}^n$ be $C^1$. Suppose $x_0 \in U$ and
$Df(x_0)$ is invertible. Then there exist open neighborhoods
$V$ of $x_0$ and $W$ of $f(x_0)$ such that $f: V \to W$ is a
bijection, its inverse $f^{-1}: W \to V$ is $C^1$, and

$$
D(f^{-1})(f(x)) = [Df(x)]^{-1} \quad \text{for } x \in V.
$$

**Proof sketch.** The heart of the proof is the **contraction
mapping theorem** (proved in §6 below), which is why I am
presenting the inverse function theorem in this order:
contraction mapping first, then IFT and implicit function as
consequences. Here is the structure.

Without loss of generality, take $x_0 = 0$ and $f(0) = 0$,
and by composing with $[Df(0)]^{-1}$, take $Df(0) = I$.
Define $g(x) = x - f(x)$. Then $Dg(0) = 0$, so by continuity
of $Dg$, there is a neighborhood of $0$ on which
$\|Dg(x)\| \leq 1/2$. This makes $g$ a contraction (by the
mean value inequality — which generalizes the mean value
theorem to vector-valued functions).

Now: for each $y$ near $0$, the equation $f(x) = y$ is
equivalent to $x = y + g(x)$, i.e., $x$ is a fixed point of
the map $x \mapsto y + g(x)$. Since $g$ is a contraction with
contraction constant $\leq 1/2$, the map $x \mapsto y + g(x)$
is also a contraction (by the same constant). The contraction
mapping theorem gives a unique fixed point, which is the
unique $x$ near $0$ with $f(x) = y$. This defines $f^{-1}$.

The regularity of $f^{-1}$ (continuity, then differentiability,
then $C^1$) is extracted by standard arguments from the
contraction structure; I omit the details. The derivative
formula $D(f^{-1})(f(x)) = [Df(x)]^{-1}$ is the chain rule
applied to $f^{-1}(f(x)) = x$. $\blacksquare$

### Implicit function theorem

This is the one that matters most for comparative statics.

**Theorem.** Let $U \subseteq \mathbb{R}^n \times \mathbb{R}^m$
be open and $F: U \to \mathbb{R}^m$ be $C^1$. Write points of
$U$ as $(x, y)$ with $x \in \mathbb{R}^n$ and $y \in
\mathbb{R}^m$. Suppose $(x_0, y_0) \in U$ satisfies:

1. $F(x_0, y_0) = 0$.
2. $D_y F(x_0, y_0)$, the $m \times m$ matrix of partial
   derivatives with respect to the $y$ variables, is
   invertible.

Then there exist open neighborhoods $V$ of $x_0$ in
$\mathbb{R}^n$ and $W$ of $y_0$ in $\mathbb{R}^m$, and a
unique $C^1$ function $\varphi: V \to W$ such that
$\varphi(x_0) = y_0$ and

$$
F(x, \varphi(x)) = 0 \quad \text{for all } x \in V.
$$

Moreover,

$$
D\varphi(x) = -[D_y F(x, \varphi(x))]^{-1} D_x F(x, \varphi(x)).
$$

**Proof.** Define $G: U \to \mathbb{R}^n \times \mathbb{R}^m$
by $G(x, y) = (x, F(x, y))$. Then $G$ is $C^1$ and

$$
DG(x_0, y_0) = \begin{pmatrix} I_n & 0 \\ D_x F(x_0, y_0) & D_y F(x_0, y_0) \end{pmatrix}.
$$

This block-triangular matrix is invertible iff its diagonal
blocks are, i.e., iff $D_y F(x_0, y_0)$ is invertible. By
hypothesis it is, so by the inverse function theorem, $G$ has
a local $C^1$ inverse $G^{-1}$ near $(x_0, y_0)$.

Write $G^{-1}(x, z) = (x, H(x, z))$ for some $C^1$ function
$H$. Then $G(G^{-1}(x, z)) = (x, z)$ expands to
$(x, F(x, H(x, z))) = (x, z)$, so $F(x, H(x, z)) = z$.
Setting $z = 0$ gives $F(x, H(x, 0)) = 0$. Define
$\varphi(x) = H(x, 0)$; this is the function we want.

Uniqueness and the value at $x_0$ follow from the inverse
function theorem. The derivative formula is the chain rule
applied to $F(x, \varphi(x)) = 0$: differentiating with
respect to $x$,

$$
D_x F(x, \varphi(x)) + D_y F(x, \varphi(x)) D\varphi(x) = 0,
$$

and solving for $D\varphi(x)$. $\blacksquare$

**Why this proof is worth having.** The implicit function
theorem in economics is most often presented as a formula:
"$dy/dx = -F_x / F_y$." This is correct but misleading,
because it hides the hypotheses (in particular, the
invertibility of $D_y F$) and the locality of the conclusion
(only in a neighborhood of $(x_0, y_0)$). Having the proof on
hand reminds Markus that:

1. Comparative statics works only locally around an
   equilibrium. Large parameter changes can push the system
   past points where $D_y F$ is singular, and the local
   formula cannot be integrated naively.
2. The invertibility condition on $D_y F$ is nontrivial. In
   an economic model, it typically corresponds to "the
   equilibrium is locally unique" or "the system of
   first-order conditions determines $y$ uniquely given $x$."
   Cases where this fails — degenerate equilibria, bifurcation
   points — are exactly where comparative statics can behave
   strangely.
3. The derivative formula can be computed in coordinates,
   but behind the formula is the existence argument that
   guarantees $\varphi$ is well-defined. The formula without
   the existence argument is unsound.

This is the payoff of doing the math layer rigorously:
Markus does not just use the IFT formula; he knows what it
assumes and when it fails.

---

## 6. The contraction mapping theorem

The single most important theorem in this file for macro
work. It is the reason dynamic programming works, the reason
the inverse and implicit function theorems have proofs that
go through, and one of the standard tools for proving
existence of equilibria and fixed points.

Let $(X, d)$ be a metric space. A map $T: X \to X$ is a
**contraction** if there exists a constant $c \in [0, 1)$
such that

$$
d(T(x), T(y)) \leq c \cdot d(x, y) \quad \text{for all } x, y \in X.
$$

The constant $c$ is called the **contraction constant** or
**Lipschitz constant**. The crucial requirement is $c < 1$
strictly; $c = 1$ is not enough (such maps are called
*non-expansive* and do not in general have fixed points).

### Proof: contraction mapping theorem (Banach fixed-point theorem)

**Theorem.** Let $(X, d)$ be a nonempty complete metric space
and $T: X \to X$ a contraction with constant $c < 1$. Then:

1. $T$ has a unique fixed point $x^* \in X$, i.e., $T(x^*) = x^*$.
2. For any starting point $x_0 \in X$, the sequence defined
   by $x_{n+1} = T(x_n)$ converges to $x^*$.
3. The rate of convergence is geometric:
   $d(x_n, x^*) \leq \frac{c^n}{1 - c} d(x_1, x_0)$.

**Proof.** Pick any $x_0 \in X$ and define $x_{n+1} = T(x_n)$.

**Step 1: the sequence is Cauchy.** By induction on $n$,
$d(x_{n+1}, x_n) \leq c^n d(x_1, x_0)$: the base case $n = 0$
is immediate, and $d(x_{n+1}, x_n) = d(T(x_n), T(x_{n-1}))
\leq c \cdot d(x_n, x_{n-1}) \leq c \cdot c^{n-1}
d(x_1, x_0) = c^n d(x_1, x_0)$.

For $m > n$, by the triangle inequality,

$$
d(x_m, x_n) \leq \sum_{k=n}^{m-1} d(x_{k+1}, x_k)
           \leq d(x_1, x_0) \sum_{k=n}^{m-1} c^k
           \leq d(x_1, x_0) \frac{c^n}{1 - c}.
$$

Since $c < 1$, $c^n \to 0$, so $d(x_m, x_n) \to 0$ as
$n, m \to \infty$. The sequence is Cauchy.

**Step 2: the limit exists.** Since $X$ is complete, the
Cauchy sequence $(x_n)$ converges to some $x^* \in X$.

**Step 3: the limit is a fixed point.** $T$ is a
contraction and hence continuous (take $\delta = \varepsilon$
in the ε-δ definition). So $T(x_n) \to T(x^*)$, but
$T(x_n) = x_{n+1}$, and $x_{n+1} \to x^*$, so $T(x^*) = x^*$.

**Step 4: uniqueness.** Suppose $x^*$ and $y^*$ are both
fixed points. Then

$$
d(x^*, y^*) = d(T(x^*), T(y^*)) \leq c \cdot d(x^*, y^*).
$$

Since $c < 1$, this forces $d(x^*, y^*) = 0$, so
$x^* = y^*$.

**Step 5: rate of convergence.** Taking $m \to \infty$ in
the inequality from Step 1,

$$
d(x^*, x_n) = \lim_{m \to \infty} d(x_m, x_n)
            \leq \frac{c^n}{1 - c} d(x_1, x_0).
$$

$\blacksquare$

**Why this theorem is central to macro.** In dynamic
programming, the **Bellman operator** on the space of
bounded continuous value functions is a contraction under
the sup norm, with contraction constant equal to the
discount factor $\beta < 1$. The space of bounded continuous
functions on a compact state space is complete under the sup
norm. So the contraction mapping theorem applies and
guarantees:

1. **The Bellman equation has a unique solution** (the value
   function exists and is unique).
2. **Value function iteration converges** from any starting
   guess — this is the standard numerical procedure for
   solving dynamic programming problems.
3. **The rate of convergence is geometric**, with rate $\beta$.

Everything in dynamic programming — the existence of optimal
policies, the characterization of the value function, the
validity of value function iteration as a solution method —
traces back to this theorem. A graduate macro class that
does not prove the contraction mapping theorem is hiding the
foundation of the material it teaches. Markus does not hide
it.

The theorem also shows up in:

- **Existence of Walrasian equilibrium** in some settings
  (though Kakutani and Brouwer fixed-point theorems are
  more common tools there).
- **Existence and uniqueness of solutions to ODEs**
  (Picard-Lindelöf theorem), via the contraction structure
  of the Picard iteration.
- **Proofs of the inverse and implicit function theorems**
  (as sketched in §5).
- **Numerical methods for fixed-point problems** (the
  iteration $x_{n+1} = T(x_n)$ is the basic template).

---

## 7. Uniform convergence

Sequences of functions can converge in several distinct ways,
and the distinctions matter for whether limits preserve the
properties of the functions in the sequence.

Let $f_n: X \to \mathbb{R}$ (or to any metric space) be a
sequence of functions and $f: X \to \mathbb{R}$ a candidate
limit.

**Pointwise convergence.** $f_n \to f$ pointwise on $X$ if
for every $x \in X$, $f_n(x) \to f(x)$ in $\mathbb{R}$. This
is the weak notion: each point has its own rate, and the
rates can differ arbitrarily across points.

**Uniform convergence.** $f_n \to f$ uniformly on $X$ if

$$
\sup_{x \in X} |f_n(x) - f(x)| \to 0 \quad \text{as } n \to \infty.
$$

Equivalently: for every $\varepsilon > 0$, there exists $N$
such that $|f_n(x) - f(x)| < \varepsilon$ for all $n \geq N$
*and* all $x \in X$. The quantifier on $x$ is inside the
existence of $N$, so $N$ is allowed to depend on
$\varepsilon$ but not on $x$.

Uniform convergence implies pointwise convergence; the
converse fails dramatically.

**Why uniform convergence matters: preservation of
properties.** The central facts are:

1. **Uniform limits of continuous functions are continuous.**
   If each $f_n$ is continuous at $x_0$ and $f_n \to f$
   uniformly on a neighborhood of $x_0$, then $f$ is
   continuous at $x_0$.
2. **Uniform convergence interchanges with limits.** If
   $f_n \to f$ uniformly on $X$ and $x_n \to x$ in $X$ with
   each $f_n$ continuous at $x$, then
   $f_n(x_n) \to f(x)$.
3. **Uniform convergence interchanges with Riemann
   integration** on compact intervals (the analogous result
   for Lebesgue integration — the dominated convergence
   theorem — lives in `measure-theory.md`).

The pointwise versions of all three fail. Standard
counterexample: $f_n(x) = x^n$ on $[0, 1]$. These are
continuous, converge pointwise to $f(x) = 0$ for $x \in
[0,1)$ and $f(1) = 1$, which is discontinuous. The pointwise
limit does not preserve continuity because the convergence is
not uniform on $[0, 1]$.

### Proof: uniform limit of continuous functions is continuous

**Theorem.** Let $(X, d)$ be a metric space, $f_n: X \to
\mathbb{R}$ a sequence of continuous functions, and suppose
$f_n \to f$ uniformly on $X$. Then $f$ is continuous on $X$.

**Proof.** Fix $x_0 \in X$ and $\varepsilon > 0$. By uniform
convergence, there exists $N$ such that
$|f_N(x) - f(x)| < \varepsilon/3$ for all $x \in X$.

By continuity of $f_N$ at $x_0$, there exists $\delta > 0$
such that $|f_N(x) - f_N(x_0)| < \varepsilon/3$ whenever
$d(x, x_0) < \delta$.

Combine: for $x$ with $d(x, x_0) < \delta$,

$$
|f(x) - f(x_0)| \leq |f(x) - f_N(x)| + |f_N(x) - f_N(x_0)|
                + |f_N(x_0) - f(x_0)|
                < \frac{\varepsilon}{3} \cdot 3 = \varepsilon.
$$

So $f$ is continuous at $x_0$. $\blacksquare$

This is the "$\varepsilon/3$ argument," one of the standard
proof techniques in analysis. Worth seeing for the technique
alone.

**Relevance to econ work.** Uniform convergence shows up in:

- **Empirical process theory.** The uniform law of large
  numbers is the statement that an empirical distribution
  converges to the true distribution uniformly over a class
  of events, and it is what justifies using sample averages
  to estimate population expectations. The formal treatment
  uses functional-analytic tools that belong in
  `functional-analysis.md`.
- **M-estimation.** The consistency of M-estimators
  (maximum likelihood, GMM, quantile regression, and so on)
  requires uniform convergence of the sample objective
  function to the population objective over the parameter
  space. This is where "compactness of the parameter space"
  and "continuity of the objective" assumptions come from
  in econometrics theorems.
- **Approximation theory.** Results about approximating
  functions by polynomials (Stone-Weierstrass) or by
  simpler function classes are statements about uniform
  convergence on compact sets.

The deeper treatment of convergence modes — in particular,
almost sure convergence, convergence in probability, and
convergence in distribution, which are the convergence modes
that probability and econometrics actually use — requires
measure theory and belongs in `measure-theory.md`. The point
here is just that "uniform" and "pointwise" are the starting
distinction, and that uniform is the mode that preserves
continuity and integrability.

---

## 8. What this file doesn't cover, and where to find it

**Riemann integration.** Covered briefly in
`measure-theory.md` as the intuition pump that Lebesgue
integration generalizes. The Riemann-to-Lebesgue transition
is easier to teach when both are in the same file, so
Riemann lives there rather than here.

**Lebesgue integration and measure theory.** All of it —
σ-algebras, measurable functions, construction of the
Lebesgue integral, dominated and monotone convergence,
Fubini-Tonelli, modes of convergence (a.s., in probability,
in $L^p$) — belongs in `measure-theory.md`. That file builds
on this one.

**Banach and Hilbert spaces, operator theory.** All
functional analysis goes in `functional-analysis.md`. That
file builds on both this one and measure theory. Specific
topics waiting there: $L^p$ spaces, dual spaces, bounded
linear operators, the Hahn-Banach theorem, the open mapping
and closed graph theorems, weak and weak-* convergence,
compact operators, the spectral theorem for self-adjoint
operators, reproducing kernel Hilbert spaces (for
nonparametric econometrics).

**Ordinary differential equations beyond the Picard
theorem.** The Picard-Lindelöf existence-and-uniqueness
theorem is a contraction mapping application mentioned in
§6. Deeper ODE theory — stability, bifurcation, phase
portraits — lives in its own file if one gets built, probably
under a `dynamical-systems.md` heading.

**Real analysis on $\mathbb{R}^n$ beyond compactness and
differentiation.** Multivariate integration, the change of
variables formula, Stokes' theorem, differential forms —
none of this is in the standard working tool kit of applied
econ, and it does not get its own coverage here. If a
structural project actually needs Stokes' theorem, the
reference to consult is Spivak's *Calculus on Manifolds*, not
this file.

**Complex analysis.** Entirely out of scope for the math
layer as currently planned. Complex analysis shows up in
spectral methods in time series (characteristic functions,
the Fourier inversion theorem for distributions) and in
some macroeconomic modeling contexts (solving linear
rational expectations models via the Blanchard-Kahn
conditions), but those specific applications can be covered
in-place in the files where they are used rather than
requiring a general complex analysis note.

These exclusions are deliberate. The math layer is not a
replacement for a mathematics education; it is a reference
that covers the mathematical objects Markus actually reaches
for in econ work, organized to be usable rather than
comprehensive.

---

## Cross-references

- `00_Identity/Principles.md` — the section on uncertainty
  and honesty, and the commitment that "every identification
  strategy makes assumptions you cannot test." The parallel
  in math is that every proof makes hypotheses that can fail,
  and Markus's job is to know what the hypotheses are and
  when they bind.
- `20_Math/measure-theory.md` — The direct sequel: Lebesgue
  integration, the three convergence theorems (monotone,
  Fatou, dominated), Fubini-Tonelli, modes of convergence,
  measure-theoretic foundations of probability.
- `20_Math/functional-analysis.md` — The further sequel: Banach
  and Hilbert spaces, the big four theorems, $L^p$ duality,
  the Radon-Nikodym theorem proved via Riesz representation,
  compact operators and the spectral theorem, and conditional
  expectation as $L^2$ projection.
- `20_Math/optimization.md` — The applied counterpart: convexity,
  KKT conditions, duality, the envelope theorem, econometric
  applications (OLS, MLE, GMM), and numerical methods. Leans
  on this file for the extreme value theorem, compactness,
  continuity, the implicit function theorem, and the
  contraction mapping theorem (nested fixed-point problems).
- `20_Math/dynamic-programming.md` — The
  contraction mapping theorem of §6 is the foundation.
- `10_Methods/modern-toolkit-references.md` — the
  econometrics directory. Many results in the econometric
  theory underlying the modern toolkit (consistency and
  asymptotic normality of estimators, uniform laws of large
  numbers, empirical process bounds) rest on the machinery
  introduced here and extended in `measure-theory.md` and
  `functional-analysis.md`.
- `50_Workflows/run-a-regression-properly.md` — Stage 4 on
  identification uses the language of "assumptions" in a way
  that parallels the role of hypotheses in mathematical
  theorems. Different domain, same discipline.

---

*Last updated: 2026-04. The contents of this file are stable
— real analysis as a field has not moved in decades, and
the selection here reflects what Markus reaches for in econ
work. The file will be revised if measure theory or
functional analysis turn out to require results not covered
here, and if subsequent notes in the math layer reveal gaps
in the foundation.*