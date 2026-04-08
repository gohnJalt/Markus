# [Project name]

**Status**: [active / paused / complete / abandoned]
**Started**: [YYYY-MM-DD]
**Last touched**: [YYYY-MM-DD]
**Primary researcher**: [name]
**Project folder**: `60_Projects/[project-name]/`

---

## The question

One paragraph. Plain language. What are we trying to find out and
why does it matter? If the question has evolved since the project
started, the current version goes here and the history lives in
the log.

The question should pass the test from `Run a regression properly`
Stage 0: specific enough that you could recognize an answer if
you saw one. "How does minimum wage affect employment" fails. "How
did the 2019-2024 Turkish minimum wage increases affect formal-
sector employment of workers aged 18-29 in the manufacturing and
services sectors, relative to older workers in the same sectors"
passes.

## The estimand

What population quantity, causal effect, structural parameter, or
descriptive statistic are we after? In words, not equations.
Directly from `Principles.md` Section II: the estimand is what we
want to know, not what the regression computes.

If there are multiple estimands — descriptive, causal, predictive
— list them separately. A single project can legitimately have
all three.

## The contribution

What does this project add that the existing literature doesn't?
One paragraph. Be honest: is the contribution a new dataset, a
new method, a new setting, a new mechanism, a synthesis? If you
can't name the contribution, you don't yet have a project — you
have a question. Go back.

## The audience

Who is this for? A specific journal, a working paper series, a
policy brief, a chapter of a dissertation, an internal memo?
The audience determines the standard of evidence, the length,
the style, and the amount of methodological ceremony that
belongs in the final product.

## Data

Which datasets will this project use? For each, a one-line
summary and a pointer to the dataset card.

- **[Dataset name]**: [one-line description]. See
  `[path to dataset card]`.
- **[Dataset name]**: [one-line description]. See
  `[path to dataset card]`.

If a dataset is planned but not yet acquired, list it here with
status "pending" and a note on what's required to obtain it.

## Identification strategy

For causal projects, one paragraph on how the question gets
answered. What is the source of variation? What is the
identifying assumption in plain language? What are the threats
to that assumption?

This is the hardest section to write and the most important.
If you cannot write it, the project is not yet ready for
estimation — it is ready for more thinking. That is a
legitimate state and the log should reflect it.

For descriptive and predictive projects, one paragraph on the
methodological approach instead: what measures, what
decompositions, what validation?

## Methods

Which methods from `modern-toolkit-references.md` are planned?
List them with a one-line justification for each. This section
will evolve as the project develops; the log tracks the evolution.

- [Method 1]: [why this method fits the question]
- [Method 2]: [why this method fits the question]

## Current status

One paragraph on where the project stands right now. Updated
whenever the status changes materially. This should be readable
in thirty seconds and tell you what state the project is in
without requiring you to read the log.

Example: "Data acquired and cleaned. Sample restriction
finalized (prime-age workers in manufacturing and services,
2015-2024). Main specification running; Callaway-Sant'Anna
estimator implemented, event study plots generated. Pre-trends
look reasonable. Robustness checks on sample and method next.
Next session: drop 2020-2021 for pandemic robustness, rerun."

## Open questions

A running list of unresolved substantive questions. Not a
to-do list — the log handles that. This list is for the
conceptual and methodological questions that are genuinely
open and that the researcher and Markus need to think about.

- [Question 1, in plain language]
- [Question 2, in plain language]

When a question gets resolved, it moves out of this section
and into the log entry where it was resolved, with a brief
note on the resolution.

## Decisions made

A running list of the consequential methodological decisions
that have been made and are now treated as settled (unless
revisited). Each decision has a one-line rationale.

- **[Decision]**: [rationale]. [date]
- **[Decision]**: [rationale]. [date]

Example:
- **Restrict sample to workers aged 25-54**: standard prime-age
  definition, avoids education and retirement margins. 2026-03-12.
- **Use Callaway-Sant'Anna rather than TWFE**: staggered
  treatment adoption, heterogeneous effects likely. 2026-03-14.
- **Cluster SEs at province level**: treatment varies at
  province level. 2026-03-14.

This section is the "project's institutional memory." When
Markus or the researcher is about to make a choice that's
already been made, they consult this section first.

## Files and structure

A brief map of what's in the project folder. Updated as things
are added.

```
60_Projects/[project-name]/
├── README.md                 ← this file
├── log.md                    ← running session log
├── data/
│   ├── raw/                  ← never edit
│   ├── interim/              ← intermediate outputs
│   ├── processed/            ← analysis-ready
│   └── cards/                ← dataset cards
├── code/
│   ├── 01_clean.py
│   ├── 02_build_sample.py
│   ├── 03_main_analysis.py
│   └── ...
├── results/
│   ├── tables/
│   ├── figures/
│   └── regressions/
├── drafts/
│   └── [paper or memo drafts]
└── refs/
    └── [key papers for this project]
```

## References

Key papers for this project. Not a full literature review — just
the handful that are actively informing the work. Each reference
gets a one-line note on why it's here.

- **[Author (Year)]**: [one line on what this paper contributes
  to our project].
- **[Author (Year)]**: [one line on what this paper contributes
  to our project].

## Notes to future self

Anything the researcher wants to remember that doesn't fit
elsewhere. Hunches, connections, worries, things to come back
to. Low-ceremony, high-value.
