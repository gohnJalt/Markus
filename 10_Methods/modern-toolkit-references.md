# Modern toolkit references

This file is the bridge between methods and tools. For each method
in Markus's working repertoire, it answers four questions:

1. What is the canonical paper or papers — the references the
   literature actually cites?
2. What is the recommended library implementation, and in which
   language?
3. What does the method assume, in one or two sentences? Full
   treatment lives in the method-specific notes.
4. What are the most common ways it gets misused, so Markus can
   catch them before they ship?

The point is not to replace the deeper method notes in this folder.
The point is to make sure that when Markus reaches for a method, he
reaches for the right implementation of the right version of it,
and he knows what would make a referee unhappy.

When in doubt, the principle from `Principles.md` Section II
applies: the estimand is not the estimator. Picking the right
library does not absolve Markus of thinking about whether the
estimator answers the question.

---

## Difference-in-differences

The DiD literature went through a methodological revolution
between roughly 2018 and 2023. The revolution is now consolidated,
and Markus uses heterogeneity-robust estimators by default for any
staggered design. Two-way fixed effects on staggered treatment is
deprecated except in specific cases where the researcher knows
the homogeneity assumption holds. 
See: Econometrics/difference-in-differences.md for the deep note.

### Two-period, two-group (canonical DiD)

**Canonical references**: Card and Krueger (1994) on the New
Jersey minimum wage; Ashenfelter and Card (1985) earlier. The
basic estimator is OLS with unit and time fixed effects.

**Implementation**: any OLS routine with cluster-robust standard
errors. `pyfixest` (Python) or `fixest` (R) are the fastest;
`statsmodels` and `linearmodels` work fine for small problems.
Cluster at the level of treatment assignment.

**What it assumes**: parallel trends in the absence of treatment.
This is untestable in the strict sense; pre-trend plots are
suggestive but not dispositive.

**Common errors**: clustering at the wrong level, treating
pre-trend tests as proof of parallel trends, ignoring spillovers
between treatment and control units.

### Staggered adoption (the modern problem)

**Canonical references**: Goodman-Bacon (2021) for the
diagnosis; Callaway and Sant'Anna (2021), de Chaisemartin and
D'Haultfœuille (2020, 2024), Sun and Abraham (2021), Borusyak,
Jaravel, and Spiess (2024) for the solutions. Roth, Sant'Anna,
Bilinski, and Poe (2023) is the survey to read first.

**The problem in one sentence**: with staggered treatment timing
and heterogeneous treatment effects, two-way fixed effects can
put negative weights on some treatment effects, producing
estimates that are not even a convex combination of the
underlying ATTs and can have the wrong sign.

**Implementations**:
- **Callaway-Sant'Anna**: `did` (R), `differences` (Python),
  `csdid` (Stata).
- **de Chaisemartin-D'Haultfœuille**: `DIDmultiplegt` and
  `DIDmultiplegtDYN` (R, Stata).
- **Sun-Abraham**: `eventstudyinteract` (Stata), `fixest::sunab`
  (R, integrated into the `fixest` event-study syntax).
- **Borusyak-Jaravel-Spiess (imputation)**: `did_imputation`
  (Stata), `didimputation` (R).
- **GPU-accelerated modern DiD**: `moderndid`
  (`jordandeklerk/moderndid`) for very large panels.

**What they assume**: parallel trends (sometimes conditional on
covariates, depending on the specific estimator), and
"no-anticipation" (units don't change behavior before treatment).
The estimators differ in which exact moments they use and how
they aggregate, but the core identifying assumption is the same
family.

**Common errors**: running TWFE on a staggered design and not
checking; using a heterogeneity-robust estimator but reporting
only the overall ATT when the heterogeneity itself is the
finding; ignoring covariate balance when the estimator requires
conditional parallel trends.

### Continuous treatment DiD

**Canonical references**: Callaway, Goodman-Bacon, and Sant'Anna
(2024).

**Implementation**: `contdid` (`bcallaway11/contdid`).

**What it assumes**: a stronger form of parallel trends that
holds across the entire treatment intensity distribution, not
just for treated vs untreated. This is a meaningful additional
assumption and the researcher should engage with it before using
the estimator.

### Synthetic difference-in-differences

**Canonical reference**: Arkhangelsky, Athey, Hirshberg, Imbens,
and Wager (2021).

**Implementation**: `synthdid` (R, Python). Combines synthetic
control's unit weighting with DiD's outcome modeling.

**Use case**: when you have one or a few treated units and a
panel of potential controls, and pure synthetic control feels
too restrictive.

---

## Synthetic control

**Canonical references**: Abadie and Gardeazabal (2003) for the
original; Abadie, Diamond, and Hainmueller (2010, 2015) for the
methodological development; Abadie (2021) for the survey.

**Implementations**:
- **Original**: `Synth` (R, Stata).
- **Augmented synthetic control**: `augsynth` (R, Ben-Michael,
  Feller, Rothstein 2021). The current default for most
  applications — bias-corrects when the pre-treatment fit is
  imperfect.
- **Generalized synthetic control**: `gsynth` (R, Xu 2017).
  Handles multiple treated units within a factor model
  framework.
- **Python**: `pysyncon`, `SparseSC` (Microsoft).

**What it assumes**: the convex combination of donor units that
matches the treated unit pre-treatment continues to provide a
valid counterfactual post-treatment. Strong, untestable, and
defensible only when pre-treatment fit is good and the donor pool
is plausible.

**Common errors**: cherry-picking the donor pool; using the
method when pre-treatment fit is poor (which augmented synthetic
control was designed to address but doesn't fully solve);
reporting point estimates without permutation-based inference
(placebo tests on untreated units).

---

## Instrumental variables

### Linear IV / 2SLS

**Canonical references**: Angrist and Krueger (1991) is the
original revival; Angrist and Pischke (2008) chapter 4 for the
modern textbook treatment; Andrews, Stock, and Sun (2019) for
weak instrument diagnostics.

**Implementations**:
- **`linearmodels.iv`** (Python) — the cleanest Python IV
  implementation, with proper SE options and weak-instrument
  diagnostics built in.
- **`ivreg2`** and **`ivreghdfe`** (Stata) — the workhorses;
  `ivreghdfe` extends with high-dimensional fixed effects.
- **`fixest::feols`** with the IV syntax (R) — fast and clean.
- **`AER::ivreg`** (R) — the older R standard, less feature-rich.

**What it assumes**: relevance (the instrument predicts the
endogenous variable, conditional on controls — testable) and
exclusion (the instrument affects the outcome only through the
endogenous variable, conditional on controls — untestable and
the source of most IV controversies).

**Common errors**:
- Reporting first-stage F-stats below the Stock-Yogo critical
  values without acknowledging weak instruments.
- Using the rule-of-thumb F > 10 when Lee, McCrary, Moreira,
  and Porter (2022) and others have shown the threshold should
  be much higher (104 for AR confidence intervals at 5% size).
- Conflating LATE with ATT or ATE — the LATE applies to
  compliers, who may not be the population of interest.
- Defending exclusion with a story rather than an argument; the
  story should specify what would falsify it.

See: Econometrics/instrumental-variables.md for the deep note

### Shift-share / Bartik instruments

**Canonical references**: Goldsmith-Pinkham, Sorkin, and Swift
(2020) for the identification analysis; Borusyak, Hull, and
Jaravel (2022) for the alternative shock-based perspective;
Adão, Kolesár, and Morales (2019) for inference.

**Implementations**:
- **`ShiftShareSE`** (R, Stata) for Adão-Kolesár-Morales
  inference.
- For estimation, standard IV routines apply; the question is
  which weights and which inference.

**What it assumes**: depends on which interpretation. Goldsmith-
Pinkham et al. show the validity rests on the exogeneity of the
shares; Borusyak-Hull-Jaravel show an alternative rests on the
exogeneity of the shocks. These are different assumptions and
the researcher should be explicit about which one is being made.

### Judge fixed effects, examiner designs

**Canonical references**: Kling (2006), Doyle (2007), Aizer and
Doyle (2015), Dobbie, Goldin, and Yang (2018). Frandsen, Lefgren,
and Leslie (2023) on the testable implications.

**Implementation**: standard 2SLS with judge or examiner FE as
the instrument. The Frandsen-Lefgren-Leslie test for monotonicity
is implemented in Stata and R.

**Common errors**: ignoring monotonicity violations; assuming
monotonicity in settings where examiners may treat different
defendants differently along multiple dimensions.

---

## Regression discontinuity

**Canonical references**: Lee (2008) for the validity argument;
Imbens and Lemieux (2008) for the practical guide; Calonico,
Cattaneo, and Titiunik (2014) for robust bias-corrected
inference; Cattaneo, Idrobo, and Titiunik (2019, 2024) for the
two-volume practical guides that are now the field standard.

**Implementations**:
- **`rdrobust`** (R, Stata, Python) — the Cattaneo group's
  reference implementation. Markus uses this by default.
- **`rddensity`** (R, Stata, Python) — for the McCrary
  manipulation test.
- **`rdmulti`** (R, Stata) — for multiple cutoffs or scores.

**What it assumes**: continuity of potential outcomes at the
cutoff. Strong but plausible in settings where assignment is
genuinely determined by a sharp rule and units cannot precisely
manipulate their score.

**Common errors**:
- Using parametric polynomial fits (especially high-order) when
  Gelman and Imbens (2019) showed they're a bad idea — local
  linear with proper bandwidth selection is the default.
- Not testing for manipulation of the running variable.
- Choosing a bandwidth ad hoc when data-driven selectors exist.
- Interpreting the LATE at the cutoff as if it generalized to
  units far from it.

### Regression kink design

**Canonical reference**: Card, Lee, Pei, and Weber (2015).

**Implementation**: `rdrobust` handles the kink design.

---

## Panel data methods

### Static panel: fixed effects, random effects

**Canonical references**: Wooldridge (2010) for the textbook
treatment.

**Implementations**:
- **`linearmodels.panel`** (Python) — `PanelOLS`,
  `RandomEffects`, `BetweenOLS`, `FirstDifferenceOLS`. The
  cleanest Python implementation.
- **`fixest::feols`** (R) and **`pyfixest`** (Python) —
  Berge's high-dimensional fixed effects estimator. The fastest
  option for large panels with many fixed effects, and Markus's
  default.
- **`reghdfe`** (Stata) — the same idea in Stata. Now mostly
  superseded by Stata's native `reghdfe` integration.
- **`plm`** (R) — older R workhorse, more feature-rich for
  RE/Hausman testing but slower.

**What FE assumes**: strict exogeneity conditional on the unit
fixed effect. Anything that varies within unit and correlates
with the regressor and the error is still a problem.

**Common errors**: clustering at the wrong level (cluster at the
level of treatment assignment, not at the smallest available
level); using random effects without engaging with the Hausman
test or its limitations; ignoring serial correlation in the
errors.

### Dynamic panel: Arellano-Bond, Blundell-Bond

**Canonical references**: Arellano and Bond (1991), Arellano
and Bover (1995), Blundell and Bond (1998). Roodman (2009) is
the practical guide.

**Implementations**:
- **`xtabond2`** (Stata, Roodman) — the reference
  implementation. Most published dynamic panel work uses this.
- **`linearmodels.panel.IVGMM`** and related (Python) — basic
  support; less feature-rich than `xtabond2`.
- **`pdynmc`** (R) — the most feature-complete R option.

**What they assume**: instrument validity for the lagged
levels/differences, and no serial correlation in the errors at
the relevant lag (testable via Arellano-Bond AR(2) test).

**Common errors**: instrument proliferation (the number of
instruments grows with T, weakening the Hansen test);
not reporting the AR(1) and AR(2) tests; using system GMM
without justifying the additional moment conditions.

### Long panels: heterogeneous coefficients, common factors

**Canonical references**: Pesaran (2006) for common correlated
effects; Bai (2009) for interactive fixed effects; Chudik and
Pesaran (2015) for dynamic versions.

**Implementations**:
- **`xtmg`** (Stata) for mean group and CCE estimators.
- **`phtt`** (R) for Bai's interactive fixed effects.

---

## Time series

### Stationarity, unit roots, cointegration

**Canonical references**: Dickey and Fuller (1979), Phillips and
Perron (1988), Kwiatkowski et al. (1992) for the basic tests;
Engle and Granger (1987) and Johansen (1991) for cointegration;
Elliott, Rothenberg, and Stock (1996) for DF-GLS.

**Implementations**:
- **`statsmodels.tsa.stattools`** (Python) — `adfuller`,
  `kpss`, `coint`. The standard.
- **`urca`** (R) — more feature-rich, including Johansen
  cointegration tests.
- **`arch.unitroot`** (Python) — Kevin Sheppard's
  implementations, including DF-GLS and PP variants.

**Common errors**: testing for unit roots without accounting
for structural breaks (Perron 1989 showed breaks bias toward
finding unit roots); using ADF without reporting lag selection;
treating failure to reject the null as evidence of a unit root.

### Structural breaks

**Canonical references**: Bai and Perron (1998, 2003) for
multiple structural breaks; Andrews (1993) for unknown
breakpoint tests; Hansen (2001) for the practical perspective.

**Implementations**:
- **`strucchange`** (R) — the reference implementation.
- **`ruptures`** (Python) — change point detection, broader
  in scope than Bai-Perron.
- **`statsmodels`** has limited structural break support; `arch`
  has more.

### Vector autoregressions

**Canonical references**: Sims (1980) for the original; Stock
and Watson (2001) for the practical introduction; Kilian and
Lütkepohl (2017) for the structural identification treatment.

**Implementations**:
- **`statsmodels.tsa.vector_ar`** (Python) — VAR, VECM,
  impulse responses, FEVD. Adequate for most applications.
- **`vars`** (R) — the R reference, more feature-rich.
- **`bvartools`** and **`BVAR`** (R) for Bayesian VARs.

**Common errors**: using Cholesky identification without
engaging with the ordering assumption; reporting orthogonalized
IRFs as if the orthogonalization were neutral; using too many
lags relative to sample size.

### Local projections

**Canonical reference**: Jordà (2005). Plagborg-Møller and Wolf
(2021) on the equivalence (and non-equivalence) with VAR
impulse responses.

**Implementations**:
- **`lpirfs`** (R) — the reference implementation, by Adämmer.
- **`localprojections`** (Python) — newer, less mature.
- For most applications Markus implements LPs directly with
  `pyfixest` or `fixest`, since the method is just a sequence
  of OLS regressions with appropriate SEs.

**Why prefer LPs over VARs**: LPs are more robust to
misspecification at long horizons, and they extend cleanly to
state-dependent IRFs (Auerbach and Gorodnichenko 2012, 2013).
The cost is efficiency loss when the VAR is correctly
specified.

### State space and filtering

**Canonical references**: Durbin and Koopman (2012) for the
textbook; Harvey (1989) for the foundational treatment.

**Implementations**:
- **`statsmodels.tsa.statespace`** (Python) — comprehensive,
  including unobserved components, dynamic factor models, and
  custom state space.
- **`KFAS`** (R) — the R reference.
- **`dlm`** (R) for dynamic linear models.

### Modern time series ML and foundation models

**References**: Makridakis, Spiliotis, and Assimakopoulos
(2018, 2020) on the M4/M5 competitions, where simple methods
beat ML for most macro-relevant tasks. Hewamalage, Bergmeir,
and Bandara (2021) for a fair survey.

**Implementations**:
- **`statsforecast`** (`Nixtla/statsforecast`) — fast classical
  baselines (auto-ARIMA, ETS, Theta). Markus uses this as the
  benchmark before any fancier model.
- **`Chronos`** (`amazon-science/chronos-forecasting`) —
  pretrained foundation model.
- **`uni2ts`** (`SalesforceAIResearch/uni2ts`) — Moirai
  family of time series transformers.

**The honest position**: on small macroeconomic samples,
classical methods are very hard to beat. Foundation models earn
their keep on certain higher-frequency or richer-feature
problems, not by default.

---

## Heterogeneous treatment effects and double machine learning

### Double / debiased machine learning

**Canonical references**: Chernozhukov, Chetverikov, Demirer,
Duflo, Hansen, Newey, and Robins (2018). Chernozhukov,
Newey, and Singh (2022) on debiased estimation more generally.

**Implementations**:
- **`DoubleML`** (Python, R) — the reference implementation,
  by the authors. Supports partially linear, IV, and difference-
  in-differences variants.
- **`EconML`** (`py-why/EconML`, Microsoft) — broader scope,
  includes DML alongside causal forests and meta-learners.

**What it assumes**: the standard ignorability assumption, plus
sufficient regularity for the nuisance estimators to converge
fast enough. Cross-fitting handles the overfitting bias.

**Common errors**: using DML when the question is actually
about identification (DML doesn't solve identification, it
solves a finite-sample bias problem given identification); not
checking that the ML nuisance estimators are reasonable;
treating the resulting standard errors as if they reflected all
sources of uncertainty.

### Causal forests

**Canonical references**: Wager and Athey (2018), Athey, Tibshirani,
and Wager (2019).

**Implementations**:
- **`grf`** (R, generalized random forests) — the reference
  implementation, by the authors.
- **`econml.dml`** has causal forest variants in Python.

**What it estimates**: heterogeneous treatment effects under
the same assumptions as standard causal inference, with the
heterogeneity learned from the data.

### Meta-learners

**Canonical references**: Künzel, Sekhon, Bickel, and Yu
(2019).

**Implementations**: `EconML` and `causalml` (Uber) implement
S-, T-, X-, R-, and DR-learners.

---

## Bayesian methods

**Canonical references**: Gelman, Carlin, Stern, Dunson,
Vehtari, and Rubin (2013); Kruschke (2015) for the gentler
introduction; McElreath (2020) for the most readable.

**Implementations**:
- **`PyMC`** (Python) — the Python reference.
- **`Stan`** with `cmdstanpy` (Python) or `cmdstanr` (R) — the
  most mature, the best documentation, the gold standard for
  serious Bayesian work.
- **`brms`** (R) — the easiest path into Bayesian regression
  modeling, compiles to Stan under the hood.
- **`numpyro`** (Python) — JAX-based, fast for large problems.
- **`bambi`** (Python) — `brms`-like interface for PyMC.

**Use cases in macro and labor**: Bayesian VARs (especially
large BVARs), DSGE estimation, hierarchical models for
heterogeneous parameter estimation across groups, anything
where prior information is genuinely informative and the
researcher can defend it.

**Common errors**: treating the prior as a nuisance rather than
a modeling choice; not checking convergence diagnostics
(R-hat, ESS, divergences); confusing posterior probability with
classical p-value interpretation.

---

## Structural and dynamic models

### DSGE

**Canonical references**: Christiano, Eichenbaum, and Evans
(2005); Smets and Wouters (2007); An and Schorfheide (2007) for
estimation; Fernández-Villaverde, Rubio-Ramírez, and Schorfheide
(2016) for the survey.

**Implementations**:
- **`Dynare`** (Matlab, Octave) — still the field standard for
  DSGE estimation. Most published DSGE work uses Dynare.
- **`MacroModelling.jl`** (`thorek1/MacroModelling.jl`) — the
  most active Julia DSGE toolkit.
- **`gEconpy`** (`jessegrabowski/gEconpy`) — Python DSGE
  toolkit.
- **`Dolo`** — Python DSGE with focus on global solution
  methods.

**The honest position**: DSGE models have well-known limitations
and the literature on them has been contentious for fifteen
years. Markus knows the Romer (2016) critique, the Blanchard
(2018) defense, and the HANK literature's response. He uses
DSGE when the question calls for it and resists when it
doesn't.

### Heterogeneous-agent macro (HANK)

**Canonical references**: Kaplan, Moll, and Violante (2018) for
HANK; Auclert, Bardóczy, Rognlie, and Straub (2021) for the
sequence-space Jacobian method that made HANK tractable.

**Implementations**:
- **`sequence-jacobian`** (`shade-econ/sequence-jacobian`) —
  the reference implementation by the Auclert et al. team. The
  modern way to solve and estimate HANK models.
- **HANK toolkits in Julia and Matlab** circulate informally;
  there is no fully canonical Julia version yet.

### Discrete choice and structural labor

**Canonical references**: Rust (1987) for DDC; Keane and Wolpin
(1997) for life-cycle structural models; Aguirregabiria and
Mira (2010) for the survey.

**Implementations**:
- **`OpenSourceEconomics`** ecosystem (`respy`, `estimagic`,
  `pybobyqa`) — JAX-compatible structural econometrics
  infrastructure.
- **`optimagic`** (formerly `estimagic`) — numerical
  optimization with the diagnostics structural estimation
  actually needs.

---

## Inequality and distribution measurement

### Inequality indices

**Canonical references**: Atkinson (1970) for the Atkinson
index and the social welfare framing; Cowell (2011) for the
textbook treatment of all the standard measures.

**Implementations**:
- **`DescTools::Gini`** and similar in R.
- **`inequality`** (Python) — Gini, Theil, Atkinson, etc.
- **`ineq`** (R) — the R reference for inequality indices.
- For Stata, `ineqdec0`, `ineqdeco`, and `glcurve` from SSC.

**Choices that matter**:
- Pre-tax vs post-tax, market vs disposable income.
- Equivalence scale (per capita, square root, OECD modified).
- Income definition (cash, cash plus in-kind, plus imputed
  rent, plus capital gains, plus undistributed corporate
  earnings).
- Top-coding and how it's handled.

These choices change the answer more than the choice of index
does. Document all of them.

### Top income shares

**Canonical references**: Piketty (2003), Piketty and Saez
(2003), Atkinson, Piketty, and Saez (2011) for the survey,
Piketty, Saez, and Zucman (2018) for distributional national
accounts. Auten and Splinter (2024) for the major recent
critique of the US series.

**Data**: WID.world is the source. The methodology is the DINA
framework documented at the WID site.

**The dispute**: Auten-Splinter and the Piketty-Saez-Zucman
team disagree about how to allocate retained corporate earnings,
how to handle pension wealth, and several other choices.
Different choices give substantially different US top-share
trends. Markus knows both sides and presents the dispute when
the question depends on it.

### Mobility measurement

**Canonical references**: Chetty, Hendren, Kline, and Saez
(2014) on US intergenerational mobility; Corak (2013) for the
Great Gatsby curve; Black and Devereux (2011) for the survey.

**Implementations**: standard regression and rank-rank methods.
The Opportunity Insights team publishes code and data for
their measures.

### Decomposition methods

**Canonical references**: Oaxaca (1973), Blinder (1973) for
the original; DiNardo, Fortin, and Lemieux (1996) for
distributional decomposition; Firpo, Fortin, and Lemieux
(2009, 2018) for RIF regressions.

**Implementations**:
- **`oaxaca`** (Stata, Jann) — the reference.
- **`Blinder-Oaxaca`** in Python via `scikit-learn` extensions
  or hand-coded.
- **`rifreg`** (Stata) for recentered influence function
  regressions.

**Common errors**: applying Oaxaca-Blinder to non-linear
outcomes without a non-linear extension; treating the
"unexplained" component as discrimination without engaging
with the omitted-variable critique (Cain 1986, Altonji and
Blank 1999).

---

## Labor market structural methods

### Search and matching

**Canonical references**: Mortensen and Pissarides (1994);
Pissarides (2000) for the textbook; Shimer (2005) for the
calibration debate; Hagedorn and Manovskii (2008) for the
response.

**Implementations**: typically hand-coded; the models are
small enough that ad-hoc Matlab/Julia/Python implementations
suffice. `QuantEcon` lectures have reference implementations.

### Monopsony and wage-setting

**Canonical references**: Manning (2003) for the textbook;
Card, Cardoso, Heining, and Kline (2018) for the modern AKM
extension; Berger, Herkenhoff, and Mongey (2022) for general
equilibrium monopsony.

**Implementations**:
- **AKM and limited-mobility-bias-corrected versions**:
  `LeaveOutTwoWay` (Matlab, Kline-Saggio-Sølvsten); `pytwoway`
  (Python, Bonhomme-Lamadon-Manresa). The bias correction
  matters a lot in practice.
- For minimum wage and monopsony empirical work, standard
  panel and DiD tools apply; the structural content lives in
  interpretation.

---

## Text and NLP for economics

**Canonical references**: Gentzkow, Kelly, and Taddy (2019)
for the survey of text-as-data in economics; Hansen, McMahon,
and Prat (2018) for an exemplar of text analysis applied to
central bank communications.

**Implementations**:
- **`scikit-learn`** for classical text classification.
- **`spaCy`** for tokenization, NER, and pipeline work.
- **`transformers`** (HuggingFace) for embeddings and modern
  classification.
- **`FinBERT`** (`ProsusAI/finbert`) for financial sentiment;
  widely used in central-bank-communication research.
- **`BERTopic`** for topic modeling; supersedes LDA for most
  applications.
- LLM API calls (OpenAI, Anthropic) for classification and
  extraction at scale, with the caveats from
  `tools-and-integrations.md` about reproducibility and cost.

**Common errors**: using off-the-shelf sentiment scores
designed for product reviews on monetary policy text; treating
LLM classification as deterministic when it isn't; not
documenting the model version, prompt, and date.

---

## Spatial methods

**Canonical references**: Anselin (1988) for the foundational
treatment; LeSage and Pace (2009) for the textbook; Conley
(1999) for spatial standard errors.

**Implementations**:
- **`PySAL`** ecosystem (Python) — comprehensive spatial
  econometrics and spatial statistics.
- **`spdep`** and **`spatialreg`** (R) — the R workhorses.
- **`acreg`** (Stata) for Conley standard errors.
- **`reg2hdfespatial`** (Stata) for high-dimensional FE with
  spatial SEs.

**Common errors**: ignoring spatial autocorrelation in the
errors when units are geographic; using contiguity weights
mechanically without thinking about whether they capture the
relevant spillovers.

---

## A note on Stata

Many of the canonical references in applied microeconomics
were implemented in Stata first, and for some methods (Roodman's
`xtabond2`, Jann's `oaxaca`, the de Chaisemartin-D'Haultfœuille
`DIDmultiplegt`) the Stata version is still the most feature-
complete. Markus's default is Python, but he does not pretend
that Stata doesn't exist, and for replication of published work
he uses whatever the original used.

The `Stata-MCP` server (`hanlulong/stata-mcp`) makes it
practical to drive Stata from a Claude-augmented environment
when the situation calls for it.

---

## Things to add as they're encountered

This file should grow as Markus encounters methods that aren't
yet here, and as the canonical references and implementations
shift over time. The DiD section in particular will need
periodic updates — the literature is still moving.

When adding a new method, the four questions to answer are the
same as the rest of this file: canonical reference, recommended
implementation, what it assumes, and how it gets misused.