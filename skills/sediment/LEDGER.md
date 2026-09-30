# Sediment ledger and patterns

The files a repo keeps between sediment runs. `sediment-audit` writes them; `sediment` reads them
when they exist. Vocabulary is in [REFERENCE.md](REFERENCE.md).

## Ledger

The **ledger** holds unresolved sediment findings between runs, in a `sediment/` folder beside
the repo's in-tree tickets, or at `.sediment/` in the repo root when it keeps none. It is a
tracked file, committed with the work that changes it, so git history records when each finding
appeared and resolved.

An **entry** carries an id, verdict, `[proven|ask]`, the one-line finding, its evidence, the
date first seen, the date last verified, and a status: **unverified** (recorded without an
audit's proof) or **active**.

A resolved finding leaves the entry list:

- **Retired** (the code is gone): delete the entry in the same commit that removes the code, one
  commit per entry, so each removal can be reverted alone. Git holds its history; nothing is left
  for a run to re-find.
- **Disproven** (the evidence failed) or **kept** (the user confirmed its reason): shrink it to
  one line under **Do not re-report**: `<id> <what>. <why it stays>. <date>`. A kept mechanism
  also gets its reason marker beside the code. These lines suppress re-reporting while the
  thing still exists; when its reason goes stale, it returns to the entries as active.

The ledger also keeps a **Next scope** list (the areas no run has swept yet) and a **run log**:
one line per audit.

`<date> <deep|fast> <commit audited>; untested parts that changed an outcome: <names or none>`

Fast audits start from the last line's commit. The untested parts are the lifecycle rules below
that carry a reason marker; a deep run names each one that changed a finding, a pattern or a
report this run.

## Patterns

Every codebase lays down sediment its own way. The **patterns** file records how this one does:
`patterns.md` beside the ledger, tracked the same way. The skills stay generic; a repo's own
habits live here.

The patterns file also keeps an **Audit calibration** section: the ways audits misread this repo.
It is consulted before proving findings. Planning-time prevention, promotion, quiet runs and
graduation apply only to codebase patterns, never to calibration entries.

A **calibration entry** carries a **Signal** (what the misreading looks like: the kind of
candidate that proved not to be sediment), a **Check** (what to verify before proving such a
candidate) and **Evidence** (the withdrawn findings, with what disproved them). An entry is
added or extended when a finding is disproven and a check would have caught it; one withdrawn
finding is enough, since each one cost a wrong report.

A **pattern** carries:

- **Shape**: what it looks like in the code.
- **Signal**: a general search that can discover new instances. Known names and cases belong
  under Evidence, so quiet runs measure discovery, not re-finding.
- **Origin**: where its findings were laid down: the part of the written record (a commit, spec,
  ticket or conversation) that introduced them or left them behind, cited.
- **Missed by**: **user**, **agent**, both, or **unknown** (see below).
  <!-- reason: untested lifecycle part (missed by); until 3 deep runs after 2026-09-30 in which it changed no outcome -->
- **Cause**: what the origin shows was missed, in one sentence; **unknown** when no origin shows
  it. Causes are often hard to find, and an unknown cause is a normal state, not a defect.
- **Prevention**: a **check** the agent runs itself, derived from the shape when the cause is
  unknown or an agent miss; a **question** the agent asks the user for a user miss.
- **Evidence**: the findings that show it (ledger ids, with a few words each, since retired
  entries leave the ledger).
- **Quiet runs**: deep audits in a row that found no new instance.
  <!-- reason: untested lifecycle part (quiet runs); until 3 deep runs after 2026-09-30 in which it changed no outcome -->

### Identifying a pattern

1. **Trace each finding to its origin.** Find the part of the written record that introduced
   the mechanism, and the one that should have retired it. Cite the line. Stop once more
   history could not change the classification. No origin found: missed by and cause stay
   unknown.
2. **Classify the miss** from what the origin says, never from a guess about intent:
   - **User miss**: the origin never states the requirement or assumption that decided the
     mechanism, or states one that was never revisited when the world changed. Only the user
     could have supplied it.
   - **Agent miss**: the origin had the information, or the code was there to trace, and the
     work did not reach it: a trace stopped short, a scope was drawn too narrowly, a rule was
     applied past its purpose.
   - **Both**: a rule or requirement from the user that the agent applied as written, where
     the rule itself is silent on the case.
3. **Group by shape.** Findings with the same shape form a candidate pattern, whether or not their
   cause is known. A shared cause, when found, confirms the grouping; it is not required.
4. **Write the prevention from the miss.** Agent miss: a check the agent can run without the
   user. User miss: a question for planning time. A prevention that the evidence does not
   support is left out.

Its life:

- **Promotion**: a pattern is written only when two findings share a shape. One finding is a
  finding.
- **Tuning**: a pattern is revised when evidence disagrees with it: an origin shows a different
  cause or a different miss; its signal misses instances other seams found, or flags things that
  prove not to be sediment; its prevention would not have caught its own evidence. A pattern
  written without origins is completed by the next run's origin trace. Revise the field and add
  one line under the pattern's **Tuned**: `<date> <field>: <what changed>. <evidence>.` The user
  may tune any field directly; a user-tuned field changes only by proposal to the user.
- **Expiry**: after three quiet runs, delete it. The habit is fixed or the rule was never real;
  git keeps it.
  <!-- reason: untested lifecycle part (expiry); until 3 deep runs after 2026-09-30 in which it changed no outcome -->
- **Graduation**: a pattern that keeps producing findings is a standing failure. Propose its
  prevention for wherever the repo instructs agents (checks) or plans changes (questions); the user
  decides.
