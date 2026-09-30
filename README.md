# Sediment skills

Agent skills for finding and retiring **sediment**: working code whose reason has expired. A
requirement changes or a replacement ships, the old mechanism keeps running, and repair code
("tape") grows to keep it agreeing with the new one. Dead-code tools miss it because it is still
called; review skills miss it because nothing proposed removing it.

| Skill | Use |
| --- | --- |
| [`sediment`](skills/sediment/SKILL.md) | Before or during a change: map what the change touches, find why each neighbour exists, and retire what the change makes redundant. |
| [`sediment-audit`](skills/sediment-audit/SKILL.md) | With no change proposed: dig the repo (deep) or re-check what changed since the last audit (fast), prove each finding, and settle them with the user. |

Both read the shared [reference](skills/sediment/REFERENCE.md): the vocabulary, reason markers,
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
```

The skills assume no particular docs layout: they read whatever written record a repo has
(decision records, specs, tickets, READMEs, commit history). They use the `grilling` and
`domain-modeling` skills when available, and ask directly or write plain decision notes otherwise.
