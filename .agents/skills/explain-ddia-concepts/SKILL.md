---
name: explain-ddia-concepts
description: Explain key concepts and terminology that a learner records while reading chapters in /workspaces/ddia/content/en. Use when the user asks to clarify, connect, compare, or give examples for entries in the Concepts section of a DDIA chapter note.
---

# Explain DDIA Concepts

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md` and its note to `/workspaces/ddia/notes/chN.md`. If the chapter cannot be inferred, ask which chapter.
2. Read the learner's entries under `## Concepts`. If the section is empty, tell the user to add the terms or concepts they want explained; do not invent a list.
3. Read the relevant chapter passages. Treat the local chapter as authoritative and do not browse unless the user explicitly requests outside context.
4. Explain every populated concept unless the user selects a subset.
5. Write explanations under `## Concept explanations`, creating that section immediately after `## Concepts` if needed.
6. Preserve the learner's concept entries and all other user-written text exactly.
7. Report the clickable note path after writing.

## Explanation Format

For each concept, use:

```markdown
### <concept as written by the learner>

**Plain-language meaning:** <concise explanation>

**Why it matters here:** <role in the chapter's argument or design trade-off>

**Example:** <small concrete example>

**Easy to confuse with:** <comparison when useful; otherwise omit>

**Chapter location:** <actual section heading>
```

Keep each explanation proportional to the concept: normally 1–3 sentences per field. When several terms form one model, explain their relationship once and cross-reference it instead of repeating the same material.

## Updating Existing Explanations

Replace only explanations previously generated under `## Concept explanations`. Preserve any learner edits, annotations, questions, or examples in that section. If generated text cannot be distinguished safely from learner text, append a new explanation rather than replacing existing content.

If a concept entry is ambiguous, explain the most likely chapter-specific meaning and state the interpretation. If two meanings are materially plausible, ask one concise question before writing.

## Guardrails

- Explain mechanisms and trade-offs, not only dictionary definitions.
- Distinguish the chapter's claims from additional interpretation.
- Do not silently correct or rewrite the learner's concept notes.
- Do not duplicate a full chapter summary.
- Do not add unrelated concepts merely because they appear nearby in the chapter.
