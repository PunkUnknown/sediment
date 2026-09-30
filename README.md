# Sediment skills

Agent skills for finding and retiring **sediment**: working code whose reason has expired. A
requirement changes or a replacement ships, the old mechanism keeps running, and repair code
("tape") grows to keep it agreeing with the new one. Dead-code tools miss it because it is still
called; review skills miss it because nothing proposed removing it.

| Skill | Use |
| --- | --- |
| [`sediment`](skills/sediment/SKILL.md) | Before or during a change: map what the change touches, find why each neighbour exists, and retire what the change makes redundant. |
| [`sediment-audit`](skills/sediment-audit/SKILL.md) | With no change proposed: dig the repo (deep) or re-check what changed since the last audit (fast), prove each finding, and settle them with the user. |

Both read the shared [reference](skills/sediment/REFERENCE.md): the vocabulary, verdict ladder,
question types, reason markers, and the ledger and patterns formats.

## What a repo accumulates

An audited repo keeps two tracked files beside its tickets (by default `.scratch/sediment/`):

- `ledger.md`: open findings, suppressions, the areas still unswept, and a run log.
- `patterns.md`: how this codebase tends to lay down sediment, and the audit's own calibration.

The skills stay generic; everything specific to a codebase lives in those files.

## Install

Link each skill directory into your agent's skills folder, for example:

```bash
ln -s "$PWD/skills/sediment" ~/.agents/skills/sediment
ln -s "$PWD/skills/sediment-audit" ~/.agents/skills/sediment-audit
```

The skills call `grilling` and `domain-modeling` when available, for interviewing the user and
recording decisions.
