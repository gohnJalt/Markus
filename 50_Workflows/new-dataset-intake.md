# New dataset intake

This is the procedure Markus follows the first time he encounters
a dataset. It is the cheapest insurance in applied research: most
of the catastrophic errors that ruin papers are not modeling
errors but data errors made early and propagated silently into
everything downstream. Forty minutes of careful intake at the
start of a project saves days of debugging at the end.

The principle from `Principles.md` Section III governs this file:
data are not given, they are constructed, and knowing the
construction is part of knowing the data. The intake procedure is
how Markus turns "I have a file" into "I understand what is in
this file and what it is and is not appropriate for."

The procedure has eleven steps. The deliverable is a **dataset
card**: a short markdown document, saved alongside the data, that
captures everything the next person to use the data (including
the researcher in six months) needs to know. The card is not
optional. A dataset without a card is not yet ready to be used.

---

## When this workflow runs

Markus runs the full intake procedure when:

- A dataset arrives that he has not seen before in this project.
- A dataset he has used before has been updated, revised, or
  re-released. New vintages get new intake.
- A dataset is being used for a purpose meaningfully different
  from the one it was inspected for. Intake done for a
  descriptive question may not cover what a causal question
  needs.

He runs an abbreviated version when:

- A dataset is a familiar source the researcher uses constantly
  (e.g., the latest FRED pull of a series the researcher has
  used a hundred times). Even then, the units, vintage, and
  date range get checked.

He does not skip intake entirely on the grounds that "I know
this data." That sentence is where errors live.

---

## Step 1 — Provenance

Before opening the file, Markus records where it came from.
Provenance is the most important single piece of metadata and
the one most often missing from older project folders.

The provenance record answers:

- **Source institution.** Not "the internet" but "BLS, Current
  Population Survey, IPUMS-harmonized version."
- **Specific source URL or API call or database name.** A real
  link, not a description. If the source is an internal
  database or a colleague, name them.
- **Access date.** When the file was downloaded or pulled. This
  matters because data revisions are real and the same query
  on the same series at different times can return different
  numbers.
- **Vintage.** Which version of the data this is. For revised
  series, the vintage is part of the identity of the file.
  "FRED GDPC1 pulled 2026-04-08" is a complete identifier;
  "GDPC1" is not.
- **License and access conditions.** Public domain, restricted,
  embargoed, requires attribution? This determines what can be
  shared, published, and reproduced.
- **Citation.** The form the source itself recommends, if
  available. Codify it now so the paper draft doesn't have to
  reconstruct it later.

If the researcher passed Markus a file without provenance,
Markus asks. He does not guess and proceed with an unknown
source.

---

## Step 2 — Read the documentation

Before any code touches the file, Markus reads the
documentation. This is the single most undervalued step in
applied work and the one that catches the most variable-meaning
errors.

The documentation is whatever the source provides:

- A codebook (variable definitions, value labels, missing-value
  conventions).
- A user guide (sample design, sampling weights, methodology).
- Technical documentation (estimation methods, imputation
  procedures, top-coding rules).
- Release notes (what changed in this version vs previous).
- Methodology papers (the underlying construction of the
  variables, especially for survey-based and constructed
  series).

For major sources, this documentation can run to hundreds of
pages. Markus does not read all of it the first time. He reads:

1. The release notes for the current version, in full.
2. The variable definitions for every variable he will use, in
   full. This includes derived variables he might compute, not
   just the ones in the file as it sits.
3. The methodology section relevant to his use case. For wage
   data, the imputation rules and top-coding conventions. For
   employment, the difference between household and
   establishment measurements. For prices, the basket and the
   chaining method.
4. The known issues or caveats section, if one exists. This is
   where the source itself flags the things you should be
   careful about, and it is gold.

Markus surfaces the relevant findings from the documentation in
the dataset card. "CPS ASEC top-codes wages at the 99th
percentile and uses bracket means for top-coded values starting
in 2011" is the kind of fact that needs to be in the card and
needs to be known before any wage analysis runs.

When the documentation is unclear or absent — which happens
more often than it should — Markus says so in the card and
flags the question for the researcher to investigate or
escalate.

---

## Step 3 — Shape, types, and memory

Now Markus opens the file. The first questions are
mechanical:

- **Number of rows and columns.**
- **Column names.** Are they meaningful? Standardized? Are
  there obvious naming conflicts (two columns with the same
  meaning under different names)?
- **Data types of each column.** Numeric, string, datetime,
  boolean, categorical. Are they what they should be? A
  numeric variable stored as string is a common upstream
  artifact and a common source of silent errors.
- **Memory footprint.** Will this fit in memory at full
  resolution, or does Markus need a chunked workflow?
- **File format.** CSV, parquet, Stata, SAS, SPSS, JSON,
  Excel. Each has its own quirks. Excel files deserve special
  suspicion because Excel silently transforms data
  (gene names becoming dates is the famous case; date formats
  silently changing across regional settings is the common one).

A skeleton in Python:

```python
import pandas as pd

df = pd.read_csv("path/to/file.csv")  # or read_parquet, read_stata, etc.

print(f"Shape: {df.shape}")
print(f"Memory: {df.memory_usage(deep=True).sum() / 1e6:.1f} MB")
print("\nColumn types:")
print(df.dtypes)
print("\nFirst few rows:")
print(df.head())
```

If any column has an unexpected type, Markus investigates
before proceeding. A "numeric" column with type `object` usually
means there are non-numeric values mixed in (often missing-
value codes that didn't get parsed, or thousands separators in
a CSV).

---

## Step 4 — Keys and uniqueness

What identifies a row? Every dataset has an answer, even if it
isn't documented, and Markus finds it before he uses the data.

For each candidate primary key:

- **Is it unique?** Run the uniqueness check explicitly. If
  the primary key isn't unique, that's either a bug, a
  documentation error, or evidence that the file is at a
  different unit of observation than the researcher thinks.
- **Are there nulls in the key?** A null in a primary key is
  almost always an error.
- **Is it stable across time?** For panel data, person IDs and
  firm IDs sometimes get reassigned across waves. The
  documentation should say; if it doesn't, this is a question
  for the source.

For panel data specifically, Markus identifies the **unit of
observation** explicitly. Is each row a person-month, a
person-year, a firm-quarter, a country-year, an
establishment-occupation cell? The unit of observation is part
of the identity of the dataset and gets recorded in the card.

```python
# Uniqueness check
key_candidates = ["person_id", "year"]
n_total = len(df)
n_unique = df[key_candidates].drop_duplicates().shape[0]
print(f"Rows: {n_total}, Unique on {key_candidates}: {n_unique}")
print(f"Duplicates: {n_total - n_unique}")

# Nulls in key
print(df[key_candidates].isnull().sum())
```

If the keys don't behave as expected, Markus stops and surfaces
the issue. This is not something to "clean up" by dropping
duplicates without understanding why they exist. Duplicates may
indicate a real underlying structure (multiple jobs per worker,
multiple addresses per household) that the analysis needs to
handle correctly.

---

## Step 5 — Coverage

Coverage is the answer to "what's actually in here, and what
isn't?" Markus inspects coverage along every dimension that
matters for the intended use:

**Temporal coverage.** What is the date range? Is it complete,
or are there gaps? For monthly or quarterly data, are all
periods present? Plot the count of observations by period — a
visual gap is hard to miss.

**Geographic coverage.** What countries, states, regions,
counties are present? Is the coverage balanced over time? A
country dropping in or out of an international panel can drive
results entirely.

**Entity coverage.** What firms, individuals, sectors? For
panel data, what fraction of entities are present in every
period (balanced) versus only some periods (unbalanced)? The
balance structure matters for many estimators.

**Sample size by relevant subgroup.** If the analysis will
look at women aged 25-34 in manufacturing, how many such
observations are there? Many failed analyses are failures of
sample size in the relevant cell, not failures of the overall
N.

```python
# Temporal coverage
print(df.groupby("year").size())

# Geographic coverage
print(df["country"].value_counts())

# Cell-level sample sizes for relevant cuts
print(df.groupby(["year", "country"]).size().unstack().head())
```

If coverage is uneven in a way that matters, Markus records it
in the card. "Country X drops out of the panel in 2014 due to
methodology change" is a fact downstream analyses need to
know.

---

## Step 6 — Units, frequencies, and deflators

This is the step from `Principles.md` Section III, and it is
the step Markus is most religious about. For every numeric
variable he will use, Markus confirms:

**The unit of measurement.** Dollars, euros, lira, percentage
points, percent of GDP, persons, hours, kilowatt-hours, log
points. The unit should be in the codebook. If it isn't,
Markus infers it from context and *flags the inference* in the
card so it can be verified.

**Real or nominal.** For any monetary variable, this is non-
negotiable. Nominal series compared across years are
meaningless except as raw historical record. The deflator —
which one, what base year, what methodology — is part of the
identity of a real series.

**Stock or flow.** GDP is a flow (per unit time). Capital is a
stock (at a point in time). Wages can be either. Conflating
stocks and flows produces nonsense quietly.

**Per capita, per worker, or aggregate.** Three different
series, three different stories. The literature is full of
papers that tell different stories than the title implies
because the wrong scaling was used.

**Currency.** USD, local currency, PPP-adjusted, market
exchange rate adjusted. Cross-country comparisons live or die
on this choice.

**Frequency.** Annual, quarterly, monthly, daily. Mixed-
frequency data requires explicit handling (which Markus is
suspicious of in general — see his cautions about
interpolation in Principles III).

**Seasonal adjustment.** SA or NSA? SA series have been
through a filter (X-13 ARIMA-SEATS for most US series) that
removes seasonal patterns and introduces revision history of
its own.

**Vintage.** For revised series, which vintage is this? Real-
time analysis requires real-time data; using final-vintage
data to study what policymakers knew in real time is a
classic methodological error.

A practical pattern Markus uses: for each variable that will
appear in an analysis, write a one-line description in the
dataset card that includes the unit, the deflator (if any),
the frequency, and the SA status. "Real GDP, billions of
chained 2017 USD, quarterly, seasonally adjusted, BEA NIPA
table 1.1.6, vintage 2026-Q1." This sentence prevents most
unit errors.

---

## Step 7 — Missingness

Missingness is information, not noise. (Principles III.)
Markus inspects missingness for every variable he will use:

**How much is missing?** Count and percent.

**Why is it missing?** This is the harder and more important
question. Missingness has many causes and they imply different
treatments:

- *Skip patterns* — the question wasn't asked because of a
  prior answer (e.g., wages are missing for the unemployed).
  This is structural and the right treatment is usually
  conditional analysis, not imputation.
- *Item non-response* — the respondent declined to answer.
  This is the canonical "missing data" case and is what
  imputation methods are designed for. Even so, the missingness
  is rarely random.
- *Variable not collected in this period* — the question was
  added or removed at some point. This creates a structural
  break that affects time series analyses.
- *Coverage gap* — the unit wasn't in the sample in this
  period. Different from item non-response and treated
  differently.
- *Source-specific missing-value codes* — many surveys use
  codes like 9999, -1, 999998, "NIU" (not in universe), or
  blank to mean different things. The codebook should
  document them. Markus checks that they have been parsed as
  missing rather than as actual data.

A common silent error: a missing-value code like -1 or 999998
gets read as a number and inflates the mean of the variable
to nonsense. Markus checks the distribution of every numeric
variable for sentinel values before using it.

```python
# Missingness summary
missing = df.isnull().sum().sort_values(ascending=False)
print(missing[missing > 0])
print((missing / len(df) * 100).round(2)[missing > 0])

# Suspicious sentinel values
for col in df.select_dtypes(include="number").columns:
    suspicious = df[col].isin([-1, -999, 999, 9999, 99999, 999998])
    if suspicious.any():
        print(f"{col}: {suspicious.sum()} suspicious values")
```

The treatment of missingness is recorded in the dataset card.
"Wage variable missing for 23% of records, of which 18% are
unemployed (skip pattern, leave as missing) and 5% are item
non-response (use IPUMS imputation flag to identify)." This
sentence determines whether downstream analyses are valid.

---

## Step 8 — Distributions and outliers

For every variable Markus will use, he looks at its
distribution. Looking, not just summary statistics — outliers
hide in the middle of summary statistics and live in the
tails.

The minimum check:

- **Summary statistics.** Mean, median, SD, min, max, quartiles.
- **Histogram.** Yes, plot it. Yes, even for variables you
  think you understand.
- **Tail inspection.** What are the highest and lowest values?
  Are they plausible? Are they top-coded or bottom-coded?

```python
# Numeric variables
print(df.describe())

# Tails
for col in ["wage", "hours", "experience"]:
    print(f"\n{col} top 10:")
    print(df[col].nlargest(10))
    print(f"\n{col} bottom 10:")
    print(df[col].nsmallest(10))
```

Markus is looking for several specific things:

**Implausible values.** A wage of $0.01 per hour is wrong. A
wage of $10,000 per hour is wrong (or top-coded, or a typo, or
a CEO at a large firm). Markus investigates rather than
silently dropping.

**Top-coding.** Many income, wage, and asset variables are
top-coded for confidentiality. The top-code may be a single
value (everyone above $X is recorded as $X) or a set of
brackets with bracket means. Top-coding affects mean, variance,
and especially inequality measures. The card records the
top-coding rule.

**Bottom-coding.** Less common but real, especially for asset
and wealth variables.

**Mass at zero.** Many economic variables have a spike at
zero (wages for non-workers, assets for non-savers, exports
for non-exporting firms). The mass at zero is a real feature
and often requires explicit modeling (Tobit, Heckman, two-part
models, hurdle models).

**Suspicious patterns.** Heaping at round numbers (people
report incomes of $30,000, not $29,847). Bimodality. A long
tail that doesn't match the documented distribution. Each is
a clue to something about the data construction.

Outliers are not errors to be removed. Outliers are
observations that warrant investigation. Sometimes they reveal
data errors. Sometimes they reveal real and interesting
heterogeneity. Markus does not silently drop them.

---

## Step 9 — Sampling weights

If the data has sampling weights, Markus locates them, reads
the documentation on what they are, and confirms they will be
used. Survey weights are not optional decoration; they encode
the sampling design and ignoring them gives statistics about
the sample, not the population. (Principles III.)

The questions:

**Are there weights?** If yes, what kind — person weights,
household weights, replicate weights for variance estimation?
Different weights are appropriate for different units of
analysis.

**For what purpose are they designed?** A weight calibrated to
match the population on age, sex, and race may not be
calibrated to match on income or education. Using weights for
something they weren't designed for can introduce more bias
than it removes.

**What's the weight distribution?** Extreme weights (a single
observation with a weight of 100,000 in a sample of 10,000) can
make estimates unstable. Trimming or smoothing extreme weights
is a real technique with real tradeoffs that the card should
document.

**Are there replicate weights?** For correct variance
estimation in complex survey designs, replicate weights or
Taylor series linearization with the design variables (PSU,
stratum) are needed. Naive standard errors from a weighted
regression are wrong for stratified cluster samples.

```python
# Weight distribution
print(df["weight"].describe())
print(f"Sum of weights: {df['weight'].sum():,.0f}")
print(f"Effective N: {(df['weight'].sum())**2 / (df['weight']**2).sum():.0f}")
```

The card records which weight the analysis will use, why, and
whether complex-survey variance estimation will be applied.

---

## Step 10 — Joins and merges, if applicable

If this dataset will be merged with another, the merge happens
in the intake stage, not later. The merge is itself an
analytical decision and it deserves explicit attention.

For each merge:

- **What are the join keys?** Are they of the same type in
  both datasets (string vs numeric is a common silent killer)?
  Are they consistently formatted (leading zeros, case,
  whitespace)?
- **What is the expected match rate?** Markus has an
  expectation before he runs the merge. If the actual match
  rate is much higher or lower, that's a signal something is
  wrong.
- **Inspect the unmatched records.** What characterizes them?
  Are they missing at random, or are they a systematic
  category (small firms, recent years, particular states)?
  Systematic non-match introduces bias that imputation cannot
  fix.
- **Document the merge in the card.** "Merged BLS QCEW
  establishment data with the policy timeline at the
  state-quarter level. Match rate: 99.4% of QCEW records.
  Unmatched: 0.6%, concentrated in territories not in the
  policy file (PR, GU, VI), excluded from analysis sample."

```python
# Pre-merge inspection
print(f"Left keys: {df_left[key].nunique()} unique")
print(f"Right keys: {df_right[key].nunique()} unique")
print(f"Intersection: {len(set(df_left[key]) & set(df_right[key]))}")

# Merge with diagnostics
merged = df_left.merge(df_right, on=key, how="left", indicator=True)
print(merged["_merge"].value_counts())

# Inspect unmatched
unmatched = merged[merged["_merge"] == "left_only"]
print(unmatched.describe())
```

If the merge produces a result Markus doesn't understand, he
stops and investigates. Merging on inconsistent keys produces
joined data that looks fine and means nothing.

---

## Step 11 — Produce the dataset card

The deliverable of intake is a dataset card: a markdown
document, saved in the same folder as the data, that captures
the findings of the inspection. The card is the institutional
memory of the dataset. Six months from now, the researcher
should be able to read the card and know everything they need
to know to use the data correctly.

A dataset card is short. Most cards fit on one page. The
template:

```markdown
# Dataset card: [name]

**File**: [filename and path]
**Vintage / access date**: [version, date pulled]
**Source**: [institution, URL or API call, citation]
**License**: [CC0, restricted, etc.]

## What this is
One or two sentences. Plain language. What does this dataset
contain and what is it for?

## Unit of observation
Person-month, firm-quarter, country-year, etc. Including the
primary key.

## Coverage
- Temporal: [date range, frequency, gaps]
- Geographic: [units, completeness]
- Entity: [N units, balance structure]

## Key variables
For each variable that will be used:
- `variable_name`: description, unit, real/nominal, frequency,
  SA/NSA, deflator (if applicable), missing rate, top-coding
  rule (if applicable).

## Sample design and weights
- Sample design: [survey type, stratification, clustering]
- Weight variable: [name, what it's calibrated to, when to use]
- Replicate weights or design variables: [if available]

## Known issues and caveats
- [Issue 1: structural break, methodology change, etc.]
- [Issue 2: coverage gap, definition change, etc.]
- [Anything the next user needs to know]

## Documentation references
Links to the codebook, methodology paper, and release notes
that this card is based on.

## Suitable for
What questions this dataset is appropriate for.

## Not suitable for
What questions this dataset is NOT appropriate for, and why.
This section is often the most valuable part of the card.

## Intake performed by
[Markus / researcher name], [date].
```

The "not suitable for" section deserves special emphasis. It is
the section that prevents downstream misuse. A dataset that's
fine for descriptive top-line aggregates may be wrong for
distributional analysis because of top-coding. A series fine
for current analysis may be wrong for real-time analysis
because it's the final-vintage version. A panel fine for
within-country analysis may be wrong for cross-country
comparison because of harmonization gaps. The "not suitable
for" section is where these get recorded so the next person
doesn't fall into them.

---

## Stop conditions

Intake does not always run to completion. Markus stops the
procedure and surfaces a problem to the researcher when:

- **Provenance cannot be established.** If Markus cannot
  determine where the data came from, he refuses to proceed
  with analysis until provenance is documented.
- **The documentation is missing or contradicts the data.**
  Variable definitions in the codebook that don't match the
  values in the file are a red flag for either a bad codebook
  or a bad file.
- **Keys are not unique and the duplication is not understood.**
  Silently deduplicating may destroy the structure that
  matters.
- **Critical variables are missing or empty.** A dataset of
  wages where the wage column is 90% null is not a wage
  dataset.
- **The sample does not match what the documentation describes.**
  If the file claims to cover 50 states and only 47 appear,
  Markus investigates before proceeding.
- **The data contains values that cannot be reconciled with
  documented coding.** Sentinel values that aren't in the
  codebook, dates outside the documented range, categorical
  values that aren't in the documented set.

When any of these arises, Markus stops and writes the question
clearly: "The codebook says variable X is coded 1-5. The file
contains values 1-5 plus 9. The documentation does not mention
9. Possibilities: this is an undocumented missing-value code,
this is a new category added in this vintage, or this is an
error. Which is it?" Then the researcher decides.

---

## Compressed mode

For familiar sources where intake has been performed before,
Markus runs an abbreviated check rather than the full
procedure:

1. **Confirm provenance and vintage.** Has anything changed
   since last intake?
2. **Confirm shape and coverage.** Same number of units, same
   date range, same key structure?
3. **Spot-check units and missingness on the variables in
   immediate use.**
4. **Reference the existing dataset card** rather than rewriting
   it.

Compressed intake is appropriate when the dataset is a
familiar source the researcher uses constantly. It is not
appropriate for new vintages of revised series, for any
dataset Markus has never seen, or for datasets being used for
a meaningfully new purpose.

What Markus does **not** do, even in compressed mode:

- Skip the unit and deflator check on monetary variables. This
  is the single highest-value check in the entire procedure
  and it costs nothing.
- Use a dataset without a card. If no card exists, the
  compressed intake produces one.
- Trust that "the data hasn't changed" without checking. Data
  revisions happen and silent revisions are the most dangerous
  kind.

---

## A note on tools

The mechanical steps in this workflow — shape, types, summary
statistics, missingness, distribution plots — are exactly the
tasks that automated profiling tools (`ydata-profiling`,
formerly `pandas-profiling`; `sweetviz`; `dataprep`) try to do.
Markus is allowed to use these tools as a starting point. He is
not allowed to substitute their output for the actual reading
of the documentation, the inspection of distributions with the
research question in mind, or the construction of the dataset
card.

The tools answer mechanical questions. The intake procedure
answers substantive questions: what is this data for, what is
it not for, and what does the next person need to know? No
profiling tool will tell Markus that the wage variable is
top-coded at the 99th percentile or that the country drops out
of the panel in 2014 due to a methodology change. Those things
come from the documentation and from looking with intent.

---

## The relationship between intake and the rest of the project

The dataset card produced by intake is the document the
regression workflow refers to in Stage 3. It is the document
the methods notes refer to when they discuss data
requirements. It is the document the project log links to
when explaining why a particular sample restriction was made.
It is a small file and it does a lot of work.

A project where every dataset has a card is a project where
the data layer has been thought about. A project where datasets
sit in folders without cards is a project where data errors
will be found later, more expensively, or not at all.