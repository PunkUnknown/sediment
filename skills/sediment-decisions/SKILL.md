---
name: sediment-decisions
description: Record why a mechanism exists or why a decision was made, where the next reader meets it, and retire records the decision no longer backs. Called by the sediment skills after a decision is settled; also use when a settled decision needs writing down.
---

# Sediment decisions

A reason nobody wrote down cannot be checked. Write it where the next reader meets it, in a form
that says when it stops being true.

## Reason markers first

The default record is a reason marker beside the code it explains, in the format of
[REFERENCE.md](../sediment/REFERENCE.md): `reason: <claim>; until <condition>`, on one line.
Most decisions need nothing more.

## When a decision record is worth writing

Write a decision record as well only when all three hold:

1. **Hard to reverse**: changing your mind later has a real cost.
2. **Surprising without context**: a future reader would wonder why, and might "fix" it.
3. **A real trade-off**: there were genuine alternatives, and one was picked for reasons.

Missing any one, the marker is enough.

## Form and place

Use the repo's existing form and location for decisions: its decision records, design docs, or
wherever its written record keeps them. With none, create `docs/decisions/` when the first record
is needed, and name files `NNNN-slug.md`, numbered after the highest existing one.

A record is short: the decision and its reason in a few sentences, and the condition that would
make it false. Add rejected alternatives only when the rejection is not obvious.

## Replacing a decision

A decision that replaces an earlier one retires the earlier record in the same change: mark it
superseded with a pointer to the new one, or delete it if the repo keeps no history of decisions.
Update or remove the reason markers that cited it. A record describing the old system as current
is sediment.
