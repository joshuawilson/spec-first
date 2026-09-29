# {{PROJECT_NAME}}

{{DESCRIPTION}}

## Reading order

1. This file — project overview and structure guide
1. `constraints.md` — non-negotiable project rules
1. `architecture.md` — system structure and boundaries
{{IF_GLOSSARY}}1. `glossary.md` — canonical terms and definitions{{/IF_GLOSSARY}}
1. `features/` — feature specs as needed for current work
1. `decisions/README.md` — decision record format

## Spec structure

{{STRUCTURE_TREE}}

- **constraints.md** — rules every feature must respect
- **architecture.md** — how the system is structured, component boundaries, integration points
- **features/** — one file per planned unit of work, named by descriptive slug (lowercase, hyphenated)
{{IF_GLOSSARY}}- **glossary.md** — domain-specific terms that must be used consistently{{/IF_GLOSSARY}}
- **decisions/** — required decision record scaffold; add numbered records when decisions are made (append-only, superseded by new records)
{{IF_STANDARDS}}- **standards/** — output production rules (style, formatting, terminology){{/IF_STANDARDS}}
- **health-report.md** — agent-generated evaluation of spec health (do not edit manually)
