# Responsive and viewport review

## Declare coverage before design

For every route, classify each relevant viewport:

- `required`
- `optional`
- `not_applicable`

Record device assumptions, orientation, input mode, browser chrome, safe areas, and text scaling where material.

Do not discover required coverage only at final approval.

## Evidence types

Keep these distinct:

- full-page canonical artboard,
- component-state strip,
- responsive rule note,
- source layout rule,
- DOM measurement,
- live browser render,
- screenshot.

A component at approximately the same width is useful component evidence but not proof of the full composition.

## Review matrix

For each required viewport, inspect:

- container and column changes,
- minimum and maximum widths,
- wrap order,
- overflow and clipping,
- fixed or sticky elements,
- sheet and dialog height,
- safe-area inset,
- virtual keyboard interaction,
- text zoom and localization expansion,
- touch target and spacing,
- focus visibility,
- scroll ownership,
- orientation change,
- image crop and asset density.

## Breakpoints

Treat a breakpoint as a behavior change, not a device name. Verify immediately below, at, and above consequential breakpoints when runtime exists.

## Missing evidence

Use `unverified`; never infer "no impact."

A required viewport blocks approval unless:

1. the user explicitly accepts deferral,
2. the target gate is named,
3. concrete acceptance checks are recorded,
4. failure blocks commit or promotion,
5. implementation cannot silently alter canon to pass.

## Design-to-runtime boundary

Canvas review establishes intent. G3 live-browser review establishes reflow, focus, scroll, timing, and interaction behavior.

If runtime verification requires a visual change, return to the project's design-change path rather than silently patching implementation.
