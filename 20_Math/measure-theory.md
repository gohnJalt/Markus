---
tags: [math, measure-theory, foundations, probability]
related: [real-analysis, functional-analysis, optimization, probability, stochastic-processes, modern-toolkit-references]
---

# Measure Theory

This file is the direct sequel to `real-analysis.md`. It picks
up where that file stops: at the boundary where the Riemann
integral runs out. The central move is to replace the
partition-based definition of integration with a
measure-theoretic one — to integrate not by subdividing the
domain but by assigning weights to sets determined by the
function's values. The resulting Lebesgue integral handles
pointwise limits correctly, and that is the property
everything else in probability and asymptotic econometrics
needs.

The file covers: Riemann integration and where it breaks,
σ-algebras and measures, Lebesgue measure on $\mathbb{R}$,
measurable functions and simple functions, the Lebesgue
integral, the three convergence theorems (monotone, Fatou,
dominated), product measures and Fubini-Tonelli, modes of
convergence (almost sure, in probability, in $L^p$, in
distribution) and the relationships between them, and the
measure-theoretic foundations of probability.

It stops before $L^p$ spaces treated as Banach and Hilbert
spaces — that goes in `functional-analysis.md`. It stops
before the full treatment of conditional expectation as an
$L^2$ projection — also there. It stops before the CLT, LLN,
and other limit theorems — those go in `probability.md`. And
it stops before martingales and stochastic integration, which
belong in `stochastic-processes.md`.

This file is a prerequisite for all three of those. It is
also the technical foundation for asymptotic econometrics:
every consistency result lives in probability-space language,
and the convergence theorems here are the tools those proofs
turn on.

Proofs are included for the convergence theorems — monotone
convergence, Fatou, dominated convergence — because each
builds on the one before and because these are the theorems
that appear in econometric arguments. The approximation
theorem for simple functions is also proved because it is the
engine of the MCT proof. Fubini-Tonelli is stated without
full proof; the complete construction-based argument adds
length without adding to the working understanding. The
Radon-Nikodym theorem is stated with the proof deferred to
`functional-analysis.md`, where the Hilbert space machinery
it requires is developed.

For a first reading, Royden and Fitzpatrick's *Real Analysis*
or Folland's *Real Analysis* are the standard references.
Billingsley's *Probability and Measure* is the bridge from
the analyst's to the probabilist's perspective. This file
assumes one of them has been read; it serves as the working
reference afterward.

---

## 1. Riemann integration and where it breaks

The Riemann integral of a bounded function $f: [a, b] \to \mathbb{R}$
is defined through partitions. A partition
$P = \{a = x_0 < x_1 < \cdots < x_n = b\}$ produces an
upper Darboux sum
$U(f, P) = \sum_{i=1}^n \sup_{[x_{i-1}, x_i]} f \cdot (x_i - x_{i-1})$
and a lower sum $L(f, P)$ with $\inf$ in place of $\sup$.
The function is Riemann-integrable if $\inf_P U(f,P) =
\sup_P L(f,P)$, and the common value is the Riemann integral.

This works well for continuous functions on bounded intervals,
and the results from `real-analysis.md` — particularly §7
on uniform convergence — characterize exactly when the Riemann
integral and limits can be exchanged. But that exchange
requires **uniform convergence**: for every $\varepsilon > 0$,
a single $N$ works for all $x$ simultaneously. Sequences
that arise in probability and statistics rarely deliver this.

**The fundamental failure.** Enumerate the rationals in
$[0,1]$ as $q_1, q_2, \ldots$ and define

$$
f_n(x) = \begin{cases} 1 & \text{if } x \in \{q_1, \ldots, q_n\}, \\ 0 & \text{otherwise.} \end{cases}
$$

Each $f_n$ has finitely many discontinuities and is
Riemann-integrable with integral $0$. The pointwise limit is
the Dirichlet function $f = \mathbf{1}_{\mathbb{Q} \cap [0,1]}$:
equal to $1$ on rationals, $0$ on irrationals. This function
is nowhere Riemann-integrable: every subinterval contains
both rational and irrational points, so every upper sum is
$1$ and every lower sum is $0$.

The Riemann integral cannot handle this limit. Yet the
"correct" answer is clear: the rationals are countable, so
they form a negligible set, and the integral should be $0$.
The Lebesgue integral is $0$, consistent with the limit of
the integrals.

**The deeper issue.** Riemann integrability is a condition on
the domain: the function must not oscillate too badly over
any interval. Lebesgue integrability is a condition on the
range: the level sets $\{x : f(x) > c\}$ must be
measurable. The shift from domain-based to range-based
integration is what allows the Lebesgue theory to handle
pointwise limits. The dominated convergence theorem (§6)
replaces the uniform convergence requirement with something
weaker and more natural: a single integrable dominating
function.

---

## 2. σ-algebras and measures

**σ-algebras.** A **σ-algebra** on a set $X$ is a collection
$\mathcal{A}$ of subsets of $X$ satisfying:

1. $\emptyset \in \mathcal{A}$.
2. If $A \in \mathcal{A}$, then $A^c \in \mathcal{A}$ (closed
   under complements).
3. If $A_1, A_2, \ldots \in \mathcal{A}$, then
   $\bigcup_{n=1}^\infty A_n \in \mathcal{A}$ (closed under
   countable unions).

By De Morgan's laws, $\mathcal{A}$ is also closed under
countable intersections. The pair $(X, \mathcal{A})$ is a
**measurable space**; the sets in $\mathcal{A}$ are the
**measurable sets**.

The countability in condition (3) is the key. Why not allow
uncountable unions and work with all subsets of $X$? Because
no measure can be defined consistently on all subsets of
$\mathbb{R}$: a result due to Vitali shows that there is no
function on $\mathcal{P}(\mathbb{R})$ that is simultaneously
translation-invariant, assigns measure $1$ to $[0,1]$, and
is countably additive. The construction of the counterexample
uses the axiom of choice. The response is to restrict
attention to a σ-algebra of "nice" sets — large enough to
include everything needed in practice, small enough to
exclude the pathological sets.

**The Borel σ-algebra.** On any topological space $X$, the
**Borel σ-algebra** $\mathcal{B}(X)$ is the smallest σ-algebra
containing all open sets. On $\mathbb{R}$, it is generated
by the open intervals (or equivalently by the half-lines
$(-\infty, a)$ for $a \in \mathbb{R}$). Borel sets include
all open sets, all closed sets, all countable sets, and
essentially every set that arises naturally in analysis.

**Measures.** A **measure** on $(X, \mathcal{A})$ is a
function $\mu: \mathcal{A} \to [0, +\infty]$ satisfying:

1. $\mu(\emptyset) = 0$.
2. **Countable additivity**: if $A_1, A_2, \ldots \in
   \mathcal{A}$ are pairwise disjoint, then
   $$\mu\!\left(\bigcup_{n=1}^\infty A_n\right) = \sum_{n=1}^\infty \mu(A_n).$$

The triple $(X, \mathcal{A}, \mu)$ is a **measure space**. A
measure is **finite** if $\mu(X) < \infty$ and **σ-finite**
if $X = \bigcup_n A_n$ with $\mu(A_n) < \infty$ for each $n$.
σ-finiteness is the standard working assumption; most results
that fail for non-σ-finite measures are theorems (Fubini,
Radon-Nikodym) where uniqueness or existence would break.

A **probability measure** is a measure with $\mu(X) = 1$.

**Consequences of countable additivity.** These follow from
applying the definition to telescoping or nested sequences:

- *Continuity from below*: if $A_1 \subseteq A_2 \subseteq \cdots$
  are measurable, then $\mu(\bigcup_n A_n) = \lim_n \mu(A_n)$.
- *Continuity from above*: if $A_1 \supseteq A_2 \supseteq
  \cdots$ are measurable and $\mu(A_1) < \infty$, then
  $\mu(\bigcap_n A_n) = \lim_n \mu(A_n)$.
- *Monotonicity*: $A \subseteq B \Rightarrow \mu(A) \leq \mu(B)$.
- *Subadditivity*: $\mu(A \cup B) \leq \mu(A) + \mu(B)$.

Continuity from above requires $\mu(A_1) < \infty$; without
it the result can fail ($A_n = [n, \infty)$ on $\mathbb{R}$
with Lebesgue measure: $\mu(A_n) = +\infty$ for all $n$ but
$\bigcap A_n = \emptyset$). This is not a minor technical
point — it is the reason moment conditions in econometric
theorems specify finite second (or higher) moments.

**Null sets and almost everywhere.** A set $N \in \mathcal{A}$
with $\mu(N) = 0$ is a **null set**. A property holds
**$\mu$-almost everywhere** ($\mu$-a.e.) if the set where it
fails is a null set. In probability contexts, "a.e." is
written **almost surely** (a.s.). A measure is **complete**
if every subset of every null set is measurable (and hence
a null set). Completeness is a convenience assumption that
ensures "a.e.-equal functions are equally measurable."

---

## 3. Lebesgue measure on $\mathbb{R}$

The **Lebesgue measure** $\lambda$ on $\mathbb{R}$ is the
unique complete measure on an appropriate σ-algebra
(the **Lebesgue σ-algebra** $\mathcal{L}(\mathbb{R})$,
which contains $\mathcal{B}(\mathbb{R})$ and is complete)
satisfying $\lambda([a, b]) = b - a$ for all $a \leq b$.

**Construction (outer measure).** For any $E \subseteq
\mathbb{R}$, define the **outer measure**

$$
\lambda^*(E) = \inf\!\left\{\,\sum_{n=1}^\infty (b_n - a_n) :
E \subseteq \bigcup_{n=1}^\infty (a_n, b_n)\right\}.
$$

This assigns to each set the infimum of the total length of
all countable open-interval covers. It is defined for every
subset of $\mathbb{R}$, is monotone, and is countably
subadditive, but not in general countably additive on all
subsets.

**Carathéodory measurability.** Call a set $E$ *Lebesgue-
measurable* if for every $A \subseteq \mathbb{R}$,

$$
\lambda^*(A) = \lambda^*(A \cap E) + \lambda^*(A \cap E^c).
$$

The Lebesgue-measurable sets form a σ-algebra. Restricting
$\lambda^*$ to this σ-algebra gives a complete measure:
Lebesgue measure $\lambda$. Every Borel set is
Lebesgue-measurable.

**Key properties.**

- Translation-invariance: $\lambda(E + t) = \lambda(E)$ for
  all measurable $E$ and $t \in \mathbb{R}$.
- Every countable set has measure zero. In particular,
  $\lambda(\mathbb{Q}) = 0$.
- The **Cantor set** is an uncountable, nowhere-dense, closed
  subset of $[0,1]$ with $\lambda = 0$. It refutes the
  intuition that "measure zero $\Rightarrow$ topologically
  small." The correct intuition is "measure zero $\Rightarrow$
  negligible for integration purposes."

Lebesgue measure on $\mathbb{R}^n$ is the product of $n$
copies of Lebesgue measure on $\mathbb{R}$ (via the
product-measure construction of §7); it assigns to each box
$\prod_{i=1}^n [a_i, b_i]$ its volume $\prod_i (b_i - a_i)$.

---

## 4. Measurable functions and simple functions

**Measurable functions.** Let $(X, \mathcal{A})$ and
$(Y, \mathcal{B})$ be measurable spaces. A function
$f: X \to Y$ is **measurable** if $f^{-1}(B) \in \mathcal{A}$
for every $B \in \mathcal{B}$.

In the standard case $Y = \mathbb{R}$ with the Borel
σ-algebra, it suffices to check that $\{x : f(x) > c\} \in
\mathcal{A}$ for every $c \in \mathbb{R}$ — checking against
half-lines (a generating class) is enough. Every continuous
function is Borel-measurable; sums, products, and pointwise
limits of measurable functions are measurable. That last
fact — measurability is preserved under pointwise limits — is
false for Riemann-integrable functions (the Dirichlet
function is a pointwise limit of Riemann-integrable functions
but is not Riemann-integrable) and is precisely what makes
the Lebesgue theory suitable for limit arguments.

**Simple functions.** A **simple function** is a measurable
function with finite range. If the distinct values are
$a_1, \ldots, a_k$ and $A_i = \{x : f(x) = a_i\}$, then

$$
f = \sum_{i=1}^k a_i \mathbf{1}_{A_i}.
$$

Simple functions are the building blocks of the Lebesgue
integral, playing the role that step functions play in the
Riemann theory.

### Proof: approximation by simple functions

**Theorem.** Let $f: X \to [0, +\infty]$ be a nonneg
measurable function. Then there exists a sequence of nonneg
simple functions $\phi_1, \phi_2, \ldots$ with
$\phi_n \nearrow f$ pointwise (i.e., $\phi_n \leq \phi_{n+1}$
and $\phi_n(x) \to f(x)$ for every $x$).

**Proof.** For each $n \geq 1$, define

$$
\phi_n(x) = \begin{cases}
\dfrac{k-1}{2^n} & \text{if } \dfrac{k-1}{2^n} \leq f(x) < \dfrac{k}{2^n},
\quad k = 1, 2, \ldots, n \cdot 2^n, \\[6pt]
n & \text{if } f(x) \geq n.
\end{cases}
$$

Each $\phi_n$ is simple (finitely many values, each a
measurable preimage) and nonneg. The sequence is increasing:
$\phi_n(x) \leq \phi_{n+1}(x)$ for all $x$ (each bin at step
$n+1$ is a half-size subset of a bin at step $n$). Where
$f(x) < +\infty$, once $n > f(x)$ we have
$\phi_n(x) \geq f(x) - 2^{-n} \to f(x)$. Where $f(x) = +\infty$,
$\phi_n(x) = n \to +\infty = f(x)$. $\blacksquare$

This theorem is the engine of the monotone convergence proof.
It says: every nonneg measurable function is a limit of
functions whose integrals we already know how to compute.

---

## 5. The Lebesgue integral

The integral is built in three stages.

**Stage 1: nonneg simple functions.** For
$\phi = \sum_{i=1}^k a_i \mathbf{1}_{A_i}$ with $a_i \geq 0$
and pairwise-disjoint measurable $A_i$, define

$$
\int \phi \, d\mu = \sum_{i=1}^k a_i \, \mu(A_i),
$$

with the convention $0 \cdot \infty = 0$. This is
well-defined independently of the representation of $\phi$.

**Stage 2: nonneg measurable functions.** For
$f: X \to [0, +\infty]$ measurable, define

$$
\int f \, d\mu = \sup\!\left\{\int \phi \, d\mu :
0 \leq \phi \leq f,\; \phi \text{ simple}\right\}.
$$

This supremum is finite iff the function is integrable in a
reasonable sense. It may equal $+\infty$.

**Stage 3: general measurable functions.** Write
$f = f^+ - f^-$ where $f^+(x) = \max(f(x), 0)$ and
$f^-(x) = \max(-f(x), 0)$. Both are nonneg measurable.
Define

$$
\int f \, d\mu = \int f^+ \, d\mu - \int f^- \, d\mu
$$

when at least one side is finite. The function $f$ is
**integrable** (written $f \in L^1(\mu)$) if $\int |f| \, d\mu
< \infty$, equivalently if both $\int f^+ < \infty$ and
$\int f^- < \infty$.

**Properties.** For integrable $f, g$ and real $c$: linearity
($\int (cf + g) = c \int f + \int g$); monotonicity
($f \leq g$ a.e. $\Rightarrow \int f \leq \int g$); triangle
inequality ($|\int f \, d\mu| \leq \int |f| \, d\mu$);
null-set insensitivity ($f = g$ a.e. $\Rightarrow \int f =
\int g$, so $L^1(\mu)$ is properly defined on a.e.-equivalence
classes).

**Comparison with Riemann.** If $f: [a, b] \to \mathbb{R}$ is
Riemann-integrable, then it is Lebesgue-integrable and the
two integrals agree. The Lebesgue integral is strictly more
general: the Dirichlet function is Lebesgue-integrable with
integral $0$ but not Riemann-integrable.

---

## 6. The convergence theorems

Three theorems, in increasing order of power, answer the
question: when can we pass a limit through the integral?
Together they constitute the main technical payoff of
building the Lebesgue theory.

### Proof: monotone convergence theorem

**Theorem (MCT).** Let $(f_n)$ be a sequence of nonneg
measurable functions with $f_n \nearrow f$ a.e. (the
sequence is pointwise nondecreasing almost everywhere, with
limit $f$). Then

$$
\lim_{n \to \infty} \int f_n \, d\mu = \int f \, d\mu.
$$

The value $+\infty$ is allowed on either side.

**Proof.** Since $f_n \leq f_{n+1} \leq f$ a.e., the
integrals $\int f_n$ form a nondecreasing sequence bounded
above by $\int f$. Let $L = \lim \int f_n \leq \int f$.

For the reverse inequality, let $\phi$ be any nonneg simple
function with $\phi \leq f$, and fix $c \in (0,1)$. Define

$$
E_n = \{x : f_n(x) \geq c\phi(x)\}.
$$

Each $E_n$ is measurable, $E_n \subseteq E_{n+1}$ (since
$f_n$ is nondecreasing), and $\bigcup_n E_n = X$ up to a
null set (since $f_n \nearrow f \geq \phi > c\phi$ a.e.
when $\phi > 0$, and both sides equal $0$ when $\phi = 0$).
By continuity of $\mu$ from below:

$$
\int f_n \, d\mu \geq \int_{E_n} f_n \, d\mu
\geq c \int_{E_n} \phi \, d\mu \to c \int \phi \, d\mu.
$$

The last step uses continuity from below applied to the
measure $B \mapsto \int_B \phi \, d\mu$ (which is a measure
by countable additivity of $\mu$). Thus $L \geq c \int \phi
\, d\mu$. Letting $c \to 1$ gives $L \geq \int \phi \,
d\mu$. Taking the sup over all simple $\phi \leq f$:

$$
L \geq \sup_{\phi \leq f} \int \phi \, d\mu = \int f \, d\mu.
$$

So $L = \int f \, d\mu$. $\blacksquare$

**Why this proof is the prototype.** The structure — trivial
upper bound, then sandwich using continuity of measure from
below — recurs throughout analysis and probability. The key
step is that continuity of measure is a consequence of
countable additivity, which is why countable additivity (not
merely finite additivity) is the right axiom. With finite
additivity you get the Riemann integral; with countable
additivity you get the Lebesgue integral and all the
convergence theorems.

**Corollary.** For nonneg measurable $f_n$: $\int \sum_n f_n
\, d\mu = \sum_n \int f_n \, d\mu$. Apply the MCT to the
partial sums.

### Proof: Fatou's lemma

**Lemma (Fatou).** Let $(f_n)$ be a sequence of nonneg
measurable functions. Then

$$
\int \liminf_{n \to \infty} f_n \, d\mu \leq
\liminf_{n \to \infty} \int f_n \, d\mu.
$$

**Proof.** Let $g_n = \inf_{k \geq n} f_k$. Then $g_n$ is
measurable (inf of measurable functions is measurable),
$g_n \leq f_n$, and $g_n \nearrow \liminf_n f_n$ pointwise.

Since $g_n \leq f_k$ for all $k \geq n$:
$\int g_n \leq \int f_k$ for all $k \geq n$, hence
$\int g_n \leq \inf_{k \geq n} \int f_k$.

By the MCT applied to $(g_n)$:

$$
\int \liminf_n f_n = \lim_n \int g_n \leq
\lim_n \inf_{k \geq n} \int f_k = \liminf_n \int f_n.
\quad \blacksquare
$$

**The content.** Fatou is "MCT with the monotonicity
hypothesis replaced by nonnegativity, at the cost of an
inequality in place of equality." The inequality can be
strict: let $f_n = \mathbf{1}_{[n, n+1]}$ on $\mathbb{R}$.
Then $\liminf f_n = 0$ everywhere (each $x$ is in only
finitely many $[n, n+1]$), so $\int \liminf f_n = 0$. But
$\int f_n = 1$ for every $n$, so $\liminf \int f_n = 1$.
Mass has escaped to infinity; the integral cannot see it.
This is the failure mode that the dominated convergence
theorem rules out.

### Proof: dominated convergence theorem

**Theorem (DCT).** Let $f_n \to f$ a.e., and suppose there
exists $g \in L^1(\mu)$ with $|f_n| \leq g$ a.e. for all $n$.
Then $f \in L^1(\mu)$ and

$$
\lim_{n \to \infty} \int |f_n - f| \, d\mu = 0,
$$

which implies $\lim_{n \to \infty} \int f_n \, d\mu =
\int f \, d\mu$.

**Proof.** Since $|f_n| \leq g$ a.e. and $f_n \to f$ a.e.,
we have $|f| \leq g$ a.e., so $f \in L^1$. Since
$|f_n - f| \leq |f_n| + |f| \leq 2g$ a.e., the sequence
$2g - |f_n - f|$ is nonneg a.e. Apply Fatou:

$$
\int \liminf_n (2g - |f_n - f|) \, d\mu \leq
\liminf_n \int (2g - |f_n - f|) \, d\mu.
$$

Left side: since $f_n \to f$ a.e., $|f_n - f| \to 0$ a.e.,
so $\liminf_n (2g - |f_n - f|) = 2g$ a.e., and the left
side equals $\int 2g \, d\mu$.

Right side: $\int 2g - \limsup_n \int |f_n - f|$ (using
$\int 2g < \infty$). So

$$
\int 2g \leq \int 2g - \limsup_n \int |f_n - f|,
$$

giving $\limsup_n \int |f_n - f| \leq 0$. Since integrals
of nonneg functions are nonneg, $\int |f_n - f| \to 0$. By
the triangle inequality, $|\int f_n - \int f| \leq \int |f_n
- f| \to 0$. $\blacksquare$

**Why the dominating function is load-bearing.** Without the
domination condition, the theorem fails: $f_n =
\mathbf{1}_{[n,n+1]}$ satisfies $f_n \to 0$ pointwise but
$\int f_n = 1$ for all $n$. Any dominating function would
need $g(x) \geq 1$ for all $x \geq 1$, giving $\int g =
+\infty$. The condition $g \in L^1$ rules out mass escaping
to infinity.

In econometrics, the domination condition appears as "uniform
integrability" in more general limit theorems. The principle
is the same: convergence in probability alone is not enough
to pass the limit through an expectation; you need the tails
to be uniformly controlled. The "bounded $r$-th moments for
some $r > 2$" assumptions in central limit theorem
statements are sufficient conditions for the domination to
hold.

**What this theorem powers.** In a consistency proof for an
M-estimator $\hat{\theta}_n$, one needs to show that the
sample criterion function $Q_n(\theta) = n^{-1} \sum_i
q(X_i, \theta)$ converges uniformly to the population
criterion $Q(\theta) = E[q(X, \theta)]$. The DCT justifies
the first step: for each fixed $\theta$, the law of large
numbers gives $Q_n(\theta) \to Q(\theta)$ a.s., and the DCT
applied to the objective function (given the domination
condition) lets one pass the limit through the expectation.
Every "compact-parameter-space-plus-continuity-plus-bounded-
moments" assumption in econometric asymptotics is a package
designed to satisfy the DCT.

---

## 7. Product measures and Fubini-Tonelli

**Product σ-algebra and product measure.** Let
$(X, \mathcal{A}, \mu)$ and $(Y, \mathcal{B}, \nu)$ be
σ-finite measure spaces. The **product σ-algebra**
$\mathcal{A} \otimes \mathcal{B}$ is the smallest σ-algebra
on $X \times Y$ containing all rectangles $A \times B$ with
$A \in \mathcal{A}$, $B \in \mathcal{B}$. The **product
measure** $\mu \otimes \nu$ is the unique measure on
$\mathcal{A} \otimes \mathcal{B}$ satisfying

$$
(\mu \otimes \nu)(A \times B) = \mu(A) \cdot \nu(B)
$$

for all such rectangles. Existence and uniqueness follow from
the Carathéodory extension theorem (the same machinery as
Lebesgue measure on intervals), requiring σ-finiteness.

### Tonelli's theorem

**Theorem (Tonelli).** Let $f: X \times Y \to [0, +\infty]$
be $(\mathcal{A} \otimes \mathcal{B})$-measurable and nonneg.
Then for $\mu$-a.e. $x$, the function $y \mapsto f(x,y)$ is
$\mathcal{B}$-measurable; the function
$x \mapsto \int_Y f(x,y) \, d\nu(y)$ is $\mathcal{A}$-
measurable; and

$$
\int_{X \times Y} f \, d(\mu \otimes \nu)
= \int_X \!\left(\int_Y f(x,y) \, d\nu(y)\right) d\mu(x)
= \int_Y \!\left(\int_X f(x,y) \, d\mu(x)\right) d\nu(y).
$$

No integrability assumption is needed for Tonelli; the
iterated integrals may all be $+\infty$.

### Fubini's theorem

**Theorem (Fubini).** Let $f \in L^1(\mu \otimes \nu)$. Then
all three quantities in Tonelli's theorem are finite and
equal.

The standard practice: use Tonelli first on $|f|$ to check
that $\int |f| \, d(\mu \otimes \nu) < \infty$, then apply
Fubini to $f$ itself.

**When Fubini fails: the integrability condition is not
optional.** Define $f: \mathbb{N} \times \mathbb{N} \to
\mathbb{R}$ on counting measure by $f(m, n) = 1$ if $m = n$,
$f(m, n) = -1$ if $m = n + 1$, and $f(m, n) = 0$ otherwise.
Summing over $m$ first: for each $n$, $\sum_m f(m,n) =
f(n,n) + f(n+1,n) = 1 + (-1) = 0$, so $\sum_n \sum_m f(m,n)
= 0$. Summing over $n$ first: for $m = 1$, $\sum_n f(1,n) =
f(1,1) = 1$ (no $n = 0$ term); for $m \geq 2$,
$\sum_n f(m,n) = f(m,m) + f(m, m-1) = 1 + (-1) = 0$. So
$\sum_m \sum_n f(m,n) = 1 \neq 0$. Tonelli applied to $|f|$
shows $\int |f| = +\infty$: Fubini does not apply, and the
order of summation matters.

**Why econ cares.** Fubini is the theorem behind every
"change order of integration" step in a proof. Most
concretely:

- **Law of iterated expectations**: $E[X] = E[E[X | Y]]$.
  This follows from Fubini applied to the joint distribution
  of $(X, Y)$.
- **Joint distributions from conditionals**: if you specify
  $P(Y | X = x)$ and $P_X$, the joint measure exists (by the
  construction of product and regular conditional probability
  measures) and satisfies Fubini.
- **GMM moment conditions**: when the moment is
  $E[g(X, Z) \cdot h(\theta)]$ with instruments $Z$, the
  ability to compute this as $E_Z[E_{X|Z}[g(X,Z)]] \cdot h(\theta)$
  is Fubini.

---

## 8. Modes of convergence

Five distinct notions of convergence appear in asymptotic
econometrics. They form a strict hierarchy; knowing where
each lives is essential for reading proofs correctly.

Let $(\Omega, \mathcal{F}, P)$ be a probability space and
$X_n, X$ real-valued random variables on it.

**Almost sure convergence.** $X_n \xrightarrow{\text{a.s.}} X$ if

$$
P\!\left(\{\omega : X_n(\omega) \to X(\omega)\}\right) = 1.
$$

This is literal pointwise convergence of functions, except
on a set of probability zero. The strong law of large numbers
is an a.s. convergence result.

**Convergence in probability.** $X_n \xrightarrow{P} X$ if,
for every $\varepsilon > 0$,

$$
P(|X_n - X| > \varepsilon) \to 0 \quad \text{as } n \to \infty.
$$

The bad set can wander (unlike a.s. convergence, where it
must eventually stay empty), as long as its probability
shrinks. Consistency of an estimator means
$\hat{\theta}_n \xrightarrow{P} \theta_0$.

**$L^p$ convergence.** $X_n \xrightarrow{L^p} X$ if
$E[|X_n - X|^p] \to 0$ for $p \geq 1$. The case $p = 2$
(mean-square convergence) is the most common in linear
models. By Markov's inequality:

$$
P(|X_n - X| > \varepsilon)
\leq \frac{E[|X_n - X|^p]}{\varepsilon^p},
$$

so $L^p$ convergence implies convergence in probability.

**Convergence in distribution.** $X_n \xrightarrow{d} X$ if
the CDFs converge: $F_{X_n}(t) \to F_X(t)$ at every
continuity point $t$ of $F_X$. Equivalently (by the
**portmanteau lemma**): $E[h(X_n)] \to E[h(X)]$ for every
bounded continuous $h: \mathbb{R} \to \mathbb{R}$. This is
the weakest mode; it is a statement about distributions, not
about realizations. $X_n$ and $X$ need not live on the same
probability space. Asymptotic normality —
$\sqrt{n}(\hat{\theta}_n - \theta_0) \xrightarrow{d}
\mathcal{N}(0, V)$ — is a convergence-in-distribution claim.

**The hierarchy of implications.**

$$
X_n \xrightarrow{\text{a.s.}} X
\implies X_n \xrightarrow{P} X
\implies X_n \xrightarrow{d} X.
$$
$$
X_n \xrightarrow{L^p} X
\implies X_n \xrightarrow{P} X
\implies X_n \xrightarrow{d} X.
$$

None of the reverses holds in general. Counterexample for
a.s. not implied by in probability: the **typewriter sequence**
on $([0,1], \mathcal{B}([0,1]), \lambda)$. Enumerate the
dyadic intervals: $I_1 = [0,1]$, $I_2 = [0,\frac{1}{2}]$,
$I_3 = [\frac{1}{2},1]$, $I_4 = [0,\frac{1}{4}]$, $\ldots$
Let $X_n = \mathbf{1}_{I_n}$. Then $\lambda(I_n) \to 0$, so
$X_n \xrightarrow{P} 0$. But for every $x \in [0,1]$,
infinitely many $I_n$ contain $x$, so $\limsup X_n(x) = 1$
for a.e. $x$: no a.s. convergence.

**Key special cases and results.**

- If $X_n \xrightarrow{d} c$ for a constant $c$, then
  $X_n \xrightarrow{P} c$.
- **Subsequence principle**: $X_n \xrightarrow{P} X$ iff
  every subsequence $(X_{n_k})$ has a further subsequence
  $(X_{n_{k_j}})$ with $X_{n_{k_j}} \xrightarrow{\text{a.s.}} X$.
- **Slutsky's theorem**: if $X_n \xrightarrow{d} X$ and
  $Y_n \xrightarrow{P} c$ (a constant), then
  $X_n + Y_n \xrightarrow{d} X + c$ and
  $X_n Y_n \xrightarrow{d} cX$. Every $t$-statistic
  uses Slutsky: the numerator converges in distribution to
  a normal, the estimated standard error converges in
  probability to the true one, and Slutsky gives the
  asymptotic pivot.
- **Continuous mapping theorem (CMT)**: if
  $X_n \xrightarrow{d} X$ and $g$ is continuous, then
  $g(X_n) \xrightarrow{d} g(X)$. Proved via the portmanteau
  characterization. The delta method is the CMT with $g$
  linearized via a first-order Taylor expansion.
- **Uniform integrability and $L^1$ convergence**: the family
  $\{X_n\}$ is **uniformly integrable** if
  $\sup_n E[|X_n| \mathbf{1}_{|X_n| > M}] \to 0$ as
  $M \to \infty$. Then $X_n \xrightarrow{P} X$ implies
  $X_n \xrightarrow{L^1} X$. Domination by an integrable
  function is a sufficient condition for uniform
  integrability, connecting this back to the DCT.

**Why this section matters.** Consistency is in probability;
asymptotic normality is in distribution; these are different
claims. The delta method says: if
$\sqrt{n}(\hat{\theta}_n - \theta_0) \xrightarrow{d}
\mathcal{N}(0, V)$ and $g$ is differentiable at $\theta_0$,
then $\sqrt{n}(g(\hat{\theta}_n) - g(\theta_0))
\xrightarrow{d} \mathcal{N}(0, \nabla g(\theta_0)^\top V
\nabla g(\theta_0))$. The modes of convergence framework is
what makes this statement precise.

---

## 9. Measure-theoretic foundations of probability

**The probability space.** A **probability space** is a triple
$(\Omega, \mathcal{F}, P)$ with $\Omega$ the sample space
(the set of all possible outcomes), $\mathcal{F}$ a σ-algebra
on $\Omega$ (the **events**), and $P: \mathcal{F} \to [0,1]$
a probability measure ($P(\Omega) = 1$, countably additive).

The abstraction pays off in two ways. First, it unifies all
the probability we need: finite, countable, and continuous
sample spaces are all special cases. Second, it separates
the sample space (which is usually irrelevant beyond its
existence) from the distributions of random variables (which
are what we actually work with).

**Random variables.** A **random variable** is a measurable
function $X: (\Omega, \mathcal{F}) \to (\mathbb{R},
\mathcal{B}(\mathbb{R}))$. Measurability means $\{X \leq c\}
\in \mathcal{F}$ for every $c$, so the event "X takes a
value at most $c$" is a legitimate event to which $P$
assigns a probability.

The **distribution** (or law) of $X$ is the pushforward
measure $P_X$ on $\mathcal{B}(\mathbb{R})$: for
$B \in \mathcal{B}(\mathbb{R})$, $P_X(B) = P(X^{-1}(B))$.
The CDF $F_X(c) = P(X \leq c)$ is the evaluation of $P_X$
on half-lines. All properties of CDFs (right-continuity,
limits at $\pm\infty$) follow from the measure properties of
$P_X$.

**Expectation as Lebesgue integration.** The **expectation**
of $X$ is

$$
E[X] = \int_\Omega X(\omega) \, dP(\omega)
= \int_\mathbb{R} x \, dP_X(x).
$$

The second expression is the integral of the identity
function against the distribution of $X$; equality follows
from the change-of-variables theorem for pushforward
measures. When $X$ has a density $f_X$ (i.e., when
$P_X \ll \lambda$ and $f_X = dP_X/d\lambda$), this becomes

$$
E[X] = \int_\mathbb{R} x \, f_X(x) \, dx,
$$

the familiar formula. All properties of the Lebesgue
integral carry through: linearity, monotone convergence,
dominated convergence (the probabilistic form: if
$X_n \to X$ a.s. and $|X_n| \leq Z$ with $E[Z] < \infty$,
then $E[X_n] \to E[X]$).

**Independence.** Events $A, B \in \mathcal{F}$ are
**independent** if $P(A \cap B) = P(A) P(B)$. Random
variables $X, Y$ are **independent** if

$$
P_{(X,Y)} = P_X \otimes P_Y,
$$

equivalently if $P(X \in A, Y \in B) = P(X \in A) P(Y \in B)$
for all $A, B \in \mathcal{B}(\mathbb{R})$. Independence in
the sense of "$X$ and $Y$ have no relationship" is this
product-measure statement. The key consequence: if $X \perp Y$
and $E[|X|], E[|Y|] < \infty$, then $E[XY] = E[X] E[Y]$ —
by Fubini applied to the product distribution.

**The Radon-Nikodym theorem.** A measure $\nu$ on
$(X, \mathcal{A})$ is **absolutely continuous** with respect
to $\mu$ (written $\nu \ll \mu$) if $\mu(A) = 0 \Rightarrow
\nu(A) = 0$.

**Theorem (Radon-Nikodym).** If $\mu, \nu$ are σ-finite
measures on $(X, \mathcal{A})$ with $\nu \ll \mu$, then
there exists a nonneg measurable function $f$, unique
$\mu$-a.e., such that

$$
\nu(A) = \int_A f \, d\mu \quad \text{for all } A \in \mathcal{A}.
$$

The function $f = d\nu / d\mu$ is the **Radon-Nikodym
derivative** (or density) of $\nu$ with respect to $\mu$.

The standard modern proof goes through the Riesz
representation theorem for Hilbert spaces: consider the
functional $\Phi(g) = \int g \, d\nu$ on $L^2(\mu + \nu)$,
which is bounded; the Riesz theorem gives an $h \in L^2$
with $\Phi(g) = \int gh \, d(\mu + \nu)$; algebraic
manipulation then yields $f = h/(1-h)$. This proof belongs
in `functional-analysis.md` because it requires $L^2$
Hilbert space structure developed there.

**What Radon-Nikodym underpins in probability and statistics.**

1. **Densities**: $P_X \ll \lambda$ iff $X$ has a pdf; the
   pdf is $dP_X/d\lambda$, guaranteed unique $\lambda$-a.e.
2. **Likelihood ratios**: the likelihood ratio of $P_1$ with
   respect to $P_0$ is $dP_1/dP_0$. Neyman-Pearson,
   maximum likelihood, and the exponential family are all
   this framework.
3. **Conditional expectation**: the definition of
   $E[X | \mathcal{G}]$ is a Radon-Nikodym derivative (see
   below).

**Conditional expectation.** For $X \in L^1(P)$ and a
sub-σ-algebra $\mathcal{G} \subseteq \mathcal{F}$, the
**conditional expectation** $E[X | \mathcal{G}]$ is the
unique (up to a.s. equality) $\mathcal{G}$-measurable
function $Z$ satisfying

$$
\int_G Z \, dP = \int_G X \, dP \quad \text{for all }
G \in \mathcal{G}.
$$

Existence: the map $G \mapsto \int_G X \, dP$ is a finite
signed measure on $(\Omega, \mathcal{G})$, absolutely
continuous with respect to $P|_\mathcal{G}$; its
Radon-Nikodym derivative is $E[X | \mathcal{G}]$.

The essential properties:

- **Tower property** (law of iterated expectations):
  $E[E[X | \mathcal{G}]] = E[X]$. Taking $G = \Omega$
  in the definition.
- **Iterated conditioning**: if $\mathcal{H} \subseteq
  \mathcal{G}$, then $E[E[X|\mathcal{G}]|\mathcal{H}]
  = E[X|\mathcal{H}]$ a.s.
- **Taking out known factors**: if $Z$ is
  $\mathcal{G}$-measurable and $E[|ZX|] < \infty$, then
  $E[ZX|\mathcal{G}] = Z \cdot E[X|\mathcal{G}]$ a.s.
- **Independence**: if $X$ is independent of $\mathcal{G}$
  (meaning $\sigma(X)$ and $\mathcal{G}$ are independent),
  then $E[X|\mathcal{G}] = E[X]$ a.s.

The geometric interpretation — $E[X|\mathcal{G}]$ as the
$L^2(P)$ projection of $X$ onto the closed subspace of
$\mathcal{G}$-measurable square-integrable functions — belongs
in `functional-analysis.md`. That perspective makes all four
properties above obvious from the geometry of projections in
Hilbert spaces, but requires the $L^2$ structure developed
there.

---

## 10. What this file doesn't cover, and where to find it

**$L^p$ spaces as Banach and Hilbert spaces.** The completeness
of $L^p(\mu)$ (the Riesz-Fischer theorem), the identification
$(L^p)^* \cong L^q$ for $1 < p < \infty$ (Hölder duality),
the inner product structure of $L^2$ and the $L^2$ projection
theorem — all belong in `functional-analysis.md`. The $L^2$
projection characterization of conditional expectation, which
makes the tower property and regression geometry obvious, also
belongs there.

**Proof of the Radon-Nikodym theorem.** Stated here (§9);
proved in `functional-analysis.md` where the $L^2$ Hilbert
space machinery is available.

**Probability theory proper.** The weak law of large numbers,
the strong law, the central limit theorem, characteristic
functions, the delta method as a formal result, and weak
convergence of probability measures on $\mathbb{R}^k$ all go
in `probability.md`. The modes of convergence defined in §8
are the vocabulary for that file; the probability foundations
of §9 are the setup.

**Martingale theory.** Filtrations, adapted processes, the
optional stopping theorem, martingale convergence theorems —
belong in `stochastic-processes.md`. The connection to this
file: conditional expectation (§9) and its tower property are
the foundation of the martingale definition
($E[X_{n+1} | \mathcal{F}_n] = X_n$), and almost sure
convergence (§8) is the mode used in martingale convergence.

**Ergodic theory.** The ergodic theorem (the time-average
equals the space-average for stationary ergodic processes) is
the right version of the law of large numbers for dependent
data and belongs in `stochastic-processes.md`. It is what
underpins the asymptotics of time-series estimators.

**Stochastic integration.** Itô integrals and SDEs build on
both Brownian motion and the measure-theoretic foundations
here. They belong in `stochastic-processes.md`.

**Disintegration of measures.** For conditioning on an event
of probability zero (e.g., $Y = y$ for a continuous $Y$),
the disintegration theorem guarantees the existence of
conditional distributions $P(X \in \cdot | Y = y)$ as a
measurable family of probability measures. Not covered here;
the Radon-Nikodym theorem handles the $L^1$-based definition
of conditional expectation (§9), but the pointwise-$y$
version requires additional regularity.

---

## Cross-references

- `20_Math/real-analysis.md` — the direct prerequisite.
  Completeness of $\mathbb{R}$, metric spaces, and uniform
  convergence (§7, which is the Riemann analogue of the DCT)
  are all assumed here. The contraction mapping theorem (§6
  there) appears in the Hilbert-space proof of
  Radon-Nikodym deferred to `functional-analysis.md`.
- `20_Math/functional-analysis.md` — The $L^p$ spaces introduced
  here are treated as Banach and Hilbert spaces there
  (Riesz-Fischer theorem, $L^p$ duals). The Radon-Nikodym
  theorem is proved there via the Riesz representation theorem.
  The $L^2$ projection characterization of conditional
  expectation is developed there in full.
- `20_Math/probability.md` — The direct sequel for
  probabilistic limit theorems: LLNs (Chebyshev, Kolmogorov,
  ergodic, mixing), CLTs (classical, Lindeberg-Feller,
  multivariate, dependent-data), the delta method, and the
  $o_P/O_P$ asymptotic framework. The modes of
  convergence of §8 and the probability foundations of §9
  here are its prerequisites.
- `20_Math/stochastic-processes.md` — Martingale theory and
  stochastic integration build on conditional expectation
  (§9) and a.s./in-probability
  convergence (§8).
- `20_Math/optimization.md` — The applied counterpart to the
  foundational math layer: convexity, KKT, duality, envelope
  theorem, econometric applications, numerical methods. The
  dominated convergence theorem here justifies differentiating
  under the expectation sign in the MLE score and information
  matrix discussion.
- `10_Methods/modern-toolkit-references.md` — the
  econometrics directory. Consistency and asymptotic
  normality proofs for all estimators in the toolkit use the
  modes of convergence of §8. The moment conditions that
  appear throughout (finite second moments, bounded support,
  uniform integrability) are the domination conditions that
  make the DCT apply.
- `10_Methods/Econometrics/instrumental-variables.md` — the
  asymptotic theory of 2SLS and the modern weak-instrument
  literature invoke convergence-in-distribution results; the
  LATE framing involves expectations that are well-defined
  only through the measure-theoretic framework here.
- `10_Methods/Econometrics/local-projections.md` — the
  Plagborg-Møller-Wolf equivalence and the Montiel
  Olea-Plagborg-Møller long-horizon bias correction both rest
  on asymptotic arguments in the modes-of-convergence
  language of §8.
- `50_Workflows/run-a-regression-properly.md` — Stage 1's
  estimand framing invokes expectations of potential
  outcomes; Stage 4's identification discussion implicitly
  uses conditional expectations of the form $E[Y | D, X]$.
  Both are well-defined only through the probability
  foundations of §9.

---

*Last updated: 2026-04. This file builds directly on
`real-analysis.md` (stable, April 2026). The mathematical
content is standard and stable. The file will be revised if
`functional-analysis.md` or `probability.md` reveal
prerequisites that belong here rather than there — most
likely around disintegration of measures or the
Radon-Nikodym proof — and if further econometrics work
reveals gaps in the asymptotic foundations covered.*
