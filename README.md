# Sediment skills

Agent skills for finding and retiring **sediment**: working code whose reason has expired. A
requirement changes or a replacement ships, the old mechanism keeps running, and repair code
("tape") grows to keep it agreeing with the new one. Dead-code tools miss it because it is still
called; review skills miss it because nothing proposed removing it.

| Skill | Use |
| --- | --- |
| [`sediment`](skills/sediment/SKILL.md) | Before or during a change: map what the change touches, find why each neighbour exists, and retire what the change makes redundant. |
| [`sediment-audit`](skills/sediment-audit/SKILL.md) | With no change proposed: dig the repo (deep) or re-check what changed since the last audit (fast), prove each finding, and settle them with the user. |
| [`sediment-grilling`](skills/sediment-grilling/SKILL.md) | Helper: puts open decisions to the user in rounds, dependencies first, each with a recommendation. |
| [`sediment-decisions`](skills/sediment-decisions/SKILL.md) | Helper: records a settled decision as a reason marker, or a decision record when one is worth it, and retires records it replaces. |

`sediment` and `sediment-audit` read the shared [reference](skills/sediment/REFERENCE.md): the vocabulary, reason markers,
verdict ladder, question types, signals and anti-patterns. The ledger and patterns formats live in
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
