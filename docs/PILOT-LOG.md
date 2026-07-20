# Pilot log

This log records evidence from the first real design-sync pilot. Product-specific policy is intentionally excluded from the reusable skill.

## Pilot 001 — design recut and canon promotion preflight

Status: in progress. Completed through Design response, verified zip recovery, conditional human visual approval, creation of an unmerged G2 promotion PR, explicit artifact ratification, and successful CI recovery. Merge is blocked by stale implementation briefs discovered in review.

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
| An approval prompt manually transcribed a verified zip SHA with one extra character | Approval bound to a nonexistent artifact | Validate 64 lowercase hex characters and source the digest directly from verifier output before requesting approval |
| A second manually corrected digest was still transposed, while the executor used the correct disk artifact and continued | Technically correct mutation with an invalid authorization chain | Bind approval to one identity tuple; any contradictory identifier hard-stops; require explicit ratification before merge |
| Package metadata lagged behind the repository while design files were current | Whole-snapshot promotion would revert protocol docs | Separate Design-owned payload from repository-managed metadata and diff the full snapshot |
| A malformed closing tag was found after the first DRES | Broken canvas structure | Preserve the old DRES and publish a superseding sequence |
| A request was sent to the wrong actor | Ownership boundary violation | Put an explicit destination label on every instruction and hard-stop wrong-actor work |
| Human approval was remembered in one turn and missing in the next | Gate loss through chat state | Maintain a cumulative decision ledger and stop on inconsistent state |
| Viewport approval was requested where no dedicated render existed | Unsupported "no impact" inference | Classify viewport coverage in DREQ; record missing evidence as unverified; require explicit G3 deferral |
| A component-state strip approximated mobile width but no full mobile screen existed | Component evidence could be mistaken for full-page evidence | Record the evidence type and require the actual composition in G3 |
| The user relayed every intermediate message through an independent reviewer | Manual bottleneck | Keep routine visual review inside Design; make independent audit optional |
| A dirty long-lived worktree existed beside the pilot | Accidental loss of unrelated work | Use a clean sibling worktree from an immutable base and never clean the legacy tree |
| Visual canon changed while three implementation briefs remained byte-identical to the old baseline | Implementers could faithfully recreate removed states and dead actions | Treat prescriptive companion drift as a pre-merge coherence failure; repair at the owning source and re-verify rather than defer to G3 |

### Authorization recovery checkpoint

- The user explicitly voided both incorrectly transcribed zip digests and ratified the exact verified candidate digest, DRES digest, 36-file snapshot, and immutable PR head.
- The executor recorded that correction in the unmerged promotion PR without changing code, documents, commits, or branches.
- This recovered the G2 audit trail but did not authorize merge or G3.
- Three required workflows initially failed before executing any steps because of an external GitHub Actions account gate; no code workaround was attempted. After the account gate was resolved, all required checks passed.

### Visual review checkpoint

- Four screen-level decisions were explicitly approved: explore, detail, guardian sheet, and child completion.
- Mobile 390 was approved from canonical frames, DOM measurements, and a component-state specification; the full Explore restricted composition was deferred to G3.
- Tablet 768 was recorded as unverified rather than "no impact." The user explicitly allowed deferral to G3 because viewport coverage was absent from the DREQ.
- Desktop 1280 was approved only where canon evidence existed. Mobile-only routes were marked not applicable instead of being inferred.
- The pre-existing documentation-only drift was explicitly preserved.
- G3 inherited concrete blocking checks. Visual approval did not execute or imply G2.
- The review phase changed zero files.

### Review-framework extraction

The pilot showed that workflow rules and review judgment should not live in one instruction file. The alpha now separates:

- orchestration and gates,
- evidence and decision quality,
- visual hierarchy and state identity,
- UI interaction,
- UX flow and copy truth,
- accessibility,
- responsive evidence,
- domain ownership boundaries.

The framework publishes criteria and concise rationale, not private hidden reasoning. Its purpose is reproducibility: another reviewer should be able to inspect the same evidence and understand why a finding was made.

A remote API update also produced a duplicated skill file while reporting success. The branch was repaired with an atomic full-tree commit, and complete remote re-read is now required after writes.

### Open questions

- Verifier-generated approval tokens and durable, machine-readable human signatures
- A deterministic verifier CLI for archive safety, manifests, bytes, and SHA-256
- Standard adapters for differing Design tool capabilities
- Reliable viewport capture and visual-diff evidence
- Ownership rules for package metadata in projects that mix authoring and canon documentation
- Migration rules for projects that do not use whole-snapshot promotion

### Alpha exit evidence still required

- Repair the stale implementation briefs through their declared owner, re-verify the candidate, and merge the promotion PR
- Implement the approved design
- Complete live-browser G3
- Publish DVER
- Run a second pilot in the same project
- Run at least one pilot in a different project
