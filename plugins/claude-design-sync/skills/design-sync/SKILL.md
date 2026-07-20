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

Draft one append-only DREQ. Include immutable repo base, target routes, target design files, current canon bytes/SHA, scope, nonvisual constraints, explicit exclusions, unresolved flags, requested outputs, and approval owner. Stop at G1 before upload or design mutation.

### verify

Read the newest non-superseded DRES, but preserve prior responses. Materialize the candidate in an isolated location. Verify archive safety, exact path set, raw byte counts, SHA-256, lineage, expected diff, structural checks, and unexpected drift. Visual inspection is a separate human step.

### promote

Require an explicit G2 approval for the exact verified DRES and candidate digest. Promote the complete verified snapshot only. Regenerate repository-managed integrity metadata after promotion. Never mix stale repository metadata into a Design-owned payload.

### implement

Implementation begins only after canon promotion. Respect repository ownership, tests, and pre-commit live-browser gate. Design tools must not implement backend, policy, game-engine, or persistence behavior unless the project profile explicitly assigns that ownership.

### close

Create DVER only after implementation and the required deployment or browser verification. Record deviations and unresolved defects. Never rewrite an earlier DVER.

## Visual review

Keep the visual review inside the Claude Design session when possible. The user, not another model, approves each screen.

Maintain a cumulative decision set and echo the complete set after every answer. If a prior decision disappears or conflicts with a later report, stop before continuing.

At completion, produce one `visual-review-summary` containing every screen decision, viewport decision, drift decision, overall approve/hold, and a zero-change assertion for the review phase.

## Transport

Choose transport from proven capabilities, not file size alone.

- Use direct shared storage only after project identity and bidirectional visibility are confirmed.
- Use raw direct retrieval only when the receiver can persist exact bytes and independently reproduce the declared hash.
- Use a manifest-bearing zip through the user when content exceeds a hard limit, is truncated, or exact raw retrieval is unavailable.
- A zip relay is not G2 approval.

Do not calculate a claimed remote SHA from re-encoded model text.

## Output

Lead with the current state and the blocking gate. Include evidence, unexpected drift, and one copy-ready next instruction with its destination actor. Do not make unrelated changes.
