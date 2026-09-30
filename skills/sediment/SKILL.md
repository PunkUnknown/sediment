---
name: sediment
description: Map what a change touches and retire what it makes redundant, asking the user where only they can decide. Use when planning a code or system change, or before patching a mismatch between two parts.
---

# Sediment

Code outlives its reasons. A requirement changes or a replacement ships, and the old mechanism
stays, wrapped in tape that keeps it agreeing with the new one. This skill removes it where it is
cheapest: before the change is written.

Read [REFERENCE.md](REFERENCE.md) first: the vocabulary, verdict ladder and question types below
are defined there. Read `CONTEXT.md` and the ADRs in the area; they hold most written reasons.

Depth follows the change. A change that touches no shared fact gets one line, "no shared facts
touched", and nothing more.

## 1. Map the contact surface

List the facts the change reads, writes, or redefines. For each fact, trace its owner and every
reader and writer through the state infrastructure and the architecture's ownership docs, then
every place a module computes the fact for itself, stored or not. A grep hit is a lead; a traced
reader is a neighbour. Defaults, fallbacks and optional calls that supply the fact are neighbours,
and so are the docs, ADRs, tests, fixtures and devtools that encode it.

If the repo has a sediment ledger (see the reference), its active entries on these facts are
known sediment: take them onto the map. Its Do not re-report lines are resolved; leave them be.

## 2. Find each neighbour's reason

**This is the skill.** A neighbour is a Chesterton's fence: it can only be judged against why it
exists, and the reason is usually the thing nobody wrote down. Look in this order: its reason
marker, ADR, spec or ticket, the commit that introduced it, the comment beside it. Trace to the
commit that introduced it, not the last one that touched it: follow moves and renames
(`git log --follow`, `git log -S`) past reformatting. Tests that guard the behaviour are
evidence of its reason. Stop once more history could not change the verdict.

State each reason as one claim about the world. A neighbour with no findable reason is itself a
finding; say so, and tag its verdict ask (see the reference).

## 3. Climb the verdict ladder

Give every neighbour a verdict from the ladder, tagged by who decides it: **proven** when the
evidence decides it, **ask** when it rests on a requirement only the user can confirm.

If the repo has a patterns file, apply each pattern's prevention to the change: run its checks
yourself, and put its questions to the user as ask verdicts.

Done when:

- [ ] Every fact the change touches has its owner, readers and writers listed.
- [ ] Every neighbour has a one-sentence reason, or is marked "no reason found".
- [ ] Every neighbour has a verdict, tagged proven or ask.
- [ ] Every pattern's checks have run, and its questions are among the ask verdicts.

## 4. Grill the ask verdicts

Call the Skill tool with "grilling". Seed its design tree with the ask verdicts, phrased as the
question types. Assumption questions are roots: their answers settle the merge and retire
questions hanging off them.

Done when every ask verdict has the user's answer.

## 5. Record

Put the map and its verdicts in the ticket. Write each surviving reason where the next reader
meets it: a reason marker at the seam (see the reference), and an ADR through the
"domain-modeling" skill when a future reader would re-question the decision without it.

## During the change: the tape tripwire

If you catch yourself writing code whose job is to make two things agree (a sync, a refresh, a
tolerance, a clock correction), **stop: that is the exact failure this skill prevents.** Name the
one fact both things represent and the reason two copies exist, then climb the ladder. Expired:
collapse to one owner; the repair goes with the duplicate. Needs the user: ask it as an
Assumption question. Holds: patch, and write the reason marker beside the seam.

## Closing the change

- [ ] Every retire verdict is removed, with its tests, fixtures, devtools and diagnostics.
- [ ] Every merge leaves one owner.
- [ ] Docs, ADRs and CONTEXT describe the new system; superseded explanations are gone.
- [ ] Every version, revision or count the change alters agrees across every doc and gate.
- [ ] Tracker and plan statuses match git.
- [ ] Ledger entries this change resolves are resolved in the commit that resolves them, one
      entry per commit: retired ones deleted, kept ones moved to Do not re-report with a reason
      marker beside the code.
- [ ] A pattern whose check or question proved wrong or noisy on this change is tuned, per the
      reference.
