# RFC 001: Frontend Quality Intelligence

- Status: Draft (request for comments)
- Revision: 0.2 (addresses the first-round review; still document-only, no implementation, no merge)
- Target: `claude-design-sync` plugin, additive to the existing `design-sync` and `design-review` skills
- Base: `codex/alpha4-orchestration` (v0.1.0-alpha.4)
- Author: proposed by the frontend-quality working thread
- Supersedes: none
- Related roadmap items: v0.3 "Define a generic design-intelligence provider adapter contract", v0.3 "Test optional UI UX Pro Max activation precision and conflict handling"

This RFC proposes structure only. It does not add runtime skill logic, module knowledge files, JSON Schema files, evaluation fixtures, or auto-fixers. Those arrive in later, separately reviewed implementation PRs (see Section 13).

### Revision 0.2 change summary

This revision answers a review that accepted the direction but held Accepted and implementation until structure was fixed. Changes:

1. Shared contracts move to a **neutral `shared-contracts` layer** that both skills import, removing the sibling-to-sibling dependency (Sections 3.2, 5, 11).
2. Advisory output is **claim-typed, not evidence-graded**; the existing `E2` advisor example is corrected (Sections 2, 8, 9, Appendix C).
3. v1 gains **user-task-completion coverage**; React and Next.js become **profile-triggered framework adapters**, not core (Sections 3, 10, 15).
4. Rules get a **single canonical format** (data is authoritative; human docs are generated or reference it) (Sections 4, 5, 11).
5. A **minimal eval runner moves to PR-A**, and every module PR ships its own fixtures (Sections 12, 13).
6. A **two-stage activation** contract is defined (Section 5.1).
7. A **design-sync integration and update contract** is defined (Section 6.1).
8. A standard's **technical fact is separated from its applicability** (Sections 2, 4, 10).

Required consistency fixes to existing alpha.4 files are listed in Appendix C and are not applied in this document-only PR.

---

## 0. Summary

Claude Design Sync can already orchestrate a human-gated lifecycle and run adaptive design-artifact review. It does not yet carry senior frontend-engineering and UI/UX judgment for the *implementation and runtime* side of a change: whether the built code accessibly, responsively, performantly, and resiliently realizes the accepted design in a real framework and browser, for a user who has no frontend expertise of their own.

This RFC proposes a new sibling skill, `frontend-quality`, that:

1. adds implementation and runtime engineering judgment as a modular, router-loaded reference set,
2. encodes rules as machine-readable records with a single canonical format, explicit evidence, exceptions, and verification,
3. imports evidence, claim types, severity, authority order, response modes, and the source registry from a neutral `shared-contracts` layer shared with `design-review`, rather than depending on a sibling skill,
4. treats external frontend skills (including `ui-ux-pro-max`) as optional, read-only advisors whose output is claim-typed as a recommendation or hypothesis, never an evidence grade on its own,
5. works with zero external skills installed by falling back to first-party evidence,
6. is evaluated from PR-A onward against fixtures that assert expected findings, expected non-findings, and router non-activation, so misses, false positives, and over-fixing are all measured.

The core carries no framework, language, locale, or route assumptions, and states standard facts without deciding where they apply. All project specifics and applicability decisions live in a project profile.

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

These principles govern every module, rule, and mode. Shared ones are defined once in the neutral `shared-contracts` layer (Section 3.2) and imported by both skills, not restated as new law.

- **Evidence over taste.** A finding cites observed evidence and its grade. A confident tone does not raise evidence quality. (From `shared-contracts/evidence` and `shared-contracts/claim-types`.)
- **User impact before technical detail.** Lead with what the user cannot do or will misunderstand; put the framework mechanism second.
- **Separate a standard's technical fact from its applicability.** The core may state that a criterion exists and what it measures. Whether that criterion binds this project is an applicability decision owned by the profile or a `regulated_decision`. The core never declares a general legal, safety, or conformance obligation on a project's behalf.
- **Separate standards minimum, project convention, and recommendation.** A conformance minimum, a project rule, and an audience recommendation are three different claims and must never be merged.
- **Separate fact, inference, recommendation, hypothesis, decision-required, and unverified.** (From `shared-contracts/claim-types`.)
- **Progressive disclosure.** Load and show only what the current decision needs. Compact by default; expand on ambiguity or request.
- **Measure before optimize.** No performance change without a measurement that motivates it and a measurement that confirms it.
- **Normal flow before layout hacks.** Prefer document flow and intrinsic sizing before absolute positioning, negative margins, or magic numbers.
- **Semantic HTML before ARIA.** Native elements first; ARIA only to fill a genuine gap. An ARIA patch over a wrong element is a defect, not a fix.
- **CSS before JavaScript layout.** Prefer CSS layout and container queries before measuring and positioning in script.
- **Clarity before premature abstraction.** Do not extract a component, hook, or token until duplication is real and stable.
- **Actual runtime before baseline update.** A visual baseline is updated only after runtime evidence justifies it, never to make a failing check pass.
- **Missing evidence is not "no impact".** Absent evidence is `unverified`, never a silent pass.
- **External advisor is advisory, not authority.** Providers rank below project truth and canon and cannot approve gates, mutate canon, or implement code. Advisory output enters as a recommendation or hypothesis and carries no evidence grade unless it attaches a separate, reproducible observation. (From `shared-contracts/authority`.)
- **Project instructions and canon take precedence.** The reusable core never overrides repository rules or accepted canon.

---

## 3. Modular taxonomy

`frontend-quality` is organized as independent modules. Each module is a leaf reference loaded only when its triggers fire.

Modules fall into three groups:

- **Core modules** (stack-neutral): visual-design, ux-product, accessibility, semantic-html, css-layout, responsive, typography-i18n, interaction, performance, security-privacy, resilience, assets-fonts, browser-compatibility, testing-verification, maintainability, design-to-code, observability.
- **Composite modules**: `task-integrity` composes forms, state-data, and routing to answer one question, "can the user complete and recover the task," because a dead CTA, a false success, a double submit, a lost deep link, or broken history is rarely visible from any single one of those modules alone.
- **Framework-adapter modules** (activated only when a profile declares the framework): react, nextjs. These hold framework-specific judgment and are never loaded for a project that does not use them.

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

`design-review` already owns several domains. `frontend-quality` must not duplicate them.

| Domain | `design-review` owns | `frontend-quality` owns |
|---|---|---|
| Accessibility | design-time intent in canon and specs (contrast in tokens, target size in spec, semantic intent) | runtime and code reality (computed contrast, focus lifecycle in live DOM, correct native element and ARIA in built component) |
| Responsive | viewport classification and canvas intent | intrinsic sizing, flex/grid min-size, overflow, reflow behavior in real CSS and DOM |
| Visual design | hierarchy, grouping, state identity, token match in the artifact | faithful token application in code, CLS and font-swap behavior, rendered fidelity |
| UX | flow, copy truth, audience, cognitive load in the design | dead CTA in the built route, loading/empty/error states in the running app, double submit, misleading success |
| Interaction | intended state machine and affordance in the design | implemented keyboard, focus return, abort/cancel, race handling |

Rule of ownership:

1. A normative standard (a WCAG success criterion, an HTML content model rule, a CSS specified behavior) is stated **once** in `shared-contracts/source-registry` and cited by both skills. Neither restates it.
2. Shared vocabulary (evidence ladder, claim types, severity, authority order, regret modes, response modes, the finding core) lives in the neutral `shared-contracts` layer and is imported by both skills. Neither skill owns it, so neither depends on the other.
3. When a domain appears in both skills, `design-review` holds the design-time lens and `frontend-quality` holds the runtime and code lens. A finding names which lens produced it so the two never silently overwrite each other.

`frontend-quality`'s genuinely new territory, with no `design-review` overlap, is: semantic-html, css-layout, typography-i18n, react, nextjs, state-data, forms, routing, performance, security-privacy, resilience, assets-fonts, browser-compatibility, testing-verification, maintainability, observability, and the `task-integrity` composite.

### 3.2 Shared contracts (neutral layer)

The first-round review correctly noted that importing contracts from `design-review` into `frontend-quality` creates a reverse dependency between sibling skills. The fix is a neutral layer that neither skill owns:

```
shared-contracts/
  evidence           # the E0 to E4 evidence ladder and runtime evidence kinds
  claim-types        # fact, inference, recommendation, hypothesis, decision-required, unverified
  severity           # blocker, major, moderate, minor, and how each maps to a gate
  authority          # the conflict-resolution authority order
  source-registry    # canonical standards with version and last-reviewed metadata
  finding-core       # the shared finding fields both skills specialize
```

Both `design-review` and `frontend-quality` import these. Promoting the current `design-review` references into this layer is an alpha.4 consistency change, listed in Appendix C, to be done in PR-A or the alpha.4 branch, not in this document-only PR.

---

## 4. Rule schema

Rules are machine-readable records with a **single canonical format**, so they can be selected by trigger, versioned, evaluated, and overridden precisely, and never drift between two copies.

Canonical-format decision:

- The authoritative rule record is **structured data (YAML) validated against `rule.schema.json`**. This is the single source of truth.
- Human-readable module documents (`modules/*.md`) are **generated from or reference the rule data by ID**. They never re-author a rule's normative text. A module doc that restates a rule is a lint failure.
- The eval harness reads the same rule data, so a rule, its documentation, and its tests cannot diverge.

Fields per rule:

- `id` (stable, namespaced by module)
- `category` (module)
- `level` (`MUST` | `SHOULD` | `MAY`)
- `title`
- `trigger` (when the rule applies)
- `applicable_stacks` (for example: any, react, nextjs, css; never a hardcoded project)
- `standard_fact` (the technical criterion the rule reflects, stated neutrally)
- `applicability` (who decides whether this binds: `always`, `profile`, or `regulated_decision`)
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
- `sources` (source-registry IDs)
- `source_version`
- `last_reviewed`
- `override_policy` (whether and how a project profile may override)

Constraints:

- MUST rules and recommendations are never mixed in one list. A rule carries exactly one `level`.
- `standard_fact` and `applicability` are separate. The core may assert the fact; only the profile or a `regulated_decision` decides where it binds. A rule whose `applicability` is `regulated_decision` cannot block a gate until an owner has decided it applies.
- `override_policy` defines the exact range a project profile may change. A rule whose applicability a project cannot lower (a safety or legal minimum the profile has adopted) is marked non-overridable within that profile. A stylistic convention is fully overridable.
- A rule with no `deterministic_checks` and no `visual_checks` cannot produce a blocker; it can only produce a recommendation or a decision-required flag.

---

## 5. Router architecture

There is no single large prompt. `SKILL.md` is a router that reads only the modules the current work needs. Proposed layout, additive under the existing plugin, with the neutral layer as a sibling of the skills:

```
plugins/claude-design-sync/
  shared-contracts/                 # neutral; imported by design-review and frontend-quality
    evidence.md
    claim-types.md
    severity.md
    authority.md
    source-registry.md
    finding-core.md
  skills/
    frontend-quality/
      SKILL.md                      # router only; two-stage activation; loads minimum modules
      references/
        taxonomy.md                 # module index and the design-review boundary
        rule-authoring.md           # how to author rule data; docs are generated or reference it
        activation.md               # two-stage activation and budgets (Section 5.1)
        modules/                    # narrative views; cite rule IDs, never re-author rules
          core/                     # stack-neutral modules
          composites/task-integrity.md
          adapters/react.md
          adapters/nextjs.md
      rules/                        # CANONICAL rule data (YAML), validated by rule.schema.json
        core/
        composites/
        adapters/
      schemas/
        rule.schema.json
        finding.schema.json
        frontend-profile.schema.json
      evals/
        runner/                     # minimal eval runner (ships in PR-A)
        cases/                      # positive, negative, and router-non-activation fixtures
        expected/
```

Router rules:

- `SKILL.md` never inlines module knowledge. It runs two-stage activation (Section 5.1), then loads the minimum set of module files.
- Contract references resolve to `shared-contracts`, not to a sibling skill.
- Rule data under `rules/` is canonical; `modules/*.md` are views; schemas and evals never load as prose during review.

### 5.1 Two-stage activation

Activation is cheap first, expensive only when justified:

- **Stage 1, signal detection (low cost).** Read the current lifecycle step, the changed file set, and coarse risk signals (file types, whether a route or form changed, whether tests exist). No module knowledge is loaded yet.
- **Stage 2, module selection.** Load only modules whose triggers fire, ordered by user impact, up to a **module budget** (a small, profile-configurable cap). If more modules would fire than the budget allows, the router narrows to the highest-impact set for the current gate and records what it deferred, rather than loading everything.
- **Advisor call (optional, last).** Call an external advisor only when residual uncertainty is high and expected information gain exceeds context and conflict cost. **Skip conditions**: implementing exact accepted canon, a mechanical or token-only change, backend or API-only work, a regulated policy decision, or when the advisor would only repeat evidence already in hand.
- **Context budget.** The router tracks how much reference text it has loaded and stays within a profile-set budget. Exceeding it forces narrowing or splitting into separate modes, never silent truncation of evidence.

This mirrors the intended orchestration flow in Section 6.1.

---

## 6. Operating modes

Each mode declares which modules it loads and what it produces. Framework-adapter modules load only when the profile declares that framework.

| Mode | Loads (typical) | Produces | Ties to gate |
|---|---|---|---|
| `build` | semantic-html, css-layout, responsive, forms, interaction, plus framework adapter if declared | implementation guidance and inline rule checks during authoring | none (pre-gate authoring) |
| `review` | mode-relevant modules by trigger | findings on the shared finding core, compact brief plus audit detail | feeds `implementation_acceptance` |
| `audit` | evidence, severity, all fired modules | reproducible evidence bundle, no new taste claims | audit plane on request or ambiguity |
| `responsive` | responsive, css-layout, typography-i18n | reflow and overflow findings across profile viewports | `implementation_acceptance` |
| `accessibility` | accessibility, semantic-html, interaction | a11y findings separated into standards fact, project applicability, recommendation | `implementation_acceptance` |
| `performance` | performance, assets-fonts, framework adapter if declared | measured findings with before and after evidence and a budget check | `implementation_acceptance` |
| `design-to-code` | design-to-code, visual-design, css-layout, typography-i18n | fidelity findings that respect canvas-intent-not-fixed-width | `design_acceptance` support, `implementation_acceptance` |
| `incident` | resilience, state-data, observability, routing, task-integrity | regression-focused triage findings and a reproduction | recovery and correction events |
| `implementation-acceptance` | testing-verification, task-integrity, plus prior mode outputs | a synthesis that separates test-pass from experience-pass | `implementation_acceptance` |

A mode never claims a gate. It produces evidence that a human gate consumes.

### 6.1 Integration with design-sync and the update contract

`frontend-quality` is invoked by orchestration, not on its own:

```
design-sync
  -> read current step, changed files, and risk signals
  -> select only the needed frontend-quality modules (Section 5.1)
  -> call an external advisor only when uncertainty is high
  -> hand implementation-verification evidence to implementation_acceptance
```

Update contract: `design-sync update` reports, in addition to the plugin itself, the install status, current version, and compatibility of any optional advisor named in the profile. It does **not** auto-update an advisor during an active request. An advisor version change takes effect only at a durable checkpoint, never mid-gate, matching the existing update discipline for the plugin.

---

## 7. Senior engineering heuristics

The RFC records the specific heuristics each engineering module must encode. These are the substance of "senior" judgment, listed so reviewers can debate coverage before any module file is written.

### React (framework adapter)

component responsibility; server and client boundary; derived state instead of duplicated state; effect misuse (effects used for derivation or event logic); stable keys; controlled versus uncontrolled inputs; concurrency and race handling; abort and cancellation on unmount or supersession; error and loading boundaries; memoization only after measurement; context overuse; render stability; stale closures; hydration correctness.

### CSS

normal flow first; intrinsic sizing; flex and grid min-size behavior; the `min-width: 0` overflow fix; risk of fixed height and width; `max-width` with fluid sizing; cascade, layers, and specificity; token use over literals; logical properties for internationalization; container versus breakpoint selection; overflow ownership; sticky, fixed, and z-index stacking; viewport units and safe-area insets; typography and `word-break` for scripts including Korean; reduced-motion support; print and high-contrast rendering.

### UX and task integrity

one primary action per decision point; information architecture; progressive disclosure; feedback and system status; error prevention and recovery; loading, empty, and error states; destructive-action confirmation; undo; no-op and dead CTA detection; misleading success detection; double-submit prevention; deep-link restorability and correct browser history; role and audience specific copy; cognitive load; consistency; a completable and recoverable task path.

### Optimization

measure before optimizing; user-perceived performance over synthetic numbers; Core Web Vitals evidence; bundle, request, and render cost; image and font strategy; cache and data freshness; avoiding premature memoization; avoiding blanket lazy loading; preventing cumulative layout shift; a performance budget carried in the project profile.

Each heuristic becomes one or more rule records with the fields from Section 4, including counterexamples so the rule does not overfire.

---

## 8. External advisor adapter

External frontend and UI/UX skills, including `ui-ux-pro-max`, are optional advisors. This section extends the existing provider contract to frontend advisors rather than inventing a parallel one, and imports the authority order from `shared-contracts/authority`.

### 8.1 Two provider kinds: advisory versus evidence

Research (Appendix A) shows two distinct provider roles, and conflating them would violate "evidence over taste":

- **Advisory (prescriptive) providers** recommend designs, palettes, patterns, and checklists. `ui-ux-pro-max` is one: a compact, offline, self-contained recommendation database. It suggests; it does not observe a running application. Its output is opinion or convention until confirmed by evidence.
- **Evidence (observational) providers** measure a real artifact. Playwright supplies live render, an accessibility tree, computed layout, console, and network. Figma supplies authored design intent and tokens. context7 supplies versioned normative docs. Only measured evidence from an observational provider, or a first-party measurement, can support a blocking finding.

Consequence for grading: an advisory provider's suggestion enters the finding core with **claim type `RECOMMENDATION` or `HYPOTHESIS` and no evidence grade**. It receives an evidence grade only if a separate, reproducible observation is attached to it. It cannot, by itself, reach a grade that supports a blocker. An evidence provider's measurement enters as a graded `FACT` at the grade its method supports.

### 8.2 Scoping a real advisor: `ui-ux-pro-max`

Because this advisor was verified in detail (Appendix A), the RFC records how a real project should scope it, without vendoring or hard-depending on it:

- **Use only the offline advisory core.** Its search core is Python standard library only, needs no network or API key, and can emit JSON, the clean adapter surface. Its generative sub-skills (logo, banner, icon, social image) call an external image model and are online and not read-only; the read-only advisory adapter MUST exclude them.
- **Pin an explicit version.** An installed marketplace copy can lag upstream (observed: an in-use copy behind the latest release). The adapter pins a version per request and never assumes installed equals latest.
- **Respect its license before any reuse.** It is MIT per its manifest metadata, which permits reuse and vendoring with attribution, but the exact notice wording must be confirmed from the LICENSE file before any copy is made. The core does not vendor it.

### 8.3 The adapter contract

- **capability discovery**: detect whether an advisor is installed and what it can answer, without assuming a specific one.
- **provider selection**: choose an advisor only when expected information gain exceeds context and conflict cost, and only for its declared capability.
- **version and provenance**: pin the advisor version for the request and record provider ID, version, and query.
- **input contract**: send one scoped question plus the relevant project constraints, never the whole workspace.
- **output normalization**: convert advisor output into shared finding-core records, claim-typed as `RECOMMENDATION` or `HYPOTHESIS`; unnormalizable output is recorded as `unverified` context, not a finding.
- **claim type and grade**: advisory output carries no evidence grade on its own. A grade is assigned only to a separately attached, reproducible observation.
- **conflict resolution**: resolve conflicts by the authority order in `shared-contracts/authority` (law and safety, explicit user decisions, project instructions and product truth, accepted canon, measured implementation evidence, provider guidance, generic convention). An advisor never wins over canon.
- **unavailable fallback**: if no advisor is installed, continue with first-party evidence and mark only material advisor-dependent questions as unverified.
- **no direct authority**: advisors cannot approve gates, mutate canon, persist a design system, implement code, or expand scope.
- **no automatic canon mutation**: advisor suggestions enter the normal design-change path, never a silent edit.
- **no hidden chain-of-thought requirement**: the adapter consumes evidence, alternatives, and rationale, not a private token-by-token trace.

The core does not vendor any advisor repository and does not make any advisor a hard dependency. License, version, and availability must be verified before an advisor is referenced in a real project profile; unverified advisor content is never copied into the core. The existing `E2` example for this provider in the alpha.4 files is inconsistent with this section and is corrected per Appendix C.

---

## 9. Finding schema

One finding shape, specialized from `shared-contracts/finding-core`, carries both first-party and normalized advisor findings. Fields:

- `id` (stable)
- `category` (module)
- `claim_type` (`FACT` | `INFERENCE` | `RECOMMENDATION` | `HYPOTHESIS` | `DECISION_REQUIRED` | `UNVERIFIED`, from shared claim-types)
- `severity` (`blocker` | `major` | `moderate` | `minor`, from shared severity)
- `confidence` (`high` | `medium` | `low`, from shared contracts)
- `affected_user` (role or audience)
- `affected_context` (route, state, viewport, browser, locale, role)
- `surface` (`deployed` | `candidate` | `both`)
- `observation`
- `evidence` (optional; grade E0 to E4 from the shared ladder; absent for a pure recommendation or hypothesis)
- `reference` (standard fact, project applicability, or canon, kept distinct)
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

Constraints: a finding whose `claim_type` is `RECOMMENDATION` or `HYPOTHESIS` has no evidence grade and cannot carry `severity: blocker`. Only a graded `FACT` (or an `INFERENCE` with named bridge and evidence) can block a gate. There is no fabricated aggregate score; a finding never rolls up into a 0 to 100 number that hides which user, in which context, cannot do what.

---

## 10. Project profile

All project specifics and applicability decisions live in a `frontend-profile.schema.json` instance, never in the core. Fields:

- framework and runtime (activates the matching framework-adapter modules; absent means no framework adapter loads)
- styling system
- design source
- tokens and components
- supported browsers
- locales
- viewports
- accessibility target and the applicability decision for standard requirements
- performance budgets, module budget, and context budget
- user roles
- sensitive surfaces
- regulated boundaries and their decision owners
- required tests
- optional advisors (with pinned version policy)
- override rules (which rule IDs a project may relax, within each rule's `override_policy`)

The core MUST NOT hardcode Next.js, Tailwind, a specific locale such as Korean, or a specific route. Those appear only as values in a profile or as clearly labeled examples. React and Next.js judgment lives in framework-adapter modules that activate only when the profile declares them. Korean line-break handling is a `typography-i18n` capability that activates when a profile declares a relevant locale, not a core assumption. Whether a given accessibility or safety standard binds is an applicability decision in the profile or a `regulated_decision`, not a core default.

---

## 11. Documentation budget

- Not every check is stored as Markdown. Routine successful checks are transient or CI artifacts, not durable records.
- Durable decisions and machine evidence are separated. Persist findings, dispositions, and reproduction evidence; do not persist full reviewer monologues.
- Shared contracts have a single neutral owner (`shared-contracts`), imported by both skills. Standards and shared vocabulary are cited, never copied. This is the mechanism that prevents both duplication and sibling-to-sibling dependency.
- Rules have one canonical format (data), and human docs are generated from or reference it. The same rule text never lives in two files.
- Every rule and source carries `source_version` and `last_reviewed`. A stale source is visible, not silently trusted.
- A compact owner brief and the audit artifact are separate outputs. The brief is short; the audit is reproducible.
- Module references load only when their triggers fire, within the profile's context budget.
- Deprecated rules are marked deprecated with a replacement pointer and a removal target, never deleted silently while findings still cite them.

---

## 12. Evaluation strategy

Evaluation asserts what MUST be found, what MUST NOT be flagged, and what MUST NOT even activate. Over-flagging and over-fixing are failures, not neutral. A minimal eval runner ships in PR-A, and every later module PR adds its own fixtures.

Fixture kinds:

- **positive**: a real defect the module must find.
- **negative (non-finding)**: a correct pattern the module must not flag.
- **router non-activation**: an unrelated change for which the module must not activate at all, so the two-stage router is tested, not just the rules.

Minimum fixture set:

| Fixture | Kind | Expectation |
|---|---|---|
| 1920 fixed-width page | positive | fixed width misread from canvas; horizontal scroll on smaller viewports |
| Korean mid-word wrapping | positive | `word-break` and column width break comprehension |
| flex min-width overflow | positive | missing `min-width: 0` causes overflow |
| grid overflow | positive | track sizing overflows container |
| 200% zoom failure | positive | content loss or overlap at 200% |
| sub-24px target | positive | target below the standard minimum (stated as fact; applicability per profile) |
| modal focus loss | positive | focus not trapped or not returned |
| dead CTA | positive | action leads nowhere valid |
| fake success | positive | success shown before the effect occurred |
| hydration mismatch | positive | server and client markup diverge |
| unauthorized content flash | positive | protected content renders before auth resolves |
| stale async response | positive | superseded response overwrites current state |
| double submit | positive | duplicate submission not prevented |
| missing loading or error state | positive | state gap in the running app |
| CLS from font or image | positive | layout shift from unsized asset or font swap |
| unnecessary React rerender | positive | avoidable rerender, with measurement |
| missing deep-link handling | positive | route state not restorable from URL |
| broken browser history | positive | back and forward behave incorrectly |
| correct fluid `max-width` | negative | must not be flagged as fixed width |
| valid accessibility exception | negative | a legitimately exempt control must not be flagged |
| backend-only or copy-only change | router non-activation | frontend layout and a11y modules must not activate |

Each case declares expected findings, expected non-findings, expected non-activation, and the evidence needed to reach the verdict. False positives, over-fixes, and spurious activation are scored explicitly.

---

## 13. PR decomposition

After this RFC is accepted, implementation is split so each PR is independently reviewable, verifiable on arrival, and never a monolith:

- **PR-A (foundation and verifiability first)**: promote the shared contracts into `shared-contracts`; add `rule.schema.json`, `finding.schema.json`, `frontend-profile.schema.json`; add the router `SKILL.md` with two-stage activation; add the **minimal eval runner**. Nothing after this lands without tests.
- **PR-B**: semantic-html, css-layout, responsive, typography-i18n. Each module ships its rules plus positive, negative, and router-non-activation fixtures.
- **PR-C**: accessibility, interaction, ux-product, plus the `task-integrity` composite. Fixtures colocated.
- **PR-D**: testing-verification and the implementation-acceptance synthesis (test-pass versus experience-pass). Fixtures colocated.
- **PR-E**: framework adapters react and nextjs, profile-triggered. Fixtures colocated.
- **PR-F**: optional advisor adapters and claim-typed output normalization.
- **PR-G**: remaining modules (performance, security-privacy, resilience, assets-fonts, browser-compatibility, maintainability, design-to-code, observability, visual-design runtime lens). Fixtures colocated.
- **PR-H**: integration, performance of the router itself, and a real-project pilot.

Ordering rationale: contracts, schemas, and the eval runner first so every later PR arrives verified; then the v1 core modules with the strongest deterministic and runtime evidence and the highest non-expert user impact; then framework adapters; then advisors once the finding core is stable; then the remaining depth; then a pilot. No module PR accumulates without its own fixtures.

---

## 14. Acceptance criteria

- A user with no frontend or UI/UX expertise receives guidance framed by real user impact.
- The core operates with no external skill installed.
- Shared contracts have a single neutral owner and neither skill depends on the other.
- Advisory output is claim-typed as recommendation or hypothesis and never carries an evidence grade or a blocker on its own.
- When an external advisor is present, its evidence and provenance are preserved and reconciled against the authority order.
- References unrelated to the current task are not loaded, and unrelated modules do not activate.
- Every rule carries at least one exception, a verification method, and a single canonical data record.
- The v1 set can detect user-task-completion defects (dead CTA, false success, double submit, lost recovery), not only layout and markup issues.
- A standard's technical fact is stated without the core deciding its applicability.
- A canonical canvas is never treated as a fixed page width.
- A visual baseline never substitutes for user approval or runtime evidence.
- React and CSS optimizations are never forced without measurement.
- Accessibility, UX, performance, and maintainability are considered together, not in isolation.
- No project-specific policy enters the reusable core.
- Every module PR ships with its own positive, negative, and non-activation fixtures.
- Human-facing reports are compact while audit evidence is reproducible.

---

## 15. Open questions

Reviewers are asked to weigh in on:

1. Which rules belong in the core as non-overridable within an adopting profile, and which are always project-overridable?
2. What is the update policy for the source registry (WCAG, React, browser support), and who owns `last_reviewed`?
3. Is claim-type `RECOMMENDATION` or `HYPOTHESIS` with no default grade the right treatment for all advisory output, or are there advisory outputs that should never even reach a hypothesis?
4. Where exactly is the boundary between a framework-adapter module and the core, especially for Next.js server and client specifics?
5. What is the allowed range of future automated fixes, per module `auto_fix_boundary`?
6. What is the minimum fixture bar (positive, negative, non-activation) required to merge each module PR?
7. What are sensible defaults for the module budget and context budget in two-stage activation?
8. How should `applicability` interact with `override_policy` when a profile adopts, then tries to relax, a safety or legal minimum?
9. Where is the line between subjective visual taste and an objective defect, so taste is offered as opinion and defects as findings?
10. Confirm the proposed v1 core set: semantic-html, css-layout, responsive, typography-i18n, accessibility, ux-product, interaction, testing-verification, and the `task-integrity` composite (forms, state-data, routing), with react and nextjs shipped as profile-triggered framework adapters. Deferred: performance depth, security-privacy, resilience, assets-fonts, browser-compatibility, maintainability, design-to-code automation, observability, and the visual-design runtime lens.

---

## Scope and non-goals

**In scope for this RFC**: the structure above (taxonomy, boundary, shared contracts, rule schema and canonical format, router and activation, modes, integration and update contract, advisor adapter, finding schema, project profile, documentation budget, evaluation strategy, PR decomposition, acceptance criteria, open questions).

**Non-goals for this RFC**:

- no runtime skill logic or `SKILL.md` router implementation,
- no module knowledge files or rule data,
- no JSON Schema files,
- no evaluation fixtures or runner,
- no auto-fixer,
- no copying of any external skill's content,
- no project-specific policy in the core,
- no edits to `design-sync`, `design-review`, or other alpha.4 files (the consistency fixes in Appendix C are proposed, not applied here),
- no merge.

## Not implemented in this PR

This PR adds only this RFC document and a docs index entry. Everything in Sections 3 through 13, and every fix in Appendix C, is a proposal to be built in the PR-A through PR-H sequence after acceptance.

## Review request

Codex and Claude reviewers are asked to evaluate:

- the neutral `shared-contracts` layer and whether it fully removes the sibling dependency (Sections 3.2, 5, 11),
- the claim-typed treatment of advisory output and the `E2` correction (Sections 8, 9, Appendix C),
- the revised v1 set and the framework-adapter split (Sections 3, 10, 15),
- the single canonical rule format and the data-to-docs relationship (Sections 4, 5, 11),
- the PR order that moves the eval runner to PR-A and colocates fixtures (Sections 12, 13),
- the two-stage activation and the design-sync update contract (Sections 5.1, 6.1).

---

## Appendix A: Advisor and evidence-provider landscape (research)

Capability-level research used to ground Section 8. No external skill content was copied. Items marked "verify" are not settled and must be confirmed before a project relies on them.

**Advisory (prescriptive) provider.** `nextlevelbuilder/ui-ux-pro-max-skill`: a compact, self-contained UI/UX recommendation database (a small bundled CSV and JSON corpus with a thin Python query layer). It recommends palettes, font pairings, per-stack patterns, anti-patterns, and pre-delivery checklists from a natural-language request. It exposes a deterministic query CLI that can emit JSON, which is the clean adapter surface. Its advisory core is offline and needs no network or API key. Its generative image sub-skills are online and not read-only and are out of scope for a read-only adapter. License is MIT per manifest metadata (permits reuse and vendoring with attribution; verify the exact LICENSE text before any copy). Versioning is formal SemVer with frequent releases; an installed marketplace copy can lag upstream, so pin a version and do not assume installed equals latest (verify the installed version at use time). Because it is prescriptive, its output is a recommendation or hypothesis, not evidence.

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

Grounds `shared-contracts/source-registry` and the per-rule `sources`, `source_version`, and `last_reviewed` fields. Status values are as of mid-2026 and must be re-reviewed on the cadence the profile sets.

| Domain | Canonical primary source | Versioning and citation |
|---|---|---|
| HTML | WHATWG HTML Living Standard | living standard, no version number; cite by section anchor plus a commit or permalink and the last-updated date |
| CSS | W3C CSS Working Group modular specs, with MDN and browser-compat data as companions | each spec carries a maturity status (Working Draft, Candidate Recommendation, Recommendation); cite the module and status |
| React | react.dev official docs | versioned to the React release line; track by release and a doc last-reviewed date |
| Accessibility | W3C WCAG 2.2 (current Recommendation), with WAI-ARIA and the ARIA Authoring Practices Guide | cite by success-criterion number plus WCAG version; WCAG 3.0 is a Working Draft only and must not be cited as normative |
| Web performance | Google web.dev and the web-vitals definitions | Core Web Vitals evaluated at the 75th percentile of field data; interaction metric replaced the older input-delay metric in 2024; field data and lab tooling are separate sources |

Management model: treat references as docs-as-code. Keep one machine-readable sources registry with an entry per source carrying organization, document, canonical URL or permalink, version or spec status, retrieved date, last-reviewed date, next-review-due date, and applicable scope. Pin living standards to a commit or permalink rather than a moving latest pointer. Run scheduled reviews and link checks. This registry is the single owner of normative claims that both `design-review` and `frontend-quality` cite, and it can be vendored offline as the fallback normative layer.

## Appendix C: Required consistency fixes to existing alpha.4 files

These changes make the current alpha.4 code coherent with this RFC. They are **not applied in this document-only PR**; they are proposed for PR-A or the alpha.4 branch and are listed here so the maintainer can schedule them.

1. **Promote shared contracts to a neutral layer.** Move the contract portions of the current `design-review` references into `plugins/claude-design-sync/shared-contracts/`:
   - `skills/design-review/references/evidence-and-decisions.md` provides the evidence ladder and the claim-type labels, which become `shared-contracts/evidence` and `shared-contracts/claim-types`.
   - the severity list in `skills/design-review/SKILL.md` becomes `shared-contracts/severity`.
   - the authority order in `skills/design-review/references/design-intelligence-providers.md` becomes `shared-contracts/authority`.
   - the response modes in `skills/design-review/references/response-modes.md` become the shared response-mode contract.
   - the accessibility source URLs and any other standard citations seed `shared-contracts/source-registry`.
   `design-review` then references the neutral layer instead of owning it, so `frontend-quality` can import the same layer without a sibling dependency.

2. **Correct the advisory evidence grade.** In `skills/design-review/references/design-intelligence-providers.md` (the recommended profile block) and in `examples/project-profile.example.yaml` (`integrations.ui_ux_pro_max.evidence_grade: E2`), remove the fixed `E2` grade for the advisory provider. Replace it with a claim-type default of `RECOMMENDATION` or `HYPOTHESIS` and no evidence grade; a grade attaches only to a separate reproducible observation. Update any prose that describes the provider as carrying a project-evidence grade.

3. **Note the framework-adapter split.** No code change is required in alpha.4, but the roadmap item that names React or Next.js work should record that framework judgment lives in profile-triggered adapter modules, not the core, consistent with Section 10.
