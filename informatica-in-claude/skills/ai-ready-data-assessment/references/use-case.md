# AI use case picker (ask first)

Ask this **before** any CDGC query, topology pass, or `A` / `D` / `U` prompt. Do not start catalog work until the use case is accepted.

Present the question with a numbered choice (AskQuestion when available, otherwise this exact block). Mark **Support / Service Agent** as the default. If the user omits a choice, says default, or only names this skill, select **Support / Service Agent** and continue.

```text
Which AI use case should this assessment score against?

❯ 1. Support / Service Agent
     Answers customer questions, triages tickets, retrieves order + warranty + install-base data at conversation speed. Weights hardest on: Discoverability · Governed · Observability.
  2. RAG / Knowledge Assistant
     Answers questions from documents, wikis, tickets and other unstructured content. Weights hardest on: Grounded · Discoverability (unstructured).
  3. Analytics / BI Copilot
     Answers metric and reporting questions over your governed warehouse or lakehouse. Weights hardest on: Quality · Accessible.
  4. Autonomous / Action Agent
     Takes actions on your systems — updates records, triggers workflows, moves money. Weights hardest on: Governed · Provable/Trust · Observability.
  5. Type something.
```

AskQuestion labels (same five options, option 1 recommended):

1. Support / Service Agent — Answers customer questions, triages tickets, retrieves order + warranty + install-base data at conversation speed. Weights hardest on: Discoverability · Governed · Observability. (Recommended / default)
2. RAG / Knowledge Assistant — Answers questions from documents, wikis, tickets and other unstructured content. Weights hardest on: Grounded · Discoverability (unstructured).
3. Analytics / BI Copilot — Answers metric and reporting questions over your governed warehouse or lakehouse. Weights hardest on: Quality · Accessible.
4. Autonomous / Action Agent — Takes actions on your systems — updates records, triggers workflows, moves money. Weights hardest on: Governed · Provable/Trust · Observability.
5. Type something.

## What is implemented

**Only option 1 is supported.** Accepted answers: `1`, `Support / Service Agent`, `Support`, `Service Agent`, `default`, or equivalent.

## Other choices

If the user picks 2, 3, or 4, or types anything that is not option 1, **do not query CDGC**. Reply:

```text
That use case is not supported yet. Choose Support / Service Agent (option 1) to continue.
```

Then ask again for Support / Service Agent. Do not run the assessment against an unsupported use case, and do not invent scoring weights for options 2–5.

## After option 1 is accepted

- Set `meta.useCase` and `meta.heroTitle` to **Support / Service Agent**.
- Frame failure scenarios as a support / service agent (customer questions, ticket triage, order + warranty + install-base) using **this catalog’s** sources and counts — never Meridian / CX-4471 demo copy.
- In `method.scoring` (or the default scoring box), note that this run scores Support / Service Agent and emphasizes Discoverability, Governed, and Observability. Overall status is still the weakest measure; do not change formulas or invent a weighted headline.
- Record the choice in `inputs` (`AI use case` = Support / Service Agent, `user-supplied` or `default`).
