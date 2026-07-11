---
name: question-ddia-chapter
description: Generate high-value active-reading and comprehension questions from chapters in /workspaces/ddia/content/en. Use when the user wants focus questions before or during reading, a chapter quiz, retrieval practice, or help identifying the most important DDIA concepts and trade-offs.
---

# Question a DDIA Chapter

## Workflow

1. Resolve the requested chapter to `/workspaces/ddia/content/en/chN.md`. If no chapter is specified, ask which chapter.
2. Read the chapter itself, including its summary. Use the local text as authoritative.
3. Select questions by instructional value, not by easy extraction. Emphasize the chapter's central models, causal reasoning, trade-offs, and boundary conditions.
4. Do not include answers unless the user explicitly asks for them.
5. Write the questions to `/workspaces/ddia/notes/chN.md`, creating the file and `notes/` when needed.
6. Include a closed-book recall checkpoint for the learner to complete immediately after reading and before consulting the chapter or requesting review.

## Default Question Set

Create 8–12 questions in reading order:

- 2–3 **orienting questions** about the problem being solved and why it matters.
- 3–5 **explanation questions** requiring the learner to reconstruct a mechanism or distinction.
- 2–3 **application questions** using a small scenario or design choice.
- 1 **synthesis question** connecting multiple sections or asking when a claim stops applying.

Label each question with the relevant chapter section. Mark the 3–5 most important questions with `★`. Keep questions open-ended and answerable from the chapter.

Place this guidance sentence immediately below `## Focus questions`: “Answer in your own words; uncertainty is useful—mark anything you’re unsure about.”

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
