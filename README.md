# spec-first

Tools to organize project knowledge so AI agents can understand your codebase.

## Skills

This plugin provides three skills for the spec lifecycle:

### spec-first:init — Create the spec structure

Scaffolds a standardized spec structure for any project. Two modes:
- **Init:** New project — explores the codebase, asks targeted questions, creates structure with real content
- **Adopt:** Existing project — inventories scattered documentation, presents a migration plan, consolidates into the standard structure

### spec-first:verify — Verify content against spec

Dispatches an independent agent (with no authoring context) to verify content against its spec. Four verification passes:
1. Acceptance criteria — binary pass/fail for every criterion
2. Constraint compliance — checks every project constraint
3. Term consistency — verifies glossary term usage
4. Internal reference accuracy — checks that references point to real targets

### spec-first:health — Evaluate spec freshness

Checks whether specs are stale, missing information, or structurally unsound. Two modes:
- **Staleness check:** Quick scan for stale references and outdated content
- **Structural evaluation:** Deeper assessment of findability, completeness, accuracy, and boundaries

## The spec structure

```
spec/
  README.md          — entry point: project description, reading order
  constraints.md     — non-negotiable project rules
  architecture.md    — system structure, boundaries, data flow
  glossary.md        — canonical terms (optional)
  health-report.md   — agent-generated evaluation (maintained by spec-first:health)
  features/          — one file per feature
    <slug>.md        — lowercase, hyphenated, descriptive
  decisions/         — EXTENSION: architecture decision records
    NNNN-<slug>.md
  standards/         — EXTENSION: output production rules
```

## Installation

```bash
claude plugin add joshuawilson/spec-first
```

## Usage

```
/spec-first:init          Set up spec structure for this project
/spec-first:verify        Verify content against its spec
/spec-first:health        Check spec freshness and structure
```
