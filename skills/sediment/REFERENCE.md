# Sediment reference

Shared by the `sediment` and `sediment-audit` skills.

## Vocabulary

- **Fact**: one piece of information the system holds, stored or computed on the fly. Each fact
  has one owner.
- **Reason**: the requirement or assumption a mechanism exists for, stated as a claim about the
  world that can turn false. History is evidence of a reason, not authority for it: a reason found
  in history still answers to current requirements, and current requirements win.
- **Sediment**: a mechanism whose reason has expired.
- **Tape**: code whose job is keeping sediment working: sync, refresh, reconcile, retry, clamp,
  tolerance, fallback, an optional call with a permissive default.
- **Neighbour**: a mechanism that feeds a change or fact, consumes it, owns it, repairs something
  it touches, or collides with it. A change's neighbours form its **contact map**.

Facts are the agent's job. Requirements are the user's.

## Reason markers

A reason nobody wrote down cannot be checked; one written with the condition that ends it can.
When a reason is recorded beside the code, it uses a fixed marker in the language's comment
syntax:

`reason: <claim>; until <condition that makes it false>`

The marker makes reasons greppable (`(#|//|--) ?reason:`), so an audit can test every condition
mechanically. A reason with no honest ending condition says `until the requirement changes`.

## Verdict ladder

Stop at the first rung that holds:

1. **Retire**: its reason has expired, or the change makes it false.
2. **Merge**: something else needs what it owns. Extend it; the fact keeps one owner.
3. **Rewrite**: its reason holds, but one of its rules collides with the change.
4. **Keep**: its reason holds and nothing collides.

No reason found is not an expired reason. Chesterton's fence: failing to find why something
exists is evidence about the search, not about the thing. Such a mechanism is tagged **ask**,
never retired on proof.

## Question types

For ask verdicts, put to the user through the "grilling" skill. Each names the mechanism, its
reason, what changed about that reason, and a recommendation.

- **Assumption**: "X exists because it assumes A. Does A still hold?"
- **Retire**: "X exists for R, and R is now false. Remove X?"
- **Merge**: "X and Y both own fact F. Which is the single owner, and does the other become a
  reader, presentation only, or nothing?"
- **Collide**: "X enforces rule Q; the change needs rule P. Which wins, and where does the
  losing case go?"
- **Keep**: "X stays because R. Confirm, and I will write R beside it."

Assumption questions are roots: their answers settle the others.

## Ledger

The **ledger** holds unresolved sediment findings between runs: `.scratch/sediment/ledger.md`, or
wherever the repo keeps its tickets. It is a tracked file, committed with the work that changes
it, so git history records when each finding appeared and resolved.

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
one line per audit, `<date> <deep|fast> <commit audited>`. Fast audits start from the last line's
commit.

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
- **Origin**: where its findings were laid down: the specs, tickets, commits or conversations
  that introduced them or left them behind, cited.
- **Missed by**: **user**, **agent**, both, or **unknown** (see below).
- **Cause**: what the origin shows was missed, in one sentence; **unknown** when no origin shows
  it. Causes are often hard to find, and an unknown cause is a normal state, not a defect.
- **Prevention**: a **check** the agent runs itself, derived from the shape when the cause is
  unknown or an agent miss; a **question** the agent asks the user for a user miss.
- **Evidence**: the findings that show it (ledger ids, with a few words each, since retired
  entries leave the ledger).
- **Quiet runs**: audits in a row that found no new instance.

### Identifying a pattern

1. **Trace each finding to its origin.** Find the spec, ticket, commit or conversation that
   introduced the mechanism, and the one that should have retired it. Cite the line. Stop once
   more history could not change the classification. No origin found: missed by and cause stay
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
- **Graduation**: a pattern that keeps producing findings is a standing failure. Propose its
  prevention for the repo's agent instructions (checks) or spec template (questions); the user
  decides.

## Signals

Generic signals, true of any codebase. A repo's own signals live in its patterns.

- Two representations of one fact, with code keeping them in step.
- One value computed independently in two modules, even when only one copy is stored.
- A run of fixes on one seam; a fix titled by a symptom of drift.
- A union with one reachable value; a counter that only ever reads 1.
- A version branch behind a stricter gate; a required dependency called with a fallback.
- A production default only tests rely on; a public method with no production callers.
- State published but read only by diagnostics; a comment describing a mechanism the code lacks.
- Docs or gates disagreeing on one number; a fixture named for what it no longer asserts.

## Anti-patterns

- **The parallel system**: building a new mechanism beside one that already owns the fact.
  Merge instead.
- **Taping the seam**: fixing drift between two copies without asking why two exist.
- **Doc sediment**: leaving a doc, ADR or tracker status that describes the old system.
- **Asking for facts**: putting a question to the user that the code or git history answers.
- **Grilling the proven**: sending the user verdicts that evidence already decides.

## Boundaries

Over-engineering in new code belongs to ponytail; broken behaviour to diagnosing-bugs; module
depth to improve-codebase-architecture. ADRs are settled decisions, except where their reason has
expired: that is exactly what sediment reopens.
