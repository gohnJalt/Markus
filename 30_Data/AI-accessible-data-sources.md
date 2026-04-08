# AI-accessible data sources

This is Markus's working knowledge of the data sources relevant to
macroeconomics, labor economics, and the economics of distribution.
It is organized by what you want to know, not by who publishes the
data, because researchers come to data with questions and the
question should drive the source.

For each source, the relevant facts are: what it covers, what its
known quirks are, how to access it (API, MCP server, bulk download,
restricted), and what it is and is not appropriate for. The
quirks matter more than the coverage. A source you understand is
worth more than a bigger source you don't.

The general principle from `Principles.md` Section III applies
throughout: data are constructed, not given. Every series here is
the output of methodological choices made by an institution under
constraints, and those choices show up in the numbers. Markus reads
the documentation before he uses the data, every time.

---

## On the geography of this file

Most major economic data sources are produced by US, European, or
international institutions, and the file reflects that. For
country-specific work outside this orbit — including Turkey, where
this researcher is based — Markus relies on national statistical
offices and central banks directly, and adds entries to this file
as he learns each one's quirks. National sources are almost always
better than international aggregators for the country in question,
because aggregators harmonize and harmonization hides things.

A short note on the Turkish sources at the end of the file. They
will grow as needed.

---

## Macroeconomic time series

### FRED — Federal Reserve Bank of St. Louis

The single most useful source for US and many international macro
time series. ~800,000 series across categories, with a clean API,
good documentation, and an MCP server.

**Coverage**: US macro is comprehensive (GDP, inflation,
employment, interest rates, financial conditions, regional data).
International coverage is real but uneven — FRED redistributes
series from BIS, OECD, World Bank, and national sources, and the
quality is inherited from upstream.

**Access**: REST API, Python clients (`fredapi`, `fedfred`,
`pandas-datareader`), and the FRED MCP server
(`stefanoamorelli/fred-mcp-server`). The MCP server is the
preferred path when Markus is fetching for the researcher.

**Quirks that matter**:
- FRED serves the **latest vintage** by default. If the question
  is about what policymakers knew in real time, this is wrong, and
  Markus needs to use ALFRED (the FRED archival service that
  serves vintages by date).
- Seasonal adjustment is series-specific. SA and NSA series often
  have different IDs that look almost identical. Read the title.
- Many international series on FRED are republished with a lag and
  may not reflect the upstream source's most recent revision.
- The `units` parameter can return levels, changes, percent
  changes, log changes, year-over-year, and so on. Convenient and
  also a footgun — the researcher should usually compute
  transformations themselves so they know what was done.
- Discontinued series remain available but stop updating without
  always being clearly flagged.

**Use for**: any US macro question, as the first stop. Cross-
country macro when convenience matters more than authoritative
sourcing.

**Don't use for**: real-time vintage analysis (use ALFRED or
the OECD Real-Time Database). Country-specific work where the
national source is better.

### ALFRED — Archival FRED

The vintage version of FRED. Every series has a release calendar
and a history of revisions, and ALFRED lets you query "what did
this series look like as of date X."

**Use for**: any analysis of how policymakers responded to data,
forecasting evaluations against real-time data, studies of GDP
revisions, anything where the difference between the first
estimate and the final value is the question.

**Quirks**: not every FRED series has full vintage history.
Coverage is best for headline US series.

### BEA — Bureau of Economic Analysis

The original source for US national accounts (GDP, NIPA tables,
personal income, international transactions, regional GDP, fixed
assets). FRED republishes most of what BEA produces, but BEA's own
interactive tables have detail that FRED's aggregated series hide.

**Access**: BEA API with a free key. No MCP server as of writing.

**Use for**: detailed NIPA decompositions, sectoral breakdowns,
state and metro GDP, the underlying tables behind the headline
numbers.

### BLS — Bureau of Labor Statistics

The source for US labor market data: CPS, CES, JOLTS, unemployment,
CPI, PPI, employment cost index, productivity, occupational
employment.

**Access**: BLS API (rate-limited; a free registered key gives
higher limits). FRED republishes the headline series.

**Quirks that matter**:
- CPS and CES measure employment differently (household vs
  establishment survey). They disagree, sometimes substantially,
  and the disagreement is itself informative — see the literature
  on the post-2020 employment gap. Always know which one you're
  using.
- CPI methodology has changed over time (geometric vs Laspeyres,
  shelter measurement, hedonic adjustments). For long historical
  comparisons this matters and the BLS provides research series
  (CPI-U-RS) that backcast the current methodology.
- JOLTS is a sample survey with meaningful sampling error. Treat
  monthly JOLTS movements with appropriate skepticism.

### OECD — Organisation for Economic Co-operation and Development

The default source for harmonized cross-country data among rich
economies. National accounts, labor statistics, productivity,
trade, prices, household income, education, health, pensions.

**Access**: OECD Data Explorer and SDMX API. No widely used MCP
server as of writing. Bulk downloads are practical for many
datasets.

**Quirks**:
- Harmonization smooths out the very thing you sometimes want to
  study. The OECD's "harmonized unemployment rate" is not the same
  as any country's headline rate.
- Some datasets have substantial publication lag (1–3 years for
  the more detailed cross-country series).
- The OECD Real-Time Database provides vintage data for major
  macro aggregates across member countries — useful and
  underused.

**Use for**: cross-country comparison among OECD members.

### Eurostat

The Eurostat database is the European counterpart to BEA + BLS +
Census combined, covering EU and EEA countries.

**Access**: Eurostat API (REST and SDMX), bulk downloads, the
`eurostat` Python and R packages.

**Quirks**: country coverage varies by series and over time
(accession countries enter at different dates). Some series are
flagged provisional, estimated, or break-in-series — always check
the flag column.

### IMF — International Monetary Fund

WEO (forecasts and historical macro), IFS (International Financial
Statistics), BOP (balance of payments), DOTS (Direction of Trade),
GFS (Government Finance Statistics).

**Access**: IMF Data API, MCP server (`c-cf/imf-data-mcp`), and
the `pandasdmx` Python library.

**Use for**: cross-country macro for non-OECD countries, balance
of payments, official IMF forecasts. WEO is the canonical source
for "what did the IMF think" but Markus does not treat WEO
forecasts as ground truth — they are themselves an object of
study.

### BIS — Bank for International Settlements

Banking statistics, debt securities, credit, property prices,
effective exchange rates, central bank policy rates.

**Access**: BIS data portal, bulk downloads. Available
indirectly through some of the aggregator MCP servers.

**Use for**: cross-country credit and debt, real and effective
exchange rates, the BIS property price database (one of the few
serious sources for international house prices).

### World Bank — WDI and beyond

The World Development Indicators is the default cross-country
development dataset. The World Bank also publishes the Global
Financial Development Database, the Enterprise Surveys, the
Human Capital Index, and other thematic datasets.

**Access**: World Bank API, the `wbdata` and `pandas-datareader`
Python packages, the World Bank MCP server
(`anshumax/world_bank_mcp_server`).

**Quirks**:
- WDI is an aggregator of national sources, so quality varies
  enormously by country. Sub-Saharan African series in particular
  have well-documented quality problems for some indicators
  (Jerven's *Poor Numbers* is the relevant reference).
- "Country" is not a stable concept — the WDI handles
  USSR/Russia, Yugoslavia/successors, Sudan/South Sudan, and
  similar in ways that affect long time series.
- GDP per capita comes in multiple variants (current USD,
  constant USD, PPP current, PPP constant) and they tell
  different stories. Always know which one you're using.

---

## Inequality and distribution

This is the section the awesome list misses, and it matters most
for distribution work. The mainstream econ-AI tooling is built
around macro aggregates and microdata for treatment effects, not
around the distributional sources that actually move the
inequality literature.

### WID.world — World Inequality Database

The Piketty/Saez/Zucman/Chancel project. Top income shares, top
wealth shares, factor shares, distributional national accounts
(DINA) for an increasing number of countries. The most ambitious
attempt to construct internationally comparable inequality series
combining tax data, surveys, and national accounts.

**Access**: WID website, bulk download, the `wid` R package, and
direct CSV downloads. No MCP server as of writing.

**Quirks**:
- WID series rest on strong methodological choices (the DINA
  framework). These are defensible but contested — read Auten and
  Splinter's critique of the US series before treating WID
  numbers as ground truth, and read the WID team's response, and
  then form your own view.
- Coverage varies enormously across countries and over time. The
  US, France, and the UK have rich long series; many countries
  have a few decades of recent data only.
- Methodology is updated periodically and historical numbers
  change. Cite the version.

**Use for**: long-run top shares, wealth concentration, factor
shares, distributional national accounts. This is the canonical
source and Markus reaches for it first on any distribution
question.

### LIS — Luxembourg Income Study

The reference cross-country microdata source for income
distribution. Harmonized household income microdata for ~50
countries, typically every few years.

**Access**: LISSY remote system. You write code (Stata, R, SAS,
SPSS), submit it, and get the output back — no direct microdata
access for confidentiality reasons. There is no API and no MCP
server because the access model fundamentally doesn't allow it.

**Quirks**:
- LISSY's remote-execution model is the right answer for
  privacy and the wrong answer for iterative work. Plan analyses
  before submitting.
- The LIS Key Figures site provides pre-computed inequality
  measures (Gini, percentiles, poverty rates) without requiring
  LISSY access. Use it for quick lookups.
- Income concept harmonization across countries involves real
  judgment calls, especially around imputed rents, in-kind
  transfers, and capital income. Read the LIS technical
  documentation.

**Use for**: cross-country comparison of household income
distributions, comparative welfare state analysis, anything where
you need actually-comparable inequality measures across countries.

### OECD Income Distribution Database (IDD)

OECD's harmonized inequality and poverty measures across member
countries. Built from national household surveys with OECD
harmonization.

**Access**: OECD Data Explorer, OECD API.

**Use for**: cross-OECD inequality comparisons when LISSY access
is not available or the needed measure is in IDD already.

**Caveat**: harmonization is real but imperfect, and the IDD's
numbers differ from WID's and from national sources for the same
country and year. The differences are methodological, not
errors, and they are themselves informative.

### Chartbook of Economic Inequality (Atkinson, Hasell, Morelli, Roser)

Long-run inequality series for ~25 countries assembled from
historical sources. Less methodologically uniform than WID but
covers a longer span for some countries.

**Access**: web download, plus the underlying data is on Our
World in Data.

### EU-SILC — European Union Statistics on Income and Living Conditions

The EU's harmonized household income and living conditions
survey. The reference source for European inequality and
poverty.

**Access**: Eurostat for aggregated indicators. Microdata access
requires application to Eurostat under their research access
scheme.

### National sources for distribution

For any country-specific distribution question, the national
source is usually better than the international ones:

- **United States**: CPS ASEC for income, SCF for wealth, IRS
  SOI for tax-based top incomes, PSID for longitudinal income
  and wealth, ACS for fine geographic detail.
- **United Kingdom**: HBAI from DWP, ONS Wealth and Assets
  Survey, the Resolution Foundation's analysis files.
- **France**: INSEE ERFS, INSEE wealth surveys.
- **Turkey**: see the Turkish sources section below.

---

## Labor market microdata

### CPS — Current Population Survey

The US monthly labor force survey. The basic monthly file for
employment status; the ASEC supplement for annual income; the
ORG (Outgoing Rotation Group) files for wage analysis; the
displaced workers, contingent workers, and other supplements
for specialized questions.

**Access**: IPUMS-CPS for the cleanest harmonized version. The
NBER also distributes CPS files. BLS publishes the underlying
data but IPUMS does the harmonization across time that makes the
data actually usable.

**Quirks**:
- Top-coding of income variables matters for inequality work and
  has changed over time. IPUMS documents the changes; read them.
- The 2014 ASEC redesign created a structural break in the
  income series. There are bridge files; use them.
- CPS measures earnings of "usual hours" workers, with edits and
  imputations that affect the lower tail in particular.
- Sample weights are essential. Replicate weights are available
  for variance estimation.

### ACS — American Community Survey

The annual successor to the long-form Census, with a 1% sample
each year and 5-year combined files for fine geography.

**Access**: IPUMS-USA, Census API, the Census MCP server.

**Use for**: anything requiring fine geographic or demographic
disaggregation of US labor market or income data.

### IPUMS — Integrated Public Use Microdata Series

Not a dataset but the most important data infrastructure project
in social science. IPUMS harmonizes census and survey microdata
across time and across countries (IPUMS International), making
otherwise incompatible files comparable.

**Access**: IPUMS website, free with registration. No MCP server,
but the workflow (extract builder → download → analyze locally)
is straightforward.

**Use for**: any project using historical US census data, any
cross-country project using harmonized census microdata, any
longitudinal work on the CPS, ACS, or international census files.

### Administrative tax data and linked employer-employee data

The frontier of labor and distribution research increasingly uses
administrative data that is restricted-access by design. These
sources do not have APIs and will not have MCP servers, because
the access model is fundamentally different — the researcher
applies, signs agreements, and works in a controlled environment.

Markus knows these exist and can discuss what they contain and
what kinds of questions they answer, but cannot fetch them:

- **US**: IRS administrative tax data via the IRS-SOI program,
  LEHD (Longitudinal Employer-Household Dynamics) at the Census,
  the SSA Continuous Work History Sample.
- **Scandinavia**: full-population administrative registers in
  Denmark, Sweden, Norway, Finland — the gold standard for
  longitudinal work.
- **Germany**: IAB linked employer-employee data via FDZ.
- **France**: DADS, EDP, échantillons démographiques.
- **Italy**: INPS employer-employee data.
- **Turkey**: SGK administrative employment data, accessed
  through application to the relevant agency.

For any project that would benefit from administrative data,
Markus's role is to help the researcher think about whether the
research question justifies the access process.

### Job postings and online labor market data

A newer category of "alternative data" for labor research.

- **Lightcast** (formerly Burning Glass) — large-scale job
  postings data with skills taxonomy. Used in research on skill
  demand, returns to skills, geographic labor market dynamics.
  Academic access through institutional licenses.
- **Revelio Labs** — employment records constructed from public
  online profiles. Academic access via WRDS at many institutions.
- **LinkedIn Economic Graph** — limited research access through
  LinkedIn's data-for-good program.

Markus is appropriately cautious about online labor market data:
selection into the data is non-random (which workers and which
jobs appear online), the underlying definitions can change
without notice, and the time series are short. These sources
complement traditional labor data; they do not replace it.

---

## Trade, firms, and production

### UN Comtrade

The UN's international trade database. Bilateral merchandise
trade by HS code for essentially every country.

**Access**: Comtrade API (rate-limited free tier; paid tier for
heavy use). The CEPII BACI database is a cleaned and
reconciled version of Comtrade that fixes mirror inconsistencies
and is preferred for serious trade work.

**Quirks**: Comtrade's reported and mirrored flows often
disagree (importer-reported and exporter-reported should
match but don't). BACI reconciles them. Comtrade also has
reporting gaps for many country-years.

### CEPII

CEPII (the French international economics research center)
publishes BACI (cleaned trade), GeoDist (bilateral distance and
geography), Trade Unit Values, and several other useful
international economics datasets.

### Penn World Table (PWT)

The standard source for cross-country comparisons of real GDP,
productivity, and capital. Currently maintained at the
University of Groningen.

**Access**: direct download (Stata, Excel, CSV).

**Use for**: any cross-country growth, productivity, or
development accounting exercise. PWT is the source the
literature actually uses.

**Quirks**: PWT versions differ in non-trivial ways. Cite the
version. The methodology for constructing PPPs is contested
(see the debates around ICP rounds).

### Compustat and CRSP

The standard sources for US firm financials and stock prices,
respectively. Used in any corporate finance, productivity, or
firm-level macro work.

**Access**: WRDS (Wharton Research Data Services) for academic
users with institutional access.

### Orbis

Bureau van Dijk's database of firm-level financial information
across countries. Used in cross-country firm-level work,
including the recent literature on markups (De Loecker, Eeckhout,
Unger).

**Caveat**: Orbis coverage varies enormously by country and over
time, and the literature has noted that this affects results.
The Bajgar et al. (2020) paper on Orbis coverage is required
reading before using it for cross-country comparisons.

---

## Macro databases for the long run

### Global Macro Database

A recent project (`globalmacrodata.com`) compiling 241
countries from 1086 to 2024 from 27 contemporary and 84
historical sources. Useful as a starting point for very long-run
macro questions.

**Caveat**: any single source covering a millennium is doing a
lot of harmonization across very different underlying records.
Markus uses it as a directory to underlying sources, not as a
final authority.

### Maddison Project

Long-run GDP per capita series from Angus Maddison and
successors. The reference for very long-run growth comparisons.

**Access**: Maddison Project Database, free download.

**Use for**: anything that needs GDP per capita before the 20th
century. Cite the version.

### Jordà-Schularick-Taylor Macrohistory Database

Long-run macro and financial data for 18 advanced economies
covering 1870 onward. Includes credit, money, asset prices, and
the variables needed to study financial cycles.

**Access**: free download from the macrohistory.net website.

**Use for**: long-run financial cycle work, credit-and-crisis
literature, anything in the Schularick-Taylor research program.

---

## Central bank and monetary policy data

### Federal Reserve

FRED is the public face. Beyond FRED, the Fed publishes
detailed data through individual reserve banks (especially
Dallas, Atlanta, NY for nowcasting and financial conditions),
H.8 (bank assets and liabilities), Z.1 (financial accounts of
the US), and the Senior Loan Officer Survey.

The **Survey of Consumer Finances** is the canonical US wealth
microdata source, published triennially. SCF microdata is
free; the design is complex and the documentation is essential
reading.

### ECB Statistical Data Warehouse

The European counterpart to FRED. SDMX-based access. The ECB
also publishes the Household Finance and Consumption Survey
(HFCS), Europe's answer to the SCF.

### Bank of England, Bank of Japan, etc.

National central banks publish increasingly rich data through
their own portals. For country-specific monetary policy
research, the national central bank is usually the source of
record.

---

## Policy text and qualitative sources

### Central bank communications

For any project on monetary policy text — sentiment, topic
modeling, narrative identification — the source is the central
bank's own publications: FOMC statements and minutes, ECB press
conferences, Inflation Reports, monetary policy reports.

These are typically available as PDFs or HTML on the bank's
website. The Marker and Docling tools (see
`tools-and-integrations.md`) handle the conversion to clean
text for analysis.

### Legislative and regulatory text

- **US Congress**: the Library of Congress and govinfo.gov for
  bills, the Federal Register for regulations.
- **EU**: EUR-Lex for EU law and regulations.
- **Plural Policy** and similar tools provide AI-assisted
  search and version comparison.

### Newspaper archives

- **Factiva**, **ProQuest Historical Newspapers**, **Nexis Uni**
  for historical newspaper text. Institutional access required.
- **GDELT** for a free, large-scale news event database
  (with caveats about coverage and accuracy).

---

## Turkish sources (initial set, will grow)

For the researcher's country-specific work. These are the
default sources for Turkish macro, labor, and distribution
questions, and the international aggregators should not be used
when these are available.

- **TÜİK / TurkStat** — the Turkish Statistical Institute.
  National accounts, labor force survey (Hanehalkı İşgücü
  Anketi / HLFS), CPI, household budget survey, income and
  living conditions survey. The HBS and SILC are the primary
  microdata sources for Turkish income and consumption
  distribution.
- **TCMB / CBRT** — the Central Bank of the Republic of Türkiye.
  Monetary aggregates, interest rates, exchange rates, balance
  of payments, EVDS (Electronic Data Delivery System) is the
  main interface. EVDS has a REST API and a Python client.
- **SGK** — Social Security Institution. Administrative
  employment and earnings data, accessed through application.
  The reference source for any serious Turkish labor question
  that needs more than HLFS gives you.
- **BDDK** — Banking Regulation and Supervision Agency.
  Banking sector data.

Quirks of Turkish data that Markus should remember:
- TÜİK has a contested history of methodological revisions,
  particularly around CPI and unemployment, and the question
  of whether headline series accurately reflect underlying
  conditions has been a matter of public debate. Markus is
  careful to note when a result depends on a series whose
  methodology has been questioned, without taking a political
  position on the disputes themselves.
- Turkish series often have structural breaks at currency
  redenomination (2005), methodology changes, and base-year
  updates. Always check for breaks before estimating on long
  series.
- The Turkish lira's high inflation environment makes the
  real-vs-nominal distinction even more important than usual.
  Working in real terms is essentially mandatory for any
  series spanning more than a year or two.

This section should grow as the researcher uses each source and
finds its rough edges.

---

## A note on aggregator MCP servers

Several MCP servers (OpenEcon, TAM-MCP) offer "natural language
access to many sources at once." These are convenient and Markus
will use them when convenience matters. But:

- Aggregators inherit the quirks of their upstream sources
  without always exposing them. The user-facing convenience can
  obscure exactly the methodological details that matter.
- Aggregators sometimes harmonize across sources in ways the
  underlying agencies would not endorse.
- For any number that goes into a paper or a serious analysis,
  Markus verifies against the source of record, not the
  aggregator.

Aggregators are good for exploration and draft analysis. They
are not the source of record.

---

## Things to add as they're encountered

This file should grow with use. When Markus encounters a source
that isn't here, or learns a new quirk about one that is, the
researcher should add it. The value of the file is in the
accumulated tacit knowledge that institutional documentation
doesn't capture.

A useful prompt for adding new sources: not "what does it
contain?" but "what would I want to have known before I trusted
a number from it?"