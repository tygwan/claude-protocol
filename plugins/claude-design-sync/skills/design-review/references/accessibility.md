# Accessibility review

Use the project's declared conformance target. When none exists, report against WCAG 2.2 without claiming certification.

Authoritative references:

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- Target Size Minimum: https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum
- Focus Not Obscured: https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum
- Error Identification: https://www.w3.org/WAI/WCAG22/Understanding/error-identification

## Visual access

Check:

- normal text contrast,
- large text contrast,
- non-text contrast for controls and state indicators,
- information not carried by color alone,
- zoom and text resize without loss,
- readable line length and spacing,
- motion and flashing risk,
- focus visibility against every background state.

WCAG 2.2 AA uses 4.5:1 for normal text and 3:1 for large text. Component boundaries and meaningful graphical objects generally require 3:1 under non-text contrast. Measure rendered colors; do not infer from token names.

## Target size

WCAG 2.2 AA Target Size Minimum is 24 by 24 CSS pixels with defined exceptions. WCAG Enhanced uses 44 by 44 CSS pixels and is a stricter level. Project or audience rules may require larger targets.

Report separately:

- standards minimum,
- project requirement,
- audience recommendation,
- measured runtime value.

Do not describe a 44, 56, or 64 pixel project recommendation as the universal WCAG AA minimum.

## Semantics

Verify at runtime:

- accessible name,
- role,
- value or state,
- heading order,
- landmark structure,
- labels and instructions,
- disabled versus aria-disabled behavior,
- live status announcement,
- error association,
- image alternatives,
- language metadata.

Visual resemblance is not semantic equivalence.

## Keyboard

Check tab order, activation, focus return, escape behavior, traps, and obscured focus. A sheet or dialog should have a defined focus lifecycle.

## Dynamic content

Loading, validation, success, restriction, and error updates should be perceivable without relying on sight or timing. Avoid unnecessary announcement of internal codes.

## Evidence

Separate canvas intent, source semantics, automated checks, and manual assistive-technology testing. Passing one does not imply the others.
