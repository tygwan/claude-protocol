# Pilot log

This log records evidence from the first real design-sync pilot. Product-specific policy is intentionally excluded from the reusable skill.

## Pilot 001 — design recut and canon promotion preflight

Status: in progress. Completed through Design response, verified zip recovery, technical G2 preflight, and conditional human visual approval. Explicit G2 promotion approval is pending.

### Confirmed capabilities

- A correctly bound Design project can expose persistent files to a repository-side DesignSync reader in the same session.
- Bidirectional canaries confirmed new-file creation and existing-file modification visibility.
- Design can publish append-only requests and responses in shared project storage.
- A manifest-bearing package can be downloaded by the user and independently verified on disk.
- A whole candidate snapshot can be checked against a clean immutable repository base without touching a dirty legacy worktree.

### Observed failures and protocol responses

| Observation | Risk | Protocol response |
|---|---|---|
| A session initially inspected the wrong Design project | Silent cross-project contamination | Require project identity plus file fingerprint and retired-ID denylist |
| A large HTML file was byte-truncated during tool retrieval | Corrupt canon and false hashes | Treat truncation as a hard stop; forbid text reconstruction |
| Remote size/hash metadata could not independently prove local raw bytes | False integrity confidence | Choose transport by capability, not size; use verified zip fallback |
| Package metadata lagged behind the repository while design files were current | Whole-snapshot promotion would revert protocol docs | Separate Design-owned payload from repository-managed metadata and diff the full snapshot |
| A malformed closing tag was found after the first DRES | Broken canvas structure | Preserve the old DRES and publish a superseding sequence |
| A request was sent to the wrong actor | Ownership boundary violation | Put an explicit destination label on every instruction and hard-stop wrong-actor work |
| Human approval was remembered in one turn and missing in the next | Gate loss through chat state | Maintain a cumulative decision ledger and stop on inconsistent state |
| Viewport approval was requested where no dedicated render existed | Unsupported "no impact" inference | Classify viewport coverage in DREQ; record missing evidence as unverified; require explicit G3 deferral |
| A component-state strip approximated mobile width but no full mobile screen existed | Component evidence could be mistaken for full-page evidence | Record the evidence type and require the actual composition in G3 |
| The user relayed every intermediate message through an independent reviewer | Manual bottleneck | Keep routine visual review inside Design; make independent audit optional |
| A dirty long-lived worktree existed beside the pilot | Accidental loss of unrelated work | Use a clean sibling worktree from an immutable base and never clean the legacy tree |

### Visual review checkpoint

- Four screen-level decisions were explicitly approved: explore, detail, guardian sheet, and child completion.
- Mobile 390 was approved from canonical frames, DOM measurements, and a component-state specification; the full Explore restricted composition was deferred to G3.
- Tablet 768 was recorded as unverified rather than "no impact." The user explicitly allowed deferral to G3 because viewport coverage was absent from the DREQ.
- Desktop 1280 was approved only where canon evidence existed. Mobile-only routes were marked not applicable instead of being inferred.
- The pre-existing documentation-only drift was explicitly preserved.
- G3 inherited concrete blocking checks. Visual approval did not execute or imply G2.
- The review phase changed zero files.

### Open questions

- Durable, machine-readable human approval signatures
- A deterministic verifier CLI for archive safety, manifests, bytes, and SHA-256
- Standard adapters for differing Design tool capabilities
- Reliable viewport capture and visual-diff evidence
- Ownership rules for package metadata in projects that mix authoring and canon documentation
- Migration rules for projects that do not use whole-snapshot promotion

### Alpha exit evidence still required

- Complete G2 promotion into repository canon
- Implement the approved design
- Complete live-browser G3
- Publish DVER
- Run a second pilot in the same project
- Run at least one pilot in a different project
