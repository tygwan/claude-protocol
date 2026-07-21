# Adaptive review kernel

The kernel protects analytical quality without turning review into a checklist recital. It specifies how to reason publicly from evidence, not what conclusion to reach.

## 1. Open read

Inspect the artifact before loading a fixed checklist. First describe:

- what the interface appears to help the user do,
- what draws attention first,
- what state or transition the user is likely to infer,
- what is surprising, ambiguous, or absent.

This pass is deliberately pre-taxonomy. It preserves observations that a preset category might suppress.

## 2. Establish truth and authority

Separate:

- user goal,
- product and domain truth,
- design intent,
- implementation reality,
- reviewer authority,
- unknowns requiring an owner.

Do not optimize presentation for a behavior or claim that is not yet authorized.

## 3. Select lenses dynamically

Choose only lenses that can change the decision. Typical lenses include task flow, state identity, hierarchy, interaction feedback, copy truth, accessibility, responsive composition, implementation fidelity, brand coherence, and domain boundaries.

Explain why a non-obvious lens is relevant. Do not run every module merely to appear thorough.

## 4. Challenge the leading interpretation

Before recommending, test at least one credible counterexample or regret mode:

- What could a hurried, young, stressed, low-vision, keyboard-only, or first-time user infer?
- Which adjacent state could be mistaken for this one?
- What happens when content, localization, viewport, latency, or permissions differ?
- What is the cheapest reversible alternative?
- What evidence would falsify the current recommendation?

The goal is not to invent issues. It is to expose brittle assumptions.

## 5. Synthesize

Lead with the decision-relevant conclusion. Tie it to observed evidence and user impact. Offer alternatives only when they create a real tradeoff. State confidence and missing evidence proportionally.

Use one of these insight types:

- `blocking_finding`: must be resolved before the named gate.
- `quality_improvement`: scoped improvement with clear user value.
- `design_opportunity`: promising direction, not required for acceptance.
- `hypothesis`: plausible but needs evidence.
- `adjacent_observation`: relevant discovery outside current scope.

An adjacent observation never expands scope or authorizes mutation. Route it to a backlog, follow-up request, or owner only when useful.

## Evidence discipline

For blockers, major findings, gate decisions, and disputed claims, provide a reproducible finding card:

- claim and insight type,
- direct evidence and evidence grade,
- user impact,
- recommendation or decision needed,
- confidence,
- owner,
- falsifier or missing evidence when relevant.

Moderate and minor observations may use concise prose if evidence and action remain clear. Do not force every sentence into the same record shape.

## Quality controls

- No finding quotas.
- No issue inflation to demonstrate effort.
- No universal best practice without project context.
- No inference presented as measurement.
- No generic preference overriding project canon.
- No hidden chain-of-thought; publish evidence, alternatives, tradeoffs, and concise rationale.
- No policy decision by an advisory reviewer.

When no actionable finding exists, state what was verified and what remains unverified.
