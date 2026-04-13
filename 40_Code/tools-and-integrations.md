# Tools and integrations

This is Markus's working knowledge of the AI-and-code tooling
landscape for empirical economics. It is organized by task, not by
vendor, because the question is always "what do I reach for to do
X," not "what does company Y sell."

The list is opinionated and incomplete by design. Inclusion here
means Markus has reason to think the tool is worth using for the
kinds of work this researcher does — macro, labor, distribution,
applied econometrics. Exclusion means either Markus doesn't know it,
doesn't trust it yet, or thinks something else is better. Both kinds
of judgment should be revisited as the landscape changes.

A useful general-purpose source for what's out there is the
`hanlulong/awesome-ai-for-economists` curated list on GitHub, which
Markus treats as a directory to consult, not a canon to follow.

---

## Data access via MCP servers

MCP (Model Context Protocol) servers let Claude pull data directly
rather than waiting for the researcher to paste it. For a research
assistant this is the highest-leverage category of tool, because it
collapses the loop between question and evidence.

When the researcher sets one of these up in the Claude client,
Markus should use it preferentially over asking for the data
manually.

- **FRED MCP server** (`stefanoamorelli/fred-mcp-server`) — full
  access to the 800,000+ FRED time series, with date filtering and
  frequency adjustment. The default for any US macro time series.
  Always check vintage and revision status; FRED serves the latest
  vintage by default.
- **World Bank MCP server** (`anshumax/world_bank_mcp_server`) —
  WDI development indicators across countries. The default for
  cross-country macro and development questions.
- **IMF Data MCP server** (`c-cf/imf-data-mcp`) — WEO forecasts,
  balance of payments, IFS series. Useful when FRED and the World
  Bank don't carry what you need.
- **Census MCP server** — natural-language queries against US
  Census demographic data. Useful for labor and distribution work
  on US microdata aggregates.
- **OpenEcon Data** (`hanlulong/openecon-data`) — a unified
  interface across FRED, World Bank, IMF, Comtrade, Eurostat, BIS
  and others. Convenient as a single front door, but Markus should
  verify the underlying source for any number that goes into a
  paper rather than trusting an aggregator.
- **TAM MCP server** (`gvaibhav/TAM-MCP-Server`) — eight sources
  in one server (Alpha Vantage, BLS, Census, FRED, IMF, Nasdaq,
  OECD, World Bank). Same caveat as above: aggregators are
  convenient but not the source of record.

When using any MCP-served data, Markus follows the data principles
in [[`Principles.md`]] Section III: confirm units, frequency, vintage,
SA/NSA, and coverage before using a number. The server will not do
this for him.

For data sources without good MCP support — WID.world, LIS, the
LISSY remote system, restricted-access microdata — Markus uses the
appropriate native API or asks the researcher to download and
share.

---

## Causal inference libraries

These are the libraries Markus reaches for when the principles in
Section II demand a specific estimator. The choice of library is
secondary to the choice of identification strategy, but the right
library makes the right strategy fast and the wrong library makes
it painful or wrong.

### Difference-in-differences

The DiD literature went through a revolution between roughly 2018
and 2023, and the implications for applied work are still
propagating. Vanilla two-way fixed effects with staggered adoption
can produce nonsense under heterogeneous treatment effects, and
Markus uses heterogeneity-robust estimators by default for any
staggered design.

- **`fixest`** (R) and **`pyfixest`** (Python) — Laurent Berge's
  fast fixed-effects estimator. Default for OLS with high-
  dimensional fixed effects and clustered SEs.
- **`did`** (R, Callaway and Sant'Anna) — group-time average
  treatment effects, the cleanest implementation of the CS
  estimator.
- **`DIDmultiplegt`** / **`DIDmultiplegtDYN`** (R/Stata, de
  Chaisemartin and D'Haultfœuille) — for designs with multiple
  treatments or treatment switching.
- **`eventstudyinteract`** (Stata, Sun and Abraham) — for event
  studies with heterogeneous treatment effects.
- **`moderndid`** (`jordandeklerk/moderndid`) — GPU-accelerated
  modern DiD with staggered adoption and event studies.
- **`contdid`** (`bcallaway11/contdid`) — DiD with continuous
  treatment, by Brantly Callaway. Use when treatment intensity
  varies rather than just on/off.

### Heterogeneous treatment effects and double ML

- **EconML** (`py-why/EconML`, Microsoft) — heterogeneous treatment
  effects via Double ML, causal forests, meta-learners. The
  reference Python implementation.
- **DoubleML** (`docs.doubleml.org`, Chernozhukov et al.) — Python
  and R implementations of the original Double ML framework. Use
  when the question is partially-linear and the nuisance functions
  are high-dimensional.
- **DoWhy** (`py-why/dowhy`) — end-to-end pipeline (model →
  identify → estimate → refute). Useful as a discipline for forcing
  explicit identification, even when the final estimator comes
  from elsewhere.
- **CausalML** (`uber/causalml`, Uber) — uplift modeling and meta-
  learners. More common in industry than in academic econ but
  worth knowing.
- **`causal-learn`** (`py-why/causal-learn`) — causal discovery
  algorithms (PC, FCI, GES). Markus is appropriately skeptical of
  causal discovery as a substitute for design, but it can be
  useful for hypothesis generation.

### Synthetic control and matching

- **`Synth`** (R/Stata, Abadie et al.) — original synthetic
  control.
- **`augsynth`** (R, Ben-Michael, Feller, Rothstein) — augmented
  synthetic control, the modern default.
- **`scul`** (R) and **`pysyncon`** (Python) — alternative
  implementations.
- **`MatchIt`** (R) — propensity score and other matching
  methods.

### Panel and time series

- **`linearmodels`** (Python) — panel methods (FE, RE, BE, FD),
  IV, and system estimators in Python.
- **`statsmodels`** (Python) — VAR, VECM, state space models,
  ARIMA, structural breaks. The default Python time series
  toolkit.
- **`arch`** (Python) — GARCH and volatility models.
- **`local-projections`** — for impulse response estimation à la
  Jordà. Available in R (`lpirfs`) and as ad-hoc Python
  implementations.

---

## Forecasting and nowcasting

For pure forecasting tasks (as opposed to causal estimation),
different tools apply.

- **`statsforecast`** (`Nixtla/statsforecast`) — fast classical
  forecasting (AR, ARIMA, ETS, Theta) at scale. Use as the
  baseline against which any fancier model must justify itself.
- **`Chronos`** (`amazon-science/chronos-forecasting`) — Amazon's
  pretrained foundation model for time series. Useful for quick
  zero-shot forecasts; verify against simpler baselines.
- **`uni2ts`** (`SalesforceAIResearch/uni2ts`) — unified time
  series transformer models from Salesforce.
- **`pmdarima`** (Python) — auto-ARIMA and related classical
  methods.
- **Now-Casting.com** — real-time GDP nowcasts for major
  economies. Useful as a benchmark when building your own
  nowcast, not as a substitute for understanding it.

A general principle: in forecasting, classical methods
(`statsforecast`, `pmdarima`) are very hard to beat on small
macroeconomic samples. Markus uses ML and foundation models when
they earn their keep, not by default.

---

## DSGE, structural, and computational macro

- **`MacroModelling.jl`** (`thorek1/MacroModelling.jl`) — Julia
  DSGE toolkit with first- and higher-order perturbation.
- **`gEconpy`** (`jessegrabowski/gEconpy`) — Python DSGE toolkit.
- **`OpenSourceEconomics`** ecosystem — JAX-compatible DC-EGM,
  Kalman filters, and templates for structural economics.
- **`optimagic`** (formerly `estimagic`) — numerical optimization
  with the diagnostics applied econometricians actually need.
- **HANK and heterogeneous-agent toolkits** — `sequence-jacobian`
  (Auclert, Bardóczy, Rognlie, Straub) for HANK in Python is the
  current reference; not in the awesome list but the relevant
  tool.

---

## Document processing and data extraction

Useful for scraping policy documents, central bank statements,
historical sources, and PDFs of papers and tables.

- **Docling** (`docling-project/docling`) — IBM's open-source
  document parser. Strong table extraction.
- **Marker** (`datalab-to/marker`) — fast PDF-to-Markdown with
  multi-page table merging. Good for batch processing.
- **Mistral OCR** — paid API; useful for cursive, complex tables,
  and historical documents at low DPI.
- **Transkribus** — handwritten historical document transcription.
  The tool of choice for archival economic history.

For text mining of central bank communications:

- **FinBERT** (`ProsusAI/finbert`) — fine-tuned BERT for financial
  sentiment. Widely used in the central-bank-communication
  literature.
- General-purpose LLMs via API are now competitive with FinBERT
  for sentiment classification of policy text, but at higher cost
  and with reproducibility concerns. Document the model version.

---

## Notebooks, IDEs, and workflow

- **Jupyter + JupyterLab** — the default for exploratory work.
- **`marimo`** (`marimo-team/marimo`) — reactive notebooks stored
  as plain `.py` files. Solves the worst reproducibility and
  diff problems of Jupyter. Worth trying.
- **Positron IDE** — Posit's next-gen IDE with native R and
  Python. Worth watching.
- **VS Code with Stata-MCP** — for projects that require Stata
  (replications, certain panel methods). Allows Markus to drive
  Stata sessions from a Claude or Cursor environment.
- **Quarto** — for reproducible reports that compile to PDF, HTML,
  or Word from a single source. Strongly preferred over raw
  Jupyter for anything that will become a paper.

---

## What Markus does NOT use by default

A few categories of tool that appear in the awesome list but that
Markus is appropriately cautious about:

- **End-to-end "AI research agents"** (Julius AI, CausalAgent,
  Dexter, etc.) — these promise to take a dataset and produce an
  analysis. They are sometimes useful for first passes but they
  hide the choices that matter. Markus does not delegate
  identification to a black box.
- **LLM-based forecasting of political and economic events**
  (e.g., `emerging-trajectories`) — interesting research direction,
  not yet a tool to rely on.
- **Synthetic data generators** (Gretel, MOSTLY AI, SDV) — useful
  for privacy-preserving demos and pedagogy, but Markus is
  cautious about using synthetic data in any analysis whose
  conclusions are meant to apply to the real world.
- **AI-moderated survey tools** (Outset, Conveo, TheySaid) —
  interesting for some research designs, but raise serious
  questions about response quality and the recent Nature
  reporting on AI infiltrating online surveys is a real concern.
- **Vendor-specific causal AI platforms** without open
  implementations — Markus prefers libraries he can read.

These are not blanket rejections. They are defaults that the
researcher can override for a specific purpose by saying so.

---

## Things to add as they're encountered

This file should grow. When Markus uses a tool that turns out to
be valuable, or learns of one that fills a gap, the researcher
should add it here with a one-line note on what it's good for and
what its limitations are. The value of the file is in the curation,
not the count.