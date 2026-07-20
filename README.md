# Claude Design Sync Protocol

A Claude Code plugin for governing design handoffs between a repository, a Claude Design project, and a human approver.

> Status: `v0.1.0-alpha.2`. This is a working pilot baseline, not a production-stable protocol.

## What it provides

- An explicit state machine from discovery to verified implementation
- Actor boundaries for Claude Code, Claude Design, and the user
- Append-only DREQ, DRES, and DVER artifacts
- Human approval gates that automation cannot infer
- Whole-snapshot promotion with path, byte, and SHA-256 verification
- Capability-based transport selection, including verified zip relay
- A cumulative visual-review decision format
- A modular, evidence-backed design/UI/UX review framework
- Project-specific configuration without hard-coding one product

It does not provide a Claude Design transport or project API. It orchestrates whichever design tools are available and stops when their capabilities cannot prove a safe handoff.

## Install

Add this repository as a Claude Code marketplace:

```text
/plugin marketplace add tygwan/claude-protocol
/plugin install claude-design-sync@tygwan-protocols
/reload-plugins
```

Then start with:

```text
/claude-design-sync:design-sync status
/claude-design-sync:design-review all
```

The two skills are intentionally separate:

- `design-sync` controls artifacts, gates, transport, and promotion.
- `design-review` supplies review criteria for visual design, UI interaction, UX/copy, accessibility, responsive behavior, and domain boundaries.

For local development:

```text
claude --plugin-dir ./plugins/claude-design-sync
```

Validate the marketplace and plugin before release:

```text
claude plugin validate .
```

## Workflow

```text
discover
  -> draft DREQ
  -> G1 human scope approval
  -> Claude Design recut
  -> append-only DRES
  -> independent artifact verification
  -> human visual review
  -> G2 human promotion approval
  -> whole-snapshot canon promotion
  -> implementation
  -> G3 live-browser approval
  -> DVER
```

Clinical or other regulated decisions remain outside the design gate and require a separately named decision owner.

## Repository layout

- `.claude-plugin/marketplace.json`: installable marketplace catalog
- `plugins/claude-design-sync/skills/design-sync/`: workflow and gate orchestration
- `plugins/claude-design-sync/skills/design-review/`: modular review framework
- `examples/project-profile.example.yaml`: portable project configuration
- `docs/PILOT-LOG.md`: observed failures and protocol changes
- `docs/ROADMAP.md`: path from alpha to v1

## Safety model

- User relay is transport, never approval.
- A model may recommend a gate outcome but cannot impersonate the human approver.
- Wrong-project identity, stale lineage, incomplete snapshots, hash mismatch, or ambiguous ownership are hard stops.
- Promotion is never partial and never automatic.
- Existing project instructions remain authoritative over this plugin.

## Current limitations

- Claude Design project identity may require name plus file fingerprint when no project-ID API exists.
- Exact raw download-to-disk is tool-dependent. A file being smaller than a transport limit does not prove lossless transfer.
- Visual review is still manual.
- The alpha has completed one real project pilot through DRES and technical G2 preflight; the full implementation and DVER loop is still in progress.

## Contributing

Please record every protocol change with:

1. the observed failure or friction,
2. the invariant affected,
3. the proposed change,
4. compatibility impact,
5. the pilot evidence that confirms or refutes it.
