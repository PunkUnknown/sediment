# Sediment reference

The core shared by the `sediment` and `sediment-audit` skills. The ledger and patterns formats
live in [LEDGER.md](LEDGER.md), read when a repo keeps them.

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
- **Written record**: wherever the repo keeps intent and decisions, in whatever form it has:
  decision records, specs, tickets, design and architecture docs, READMEs, a glossary, commit
  messages, pull request descriptions. Repos differ; use what exists and assume nothing else.

Facts are the agent's job. Requirements are the user's.

## Reason markers

A reason nobody wrote down cannot be checked; one written with the condition that ends it can.
When a reason is recorded beside the code, it uses a fixed marker in the file's comment syntax:

`reason: <claim>; until <condition>`

Write the condition in a checkable form whenever one fits, so a script can test it:

- `until <YYYY-MM-DD>`: expires on that date.
- `until <package>@<version>`: expires once the repo depends on that version or later.
- `until <symbol|path> is gone`: expires once that symbol or file no longer exists.

Otherwise write the condition in prose; the agent tests prose conditions by judgment. A reason
with no honest ending condition says `until the requirement changes`. Markers are found with
`(#|//|/\*|--|;|<!--) ?reason:`. A marker stays on one line, however
long, because the search reads one line at a time; never wrap it.

## Verdict ladder

Stop at the first rung that holds:

1. **Retire**: its reason has expired, or the change makes it false.
2. **Merge**: something else needs what it owns. Extend it; the fact keeps one owner.
3. **Rewrite**: its reason holds, but one of its rules collides with the change.
4. **Keep**: its reason holds and nothing collides.

No reason found is not an expired reason. Chesterton's fence: failing to find why something
exists is evidence about the search, not about the thing. Such a mechanism is tagged **ask**,
never retired on proof.

A **proven** verdict is still a claim until the change applying it passes the repo's tests and
checks. A new failure is evidence that the reason still holds, and overturns the verdict.

## Question types

Ask verdicts go to the user through the "sediment-grilling" skill, each phrased as one of these
types, naming the mechanism, its reason, what changed about that reason, and a recommendation.

- **Assumption**: "X exists because it assumes A. Does A still hold?"
- **Retire**: "X exists for R, and R is now false. Remove X?"
- **Merge**: "X and Y both own fact F. Which is the single owner, and does the other become a
  reader, presentation only, or nothing?"
- **Collide**: "X enforces rule Q; the change needs rule P. Which wins, and where does the
  losing case go?"
- **Keep**: "X stays because R. Confirm, and I will write R beside it."

Assumption questions are roots: their answers settle the others.

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
- **Patching the removal**: fixing what a removal broke so the removal can stay. The break was
  the mechanism's reason; reopen the verdict instead.
- **Doc sediment**: leaving any part of the written record describing the old system as current.
- **Asking for facts**: putting a question to the user that the code or git history answers.
- **Grilling the proven**: sending the user verdicts that evidence already decides.

## Boundaries

Sediment is about expired reasons. Over-engineering in new code, broken behaviour, and module
depth or seam design belong to other reviews. Recorded decisions are settled, except where their
reason has expired: that is exactly what sediment reopens.
