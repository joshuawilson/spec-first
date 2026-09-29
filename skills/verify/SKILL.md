---
name: verify
description: Use when content has been produced and needs independent verification against its spec. Also use when asked to verify, check, or validate work against requirements. Runs a fresh reviewer with no authoring context.
---

# Spec Verify

Dispatch an independent agent to verify content against its spec. The reviewer gets the spec, the content, constraints, and glossary — but has no context from the session that produced the content.

**Announce at start:** "I'm using the spec-first verify skill to independently verify this content against its spec."

## How to invoke

The user specifies what to verify. Examples:
- "verify chapter 1 against its spec" (book layout)
- "verify the auth module against its spec" (software layout)
- "verify features/webhook-registration.md" (book layout)
- "verify the reconciler how/ file is accurate" (software layout — how/ verification)

If the user specifies content but not the spec, find the matching spec. Check `.ai/spec/what/` first (software layout), then `spec/features/` (book layout). For software projects, the spec may be spread across multiple what/ files — identify which what/ file contains behavioral rules relevant to the content being verified.

## Layout detection

1. Check for `.ai/spec/what/` — if found, this is the **software layout**.
2. Check for `spec/features/` — if found, this is the **book layout**.

The layout determines which reviewer prompt to use and what inputs to gather.

## Steps

### Step 1: Identify inputs

**Book layout** — find four files:
1. **Spec file:** the feature spec in `spec/features/`
2. **Content:** the file being verified
3. **Constraints:** `spec/constraints.md` (if it exists)
4. **Glossary:** `spec/glossary.md` (if it exists)

**Software layout** — find:
1. **what/ file(s):** one or more what/ files containing behavioral rules for the component being verified. Include the Constraints section within each.
2. **Content:** the source code file(s) or module being verified against the spec
3. **how/ file(s):** the corresponding how/ file(s) if the verification includes code organization accuracy
4. **constraints.md:** `.ai/spec/constraints.md` (if it exists — cross-cutting project rules)
5. **Glossary:** `.ai/spec/glossary.md` (if it exists)

If constraints.md or glossary.md don't exist, skip those verification passes.

### Step 2: Run verification

Read ALL input files in full. Then read the appropriate reviewer prompt from this skill's directory:
- **Book layout:** `reviewer-prompt.md`
- **Software layout:** `reviewer-prompt-software.md`

Build a single reviewer prompt using the appropriate template. Include the FULL TEXT of all input files inline (do not summarize, excerpt, or substitute file paths for their contents). Label each input with its path so evidence can be cited. For large codebases, scope the verification with the user rather than silently omitting files. The reviewer must have NO conversation history from the authoring session.

Dispatch using the available harness:

- **Claude Code:** Use the Agent tool to start a fresh reviewer with this prompt. Wait for it and capture its complete output.
- **Pi:** Start a separate Pi process, **not** another turn in the current session. Write the full prompt to a private temporary file outside the repository (for example, a file created by `mktemp`, with permissions limited to the current user). Then run `pi --print --no-session --no-context-files --no-skills --tools read,grep,find,ls "@$prompt_file" > "$output_file"` from the project root, where both variables hold private temporary file paths. `--no-session` avoids authoring history; `--no-context-files` and `--no-skills` keep unrelated instructions out of the reviewer context. The reviewer has read-only tools so it can check referenced files, symbols, and call chains beyond the inline inputs. Check the exit status; read the **entire** captured output and save it only if the process succeeded and produced a report. Remove both temporary files afterward, including on failure. Do not pass the full prompt as a shell argument or expose it in command logs. If a model must be selected explicitly, use Pi's `--provider`/`--model` options with a model available to the user (including an OpenAI model if desired).
- **Neither dispatch path available:** Explain that independent verification cannot run in this environment; do not substitute a review in the current authoring session and label it independent.

**CRITICAL:** The fresh reviewer's output IS the verification report. Do not discard it, replace it with the prompt, or summarize it when saving. Capture the complete pass/fail results.

### Step 3: Save and present report

1. Determine the spec root (`.ai/spec/` if it exists, otherwise `spec/`). Create the verification directory if needed: `mkdir -p <spec-root>/verification`
2. Save the subagent's verification report to `<spec-root>/verification/<slug>-report.md` — the REPORT output, not the prompt
3. Present the summary and any FAIL/VIOLATION findings in conversation

**Common mistake:** Do not save the reviewer PROMPT as the report. The report is the reviewer's OUTPUT — the pass/fail table, constraint checks, term checks, and reference checks.

## What the reviewer checks

### Book layout (details in reviewer-prompt.md)

Four passes:
1. **Acceptance criteria** — every `- [ ]` criterion: PASS or FAIL with evidence
2. **Constraint compliance** — every constraint: PASS or VIOLATION
3. **Term consistency** — every glossary term: correct usage or inconsistency
4. **Internal reference accuracy** — every reference to other project parts: valid or broken

### Software layout (details in reviewer-prompt-software.md)

Four passes:
1. **Behavioral rules compliance** — every numbered rule in the what/ file: does the code comply? PASS or FAIL with code evidence
2. **Constraint compliance** — constraints from the what/ file's Constraints section AND constraints.md: PASS or VIOLATION
3. **how/ accuracy** — module map entries: do referenced files and symbols exist? Do data flow descriptions match actual call chains?
4. **Cross-reference accuracy** — what/ ↔ how/ references, decision references, [PLANNED] ticket references: valid or broken

## What the reviewer does NOT check

- Output quality (style, readability)
- Whether the approach is sound (that's human judgment)
- Formatting and conventions (that's the project's agent instruction file / linter territory)
