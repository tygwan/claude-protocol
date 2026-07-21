---
name: design-review
description: Produce evidence-backed design, UI, UX, copy, accessibility, and responsive feedback without inventing product policy. Use when reviewing mockups, design canon, screenshots, component states, interaction flows, viewport behavior, or design-to-code fidelity.
argument-hint: "[all|visual|ui|ux|copy|accessibility|responsive|decision]"
---

# Design Review

This skill turns review judgment into an auditable framework. Do not reveal private hidden reasoning or a token-by-token chain of thought. Provide the evidence, criteria, alternatives, tradeoffs, and concise rationale needed for another reviewer to reproduce the decision.

Project instructions, product policy, and named decision owners remain authoritative.

## Load only what is needed

Always read `references/evidence-and-decisions.md`, `references/review-kernel.md`, and `references/response-modes.md`.

Read `references/design-intelligence-providers.md` when a project configures an advisory provider or an activation trigger is present.

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

## Adaptive finding discipline

Begin with an open read before applying categories. Select only the review lenses that can change the decision, challenge the leading interpretation, and then synthesize. Follow `references/review-kernel.md`.

Reserve a full structured finding card for blockers, major findings, gate decisions, and disputed claims. Moderate and minor observations may use concise prose when evidence, impact, and action remain clear. There is no finding quota.

Use these severities:

- `blocker`: unsafe, misleading, inaccessible, corrupt, or prevents the task.
- `major`: likely abandonment, dead end, state confusion, or serious cross-viewport break.
- `moderate`: avoidable cognitive load, weak hierarchy, inconsistent interaction, or localized accessibility risk.
- `minor`: polish with low task impact.

Classify insights as `blocking_finding`, `quality_improvement`, `design_opportunity`, `hypothesis`, or `adjacent_observation`. Keep facts, inferences, recommendations, decisions, and unverified claims visibly distinct. Never present an inference as a measured fact.

Unexpected insights are welcome, but they do not expand scope. Route useful adjacent observations to a follow-up owner without mutating the current artifact.

## Decision behavior

Prefer the lowest-regret reversible recommendation that satisfies the user task, product truth, accessibility baseline, and ownership boundaries.

A recommendation is not a human gate approval. When a decision changes policy, meaning, ownership, regulated content, or an irreversible workflow, identify the decision owner and stop.

## Output

Choose checkpoint, critique, audit, or a deliberate combination from `references/response-modes.md`.

In critique mode, lead with the strongest decision-relevant insight, then provide enough evidence, alternatives, tradeoffs, unresolved ownership, and regret analysis to support action. Headings and narrative shape are adaptive; interoperability fields remain structured only where a gate or handoff consumes them.

End with one copy-ready next instruction labeled with its destination actor when another actor must act.

When there are no actionable findings, say so and name what was actually verified. Do not invent issues to make the review look thorough.
