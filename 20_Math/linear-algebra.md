---
tags: [math, linear-algebra, foundations]
related: [real-analysis, functional-analysis, optimization, probability, modern-toolkit-references, Principles]
---

# Linear Algebra

This file is the finite-dimensional companion to
`functional-analysis.md`. That file developed the general
theory of normed spaces, Hilbert spaces, bounded operators, and
the spectral theorem for compact self-adjoint operators in
arbitrary (including infinite-dimensional) spaces. This file
specializes to $\mathbb{R}^n$, where the abstract machinery
acquires concrete matrix representations, constructive
decomposition algorithms, and the numerical considerations that
determine whether an implementation is trustworthy.

The results that matter most for econ work are: the projection
geometry that underlies OLS and its generalizations (§2), the
spectral theorem for real symmetric matrices and its
applications to PCA and VAR stability (§3), the QR
decomposition as the numerically correct way to solve least-
squares problems (§4), the singular value decomposition as the
canonical tool for low-rank approximation, rank-deficient
regression, and understanding what a matrix actually does (§5),
the Moore-Penrose pseudoinverse for ill-posed or
underidentified systems (§6), Cholesky for positive-definite
systems (§7), and the numerical linear algebra discipline that
separates trustworthy implementations from ones that silently
return garbage (§8).

This file is downstream of `functional-analysis.md` for the
abstract structure and upstream of `probability.md` (covariance
matrices, the delta method via matrix calculus, the Cramér-Rao
bound) and `stochastic-processes.md` (VAR stability via
companion matrices, state-space representations via matrix
exponentials). The cross-references at the end specify these
dependencies precisely.

Proofs are included for the spectral theorem for real symmetric
matrices and for the Gram-Schmidt construction underlying QR.
The SVD existence is proved in sketch. Other results are stated
with enough detail to apply correctly and understand where
things can go wrong.

---

## 1. Vector spaces, subspaces, and linear maps

A *vector space* over $\mathbb{R}$ is a set $V$ with addition
and scalar multiplication satisfying the usual axioms
(commutativity, associativity, distributivity, neutral
elements). $\mathbb{R}^n$ is the canonical finite-dimensional
vector space; a *basis* for $\mathbb{R}^n$ is any set of $n$
linearly independent vectors that span the whole space.

A *subspace* of $V$ is a nonempty subset closed under addition
and scalar multiplication. In $\mathbb{R}^n$, every subspace
is the column space of some matrix: the set
$\mathrm{col}(A) = \{Ax : x \in \mathbb{R}^k\}$ for some
$A \in \mathbb{R}^{n \times k}$. The *dimension* of
$\mathrm{col}(A)$ is the rank of $A$.

A *linear map* $T: \mathbb{R}^n \to \mathbb{R}^m$ satisfies
$T(\alpha x + \beta y) = \alpha Tx + \beta Ty$ and is
represented by a matrix $A \in \mathbb{R}^{m \times n}$ via
$Tx = Ax$. Composition of linear maps is matrix multiplication.
Every linear map between finite-dimensional spaces is bounded
(continuous), so all the operator-norm results from
`functional-analysis.md` §4 apply automatically.

**Four fundamental subspaces.** For $A \in \mathbb{R}^{m \times n}$:
- $\mathrm{col}(A) = \{Ax : x \in \mathbb{R}^n\} \subseteq \mathbb{R}^m$ — the column space (range of $A$)
- $\mathrm{null}(A) = \{x : Ax = 0\} \subseteq \mathbb{R}^n$ — the null space (kernel)
- $\mathrm{col}(A^\top) \subseteq \mathbb{R}^n$ — the row space
- $\mathrm{null}(A^\top) \subseteq \mathbb{R}^m$ — the left null space

The rank-nullity theorem: $\dim(\mathrm{col}(A)) + \dim(\mathrm{null}(A)) = n$. The fundamental theorem of linear algebra: $\mathrm{col}(A)$ and $\mathrm{null}(A^\top)$ are orthogonal complements in $\mathbb{R}^m$; $\mathrm{col}(A^\top)$ and $\mathrm{null}(A)$ are orthogonal complements in $\mathbb{R}^n$. Every vector in $\mathbb{R}^n$ decomposes uniquely into a component in $\mathrm{col}(A^\top)$ and a component in $\mathrm{null}(A)$.

---

## 2. Projection geometry and OLS

### 2.1 Orthogonal projection onto a subspace

Let $M \subseteq \mathbb{R}^n$ be a subspace (closed, since we
are in finite dimensions). The orthogonal projection of
$y \in \mathbb{R}^n$ onto $M$ is the unique $\hat y \in M$
minimizing $\|y - v\|$ over $v \in M$. It satisfies the
orthogonality condition: $y - \hat y \perp M$.

If $M = \mathrm{col}(A)$ for $A \in \mathbb{R}^{n \times k}$
with full column rank $k$, the projection is
$$\hat y = A(A^\top A)^{-1} A^\top y =: P_A y,$$
and $P_A = A(A^\top A)^{-1}A^\top$ is the *projection matrix*
(also called the hat matrix in the regression context). The
orthogonal complement projection is $M_A = I - P_A$.

Properties of $P_A$:
- *Idempotent*: $P_A^2 = P_A$ (projecting twice gives the same result).
- *Symmetric*: $P_A^\top = P_A$ (self-adjoint in the operator sense).
- *Rank*: $\mathrm{rank}(P_A) = k$.
- *Eigenvalues*: 0 (with multiplicity $n - k$) and 1 (with multiplicity $k$). This follows from idempotency: $P_A^2 x = P_A x$ implies $P_A(P_A - I)x = 0$, so eigenvalues satisfy $\lambda(\lambda - 1) = 0$.

Conversely, any symmetric idempotent matrix is an orthogonal projection.

### 2.2 OLS as projection

In the linear model $y = X\beta + \varepsilon$, OLS solves
$\min_\beta \|y - X\beta\|^2$. The solution $\hat\beta = (X^\top X)^{-1} X^\top y$ (when $X$ has full column rank) gives fitted values
$$\hat y = X\hat\beta = X(X^\top X)^{-1}X^\top y = P_X y.$$
The residuals $e = y - \hat y = M_X y$ lie in
$\mathrm{col}(X)^\perp = \mathrm{null}(X^\top)$. The normal
equations $X^\top e = 0$ are the orthogonality condition.

The geometry is independent of the probabilistic assumptions
about $\varepsilon$. OLS always projects $y$ onto
$\mathrm{col}(X)$. The Gauss-Markov theorem adds assumptions
($E[\varepsilon] = 0$, $\mathrm{Var}[\varepsilon] = \sigma^2 I$)
and concludes OLS is BLUE — but the projection is prior to and
separate from those assumptions.

### 2.3 Partitioned projection and the Frisch-Waugh theorem

Partition $X = [X_1, X_2]$ and write the projection identity:
$$P_X = P_{X_1} + P_{M_{X_1} X_2},$$
where $M_{X_1} = I - P_{X_1}$ residualizes $X_2$ on $X_1$.

The Frisch-Waugh-Lovell theorem: the OLS coefficient on $X_2$
in the full regression is identical to the OLS coefficient on
$M_{X_1} X_2$ in the regression of $M_{X_1} y$ on $M_{X_1} X_2$.
Partialling out $X_1$ from both $y$ and $X_2$ and regressing
gives the same coefficient as the full regression.

This is the matrix-algebra statement of the added-variable plot
(the regression of $y$ on $X_1$ and the regression of $X_2$
on $X_1$ leave residuals; the slope relating those residuals
is the coefficient on $X_2$). It justifies the common
practice of "controlling for" covariates by adding them to
the regression.

### 2.4 Leverage and influence

The hat matrix $H = P_X = X(X^\top X)^{-1}X^\top$ maps observed
$y$ to fitted $\hat y$. The diagonal entries $h_{ii}$ are the
*leverages* — the influence of the $i$-th observation on its
own fitted value. Since $H$ is a projection, $0 \leq h_{ii}
\leq 1$ and $\sum_i h_{ii} = \mathrm{rank}(H) = k$ (the number
of regressors). High-leverage observations ($h_{ii}$ near 1)
have $\hat y_i \approx y_i$ regardless of the model fit for
other observations — they anchor the regression.

The residual for observation $i$ is $e_i = (1 - h_{ii})(y_i - \hat y_{-i})$ where $\hat y_{-i}$ is the leave-one-out prediction. This is the connection between leverage and the delete-one residual, and it explains why high-leverage points can fit well (small residual) while being influential (large effect on $\hat\beta$).

---

## 3. Eigenvalues and the spectral theorem

### 3.1 Eigenvalues and eigenvectors

For $A \in \mathbb{R}^{n \times n}$, a scalar $\lambda \in \mathbb{C}$
is an *eigenvalue* and $v \neq 0$ is an *eigenvector* if
$Av = \lambda v$. The eigenvalues are the roots of the
*characteristic polynomial* $\det(A - \lambda I) = 0$. Over
$\mathbb{C}$, there are always $n$ eigenvalues counted with
multiplicity (fundamental theorem of algebra). Over $\mathbb{R}$,
real symmetric matrices always have all eigenvalues in $\mathbb{R}$.

The *trace* of $A$ equals the sum of eigenvalues; the
*determinant* of $A$ equals the product of eigenvalues. If any
eigenvalue is zero, $A$ is singular (not invertible). The
*spectral radius* $\rho(A) = \max_i |\lambda_i|$ governs the
long-run behavior of the matrix-power sequence $A^k$.

### 3.2 Spectral theorem for real symmetric matrices

**Theorem.** Let $A \in \mathbb{R}^{n \times n}$ be symmetric
($A = A^\top$). Then:

1. All eigenvalues of $A$ are real.
2. Eigenvectors corresponding to distinct eigenvalues are
   orthogonal.
3. $A$ is orthogonally diagonalizable: there exists an
   orthogonal matrix $Q$ ($Q^\top Q = I$) and a real diagonal
   matrix $\Lambda = \mathrm{diag}(\lambda_1, \ldots, \lambda_n)$
   such that $A = Q\Lambda Q^\top$.

**Proof.** 

**(1) Eigenvalues are real.** Suppose $Av = \lambda v$ for
$v \in \mathbb{C}^n$, $v \neq 0$. Take the conjugate transpose
of both sides: $v^* A^\top = \bar\lambda v^*$, so $v^* A = \bar\lambda v^*$ (using $A^\top = A$). Multiply the original
equation on the left by $v^*$:
$$v^* A v = \lambda v^*v = \lambda \|v\|^2.$$
Multiply the conjugated equation on the right by $v$:
$$v^* A v = \bar\lambda v^*v = \bar\lambda \|v\|^2.$$
Since $\|v\|^2 > 0$, we get $\lambda = \bar\lambda$, so $\lambda \in \mathbb{R}$.

**(2) Orthogonality of eigenvectors.** Suppose $Av_1 = \lambda_1 v_1$ and $Av_2 = \lambda_2 v_2$ with $\lambda_1 \neq \lambda_2$ and both real. Then
$$\lambda_1 v_1^\top v_2 = (Av_1)^\top v_2 = v_1^\top A^\top v_2 = v_1^\top A v_2 = v_1^\top (\lambda_2 v_2) = \lambda_2 v_1^\top v_2.$$
Since $\lambda_1 \neq \lambda_2$, we get $v_1^\top v_2 = 0$.

**(3) Orthogonal diagonalizability.** This follows by induction on $n$ (always produce one eigenvector, restrict to its orthogonal complement, which is also invariant under a symmetric matrix, and apply induction) combined with the Gram-Schmidt process to orthonormalize eigenvectors within each eigenspace. $\blacksquare$

The spectral expansion $A = Q\Lambda Q^\top = \sum_{i=1}^n \lambda_i q_i q_i^\top$ expresses $A$ as a sum of rank-1 projections onto its eigenvectors, scaled by eigenvalues. This is the finite-dimensional case of the spectral theorem for compact self-adjoint operators proved in `functional-analysis.md` §8.2.

### 3.3 Positive (semi)definite matrices

A symmetric matrix $A$ is *positive semidefinite* (PSD,
$A \succeq 0$) if $x^\top A x \geq 0$ for all $x$, and
*positive definite* (PD, $A \succ 0$) if $x^\top A x > 0$
for all $x \neq 0$.

Equivalent characterizations:
- $A \succeq 0$ iff all eigenvalues $\lambda_i \geq 0$.
- $A \succ 0$ iff all eigenvalues $\lambda_i > 0$.
- $A \succeq 0$ iff $A = B^\top B$ for some $B$ (not necessarily square).
- $A \succ 0$ iff $A = B^\top B$ for some $B$ with full column rank.

The sample covariance matrix $\hat\Sigma = n^{-1} X^\top X$ is PSD always and PD when $X$ has full column rank. The information matrix $\mathcal{I}(\theta) = E[-\nabla^2_\theta \log p(X;\theta)]$ is PSD; PD is an identification condition. PSD matrices are the natural domain for the Cholesky decomposition (§7).

### 3.4 Eigenvalues in econometrics

**PCA.** The sample covariance matrix $\hat\Sigma \in
\mathbb{R}^{p \times p}$ (for $p$ variables) is symmetric and
PSD. Its spectral decomposition $\hat\Sigma = Q\Lambda Q^\top$
gives the principal components (columns of $Q$) and the
variance along each direction (diagonal of $\Lambda$). The
first $k$ principal components are the $k$ directions of
greatest variance, solving:
$$\max_{V \in \mathbb{R}^{p \times k},\, V^\top V = I} \mathrm{tr}(V^\top \hat\Sigma V).$$
The solution is $V = Q_{:,1:k}$ (the first $k$ columns of $Q$).
PCA is eigendecomposition of the sample covariance.

**VAR stability.** A VAR(p) in $n$ variables has a companion
matrix $C \in \mathbb{R}^{np \times np}$ whose eigenvalues
determine stationarity: the VAR is stable (covariance-
stationary) if and only if all eigenvalues of $C$ satisfy
$|\lambda_i| < 1$ — i.e., the spectral radius $\rho(C) < 1$.
The impulse response function at horizon $h$ is the $(1,1)$
block of $C^h$, so eigenvalues with $|\lambda_i|$ close to 1
("near unit roots") produce persistent impulse responses that
decay slowly. Checking VAR stability is eigenvalue computation.

**Collinearity diagnostics.** The eigenvalues of $X^\top X$
diagnose multicollinearity. If the smallest eigenvalue
$\lambda_{\min}$ is near zero, the system is nearly singular.
The condition number $\kappa(X^\top X) = \lambda_{\max}/
\lambda_{\min}$ (the ratio of largest to smallest eigenvalue)
measures ill-conditioning. $\kappa \approx 10^d$ means roughly
$d$ digits of precision are lost in a linear solve — so
$\kappa \approx 10^8$ in double precision (15-16 digits) leaves
only 7-8 significant digits in $\hat\beta$.

---

## 4. QR decomposition

### 4.1 The decomposition

For $A \in \mathbb{R}^{m \times n}$ with $m \geq n$, the (thin)
QR decomposition is $A = QR$ where $Q \in \mathbb{R}^{m \times n}$
has orthonormal columns ($Q^\top Q = I_n$) and
$R \in \mathbb{R}^{n \times n}$ is upper triangular with
positive diagonal.

If $A$ has full column rank, the decomposition is unique. If
$m = n$, the full QR has $Q$ orthogonal ($Q^\top Q = QQ^\top = I$).

### 4.2 Gram-Schmidt construction

**Theorem (QR via Gram-Schmidt).** Every $A \in \mathbb{R}^{m \times n}$ with linearly independent columns has a thin QR decomposition.

**Proof (constructive).** Let $a_1, \ldots, a_n$ be the columns
of $A$. Define:
$$q_1 = \frac{a_1}{\|a_1\|},$$
and for $k = 2, \ldots, n$:
$$\tilde q_k = a_k - \sum_{j=1}^{k-1} \langle a_k, q_j \rangle q_j, \qquad q_k = \frac{\tilde q_k}{\|\tilde q_k\|}.$$
Each $\tilde q_k$ subtracts from $a_k$ its projections onto
the previously constructed orthonormal vectors, leaving the
component perpendicular to all of $q_1, \ldots, q_{k-1}$.
Since the columns are linearly independent, $\tilde q_k \neq 0$
at each step.

The matrix $Q = [q_1, \ldots, q_n]$ has orthonormal columns by
construction. Setting $R_{jk} = \langle a_k, q_j \rangle$ for
$j \leq k$ and $R_{jk} = 0$ for $j > k$ gives $A = QR$ with
$R$ upper triangular. $\blacksquare$

The Gram-Schmidt construction is numerically unstable for
nearly linearly dependent columns (the subtraction of nearly
equal numbers causes cancellation). The *modified Gram-Schmidt*
algorithm and *Householder reflections* are the numerically
stable implementations used in practice. All software QR
routines (LAPACK's `dgeqrf`, R's `qr()`, NumPy's
`linalg.qr`) use Householder reflections.

### 4.3 Least squares via QR

The least-squares problem $\min_\beta \|y - X\beta\|^2$ with
$X = QR$ (thin QR of $X$) becomes:
$$\|y - QR\beta\|^2 = \|Q^\top y - R\beta\|^2 + \|M_Q y\|^2,$$
using the Pythagorean theorem and the fact that $Q$ has
orthonormal columns. Minimizing over $\beta$ requires
$R\beta = Q^\top y$, a triangular system solved by back
substitution in $O(n^2)$ operations rather than the $O(n^3)$
inversion of $X^\top X$.

The QR solution $\hat\beta = R^{-1} Q^\top y$ avoids forming
$X^\top X$ — which squares the condition number — and is
numerically stable for ill-conditioned $X$. All statistical
software that correctly implements OLS uses QR internally.
`lm()` in R uses LAPACK's QR; `numpy.linalg.lstsq` uses the
SVD (§5) for even greater robustness in rank-deficient cases.

---

## 5. Singular value decomposition

### 5.1 The decomposition

**Theorem (SVD).** For any $A \in \mathbb{R}^{m \times n}$,
there exist orthogonal matrices $U \in \mathbb{R}^{m \times m}$
and $V \in \mathbb{R}^{n \times n}$ and a matrix
$\Sigma \in \mathbb{R}^{m \times n}$ (zeros off the main
diagonal, with nonneg diagonal entries $\sigma_1 \geq \sigma_2
\geq \cdots \geq \sigma_r > 0 = \sigma_{r+1} = \cdots$, where
$r = \mathrm{rank}(A)$) such that
$$A = U \Sigma V^\top.$$
The diagonal entries $\sigma_i$ are the *singular values* of $A$.

**Proof (sketch via eigendecomposition of $A^\top A$).** The
matrix $A^\top A \in \mathbb{R}^{n \times n}$ is symmetric and
PSD, so by the spectral theorem (§3.2), $A^\top A = V \Lambda V^\top$
with $V$ orthogonal and $\Lambda = \mathrm{diag}(\lambda_1, \ldots, \lambda_n)$, $\lambda_i \geq 0$. Set $\sigma_i = \sqrt{\lambda_i}$.

For each $\sigma_i > 0$, define $u_i = A v_i / \sigma_i$
where $v_i$ is the $i$-th column of $V$. Then:
$$u_i^\top u_j = \frac{v_i^\top A^\top A v_j}{\sigma_i \sigma_j} = \frac{v_i^\top \lambda_j v_j}{\sigma_i \sigma_j} = \frac{\lambda_j \delta_{ij}}{\sigma_i \sigma_j} = \delta_{ij}.$$
So $\{u_1, \ldots, u_r\}$ are orthonormal. Extend to a full
orthonormal basis of $\mathbb{R}^m$ to form $U$. By
construction, $Av_i = \sigma_i u_i$ for $i \leq r$ and
$Av_i = 0$ for $i > r$, which gives $A = U\Sigma V^\top$.
$\blacksquare$

### 5.2 Geometric interpretation

The SVD says that any linear map $A: \mathbb{R}^n \to
\mathbb{R}^m$ decomposes into three steps:
1. **Rotate** in the domain: $x \mapsto V^\top x$ (orthogonal transformation in $\mathbb{R}^n$).
2. **Scale** along coordinate axes: $(V^\top x)_i \mapsto \sigma_i (V^\top x)_i$ (with possible rank reduction).
3. **Rotate** in the codomain: $\mapsto U(\cdot)$ (orthogonal transformation in $\mathbb{R}^m$).

Every matrix is a rotation, then a scaling, then another
rotation. The singular values measure how much the map
stretches in each direction; the largest singular value is the
operator norm $\|A\| = \sigma_1$.

The *economy* (thin) SVD: $A = U_r \Sigma_r V_r^\top$, where
$U_r$ is the first $r$ columns of $U$, $\Sigma_r =
\mathrm{diag}(\sigma_1, \ldots, \sigma_r)$, and $V_r$ is the
first $r$ columns of $V$. This is the computationally useful
form when $r \ll \min(m, n)$.

### 5.3 Low-rank approximation

**Theorem (Eckart-Young-Mirsky).** The best rank-$k$
approximation to $A$ in the Frobenius norm (and in the operator
norm) is
$$A_k = \sum_{i=1}^k \sigma_i u_i v_i^\top = U_k \Sigma_k V_k^\top.$$
The approximation error is $\|A - A_k\|_F^2 = \sum_{i=k+1}^r \sigma_i^2$.

Applications: **factor models** in macro and finance are
low-rank approximations to panels of time series — the $k$
factors are the first $k$ left singular vectors; **text
analysis** (LSA) is a low-rank SVD of a document-term matrix;
**matrix completion** (for partially observed matrices)
exploits the low-rank structure via nuclear-norm regularization.

The *effective rank* of a matrix — the number of singular
values above a chosen threshold — diagnoses how many latent
dimensions are informative. A steep drop in the ordered
singular values (a "scree plot" for the SVD rather than PCA)
indicates a clean low-rank structure.

---

## 6. The Moore-Penrose pseudoinverse

### 6.1 Definition

For $A \in \mathbb{R}^{m \times n}$ with SVD $A = U\Sigma V^\top$, the *Moore-Penrose pseudoinverse* is
$$A^+ = V \Sigma^+ U^\top,$$
where $\Sigma^+$ replaces each nonzero singular value $\sigma_i$
by $1/\sigma_i$ and leaves zeros unchanged.

The pseudoinverse satisfies the four Moore-Penrose conditions:
1. $AA^+A = A$
2. $A^+AA^+ = A^+$
3. $(AA^+)^\top = AA^+$ (i.e., $AA^+$ is symmetric)
4. $(A^+A)^\top = A^+A$ (i.e., $A^+A$ is symmetric)

When $A$ is square and invertible, $A^+ = A^{-1}$. When $A$
has full column rank, $A^+ = (A^\top A)^{-1}A^\top$ (the left
inverse). When $A$ has full row rank, $A^+ = A^\top(AA^\top)^{-1}$
(the right inverse). In the rank-deficient case, none of these
formulas apply and one must use the SVD directly.

### 6.2 Pseudoinverse and least squares

The least-squares problem $\min_\beta \|y - A\beta\|^2$ always
has a solution. When $A$ has full column rank, the unique
solution is $A^+y = (A^\top A)^{-1}A^\top y$. When $A$ is
rank-deficient (or when $m < n$, i.e., there are fewer
observations than parameters), the problem is underdetermined
and has infinitely many solutions. The pseudoinverse selects
the *minimum-norm* solution: $\hat\beta = A^+y$ has the
smallest Euclidean norm among all $\beta$ minimizing
$\|y - A\beta\|$.

**Application: rank-deficient regressors.** Perfect
multicollinearity ($X$ rank-deficient) means the OLS normal
equations have infinitely many solutions. The pseudoinverse
selects one — but the researcher should ask why the
multicollinearity exists before blindly using $X^+y$. Perfect
collinearity usually signals a model specification error
(including redundant dummies, forgetting to exclude a base
category) rather than a computation problem.

**Application: underidentified GMM.** When the dimension of
$\theta$ exceeds the number of moment conditions, the GMM
system $g(\theta) = 0$ is underdetermined and the GMM
objective is minimized by many $\theta$ values. The
pseudoinverse characterizes the minimum-norm GMM estimator,
but the real remedy is to add moment conditions (instruments)
or impose restrictions that reduce the dimension of $\theta$.

---

## 7. Cholesky decomposition

### 7.1 The decomposition

For $A \in \mathbb{R}^{n \times n}$ symmetric and positive
definite, the *Cholesky decomposition* is $A = LL^\top$, where
$L$ is a lower triangular matrix with positive diagonal
entries. The decomposition is unique.

**Why it exists.** Since $A \succ 0$, the leading principal
minors $\det(A_{k \times k}) > 0$ for all $k$ (Sylvester's
criterion). The Cholesky algorithm constructs $L$ column by
column: the $(i,j)$ entry is determined by $A_{ij} =
\sum_{k=1}^j L_{ik} L_{jk}$, which gives $L_{jj}$ as the
square root of a positive quantity (guaranteed by the leading
minor condition) and $L_{ij}$ for $i > j$ as a ratio.

Cholesky costs $O(n^3/3)$ flops — half the cost of LU
decomposition — and is the most numerically stable direct
solver for positive-definite systems because the diagonal
entries of $L$ are square roots of positive quantities and
cannot blow up.

### 7.2 Applications

**Solving positive-definite linear systems.** To solve $Ax = b$
with $A \succ 0$: compute $L$ from $A = LL^\top$ once, then
solve $Ly = b$ (forward substitution) and $L^\top x = y$
(backward substitution). Both triangular solves are $O(n^2)$.
This is the standard approach for variance-covariance matrix
inversions in MLE and FGLS.

**Simulating correlated random variables.** To draw
$X \sim N(\mu, \Sigma)$: compute $L$ from $\Sigma = LL^\top$,
draw $Z \sim N(0, I_n)$, and set $X = \mu + LZ$. The
covariance is $E[LZ Z^\top L^\top] = L I L^\top = \Sigma$.
This is the standard simulation approach in bootstrap
inference, Monte Carlo integration, and Bayesian MCMC.

**Testing positive definiteness.** A symmetric matrix is PD
if and only if its Cholesky decomposition exists with positive
diagonal. This is computationally cheaper than eigendecomposition
and is used in practice to verify that an estimated covariance
matrix is valid (a common failure mode in numerical estimation
is a near-PSD matrix that has tiny negative eigenvalues due to
floating-point error — detected by Cholesky failure).

---

## 8. Numerical linear algebra

### 8.1 The condition number

The *condition number* of a nonsingular matrix $A$ is
$$\kappa(A) = \|A\| \cdot \|A^{-1}\| = \frac{\sigma_{\max}(A)}{\sigma_{\min}(A)},$$
the ratio of the largest to the smallest singular value (using
the operator norm, which equals the largest singular value).

The condition number measures the sensitivity of the solution
to a linear system $Ax = b$ to perturbations in $b$ or $A$:
$$\frac{\|\delta x\|}{\|x\|} \leq \kappa(A) \cdot \frac{\|\delta b\|}{\|b\|}.$$
A relative error of $\varepsilon$ in $b$ can produce a relative
error of up to $\kappa(A) \cdot \varepsilon$ in $x$. In
double-precision arithmetic with machine epsilon
$\varepsilon_{\text{mach}} \approx 10^{-16}$, a matrix with
$\kappa \approx 10^{10}$ leaves only about 6 significant digits
in the solution.

The condition number of $X^\top X$ is $\kappa(X^\top X) =
\kappa(X)^2$. This is why forming the normal equations and
inverting $X^\top X$ squares the condition number relative to
working with $X$ directly via QR.

### 8.2 Never invert matrices

The most important numerical advice in linear algebra: **do
not compute $A^{-1}$ to solve $Ax = b$**. Compute the LU
(or Cholesky for PD) factorization and solve by substitution.

The reasons are both computational and numerical:
- *Cost*: Computing $A^{-1}$ via Gaussian elimination costs
  $O(n^3)$ and then applying it to $b$ costs $O(n^2)$.
  Factoring $A$ and solving triangular systems costs $O(n^3)$
  total — the same, but the forward and backward substitution
  are $O(n^2)$ if the factorization is reused for multiple
  right-hand sides $b$.
- *Accuracy*: The inverse is computed with the same relative
  error bound as any other solve, but it is then multiplied by
  $b$, accumulating additional rounding error. The direct
  factorization-solve approach is numerically equivalent and
  often more stable in practice.

In econometrics: never compute $(X^\top X)^{-1}$ explicitly to
form $\hat\beta$. Let `lm()`, `fixest`, or `pyfixest` handle
the QR factorization. Never compute $\hat\Sigma^{-1}$ explicitly
for GLS or GMM weighting — use a Cholesky solve. Never write
`solve(t(X) %*% X) %*% t(X) %*% y` in R for OLS. Write
`lm(y ~ X)` or `qr.solve(X, y)`.

### 8.3 When to use which decomposition

| Problem | Matrix type | Recommended |
|---|---|---|
| Solve $Ax = b$ | General square | LU (Gaussian elimination with pivoting) |
| Solve $Ax = b$ | Symmetric PD | Cholesky |
| Least squares $\min \|Ax - b\|$ | Full column rank | QR |
| Least squares, rank-deficient or ill-conditioned | Any | SVD / pseudoinverse |
| Eigenvalues of symmetric matrix | Symmetric | Spectral decomposition (LAPACK `dsyev`) |
| Eigenvalues of general matrix | General | QZ algorithm (LAPACK `dgeev`) |
| Low-rank approximation | Any | Economy SVD |
| Simulate from $N(\mu, \Sigma)$ | Symmetric PD | Cholesky of $\Sigma$ |

**Never invert to get $A^{-1}$** in any of these cases. The
factorization is always preferable.

### 8.4 Floating-point and cancellation

Floating-point arithmetic is not exact. The key pathologies:

**Catastrophic cancellation.** Subtracting two nearly equal
quantities loses significant digits. Example: $(1 + \delta) - 1 = \delta$ in exact arithmetic, but in floating point with $\delta < \varepsilon_{\text{mach}}$, the result is 0. This arises in computing the sample variance as $\sum x_i^2 / n - \bar x^2$ (numerically dangerous, produces negative variances for small variances) rather than $\sum (x_i - \bar x)^2 / n$ (stable). The two-pass formula is always preferred.

**Rank determination.** Deciding whether a matrix is truly
rank-deficient or merely ill-conditioned is fundamentally a
judgment call depending on the threshold applied to singular
values. LAPACK uses $\sigma_i > \varepsilon_{\text{mach}}
\cdot \sigma_1 \cdot \max(m, n)$ as the default rank threshold.
In econometrics: a near-zero singular value of $X$ signals
near-multicollinearity; whether to treat this as a numerical
artifact or as genuine near-identification is a subject-matter
question, not a computational one.

**Iterative refinement.** After solving $Ax = b$ to get $\hat x$,
the residual $r = b - A\hat x$ can be used to correct the
solution: solve $A\delta x = r$ and set $\hat x \leftarrow
\hat x + \delta x$. One step of iterative refinement typically
doubles the number of significant digits. This is available in
LAPACK via the `_gerfs` routines and is useful when
$\kappa(A)$ is large but not catastrophically so.

---

## 9. What this file doesn't cover, and where to find it

**Abstract linear algebra over general fields.** The file
focuses on $\mathbb{R}^n$. Abstract vector spaces over general
fields, modules, and the algebraic theory of linear maps are
not covered.

**Complex matrices.** Hermitian matrices, unitary
transformations, and the DFT (discrete Fourier transform as a
unitary matrix) appear in spectral analysis. The standard
references (Golub-Van Loan, Horn-Johnson) cover these in detail.

**Tensor algebra.** Higher-order arrays and tensor
decompositions (Tucker, PARAFAC/CP) appear in multilinear
factor models and in machine learning. Not covered here.

**Random matrices.** The Marchenko-Pastur law, the Wigner
semicircle, and the Tracy-Widom distribution describe the
eigenvalue behavior of large random matrices and are relevant
for high-dimensional asymptotics (the "big $p$, big $n$"
regime). These go in `probability.md` or a future file.

**Matrix calculus.** Derivatives of scalar-valued functions of
matrices ($\partial \log\det A / \partial A = A^{-\top}$,
Jacobians of the SVD and Cholesky) are used in deriving
information matrices and gradient-based optimization of
matrix-valued objectives. These are in `optimization.md` §6.2
(the information matrix) and would be expanded in a future
`matrix-calculus.md` if the researcher's structural work
demands it.

---

## Cross-references

- `20_Math/functional-analysis.md` — the abstract foundation
  for this file. The projection theorem (§3.2 there) and the
  Riesz representation theorem (§3.4 there) specialized to
  finite-dimensional Hilbert spaces give the geometric content
  of §2 here; the spectral theorem for compact self-adjoint
  operators (§8.2 there) specialized to $\mathbb{R}^n$ gives
  the spectral theorem of §3.2 here.
- `20_Math/real-analysis.md` — the condition-number discussion
  (§8.1 here) uses the operator norm (bounded from below by
  the inverse function theorem's Jacobian condition from §5
  there); matrix exponentiation and the Neumann series connect
  to the contraction mapping theorem (§6 there).
- `20_Math/optimization.md` — OLS as a convex quadratic
  program (§6.1 there); the Hessian of a quadratic form is a
  symmetric matrix, and its eigenvalues determine the
  second-order condition (PD Hessian iff unique minimizer);
  the QR-based solve (§4.3 here) is the numerically correct
  implementation of the OLS formula in §6.1 there.
- `20_Math/probability.md` — *to be written*. The covariance
  matrix is the key symmetric PSD matrix in probability; the
  Cramér-Rao bound is stated in terms of the information
  matrix (a PSD matrix) and its inverse; the delta method
  requires matrix multiplication of Jacobians; the CLT for
  vectors uses the multivariate normal, whose density involves
  $\Sigma^{-1}$ (computed via Cholesky).
- `20_Math/stochastic-processes.md` — *to be written*. VAR
  stability via eigenvalues of the companion matrix (§3.4
  here); the state-space representation of ARMA processes uses
  matrix exponentiation; the Kalman filter involves Cholesky
  (§7 here) for the covariance prediction step.
- `10_Methods/Econometrics/panel-methods.md` — *to be written*.
  The within-transformation (demeaning) is a projection matrix
  $M_{D_\alpha}$ where $D_\alpha$ is the matrix of group
  indicators; Arellano-Bond uses a specific weighting matrix
  for the GMM objective.
- `10_Methods/Econometrics/structural-labor.md` — *to be
  written*. AKM estimation requires solving a large sparse
  linear system (the normal equations for the two-way fixed
  effects model); the leave-out estimator uses the diagonal
  of the hat matrix (leverages, §2.4 here).
- `10_Methods/Econometrics/double-machine-learning.md` — *to
  be written*. The Frisch-Waugh-Lovell theorem (§2.3 here)
  is the linear-algebra motivation for the DML partialling-out
  approach: DML generalizes Frisch-Waugh from linear residuals
  to nonparametric residuals.
- `50_Workflows/run-a-regression-properly.md` — Stage 3
  (specification) of the regression workflow uses the FWL
  theorem (§2.3) for thinking about partial effects; Stage 7
  (estimation) relies on numerical stability considerations
  from §8.

---

*Last updated: April 2026.*
