---
name: review-ddia-answers
description: Review and reassess a learner's answers to questions about chapters in /workspaces/ddia/content/en using revision-first feedback and escalating hints. Use when the user submits answers, asks for grading or feedback, wants misconceptions corrected, revises an answer, or requests targeted follow-up teaching.
---

# Review DDIA Answers

## Workflow

1. Identify the chapter and questions from the conversation. If the chapter cannot be inferred, ask for it; if exact questions are missing but each answer is understandable, proceed and state the inferred question briefly.
2. Read the relevant portions of `/workspaces/ddia/content/en/chN.md`. Treat this text as authoritative.
3. Evaluate conceptual accuracy, reasoning, important omissions, and whether the answer respects the conditions and trade-offs in the chapter.
4. Preserve the learner's voice. Do not replace a basically sound answer merely to make it resemble the book.
5. Read answers from `/workspaces/ddia/notes/chN.md` unless the user supplies them elsewhere.
6. Review populated inline `#### My answer`, `#### My answer (revise)`, or
   `#### My answer (revised)` subsections under `## Focus questions` and the
   `## Closed-book recall` section. Support legacy notes that still store
   responses under `## My answers`.
7. Read `## Concepts` when populated and use the learner's terminology notes as supporting evidence, but leave concept explanations to the concept-explanation skill unless the user asks for them as part of the review.
8. Write or update a single `#### Review` subsection immediately after each
   reviewed answer, so feedback remains adjacent to its question and answer. Use
   `## Review` only for chapter-wide feedback, then add a tailored scenario under
   `## Application challenge` and a schedule under `## Spaced review`.
9. On the first pass, request revision only for a material misconception or
   omission. Rename `#### My answer` to `#### My answer (revise)` only when
   revision is needed. Keep the answer text in place and give one minimal revision
   cue without a model answer. Leave sound answers headed `#### My answer`. If a
   sound answer already has a generated `#### Review`, treat it as reviewed and
   stable unless the learner explicitly requests reassessment.
10. When reviewing an answer already headed `#### My answer (revise)`, treat the
    request as a reassessment. Check whether the answer now addresses the earlier
    cue. If it does, rename the heading to `#### My answer (revised)` and replace
    the generated review with a concise reassessment. If an important gap remains,
    explain it directly and provide a concise stronger answer; keep the heading
    as `#### My answer (revise)` so the learner can make another attempt.
11. Treat `#### My answer (revised)` as stable unless the learner explicitly asks
    for another reassessment or the answer still contains a material error. Do not
    repeatedly rewrite sound feedback.
12. Preserve the questions, answer content, recall, concepts, and all other
    user-written text exactly; the status-heading changes in steps 9–10 are the
    only automatic changes to learner-authored structure.

## Feedback Format

On the first pass, immediately below each answer give:

- **Assessment:** `Solid`, `Mostly right`, `Needs revision`, or `Not answered`.
- **What works:** identify the sound reasoning precisely.
- **Clarification:** for `Solid` or `Mostly right`, add only useful nuance that
  does not require revision.
- **Revision cue:** for `Needs revision`, identify the smallest important gap as
  a targeted question or hint. Do not supply the missing reasoning or a model
  answer on this pass.

On reassessment, replace the generated review rather than stacking another review
subsection. State whether the earlier gap is resolved and what improved. If it is
not resolved, give the direct clarification and a concise **Stronger answer**.
Do not provide a stronger answer before the learner has had a revision opportunity,
unless the learner explicitly asks for one.

Under `## Review`, provide:

- **Pattern across your answers:** 1–3 observations about the learner's mental model.
- **Review next:** up to three concepts or chapter sections, ranked by importance.
- **Follow-up:** 1–3 targeted questions that test the corrected understanding, without answers.

For a large answer set, keep each inline review compact and reserve detailed comments for misconceptions and high-value ideas. Do not put a separate assessment table at the end when inline reviews are present.

## Application Challenge

Create one realistic, compact system-design scenario targeting the chapter's most important idea or the learner's weakest understanding. Ask the learner to state:

- the decision they would make;
- which chapter concepts support it;
- their assumptions;
- how the choice could fail;
- what requirement change would make them choose an alternative.

Do not provide the solution. If an unanswered application challenge already exists, preserve it instead of replacing it.

## Spaced Review

Create three dated review checkpoints approximately 2 days, 1 week, and 1 month after the review date. Format each as:

```markdown
### YYYY-MM-DD

- [ ] Review complete

1. Retrieval question
2. Application or comparison question

#### My review answers


```

At each checkpoint:

- include a Markdown checkbox;
- ask only 2–4 retrieval questions;
- prioritize starred concepts, mistakes, uncertainty, and foundational ideas;
- vary the wording or scenario instead of repeating questions verbatim;
- omit answers.

If the user later asks for another review, preserve completed checkpoints and user
responses. Update only future, untouched checkpoints when useful. The
`review-ddia-due` skill conducts these checkpoints, evaluates their answers, and
adapts the next interval.

After writing, report the clickable note path. If no answers have been written, do not invent a review; tell the user the note is ready for their answers.

## Evaluation Principles

- Judge understanding, not wording or memorization.
- Separate factual error from missing nuance.
- Explain why a correction matters for designing or operating data systems.
- Give partial credit when reasoning is sound but incomplete.
- Be candid and encouraging; avoid generic praise.
- Cite the relevant local section by heading. Include a clickable local file link with a line number when it helps the learner return to the passage.
- If the chapter permits multiple defensible answers, name the assumptions under which each works.
- Never claim the chapter says something you have not verified in the file.
