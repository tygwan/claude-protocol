# RFC 001: Frontend Quality Intelligence

- Status: Draft (request for comments)
- Target: `claude-design-sync` plugin, additive to the existing `design-sync` and `design-review` skills
- Base: `codex/alpha4-orchestration` (v0.1.0-alpha.4)
- Author: proposed by the frontend-quality working thread
- Supersedes: none
- Related roadmap items: v0.3 "Define a generic design-intelligence provider adapter contract", v0.3 "Test optional UI UX Pro Max activation precision and conflict handling"

This RFC proposes the structure only. It does not add runtime skill logic, module knowledge files, JSON Schema files, evaluation fixtures, or auto-fixers. Those arrive in later, separately reviewed implementation PRs (see Section 13).

---

## 0. Summary

Claude Design Sync can already orchestrate a human-gated lifecycle and run adaptive design-artifact review. It does not yet carry senior frontend-engineering and UI/UX judgment for the *implementation and runtime* side of a change: whether the built code accessibly, responsively, performantly, and resiliently realizes the accepted design in a real framework and browser, for a user who has no frontend expertise of their own.

This RFC proposes a new sibling skill, `frontend-quality`, that:

1. adds implementation and runtime engineering judgment as a modular, router-loaded reference set,
2. encodes rules as machine-readable records with explicit evidence, exceptions, and verification,
3. reuses the existing evidence ladder, severity, confidence, response-mode, and advisor vocabulary instead of duplicating them,
4. treats external frontend skills (including `ui-ux-pro-max`) as optional, read-only advisors normalized into one finding schema,
5. works with zero external skills installed by falling back to first-party evidence,
6. is evaluated against fixtures that assert both expected findings and expected non-findings, so false positives and over-fixing are measured, not just misses.

The core carries no framework, language, locale, or route assumptions. All project specifics live in a project profile.

---

## 1. Problem statement

The current review surface is strong for design artifacts but has gaps for implementation quality and for non-expert users.

- **Users without frontend expertise still need high-quality decisions.** A product owner who cannot read React or CSS still has to accept or hold a change. Today the plugin can tell them a viewport is `unverified`; it cannot yet explain, in user-impact terms, that a layout will overflow on a phone, that a primary action is a dead end, or that a "success" message is shown before the write actually happened.
- **Verification is viewport, baseline, and per-finding centric.** `design-review` classifies viewports and produces individual findings, but there is no synthesis layer that reasons across accessibility, UX, performance, and maintainability together for one built change. Independent findings can each pass while the composed experience fails.
- **Canvas size is misread as a fixed implementation width.** A canonical design canvas authored at, for example, 1920 CSS pixels is design *intent at one width*, not an instruction to ship a 1920px fixed-width page. Agents and reviewers repeatedly promote a canvas dimension to a hard layout constraint. This produces horizontal scroll, broken reflow, and false "matches canon" claims.
- **Text and internationalization break silently.** Korean and other scripts wrap mid-token differently from Latin text. Narrow text columns, `word-break` choices, and navigation bars that wrap at intermediate widths are frequent real defects that a static canvas at one width and language never reveals.
- **Test-pass is mistaken for user-experience-pass.** A green unit or integration suite proves code paths, not that the user can complete the task, that focus returns correctly from a dialog, or that a control is reachable by keyboard. Agents conflate "tests pass" with "experience verified".
- **Full external dependence breaks portability.** If frontend judgment lives only in an external skill, a project without that skill installed loses the capability entirely. The core must remain useful with no advisor present.
- **A single monolithic SKILL.md explodes cost.** Placing all frontend knowledge in one prompt inflates context on every invocation, couples unrelated rules, and makes maintenance and sourcing unmanageable. Knowledge must be modular and loaded on demand.

Non-goals for this RFC are listed in Section "Scope and non-goals" below.

---

## 2. Design principles

These principles govern every module, rule, and mode. Several already exist in `design-review`; this RFC adopts them by reference rather than restating them as new law.

- **Evidence over taste.** A finding cites observed evidence and its grade. A confident tone does not raise evidence quality. (Reuses `design-review/references/evidence-and-decisions.md`.)
- **User impact before technical detail.** Lead with what the user cannot do or will misunderstand; put the framework mechanism second.
- **Separate standards minimum, project convention, and recommendation.** A WCAG 2.2 AA minimum, a project rule, and an audience recommendation are three different claims and must never be merged. (Reuses the accessibility reference's target-size treatment.)
- **Separate fact, inference, recommendation, decision-required, and unverified.** (Reuses the five-way labeling in `evidence-and-decisions.md`.)
- **Progressive disclosure.** Load and show only what the current decision needs. Compact by default; expand on ambiguity or request.
- **Measure before optimize.** No performance change without a measurement that motivates it and a measurement that confirms it.
- **Normal flow before layout hacks.** Prefer document flow and intrinsic sizing before absolute positioning, negative margins, or magic numbers.
- **Semantic HTML before ARIA.** Native elements first; ARIA only to fill a genuine gap. An ARIA patch over a wrong element is a defect, not a fix.
- **CSS before JavaScript layout.** Prefer CSS layout and container queries before measuring and positioning in script.
- **Clarity before premature abstraction.** Do not extract a component, hook, or token until duplication is real and stable.
- **Actual runtime before baseline update.** A visual baseline is updated only after runtime evidence justifies it, never to make a failing check pass.
- **Missing evidence is not "no impact".** Absent evidence is `unverified`, never a silent pass. (Reuses the responsive reference.)
- **External advisor is advisory, not authority.** Providers rank below project truth and canon and cannot approve gates, mutate canon, or implement code. (Reuses `design-review/references/design-intelligence-providers.md`.)
- **Project instructions and canon take precedence.** The reusable core never overrides repository rules or accepted canon.

---

## 3. Modular taxonomy

`frontend-quality` is organized as independent modules. Each module is a leaf reference loaded only when its triggers fire. The proposed module set:

visual-design, ux-product, accessibility, semantic-html, css-layout, responsive, typography-i18n, react, nextjs, state-data, forms, interaction, routing, performance, security-privacy, resilience, assets-fonts, browser-compatibility, testing-verification, maintainability, design-to-code, observability.

Every module definition MUST specify these fields:

- **responsibility**: the single question the module answers.
- **triggers**: signals that make the module relevant (file types, stack, mode, or finding category).
- **required evidence**: the minimum evidence grade and kind needed before the module asserts anything.
- **MUST / SHOULD / MAY rules**: separated, never mixed in one list.
- **common failure modes**: recurring real defects.
- **counterexamples**: cases where the naive rule is wrong and must not fire.
- **valid exceptions**: conditions under which a MUST is legitimately not applicable.
- **verification strategy**: deterministic checks, runtime checks, and visual checks, kept distinct.
- **auto-fix boundary**: what a future automated fixer may and may not touch (this RFC sets the boundary; it does not build the fixer).
- **decision owner**: who owns a decision the module cannot make alone (for example a product-policy or regulated call).
- **related modules**: modules that share evidence or must be considered together.

### 3.1 Boundary with the existing `design-review` skill

This is the most important structural decision in the RFC. `design-review` already owns several domains. `frontend-quality` must not duplicate them.

| Domain | `design-review` owns | `frontend-quality` owns |
|---|---|---|
| Accessibility | design-time intent in canon and specs (contrast in tokens, target size in spec, semantic intent) | runtime and code reality (computed contrast, focus lifecycle in live DOM, correct native element and ARIA in built component) |
| Responsive | viewport classification and canvas intent | intrinsic sizing, flex/grid min-size, overflow, reflow behavior in real CSS and DOM |
| Visual design | hierarchy, grouping, state identity, token match in the artifact | faithful token application in code, CLS and font-swap behavior, rendered fidelity |
| UX | flow, copy truth, audience, cognitive load in the design | dead CTA in the built route, loading/empty/error states in the running app, double submit, misleading success |
| Interaction | intended state machine and affordance in the design | implemented keyboard, focus return, abort/cancel, race handling |

Rule of ownership:

1. A normative standard (a WCAG success criterion, an HTML content model rule, a CSS specified behavior) is stated **once** in a shared, versioned sources reference and cited by both skills. Neither restates it.
2. Shared vocabulary (evidence ladder E0 to E4, the five-way fact/inference/recommendation/decision-required/unverified labels, severity, confidence, regret modes, response modes) is **owned by `design-review`'s existing references and imported by `frontend-quality`**. It is not redefined.
3. When a domain appears in both skills, `design-review` holds the design-time lens and `frontend-quality` holds the runtime and code lens. A finding names which lens produced it so the two never silently overwrite each other.

`frontend-quality`'s genuinely new territory, with no overlap, is: semantic-html, css-layout, typography-i18n, react, nextjs, state-data, forms, routing, performance, security-privacy, resilience, assets-fonts, browser-compatibility, testing-verification, maintainability, design-to-code, observability.

---

## 4. Rule schema

Rules are machine-readable records, not prose lists, so they can be selected by trigger, versioned, evaluated, and overridden precisely. Proposed fields per rule:

- `id` (stable, namespaced by module)
- `category` (module)
- `level` (`MUST` | `SHOULD` | `MAY`)
- `title`
- `trigger` (when the rule applies)
- `applicable_stacks` (for example: any, react, nextjs, css; never a hardcoded project)
- `principle` (the underlying design principle from Section 2)
- `rationale`
- `user_impact`
- `bad_signals` (observable symptoms that suggest a violation)
- `recommended_strategies`
- `exceptions`
- `counterexamples`
- `evidence` (what proves or disproves it, with required grade)
- `deterministic_checks` (statically or programmatically checkable)
- `visual_checks`
- `regression_requirements`
- `sources` (primary reference IDs; see Section on sources)
- `source_version`
- `last_reviewed`
- `override_policy` (whether and how a project profile may override)

Constraints:

- MUST rules and recommendations are never mixed in one list. A rule carries exactly one `level`.
- `override_policy` defines the exact range a project profile may change. A rule that encodes a legal or safety minimum is marked non-overridable. A rule that encodes a stylistic convention is fully overridable. Everything else states its overridable range explicitly.
- A rule with no `deterministic_checks` and no `visual_checks` cannot produce a blocker; it can only produce a recommendation or a decision-required flag.

---

## 5. Router architecture

There is no single large prompt. `SKILL.md` is a router that reads only the reference modules the current work needs. Proposed layout, additive under the existing plugin:

```
plugins/claude-design-sync/
  skills/
    frontend-quality/
      SKILL.md                      # router only; selects modules by mode and trigger
      references/
        taxonomy.md                 # module index and boundary with design-review
        rule-schema.md              # rule record shape and authoring rules
        evidence-model.md           # imports design-review evidence ladder; adds runtime evidence kinds
        severity.md                 # imports design-review severities; maps to blocking gates
        modules/
          visual-design.md
          ux-product.md
          accessibility.md
          semantic-html.md
          css-layout.md
          responsive.md
          typography-i18n.md
          react.md
          nextjs.md
          state-data.md
          forms.md
          interaction.md
          routing.md
          performance.md
          security-privacy.md
          resilience.md
          assets-fonts.md
          browser-compatibility.md
          testing-verification.md
          maintainability.md
          design-to-code.md
          observability.md
      schemas/
        frontend-profile.schema.json
        finding.schema.json
        rule.schema.json
      evals/
        cases/
        expected/
```

Router rules:

- `SKILL.md` never inlines module knowledge. It resolves the mode and triggers, then loads the minimum set of module files.
- `evidence-model.md` and `severity.md` explicitly import from `design-review` rather than restating the ladder and severities.
- Schemas and evals live outside `references/` so review and runtime never load them as prose.

This mirrors and extends the selective-loading pattern already used by `design-review`, scaled to a larger module set.

---

## 6. Operating modes

Each mode declares which modules it loads and what it produces. Modes align to the existing lifecycle and gates.

| Mode | Loads (typical) | Produces | Ties to gate |
|---|---|---|---|
| `build` | semantic-html, css-layout, responsive, react/nextjs, forms, interaction | implementation guidance and inline rule checks during authoring | none (pre-gate authoring) |
| `review` | mode-relevant modules by trigger | findings normalized to `finding.schema.json`, compact brief plus audit detail | feeds `implementation_acceptance` |
| `audit` | evidence-model, severity, all fired modules | reproducible evidence bundle, no new taste claims | audit plane on request or ambiguity |
| `responsive` | responsive, css-layout, typography-i18n | reflow and overflow findings across profile viewports | `implementation_acceptance` |
| `accessibility` | accessibility, semantic-html, interaction | a11y findings separated into standards minimum, project, recommendation | `implementation_acceptance` |
| `performance` | performance, assets-fonts, react | measured findings with before and after evidence and a budget check | `implementation_acceptance` |
| `design-to-code` | design-to-code, visual-design, css-layout, typography-i18n | fidelity findings that respect canvas-intent-not-fixed-width rule | `design_acceptance` support, `implementation_acceptance` |
| `incident` | resilience, state-data, observability, routing | regression-focused triage findings and a reproduction | recovery and correction events |
| `implementation-acceptance` | testing-verification plus prior mode outputs | a synthesis that separates test-pass from experience-pass | `implementation_acceptance` |

A mode never claims a gate. It produces evidence that a human gate consumes.

---

## 7. Senior engineering heuristics

The RFC records the specific heuristics each engineering module must encode. These are the substance of "senior" judgment. They are listed here so reviewers can debate coverage before any module file is written.

### React

component responsibility; server and client boundary; derived state instead of duplicated state; effect misuse (effects used for derivation or event logic); stable keys; controlled versus uncontrolled inputs; concurrency and race handling; abort and cancellation on unmount or supersession; error and loading boundaries; memoization only after measurement; context overuse; render stability; stale closures; hydration correctness.

### CSS

normal flow first; intrinsic sizing; flex and grid min-size behavior; the `min-width: 0` overflow fix; risk of fixed height and width; `max-width` with fluid sizing; cascade, layers, and specificity; token use over literals; logical properties for internationalization; container versus breakpoint selection; overflow ownership; sticky, fixed, and z-index stacking; viewport units and safe-area insets; typography and `word-break` for scripts including Korean; reduced-motion support; print and high-contrast rendering.

### UX

one primary action per decision point; information architecture; progressive disclosure; feedback and system status; error prevention and recovery; loading, empty, and error states; destructive-action confirmation; undo; no-op and dead CTA detection; misleading success detection; role and audience specific copy; cognitive load; consistency; a completable task path.

### Optimization

measure before optimizing; user-perceived performance over synthetic numbers; Core Web Vitals evidence; bundle, request, and render cost; image and font strategy; cache and data freshness; avoiding premature memoization; avoiding blanket lazy loading; preventing cumulative layout shift; a performance budget carried in the project profile.

Each heuristic becomes one or more rule records with the fields from Section 4, including counterexamples so the rule does not overfire.

---

## 8. External advisor adapter

External frontend and UI/UX skills, including `ui-ux-pro-max`, are optional advisors. This section extends the existing `design-intelligence-providers.md` contract to frontend advisors rather than inventing a parallel one.

### 8.1 Two provider kinds: advisory versus evidence

Research (Appendix A) shows two distinct provider roles, and conflating them would violate "evidence over taste":

- **Advisory (prescriptive) providers** recommend designs, palettes, patterns, and checklists. `ui-ux-pro-max` is one: it is a compact, offline, self-contained recommendation database. It suggests; it does not observe a running application. Advisory output is opinion or convention until confirmed by evidence. It may raise a question or an alternative, but on its own it can only produce a recommendation or a hypothesis, never a blocker.
- **Evidence (observational) providers** measure a real artifact. Playwright supplies live render, an accessibility tree, computed layout, console, and network for the running app. Figma supplies authored design intent and tokens. context7 supplies versioned normative docs. Only measured evidence from an observational provider (or a first-party measurement) can support a blocking finding.

The adapter treats these kinds differently: an advisory provider's suggestion enters as a `hypothesis` or `recommendation` and must be reconciled against evidence; an evidence provider's measurement enters as a graded `FACT` at the grade its method supports.

### 8.2 Scoping a real advisor: `ui-ux-pro-max`

Because this advisor was verified in detail (Appendix A), the RFC records how a real project should scope it, without vendoring or hard-depending on it:

- **Use only the offline advisory core.** Its search core is Python standard library only, needs no network or API key, and can emit JSON, which is the clean adapter surface. Its generative sub-skills (logo, banner, icon, social image) call an external image model and are online and not read-only; the read-only advisory adapter MUST exclude them.
- **Pin an explicit version.** An installed marketplace copy can lag upstream (observed: an in-use copy behind the latest release). The adapter pins a version per request and never assumes installed equals latest.
- **Respect its license before any reuse.** It is MIT per its manifest metadata, which permits reuse and vendoring with attribution, but the exact notice wording must be confirmed from the LICENSE file before any copy is made. The core does not vendor it.

### 8.3 The adapter contract

- **capability discovery**: detect whether an advisor is installed and what it can answer, without assuming a specific one.
- **provider selection**: choose an advisor only when expected information gain exceeds context and conflict cost, and only for its declared capability.
- **version and provenance**: pin the advisor version for the duration of one request and record provider ID, version, and query.
- **input contract**: send one scoped question plus the relevant project constraints, never the whole workspace.
- **output normalization**: convert advisor output into `finding.schema.json` records; unnormalizable output is recorded as `unverified` context, not a finding.
- **confidence**: attach a confidence and evidence grade to advisor-derived findings; advisor output defaults to no higher than a component-level grade unless it carries reproducible runtime evidence.
- **conflict resolution**: resolve conflicts by the existing authority order (law and safety, explicit user decisions, project instructions and product truth, accepted canon, measured implementation evidence, provider guidance, generic convention). An advisor never wins over canon.
- **unavailable fallback**: if no advisor is installed, continue with first-party evidence and mark only material advisor-dependent questions as unverified.
- **no direct authority**: advisors cannot approve gates, mutate canon, persist a design system, implement code, or expand scope.
- **no automatic canon mutation**: advisor suggestions enter the normal design-change path, never a silent edit.
- **no hidden chain-of-thought requirement**: the adapter consumes evidence, alternatives, and rationale, not a private token-by-token trace.

Every advisor-derived finding is labeled with provider ID, pinned version, and evidence grade, and is reconciled against project authority before it can influence a decision. The core does not vendor any advisor repository and does not make any advisor a hard dependency. License, version, and availability of any advisor must be verified before that advisor is referenced in a real project profile; unverified advisor content is never copied into the core.

---

## 9. Finding schema

One finding shape carries both first-party and normalized advisor findings. Proposed fields:

- `id` (stable)
- `category` (module)
- `severity` (`blocker` | `major` | `moderate` | `minor`, reusing `design-review`)
- `confidence` (`high` | `medium` | `low`, reusing `design-review`)
- `affected_user` (role or audience)
- `affected_context` (route, state, viewport, browser, locale, role)
- `surface` (`deployed` | `candidate` | `both`)
- `observation`
- `evidence` (with grade E0 to E4, reusing the ladder)
- `reference` (standard, project rule, or canon, kept distinct)
- `user_impact`
- `reproduction`
- `suggested_resolution`
- `alternative`
- `tradeoff`
- `counterexample`
- `owner` (decision owner when not a pure implementation call)
- `disposition` (`open` | `accepted` | `deferred` | `rejected`)
- `regression_test` (what test would catch a recurrence)
- `blocking_gate` (which semantic gate it blocks, if any)

There is no fabricated aggregate score. A finding does not roll up into a 0 to 100 number that hides which user, in which context, cannot do what.

---

## 10. Project profile

All project specifics live in a `frontend-profile.schema.json` instance, never in the core. Proposed fields:

- framework and runtime
- styling system
- design source
- tokens and components
- supported browsers
- locales
- viewports
- accessibility target
- performance budgets
- user roles
- sensitive surfaces
- regulated boundaries
- required tests
- optional advisors
- override rules (which rule IDs a project may relax, within each rule's `override_policy`)

The core MUST NOT hardcode Next.js, Tailwind, a specific locale such as Korean, or a specific route. Those appear only as values in a profile or as clearly labeled examples. Korean line-break handling, for instance, is a `typography-i18n` capability that activates when a profile declares a relevant locale, not a core assumption.

---

## 11. Documentation budget

- Not every check is stored as Markdown. Routine successful checks are transient or CI artifacts, not durable records.
- Durable decisions and machine evidence are separated. Persist findings, dispositions, and reproduction evidence; do not persist full reviewer monologues.
- Each rule and reference has a single owner. Shared standards and shared vocabulary are cited, never copied. This is the mechanism that prevents `frontend-quality` from duplicating `design-review`.
- Every rule and source carries `source_version` and `last_reviewed`. A stale source is visible, not silently trusted.
- A compact owner brief and the audit artifact are separate outputs. The brief is short; the audit is reproducible.
- Discipline references load only when their module fires.
- Deprecated rules are marked deprecated with a replacement pointer and a removal target, never deleted silently while findings still cite them.

---

## 12. Evaluation strategy

Evaluation asserts both what MUST be found and what MUST NOT be flagged. Over-flagging and over-fixing are failures, not neutral. Minimum fixture set, each with an expected finding or an expected non-finding:

| Fixture | Expectation |
|---|---|
| 1920 fixed-width page | finding: fixed width misread from canvas; horizontal scroll on smaller viewports |
| Korean mid-word wrapping | finding: `word-break` and column width break comprehension |
| flex min-width overflow | finding: missing `min-width: 0` causes overflow |
| grid overflow | finding: track sizing overflows container |
| 200% zoom failure | finding: content loss or overlap at 200% |
| sub-24px target | finding: target below WCAG 2.2 AA minimum (as standards claim, not project) |
| modal focus loss | finding: focus not trapped or not returned |
| dead CTA | finding: action leads nowhere valid |
| fake success | finding: success shown before the effect occurred |
| hydration mismatch | finding: server and client markup diverge |
| unauthorized content flash | finding: protected content renders before auth resolves |
| stale async response | finding: superseded response overwrites current state |
| double submit | finding: duplicate submission not prevented |
| missing loading or error state | finding: state gap in the running app |
| CLS from font or image | finding: layout shift from unsized asset or font swap |
| unnecessary React rerender | finding: avoidable rerender, with measurement |
| missing deep-link handling | finding: route state not restorable from URL |
| broken browser history | finding: back and forward behave incorrectly |
| expected `max-width` that must not be falsely reported | non-finding: correct fluid `max-width` must not be flagged as fixed width |
| valid accessibility exception | non-finding: a legitimately exempt control must not be flagged |

Each case declares expected findings, expected non-findings, and the evidence needed to reach the verdict. False positives and over-fixes are scored explicitly.

---

## 13. PR decomposition

After this RFC is accepted, implementation is split so each PR is independently reviewable and none is a monolith:

- **PR-A**: router `SKILL.md`, `rule.schema.json`, `finding.schema.json`, `frontend-profile.schema.json`, taxonomy and evidence-model references that import from `design-review`.
- **PR-B**: HTML, CSS, responsive, typography-i18n modules.
- **PR-C**: React, Next.js, state-data, forms modules.
- **PR-D**: UX, accessibility, design-to-code modules.
- **PR-E**: performance, security-privacy, resilience, testing-verification modules.
- **PR-F**: optional advisor adapters and output normalization.
- **PR-G**: deterministic verifier and eval harness (cases and expected results).
- **PR-H**: pilot integration and documentation.

Ordering rationale: schemas and the boundary contract first (PR-A), then modules with the strongest deterministic and runtime evidence (PR-B, PR-C), then judgment-heavy modules (PR-D), then cross-cutting modules (PR-E), then advisors (PR-F) once the finding schema is stable, then the verifier and evals (PR-G), then a pilot (PR-H).

---

## 14. Acceptance criteria

- A user with no frontend or UI/UX expertise receives guidance framed by real user impact.
- The core operates with no external skill installed.
- When an external advisor is present, its evidence and provenance are preserved and reconciled against authority.
- References unrelated to the current task are not loaded.
- Every rule carries at least one exception and a verification method.
- A canonical canvas is never treated as a fixed page width.
- A visual baseline never substitutes for user approval or runtime evidence.
- React and CSS optimizations are never forced without measurement.
- Accessibility, UX, performance, and maintainability are considered together, not in isolation.
- No project-specific policy enters the reusable core.
- Human-facing reports are compact while audit evidence is reproducible.

---

## 15. Open questions

Reviewers are asked to weigh in on:

1. Which rules belong in the core as non-overridable MUSTs, and which are always project-overridable?
2. What is the update policy for WCAG, React, and browser-support source versions, and who owns `last_reviewed`?
3. What confidence ceiling should advisor-derived findings carry by default?
4. Where exactly is the boundary between a framework adapter and the core, especially for Next.js specifics?
5. What is the allowed range of future automated fixes, per module `auto_fix_boundary`?
6. What is the minimum eval fixture bar required to merge each module PR?
7. What is the documentation budget ceiling for durable frontend findings per work item?
8. Which rules must never be project-overridable because they encode a legal, safety, or accessibility-standard minimum?
9. Where is the line between subjective visual taste and an objective defect, so taste is offered as opinion and defects as findings?
10. Which modules ship in v1 and which are deferred? A proposed v1 set: semantic-html, css-layout, responsive, accessibility, typography-i18n, react, and testing-verification, because they have the strongest deterministic and runtime evidence and the highest non-expert user impact. Deferred candidates: observability, security-privacy depth, and design-to-code automation.

---

## Scope and non-goals

**In scope for this RFC**: the structure above (taxonomy, boundary, rule schema, router layout, modes, finding schema, project profile, documentation budget, evaluation strategy, PR decomposition, acceptance criteria, open questions).

**Non-goals for this RFC**:

- no runtime skill logic or `SKILL.md` router implementation,
- no module knowledge files,
- no JSON Schema files,
- no evaluation fixtures or harness,
- no auto-fixer,
- no copying of any external skill's content,
- no project-specific policy in the core,
- no changes to `design-sync` or `design-review` behavior,
- no merge.

## Not implemented in this PR

This PR adds only this RFC document and a docs index entry. Everything in Sections 3 through 13 is a proposal to be built in the PR-A through PR-H sequence after acceptance.

## Review request

Codex and Claude reviewers are asked to evaluate:

- the `design-review` versus `frontend-quality` ownership boundary in Section 3.1,
- whether the rule and finding schemas reuse existing vocabulary correctly instead of duplicating it,
- the evaluation strategy's treatment of expected non-findings and over-fixing,
- the external advisor adapter's authority limits,
- the v1 module set proposed in Open Question 10.

---

## Appendix A: Advisor and evidence-provider landscape (research)

Capability-level research used to ground Section 8. No external skill content was copied. Items marked "verify" are not settled and must be confirmed before a project relies on them.

**Advisory (prescriptive) provider.** `nextlevelbuilder/ui-ux-pro-max-skill`: a compact, self-contained UI/UX recommendation database (a small bundled CSV and JSON corpus with a thin Python query layer). It recommends palettes, font pairings, per-stack patterns, anti-patterns, and pre-delivery checklists from a natural-language request. It exposes a deterministic query CLI that can emit JSON, which is the clean adapter surface. Its advisory core is offline and needs no network or API key. Its generative image sub-skills are online and not read-only and are out of scope for a read-only adapter. License is MIT per manifest metadata (permits reuse and vendoring with attribution; verify the exact LICENSE text before any copy). Versioning is formal SemVer with frequent releases; an installed marketplace copy can lag upstream, so pin a version and do not assume installed equals latest (verify the installed version at use time).

**Evidence (observational) providers** available as MCP servers in typical environments, characterized at capability level:

| Provider | Can supply as frontend evidence | Cannot supply |
|---|---|---|
| Playwright | live render of the running app, accessibility tree snapshot, computed layout and styles, console errors, network requests, responsive behavior by resize, lab performance metrics only if a measurement library is injected | field performance data; design intent; it is not natively an axe or Lighthouse engine, so WCAG scoring and Core Web Vitals require injected libraries; needs a served URL |
| Figma | authored design intent: tokens and variables, component metadata, a design screenshot, design-system membership, design-to-code mappings | anything about the running app; needs a Figma file and authenticated session |
| Stitch | generated design systems and screens from text | any observed evidence about an existing app; it authors designs, it does not measure them |
| context7 | current, version-aware official framework and library docs (the normative reference) | anything about the specific app; needs network |

A first-party skill for Core Web Vitals and accessibility gaps via a browser devtools path may also be present, but its required devtools server is not always connected; treat it as conditional (verify).

**Offline fallback (no provider installed).** Realistic and bounded. Always available: reading and static analysis of source (semantic HTML versus generic containers, presence of `alt`, `label`, and `aria`, heading order, token versus literal values) and deterministic contrast math computed directly from CSS color values, which is a genuine standards-based accessibility check done purely in code. Conditional: local linters and auditors such as an accessibility lint plugin, a style linter, a type check, or a CLI accessibility auditor, if installed. Available here: a local headless browser through Playwright, which yields real render, accessibility tree, computed layout, console, network, and lab performance metrics if the app builds and serves. Fundamentally unavailable offline: field performance data, and design-conformance without the design file. Net: offline gives strong static and lab evidence plus deterministic accessibility math, but not field truth or design conformance.

## Appendix B: Primary sources registry (research)

Grounds the per-rule `sources`, `source_version`, and `last_reviewed` fields (Section 4) and the single-ownership rule (Section 11). Status values are as of mid-2026 and must be re-reviewed on the cadence the profile sets.

| Domain | Canonical primary source | Versioning and citation |
|---|---|---|
| HTML | WHATWG HTML Living Standard | living standard, no version number; cite by section anchor plus a commit or permalink and the last-updated date |
| CSS | W3C CSS Working Group modular specs, with MDN and browser-compat data as companions | each spec carries a maturity status (Working Draft, Candidate Recommendation, Recommendation); cite the module and status |
| React | react.dev official docs | versioned to the React release line; track by release and a doc last-reviewed date |
| Accessibility | W3C WCAG 2.2 (current Recommendation), with WAI-ARIA and the ARIA Authoring Practices Guide | cite by success-criterion number plus WCAG version; WCAG 3.0 is a Working Draft only and must not be cited as normative |
| Web performance | Google web.dev and the web-vitals definitions | Core Web Vitals evaluated at the 75th percentile of field data; interaction metric replaced the older input-delay metric in 2024; field data and lab tooling are separate sources |

Management model: treat references as docs-as-code. Keep one machine-readable sources registry with an entry per source carrying organization, document, canonical URL or permalink, version or spec status, retrieved date, last-reviewed date, next-review-due date, and applicable scope. Pin living standards to a commit or permalink rather than a moving latest pointer. Run scheduled reviews and link checks. This registry is the single owner of normative claims that both `design-review` and `frontend-quality` cite, and it can be vendored offline as the fallback normative layer.
