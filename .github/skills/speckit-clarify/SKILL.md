---
name: "speckit-clarify"
description: "Identify all ambiguities in the current feature spec, recommend resolutions, and write a non-interactive clarify-report.md."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/clarify.md"
---


## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before clarification)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_clarify` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing command invocations from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```
  - **Mandatory hook** (`optional: false`):
    ```
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Outline.
    ```
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.
- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Outline

Goal: Perform a comprehensive, non-interactive ambiguity audit of the active feature specification and write every finding with a recommended resolution to `FEATURE_DIR/clarify-report.md`. Complete this workflow before invoking `/speckit-plan`.

## Operating Constraints

- **NON-INTERACTIVE**: Do not ask the user clarification questions, pause for answers, or limit the audit to a question quota.
- **REPORT-ONLY**: Do not modify `FEATURE_SPEC` or any checklist. Write or replace only `FEATURE_DIR/clarify-report.md`.
- **COMPLETE COVERAGE**: Include every ambiguity, inconsistency, missing decision, and untestable statement found in the spec. Do not cap the number of findings.
- **ACTIONABLE RECOMMENDATIONS**: Every finding MUST include a concrete recommended resolution and enough rationale for a reviewer to decide whether to apply it.

Execution steps:

1. Run `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` from repo root **once** (combined `--json --paths-only` mode / `-Json -PathsOnly`). Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)
   - If JSON parsing fails, abort and instruct user to re-run `/speckit-specify` or verify feature branch environment.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **IF EXISTS**: Load `.specify/memory/constitution.md` for project principles and governance constraints.

3. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing / Not Applicable. Use the coverage map to ensure the report accounts for every category.

   Functional Scope & Behavior:
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   Domain & Data Model:
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle/state transitions
   - Data volume / scale assumptions

   Interaction & UX Flow:
   - Critical user journeys / sequences
   - Error/empty/loading states
   - Accessibility or localization notes

   Non-Functional Quality Attributes:
   - Performance (latency, throughput targets)
   - Scalability (horizontal/vertical, limits)
   - Reliability & availability (uptime, recovery expectations)
   - Observability (logging, metrics, tracing signals)
   - Security & privacy (authN/Z, data protection, threat assumptions)
   - Compliance / regulatory constraints (if any)

   Integration & External Dependencies:
   - External services/APIs and failure modes
   - Data import/export formats
   - Protocol/versioning assumptions

   Edge Cases & Failure Handling:
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution (e.g., concurrent edits)

   Constraints & Tradeoffs:
   - Technical constraints (language, storage, hosting)
   - Explicit tradeoffs or rejected alternatives

   Terminology & Consistency:
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   Completion Signals:
   - Acceptance criteria testability
   - Measurable Definition of Done style indicators

   Misc / Placeholders:
   - TODO markers / unresolved decisions
   - Ambiguous adjectives ("robust", "intuitive") lacking quantification

   For each category with Partial or Missing status, add one or more findings. If the unresolved decision belongs in planning rather than the specification, still include it and recommend explicitly deferring that decision to `/speckit-plan`.

4. Build a complete finding inventory. For each issue:
   - Sort findings by severity (CRITICAL→LOW), then by spec location.
   - Assign a stable sequential ID in that sorted order using `CLAR-001`, `CLAR-002`, and so on.
   - Record the taxonomy category and source location using section names and line numbers when available.
   - State the ambiguity as a direct question that identifies the decision the spec leaves unresolved.
   - Assign severity:
     - **CRITICAL**: Blocks the feature's core purpose, conflicts with the constitution, or leaves a security/compliance obligation undefined.
     - **HIGH**: Materially affects architecture, data modeling, acceptance testing, user access, or integration behavior.
     - **MEDIUM**: Affects edge cases, operational readiness, measurable quality, or consistency.
     - **LOW**: Limited implementation or validation impact but still ambiguous.
   - Provide a specific recommendation grounded in the spec, constitution, established project constraints, and risk reduction.
   - Explain the recommendation in 1–3 concise sentences.
   - Where multiple viable choices exist, list the alternatives and their tradeoffs, but still identify one preferred recommendation.
   - Do not invent product decisions silently or present assumptions as confirmed requirements.

5. Write `FEATURE_DIR/clarify-report.md` with this structure:

   ```markdown
   # Clarification Report: [Feature Name]

   **Spec**: [relative path to spec.md]
   **Generated**: YYYY-MM-DD
   **Findings**: [total]

   ## Executive Summary

   [Brief readiness assessment and highest-risk themes.]

   ## Severity Summary

   | Severity | Count |
   |----------|-------|
   | CRITICAL | [count] |
   | HIGH | [count] |
   | MEDIUM | [count] |
   | LOW | [count] |

   ## Clarification Findings

   ### CLAR-001 — [Short title]

   - **Category**: [taxonomy category]
   - **Severity**: [CRITICAL/HIGH/MEDIUM/LOW]
   - **Location**: [section and line(s)]
   - **Ambiguity**: [direct clarification question]
   - **Recommendation**: [preferred resolution]
   - **Rationale**: [why this resolution best fits]
   - **Alternatives**: [viable alternatives and tradeoffs, or "None identified"]

   ## Coverage Summary

   | Taxonomy Category | Status | Finding IDs | Notes |
   |-------------------|--------|-------------|-------|

   ## Recommended Next Actions

   [Ordered remediation guidance, including whether planning should wait.]
   ```

   If no ambiguities exist, still create the report with `Findings: 0`, a zeroed severity summary, an empty-state message under Clarification Findings, a complete coverage table, and a recommendation to proceed to `/speckit-plan`.

6. Validate the report before completion:
   - Every Partial or Missing item is represented by a finding.
   - Every finding has a unique ID, severity, location, ambiguity, recommendation, rationale, and alternatives field.
   - Every recommendation answers its ambiguity directly and does not conflict with the spec or constitution.
   - Finding counts in the metadata, body, severity summary, and coverage table agree.
   - Findings are ordered by severity, then by spec location.
   - The report contains no request for immediate user input and does not imply an interactive follow-up.
   - `FEATURE_SPEC` and checklist files remain unchanged.

Behavior rules:

- On every successful invocation with an existing feature spec, create `FEATURE_DIR/clarify-report.md`, including when no ambiguities are found.
- If spec file missing, instruct user to run `/speckit-specify` first (do not create a new spec here).
- Never ask the user questions during execution and never impose a maximum finding count.
- Avoid speculative tech stack questions unless the absence blocks functional clarity.
- Include LOW-severity ambiguities rather than silently omitting them.
- Prefer recommendations that preserve technology-agnostic requirements; defer non-blocking implementation mechanics to planning.

Context for analysis: $ARGUMENTS

## Mandatory Post-Execution Hooks

**You MUST complete this section before reporting completion to the user.**

Check if `.specify/extensions.yml` exists in the project root.
- If it does not exist, or no hooks are registered under `hooks.after_clarify`, skip to the Completion Report.
- If it exists, read it and look for entries under the `hooks.after_clarify` key.
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue to the Completion Report.
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing command invocations from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Mandatory hook** (`optional: false`) — **You MUST emit `EXECUTE_COMMAND:` for each mandatory hook**:
    ```
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```
    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.
  - **Optional hook** (`optional: true`):
    ```
    ## Extension Hooks

    **Optional Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```

## Completion Report

Report completion after the report is written:
- Path to `clarify-report.md`.
- Total findings and severity counts.
- Taxonomy categories with findings.
- Whether any planning-blocking findings remain.
- Confirm that the spec and checklists were not modified.
- Suggested next command.

## Done When

- [ ] All spec ambiguities identified without asking the user questions
- [ ] Every finding includes a recommendation and rationale
- [ ] `FEATURE_DIR/clarify-report.md` created and validated
- [ ] Spec and checklist files left unchanged
- [ ] Extension hooks dispatched or skipped according to the rules in Mandatory Post-Execution Hooks above
- [ ] Completion reported with report path, finding counts, blocking status, and next command
