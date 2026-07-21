# Protocol invariants

## Roles

The protocol uses roles rather than hard-coding one orchestration product.

- Human owner: approves scope and promotion.
- Workflow orchestrator: writes request and verification artifacts, validates candidates, publishes implementation references, and coordinates implementation.
- Design author: owns the visual authoring source and writes DRES.
- Independent auditor: optional; reviews protocol or disputed evidence but is not a mandatory relay.
- Delivery adapter: applies the accepted implementation through the project's configured merge, changeset, publish, or deployment mechanism.
- Domain decider: resolves clinical, legal, safety, or other regulated flags.

Claude Code is the first workflow-orchestrator adapter. Claude Design is the first design-author adapter. Future adapters may fulfill either role without changing the core lifecycle.

## Authority boundaries

Projects must declare:

- design authoring source,
- accepted implementation reference,
- adapter-managed integrity metadata,
- implementation targets,
- handoff storage,
- human approval roles,
- regulated decision roles,
- delivery mechanism and target environments,
- final verification record.

A copy is not authoritative merely because it is newer. Authority follows the declared ownership map and gate history.

## Artifact lineage

DREQ, DRES, and DVER artifacts are append-only.

- IDs are stable and sortable.
- A correction creates a new sequence.
- The new artifact names the exact artifact it supersedes.
- Superseded files remain available for audit.
- Consumers select the newest valid, non-superseded artifact only after verifying lineage.

Suggested IDs:

```text
DREQ-{work_id}-{request_seq}
DRES-{work_id}-{request_seq}-{response_seq}
DVER-{work_id}-{request_seq}-{verification_seq}
```

## Typed handoff and receipts

Every cross-actor transfer uses two records:

1. an immutable producer envelope describing what was sent and what the receiver must check,
2. an immutable receiver receipt describing what was actually read, measured, and accepted or rejected.

The producer envelope is a claim. Only the receiver receipt can establish reception. Neither record grants a human gate.

When a producer cannot measure raw bytes, it records `declared_digest: null` and `digest_status: unavailable_at_source`. It never uses a placeholder digest as an approval identity.

Persist a minimal append-only event log for state transitions and decisions. Conversation transcripts and model reasoning are not workflow state. Resume derives from validated artifacts, receipts, events, and the current checkpoint. See `handoff-and-events.md` and the JSON Schemas in the plugin's `schemas/` directory.

The core defines required durable information; a project profile defines storage, retention, privacy, and compaction.

## Outcome lifecycle

The core tracks distinct result layers:

1. `design_authored`: the design authoring source was created or changed.
2. `candidate_verified`: the transferable candidate's integrity, scope, lineage, and coherence were verified.
3. `implementation_reference_accepted`: the verified artifact was accepted as the implementation reference.
4. `implementation_in_progress`: the real product implementation is being changed.
5. `implementation_verified`: the implementation was verified in its configured execution environment.
6. `delivered`: the accepted result reached the configured target environment.
7. `closed`: final verification and unresolved items were recorded.

These states describe different results and must not be collapsed into "complete." A checkpoint records only states supported by evidence. A design-only request may stop after implementation_reference_accepted, but it must explicitly record that product implementation and delivery were not requested.

## Semantic gates

Core gate names describe meaning rather than sequence numbers:

- `request_scope_approval`: approves the requested outcome, scope, exclusions, and decision owners before design mutation.
- `design_acceptance`: accepts an exact verified design candidate as an implementation reference.
- `implementation_acceptance`: accepts runtime evidence that the product matches the reference and required behavior.
- `delivery_approval`: authorizes the configured delivery operation when it is not already implied by project policy.
- `regulated_decision`: resolves clinical, legal, safety, financial, or other regulated policy.

A project may map these to local labels such as G1, G2, and G3. Durable artifacts keep the semantic name and may add the local alias. Transport, model recommendations, CI success, and prior approvals do not satisfy a gate.

## Continuity

Every request declares a requested terminal outcome. When that outcome includes a product change, accepting design is an intermediate state, not closure. The orchestrator keeps the request active through implementation-reference acceptance, product mutation, runtime verification, delivery, and final verification.

At every checkpoint, propose the next safe action and its actor. Do not make the user remember or separately re-request the next lifecycle stage. A pause at a human gate preserves state and resumes from that gate in a later session.

## User-impact checkpoint

Every checkpoint states:

- current lifecycle state,
- design artifacts changed: yes or no,
- actual product changed: yes or no,
- visible in target environment: yes, no, or unverified,
- blockers,
- next action,
- next actor.

Normal mode shows this summary, the decision required, and the next action. Audit mode adds hashes, manifests, lineage, receipts, verifier output, and recovery history. Automatically expand audit details whenever evidence or authorization is ambiguous, including verification failure, identity mismatch, approval-target change, conflicting receipts, schema incompatibility, stale lineage, or recovery.

## Adapters

The core does not require a repository path, framework, branch model, pull request system, design tool, or deployment provider. Project profiles and adapters define:

- design authoring source,
- accepted implementation-reference storage,
- actual implementation targets,
- change-delivery mechanism,
- verification environments,
- approval roles,
- final verification format.

Adapters translate core operations into local tools without changing lifecycle state or gate meaning.

## Advisory intelligence providers

Optional design-intelligence providers may enrich discovery and critique. They are read-only advisory sources by default, load only when a configured activation trigger is present, and rank below project truth and accepted canon.

Providers cannot approve gates, decide policy, expand scope, edit artifacts, persist design systems, or implement code unless the project profile explicitly grants that capability. Pin provider versions per active request and record accepted or rejected advice. Provider guidance is evidence, not authority.

Do not make an external marketplace plugin a hard dependency of the core when the workflow can operate without it.


## Snapshot promotion

Default to whole-snapshot promotion when Design publishes a package.

Verification must compare:

1. declared manifest path set,
2. extracted candidate path set,
3. current implementation-reference path set,
4. byte count for every candidate file,
5. SHA-256 for every candidate file,
6. expected and unexpected diff,
7. adapter-managed metadata exclusions.

No partial overlay is allowed unless the project profile explicitly defines a different atomic unit and its consistency rules.

## Canon coherence

A promoted snapshot is not coherent when its visual source changed but its implementation brief, route mapping, generated reference, or other prescriptive companion still describes the previous state.

Before merge, compare every changed authoring file with all companion files that instruct implementation. Stale prescriptive companions block merge even when hashes, CI, and the visual files themselves pass. Runtime deferral to implementation verification is allowed only for evidence that requires a live implementation; it must not be used to defer known canon contradictions.

Repair a contradiction at the declared source owner. If the companion belongs to the Design-authored candidate, request an append-only superseding response and re-run verification and design_acceptance. If it is adapter-managed metadata, update it through the configured same-change path and regenerate integrity metadata. Never resolve the contradiction by editing a read-only canon copy or by merging first and promising later cleanup.

## Transport decision

Transport selection is capability-based:

| Condition | Route |
|---|---|
| Shared persistent storage and exact re-read proven | direct shared storage |
| Exact raw bytes can be downloaded and hashed | direct raw retrieval |
| Hard size limit exceeded | manifest zip |
| Response is truncated | manifest zip |
| API returns only reconstituted text or cannot persist raw bytes | manifest zip |
| Identity or storage binding uncertain | stop and run a harmless canary |

A soft authoring guideline is not a transport limit. Record both independently.

## Visual decisions

Visual approval belongs to the human owner. Design may guide navigation and summarize the result, but must not self-approve.

The DREQ classifies every relevant viewport as `required`, `optional`, or `not_applicable`. The review decision for each viewport is `approve`, `hold`, `unverified`, or `not_applicable`.

The cumulative approval record must include:

- artifact ID and candidate digest,
- screen/state identifier,
- user decision,
- reason when held,
- viewport classification and evidence,
- explicit implementation-verification deferral and acceptance checks, if any,
- intentional drift decision,
- overall approve or hold.

Missing evidence is `unverified`, never "no impact." A required viewport blocks design_acceptance unless the user explicitly defers that exact coverage to implementation_acceptance. Such a deferral must identify the live-render checks that will block implementation commit if they fail.

Conversation memory alone is not a durable approval ledger. Until a signed artifact format is implemented, echo the full decision set after each step and require the final summary to match.

## Project profile

Keep product-specific details outside the core skill:

- repository, version-control, or content source when applicable,
- design project name, ID, and file fingerprint,
- retired project IDs,
- implementation-reference location and atomicity,
- design source selectors,
- package and handoff locations,
- implementation targets,
- delivery adapter and target environments,
- transport limits,
- atomic promotion unit,
- route-to-design-file mapping,
- required checks and runtime evidence,
- semantic-gate roles and optional local aliases,
- requested-outcome defaults,
- final verification artifact,
- domain flags and decision owners.

Never copy one project's clinical or product policy into the reusable core.
