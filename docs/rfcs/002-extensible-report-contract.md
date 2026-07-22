# RFC 002: Extensible Report Contract

- Status: Draft (request for comments)
- Target: `claude-design-sync` plugin core, additive to the existing artifact and schema set
- Base: `codex/alpha4-orchestration` (v0.1.0-alpha.4), authored on top of the RFC 001 branch for a clean docs index
- Author: proposed by the protocol working thread
- Supersedes: none
- Relationship: RFC 001 findings normalize into this contract's `findings`; this contract is logically more foundational, so a maintainer may prefer to land it first.

This RFC proposes structure only. It does not implement a fixed Route Coverage template, a design-to-code status table, runtime renderers, adapters, or schema files in the plugin. The JSON Schema, renderer, and compatibility-test blocks below are illustrative examples for review, not shipped code.

---

## 0. Summary

A specific project's Route Coverage table or design-to-code status list must not become the plugin's fixed output format. Those are one workflow's view. If the core hardcodes them, every other use (code review, incident triage, roadmap, deployment) either does not fit or forces a core change.

This RFC proposes a minimal common **report envelope** plus a **per-kind payload registry**, a **workflow profile** that owns lifecycle, gates, and project fields, a set of **output adapters** that route a report by kind, scope, and size, a **dynamic Markdown renderer** that shows only what matters, a **schema evolution** policy that preserves unknown data, a **fallback** that never loses raw output or a user decision when structuring fails, and a **documentation budget** that persists only boundary results.

The goal is one stable, small core contract that many workflows extend without a plugin update, and that degrades to plain chat when no richer destination exists.

---

## 1. Problem statement

- **Fixed templates do not generalize.** A Route Coverage table encodes one project's routes, viewports, and design-file mapping. A design-to-code status list encodes one pipeline. Baking either into the core makes the core wrong for a backend incident report or a deployment summary.
- **Different workflows have different states and fields.** One project gates on G1/G2/G3 and tracks DREQ/DRES; another gates on a security sign-off and tracks CVE IDs. The core cannot own both vocabularies.
- **Destinations differ.** Sometimes a report belongs in a PR comment, sometimes a tracking issue, a GitHub Check, an ADR, an artifact file, or just chat. The producer should not hardcode one destination, and the system must still work with no GitHub at all.
- **Reports must evolve without breaking consumers.** New fields and new report kinds will appear. An older consumer must not corrupt or drop what it does not understand.
- **Structuring can fail.** A model may return prose that does not parse into a schema. That must not fail the actual work or lose the content; the user still needs the summary and the decision.
- **Over-documentation is its own failure.** Persisting every turn as Markdown buries the few results that matter (a decision, an acceptance, a verification, a release) under intermediate logs.

These pressures are exactly why the plugin already separates a compact checkpoint from an audit plane and keeps project specifics in a profile. This RFC extends that separation to all reporting.

---

## 2. Report envelope (minimal common)

Every report, of every kind, is one envelope. Only the envelope is core. Project and workflow specifics live in `payload` and `extensions`.

Fields:

- `schema_version`: the envelope contract version, `major.minor`.
- `report_id`: stable, unique.
- `kind`: an open string naming the payload shape (Section 3). Open so new kinds need no core change.
- `workflow`: which workflow profile and version produced this (Section 4).
- `subject`: what the report is about (a work item, artifact, PR, route set, or incident), by reference.
- `status`: a coarse, cross-workflow status for interoperability. Fine-grained project state lives in `payload` or `extensions`.
- `summary`: one short human string.
- `facts`: observed, graded statements.
- `findings`: issues, using the RFC 001 and `shared-contracts` finding core.
- `checks`: pass, fail, or blocked checks with evidence references.
- `artifacts`: references to raw output, evidence, screenshots, and attachments, never inlined bodies.
- `decisions_required`: human decisions or gates that block progress.
- `next_actions`: destination-labeled next steps.
- `extensions`: namespaced, open, lossless. This is where a project puts its own fields.
- `raw_ref`: pointer to the preserved raw output artifact (Section 8 fallback).

Coarse `status` vocabulary (interop only): `ok`, `attention`, `blocked`, `failed`, `pending`, `unverified`. A workflow profile maps its own states (for example `candidate_verified`, `delivered`) onto one of these coarse buckets so a generic consumer can act without understanding every project state.

Growth is channeled through `extensions` and `payload`, so the envelope's top-level shape stays stable across a major version.

### Example envelope schema (illustrative)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/tygwan/claude-protocol/schemas/report-envelope.schema.json",
  "title": "Report envelope",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "report_id", "kind", "workflow", "subject", "status", "summary"],
  "properties": {
    "schema_version": {"type": "string", "pattern": "^[0-9]+\\.[0-9]+$"},
    "report_id": {"type": "string", "minLength": 1},
    "kind": {"type": "string", "minLength": 1},
    "workflow": {
      "type": "object",
      "additionalProperties": false,
      "required": ["profile", "version"],
      "properties": {
        "profile": {"type": "string", "minLength": 1},
        "version": {"type": "string", "minLength": 1}
      }
    },
    "subject": {
      "type": "object",
      "additionalProperties": false,
      "required": ["ref"],
      "properties": {
        "ref": {"type": "string", "minLength": 1},
        "title": {"type": ["string", "null"]},
        "revision": {"type": ["string", "null"]}
      }
    },
    "status": {"enum": ["ok", "attention", "blocked", "failed", "pending", "unverified"]},
    "summary": {"type": "string", "minLength": 1, "maxLength": 1000},
    "facts": {"type": "array", "items": {"$ref": "#/$defs/fact"}},
    "findings": {"type": "array", "items": {"$ref": "#/$defs/finding_ref"}},
    "checks": {"type": "array", "items": {"$ref": "#/$defs/check"}},
    "artifacts": {"type": "array", "items": {"$ref": "#/$defs/artifact"}},
    "decisions_required": {"type": "array", "items": {"$ref": "#/$defs/decision"}},
    "next_actions": {"type": "array", "items": {"$ref": "#/$defs/next_action"}},
    "payload": {"type": "object"},
    "extensions": {
      "type": "object",
      "propertyNames": {"pattern": "^[a-z0-9_.-]+/[a-z0-9_.-]+$"},
      "additionalProperties": true
    },
    "raw_ref": {"type": ["string", "null"]}
  },
  "$defs": {
    "fact": {
      "type": "object",
      "additionalProperties": false,
      "required": ["statement"],
      "properties": {
        "statement": {"type": "string", "minLength": 1},
        "grade": {"enum": ["E0", "E1", "E2", "E3", "E4", null]},
        "evidence_ref": {"type": ["string", "null"]}
      }
    },
    "finding_ref": {
      "type": "object",
      "additionalProperties": false,
      "required": ["id", "severity"],
      "properties": {
        "id": {"type": "string", "minLength": 1},
        "severity": {"enum": ["blocker", "major", "moderate", "minor"]},
        "claim_type": {"enum": ["FACT", "INFERENCE", "RECOMMENDATION", "HYPOTHESIS", "DECISION_REQUIRED", "UNVERIFIED"]},
        "ref": {"type": ["string", "null"]}
      }
    },
    "check": {
      "type": "object",
      "additionalProperties": false,
      "required": ["name", "result"],
      "properties": {
        "name": {"type": "string", "minLength": 1},
        "result": {"enum": ["pass", "fail", "blocked", "skipped", "unverified"]},
        "evidence_ref": {"type": ["string", "null"]}
      }
    },
    "artifact": {
      "type": "object",
      "additionalProperties": false,
      "required": ["role", "ref"],
      "properties": {
        "role": {"type": "string", "minLength": 1},
        "ref": {"type": "string", "minLength": 1},
        "media_type": {"type": ["string", "null"]}
      }
    },
    "decision": {
      "type": "object",
      "additionalProperties": false,
      "required": ["question", "owner"],
      "properties": {
        "question": {"type": "string", "minLength": 1},
        "owner": {"type": "string", "minLength": 1},
        "gate": {"type": ["string", "null"]},
        "options": {"type": "array", "items": {"type": "string"}}
      }
    },
    "next_action": {
      "type": "object",
      "additionalProperties": false,
      "required": ["actor", "action"],
      "properties": {
        "actor": {"type": "string", "minLength": 1},
        "action": {"type": "string", "minLength": 1},
        "stop_condition": {"type": ["string", "null"]}
      }
    }
  }
}
```

Note how the envelope references, not inlines: `findings` carry an `id` and a `ref` to the full finding record; `artifacts` and evidence are pointers. This keeps the envelope small and the durable protocol records (checkpoint, event, handoff, receipt, DREQ, DRES, DVER) authoritative.

---

## 3. Kind payload registry

`kind` selects a payload schema from a registry. The core ships a base set; a workflow profile may register more without a plugin update. Base kinds:

`discovery`, `plan`, `design_request`, `design_response`, `implementation`, `review`, `verification`, `visual_verification`, `acceptance`, `merge`, `deployment`, `incident`, `audit`, `coverage`, `unstructured`.

Several map to existing protocol artifacts so nothing is duplicated: `design_request` corresponds to DREQ, `design_response` to DRES, `verification` to DVER, `acceptance` to a gate approval, `audit` to the audit plane. The envelope carries the summary and references; the durable artifact stays the source of truth.

`unstructured` is the guaranteed-present fallback kind (Section 7). It carries only `summary`, `status`, `next_actions`, and `raw_ref`.

A payload schema validates only its own fields. It never redefines envelope fields. Unknown `kind` values are handled as `unstructured` and their `payload` is preserved verbatim.

### Example: `coverage` payload, generic (illustrative)

Route Coverage is not a core template. The core defines a generic coverage payload; a project expresses routes as a profile extension.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/tygwan/claude-protocol/schemas/kinds/coverage.payload.schema.json",
  "title": "Coverage payload (generic)",
  "type": "object",
  "additionalProperties": false,
  "required": ["dimension", "items"],
  "properties": {
    "dimension": {"type": "string", "minLength": 1},
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["id", "state"],
        "properties": {
          "id": {"type": "string", "minLength": 1},
          "label": {"type": ["string", "null"]},
          "state": {"type": "string", "minLength": 1},
          "evidence_ref": {"type": ["string", "null"]},
          "attributes": {"type": "object"}
        }
      }
    }
  }
}
```

A route-coverage report is then just `kind: coverage`, `dimension: "route"`, with each item carrying `id: "/checkout"` and project-specific fields (viewport, design file) under `items[].attributes` or the envelope `extensions`, declared by the project profile. The design-to-code pilot's Route Coverage table becomes a rendered view of this generic payload, not a core structure.

---

## 4. Workflow profile

A workflow profile declares what the core deliberately does not own:

- **lifecycle states** and how each maps onto a coarse envelope `status`.
- **gates** (for example G1/G2/G3, or a security sign-off) and their decision owners.
- **kind extensions**: additional kinds this workflow emits, with their payload schemas.
- **field extensions**: the namespaced `extensions` keys this workflow uses, and their meaning.
- **adapters**: which output destinations are available and the default routing (Section 5).
- **documentation budget**: which kinds are persisted versus transient (Section 8).

So G1/G2/G3, DREQ/DRES, and route coverage are profile-owned. A different project declares different states and fields and reuses the same envelope and registry mechanism. The design-sync profile is one such workflow profile; a code-review or incident workflow is another, with no core change.

### Example workflow-profile fragment (illustrative)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://github.com/tygwan/claude-protocol/schemas/workflow-profile.schema.json",
  "title": "Workflow profile",
  "type": "object",
  "additionalProperties": false,
  "required": ["profile", "version", "status_map"],
  "properties": {
    "profile": {"type": "string", "minLength": 1},
    "version": {"type": "string", "minLength": 1},
    "lifecycle_states": {"type": "array", "items": {"type": "string"}},
    "status_map": {
      "type": "object",
      "additionalProperties": {"enum": ["ok", "attention", "blocked", "failed", "pending", "unverified"]}
    },
    "gates": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["name", "owner"],
        "properties": {
          "name": {"type": "string"},
          "owner": {"type": "string"},
          "alias": {"type": ["string", "null"]}
        }
      }
    },
    "kind_extensions": {
      "type": "object",
      "additionalProperties": {"type": "string"}
    },
    "field_extensions": {
      "type": "object",
      "additionalProperties": {"type": "string"}
    },
    "adapters": {"type": "array", "items": {"type": "string"}},
    "persist_kinds": {"type": "array", "items": {"type": "string"}}
  }
}
```

---

## 5. Output adapters

The producer emits one envelope. An adapter layer chooses the destination by `kind`, scope, and size:

| Adapter | Typical use |
|---|---|
| PR comment | a review or verification tied to a pull request; updates one canonical comment |
| tracking issue | a longer-lived roadmap, coverage, or incident record |
| GitHub Check | a pass, fail, or blocked gate that should show in CI status |
| Artifact (file) | the full envelope, raw output, and evidence; always written for durability |
| ADR | an accepted decision that should live in the repository's decision log |
| plain chat | the default, and the only adapter required |

Rules:

- Selection is capability-based, not hardcoded. If GitHub is absent, PR comment, issue, and Check adapters are unavailable, and the system falls back to Artifact plus chat. Nothing fails for lack of GitHub.
- A large report goes to an Artifact with a short chat or comment summary that references it, never a wall of text pasted into a comment.
- The Artifact adapter is always available and always writes the lossless envelope and raw output, so every other adapter is a view, not the system of record.

---

## 6. Dynamic Markdown renderer

The renderer turns an envelope into human Markdown. It is dynamic, not a fixed template:

- **Value-only sections.** A section renders only if it has content. An empty `findings` list produces no Findings heading.
- **Abbreviate the normal.** Passing checks and `ok` items collapse into a one-line count. The reader is not made to scan a hundred green rows.
- **Detail the exceptional.** Only `fail`, `blocked`, and `decisions_required` expand into full detail with evidence references.
- **One canonical comment.** On a PR or issue, the renderer updates a single marked comment (an identity marker in the body) rather than posting a new comment each turn.
- **Preserve everything.** The rendered Markdown is a view. The full envelope and the raw output are preserved as Artifacts, so nothing shown-in-brief is lost.

### Example: same envelope, two renders (illustrative)

Given an envelope with 40 passing checks, 1 failing check, and 1 required decision, the compact render is:

```markdown
<!-- report:canonical id=RPT-88 -->
### review: /checkout  (status: blocked)

40 checks passed. 1 failed, 1 decision needed.

**Failed check**
- contrast-min: fail. Evidence: artifact://checks/contrast-88.json

**Decision required**
- Ship with the known contrast issue behind a flag? Owner: product_owner. Gate: implementation_acceptance.

Full report: artifact://reports/RPT-88.json
```

The same envelope with all checks passing and no decision renders as a single line:

```markdown
<!-- report:canonical id=RPT-88 -->
### review: /checkout  (status: ok)

41 checks passed. No blockers. Full report: artifact://reports/RPT-88.json
```

No fixed table is imposed. A project that wants a route-coverage table renders its `coverage` payload into one through a profile-declared view, while the core renderer stays generic.

---

## 7. Schema evolution

- `schema_version` is `major.minor` on the envelope, and each payload and profile schema is versioned independently.
- **Backward-compatible growth is a minor bump**: adding an optional field, a new kind, or a new `extensions` namespace.
- **A breaking change is a major bump**: removing or renaming a core field, or tightening a type.
- **Unknown data is preserved losslessly.** A consumer at an older minor version keeps unknown fields and unknown `extensions` namespaces byte-for-byte; it must not strip them on read or rewrite.
- **Ignore, do not delete.** A consumer that does not understand an extension ignores it for its own logic but never removes it from the stored report.
- The canonical stored form is the raw envelope JSON. Validators are advisory and versioned; a validator that rejects a newer minor does not authorize deleting the report.

---

## 8. Fallback

Structuring is best-effort and never blocks the actual work:

- If a result cannot be structured into a kind payload, the work does **not** fail.
- The raw response is preserved as an Artifact and referenced by `raw_ref`.
- A minimal envelope is still emitted with `kind: unstructured`, carrying only `summary`, `status`, and `next_actions`, extracted best-effort from the raw text.
- The user still receives the summary, the status, and the next action, so an unstructured result is deliverable, not lost.

This guarantees the system degrades to "here is the raw output plus a one-line summary and next step," never to silence or a hard failure.

---

## 9. Documentation budget

- Not every turn is documented. Intermediate reasoning and repeated logs go to an Artifact or a transient ledger, not a durable record.
- Only boundary results are persisted: a decision, an acceptance, a verification, a release, an incident resolution.
- The persisted set per workflow is declared by the profile's `persist_kinds`, so one project persists `acceptance` and `verification` while another also persists `deployment`.
- This reuses the existing separation of a compact checkpoint from an expandable audit plane, applied to reporting.

---

## 10. Compatibility tests (proposed)

These are the tests the eventual implementation must pass. They map one to one onto the evaluation criteria.

1. **Express the design-to-code pilot.** A route-coverage report round-trips as `kind: coverage`, `dimension: route`, with route rows and their design-file mapping under `extensions`, and renders to the pilot's table view through the profile, with no core template.
2. **Express unrelated workflows with no core change.** A code review (`kind: review`), an incident investigation (`kind: incident`), a roadmap (`kind: coverage`, `dimension: milestone`), and a deployment (`kind: deployment`) each validate against the same envelope and their kind payloads, using only profile-declared extensions.
3. **Register a new kind or profile without a plugin update.** A profile adds a `kind: security_signoff` with its payload schema and it validates and renders, with no change to the core envelope.
4. **Round-trip an unknown extension losslessly.** A report carrying `extensions["acme/route-coverage"]` is read, re-rendered, and re-serialized by a consumer that does not understand that namespace, and the namespace survives byte-for-byte.
5. **Survive a structuring failure.** A prose-only result yields a valid `kind: unstructured` envelope with `raw_ref` set, a non-empty `summary`, a `status`, and at least one `next_action`, and the underlying work is not marked failed.
6. **Work without GitHub.** With no GitHub available, the same envelope is delivered via the Artifact and chat adapters, and the canonical-comment behavior degrades to a chat summary that references the Artifact.
7. **Minor-version forward compatibility.** An envelope at `schema_version` `1.1` with a new optional field is accepted by a `1.0` consumer, which preserves the field.
8. **Major-version guard.** An envelope at `2.0` with a removed core field is rejected by a `1.x` validator without deleting the stored report.

---

## 11. Evaluation criteria

- The existing design-to-code pilot is expressible without a fixed template.
- Code review, incident investigation, roadmap, and deployment reports are expressible with no core change.
- New kinds and profiles register without a plugin update.
- Unknown extensions survive a round-trip losslessly.
- A structuring failure loses neither the raw output nor the user's decision.
- The system works via chat and file adapters with no GitHub.

---

## 12. Relationship to existing protocol and RFC 001

- The durable records (checkpoint, event, handoff envelope, receiver receipt, DREQ, DRES, DVER) remain the source of truth. This envelope is the reporting and output layer that references them.
- RFC 001 findings and the `shared-contracts` finding core populate `findings`. The `fact` grades reuse the E0 to E4 ladder. The `decisions_required` gates reuse the semantic gate names. The `next_actions` reuse the destination-labeled actor model.
- The coarse `status` maps onto the existing lifecycle states through the workflow profile's `status_map`, so the compact checkpoint and this envelope agree.

Nothing here changes an existing schema. The envelope, the kind registry, the workflow profile, the adapters, and the renderer are additive.

---

## Scope and non-goals

**In scope**: the envelope, the kind registry, the workflow profile, the adapters, the dynamic renderer, schema evolution, fallback, the documentation budget, and illustrative schema, renderer, and compatibility-test examples.

**Non-goals**:

- no fixed Route Coverage template or design-to-code status table in the core,
- no runtime renderer, adapter, or schema files shipped in this PR,
- no changes to existing `design-sync`, `design-review`, or alpha.4 schemas,
- no project-specific states or fields in the core,
- no merge.

## Not implemented in this PR

This PR adds only this RFC document and a docs index entry. The schema, renderer, and test blocks are proposals for review.

## Open questions

1. Should the coarse `status` vocabulary be exactly the six proposed buckets, or fewer?
2. Is `kind` fully open, or should the core reserve a known set and namespace project kinds (for example `acme/security_signoff`)?
3. Where does the canonical-comment identity marker live so it is stable across renderer versions?
4. What is the minimum extraction guaranteed for `unstructured` fallback (summary and status only, or also a best-effort next action)?
5. Should `persist_kinds` have a core default, or is persistence always profile-declared?
6. How should an adapter decide the size threshold that pushes a report from a comment to an Artifact?
7. Should the envelope carry a producer identity and signature for audit, or is that left to the referenced durable records?

## Review request

Codex and Claude reviewers are asked to evaluate:

- whether the envelope's twelve common fields are the right minimal set,
- whether `extensions` plus a per-kind payload registry fully removes the need for any fixed project template,
- the schema-evolution rules for lossless round-trip of unknown data,
- the fallback path for unstructured results,
- the no-GitHub chat and file adapter behavior,
- the boundary between this report layer and the existing durable protocol records.
