---
name: explain-ddia-concepts
description: Explain key concepts and terminology that a learner records while reading chapters in /workspaces/ddia/content/en. Use when the user asks to clarify, connect, compare, or give examples for entries in the Concepts section of a DDIA chapter note.
---

# Explain DDIA Concepts

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md` and its note to `/workspaces/ddia/notes/chN.md`. If the chapter cannot be inferred, ask which chapter.
2. Read the learner's entries under `## Concepts`. If the section is empty, tell the user to add the terms or concepts they want explained; do not invent a list.
3. Read the relevant chapter passages for context. Use standard technical knowledge when it makes an explanation clearer or more useful; explanations do not need to mirror the book. Do not browse unless the user explicitly requests outside context.
4. Explain every populated concept unless the user selects a subset.
5. Reorganize `## Concepts` when useful: group clearly related concepts, merge duplicates, and choose a logical learning order. Keep useful alternate names in parentheses so the learner's terminology is not lost. Do not force unrelated concepts into a group.
6. Keep all concept entries and explanations in the single `## Concepts` section; do not create a `## Concept explanations` section.
7. Preserve personal annotations, questions, examples, starred terms, and uncertainty markers while reorganizing. Preserve all content outside `## Concepts` exactly.
8. Report the clickable note path after writing.

## Explanation Format

Prefer a compact structure such as:

```markdown
### <related group, only when useful>

**<concept> (<useful alias>)** — <concise explanation, normally 1–3 sentences>.

**<related concept>** — <concise explanation>.
```

Use short paragraphs or 2–4 bullets when they are clearer. Add an example, comparison, trade-off, or chapter location only when it materially improves understanding. When several terms form one model, explain their relationship once instead of repeating it for every term.

## Updating Existing Explanations

The skill may rewrite and restructure generated material inside `## Concepts`, including headings, ordering, and duplicate entries. Preserve learner-authored annotations, questions, examples, emphasis, and uncertainty markers. If generated and learner-authored text cannot be distinguished safely, retain the text and reorganize around it rather than deleting it.

If a legacy note contains `## Concept explanations`, move only clearly generated explanations beneath their matching entries in `## Concepts`, then remove the legacy heading if it is empty. Do not move or remove text whose authorship or matching concept is uncertain.

If a concept entry is ambiguous, use the most likely chapter-specific meaning and briefly label the assumption. Ask a concise question only when choosing the wrong meaning would materially change the explanation.

## Guardrails

- Explain mechanisms and trade-offs, not only dictionary definitions.
- Do not contradict the chapter without briefly identifying the difference, but do not imitate the book's wording or framing unnecessarily.
- Deduplicate only when entries describe the same concept; preserve useful distinctions between closely related terms.
- Do not duplicate a full chapter summary.
- Do not add unrelated concepts merely because they appear nearby in the chapter.
