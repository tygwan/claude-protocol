# Design intelligence providers

External skills and knowledge bases may enrich critique, but they are advisory providers. They never become project canon, approval authority, or an implementation agent merely because they are installed.

## Authority order

Resolve conflicts in this order:

1. law, safety, and named regulated decisions,
2. explicit user decisions,
3. project instructions and product truth,
4. accepted design canon and design tokens,
5. measured implementation evidence,
6. provider guidance,
7. generic convention.

A provider can reveal a question or alternative. It cannot silently replace a higher authority.

## Activation

Invoke an available provider only when its expected information gain exceeds its context and conflict cost.

Useful triggers:

- a new visual pattern without project precedent,
- missing or weak project canon,
- explicit request for alternatives or inspiration,
- typography, color, layout, interaction, responsive, or accessibility risk,
- cross-stack implementation guidance,
- a review where the built-in references leave a material uncertainty.

Skip when:

- implementing an exact accepted canon,
- applying established tokens or components mechanically,
- the task is backend or API only,
- the question is a regulated product-policy decision,
- the provider would repeat evidence already available,
- invoking it would broaden scope without user value.

## Provider contract

Record:

- provider ID and version,
- activation reason,
- query or scope,
- evidence returned,
- conflicts with project authority,
- accepted, rejected, or deferred insights.

Providers are read-only by default. Unless a profile explicitly grants more, they may not:

- edit project or design files,
- persist a design system,
- implement code,
- decide product or regulated policy,
- approve a semantic gate,
- expand the current request.

Pin the provider version for the duration of one request. An update takes effect only at a durable checkpoint and must not alter an approval target mid-gate.

## UI UX Pro Max adapter

`nextlevelbuilder/ui-ux-pro-max-skill` is supported as an optional advisory provider. Do not vendor its repository into this plugin and do not make it a cross-marketplace hard dependency.

Recommended profile:

```yaml
integrations:
  ui_ux_pro_max:
    adapter: claude_plugin_skill
    plugin: ui-ux-pro-max@ui-ux-pro-max-skill
    mode: auto_if_available
    role: advisory
    evidence_grade: E2
    version_policy: pin_per_request
    permissions:
      read_only: true
      modify_project: false
      persist_design_system: false
      implement_code: false
      decide_policy: false
    activation:
      stages: [request_discovery, design_authoring, design_review]
      when_any:
        - new_visual_pattern
        - missing_project_canon
        - design_alternatives_requested
        - typography_or_color_decision
        - interaction_quality_question
        - responsive_or_accessibility_risk
        - explicit_user_request
      skip_when:
        - exact_canon_implementation
        - mechanical_css_fix
        - backend_or_api_only
        - established_token_application
        - regulated_policy_decision
```

Provider numbers or generic rules are not automatically conformance requirements. For example, distinguish normative accessibility criteria from audience-specific or project-specific recommendations and cite which one drives the decision.

## Invocation sequence

1. Read project instructions, accepted canon, and local evidence first.
2. Perform the kernel's open read without provider anchoring.
3. Test the configured activation and skip conditions.
4. If activated and installed, call the provider with one scoped question and the relevant project constraints.
5. Label returned guidance with provider ID, pinned version, and evidence grade.
6. Reconcile it against the authority order; state conflicts explicitly.
7. Keep only insights that improve the current decision.
If the provider is unavailable, continue with the built-in review references and record the provider-dependent question as unverified only when it is material. Never block routine work merely because an optional provider is absent.
