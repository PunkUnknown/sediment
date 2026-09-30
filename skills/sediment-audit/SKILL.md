---
name: sediment-audit
description: Dig existing code for sediment when no change is proposed: mechanisms whose reason expired, tape holding them up, leftovers of replaced systems. Use when asked to audit legacy, stale, dead, or taped-over code, or "what should we remove"; "fast" re-checks what changed since the last audit.
---

# Sediment audit

The sediment skill, run on what already exists. No change is proposed; the job is to find
mechanisms whose reason has expired, prove it, and settle with the user what to retire.

Read [REFERENCE.md](../sediment/REFERENCE.md) and [LEDGER.md](../sediment/LEDGER.md) first, then
the repo's written record.

## Depth

An audit runs at one of two depths. **Deep** is the default; **fast** runs when the user asks
for it.

- **Deep** does every step below: the whole scope, all five seams, pattern identification.
- **Fast** re-checks instead of digging. It runs only the mechanical parts: the ledger, reason
  markers, pattern signals, calibration checks, the caller census and stated numbers, on what
  changed since the last audit. Each step below says what fast skips. Fast proves its findings
  to the same standard as deep; only the search is narrower.

Recommend a deep run in the report when the ledger's Next scope is not empty, when the last
deep run is three or more audits ago, or when fast mode meets a candidate it cannot classify
without judgment: a shape no pattern covers, or a reason only history could supply.

## 0. Ledger

Findings persist in the ledger (see LEDGER.md), so one run's blind spot never loses an
earlier run's finding. If a ledger exists, before digging:

- Re-verify every unverified and active entry against the current code. Code already gone: delete
  the entry. Evidence failed: move it to Do not re-report. Otherwise: active, with fresh evidence.
- Check each Do not re-report line still applies. A stale reason returns it to the entries as active.

Done when every entry carries today's verification date and every suppression has been checked.
Both depths run this step in full.

## 1. Scope

If the user named an area (a module, a subsystem, a single fact), take it.
Otherwise the scope is the whole repo. The ledger's Next scope comes first; after it, hot spots
set the order, never the boundary: replacement and breaking commits and runs of fixes on one seam,
then the rest.

Done when:

- [ ] The scope is stated in one line: the named area, or "whole repo".
- [ ] Every replacement the written record describes is on the replacement list, not only
      recent commits.
- [ ] The fact list is written out (see the fact census) before any fact is examined.

**Fast:** the scope is every file changed since the ledger's last audit commit
(`git diff --name-only <commit>..`), plus every file the ledger's entries cite. Next scope is
carried forward untouched. State the commit range as the scope line.

## 2. Dig

Work five seams. Each produces candidates; none is proof yet.

1. **Fact census.** First write the fact list: every piece of shared data the code keeps, every
   fact the written record names, and every value a module computes for itself. Then,
   fact by fact, list every place it is computed, stored or not. More than one is a candidate:
   two owners of one fact. The written list is the checklist; every fact on it gets an entry.
   Include stated numbers: every version, revision or count that appears in more than one doc
   or gate is compared across all of them.
2. **Written reasons.** Grep every reason marker first. Test the checkable `until` forms (date,
   package version, symbol or path) with a script; test prose conditions by judgment. Then collect the unmarked reasons from the written
   record and seam comments, and test each claim the same way.
3. **Tape clusters.** Fix commits clustered on one seam, repair vocabulary in code. Trace each
   cluster to the two things that must agree, and to why two exist.
4. **Replacement leftovers.** For each removal or breaking commit, and each superseded decision,
   search for what the old system left: names, values, branches, fixtures, doc sentences.
5. **Caller census.** Run it as a throwaway script (in a temp dir or the repo's evidence folder),
   never by eye; its output table is the evidence. For every surface production could rely on
   (exported symbols, methods, parameter variants, defaults, optional calls, published state),
   count production readers against test and diagnostic readers. Zero production reliance is a
   candidate. Use the repo's own unused-code tooling first when it has some (knip, vulture and
   the like), then the compiler or a language server; fall back to text search and read each hit.

Then search every pattern's signal from the repo's patterns file, and scan the reference's
generic signals against every file the dig touched.

**Fast:** run only the mechanical parts, within the fast scope: the stated-number comparison from
seam 1, the reason markers from seam 2, the caller census (seam 5), every pattern signal and the
generic signals. Skip the fact list, unmarked reasons, tape clusters and replacement leftovers.

## 3. Prove

Read the patterns file's Audit calibration section first, if present, and run each entry's
check on the candidates its signal matches.

**This is the skill.** An audit that reports claims reproduces the drift it is hunting. A doc or
comment is a claim, never evidence. For each candidate, find its reason, then evidence that the
reason expired: a production caller count, a shipped data value, a commit, a gate that already
rejects the case. A candidate without evidence is dropped or reported as "unproven".

Some surfaces are used from outside the source: stored data, saved settings, URLs, external
callers. Code cannot prove them unused; that takes runtime or stored evidence. Without it, the
verdict is ask. A candidate with no findable reason is ask too, never proven (see the reference).

Done when:

- [ ] Every candidate has a reason (or "no reason found") and evidence, or is marked unproven.
- [ ] Every candidate has a verdict from the ladder, tagged proven or ask.

## 4. Report

One ranked list, ordered by disagreement: two owners of one fact first, dead helpers last.
Active ledger entries and new findings rank together, and every finding is cited by its ledger
id, so the report and the ledger never number the same finding twice. Each finding leads with
one line, followed by evidence sub-bullets where needed:

`<id> <verdict> [proven|ask] <what>. <reason>. [path:line]`

The tag says who decides the verdict, not whether the work is done: **proven** means the
evidence decides it; **ask** means it needs the user's answer. Ranking outranks the tag: an ask
two-owner finding sits above a proven dead helper.

State the coverage in one line: what was swept by script, what by reading, what only
spot-checked. End with two lines:

`net: <N> retire, <M> merge, <R> rewrite, <K> keep`
`ask: <A>`

Open the report with the depth: `mode: deep` or `mode: fast`, and for fast the seams it skipped
and whether a deep run is recommended, with the reason.

Nothing found: `No sediment. Reasons hold.`

Write every finding to the ledger; a candidate matching a Do not re-report line stays out of
the report. Replace the ledger's Next scope with the areas this run only spot-checked or did not
cover (deep only). Record the run in the ledger's run log, naming the untested lifecycle parts
that changed an outcome this run (see LEDGER.md).

The skill tests its own reason markers too. After a deep run, test each marker in LEDGER.md
against the repo's run log; when one has expired, tell the user that part of the skill changed no
outcome across its trial runs, and propose pruning it. Propose only: the skill is shared across
repos, so it is never edited from inside a repo's audit.

Update the patterns, following "Identifying a pattern" in LEDGER.md: trace each finding to
its origin and classify who missed it, leaving both unknown when no origin shows them. Add
findings to the evidence of the patterns they fit, reset those patterns' quiet runs, and count a
quiet run for the rest (deleting any that reach three). For every finding this run disproved,
add or extend an Audit calibration entry with the check that would have caught it. Tune any
pattern the evidence disagrees with, per LEDGER.md. Where two findings share a shape no
pattern covers, propose a new pattern. Where a pattern keeps producing findings, propose its
graduation.

**Fast:** add findings to the evidence of the patterns they fit and add calibration entries, but
count no quiet runs, and tune, propose and graduate nothing: a fast run never searched for new
shapes, so its silence is not evidence.

Report every pattern this run touched: each one it proposed, tuned or confirmed with new
evidence. Give each a one-line claim and a one-line justification, so the user can question and
refine it:

`<id> <proposed|tuned|confirmed> <shape>. missed by <user|agent|both|unknown>: <cause or unknown>.`
`  why: <what these findings share beyond appearance, with their ids>. origin: <citation or none found>.`

The justification says why the findings belong together, not what they look like. A tuned
line adds what changed. Patterns the run did not touch are not listed. These lines go at the
top of the report, right after the mode line. The reply to the user is the report's summary: it
leads with the mode line and these pattern lines, whatever file holds the full evidence. Report each new or
extended calibration entry the same way: `<id> <new|extended> <signal>. check: <check>.` The
user can answer with their own tuning of any field.

Then ask the user which findings to settle.

## 5. Settle

For the findings the user picks, put their ask verdicts to the user as question types, through
the "grilling" skill when it is available. Retiring a finding is itself a change: run it through the "sediment"
skill, which maps what the removal touches and resolves the ledger entry in the same commit,
one commit per entry. A finding the user decides to keep moves to Do not re-report, with a
reason marker written beside the code.
