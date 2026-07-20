# Roadmap

## v0.1 alpha — capture the proven protocol

- [x] Claude Code plugin and marketplace skeleton
- [x] Actor boundaries and human gates
- [x] Append-only DREQ, DRES, and DVER templates
- [x] Whole-snapshot verification rules
- [x] Capability-based transport fallback
- [x] Cumulative visual-review decisions
- [x] Modular design, UI, UX, accessibility, responsive, and domain-boundary review framework
- [ ] Complete the first pilot through DVER
- [ ] Validate with `claude plugin validate .`
- [ ] Local installation smoke test

## v0.2 alpha — deterministic verification

- [ ] Add a portable verifier CLI
- [ ] Validate zip safety, path sets, bytes, and SHA-256
- [ ] Generate candidate digests and machine-readable reports
- [ ] Add protocol state and lineage validation
- [ ] Add fixtures for truncation, stale metadata, partial snapshots, and superseded responses

## v0.3 alpha — adapter separation

- [ ] Define repository-orchestrator adapter interface
- [ ] Define Claude Design capability discovery
- [ ] Add optional Codex repository adapter
- [ ] Keep the protocol core vendor-neutral where evidence permits

## beta

- [ ] Complete a second pilot in the originating project
- [ ] Complete a pilot in an unrelated project
- [ ] Measure human relay count, gate latency, false stops, and recovery cost
- [ ] Test Windows, WSL, macOS, and Linux path behavior
- [ ] Document migration from project-local rules

## v1

- [ ] Stable artifact schemas
- [ ] Backward-compatible configuration policy
- [ ] Deterministic verification available without model transcription
- [ ] No mandatory independent-agent relay
- [ ] Clear security and support policy
- [ ] Release notes based on multi-project evidence
