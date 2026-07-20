# Claude Design Sync Protocol

A Claude Code plugin for managing one continuous, human-gated request from design authoring through verified product delivery.

> Status: `v0.1.0-alpha.3`. This is a working pilot baseline, not a production-stable harness.

## What it provides

- Outcome-layer lifecycle states instead of one ambiguous "complete"
- Semantic approval gates that projects may map to local labels
- Continuity from a natural-language design request to implementation, delivery, and closure
- Actor and ownership boundaries for the human, repository orchestrator, design author, delivery adapter, and domain decider
- Append-only DREQ, DRES, and DVER artifacts
- Verified implementation-reference promotion with path, byte, and SHA-256 evidence
- Capability-based transport, including manifest-bearing zip relay
- Evidence-backed visual, UI, UX, accessibility, and responsive review
- Compact normal checkpoints with audit detail on failure or request
- Project-specific adapters without hard-coding one repository, framework, design tool, branch model, or deployment provider

It does not provide a Claude Design transport or project API. It coordinates available adapters and stops when their capabilities cannot prove a safe handoff.

## Install

Add this repository as a Claude Code marketplace:

```text
/plugin marketplace add tygwan/claude-protocol
/plugin install claude-design-sync@tygwan-protocols
/reload-plugins
```

Current alpha entry points:

```text
/claude-design-sync:design-sync bootstrap
/claude-design-sync:design-sync request
/claude-design-sync:design-sync status
/claude-design-sync:design-sync continue
/claude-design-sync:design-sync audit
/claude-design-sync:design-review all
```

The planned user-facing `/design` front door will wrap these internal skills after installation and resume testing is complete.

For local development:

```text
claude --plugin-dir ./plugins/claude-design-sync
claude plugin validate .
```

## Outcome lifecycle

```text
design_authored
  -> candidate_verified
  -> implementation_reference_accepted
  -> implementation_in_progress
  -> implementation_verified
  -> delivered
  -> closed
```

Discovery, request drafting, review, and approval are activities around these result states. A product-change request stays active until the actual product is implemented, verified, delivered, and recorded. Design acceptance alone never implies product delivery.

Design-only requests may close after an accepted implementation reference, but must explicitly state that implementation and delivery were not requested.

## Semantic gates

- `request_scope_approval`
- `design_acceptance`
- `implementation_acceptance`
- `delivery_approval`
- `regulated_decision`

Projects may map these to G1, G2, G3, or other local names in their profile. Durable state retains the semantic name.

## Checkpoint experience

Normal output is concise:

```text
State: candidate_verified
Design changed: yes
Product changed: no
Visible in target environment: no
Blocked by: design_acceptance
Next: product owner accepts or holds the verified reference
```

Hashes, manifests, lineage, and recovery details stay in audit state and expand automatically only on verification failure, identifier mismatch, approval-target change, recovery, or explicit `audit`.

At every pause the orchestrator names the next safe action and actor. The user does not have to remember and separately restart the next lifecycle stage.

## Adapter boundary

A project profile supplies:

- design authoring source,
- implementation-reference location and atomicity,
- actual implementation targets,
- transfer and delivery mechanisms,
- verification environments,
- approval roles and local gate aliases,
- final verification format.

The reusable core contains no required filesystem path, framework, version-control host, pull-request model, design tool, or deployment provider.

## Plugin layout

- `.claude-plugin/marketplace.json`: marketplace catalog
- `plugins/claude-design-sync/skills/design-sync/`: lifecycle and gate orchestration
- `plugins/claude-design-sync/skills/design-review/`: modular review framework
- `examples/project-profile.example.yaml`: adapter-oriented project profile
- `docs/PILOT-LOG.md`: observed failures and protocol changes
- `docs/ROADMAP.md`: path from alpha to v1

## Safety model

- User relay is transport, never approval.
- A model recommendation, CI success, or an earlier gate does not satisfy a later semantic gate.
- Wrong identity, stale lineage, incomplete candidates, hash mismatch, ambiguous ownership, or stale prescriptive companions are hard stops.
- Known reference contradictions cannot be deferred to runtime validation.
- Existing project instructions remain authoritative over this plugin.

## Current limitations

- The user-facing `/design` front door and guided first-run setup are not implemented yet.
- Claude Design project identity may require name plus file fingerprint when no project-ID API exists.
- Exact raw download-to-disk is adapter-dependent.
- Deterministic verifier and approval-token generation are not implemented.
- Local plugin validation, installation, restart, and resume smoke tests remain pending.
- The first real project pilot is not yet closed through implementation, delivery, and DVER.

## Contributing

Record each protocol change with:

1. observed failure or friction,
2. affected invariant,
3. proposed change,
4. compatibility impact,
5. pilot evidence that confirms or refutes it.
