# Domain and ownership boundaries

Design review must not manufacture product truth.

## Classify every consequential claim

- visual decision,
- interaction decision,
- product policy,
- content policy,
- clinical decision,
- legal or compliance decision,
- safety decision,
- backend or API behavior,
- persistence or data-retention behavior,
- game or scoring behavior.

Name the owner for every nonvisual decision.

## Flag instead of guessing

Use a visible unresolved flag when:

- eligibility criteria are unknown,
- a clinical interpretation is disputed,
- an API does not expose the reason the UI wants to show,
- timing or automatic unlock is not guaranteed,
- content availability is undecided,
- the destination route does not exist,
- data saving or deletion behavior is unclear,
- two systems apply overlapping gates.

A visually elegant invented rule is still wrong.

## Interface contract

For each state, distinguish:

- what the server guarantees,
- what the client derives,
- what Design communicates,
- what the user can do,
- what remains unresolved.

When layers disagree, do not patch the copy to hide the mismatch. Escalate the contract decision.

## Audience boundary

Route information to the role that can understand or act on it. Technical codes, clinical detail, account administration, and caregiver guidance may require different surfaces.

## Review output

For each flagged item, provide:

- exact unresolved question,
- current observed behavior,
- risk of guessing,
- named decision owner,
- safe temporary presentation,
- downstream artifacts to update after the decision.
