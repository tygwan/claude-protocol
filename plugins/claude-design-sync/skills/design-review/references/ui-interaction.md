# UI and interaction review

## Start from the state machine

List states and transitions before reviewing individual controls.

For every state, record:

- entry condition,
- information shown,
- actions available,
- default focus,
- success transition,
- failure transition,
- escape or back behavior,
- persistence effect,
- owner of the underlying rule.

Missing transitions are often more harmful than imperfect visuals.

## Affordance and action truth

A control should look actionable only when it is actionable.

Distinguish:

- unavailable now,
- disabled because input is incomplete,
- locked by entitlement or relationship,
- exhausted quota,
- loading,
- failed request,
- completed,
- read-only information.

Do not use one generic disabled style for semantically different conditions when the user needs different next actions.

## Priority and alternatives

Check:

- one clear primary action per decision point,
- secondary actions do not compete with the primary,
- unavailable primary actions have an honest explanation,
- alternative actions lead somewhere valid,
- no dead CTA or placeholder destination,
- closing, cancelling, and going back have predictable effects.

A replacement action must solve the user's next problem, not merely remove an error.

## Feedback and timing

For each interaction:

- immediate acknowledgment,
- progress when delay is material,
- success only when the promised effect occurred,
- error text when an input error exists,
- status updates available beyond color or animation,
- duplicate submission prevention,
- recovery without destructive repetition.

Do not show false success to soften a failure. Separate compassionate tone from factual state.

## Keyboard and focus

Runtime verification should cover:

- logical focus order,
- visible focus,
- focus entering and returning from dialogs or sheets,
- Escape and close behavior,
- focused controls not obscured by sticky or modal content,
- no keyboard trap,
- activation by appropriate keyboard controls.

A canvas note is intent, not proof.

## Touch and pointer

Check target dimensions and spacing against project policy and accessibility requirements. Larger targets may be appropriate for children, mobility needs, motion, or one-handed use. Do not promote a project recommendation to a universal standard.

## Interaction consistency

Reuse behavior when meaning matches. If an established component is reused for a new semantic state, verify that its old affordances do not communicate the wrong rule.
