---
name: verify
description: Use when content has been produced and needs independent verification against its spec. Also use when asked to verify, check, or validate work against requirements. Dispatches a fresh agent with no authoring context.
---

# Spec Verify

Dispatch an independent agent to verify content against its spec. The reviewer gets the spec, the content, constraints, and glossary — but has no context from the session that produced the content.

**Announce at start:** "I'm using the spec-first:verify skill to independently verify this content against its spec."

## How to invoke

The user specifies what to verify. Examples:
- "verify chapter 1 against its spec"
- "verify the auth module"
- "run spec-verify on features/webhook-registration.md"

If the user specifies content but not the spec, find the matching spec. Check `.ai/spec/what/` first (software layout), then `spec/features/` (book layout). For software projects, the spec may be spread across multiple what/ files — identify which what/ file contains behavioral rules relevant to the content being verified.

## Steps

### Step 1: Identify inputs

Find four files:
1. **Spec file(s):** the spec that corresponds to the content. In software projects (`.ai/spec/what/`), this may be one or more what/ files containing behavioral rules for the component being verified. In book projects (`spec/features/`), this is the feature spec file.
2. **Content:** the file being verified
3. **Constraints:** `.ai/spec/constraints.md` or `spec/constraints.md` (whichever exists)
4. **Glossary:** `.ai/spec/glossary.md` or `spec/glossary.md` (whichever exists)

If constraints.md or glossary.md don't exist, skip those verification passes.

### Step 2: Run verification

Read ALL four input files in full. Then read `reviewer-prompt.md` in this skill's directory for the verification instructions.

Dispatch a subagent using the Agent tool with the reviewer prompt. In the prompt, include the FULL TEXT of all four files (do not summarize, excerpt, or reference file paths — the reviewer needs the actual content inline). The subagent must have NO context from the current session — it is a fresh agent.

Wait for the subagent to complete and capture its full output.

**CRITICAL:** The subagent's output IS the verification report. Do not discard it or summarize it. You need the complete pass/fail results.

### Step 3: Save and present report

1. Determine the spec root (`.ai/spec/` if it exists, otherwise `spec/`). Create the verification directory if needed: `mkdir -p <spec-root>/verification`
2. Save the subagent's verification report to `<spec-root>/verification/<slug>-report.md` — the REPORT output, not the prompt
3. Present the summary and any FAIL/VIOLATION findings in conversation

**Common mistake:** Do not save the reviewer PROMPT as the report. The report is the reviewer's OUTPUT — the pass/fail table, constraint checks, term checks, and reference checks.

## What the reviewer checks

Four passes (details in reviewer-prompt.md):
1. **Acceptance criteria** — every `- [ ]` criterion: PASS or FAIL with evidence
2. **Constraint compliance** — every constraint: PASS or VIOLATION
3. **Term consistency** — every glossary term: correct usage or inconsistency
4. **Internal reference accuracy** — every reference to other project parts: valid or broken

## What the reviewer does NOT check

- Output quality (style, readability — that's standards/)
- Whether the approach is sound (that's human judgment)
- Formatting and conventions (that's standards/)
