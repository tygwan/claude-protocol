# Visual design review

Judge visual design as a communication system, not decoration.

## Hierarchy

Check whether visual emphasis matches task priority:

- primary action dominates secondary and tertiary actions,
- titles, explanations, state labels, and actions scan in intended order,
- destructive or unavailable actions do not attract primary-action weight,
- repeated emphasis does not flatten the hierarchy,
- the most visually prominent item is actually the most useful next step.

Test at normal view, blurred vision, and quick glance. If hierarchy only works after reading every label, it is weak.

## Grouping and spatial logic

Check:

- related content shares proximity and container treatment,
- unrelated states are not fused by a common surface,
- spacing rhythm indicates nesting and sequence,
- sheets, dialogs, cards, and pages use consistent boundaries,
- alignment does not imply a relationship that does not exist.

A visually consistent group should also be semantically consistent.

## State identity

Compare normal, hover, focus, pressed, selected, loading, empty, error, unavailable, restricted, locked, completed, and exhausted states where relevant.

Ask:

- Can two different meanings look the same?
- Can the same meaning look unrelated across routes?
- Does color carry meaning alone?
- Does a disabled appearance conceal an available recovery action?
- Does a neutral state accidentally read as error or punishment?

Use side-by-side state comparison when confusion cost is high.

## Typography and copy density

Check:

- role of each text style,
- line length and wrapping,
- emphasis within paragraphs,
- hierarchy after localization,
- font behavior under zoom,
- whether playful or branded type harms comprehension,
- whether small text carries essential decisions.

Do not invent a universal font size. Compare project tokens, audience needs, viewport, and runtime behavior.

## Color, tokens, and brand

- Treat token definitions as source evidence; do not copy sampled pixels when a token exists.
- Distinguish exact token match from visually similar color.
- Verify contrast separately from brand consistency.
- Use brand expression to support recognition and tone, not to obscure state.
- Record raw literals and one-off variants as drift unless intentionally approved.

## Consistency test

Classify each difference:

- semantic: meaning differs, so visual difference is justified,
- contextual: same component adapts to space or role,
- accidental: no user-facing reason,
- historical: inherited debt,
- unverified: intent unknown.

Consistency is not sameness. Preserve meaningful differences and remove accidental ones.

## Evidence limits

Static canvas evidence can verify hierarchy and state intent. It cannot prove focus order, reflow, input behavior, loading timing, or assistive-technology output. Route those claims to runtime verification.
