<!--
================================================================================
SYNC IMPACT REPORT
================================================================================
Version Change: (new) → 1.0.0 (DRAFT — not yet ratified)

Rationale: Initial draft of the Atlas Intelligence Constitution, still under active
authorship and review. Captures the governance principles for the Atlas Federal AI
story intake and catalog platform at governance-principle altitude; implementation
detail lives in the product spec, architecture, and configuration docs. Version
remains 1.0.0 until the first ratification.

Added Sections:
  - "Mission & Scope" preamble — defines Atlas as the intake, validation,
    scoring, approval workflow, and catalog publication platform
  - Principle I:   "Human-in-the-Loop" — the human is the final decision-maker at
    every approval gate; AI drafts and recommends but does not make binding decisions
  - Principle II:  "Deterministic-First, AI-Enhanced" — prefer deterministic logic;
    reserve AI for problems that require it; the system MUST work without AI configured
  - Principle III: "Spec-First Development" — specs precede implementation, derived
    from GitHub Issues where possible; close-out required after merge
  - Principle IV:  "Human-AI Collaboration" — humans set direction and approve;
    AI agents implement and suggest; same PR process for all
  - Principle V:   "Automated Testing" — automated tests with an 80% coverage target;
    no weakening tests to pass a feature
  - Principle VI:  "Configuration-Driven Deployment" — env vars for all external
    endpoints; consistent naming; graceful fallback when unset
  - Principle VII: "Documentation Currency" — living `docs/` Markdown updated with
    every workflow/API/data-model/deployment/export change

Removed Sections:
  - (none — initial version)

Templates Requiring Updates:
  - .specify/templates/plan-template.md: ✅ No updates required
  - .specify/templates/spec-template.md: ✅ No updates required
  - .specify/templates/tasks-template.md: ✅ No updates required
================================================================================
-->

# Atlas Intelligence Constitution

## Mission & Scope

**Atlas** is the Microsoft Federal AI story intake, validation, scoring, Research
Leads management, Awaiting Catalog Approval workflow, catalog publication, and
customer-shareable slide export platform.

All work on Atlas MUST be evaluated against a single standard: *Does this make it
easier for a human curator to confidently approve, enrich, or reject a Federal AI
story, and safer to share the resulting catalog entry with customers?*

The current workbench is a **governed catalog MVP**, not a production enterprise
platform. Features that do not serve the curator review loop or catalog publication
safety MUST be deferred.

## Core Principles

### I. Human-in-the-Loop

The human is the final decision-maker at every approval gate. No automated process,
AI agent, or system action may substitute for explicit human approval. AI components
may draft, suggest, synthesize, and recommend — they MUST NOT make binding publication
or approval decisions.

### II. Deterministic-First, AI-Enhanced

Prefer deterministic algorithms for any problem that can be adequately solved that
way. Reserve agentic AI for problems that genuinely require it. AI enhancements are
layered on top of a working deterministic baseline — the system MUST function
correctly without AI components configured. AI output is advisory; it does not drive
production decisions without human review.

### III. Spec-First Development

Features MUST begin with specifications. Wherever possible, specs are derived from
feature requests documented in GitHub Issues. Specs define acceptance criteria before
implementation begins. This applies to both human and AI contributors.

- Specifications MUST reside in the `specs/` directory (`specs/<feature-name>/spec.md`)
- GitHub Copilot discovers, loads, and uses `/speckit-*` skills to create and manage
  specifications
- After the final implementation PR is merged, every spec MUST be closed out using
  the `/speckit-closeout` skill to promote durable knowledge into `docs/` living
  documents and archive the spec with a `Closed` status

### IV. Human-AI Collaboration

- Humans set direction, define specs, and provide final approval
- AI agents implement, suggest, and iterate
- All contributors (human and AI) follow the same PR process
- Code review is mandatory for all changes
- Comments MUST be minimal: prefer self-documenting code over commentary; inline
  comments are reserved for non-obvious logic only

### V. Automated Testing

All software MUST have automated tests that can run without manual intervention.
Target 80% code coverage across the entire codebase and per-package.

- Tests MUST be runnable without additional setup beyond `pip install -e .[dev]`
- New API behaviour, persistence transitions, and integration paths MUST each have
  corresponding test coverage
- Removing or weakening an existing test to make a new feature pass is not acceptable

See `docs/technical-overview.md` for the standard test commands.

### VI. Configuration-Driven Deployment

All external integration points MUST be controlled by environment variables. No
hardcoded values for endpoints, credentials, model names, or resource identifiers
are permitted in source code. Environment variables MUST follow a consistent naming
convention (`ATLAS_<INTEGRATION>_<SETTING>`). When variables are absent, the system
MUST fall back gracefully without crashing.

See `docs/configuration.md` for the full variable reference.

### VII. Documentation Currency

When workflow, API, data-model, deployment, or export behaviour changes, the
corresponding living doc under `docs/` MUST be updated in the same PR. Living
`docs/` Markdown files are the source of truth.

## Amendment Procedure

To amend this constitution:

1. Create a PR with proposed changes to `.specify/memory/constitution.md`
2. Use the `/speckit-constitution` skill to apply and propagate changes
3. Include a Sync Impact Report comment block at the top of the file documenting
   the version change, rationale, modified/added/removed sections, and any
   templates requiring updates
4. Obtain two human approvals
5. Update the version following semantic versioning:
   - **MAJOR**: Principle removed, redefined, or governance invariant changed
   - **MINOR**: New principle or section added, or materially expanded guidance
   - **PATCH**: Clarifications, wording, typo fixes, non-semantic refinements

**Version**: 1.0.0 (draft) | **Ratified**: TODO(RATIFICATION_DATE): not yet ratified — constitution is still in draft on this PR | **Last Amended**: 2026-07-14
