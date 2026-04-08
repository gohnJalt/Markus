# Session start ritual

This is the prompt Markus uses at the start of every working
session on an existing project. The point is to reload context
fast so that Markus is oriented before any substantive work
begins.

The session start ritual is not optional. A session that skips
it is a session that reinvents wheels, repeats decisions already
made, and risks contradicting prior work the researcher has
forgotten about. Thirty seconds of orientation saves an hour
of rework.

---

## The ritual

At the start of a session on an existing project, the researcher
pastes the following into the chat, with the filenames filled
in for the specific project:

```
Starting a session on [project name]. Before we work, please
read and internalize:

1. The project README: [paste contents of 60_Projects/[project]/README.md]

2. The project log, most recent three to five entries:
[paste the top of 60_Projects/[project]/log.md]

3. Any dataset cards for data we'll touch today:
[paste contents of relevant 60_Projects/[project]/data/cards/*.md]

Once you've read these, please confirm orientation by answering:

- What is the current state of this project in one sentence?
- What were the last three decisions made?
- What is the most pressing open question?
- What did the previous session recommend as next steps?

Then wait for me to tell you what we're working on today.
```

This ritual does three things. It loads the project context into
Markus's working memory for the session. It forces Markus to
confirm he has understood the context, which catches cases where
he has misread or skipped something. And it surfaces continuity
issues — if Markus's summary of the state doesn't match the
researcher's understanding, the mismatch gets caught before any
new work builds on it.

---

## The variant for new projects

For a brand new project, the ritual is different — there's
nothing yet to read. The researcher instead pastes:

```
Starting a new project. Before we work, please read and
internalize:

1. [The standard identity and knowledge files, if not already
   in Project knowledge: system prompt, principles, tools,
   data sources, modern toolkit references]

2. Your job in this first session is to help me write the
   README for the new project. We'll walk the sections of the
   README template together and produce a first-draft project
   brief. At the end of the session, I want a real README.md
   that captures the question, the estimand, the data, the
   identification strategy, and the initial open questions.

Let's start with the question. Ask me clarifying questions
until you're confident you could write the "question" section
in the plain-language way the README calls for.
```

The new-project ritual turns the first session into a structured
conversation that produces the README as its deliverable. This
is much more valuable than a session where Markus jumps
straight into data work before the question is pinned down.

---

## The variant for the end of a session

At the end of a working session, the researcher asks Markus to
produce the log entry. The prompt is short:

```
End of session. Please draft a log entry for today covering:
what we worked on, what changed, what we found, what's open,
and what to do next session. Use the template from
log.md and keep it to five to ten bullet points total.
```

Markus produces a draft entry, the researcher reviews and edits
it, and the final version gets appended to the top of the
project's `log.md`. This takes two minutes and is the single
most valuable habit in the whole continuity system.

If the researcher skips the end-of-session ritual, the next
session starts with a fuzzier log and the continuity degrades.
If the skip happens often, Markus starts making decisions that
contradict earlier sessions because the earlier sessions weren't
captured. The discipline is small and the payoff is large.

---

## When not to use the ritual

The session start ritual is overkill for very quick interactions:
a one-off question, a quick conceptual discussion, a status
check. It's intended for working sessions where real project
work will happen.

The test: if the session will produce code, analysis, decisions,
or results that the project will depend on later, run the
ritual. If the session is a conversation that won't change the
project state, skip it.

Markus can help the researcher decide: "This sounds like
project work — want me to run the session start ritual before
we dig in?" is a reasonable thing for him to ask when the
researcher opens with a project-specific question cold.

---

## A note on how this solves continuity

Claude does not have persistent memory between sessions. Within
a Project in Claude, the Project knowledge is loaded every
session, but the conversation history is not — each new chat
starts fresh. This is a real limitation and no amount of
prompting makes it go away.

The continuity system described here is the workaround: the
project's state lives in files (README, log, dataset cards),
and the session start ritual loads those files into the
session at the beginning. This is slower than true persistent
memory but it has two advantages over persistence: the state
is visible (the researcher can read it), and the state is
editable (the researcher can correct it).

The discipline is: **if it isn't in the README or the log, it
doesn't exist for the next session**. The researcher and Markus
both have to take this seriously. A decision made verbally and
not logged will be forgotten. A question raised and not captured
will be re-asked. A result generated and not recorded will be
regenerated, possibly differently.

The cost of the discipline is real — every session ends with a
small amount of writing. The payoff is that the project
actually accumulates over time rather than being a rolling
series of fresh starts.
