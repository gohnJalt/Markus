# [Project name] — log

This is the running log of work on the project. Every session
that produces anything substantive — decisions made, code
written, results generated, questions raised, dead ends hit —
gets a log entry. The log is how Markus and the researcher
maintain continuity across sessions.

Entries go in **reverse chronological order** (newest first).
This way the most recent state is at the top and the log doesn't
require scrolling to find where you left off.

An entry is short. Most entries are three to six bullets under
a date heading. The point is not to write a diary; the point is
to capture what the next session needs to know.

---

## Entry template

Copy this structure for each new entry.

```
## YYYY-MM-DD — [short session title]

**What I worked on**: [one sentence on the focus of the session]

**What changed**:
- [concrete change 1: code written, analysis run, decision made]
- [concrete change 2]
- [concrete change 3]

**What I found**:
- [result, observation, or insight]
- [result, observation, or insight]

**What's open**:
- [question, issue, or next step that carries into next session]
- [question, issue, or next step]

**Next session**: [one sentence on what to do next]
```

Not every entry needs every section. A session that only made
one small change can skip "what I found." A session that raised
no new questions can skip "what's open." The structure is a
scaffold, not a requirement.

---

## Entries

[Entries go below this line, newest first.]

---

## Example entry (delete once real entries exist)

## 2026-04-08 — initial data inspection

**What I worked on**: Running intake on the TÜİK HLFS microdata
pull for 2015-2024.

**What changed**:
- Downloaded HLFS annual files 2015-2024 from TÜİK.
- Wrote `code/01_clean.py` to standardize variable names across
  years (survey redesign in 2021 changed several variable codes).
- Produced dataset card at `data/cards/hlfs_2015_2024.md`.
- Noted the 2021 redesign as a structural break — may need to
  restrict sample or add a regime dummy.

**What I found**:
- Sample size holds up across years: ~400k person-year
  observations per year.
- The 2021 redesign changed the definition of "informal worker"
  in a way that affects ~3% of workers. Not a minor detail.
- Wage variable is top-coded starting in 2018 at the 97th
  percentile. Relevant for distributional work, less so for
  employment margins.

**What's open**:
- Decide whether to restrict to 2015-2020 for a methodologically
  consistent sample, or bridge the 2021 redesign with a
  harmonization.
- Check whether the SGK data (if we get it) would let us
  sidestep the HLFS redesign entirely.

**Next session**: Decide on the 2021 redesign treatment. If
restricting to pre-2021, finalize the sample construction and
move to identifying the treatment timing.
