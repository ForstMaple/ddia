---
name: review-ddia-due
description: Run and evaluate adaptive spaced-repetition reviews from DDIA chapter notes in /workspaces/ddia/notes. Use when the learner asks what is due, wants to review or be quizzed on due DDIA material, submits answers to a spaced-review checkpoint, or wants the next review interval adjusted from their performance.
---

# Review Due DDIA Material

## Workflow

1. Use the current date in the learner's timezone. Scan
   `/workspaces/ddia/notes/ch*.md`, or only the requested chapter, for unchecked
   dated checkpoints under `## Spaced review`.
2. Treat a checkpoint as due when its date is today or earlier. Preserve completed
   checkpoints, learner responses, and all content outside `## Spaced review`
   exactly.
3. When asked what is due, report the due checkpoints in oldest-first order. If
   none are due, report the next scheduled checkpoint. Do not create review work
   merely because nothing is due unless the learner asks for extra practice.
4. When asked to start or run a review, select one due checkpoint by default:
   choose the oldest checkpoint, breaking ties in favor of material previously
   assessed as weak or uncertain. Present its questions without answers or hints.
   Ask the learner to answer from memory in the note or conversation. Do not
   evaluate until the learner responds. Honor requests to review a particular
   chapter or several due checkpoints.
5. When the learner submits answers, read the corresponding chapter in
   `/workspaces/ddia/content/en/chN.md`, the checkpoint questions, relevant prior
   inline reviews, and the answers. Treat the chapter as authoritative. If the
   learner answered in the conversation, copy the response verbatim beneath
   `#### My review answers` so the note retains the attempt; create that heading
   when it is absent.
6. Add concise feedback beneath that checkpoint's answer block. For each answer,
   state `Remembered`, `Partial`, or `Missed`, identify the decisive reasoning,
   and correct only the smallest important gap. End with one brief retrieval or
   application prompt for any weak idea; omit its answer.
7. Mark the checkpoint complete only after answers have been evaluated. Record an
   overall outcome of `Strong`, `Developing`, or `Needs relearning`, then adapt
   the next checkpoint as described below.
8. Re-read the edited `## Spaced review` section. Verify that learner text and
   completed checkpoints are unchanged, the next checkpoint is not duplicated,
   and no answer appears in a future quiz. Report the clickable note path.

## Checkpoint Format

Support reasonable legacy formats, but create or normalize untouched future
checkpoints to this form:

```markdown
### YYYY-MM-DD

- [ ] Review complete

1. Retrieval question
2. Application or comparison question

#### My review answers


```

After evaluation, change only `[ ]` to `[x]`, retain the learner's answers, and
append:

```markdown
#### Review result

**Outcome:** Strong | Developing | Needs relearning

1. **Remembered | Partial | Missed:** Concise feedback.

**Retrieve next:** One unanswered prompt, only when useful.
```

## Adaptive Scheduling

Base the outcome on conceptual understanding rather than wording:

- **Strong:** The important mechanisms and trade-offs were recalled with at most
  minor omissions. Keep an existing later untouched checkpoint when it is
  reasonably spaced; otherwise schedule the next review in 30–60 days.
- **Developing:** The main model is present but one or more important gaps remain.
  Schedule targeted retrieval in 5–7 days.
- **Needs relearning:** A foundational mechanism is missing or materially wrong.
  Schedule a focused checkpoint in 1–2 days.

Create 2–4 questions for the next checkpoint. Target mistakes, prior uncertainty,
starred questions, and foundational relationships. Change the wording or scenario
instead of repeating questions verbatim. When a suitable untouched future
checkpoint already exists, update its date and questions instead of adding a
duplicate; preserve later checkpoints that remain useful. Never alter a completed
checkpoint or learner response.

## Guardrails

- Keep a session small by default: one checkpoint and at most four questions.
- Test retrieval, causal reasoning, boundary conditions, and application; avoid
  trivia and quotation recall.
- Do not expose answers, chapter summaries, concept definitions, or corrective
  hints before the learner attempts the questions.
- Do not mark a checkpoint complete merely because its date has passed.
- Do not browse unless the learner explicitly requests outside context.
