# Replication audit

This is the procedure Markus follows when the researcher wants
to replicate someone else's empirical paper. Replication is one
of the most valuable things an empirical economist does. It
sharpens methodological skills, reveals how robust published
results actually are, and produces understanding of a literature
that no amount of reading can match. It is also one of the
hardest things to do well, because it requires reverse-
engineering decisions the original authors made but did not
always document.

Replication audit is not the same as running someone else's
replication package. Running a provided replication package
produces the paper's tables and figures on the authors' data
with the authors' code. Replication audit is a deeper exercise:
understanding why the paper's results are what they are,
whether they survive reasonable variations, and what the paper's
findings actually license.

The procedure has ten stages. The deliverable is a **replication
report** — a short document that captures what was replicated,
what was not, what changed under variation, and what the audit
learned about the paper. Like the dataset card from
[[`new-dataset-intake.md`]], the report is the institutional memory
of the audit and the thing that makes the work reusable.

---

## Stages 0 through 2 — Before touching the data

These three stages happen before any code runs and before any
dataset is downloaded. Skipping them wastes time: you end up
reproducing tables without understanding what they mean, which
is a hollow kind of replication.

### Stage 0 — Choose the paper deliberately

Not every paper is worth replicating. The right paper to replicate
depends on why you're doing it. Markus helps the researcher name
the reason:

- **Methodological learning.** The paper uses a method you want
  to understand deeply. Choose papers with clean designs and
  full replication packages.
- **Literature orientation.** You need to understand a specific
  finding to position your own work. Choose papers that are
  actively cited in the current literature, not historical
  curiosities.
- **Robustness interrogation.** You suspect a result may not be
  robust and want to test it. Choose papers whose findings
  are consequential enough that the robustness question
  matters.
- **Data pipeline learning.** You want to learn how a specific
  dataset is used in practice. Choose papers that use the
  dataset well and document their choices clearly.
- **Direct input to your own project.** The replication is a
  step in a larger project (e.g., extending the paper's
  sample, applying the method to a new setting, using it as a
  baseline). This is the most common reason.

Naming the reason determines what success looks like. A
methodological learning replication succeeds if you understand
the method afterward, even if you can't perfectly reproduce the
numbers. A robustness interrogation succeeds if you can characterize
what the result depends on, whether or not you end up agreeing with
the published version.

### Stage 1 — Read the paper properly

Before any code, the paper gets read. Not skimmed, not
introduction-and-tables, read. The reading should produce,
by the end:

- A one-paragraph summary of the paper's question, data, method,
  and main finding, in your own words. If you can't write this,
  you don't yet understand the paper well enough to replicate it.
- A list of the paper's main tables and figures with a one-line
  note on what each one shows.
- A list of the **identifying assumptions** the paper relies on,
  in plain language. For DiD: what's the parallel trends claim?
  For IV: what's the exclusion? For RDD: what's the continuity
  argument? The paper should state these explicitly; if it doesn't,
  your job is to reconstruct them from the method.
- A list of the **judgment calls** you notice in the paper —
  the sample restrictions, the variable definitions, the
  specification choices, the SE clustering, the bandwidth
  selection, the control variables. These are the things you'll
  test under variation later.
- A list of **questions** that arose during reading. Things that
  aren't clear, things that seem too quick, things you'd want to
  ask the authors.

Markus helps with all of this. A good replication audit starts
as a careful reading session, with Markus in Socratic mode —
asking the researcher to state things back in their own words,
flagging assumptions the paper glides past, noting where the
argument does and doesn't quite land.

### Stage 2 — Locate the replication materials

What does the paper provide, and what does it not?

- **Replication package.** Many journals (AER, QJE, ReStud,
  JHR, AEJ) now require replication materials. Find them. The
  AEA Data and Code Repository and openICPSR are common hosts.
- **Data.** Is the data publicly available, restricted-access,
  or proprietary? If restricted, can the researcher get access,
  and if not, can the analysis be adapted to public data?
- **Code.** Is the code in Stata, R, Python, Matlab, Julia?
  Is it well-organized or a mess? Are the dependencies documented?
  Does it actually run as provided? (Often not.)
- **Documentation.** Is there a README explaining the code
  structure? Are the steps from raw data to final tables
  traceable?

Markus records all of this in the replication report. If the
replication materials are incomplete or broken — which is
common, even for published papers — that itself is a finding
worth noting.

---

## When there is no replication package

The procedure described above assumes the paper provides code
and ideally data. This is increasingly common for recent papers
in top journals, but it remains the exception across the
literature as a whole. Most papers — older papers, papers in
field journals, papers using restricted data, papers from before
journal data policies — provide only the text, the tables, and
sometimes a data appendix. When that's all you have, the audit
becomes a different exercise: not *replication* but
*reconstruction*.

Reconstruction is harder, slower, and produces weaker
conclusions than replication. It is also much more common in
practice and much more educational. The researcher who can
reconstruct a paper from its text alone has demonstrated
something a researcher who runs a replication package has not:
that they understand the paper deeply enough to rebuild it.

This section replaces Stages 3 through 5 of the standard
procedure when no code is available. The early stages (0-2)
and the late stages (6-10) apply unchanged.

### What "successful reconstruction" means

Before starting, name what success looks like. Reconstruction
cannot achieve digit-for-digit reproduction without the original
code and data, and pretending otherwise sets a standard the
audit will fail. Reasonable standards of success, in increasing
order of strictness:

**Qualitative match.** The reconstructed estimate has the same
sign and roughly the same order of magnitude as the published
result. This is the minimum standard. If you can't even achieve
this, something is fundamentally different between your build
and the paper's, and the gap is itself the finding.

**Quantitative similarity.** The reconstructed estimate is
within (say) 20% of the published result, with overlapping
confidence intervals. This is a respectable standard for most
reconstructions and is enough to engage substantively with the
paper's claims.

**Close match.** The reconstructed estimate is within a few
percent of the published result. This is rare without the
original code, and when it happens it strongly suggests the
audit has correctly inferred all the consequential decisions.

Markus and the researcher should agree on the target standard
before beginning, because the standard determines how much
effort to invest and when to stop.

### Stage 3a — Inventory what you have and what you don't

Read the paper a second time, this time as a forensic exercise.
For every choice the analysis required, note whether the paper
states it, implies it, or omits it entirely.

The relevant choices fall into categories:

**Data acquisition**:
- Which datasets does the paper use? Specific source, version,
  vintage, access date if mentioned.
- Are the datasets publicly available? Restricted? Proprietary?
- Does the paper name a specific extract or release? (For
  IPUMS, the extract specification matters; for FRED, the
  series IDs and vintage matter.)

**Sample construction**:
- What population is the analysis on? (Age range, country,
  industry, time period, employment status, etc.)
- What restrictions are applied and why?
- What share of the source data does the final sample represent?
  (This is often reported and is a useful target.)
- Are there imputation flags, missing-value handling, or
  topcoding adjustments?

**Variable construction**:
- How is the outcome measured? Exact definition, units,
  transformations.
- How is the treatment measured? Exact definition, units,
  variation source.
- What controls are included, and how are they constructed?
- Are any variables derived from multiple sources? How are
  they merged?

**Specification**:
- What is the regression equation, exactly? (Including any
  fixed effects, interactions, weights.)
- What is the standard error specification? (Cluster level,
  HAC, bootstrap, etc.)
- What software was used? (Sometimes inferable from
  acknowledgments or footnotes.)

For each item, mark the source: **stated explicitly** in the
paper, **implied** by context or convention, or **omitted**.
The omitted items are where reconstruction risk lives. The
implied items are where reconstruction error lives — your
inference about what was implied may be wrong.

This inventory is the most important single deliverable of
Stage 3a. It tells you what you'll have to guess and where
your guesses will matter most.

### Stage 3b — Acquire the data, with adversarial care

Now go get the data. This step is more dangerous than it sounds,
because the data you acquire may not be the data the paper used,
even when the source is the same. Specifically:

**Vintages and revisions**. A paper using "the FRED GDP series"
in 2018 was using the 2018 vintage of that series, which has
been revised since. ALFRED gives you historical vintages; FRED
gives you the latest. Use the vintage the paper would have had,
not the latest.

**Survey waves and methodology changes**. A paper using "the
CPS ASEC" before the 2014 redesign is using a different income
construct than after. A paper using "the European Labour Force
Survey" before a methodology revision is using a different
employment definition. The same survey name can mean different
things in different years.

**Extracts and harmonization**. A paper using "IPUMS-CPS" used
a specific extract built with specific variables, weights, and
year ranges. Replicating from "IPUMS-CPS in general" produces a
different file. When the paper specifies the extract, use those
specifications. When it doesn't, document your guess.

**Restricted data substitutes**. If the paper uses restricted
administrative data and you can't get access, you have a real
problem. Two options: find a public dataset that captures the
same variation imperfectly (and document the gap), or
reconstruct only the parts of the analysis that don't require
the restricted data. Both are legitimate — but be honest in
the report about which one you did.

Run the new dataset intake procedure (see
[[`new-dataset-intake.md`]]) on each acquired dataset. The
resulting cards become evidence in the reconstruction report.

### Stage 3c — Reconstruct the data pipeline

Now build the analysis dataset from the raw sources. This is
where most reconstruction effort goes, and it's where most
reconstruction failures originate.

The discipline:

**Reconstruct in the order the paper implies, not in your
preferred order.** Papers usually describe sample construction
in a specific sequence (e.g., "we start with all workers in
sector X, then drop Y, then merge Z"). Following the paper's
sequence makes it easier to compare intermediate sample sizes
to anything the paper reports. Reordering for convenience
introduces silent differences.

**At every step, check the running sample size against any
number the paper mentions.** Many papers report "our final
sample contains N observations" or "after dropping X we are
left with Y." These are reconstruction targets. If your N is
50% off after a single restriction, you've already diverged,
and the divergence is much easier to find now than after the
regression.

**Document every inferred choice.** When the paper says "we
restrict to prime-age workers" and you decide that means 25-54,
write down "inferred 25-54 from 'prime-age,' standard convention
in the literature." When the paper says "real wages" and you
choose a deflator, write down "used CPI-U-RS, base 2023, because
the paper doesn't specify and CPI-U-RS is the standard for
historical wage comparisons." These notes accumulate into the
reconstruction's audit trail.

**Stop and re-read when you're uncertain.** If you find yourself
guessing at a fourth consecutive choice, the paper isn't
specific enough for confident reconstruction and you should
note this in the report. Either the audit's standard of success
needs to drop, or the paper needs to be set aside as not
reconstructable from its text alone.

The output of Stage 3c is an analysis dataset that you believe
matches the paper's analysis dataset, with a written record of
every decision that went into building it. The written record
is as important as the dataset itself — it's what lets you
diagnose discrepancies later.

### Stage 4 (no-code variant) — Reconstruct the estimation

Now write the regression code from scratch, working from the
paper's specification. This is mechanically simpler than the
data pipeline but conceptually delicate, because the paper's
description of its specification is often less precise than it
appears.

For each piece of the specification:

**The equation.** Write it out in full, including every fixed
effect, control, and interaction. Compare to what the paper
shows, which is sometimes a stylized version of what was
actually estimated. If the paper writes `Y = α + βX + ε` but
also mentions "controlling for age, education, and state fixed
effects," the actual regression is more complex than the
displayed equation.

**Sample weights.** If the data is from a survey with weights,
the paper should say whether they were used. If it doesn't,
the convention varies by subfield: labor and demography use
weights routinely, finance and macro often don't. When in
doubt, run both and report whichever matches the published
numbers more closely, but document that you tested both.

**Standard errors.** The paper should specify the SE
construction. If it doesn't, the most common conventions are
robust SEs for cross-sectional work and cluster-robust SEs at
the unit-of-treatment level for panel work. As with weights,
when in doubt, run multiple and report which matches.

**Software conventions.** Stata, R, Python, and Matlab implement
"the same" estimator differently in small ways: degrees-of-
freedom corrections, weight normalizations, missing-value
handling, default standard error formulas. A paper that ran in
Stata may not exactly reproduce in Python even with otherwise
identical specifications. These differences are usually small
but they're real, and they're worth knowing about when
diagnosing 1-2% gaps.

Run the regression. Compare to the published result.

### Stage 5 (no-code variant) — Diagnose discrepancies

Now comes the hard part. Your reconstruction will produce a
number. The paper produces a different number. The gap is
informative, and the diagnosis of the gap is most of the value
of the audit.

Possible sources of discrepancy, in roughly the order Markus
investigates them:

**Sample.** The most common source. If your N is meaningfully
different from the paper's N (when the paper reports it), the
sample construction differs, and that's where to look first.
Walk back through Stage 3c step by step, checking sample sizes
against any reported counts. The divergence is usually one or
two specific steps that produced the wrong filter.

**Variable definition.** The second most common source. Did
the paper use hourly wages or weekly wages? Levels or logs?
Real or nominal, with which deflator? These choices look
small in the text and produce large differences in coefficients.

**Specification.** Did the paper include controls you omitted,
or omit controls you included? Are the fixed effects at the
same level? Are interactions structured the same way?

**Software differences.** Smaller and rarer, but real. If
everything else matches and you're still 1-2% off, this may be
the residual.

**Unstated choices.** The paper made a choice it didn't
mention. This is the most frustrating diagnosis because you
can't fix it without contacting the authors, but you can
characterize it: "After reconstructing the sample to match the
paper's reported N, the coefficient differs by 12%. This
likely reflects a variable construction choice not specified
in the text."

**Errors in the paper.** Rare, but they happen. If you've
exhausted the other possibilities and the gap remains, the
paper itself may have an error. This is a serious finding and
should be stated carefully and confidently.

The diagnosis goes in the reconstruction report. A diagnosis
of "I don't know why the numbers differ" is worth less than
"The numbers differ by 8%, almost certainly because of the
sample restriction at step 3, which I cannot reconstruct
exactly because the paper says only 'we exclude outliers.'"
The second diagnosis tells the next reader exactly what the
limitation is.

### A note on standards of evidence for reconstructions

A successful reconstruction does not establish a paper's
correctness. A failed reconstruction does not establish a
paper's incorrectness. Reconstructions live in a different
epistemic space than replications, and the audit report should
say so.

What a successful reconstruction *does* establish: that the
paper's findings are reachable from its text by a careful
reader, that the data construction is plausible, and that the
estimation is what the text describes. This is a meaningful
form of validation — most research, when audited this way,
turns out to be reproducible in this looser sense, and some
turns out not to be.

What a failed reconstruction *does* establish: that the paper's
description is not sufficient for an outside reader to rebuild
the analysis. This may reflect insufficient detail in the
paper, or it may reflect undocumented choices the authors made,
or it may reflect errors in the audit. The report should
present the failure as a limit on reconstructability, not as a
verdict on the paper, unless the audit has independent evidence
for stronger claims.

The general principle is that reconstructions support
*qualitative* engagement with a paper much better than
*quantitative* engagement. After a successful reconstruction,
the researcher can confidently say "the paper's main result
survives the variations I tested" or "the paper's main result
appears to depend on a sample restriction that's not
defensible in my reading." After a failed reconstruction, the
researcher can confidently say "I could not rebuild the
paper's analysis from its text" but cannot easily go further.

### The reconstruction report differs from the replication report

When the audit was a reconstruction rather than a replication,
the report has a different structure. Specifically, it adds:

- **The inventory from Stage 3a**: what was stated, implied,
  and omitted in the paper.
- **The list of inferred choices**: every guess the
  reconstruction made, with the rationale.
- **The standard of success used**: qualitative match,
  quantitative similarity, or close match.
- **The diagnosis of any gaps**: where the reconstruction
  diverged from the paper, and why.

These additions can be folded into the replication report
template by adding sections under "Reproduction status" and
"Data pipeline." A reconstruction report is necessarily longer
than a replication report because it has to document its own
limits — the limits are part of the finding.

### When to abandon a reconstruction

Not every paper can be reconstructed from its text. When
Markus encounters one, he says so clearly rather than producing
a half-finished audit that pretends to more than it has.

Signs that a reconstruction should be abandoned:

- The data pipeline requires more than three or four
  consecutive guesses about unstated choices.
- The data is restricted-access and no public substitute
  exists.
- The reconstructed N is more than 50% off from the paper's
  reported N and the cause cannot be located.
- The reconstructed estimate has the wrong sign and you've
  exhausted plausible explanations.
- The paper's specification is described in a way that isn't
  unambiguous even with charitable interpretation.

When abandoning, the report still has value. It documents what
was tried, what was learned, and where the paper was opaque.
This is itself a finding about the paper's reproducibility,
and it may be worth noting publicly if the paper is
consequential.

The discipline is the same as in [[`new-dataset-intake.md`]]'s
stop conditions: stopping honestly is better than continuing
dishonestly.

---
## Stages 3 through 5 — Reproducing the published results

The first analytical goal is to reproduce what the paper reports,
on the paper's own terms. This is the baseline. Everything else
is variation from this baseline.

### Stage 3 — Run the replication package as provided

Start by running the code exactly as the authors provided it.
Fix nothing, change nothing. Document what happens:

- Does the code run? On what system, with what software versions?
- Does it produce outputs that match the paper's tables and
  figures, to the digit?
- Where does it fail, and how?

Three outcomes are possible:

**Full reproduction.** The code runs, the numbers match. This
is the ideal and it's rarer than you'd expect. Record success
and move on.

**Partial reproduction.** The code runs but some numbers don't
match. This is the most common outcome. The discrepancies are
informative and need to be tracked down: different software
versions, unreported data updates, rounding, sample definitions
that changed between submission and publication. Investigate.

**Reproduction failure.** The code doesn't run, or produces
results wildly different from the paper. This is a finding,
not a failure of the replication — it means something about
the paper's reproducibility that the journal's review process
didn't catch. Document it honestly.

In any of these cases, Markus's job is to help diagnose, not
to paper over discrepancies. A replication that silently fixes
the authors' errors is not a replication.

### Stage 4 — Understand the data construction

Whether or not the code ran cleanly, the next step is to
understand what the data actually is. This means walking
through the data construction code step by step, in sequence,
and documenting what each step does.

For each step in the data pipeline:

- What does this step do, in plain language?
- What sample restriction or transformation does it apply?
- What is the rationale? (From the paper, or inferred.)
- Is it the choice you would have made? If not, what would
  you have done differently, and why?

This is slow work. A serious paper can have 50 steps in its
data pipeline and each one can represent a consequential
choice. Walk them anyway. The data construction is where
most of the hidden decisions live, and understanding them is
usually more valuable than understanding the regression.

Run the intake procedure from [[`new-dataset-intake.md`]] on the
paper's final analysis dataset. Produce a dataset card for it,
stored alongside the replication code. This card becomes part
of the replication report and makes the dataset reusable for
future audits or extensions.

### Stage 5 — Walk the estimation

With the data understood, walk the estimation code the same
way: step by step, what does this do, why, would you have
done it differently?

For each regression or structural estimation in the paper:

- What is the estimand, in the language of `Run a regression
  properly` Stage 1?
- What is the estimator, and does it target that estimand under
  the paper's identification assumptions?
- What are the standard errors, and why are they the right
  choice for this design?
- What diagnostics would be informative, and does the paper
  report them?

This stage often reveals that the paper's estimation is
cleaner or messier than the text implies. Papers sometimes
report results from more stringent specifications than the
text describes; papers sometimes report results from less
stringent specifications than the text describes. Both cases
are worth knowing.

---

## Stages 6 through 8 — Interrogating the results

With reproduction achieved (or honestly documented as
partial), the audit moves into interrogation. This is where
the replication goes beyond running the code and starts
asking what the results actually mean.

### Stage 6 — Vary the judgment calls

Go back to the list of judgment calls from Stage 1 and, one at
a time, vary each one and see what happens to the main result.

The logic is the same as Stage 8 of `Run a regression properly`:
a robustness check is worth running if its failure would change
your interpretation. For a replication, the variations to test
are:

- **Sample restrictions.** If the paper drops workers under
  25, what happens at 20? 30? If the paper drops the top 1%,
  what happens at 5%, 0.5%, 0%?
- **Variable definitions.** If the paper uses log wages, try
  levels. If the paper uses employment, try hours. If the paper
  uses a specific deflator, try an alternative.
- **Specification choices.** If the paper uses TWFE on a
  staggered design, try Callaway-Sant'Anna and see if the
  result survives the modern DiD revolution. If the paper uses
  a particular bandwidth in RDD, try the Cattaneo et al. data-
  driven selector.
- **SE clustering.** If the paper clusters at one level, try
  another defensible level. If the paper has few clusters, try
  wild cluster bootstrap.
- **Time period.** If the paper ends in 2015, does it survive
  extension to the present? (Use with care: extension can
  itself introduce changes in methodology or data quality.)

For each variation, document what changed and by how much. A
result that moves from 0.05 (significant) to 0.04 (also
significant) is stable. A result that moves from 0.08 to
-0.02 is not. A result that moves from 0.05 (significant) to
0.04 (not significant because the SE also changed) is
something in between.

### Stage 7 — Test the identification

The harder interrogation is of the identification itself.
This goes beyond "is the result robust?" to "is the design
valid?"

- **For DiD**: plot the pre-trends, even if the paper didn't.
  Run the Goodman-Bacon decomposition. Check whether the result
  is driven by one or two treated units.
- **For IV**: check the first stage carefully. Use the Lee-
  McCrary-Moreira-Porter critical values rather than the old
  rule-of-thumb. Run overidentification tests if the paper has
  multiple instruments. Try alternative instruments if any are
  defensible.
- **For RDD**: run the density test for manipulation. Try
  alternative bandwidths. Check the discontinuity visually
  across different windows.
- **For matching**: check covariate balance before and after.
  Assess common support.
- **For any design**: run the analysis on a placebo outcome
  that the treatment shouldn't affect, or on a pre-treatment
  period. If the "effect" is there anyway, the design is
  compromised.

These tests may or may not be in the paper. If they are,
re-run them and confirm. If they aren't, run them yourself and
report what you find. An identification test the paper didn't
run and doesn't survive is a significant finding — one you
should communicate with care and confidence.

### Stage 8 — Consider alternative interpretations

The final interrogation is substantive. Even if the result
reproduces and survives robustness, is the paper's
interpretation the right one?

- What else could explain the result besides the mechanism the
  paper proposes?
- Does the paper's story fit the magnitudes? A 2% effect on
  wages is hard to reconcile with some mechanisms and easy to
  reconcile with others.
- How does the result compare to related findings in the
  literature? If the paper's estimate is an outlier, why?
- Does the result generalize, or is it specific to the setting?
  What about the setting is doing the work?
- What would a sociologist or political scientist say about
  this finding? (Principles VIII.) Sometimes the economic
  story is the right one; sometimes the institutional or
  political story better fits the facts.

This stage is where Markus is most useful as a sparring
partner. The researcher proposes an interpretation; Markus
pushes back with alternatives; the exchange sharpens the
researcher's understanding of what the paper is and isn't
showing.

---

## Stages 9 through 10 — Writing it up

### Stage 9 — Produce the replication report

The deliverable of the audit is a replication report. Like the
dataset card, it's a short document — usually two to five pages
— that captures what the audit learned. The template:

```markdown
# Replication report: [Author (Year), "Paper title"]

**Paper**: [full citation]
**Auditor**: [name]
**Audit date**: [date]
**Replication package**: [link, if available]

## Summary
One paragraph: what the paper claims, what the audit tried to
do, and what the audit found.

## Reproduction status
- **Code runs**: yes / partially / no. Notes on dependencies
  and environment.
- **Numbers match**: yes / partially / no. List discrepancies
  and their likely sources.
- **Files produced**: yes / partially / no. Which tables and
  figures were reproduced?

## Data pipeline
One paragraph on the data construction. Pointer to the dataset
card produced in Stage 4. Note any choices that seemed
consequential.

## Estimation
One paragraph on the estimation. What estimand, what estimator,
what SEs, what diagnostics. Note any choices that seemed
consequential or that the text glosses over.

## Robustness findings
Bullet points. For each variation tested, what was tested,
what happened, what it implies.

- [Variation 1]: [result, implication]
- [Variation 2]: [result, implication]
- [Variation 3]: [result, implication]

## Identification findings
What the audit learned about the identification strategy. Was
the design as clean as claimed? Did additional tests support or
challenge it?

## Interpretation findings
What the audit learned about the interpretation. Are there
alternative stories that fit the data? How does the result sit
in the literature?

## Overall assessment
One paragraph. What does the audit conclude about the paper?
Is the main finding credible, partially credible, or not?
What caveats would you attach to citing it?

This section is where Markus and the researcher should be most
honest and most careful. A replication report is not a public
takedown — it's a working document. But it should say what was
found, even when what was found is uncomfortable.

## Lessons and next steps
What did the audit teach the researcher? What does it imply
for their own work? Are there follow-up analyses, extensions,
or questions this audit raised?
```

### Stage 10 — Decide what the audit feeds

A replication audit is usually a means to an end. At the close,
Markus helps the researcher name what the audit feeds:

- **A methods note** in `10_Methods/` capturing something
  learned about a technique.
- **A dataset card** in a shared data folder capturing
  something learned about a data source.
- **A project** in `60_Projects/` that builds on the
  replication — an extension, a critique, a re-analysis.
- **An entry in a running "literature notes" file** capturing
  the paper's place in the researcher's mental map of the
  field.
- **Nothing formal**, but the tacit understanding the
  researcher now has is its own output.

Naming the output gives the audit closure. An audit that just
ends is an audit whose lessons fade. An audit that feeds
something concrete is an audit that compounds.

---

## Compressed mode

For quick methodological checks — reading a paper and
sanity-checking one specific result — the full ten-stage
procedure is too much. Compressed replication audit looks like:

1. Read the paper, focusing on the specific result in question.
2. Find the replication package; confirm the code runs.
3. Walk the estimation for the specific result.
4. Test one or two critical variations.
5. Write a short note (half a page) on what was learned.

Compressed mode is appropriate when the researcher already
knows the paper well and has a specific, narrow question about
it. It is not appropriate for first encounters with a paper,
for robustness investigations, or for any audit whose results
will feed a publication.

What Markus does **not** do, even in compressed mode:

- Paper over discrepancies between the code's output and the
  paper's tables. Document them.
- Skip the identification test if the identification is what's
  in question.
- Present a replication as complete when it's partial. Say
  what was done and what wasn't.

---

## A note on honesty

Replication audits can produce uncomfortable findings. A paper
you respect may turn out to have a result that doesn't survive
obvious variations. A paper in your subfield may turn out to
have a data pipeline you can't trust. A senior colleague's work
may turn out to have problems you didn't expect.

Markus's job is to surface what the audit actually found, in
the replication report, honestly and with appropriate caveats.
His job is not to decide what the researcher does with that
information. Publishing a critical comment, raising concerns
privately, incorporating the caveats into the researcher's own
work, or simply updating the researcher's prior on the paper —
all are legitimate responses to a critical audit finding, and
the choice is the researcher's.

What Markus does not do is soften the findings to avoid
discomfort. A replication audit that understates its own
findings is worth less than no audit at all, because it
creates false confidence in the audited paper. The discipline
from [[`Principles.md`]] Section XI applies: honesty is more
useful than agreement.