# spec-first

Tools to organize project knowledge so AI agents can understand your codebase.

## Skills

This plugin provides skills for the spec lifecycle:

### spec-first:init — Create the spec structure (software)

Scaffolds a spec structure optimized for software projects. Creates a what/how two-layer structure at `.ai/spec/`:
- **what/** — behavioral rules per component (what the system must do)
- **how/** — codebase navigation per concern (how the code is organized)

Three modes:
- **Init:** Creates the structure from an existing codebase or greenfield project
- **Alignment:** Existing what/how specs — fills gaps, suggests improvements
- **Migration:** Existing non-what/how specs — presents a migration plan

### spec-first:book-init — Create the spec structure (content)

Scaffolds a spec structure optimized for content projects (books, documentation, courses). Creates a features/standards structure at `spec/`:
- **features/** — one file per deliverable (chapter, article, module)
- **standards/** — output production rules (style, formatting, terminology)

### spec-first:verify — Verify content against spec

Dispatches an independent agent (with no authoring context) to verify content against its spec. Four verification passes:
1. Acceptance criteria — binary pass/fail for every criterion
2. Constraint compliance — checks every project constraint
3. Term consistency — verifies glossary term usage
4. Internal reference accuracy — checks that references point to real targets

Works with both software (`.ai/spec/`) and book (`spec/`) layouts.

### spec-first:health — Evaluate spec freshness

Checks whether specs are stale, missing information, or structurally unsound. Two modes:
- **Staleness check:** Quick scan for stale references and outdated content
- **Structural evaluation:** Deeper assessment of findability, completeness, accuracy, and boundaries

Works with both software (`.ai/spec/`) and book (`spec/`) layouts.

## Spec structures

### Software projects (default)

```
.ai/spec/
  README.md          — entry point: scope, reading order, cross-references
  glossary.md        — domain terms (optional)
  what/              — behavioral rules and constraints per component
    system-overview.md
    <component>.md
  how/               — codebase navigation per concern
    project-structure.md
    <concern>.md
  decisions/         — architecture decision records (optional)
    NNNN-<slug>.md
```

Constraints are co-located in each what/ file's Constraints section, not in a separate file. Development conventions stay in CLAUDE.md.

### Content projects

```
spec/
  README.md          — entry point: project description, reading order
  constraints.md     — non-negotiable project rules
  architecture.md    — system structure, boundaries, data flow
  glossary.md        — canonical terms (optional)
  health-report.md   — agent-generated evaluation
  features/          — one file per deliverable
    <slug>.md
  decisions/         — architecture decision records (optional)
    NNNN-<slug>.md
  standards/         — output production rules (optional)
```

## Installation

```bash
claude plugin add joshuawilson/spec-first
```

## Usage

```
/spec-first:init          Set up spec structure for a software project
/spec-first:book-init     Set up spec structure for a content project
/spec-first:verify        Verify content against its spec
/spec-first:health        Check spec freshness and structure
```
