# spec-first

Tools to organize project knowledge so AI agents can understand your codebase.

## Installation

### Claude Code

Inside Claude Code, add the marketplace and install the plugin:

```
/plugin marketplace add joshuawilson/spec-first
/plugin install spec-first@spec-first-marketplace
```

Run `/plugin` to check the **Installed** tab, or try `/spec-first:init`. If needed, use `/reload-plugins` after installing.

### Pi (including OpenAI models)

Install the repository as a Pi package:

```bash
pi install git:github.com/joshuawilson/spec-first
```

Alternatively, if you have a local checkout, run `pi install /path/to/spec-first`. Pi discovers the skill directories in `skills/`; the Claude Code plugin metadata is not needed by Pi. Run `/reload` in an active Pi session after installing or editing skills.

## Usage

| Task | Claude Code | Pi |
|---|---|---|
| Set up software specs | `/spec-first:init` | `/skill:init` |
| Set up content specs | `/spec-first:book-init` | `/skill:book-init` |
| Check spec health | `/spec-first:health` | `/skill:health` |
| Verify against a spec | `/spec-first:verify` | `/skill:verify` |

Start with the init skill for your project type. It detects what you already have and guides you through creating or migrating the spec structure. The spec formats and review criteria are the same with Claude and OpenAI models; verification uses the host's available mechanism to run an independent reviewer.

## Skills

### spec-first:init — Create the spec structure (software)

Scaffolds a spec structure optimized for AI agent comprehension. Creates a what/how two-layer structure at `.ai/spec/`:
- **what/** — system contracts: behavioral rules, APIs, data formats, configuration surface
- **how/** — codebase map: file layout, call graphs, patterns, key abstractions

Four modes:
- **Init (existing codebase):** Explores the project, infers what it can, asks about the rest, creates the spec structure
- **Init (greenfield):** Structured interview, creates what/ specs, planned how/ specs, decisions scaffolding, and ARCHITECTURE.md from the proposed design; revises them after implementation
- **Alignment:** Existing what/how specs — checks structural conformance, detects unspecced components, offers to fill gaps
- **Migration:** Existing non-what/how specs (features/, standards/) — presents a migration plan

Also supports multi-repo and monorepo projects with a layered parent/child spec structure.

### spec-first:book-init — Create the spec structure (content)

Scaffolds a spec structure optimized for content projects (books, documentation, courses). Creates a features/standards structure at `spec/`:
- **features/** — one file per deliverable (chapter, article, module)
- **decisions/** — required scaffold for decision records, with a README even before the first record
- **standards/** — output production rules (style, formatting, terminology), if needed

### spec-first:health — Evaluate spec freshness

Checks whether specs are stale, missing information, or structurally unsound. Two modes:
- **Staleness check (~30s):** Dead file/symbol references in module maps, `[PLANNED]` markers with completed tickets, `[DONE]` markers ready for cleanup, spec files that haven't changed while their code has
- **Structural evaluation (~5m):** Findability, completeness (unspecced components, missing decisions/, missing ARCHITECTURE.md), accuracy (spec claims vs code), what/how boundary violations, lifecycle hygiene (rule numbering, marker cleanup), multi-repo routing index completeness

Works with both software (`.ai/spec/`) and book (`spec/`) layouts.

### spec-first:verify — Verify content against spec

Dispatches an independent agent (with no authoring context) to verify content against its spec.

**Book layout** — four passes:
1. Acceptance criteria — binary pass/fail for every `- [ ]` criterion
2. Constraint compliance — checks every project constraint
3. Term consistency — verifies glossary term usage
4. Internal reference accuracy — checks that references point to real targets

**Software layout** — four passes:
1. Behavioral rules compliance — every numbered rule: does the code comply?
2. Constraint compliance — from what/ Constraints sections and constraints.md
3. how/ accuracy — module maps, data flow descriptions, key abstractions vs actual code
4. Cross-reference accuracy — what/ ↔ how/ references, decision references, ticket references

## Spec structure

### Software projects (default)

```
.ai/spec/
  README.md            — entry point: scope, reading order, cross-references
  constraints.md       — cross-cutting project rules (optional)
  glossary.md          — domain terms (optional)
  what/                — system contracts per component
    system-overview.md
    <component>.md
  how/                 — codebase map per concern
    project-structure.md
    <concern>.md
  decisions/           — architecture decision records
    README.md
    NNNN-<slug>.md
```

Component-specific constraints go in each what/ file's Constraints section. Cross-cutting rules that span the whole project (git workflow, namespace conventions) go in `constraints.md`. Development conventions stay in the project's agent instruction file (`AGENTS.md` or `CLAUDE.md`).

The `decisions/` directory is always created as scaffolding — architectural decisions happen from day one and need a place to be captured.

### Content projects

```
spec/
  README.md            — entry point: project description, reading order
  constraints.md       — non-negotiable project rules
  architecture.md      — system structure, boundaries, data flow
  glossary.md          — canonical terms (optional)
  health-report.md     — agent-generated evaluation
  features/            — one file per deliverable
    <slug>.md
  decisions/           — required decision record scaffold
    README.md          — format and purpose
    NNNN-<slug>.md      — actual decisions (when found)
  standards/           — output production rules (optional)
```

### Multi-repo / monorepo projects

Use a layered spec structure:
- **Parent level** (workspace or monorepo root): `.ai/spec/` covers cross-repo concerns. what/ files describe end-to-end flows. how/ contains a routing index mapping concerns to repos and their spec files.
- **Child level** (each repo or service): its own `.ai/spec/` with repo-specific what/ and how/ files.

Child specs are authoritative for internal behavior. Parent specs are authoritative for cross-boundary integration contracts.

## Conventions

- **Rule numbering:** stable identifiers — gaps on removal, sub-numbers (16a, 16b) on insertion
- **Planned changes:** `[PLANNED: TICKET]` → `[DONE: TICKET]` → remove marker. Planned Changes tables use strikethrough (`~~TICKET~~`) for completed entries.
- **what/ vs how/ boundary:** would this content change if the codebase were rewritten in a different language? If no → what/. If yes → how/.
