# Changelog

## 0.1.0-alpha.2 — 2026-07-20

### Added

- Separate `design-review` skill beside the workflow-oriented `design-sync` skill
- Evidence grades and fact/inference/recommendation labels
- Visual design, UI interaction, UX/copy, accessibility, responsive, and domain-boundary modules
- Regret-mode and counterexample checks
- WCAG 2.2 evidence guidance
- Viewport evidence and explicit G3 deferral rules

### Fixed

- Replaced a duplicated remote `design-sync/SKILL.md` with one canonical copy
- Require complete remote re-read after API writes
- Require approval-bound SHA-256 syntax and source validation


## 0.1.0-alpha.1 — 2026-07-20

Initial pilot baseline.

### Added

- Claude Code marketplace and plugin manifests
- Design-sync orchestration skill
- Role, ownership, state-machine, and gate invariants
- Append-only DREQ, DRES, visual-review, and DVER templates
- Capability-based direct-storage, raw-file, and zip transport rules
- Project profile example
- Pilot evidence log and path to v1

### Known limitations

- First end-to-end pilot has not yet completed G2 promotion, implementation, G3, and DVER.
- No deterministic verifier CLI is included.
- Claude Design transport remains tool-dependent.
- Human approval is structured but not yet cryptographically or platform signed.
