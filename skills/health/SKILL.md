---
name: health
description: Use when starting a session and wanting to check spec freshness, after completing a milestone to evaluate spec structure, or when asked to assess spec health. Reports issues in the spec's health-report.md without modifying spec files.
---

# Spec Health

Evaluate the health of a project's spec structure. Check for stale references, missing context, structural concerns, and findability issues. Report findings in `health-report.md` (overwritten each evaluation) and in conversation.

**Announce at start:** "I'm using the spec-first:health skill to evaluate spec health."

## Layout detection

Before evaluating, determine which spec layout is present:

1. Check for `.ai/spec/what/` — if found, this is the **software layout**. Spec root is `.ai/spec/`.
2. Check for `spec/features/` — if found, this is the **book layout**. Spec root is `spec/`.
3. Check for `spec/README.md` or `.ai/spec/README.md` — use whichever exists as spec root.
4. If neither exists, report "No spec structure found. Run spec-first:init to create one." and stop.

Use `SPEC_ROOT` below to mean whichever root was detected.

## Two evaluation modes

```dot
digraph health {
    "What triggered this?" [shape=diamond];
    "Staleness check\n(~30 seconds)" [shape=box];
    "Structural evaluation\n(~5 minutes)" [shape=box];
    "Write health-report.md" [shape=box];

    "What triggered this?" -> "Staleness check\n(~30 seconds)" [label="session start\nquick check"];
    "What triggered this?" -> "Structural evaluation\n(~5 minutes)" [label="post-milestone\ndeeper review"];
    "Staleness check\n(~30 seconds)" -> "Write health-report.md";
    "Structural evaluation\n(~5 minutes)" -> "Write health-report.md";
}
```

### Staleness check (~30 seconds)

Quick scan for obvious issues. Run at session start or when asked for a quick check.

1. Read `SPEC_ROOT/constraints.md`. Does it reference files, modules, or components that still exist? Check by looking at the codebase.

2. **Software layout:** Read what/ files. Do behavioral rules reference components, modules, or integration points that still exist? Do any `[PLANNED: TICKET]` markers reference tickets that have been completed? Check how/ files — do module maps reference files that still exist?

   **Book layout:** Read `SPEC_ROOT/architecture.md`. Do references still exist? Read feature files in `SPEC_ROOT/features/`. Do any have `depends-on` entries that reference features with status `complete` or features that no longer exist?

3. Check `SPEC_ROOT/health-report.md` timestamp (if it exists). Has the codebase changed significantly since the last evaluation? Run `git log --oneline -10` and compare dates.

4. If issues found: overwrite health-report.md and report. If clean: say "spec health check: no issues found" and move on.

### Structural evaluation (~5 minutes)

Deeper assessment. Run after completing a feature or milestone, or when asked for a thorough evaluation.

Check all of the following:

1. **Findability:** Read the spec structure. Is information organized so an agent can find what it needs? Are related concepts in the same file or scattered across files? Flag anything that required hunting across multiple files.

2. **Completeness:** Are there obvious gaps?
   - **Software layout:** Does each major component in the codebase have a what/ spec? Does each significant implementation pattern have a how/ spec? Are constraints.md rules comprehensive?
   - **Book layout:** Does constraints.md cover the project's actual constraints? Does architecture.md describe the current architecture? Are there features in the codebase with no corresponding spec in features/?

3. **Accuracy:** Does any spec file say something that contradicts the current codebase? Check a sample of claims against the actual code structure. Check that constraints.md rules are still accurate.

4. **Boundaries:** Does any file try to cover too many concerns? Is the same information in two places? Has any file grown large enough that it should be split?
   - **Software layout:** Do what/ and how/ files maintain proper separation (behavioral rules vs. code navigation)? Is any behavioral rule in a how/ file or vice versa?

5. **Structure recommendations:** Based on this evaluation, should any file be split, merged, created, or reorganized? State specific recommendations.

## Output

Overwrite `SPEC_ROOT/health-report.md` (not append — only the latest evaluation matters):

```markdown
# Spec health report

Last evaluated: <date>
Trigger: <staleness-check | post-milestone: feature-name>
Layout: <software (.ai/spec/) | book (spec/)>

## Stale
<references to things that no longer exist, or "none">

## Missing
<context gaps discovered, or "none">

## Structural concerns
<files that cover too many concerns, duplicated info, or "none">

## Findability issues
<information that was hard to locate, or "none">

## No issues
<confirmation of what was checked and found current>
```

Also present findings in conversation.

## What this skill does NOT do

- Does not modify spec files (only reports — human decides whether to act)
- Does not verify content against spec (that's spec-first:verify)
- Does not create spec structure (that's spec-first:init)
- Does not run automatically — user or workflow invokes it
