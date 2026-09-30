# Sediment

Agents change what you ask and leave the rest. The old code keeps running beside the new, and
the next session patches the two into agreement. Nothing breaks. It just piles up.

## What sediment is

**Sediment** is code that still runs after its reason has expired.

- You moved settings to the server. The old `localStorage` read is still there.
- `isLate` is always `!onTime`. Two fields, one fact.
- The docs still say settings are saved in the browser. They moved to the server a year ago.

**Tape** is code whose only job is keeping sediment working: a sync between two copies, a
fallback like `?? true`, a retry around the sync.

Dead-code tools miss it because it's still called. Review misses it because no diff touched it.
Tests pass, often because the tape makes them pass.

## How it forms

Four sessions, each one reasonable:

1. **The ask:** "Save the theme to the server." The agent adds a `theme` column and an endpoint.
   Nobody says "and stop reading `localStorage`", so the old read stays.
2. **The bug:** on a new laptop, the theme flashes light, then dark. Two copies, two answers.
3. **The patch:** a new session copies the server value into `localStorage` on load. The bug is
   gone. That copy is tape.
4. **The patch on the patch:** offline, the copy fails and the stale value wins. The next
   session adds a retry. More tape.

Now one fact has two owners, a sync between them, and a retry around the sync, and every test is
green. The `localStorage` read is the sediment: its reason ("there are no accounts") expired in
step 1. Retire it, and the sync and the retry go with it.

## Why it forms

Mostly vibe coding: changes accepted because they work, not because anyone traced them.

- **The spec is too thin.** It says what to add, never what the change replaces.
- **The spec isn't read.** It did say the old path goes; the agent built the new one and stopped.
- **Nobody knows what the change touches.** One edit quietly breaks an assumption three files
  away, and the fix goes on top instead of at the cause.
- **Nobody knows the old code exists.** You can't retire what you never found.

The first is on the person writing the spec, the rest mostly on the agent, and usually it's
both. The audit records which for every finding, so the next spec asks the missing question and
the next agent runs the missing check.

## How it's classified

Each finding gets a verdict, from the first rung that holds:

| Verdict | When | Example |
| --- | --- | --- |
| **Retire** | Its reason has expired. | The `localStorage` read, once settings live on the server. |
| **Merge** | Two things own one fact. Keep one owner. | `isLate` and `onTime`. |
| **Rewrite** | Its reason holds, but it says something false. | The docs still describing browser-saved settings. |
| **Keep** | Its reason holds. Write the reason down. | An old URL redirect that old SMS links still use. |

And a tag for who decides:

- **proven**: the evidence decides (no production callers, the commit that replaced it).
- **ask**: only you can say (is anyone still opening those SMS links?).

No reason found is never proof. It's a question for you.

## What the skills do

**Before a change**, `sediment` lists what the change touches, finds why each piece exists, and
retires what the change makes redundant, in the same change. It asks you only what the code
can't answer. If the agent starts writing a sync or a fallback, it stops and asks why there are
two copies.

**On an existing repo**, `sediment-audit` digs for sediment, proves each finding, and ranks them:

```
S1 merge [proven] isLate mirrors !onTime; only the dashboard reads it. [orders/state.ts:40]
S2 retire [proven] localStorage theme read; settings moved to the server. [ui/theme.ts:12]
S3 keep [ask] /track redirect; no record of when old SMS links stop mattering. [web/routes.ts:21]
net: 1 retire, 1 merge, 0 rewrite, 1 keep
ask: 1
```

Kept code gets its reason written beside it, with the condition that ends it:

```ts
// reason: old SMS tracking links still get opened; until 2027-03-01
```

Findings persist in a ledger between runs, and the audit learns which kinds of sediment this repo
keeps producing; see [the ledger and the patterns](#the-ledger-and-the-patterns).

## The skills

| Skill | Use |
| --- | --- |
| [`sediment`](skills/sediment/SKILL.md) | Before or during a change: map what the change touches, find why each neighbour exists, and retire what the change makes redundant. |
| [`sediment-audit`](skills/sediment-audit/SKILL.md) | With no change proposed: dig the repo (deep) or re-check what changed since the last audit (fast), prove each finding, and settle them with you. |
| [`sediment-grilling`](skills/sediment-grilling/SKILL.md) | Helper: puts open decisions to you in rounds, dependencies first, each with a recommendation. |
| [`sediment-decisions`](skills/sediment-decisions/SKILL.md) | Helper: records a settled decision as a reason marker, or a decision record when one is worth it, and retires records it replaces. |

`sediment` and `sediment-audit` read the shared [reference](skills/sediment/REFERENCE.md): the
vocabulary, reason markers, verdict ladder, question types, signals and anti-patterns. The ledger
and patterns formats live in [LEDGER.md](skills/sediment/LEDGER.md), which the audit reads every
run and `sediment` reads only when a repo keeps those files.

## The ledger and the patterns

An audited repo keeps two files, committed like code, in a `sediment/` folder beside its tickets
(or `.sediment/` at the root). The skills stay generic; everything specific to your repo lives
here.

### `ledger.md`: what's been found

One audit never sees everything, and the next run's blind spots differ. The ledger keeps every
open finding, so nothing found once is lost.

```
## S2 retire [proven] localStorage theme read; settings moved to the server
- Evidence: no reader since a41c9e0 moved settings to /settings. [ui/theme.ts:12]
- First seen 2026-09-01. Verified 2026-09-30. Status: active

## Do not re-report
- S3 /track redirect. Old SMS links still use it. 2026-09-30

## Next scope
- web/admin (only spot-checked)

## Run log
2026-09-30 deep 1e22b8e
```

- **Each run re-checks every entry first.** Code gone: the entry is deleted. Evidence failed: it
  moves to Do not re-report.
- **Retiring a finding deletes its entry in the same commit as the code**, so git shows when each
  one was found and fixed, and each removal can be reverted on its own.
- **Do not re-report** holds what you chose to keep, so the next run doesn't ask again.
- **Next scope** lists what this run didn't reach; the next run starts there.
- **Run log**: a fast run only checks what changed since the last logged commit.

### `patterns.md`: how this repo goes wrong

Every codebase produces sediment its own way. When two findings share a shape, the audit writes
it down as a pattern:

```
### P1 - Docs outlive the module they describe
- Shape: a doc points to a file or owner that a replacement deleted.
- Signal: links and paths in docs that no longer resolve.
- Missed by: agent. The change's trace stopped before the docs.
- Prevention: when removing a module, grep every doc that names it.
- Evidence: S4, S9, S12. Quiet runs: 0.
```

- **Signal** is a search the next audit runs to find new cases.
- **Missed by** says who could have prevented it: **user** (the spec never said it), **agent**
  (the information was there and not traced), or both.
- **Prevention** is a check the agent runs (agent miss) or a question it asks you when planning
  (user miss). `sediment` applies these before every change.
- **Quiet runs** count audits that found no new case. After three, the pattern is deleted: the
  habit is fixed.
- A pattern that keeps producing findings **graduates**: the audit proposes adding its prevention
  to your agent instructions. You decide.

The file also keeps **audit calibration**: the audit's own mistakes. When a finding turns out
wrong (say, templates that looked unused but are loaded by name), the check that would have caught
it is recorded, and every later run does it first.

## Install

Every route installs all four skills together: `sediment` and `sediment-audit` call the two
helpers and share one reference file.

### Claude Code

```
/plugin marketplace add punkunknown/sediment
```
```
/plugin install sediment@sediment
```

Send them as two separate prompts. In the desktop app, type them into the Code tab, or use
**+** → **Plugins** → **Add plugin**. Skills show as `sediment:sediment`,
`sediment:sediment-audit` and so on.

### Codex

```bash
codex plugin marketplace add punkunknown/sediment
codex plugin add sediment@sediment
```

Also covers the Codex desktop app after a restart.

### GitHub Copilot CLI

```bash
copilot plugin marketplace add punkunknown/sediment
copilot plugin install sediment@sediment
```

### Gemini CLI

```bash
gemini extensions install https://github.com/punkunknown/sediment
```

### Antigravity CLI

```bash
agy plugin install https://github.com/punkunknown/sediment
```

### Devin CLI

```bash
devin plugins install punkunknown/sediment
```

### Grok Build

```bash
grok plugin install punkunknown/sediment --trust
```

Then enable it under `/plugins` and start a new session.

### Pi agent harness

```bash
pi install git:github.com/punkunknown/sediment
```

### Qoder

Qoder loads the skills through [`.qoder-plugin/plugin.json`](.qoder-plugin/plugin.json).

### Swival

```bash
swival skills add --global https://github.com/punkunknown/sediment
swival skills add sediment
```

### Cursor, OpenCode, Windsurf and any other agent

The [`skills`](https://github.com/vercel-labs/skills) CLI copies the skills into the folders each
agent reads (`.agents/skills/`, `.claude/skills/` and so on):

```bash
npx skills add punkunknown/sediment --skill '*'
```

Add `-g` to install for your user instead of the current project.

### Manually

Clone the repo and link each skill into your agent's skills folder:

```bash
for s in sediment sediment-audit sediment-grilling sediment-decisions; do
  ln -s "$PWD/skills/$s" ~/.claude/skills/$s
done
```

### Uninstall

| Host | Command |
| --- | --- |
| Claude Code | `/plugin remove sediment` |
| Codex | `codex plugin remove sediment` |
| Copilot CLI | `copilot plugin uninstall sediment` |
| Gemini CLI | `gemini extensions uninstall sediment` |
| Devin CLI | `devin plugins remove sediment` |
| Grok Build | `grok plugin uninstall sediment` |
| Pi agent | `pi uninstall sediment` |
| `skills` CLI | `npx skills remove sediment sediment-audit sediment-grilling sediment-decisions` |
| Manual | Delete the four links |

Uninstalling leaves each audited repo's `sediment/` folder alone; those files are yours.

## Requirements

None beyond git. The skills assume no particular docs layout: they read whatever written record a
repo has (decision records, specs, tickets, READMEs, commit history), and depend only on each
other.

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
