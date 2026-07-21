# Typed handoff and durable events

The protocol separates producer claims, receiver observations, and workflow state. This prevents a confident message from becoming evidence of bytes or authority that the receiver never verified.

Machine-readable artifacts use the schemas under `plugins/claude-design-sync/schemas/`. Markdown DREQ, DRES, and DVER files remain the human-readable decision records.

## Two-phase handoff

### Phase 1: producer envelope

The producer writes one immutable envelope containing:

- protocol and schema version,
- envelope, work, and source artifact identifiers,
- producer and intended consumer roles,
- source revision or design-project identity,
- declared payload paths and media types,
- declared bytes and digest when the producer can measure raw bytes,
- `digest_status`,
- required receiver checks,
- the next semantic gate, if any.

If the producer cannot measure raw bytes, use:

```yaml
declared_digest: null
digest_status: unavailable_at_source
```

Never write `pending` as an approval-bound digest. A conversationally reconstructed hash is not a measured digest.

### Phase 2: receiver receipt

The receiver independently reads or materializes the payload and writes one immutable receipt:

- envelope ID and receiver identity,
- received path set,
- raw bytes and measured digest,
- identity, lineage, path-set, digest, and truncation checks,
- `accepted | rejected | partial`,
- reasons and recovery action,
- resulting lifecycle state and next actor.

An accepted receipt proves reception, not a human gate. It cannot approve scope, design, implementation, delivery, or regulated policy.

If the payload is partial, stale, truncated, or mismatched, the receiver writes a rejected or partial receipt and stops. It does not repair the producer artifact in place.

## Identity rules

Approval binds to the receiver-verified identity tuple:

1. source artifact ID,
2. source artifact digest,
3. candidate snapshot digest,
4. immutable source revision when the profile requires one.

The envelope may declare that tuple. The receipt determines whether it was reproduced. A missing source digest can be supplied by the receiver only as a measured value; it remains distinct from what the producer declared.

## Event log

Persist one append-only NDJSON event stream per work item. Events are control records, not prose transcripts. Each line validates against `event.schema.json`.

Record only transitions and decisions:

- request created or superseded,
- handoff sent or received,
- verification passed or failed,
- gate approved or held,
- lifecycle state changed,
- delivery completed or failed,
- recovery or closure.

Do not log model chain-of-thought, full chat messages, copied manifests, screenshots, or repeated artifact bodies. Store references to those records.

Events are append-only and ordered by `occurred_at` plus `event_id`. If an event is wrong, append a correction event that references it.

## Resume algorithm

On `continue` or a new session:

1. Load the project profile and active-request index.
2. Validate protocol and schema compatibility.
3. Read the append-only event stream.
4. Resolve supersession for DREQ, DRES, DVER, envelopes, and receipts.
5. Find an inbound envelope without an accepted or rejected receipt.
6. Re-run only read-only checks until reception is known.
7. Reconstruct the latest evidence-supported lifecycle state and semantic gate.
8. Compare it with the compact checkpoint.
9. If they disagree, keep the lower proven state and expand audit output.
10. Propose or execute exactly one next safe action.

Never repeat a mutation because a prior assistant message is missing. Idempotency comes from artifact and event IDs, not chat memory.

## Documentation budget

The core defines what information must survive. The project profile defines where and how long it survives.

Persist:

- current checkpoint,
- event stream,
- immutable artifacts and receipts,
- approval references,
- unresolved flags,
- evidence references needed to reproduce a gate.

Do not persist:

- routine navigation instructions,
- duplicated hashes already referenced by an immutable report,
- full reviewer monologues,
- exploratory thoughts that did not affect a decision.

Profiles declare storage, retention, compaction, and privacy. Compaction may replace old events with a signed or verified snapshot only if artifact lineage and gate evidence remain reproducible.

## Audit expansion

Expand audit output whenever any invariant or authorization is ambiguous, including but not limited to verification failure, identity mismatch, changed approval target, stale lineage, conflicting receipts, schema incompatibility, recovery, or explicit audit request. Normal-mode brevity never hides uncertainty about permission or evidence.
