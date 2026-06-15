# Shared Knowledge Base (`memory.md`)

*(Pending — Phase 2)*

## Purpose

`memory.yaml` is a flat key/value file suitable for narrow conventions. Some projects need a longer-form, narrative knowledge base: architecture diagrams in prose, recurring bug postmortems, tribal knowledge that doesn't fit in a YAML list.

Phase 2 introduces `memory.md` as a sibling to `memory.yaml`:

- `memory.yaml` — structured fields the agent must always consider (stack, style, constraints).
- `memory.md` — long-form Markdown that the agent may consult on demand.

## Outline (proposed)

```markdown
# Memory: <Project Name>

## Architecture
(Long-form prose. Diagrams in mermaid or ASCII.)

## Conventions Beyond Code Style
(Code review culture, deployment rituals, on-call expectations.)

## Recurring Pitfalls
(One section per known footgun, with the symptom and the fix.)

## Tribal Knowledge
(Business-domain assumptions baked into the code.)

## Decision Index
(Pointer entries that link to full ADRs stored elsewhere.)
```

## Open Questions

- Should `memory.md` be loaded on every READ, or lazily?
- Granularity: one file per topic, or one big file?
- Versioning: when do entries expire or get archived?

These questions are deferred until Phase 2 design.
