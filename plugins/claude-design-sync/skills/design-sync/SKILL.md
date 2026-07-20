---
name: design-sync
description: Orchestrate a continuous human-gated lifecycle from a natural-language design request through authored design, verified implementation, delivery, and closure. Use when a task needs project setup, design handoff, DREQ/DRES/DVER artifacts, design review, implementation-reference acceptance, product implementation, delivery, or resumable status.
argument-hint: "[status|bootstrap|request|continue|verify|promote|implement|deliver|close|audit]"
---

# Claude Design Sync

Treat the user's project instructions as authoritative. This skill adds a workflow; it never weakens repository rules.

Before acting, read:

1. the nearest project instruction file,
2. the project design-sync profile if one exists,
3. `references/protocol.md`,
4. `references/templates.md`.

Use `$ARGUMENTS` to select the requested action. If it is missing, report current state and the next safe action.

For design, UI, UX, copy, accessibility, or responsive judgment, invoke the sibling `design-review` skill. This skill owns orchestration and gates; `design-review` owns review criteria. Neither replaces the human approver.

## Actor labels

Prefix every handoff block with exactly one destination:

- `[USER]`
- `[CLAUDE CODE]`
- `[CLAUDE DESIGN]`

Never give an instruction without its destination actor.

## Lifecycle state

Track results by outcome layer, not by a single word such as "complete":

`design_authored -> candidate_verified -> implementation_reference_accepted -> implementation_in_progress -> implementation_verified -> delivered -> closed`

Discovery, request drafting, review, and approval are activities or gates around these result states. A failed prerequisite keeps the current state. It never implies approval of the next state.

At request creation, record the requested terminal outcome. If the user asked for a real product change, remain active through implementation, runtime verification, delivery, and final verification. Do not close at design acceptance or ask the user to remember and restart the next phase.

Use semantic gates in the core:

- `request_scope_approval`
- `design_acceptance`
- `implementation_acceptance`
- `delivery_approval`
- `regulated_decision`

A project profile may map these to local labels such as G1, G2, or G3. Local labels never replace the semantic name in durable state.

## Hard stops

Stop and report when any of these is true:

- design project identity is ambiguous or retired,
- repository base cannot be tied to a clean immutable commit,
- ownership of authoring source or repository canon is ambiguous,
- request or response lineage is stale, missing, or overwritten,
- the candidate is partial when the project requires whole-snapshot promotion,
- path set, bytes, or SHA-256 differs from the authoritative manifest,
- transport truncated content or cannot independently reproduce raw bytes,
- a user gate has not been explicitly approved,
- a clinical, legal, safety, or product-policy flag lacks its named decision owner.

Do not infer approval from silence, file transfer, a model recommendation, or prior approval of a different sequence.

## Checkpoint contract

Every checkpoint must state:

- lifecycle state,
- whether only design artifacts changed,
- whether the actual product changed,
- whether the result is visible in the target environment,
- current blockers,
- next safe action and its actor.

Normal mode shows only that impact summary, the decision needed, and the next action. Keep hashes, manifests, lineage, and detailed verifier evidence internal unless verification fails, an identifier changes, recovery is required, or the user requests `audit`.

## Action behavior

### status

Read only. Reconstruct the active request and report the checkpoint contract. Name the exact outcome layer; never say only "complete." In audit mode also report authoritative artifact IDs, integrity evidence, unresolved flags, and the last explicit semantic gate.

### bootstrap

Discover existing project conventions and adapter capabilities first. Propose, but do not silently impose, a project profile covering authoring source, implementation reference, implementation targets, change-delivery mechanism, verification environments, approval roles, and closure record. If canonical project instructions must change, stop for the project's required structural approval.

### request

Investigate what can be learned safely, ask only for decisions that cannot be discovered, and draft one append-only DREQ. Include the requested terminal outcome, immutable source baseline when applicable, target surfaces, design sources, current implementation reference, scope, nonvisual constraints, explicit exclusions, unresolved flags, requested outputs, approval owner, and viewport classification. Stop at request_scope_approval before upload or design mutation.

### verify

Read the newest non-superseded DRES, but preserve prior responses. After authoring, record design_authored. Materialize the candidate in an isolated location and verify archive safety, exact path set, raw byte counts, SHA-256, lineage, expected diff, structural checks, companion coherence, and unexpected drift. Record candidate_verified only after all required checks pass. Visual inspection is a separate human step.

### promote

Require explicit design_acceptance for the exact verified response and candidate identity. Promote or publish the complete verified implementation reference using the configured adapter. Regenerate repository-managed integrity metadata when applicable. Never mix stale repository metadata into a Design-owned payload. Record implementation_reference_accepted only after acceptance and the configured reference update succeed.

### implement

Implementation begins only after implementation_reference_accepted when the requested outcome includes a product change. Record implementation_in_progress before mutation. Respect adapter ownership, tests, and runtime verification. Design tools must not implement backend, policy, game-engine, or persistence behavior unless the project profile explicitly assigns that ownership. Record implementation_verified only after the configured execution-environment evidence passes and implementation_acceptance is explicit.

### deliver

Use the configured delivery adapter only after the required implementation and delivery approvals. Delivery may mean merge, changeset, direct publish, deployment, or another project-defined mechanism. Record delivered only when the target environment actually contains the accepted result and visibility has been checked.

### close

Close only when the request's terminal outcome has been reached. For product-change requests, this requires implementation_verified and delivered. Create the final append-only verification record, record deviations and unresolved defects, then set closed. Never rewrite an earlier verification artifact. For design-only requests, explicitly mark implementation and delivery layers as not requested rather than implying that the product changed.

## Visual review

Keep routine visual review inside the Claude Design session. The user, not another model, approves each screen. An independent reviewer is optional and should receive checkpoint summaries rather than every conversational turn.

The DREQ classifies each viewport as `required`, `optional`, or `not_applicable`. A decision is `approve`, `hold`, `unverified`, or `not_applicable`; never translate missing evidence into "no impact."

Maintain a cumulative decision set and echo the complete set after every answer. If a prior decision disappears or conflicts with a later report, stop before continuing.

A required viewport needs actual evidence from a canonical canvas, component-state specification, or live render. If it remains unverified, design_acceptance stops unless the user explicitly accepts a named deferral to G3 with concrete acceptance checks. Implementation must verify that viewport before commit and must not silently change canon to make it pass.

At completion, produce one `visual-review-summary` containing every screen decision, viewport classification, evidence, explicit implementation-verification deferral, drift decision, overall approve/hold, and a zero-change assertion for the review phase.

## Transport

Choose transport from proven capabilities, not file size alone.

- Use direct shared storage only after project identity and bidirectional visibility are confirmed.
- Use raw direct retrieval only when the receiver can persist exact bytes and independently reproduce the declared hash.
- Use a manifest-bearing zip through the user when content exceeds a hard limit, is truncated, or exact raw retrieval is unavailable.
- A zip relay is not G2 approval.

Do not calculate a claimed remote SHA from re-encoded model text.

## Output integrity

Before emitting or accepting a SHA-256 value, validate it against `^[0-9a-f]{64}$` and re-read it from the verifier artifact. Never rely on manual transcription, abbreviated display text, Markdown emphasis, or a prior conversational summary for an approval-bound digest.

An approval is bound to one verified identity tuple: artifact or response ID, its full digest, the candidate digest, and any immutable repository head named by the gate. Every repeated identifier must agree. If even one approval-bound value is malformed or contradicts verifier evidence, the gate is invalid even when the bytes on disk are otherwise correct. An agent must not silently substitute a measured value for the value the user approved.

A corrected gate requires explicit user ratification that voids the erroneous value and restates the complete verified identity tuple. Post-mutation ratification may repair the audit trail only while the result is still reversible and unmerged; it does not authorize merge or the next gate.

Prefer a verifier-generated approval token or canonical report reference over copying long digests into conversational prose.

After any remote write, re-read the complete file from the target branch and check for truncation, duplication, or encoding damage. A successful API response is not proof of correct content.

A malformed or mismatched digest, or a corrupted remote file, is a hard stop and requires correction before the next gate.

Lead with the lifecycle state and user-impact checkpoint. In normal mode include the blocking semantic gate and one copy-ready next instruction with its destination actor; keep audit evidence collapsed. Automatically propose the next safe action until the requested terminal outcome is reached. Do not make unrelated changes.
