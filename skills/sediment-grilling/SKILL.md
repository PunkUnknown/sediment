---
name: sediment-grilling
description: Put a set of open decisions to the user in rounds, dependencies first, each with a recommended answer. Called by the sediment skills to settle their ask verdicts.
---

# Sediment grilling

Settle open decisions with the user without guessing at answers you have not heard yet.

## The tree

Arrange the decisions as a tree: each decision branches into the decisions that depend on its
answer. Assumptions ("does A still hold?") are roots; the decisions they settle hang off them.
The **frontier** is every decision whose prerequisites are settled.

## Rounds

Ask the whole frontier in one round, and nothing beyond it. Number each question, state what it
decides and why it is open, and put your recommended answer under it:

```
**Q1. <title>**: <the question, with the choices when there are several>

Recommended: <your answer, and the reason in one line>
```

Wait for the answers. Each answer settles branches and unblocks others: recompute the frontier
and ask the next round. A question whose answer depends on another question still open belongs
to a later round.

## Facts are yours

Facts are never asked. When a question needs a fact the code, git history or tools can supply,
look it up; only the questions downstream of that lookup wait for it, so ask the rest now. The
user decides; the agent finds out.

## Done

Done when the frontier is empty: every decision answered, nothing left silently assumed. Read
the settled set back in one line per decision, and act on none of it until the user confirms.
