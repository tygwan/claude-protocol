---
name: design-review
description: Produce evidence-backed design, UI, UX, copy, accessibility, and responsive feedback without inventing product policy. Use when reviewing mockups, design canon, screenshots, component states, interaction flows, viewport behavior, or design-to-code fidelity.
argument-hint: "[all|visual|ui|ux|copy|accessibility|responsive|decision]"
---

# Design Review

This skill turns review judgment into an auditable framework. Do not reveal private hidden reasoning or a token-by-token chain of thought. Provide the evidence, criteria, alternatives, tradeoffs, and concise rationale needed for another reviewer to reproduce the decision.

Project instructions, product policy, and named decision owners remain authoritative.

## Load only what is needed

Always read `references/evidence-and-decisions.md`.

Then route by scope:

- visual hierarchy, brand, tokens, consistency: `references/visual-design.md`
- controls, states, feedback, navigation: `references/ui-interaction.md`
- task flow, audience, trust, copy: `references/ux-flow-and-copy.md`
- keyboard, semantics, contrast, touch, assistive technology: `references/accessibility.md`
- viewport, reflow, component-to-page composition: `references/responsive.md`
- clinical, legal, safety, policy, or backend ownership: `references/domain-boundaries.md`

For `all`, review in this order:

1. authority and domain boundaries,
2. user goal and flow,
3. UI states and interactions,
4. responsive composition,
5. accessibility,
6. visual hierarchy and polish.

This order prevents polishing a state that is conceptually wrong.

## Review contract

Before judging, state:

- target artifact and immutable version when available,
- user and role,
- task and success condition,
- states and routes in scope,
- viewports in scope,
- baseline or canon,
- evidence available and missing,
- decisions explicitly outside reviewer authority.

If the contract is missing, inspect safely or ask for the smallest missing fact. Do not fill policy gaps with design intuition.

## Finding discipline

Every finding must include:

- area,
- severity,
- claim type,
- direct evidence,
- user impact,
- recommendation,
- confidence,
- owner.

Use these severities:

- `blocker`: unsafe, misleading, inaccessible, corrupt, or prevents the task.
- `major`: likely abandonment, dead end, state confusion, or serious cross-viewport break.
- `moderate`: avoidable cognitive load, weak hierarchy, inconsistent interaction, or localized accessibility risk.
- `minor`: polish with low task impact.

Use these claim types:

- `fact`
- `inference`
- `recommendation`
- `decision_required`
- `unverified`

Never present an inference as a measured fact.

## Decision behavior

Prefer the lowest-regret reversible recommendation that satisfies the user task, product truth, accessibility baseline, and ownership boundaries.

A recommendation is not a human gate approval. When a decision changes policy, meaning, ownership, regulated content, or an irreversible workflow, identify the decision owner and stop.

## Output

Return:

1. outcome and current confidence,
2. evidence coverage and gaps,
3. findings ordered by severity,
4. approved aspects,
5. unresolved decisions and owners,
6. viewport/state coverage,
7. recommendation with regret mode,
8. one copy-ready next instruction labeled with its destination actor.

When there are no actionable findings, say so and name what was actually verified. Do not invent issues to make the review look thorough.
