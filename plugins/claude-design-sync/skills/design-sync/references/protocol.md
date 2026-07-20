# Protocol invariants

## Roles

The protocol uses roles rather than hard-coding one orchestration product.

- Human owner: approves scope and promotion.
- Repository orchestrator: writes DREQ and verification artifacts, validates candidates, promotes canon, and coordinates implementation.
- Design author: owns the visual authoring source and writes DRES.
- Independent auditor: optional; reviews protocol or disputed evidence but is not a mandatory relay.
- Domain decider: resolves clinical, legal, safety, or other regulated flags.

Claude Code is the first repository-orchestrator adapter. Claude Design is the first design-author adapter. A future Codex adapter may fulfill repository orchestration without changing the core state machine.

## Authority boundaries

Projects must declare:

- design authoring source,
- repository implementation canon,
- repository-managed metadata,
- implementation paths,
- handoff storage,
- human approval roles,
- regulated decision roles.

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

## Gates

- G0: protocol or ownership-model change.
- G1: human approves the design request scope before Design mutation.
- G2: human approves the exact verified candidate before canon promotion.
- G3: human approves live-browser implementation before commit or merge, as configured.
- G4: named domain decider resolves regulated flags.

Transport, model recommendations, CI success, and prior approvals do not satisfy a gate.

## Snapshot promotion

Default to whole-snapshot promotion when Design publishes a package.

Verification must compare:

1. declared manifest path set,
2. extracted candidate path set,
3. current repository canon path set,
4. byte count for every candidate file,
5. SHA-256 for every candidate file,
6. expected and unexpected diff,
7. repository-managed metadata exclusions.

No partial overlay is allowed unless the project profile explicitly defines a different atomic unit and its consistency rules.

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
- explicit G3 deferral and acceptance checks, if any,
- intentional drift decision,
- overall approve or hold.

Missing evidence is `unverified`, never "no impact." A required viewport blocks G2 unless the user explicitly defers that exact coverage to G3. Such a deferral must identify the live-render checks that will block implementation commit if they fail.

Conversation memory alone is not a durable approval ledger. Until a signed artifact format is implemented, echo the full decision set after each step and require the final summary to match.

## Project profile

Keep product-specific details outside the core skill:

- repository and base branch,
- design project name, ID, and file fingerprint,
- retired project IDs,
- canon path,
- design root patterns,
- package path,
- handoff paths,
- transport limits,
- atomic promotion unit,
- route-to-design-file mapping,
- required checks,
- domain flags and decision owners.

Never copy one project's clinical or product policy into the reusable core.
