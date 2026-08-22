# DDIA Learning Skills

These skills form a reading, retrieval, feedback, and spaced-review loop around
the chapters in `content/en/` and the working notes in `notes/`.

## Recommended workflow

| Stage | Skill | What it does |
| --- | --- | --- |
| Before reading | `$preview-ddia-chapter` | Builds a spoiler-light mental map, terms to notice, prerequisites, and a reading strategy. |
| Before or during reading | `$question-ddia-chapter` | Creates 5–7 core questions plus 3–5 optional extension questions in reading order. |
| While reading or afterward | `$explain-ddia-concepts` | Turns the terms recorded under `## Concepts` into a compact reference with mechanisms, relationships, trade-offs, and useful examples. |
| After answering | `$review-ddia-answers` | Reviews answers with a minimal correction cue, lets you revise, and escalates to a model answer only when needed. It also creates an application challenge and spaced-review checkpoints. |
| When review is due | `$review-ddia-due` | Finds due checkpoints, quizzes you without hints, evaluates recall, and adapts the next review interval. |

The usual chapter loop is:

```text
preview → core questions → read and take notes → closed-book recall
        → answer review → revision → application → spaced review
```

Extension questions are optional. Complete them when the chapter is especially
important, the core questions expose uncertainty, or you want deeper design
practice.

## Example prompts

### Preview a chapter

> Use `$preview-ddia-chapter` to prepare me for chapter 5.

This creates or updates only the generated `## Before reading` material in the
chapter note.

### Generate questions

> Use `$question-ddia-chapter` to create focus questions for chapter 5.

Answer the core questions first. Mark uncertainty explicitly; incomplete reasoning
is useful evidence for the later review.

### Build the concept reference

Record unfamiliar or useful terms under `## Concepts`, then ask:

> Use `$explain-ddia-concepts` to explain the concepts I recorded for chapter 5.

The resulting section is intended as a quick reference. Add terms you have heard
before but want to understand better; follow external resources later when a term
is especially important or interesting.

### Review answers and revise

> Use `$review-ddia-answers` to review my chapter 5 answers.

For an important gap, the first pass gives a targeted cue and marks the answer
`(revise)`. Edit the answer in place, then ask:

> Reassess my revised chapter 5 answers.

A model answer is withheld until after a revision attempt unless you explicitly
request one.

### Run due reviews

> Use `$review-ddia-due` to tell me what is due.

or:

> Use `$review-ddia-due` to quiz me on the next due checkpoint.

By default, one session contains one checkpoint with no more than four questions.
After your answers are evaluated, weak material returns sooner and strong material
is scheduled farther out.

## Note ownership

- Skills may update their own generated sections and status markers.
- Your answers, annotations, examples, questions, and concept choices are
  learner-owned and should be preserved.
- `## Concepts` is a reference area you populate intentionally; preview and
  question generation never prefill it.
- Completed spaced-review checkpoints and their answers remain as learning
  history.
