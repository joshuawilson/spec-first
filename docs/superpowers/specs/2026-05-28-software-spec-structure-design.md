# Software spec structure design

## Overview

A standardized specification file structure optimized for AI agent comprehension of software projects. The agent reading these files should be able to understand the system's behavioral rules, navigate the codebase, and implement work from Jira/issue tracker stories using the spec as context.

This design replaces the current init skill's structure (README, constraints, architecture, features/) for software projects. The current init skill becomes `book-init` and continues to serve content projects (books, documentation). The new `init` becomes the default, optimized for software.

### Design context

The original spec structure (2026-05-24) was designed for and tested on a book project. When tested on a real Kubernetes operator (lightspeed-operator), the skill migrated an existing what/how structure into the flat features/standards layout. When asked to evaluate the result, the agent itself said the restructured spec was worse — the how/ files being gone made the project harder to understand.

The lightspeed-operator had independently arrived at a what/how structure through collaborative work between a developer and Claude. That structure proved effective for real software tasks: fixing reconciliation bugs, adding new components, understanding cross-cutting concerns. This design captures why that structure works and makes it the standard for software projects.

### Why the book structure doesn't work for software

The book project's unit of work is a chapter. One file per chapter in `features/` works perfectly — everything about ch-01 is self-contained in ch-01.md. Software is different: work crosses component boundaries. Fixing a reconciliation bug requires understanding both the behavioral rules (what should happen) and the implementation architecture (how it's structured in code). A single `architecture.md` file tries to capture everything; `what/reconciliation.md` + `how/reconciliation.md` gives the agent exactly the two layers it needs.

Additionally, the `features/` directory duplicates work tracking (Jira, GitHub Issues) and creates files that become stale the moment the feature is implemented. The `standards/` directory duplicates coding conventions that already live in CLAUDE.md/AGENTS.md.

### Research context

An ETH Zurich study found that architectural overviews in context files "do not provide effective overviews" — removing an Architecture section while keeping only commands, constraints, and non-standard patterns produces the same agent behavior at lower token cost. Content agents can discover independently (directory structure, tech stack from manifest files) provides no benefit and increases inference costs by over 20%.

This aligns with the agent's self-assessment: it can read code, run find/grep, check git history. What it cannot discover are behavioral invariants, hidden coupling, design intent, and the reasoning behind code organization. The spec should contain what the agent can't figure out on its own.

## The structure

```
.ai/
  spec/
    README.md              — entry point: scope, audience, reading order, quick-start table, update guide
    constraints.md         — project-wide invariants
    glossary.md            — domain terms (optional)
    what/                  — behavioral rules per component (what the system must do)
      system-overview.md   — always present: system role, component inventory, lifecycle, integration points
      <component>.md       — one file per major component or cross-cutting concern
    how/                   — codebase navigation (how the code is organized)
      project-structure.md — always present: package responsibilities, file naming, key entry points
      <concern>.md         — one file per implementation concern
    decisions/             — architecture decision records (optional, append-only)
      NNNN-<slug>.md
```

### What's NOT in this structure (and why)

- **No `features/`** — work items are tracked in Jira/GitHub Issues/Linear. Knowledge from feature brainstorming gets folded into the relevant what/ and how/ files where the agent will actually look for it.
- **No `standards/`** — coding conventions, build commands, and tool permissions live in CLAUDE.md/AGENTS.md. No duplication.
- **No `architecture.md`** — replaced by what/system-overview.md (behavioral rules and system scope) + how/project-structure.md (codebase navigation). Two focused files instead of one unfocused one.
- **No template files** — every project's component breakdown is different. The skill uses format conventions as agent guidance, not fill-in-the-blank templates.

### Root path: `.ai/spec/`

The spec lives in `.ai/spec/`, not `spec/`. This signals "AI agent tooling" and keeps the project root clean. CLAUDE.md points to `.ai/spec/README.md` as the entry point. The `.ai/` namespace is available for future tooling (`.ai/plans/`, `.ai/memory/`, etc.).

Practical note: if a project's `.gitignore` broadly ignores dotfile directories (`.*`), the user must add `!.ai/` to ensure specs are committed.

### Naming conventions

- **File names:** lowercase, hyphenated, descriptive slugs. Example: `app-server.md`, `project-structure.md`, `deployment-generation.md`.
- **what/ and how/ files for the same concern** can share a name (e.g., `what/reconciliation.md` and `how/reconciliation.md`). The directory disambiguates.
- **Decision records:** 4-digit numbered prefix for chronological ordering. Example: `0001-chose-postgresql.md`. Numbers are append-only.

## File contents

### README.md — the entry point

```markdown
# Project Name — Specifications

One-paragraph description of the project and what these specs cover.

## Structure

| Layer | Path | Purpose |
|---|---|---|
| **what/** | `.ai/spec/what/` | Behavioral rules. What the system must do. Implementation-agnostic. |
| **how/** | `.ai/spec/how/` | Codebase navigation. How the code is organized. Implementation-specific. |

## Scope

What's covered by these specs. What's out of scope (other repos, external systems).

## Audience

AI agents. Content is optimized for precision and machine consumption.

## Quick Start

| Task | Start here |
|---|---|
| Understand the system | `what/system-overview.md` |
| Fix a bug in X | `what/x.md` + `how/x.md` |
| Add a new component | `what/system-overview.md` + `how/project-structure.md` |
| Navigate the codebase | `how/project-structure.md` |

## Conventions

- **Rule numbering:** behavioral rules are numbered sequentially within each what/ file.
- **Planned changes:** unimplemented behavior is marked with `[PLANNED]` or `[PLANNED: TICKET-XXXX]` inline next to the rule it affects.
- **Authority:** what/ specs are authoritative for behavior. how/ specs are authoritative for implementation. When they conflict, what/ wins.

## Updating this spec

- **Adding a new component:** create `what/<component>.md` with behavioral rules and `how/<component>.md` with implementation navigation. Add to the quick-start table.
- **Adding rules to an existing component:** append numbered rules to the relevant section in the what/ file. Use `[PLANNED: TICKET]` for unimplemented behavior.
- **After implementation:** remove `[PLANNED]` markers from implemented rules. Update how/ files if code structure changed.
- **When to create a new file vs. extend an existing one:** if the new concern has its own lifecycle, configuration surface, and can be understood independently, it gets its own file. If it's a capability added to an existing component, it goes in that component's file.
```

### what/ files — behavioral rules

Each what/ file describes one component or cross-cutting concern. The content follows a consistent structure:

```markdown
# Component Name

One-paragraph description of what this component is and its role in the system.

## Behavioral Rules

### Section Name

1. Rule one — a testable statement about what the system must do.
2. Rule two — another testable statement.
3. [PLANNED: OLS-1234] Rule three — designed but not yet implemented.

### Another Section

4. Rules are numbered sequentially within the file (not restarting per section).
5. Numbers are stable within a spec version but may be renumbered across major revisions.

## Configuration Surface

| Field | Type | Default | Description |
|---|---|---|---|

## Constraints

Component-specific constraints. Project-wide constraints go in constraints.md.

## Planned Changes

| Ticket | Summary |
|---|---|
| OLS-1234 | Description of planned work |
```

**Content principles:**
- Rules are testable statements, not aspirational goals. "The operator adds a finalizer before reconciling resources" — not "the operator should handle cleanup properly."
- Rules describe intended behavior, not just current behavior. Planned rules are marked explicitly.
- Configuration surface tables document the user-facing API (CRD fields, flags, env vars, config keys).
- Planned changes tables summarize all `[PLANNED]` markers in the file for quick scanning.

**what/system-overview.md** is always present and covers:
- System role and purpose (one paragraph)
- Component inventory (what the system manages)
- Lifecycle (creation, update, deletion behavior)
- Deployment model (how the system runs)
- Integration points (external systems, APIs, databases)
- System-level constraints

### how/ files — codebase navigation

Each how/ file documents one implementation concern. The content follows a consistent structure:

```markdown
# Concern Name

## Module Map

| File | Key Symbols | Responsibility |
|---|---|---|

## Data Flow

Description or ASCII diagram of how data moves through this concern.
Code-level detail: function call chains, event propagation, request paths.

## Key Abstractions

Patterns, interfaces, and design decisions in the code.
Why the code is organized this way, not just what it looks like.

## Integration Points

| Consumer | Provider | Mechanism |
|---|---|---|

## Implementation Notes

Gotchas, non-obvious behavior, things that would surprise a reader.
```

**Content principles:**
- how/ files add value only when they explain what the agent can't discover by reading the code. A module map that lists files with their responsibilities is useful. A directory tree that `find` can produce is not.
- Data flow sections should show the actual call chain or event path, not abstract architecture diagrams.
- Key abstractions sections should explain why a pattern exists (design intent), not just describe the pattern.

**how/project-structure.md** is always present and covers:
- Package/directory responsibilities
- File naming conventions
- Key entry points
- Import graph or dependency relationships (if non-obvious)

### what/ and how/ relationship

The what/ and how/ files are two views of the same system:

- what/ specs describe invariants, ordering constraints, and expected behavior. They are implementation-agnostic.
- how/ specs describe code structure, function signatures, patterns, and file locations. They are implementation-specific.

When implementing a change: read the what/ spec first to understand required behavior, then read the how/ spec to find the implementation location.

If a how/ spec contradicts a what/ spec, the what/ spec is authoritative and the implementation should be updated to match.

Not every what/ file needs a corresponding how/ file, and vice versa. A cross-cutting concern like security may have a what/ spec (security rules) but no dedicated how/ file (security is implemented across multiple packages). A how/ file like project-structure.md has no corresponding what/ file (it describes the codebase, not behavior).

### constraints.md — project-wide invariants

Same role as in the current design. Non-negotiable rules that every component must respect. The test for inclusion: if an agent violates this rule, is the system wrong? If yes, it's a constraint. If the system still works but the code is lower quality, it belongs in CLAUDE.md as a convention.

Examples:
- "All API responses use JSON:API format."
- "Only one OLSConfig CR named 'cluster' is processed."
- "The operator must function in disconnected (air-gapped) environments."

### glossary.md (optional) — domain terms

Same role as in the current design. Created only if the project has domain-specific vocabulary an agent might misunderstand. A Kubernetes operator project needs a glossary (CRD, CR, reconciliation, finalizer have specific meanings). A generic CRUD app probably doesn't.

### decisions/ (optional) — architecture decision records

Same role as in the current design. Append-only. Records capture context, decision, consequences, and status (proposed | accepted | superseded by NNNN). Explains why the code is the way it is — something the agent cannot discover from the code itself.

## The init skill

### Mode detection

```
.ai/spec/ exists?
  → No:
      Has existing docs/code?
        → Substantial existing project: init mode (existing codebase)
        → New/empty project: init mode (greenfield)
  → Yes, with what/how layout:
      Evaluate what's there.
      Show what's missing or misaligned.
      Present alignment plan. Wait for approval.
  → Yes, with different layout (features/, standards/, etc.):
      Evaluate what's there.
      Show migration plan mapping to what/how. Wait for approval.
      Never delete originals until user confirms.
```

### Init mode: existing codebase (primary)

**Step 1: Explore.** Read whatever the project has:
- **Project files:** README.md, go.mod, package.json, pyproject.toml, Cargo.toml → project name, tech stack, dependencies
- **Agent instructions:** CLAUDE.md, AGENTS.md, .cursor/rules/ → existing constraints, conventions
- **Documentation:** docs/, .github/, existing specs → architecture docs, design docs
- **Source structure:** directory listing, package layout → component boundaries, architecture clues
- **Git history:** `git log --oneline -20` → recent activity, active areas
- **Issue tracker:** GitHub Issues via `gh`, Jira via MCP if available → planned work, domain terms

**Step 2: Infer or ask.** Infer what you can from the code. Ask targeted questions only for what you couldn't determine:
- "What are the non-negotiable rules for this project?" (invariants the code doesn't make obvious)
- "Are there domain-specific terms I should know?"
- "What would surprise an agent working in this codebase?" (hidden coupling, historical decisions, known gotchas)

**Step 3: Create .ai/spec/.** Create the following files with real content from exploration:
- `.ai/spec/README.md` — scope, structure table, quick-start table, conventions, update guide
- `.ai/spec/constraints.md` — project-wide invariants from CLAUDE.md, linter configs, CI rules, user statements
- `.ai/spec/what/system-overview.md` — system role, components, lifecycle, integration points
- `.ai/spec/what/<component>.md` — one file per major component discovered, with behavioral rules
- `.ai/spec/how/project-structure.md` — package responsibilities, file naming, entry points
- `.ai/spec/how/<concern>.md` — one file per implementation concern, with module maps and data flow

**Conditionally create:**
- `.ai/spec/glossary.md` — only if domain terms found or user provided them
- `.ai/spec/decisions/` — only if existing ADRs found or decisions worth recording

**Step 4: Update CLAUDE.md.** If CLAUDE.md exists, add a "Specs" section pointing to `.ai/spec/README.md`. If it doesn't exist, create a minimal one.

**Step 5: Commit.**

### Init mode: greenfield project

**Step 1: Ask.** Structured interview:
1. "What does this project do?"
2. "What tech stack?" (or "not decided yet")
3. "What are the non-negotiable rules?" (or "none yet")
4. "What components do you envision?"
5. "Any domain-specific terms?"

**Step 2: Create .ai/spec/.** Create:
- `.ai/spec/README.md` — scope, structure, update guide
- `.ai/spec/constraints.md` — stated constraints
- `.ai/spec/what/system-overview.md` — system role and planned components
- `.ai/spec/what/<component>.md` — behavioral rules from the user's design description

**Step 3: Skip how/ files.** No codebase to navigate yet. These get created after the first implementation pass.

**Step 4: Create CLAUDE.md and commit.**

### Alignment mode: existing spec with what/how layout

When `.ai/spec/` exists with what/ and how/ directories:

1. **Evaluate** — read all files, check for missing structural elements
2. **Present plan** — show what's missing or misaligned:
   - Missing constraints.md → offer to create
   - Missing README quick-start table → offer to add
   - Missing "Updating this spec" section → offer to add
   - Unnumbered behavioral rules → note as suggestion (don't rewrite)
   - Missing planned markers → note as suggestion
3. **Wait for approval** — do not modify existing files without consent
4. **Execute approved changes** — create missing files, add missing sections to README
5. **Commit**

### Alignment mode: existing spec with different layout

When `.ai/spec/` (or `spec/`) exists with a non-what/how layout:

1. **Inventory** — read all existing spec files
2. **Present migration mapping** — show where each file maps:
   - `architecture.md` → split into `what/system-overview.md` + `how/project-structure.md`
   - `features/<slug>.md` → fold behavioral rules into relevant `what/` files
   - `standards/<file>.md` → move conventions to CLAUDE.md, discard duplicates
   - `constraints.md` → keep as-is
3. **Wait for approval**
4. **Execute migration** — create new files, do not delete originals
5. **User confirms** — only then suggest removing old files
6. **Commit**

### What the init skill does NOT do

1. **Duplicate CLAUDE.md/AGENTS.md content.** Build commands, test commands, coding conventions stay where they are.
2. **Describe architecture the agent can read from code.** No file that just lists the directory tree or restates the tech stack from go.mod.
3. **Create feature files.** Work items live in the issue tracker, not in spec files.
4. **Create standards files.** Coding style lives in CLAUDE.md and linter configs.
5. **Invent behavioral rules.** Only capture rules that exist (in code, docs, or user statements).
6. **Use fixed template files.** The file format examples in this design are conventions embedded in SKILL.md to guide the agent running the skill. They are not separate template files that get read and filled in — every project's component breakdown is different, so the agent uses the conventions as guidance while writing project-specific content.
7. **Overwrite existing spec content without approval.** Always show a plan first.

## The spec lifecycle

The spec files are a living system manual, updated through brainstorming sessions:

1. **Init** — the init skill creates the initial structure from the existing codebase
2. **Brainstorm** — each new feature/bug/request goes through brainstorming, which updates the relevant what/ and how/ files with new behavioral rules and implementation guidance. New rules are marked `[PLANNED: TICKET]`.
3. **Track** — Jira epics and stories are created to track the implementation work. Stories reference spec rules.
4. **Implement** — the agent reads the spec to understand the system, finds `[PLANNED]` markers for the work, reads how/ files to navigate the code, and implements.
5. **Update** — after implementation, `[PLANNED]` markers are removed. how/ files are updated if code structure changed.
6. **Health** — the health skill periodically evaluates spec freshness, flags stale references and missing context.

The spec grows over time, accumulating the project's behavioral knowledge. It is not a backlog that gets consumed — it is a manual that gets richer.

## Differences from the current init skill

| Aspect | Current init (becomes book-init) | New init (software) |
|--------|----------------------------------|---------------------|
| Root path | `spec/` | `.ai/spec/` |
| Core structure | README, constraints, architecture, features/ | README, constraints, what/, how/ |
| Unit of knowledge | Feature (one file per planned work item) | Component (one file per system component/concern) |
| Templates | Fixed files with `{{PLACEHOLDER}}` markers | Format conventions as agent guidance, no template files |
| standards/ | Created if user needs output rules | Not created — coding conventions live in CLAUDE.md |
| features/ | Core directory, always created | Not created — work tracked in issue tracker |
| architecture.md | Single file covering everything | Replaced by what/system-overview.md + how/project-structure.md |
| Lifecycle | Static after init, updated manually | Living document, updated each brainstorming session |
| Update guide | None | "Updating this spec" section in README |
| Planned work markers | Feature file with `status: not-started` | `[PLANNED: TICKET]` markers inline in what/ files |
| Existing spec handling | Suggest health or offer adopt | Evaluate, present alignment/migration plan, wait for approval |

## Relationship to other skills

- **spec-first:book-init** (renamed from current init): scaffolds the features/standards structure for content projects. Unchanged.
- **spec-first:health**: evaluates spec freshness. Works with the new structure (checks what/ and how/ files for stale references, missing context). May need minor updates to recognize the new layout.
- **spec-first:verify**: verifies content against spec. Works unchanged — it verifies any content against any spec file.
- **superpowers:brainstorming**: the primary consumer. When brainstorming a new feature, reads `.ai/spec/README.md` to learn the update conventions, then updates the relevant what/ and how/ files with new behavioral rules and implementation guidance.

## Implementation order

1. Rename current `skills/init/` to `skills/book-init/` (update SKILL.md name and description)
2. Create new `skills/init/SKILL.md` implementing this design (no templates directory needed)
3. Update plugin.json if skill names are listed there
4. Update README.md to describe both skills
5. Test new init on a software project (lightspeed-operator or similar)
6. Test alignment mode on a project with existing what/how specs
7. Update health skill to recognize `.ai/spec/` with what/how layout
