# Changelog

## 0.1.0-alpha.3 — 2026-07-21

### Added

- Outcome-layer lifecycle states from `design_authored` through `closed`
- Semantic gates with optional project-local aliases
- Continuous request ownership through implementation, delivery, and final verification
- User-impact checkpoint contract for design, product, environment visibility, blockers, and next actor
- Compact normal output and expanded audit output
- Adapter-oriented project profile for authoring, implementation reference, implementation, delivery, and verification
- Workflow-checkpoint and delivery-aware DVER templates

### Changed

- Design acceptance is explicitly an intermediate state for product-change requests
- Runtime deferral cannot hide a known implementation-reference contradiction
- Core rules no longer require numbered gates, a repository path, framework, PR model, design tool, or deployment provider

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
