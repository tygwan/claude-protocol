# Response modes

Use the smallest response mode that supports the next decision. A fixed template is a tool for interoperability, not a ceiling on analysis.

## Checkpoint mode

Use during routine progression and human gates.

Required:

- current outcome layer,
- user-visible impact,
- decision or blocker,
- next safe action and actor.

Add at most the evidence needed for that decision. Reference the audit record instead of pasting it.

## Critique mode

Use for design, UI, UX, copy, accessibility, responsive, or fidelity feedback.

Start with the strongest decision-relevant insight in natural prose. Then include:

- evidence coverage and important gaps,
- findings ordered by user impact,
- meaningful alternatives and tradeoffs,
- unresolved decisions and owners,
- a clear recommendation and regret mode.

The exact headings, narrative order, and number of findings are adaptive. Use structured finding cards only where precision or handoff requires them. Preserve unexpected `design_opportunity`, `hypothesis`, and `adjacent_observation` insights without confusing them with accepted scope.

## Audit mode

Use on request or automatically when evidence, identity, lineage, schema, or authorization is ambiguous.

Include:

- authoritative IDs and versions,
- producer envelope and receiver receipt,
- immutable revisions,
- full approval-bound digests,
- manifest and path-set results,
- event and supersession lineage,
- failed checks and recovery history,
- exact gate evidence.

Audit mode is factual. Keep speculative design critique in the critique plane.

## Mixed decisions

When a review contains both approval and deferral:

1. state exactly what is approved,
2. state exactly what remains unverified,
3. name the gate that inherits each deferred check,
4. define the acceptance evidence,
5. prevent the approved subset from being read as whole-artifact approval.

## Copy-ready instructions

When another actor must act, provide one destination-labeled instruction that includes:

- authoritative request and artifact IDs,
- allowed action,
- prohibited actions,
- expected output,
- stop condition.

Do not force the user to relay routine commentary. The instruction should be necessary because the actor or tool boundary is real.
