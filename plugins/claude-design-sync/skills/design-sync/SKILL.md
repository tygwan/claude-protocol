---
name: design-sync
description: Orchestrate a human-gated design handoff between a repository and Claude Design. Use when a task needs a design brief, DREQ/DRES/DVER artifacts, design-canon verification, visual approval, or promotion of a design snapshot into code.
argument-hint: "[status|bootstrap|request|verify|promote|implement|close]"
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

## State machine

Only advance in this order:

`discover -> DREQ draft -> G1 -> design -> DRES -> verify -> visual review -> G2 -> promote -> implement -> G3 -> DVER`

A failed prerequisite keeps the current state. It never implies approval of the next state.

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

## Action behavior

### status

Read only. Report project identity, current state, authoritative artifact IDs, unresolved flags, last explicit human gate, and next safe action.

### bootstrap

Discover existing project conventions first. Propose, but do not silently impose, a project profile and handoff locations. If canonical project instructions must change, stop for the project's required structural approval.

### request

Draft one append-only DREQ. Include immutable repo base, target routes, target design files, current canon bytes/SHA, scope, nonvisual constraints, explicit exclusions, unresolved flags, requested outputs, approval owner, and viewport classification. Stop at G1 before upload or design mutation.

### verify

Read the newest non-superseded DRES, but preserve prior responses. Materialize the candidate in an isolated location. Verify archive safety, exact path set, raw byte counts, SHA-256, lineage, expected diff, structural checks, and unexpected drift. Visual inspection is a separate human step.

### promote

Require an explicit G2 approval for the exact verified DRES and candidate digest. Promote the complete verified snapshot only. Regenerate repository-managed integrity metadata after promotion. Never mix stale repository metadata into a Design-owned payload.

### implement

Implementation begins only after canon promotion. Respect repository ownership, tests, and pre-commit live-browser gate. Design tools must not implement backend, policy, game-engine, or persistence behavior unless the project profile explicitly assigns that ownership.

### close

Create DVER only after implementation and the required deployment or browser verification. Record deviations and unresolved defects. Never rewrite an earlier DVER.

## Visual review

Keep routine visual review inside the Claude Design session. The user, not another model, approves each screen. An independent reviewer is optional and should receive checkpoint summaries rather than every conversational turn.

The DREQ classifies each viewport as `required`, `optional`, or `not_applicable`. A decision is `approve`, `hold`, `unverified`, or `not_applicable`; never translate missing evidence into "no impact."

Maintain a cumulative decision set and echo the complete set after every answer. If a prior decision disappears or conflicts with a later report, stop before continuing.

A required viewport needs actual evidence from a canonical canvas, component-state specification, or live render. If it remains unverified, G2 stops unless the user explicitly accepts a named deferral to G3 with concrete acceptance checks. Implementation must verify that viewport before commit and must not silently change canon to make it pass.

At completion, produce one `visual-review-summary` containing every screen decision, viewport classification, evidence, explicit G3 deferral, drift decision, overall approve/hold, and a zero-change assertion for the review phase.

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

Lead with the current state and blocking gate. Include evidence, unexpected drift, and one copy-ready next instruction with its destination actor. Do not make unrelated changes.
