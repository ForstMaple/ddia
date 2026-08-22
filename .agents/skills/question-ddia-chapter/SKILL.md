---
name: question-ddia-chapter
description: Generate a prioritized core set and optional extension set of high-value active-reading and comprehension questions from chapters in /workspaces/ddia/content/en. Use when the user wants focus questions before or during reading, a chapter quiz, retrieval practice, or help identifying the most important DDIA concepts and trade-offs.
---

# Question a DDIA Chapter

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md`. If no chapter is specified, ask which chapter.
2. Read the chapter itself, including its summary. Use the local text as authoritative.
3. Select questions by instructional value, not by easy extraction. Emphasize the chapter's central models, causal reasoning, trade-offs, and boundary conditions. Keep questions in the sequence in which their answers are substantively developed in the chapter. Do not move a question earlier because an earlier section briefly mentions, previews, or cross-references the topic.
4. Do not include answers unless the user explicitly asks for them.
5. Write the questions to `/workspaces/ddia/notes/chN.md`, creating the file and `notes/` when needed.
6. Include a closed-book recall checkpoint for the learner to complete immediately after reading and before consulting the chapter or requesting review.
7. Treat `## Concepts` as learner-owned. If creating the note, include only the
   empty heading. If the note exists, preserve the section exactly.

## Default Question Set

Create 8–12 questions in reading order, split into two explicit tiers:

- **Core:** 5–7 questions sufficient to recover the chapter's essential mental
  models. Include orientation, mechanism or distinction, application, and
  synthesis. Mark the 3–5 highest-priority core questions with `★`.
- **Extension:** 3–5 optional questions for deeper application, boundary
  conditions, easily confused ideas, or connections across sections. Do not use
  extension questions merely to exhaustively cover headings.

Label each question with the relevant chapter section. Keep questions open-ended
and answerable from the chapter. A learner who completes only the core set should
still have tested the chapter's central argument and trade-offs.

Assign each question to the section that contains the explanation needed to answer it, not to a section that only points ahead to that explanation. If a question synthesizes multiple sections, place it after the latest section required to answer it and label it with the sections it actually draws from.

Place this guidance sentence immediately below `## Focus questions`: “Answer the
core questions first in your own words; uncertainty is useful—mark anything
you’re unsure about. Extension questions are optional.”

Place `**Core questions — answer these first.**` before the first core question
and `**Extension questions — optional deeper practice.**` before the first
extension question. Keep question numbering continuous across both tiers. These
labels are generated text, not learner-authored content.

Then format every question as its own third-level heading, followed immediately by a fourth-level answer heading and blank editable space:

```markdown
### 1. ★ **[Section]** Question text

#### My answer


```

Do not create a separate `## My answers` section.

Under `## Closed-book recall`, add these editable prompts:

- What are the three most important ideas in this chapter?
- Which trade-off surprised you, and why?
- What can you now explain confidently without the book?
- What remains unclear?

Add one sentence instructing the learner to close the chapter before answering. Do not supply sample answers.

If the note already exists, replace only generated focus questions with untouched blank answer areas and untouched recall prompts. Never alter an answer the user has started, or any other user-written text. Treat all text under a question's `#### My answer` heading, up to the next third-level heading or second-level section, as the learner's answer. If preserving an answer makes question replacement or numbering ambiguous, retain that question and its number, then append new questions. Remove an obsolete `## My answers` section only when it contains no learner text. Report the clickable note path after writing.

Do not create a `### Diagnostic question` block. Remove a legacy diagnostic question block only when its `#### My answer` subsection contains no learner text; otherwise preserve the entire block exactly.

Never derive or suggest entries for `## Concepts`, even when a question names an
important term. Do not put instructions, examples, comments, bullets, or
placeholders beneath that heading. The learner will record concepts while reading
and request explanations separately.

Before writing, re-read every generated question against the relevant chapter
section. Remove stray citation or footnote numbers, repair grammar, verify section
labels and reading order, split unrelated compound tasks, and ensure the wording
does not reveal the answer. Perform this check again after merging with an
existing note.

## Adaptation

- Before reading: favor predictions and questions that create curiosity without revealing conclusions.
- During reading: align questions to section order and provide optional stopping points.
- After reading: favor retrieval, application, and comparison.
- For every set, include questions that ask why a mechanism works, when a claim fails, what is easily confused, and how a changed requirement alters a design. Combine these naturally when fewer questions are requested.
- If the user requests a shorter set, preserve conceptual coverage rather than taking the first questions.
- If the user gives a background or goal, tune scenarios and difficulty to it.

## Guardrails

- Avoid trivia, quotation recall, and questions answerable by copying one sentence.
- Avoid compound questions that hide several unrelated tasks.
- Do not smuggle the answer into the wording.
- Do not introduce facts that require sources outside the local chapter.
