# Sediment skills

You know the engineer. Eight years in. Every team checks with them before touching the billing
sync. They don't remember every line, but they remember why each odd thing is there, and which
of those reasons quietly stopped being true.

Code outlives its reasons. A requirement changes, a replacement ships, and the old mechanism
keeps running because nobody told it to stop. The two disagree, so someone writes a sync. The
sync drifts, so someone adds a retry. The old mechanism is **sediment**. The sync and the retry
are **tape**.

Nothing catches it:

- Dead-code tools skip it. It's still called.
- Review skips it. Nobody proposed touching it.
- Tests pass. The tape is what makes them pass.

These skills are that engineer. Before a change, they ask every neighbour two questions: why do
you exist, and is that still true? They don't pull down a fence they can't explain; no reason
found means ask, never delete. They don't ask you what git can answer. And when they catch
themselves writing code to keep two copies in step, they stop and ask why there are two.

The cheapest fix for drift is one fewer copy.

| Skill | Use |
| --- | --- |
| [`sediment`](skills/sediment/SKILL.md) | Before or during a change: map what the change touches, find why each neighbour exists, and retire what the change makes redundant. |
| [`sediment-audit`](skills/sediment-audit/SKILL.md) | With no change proposed: dig the repo (deep) or re-check what changed since the last audit (fast), prove each finding, and settle them with the user. |
| [`sediment-grilling`](skills/sediment-grilling/SKILL.md) | Helper: puts open decisions to the user in rounds, dependencies first, each with a recommendation. |
| [`sediment-decisions`](skills/sediment-decisions/SKILL.md) | Helper: records a settled decision as a reason marker, or a decision record when one is worth it, and retires records it replaces. |

`sediment` and `sediment-audit` read the shared [reference](skills/sediment/REFERENCE.md): the
vocabulary, reason markers, verdict ladder, question types, signals and anti-patterns. The ledger and patterns formats live in
[LEDGER.md](skills/sediment/LEDGER.md), which the audit reads every run and `sediment` reads only
when a repo keeps those files.

## What a repo accumulates

An audited repo keeps two tracked files in a `sediment/` folder beside its in-tree tickets, or in
`.sediment/` at the root when it keeps none:

- `ledger.md`: open findings, suppressions, the areas still unswept, and a run log.
- `patterns.md`: how this codebase tends to lay down sediment, and the audit's own calibration.

The skills stay generic; everything specific to a codebase lives in those files.

## Install

Link each skill directory into your agent's skills folder, for example:

```bash
ln -s "$PWD/skills/sediment" ~/.agents/skills/sediment
ln -s "$PWD/skills/sediment-audit" ~/.agents/skills/sediment-audit
ln -s "$PWD/skills/sediment-grilling" ~/.agents/skills/sediment-grilling
ln -s "$PWD/skills/sediment-decisions" ~/.agents/skills/sediment-decisions
```

The skills assume no particular docs layout: they read whatever written record a repo has
(decision records, specs, tickets, READMEs, commit history). They depend only on the
skills in this repo.

## Credits

`sediment-grilling` adapts the `grilling` skill from Matt Pocock's
[skills](https://github.com/mattpocock/skills): the design tree, the frontier, and rounds of
numbered questions with recommended answers. `sediment-decisions` draws its bar for when a
decision record is worth writing from the same collection's `domain-modeling` skill.

Those skills are used under the MIT License:

```text
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
