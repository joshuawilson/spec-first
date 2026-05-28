# {{PROJECT_NAME}}

{{DESCRIPTION}}

## Reading order

1. This file — project overview and structure guide
2. `constraints.md` — non-negotiable project rules
3. `architecture.md` — system structure and boundaries
{{IF_GLOSSARY}}4. `glossary.md` — canonical terms and definitions{{/IF_GLOSSARY}}
5. `features/` — feature specs as needed for current work

## Spec structure

{{STRUCTURE_TREE}}

- **constraints.md** — rules every feature must respect
- **architecture.md** — how the system is structured, component boundaries, integration points
- **features/** — one file per planned unit of work, named by descriptive slug (lowercase, hyphenated)
{{IF_GLOSSARY}}- **glossary.md** — domain-specific terms that must be used consistently{{/IF_GLOSSARY}}
{{IF_DECISIONS}}- **decisions/** — architecture decision records (append-only, never edited, superseded by new records){{/IF_DECISIONS}}
{{IF_STANDARDS}}- **standards/** — output production rules (style, formatting, terminology){{/IF_STANDARDS}}
- **health-report.md** — agent-generated evaluation of spec health (do not edit manually)
