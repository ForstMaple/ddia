---
name: explain-ddia-concepts
description: Explain and selectively supplement key concepts and terminology in the Concepts section of DDIA chapter notes for chapters in /workspaces/ddia/content/en. Use when the user asks to clarify, connect, compare, update, or give examples for concepts recorded while reading a chapter.
---

# Explain DDIA Concepts

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md` and its note to `/workspaces/ddia/notes/chN.md`. If the chapter cannot be inferred, ask which chapter.
2. Read the learner's existing entries under `## Concepts` as the primary scope.
   Treat omissions as likely intentional because the learner may already know the
   omitted material; do not turn the section into a chapter-wide glossary. Infer
   the appropriate depth from the learner's definitions, annotations, examples,
   and questions. Do not repeat basics they already demonstrate correctly.
3. If `## Concepts` is missing or empty, do not modify the note. Tell the user to
   record the terms or concepts there while reading and ask again afterward.
   Never invent, extract, recommend, or prefill a concept list.
4. Read the relevant chapter passages for context. Use standard technical knowledge when it makes an explanation clearer or more useful; explanations do not need to mirror the book. Do not browse unless the user explicitly requests outside context.
5. Explain every populated concept unless the user selects a subset. Match the
   depth to the learner's demonstrated knowledge: focus on missing mechanisms,
   relationships, and trade-offs rather than restating what the learner already
   understands.
6. Add a missing concept only when it is important enough that omitting it would
   leave the chapter's central mechanism or trade-off materially harder to
   understand, or when it is needed to connect the learner's recorded concepts.
   Use a high bar and add as few concepts as necessary.
7. Reorganize `## Concepts` when useful: group clearly related concepts, merge duplicates, and choose a logical learning order. Keep useful alternate names in parentheses so the learner's terminology is not lost. Do not force unrelated concepts into a group.
8. Keep all concept entries and explanations in the single `## Concepts` section; do not create a `## Concept explanations` section.
9. Preserve personal annotations, questions, examples, starred terms, and uncertainty markers while reorganizing. Preserve all content outside `## Concepts` exactly.
10. Re-read the edited note and compare it with the original. Verify that content
    outside `## Concepts` is unchanged, learner-authored material is preserved,
    no concepts or explanations are duplicated, explanations remain concise, and
    every added concept satisfies the high-importance rule. Correct any violation
    before finishing.
11. Report the clickable note path after writing.

## Explanation Format

Prefer a compact structure such as:

```markdown
### <related group, only when useful>

**<concept> (<useful alias>)** — <concise explanation, normally 1–3 sentences>.

**<related concept>** — <concise explanation>.
```

Use short paragraphs or 2–4 bullets when they are clearer. Add a compact example,
analogy, scenario, comparison, or text illustration when it makes the mechanism
materially easier to understand. Keep it subordinate to the explanation and omit
it when the concept is already clear. Add a trade-off or chapter location only
when useful. When several terms form one model, explain their relationship once
instead of repeating it for every term.

## Updating Existing Explanations

The skill may rewrite and restructure generated material inside `## Concepts`, including headings, ordering, and duplicate entries. Preserve learner-authored annotations, questions, examples, emphasis, and uncertainty markers. If generated and learner-authored text cannot be distinguished safely, retain the text and reorganize around it rather than deleting it.

Treat accurate, clear existing explanations as stable. Revise an explanation only
to correct an error, close a meaningful gap, remove duplication, or materially
improve comprehension—not merely to change wording or style. Re-running the skill
against unchanged notes and chapter content should produce no material changes.

If a legacy note contains `## Concept explanations`, move only clearly generated explanations beneath their matching entries in `## Concepts`, then remove the legacy heading if it is empty. Do not move or remove text whose authorship or matching concept is uncertain.

If a concept entry is ambiguous, use the most likely chapter-specific meaning and briefly label the assumption. Ask a concise question only when choosing the wrong meaning would materially change the explanation.

Do not add a concept merely because it is useful, related, mentioned in the
chapter, or would make a group look complete. Assume the learner intentionally
omitted concepts they already understand. Supplement the list only under the
high-importance rule in Workflow step 6 or when the user explicitly requests a
concept.

## Guardrails

- Explain mechanisms and trade-offs, not only dictionary definitions.
- Do not contradict the chapter without briefly identifying the difference, but do not imitate the book's wording or framing unnecessarily.
- Deduplicate only when entries describe the same concept; preserve useful distinctions between closely related terms.
- Do not duplicate a full chapter summary.
- Prefer one well-chosen example or illustration over several variations.
- Do not add unrelated concepts merely because they appear nearby in the chapter.
