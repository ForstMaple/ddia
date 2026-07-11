---
name: preview-ddia-chapter
description: Prepare a concise, spoiler-light preview of a chapter in /workspaces/ddia/content/en before the user reads it. Use when the user asks what to expect, requests a chapter roadmap, wants prerequisites or key themes, or wants help preparing to read a DDIA chapter.
---

# Preview a DDIA Chapter

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md`. If no chapter is specified, ask which chapter.
2. Read the chapter headings, introduction, summary, and enough body text to identify its argument and important contrasts.
3. Treat the local chapter as authoritative. Do not browse unless the user explicitly requests outside context.
4. Produce a spoiler-light orientation rather than a substitute for reading.
5. Write the result to `/workspaces/ddia/notes/chN.md`, creating `notes/` when needed.

## Note Format

Create this Markdown structure:

```markdown
# Chapter N — <chapter title>

## Before reading

### Why this chapter matters
### Mental map
### Terms to notice
### Watch for
### Useful prior knowledge
### Reading strategy
### Diagnostic question
#### My answer

## Focus questions

## Closed-book recall

### Three most important ideas
### A surprising trade-off
### What I can explain confidently
### What remains unclear

## Concepts

## Concept explanations

## Review

## Application challenge

## Spaced review
```

Fill only `Before reading`. Give 2–4 sentences for why the chapter matters, a 3–6-step mental map, 4–8 terms to notice, 2–4 important distinctions, genuine prerequisites, and a reading strategy. End with one unanswered diagnostic question followed by an empty `#### My answer` subsection. Leave the other sections as editable scaffolding.

If the note already exists, update only generated text in `Before reading`. Preserve the learner's diagnostic answer and all other user-written text exactly. Never replace text under `#### My answer`; if it is populated, retain the question associated with it unless the user explicitly requests a new one. Report the clickable note path after writing.

## Guardrails

- Avoid exhaustive summaries and detailed conclusions.
- Explain the shape of the problem without resolving every trade-off.
- Distinguish the authors' claims from your interpretation.
- Refer to headings by their actual names so the learner can navigate the chapter.
- Match depth to the learner's stated background; otherwise assume working software-engineering knowledge.
