# Artifact templates

These are minimal shapes. Projects may add fields but should not weaken lineage, ownership, or gate requirements.

## DREQ

```yaml
---
request_id: DREQ-{work_id}-{request_seq}
supersedes: null
source_work_item: "{id-or-url}"
source_baseline_revision: "{immutable-revision}"
source_baseline_ref: "{optional-ref}"
design_project_name: "{name}"
design_project_id: "{id-or-unavailable}"
created_at: "{ISO-8601}"
requested_outcome: "{design_only|implementation_reference|implemented|delivered}"
lifecycle_state: request_drafting
max_file_size_kb: 200
request_scope_approval_owner_role: product_owner
target_routes: []
target_design_files: []
base_files:
  - design_path: "{path}"
    implementation_reference_path: "{path}"
    bytes: 0
    sha256: "{sha256}"
requested_outputs:
  - changed_design_files
  - promotion_snapshot_manifest
---

## design scope

## nonvisual constraints

## explicitly out of scope

## unresolved domain flags

## required pre-existing drift attribution
```

## DRES

```yaml
---
request_id: DREQ-{work_id}-{request_seq}
response_id: DRES-{work_id}-{request_seq}-{response_seq}
supersedes: null
result_status: design_authored
lifecycle_state: design_authored
completed_at: "{ISO-8601}"
changed_root_files: []
created_root_files: []
unchanged_reference_files: []
files_orchestrator_should_fetch: []
promotion_snapshot_files: []
---

## changed screens, states, and interactions

## pre-existing drift analysis

## requested-change attribution

## accessibility behavior

## unresolved flags

## implementation prohibitions
```

Create DRES only after all Design file edits, package refresh, byte counting, and SHA-256 calculation are complete.

## visual-review-summary

```yaml
artifact_id: DRES-{work_id}-{request_seq}-{response_seq}
candidate_digest: "{digest}"
decisions:
  - screen: "{screen-and-state}"
    decision: approve
    reason: null
viewports:
  - width: 390
    classification: required
    decision: approve
    evidence: "{canonical-canvas-or-live-render}"
    deferred_to: null
    acceptance_checks: []
  - width: 768
    classification: required
    decision: unverified
    evidence: null
    deferred_to: implementation_acceptance
    acceptance_checks:
      - no_overflow
      - touch_targets
      - responsive_layout
intentional_drift:
  - id: "{drift-id}"
    decision: preserve
overall: approve
files_changed_during_review: 0
approved_gate: design_acceptance
approved_by_role: product_owner
```

## workflow-checkpoint

```yaml
schema_version: 1.0.0-alpha.4
work_id: "{work_id}"
requested_outcome: delivered
lifecycle_state: candidate_verified
design_artifacts_changed: true
actual_product_changed: false
visible_in_target_environment: "no"
blocking_gate: design_acceptance
blockers: []
next_action: accept_verified_implementation_reference
next_actor: product_owner
last_event_id: EVT-{work_id}-{event_seq}
active_artifacts:
  - DREQ-{work_id}-{request_seq}
audit_reference: "{verification-report-id-or-path}"
updated_at: "{ISO-8601}"
```

Use this compact shape at every pause and session resume. Do not expose the audit reference contents in normal mode unless verification failed, identity changed, recovery is required, or the user requests audit.

## machine handoff envelope

The JSON record must validate against `schemas/handoff-envelope.schema.json`. Keep the artifact body and large manifest outside the envelope and reference them by path.

```yaml
schema_version: 1.0.0-alpha.4
envelope_id: ENV-{work_id}-{sequence}
work_id: "{work_id}"
artifact_id: DRES-{work_id}-{request_seq}-{response_seq}
artifact_kind: DRES
producer: {role: design_author, adapter: claude_design}
consumer: {role: workflow_orchestrator, adapter: claude_code}
created_at: "{ISO-8601}"
payload:
  - path: _handoff/responses/DRES-{work_id}-{request_seq}-{response_seq}.md
    media_type: text/markdown
    bytes: null
    sha256: null
integrity:
  declared_digest: null
  digest_status: unavailable_at_source
  manifest_reference: "{path-or-null}"
required_receiver_checks: [identity, lineage, path_set, raw_bytes, digest, truncation, schema]
next_semantic_gate: design_acceptance
```

## receiver receipt

The receiver writes this after independent retrieval and validates it against `schemas/receiver-receipt.schema.json`.

```yaml
schema_version: 1.0.0-alpha.4
receipt_id: RCP-{work_id}-{sequence}
envelope_id: ENV-{work_id}-{sequence}
work_id: "{work_id}"
receiver: {role: workflow_orchestrator, adapter: claude_code}
received_at: "{ISO-8601}"
status: accepted
checks:
  identity: true
  lineage: true
  path_set: true
  raw_bytes: true
  digest: true
  truncation: true
  schema: true
measured_payload:
  - {path: "{path}", bytes: 0, sha256: "{sha256}"}
measured_candidate_digest: "{sha256}"
reasons: []
recovery_action: null
resulting_state: candidate_verified
next_actor: product_owner
```

Events are one JSON object per line and validate against `schemas/event.schema.json`. Do not use Markdown chat summaries as the event stream.

## DVER

```yaml
---
request_id: DREQ-{work_id}-{request_seq}
verification_id: DVER-{work_id}-{request_seq}-{verification_seq}
implementation_revision: "{immutable-revision}"
change_delivery_reference: "{adapter-specific-reference}"
requested_outcome: delivered
lifecycle_state: closed
implementation_reference: "{artifact-id-or-version}"
implemented_surfaces: []
verification_environments: []
viewports: []
implementation_acceptance: "{approved|held|not_required}"
delivery_mechanism: "{merge|changeset|publish|deploy|other}"
delivery_status: "{not-run|delivered|failed}"
delivered_at: "{ISO-8601-or-null}"
target_environment: "{environment-or-not-applicable}"
visible_to_target_users: "{yes|no|unverified}"
design_rereview_requested: false
---

## screenshots

## visual differences

## accessibility results

## intentional deviations

## unresolved defects

## closure decision
```
