# Evidence and decision framework

## Separate five things

Label important claims:

- `FACT`: directly observed in an authoritative artifact or runtime.
- `INFERENCE`: a conclusion drawn from facts; state the bridge.
- `RECOMMENDATION`: a proposed action with tradeoffs.
- `DECISION REQUIRED`: a choice owned by a named human or domain role.
- `UNVERIFIED`: evidence is absent, indirect, stale, or cannot be reproduced.

A confident tone does not increase evidence quality.

## Evidence ladder

Use the strongest available evidence and name its limits.

| Grade | Evidence | Appropriate use |
|---|---|---|
| E4 | live runtime, raw artifact, DOM measurement, reproducible test | behavior and exact dimensions |
| E3 | authoritative canon, source code, signed manifest, current design file | intended structure and state |
| E2 | component-state strip, token definition, interaction specification | component judgment, not full-page composition |
| E1 | screenshot, transcript, verbal report | orientation and hypothesis |
| E0 | absent or contradictory evidence | no approval; mark unverified |

A component strip is not evidence for full-page reflow. A screenshot is not evidence for keyboard order. Code is not proof of rendered appearance. A tool success message is not proof of persisted bytes.

## Reproducible review loop

1. Define user, task, state, route, viewport, and authority.
2. Inventory evidence by grade.
3. Compare baseline, candidate, and expected change.
4. Identify what changed, what did not, and what cannot be observed.
5. Test the most harmful plausible misunderstanding.
6. Consider the counterfactual: what would the user believe and do if the recommendation were wrong?
7. Compare reversibility and regret.
8. Name the decision owner.
9. Produce a concise rationale and verification condition.

## Reasoning lenses

Apply the relevant lenses rather than a universal checklist.

- Goal: can the user complete the intended task?
- Truth: does the interface accurately describe system state?
- Distinction: can adjacent states be confused?
- Sequence: is the next action visible at the right time?
- Recovery: is there a safe way out of failure or restriction?
- Consistency: does the pattern match existing canon where meaning matches?
- Difference: is a visual difference justified by a semantic difference?
- Audience: is information shown to the correct role?
- Reversibility: can a wrong decision be cheaply undone?
- Downstream impact: which routes, components, code, tests, or users inherit it?
- Negative space: what action, explanation, or state is missing?
- Counterexample: under what realistic condition does the proposed rule fail?

## Regret modes

Name at least one plausible regret for consequential recommendations:

- false approval,
- false rejection,
- stale canon promotion,
- hidden dead end,
- misleading success,
- policy invented by presentation,
- accessibility passing in a static artifact but failing at runtime,
- one viewport fixed while another regresses,
- local consistency that creates global inconsistency.

Prefer a recommendation that makes the dangerous regret observable before irreversible action.

## Confidence

Use `high`, `medium`, or `low`.

- High: direct, current, reproducible evidence covers the claim.
- Medium: authoritative intent exists but runtime or composition is missing.
- Low: evidence is indirect, stale, or disputed.

Confidence never replaces a required gate.
