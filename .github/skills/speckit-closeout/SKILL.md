---
name: "speckit-closeout"
description: "Close out a completed feature specification by promoting durable knowledge into docs/ living documents, creating Architecture Decision Records, harvesting cross-cutting lessons learned, and archiving the spec. This is the final step in the spec-driven development lifecycle."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/closeout.md"
---


## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

The user input provides:

1. **A spec folder path** (e.g., `specs/0014-alert-filtering`) — REQUIRED
2. **An optional `--dry-run` flag** — if present, generate the close-out summary
   without modifying any `docs/` files

If no spec folder path is provided, STOP and ask the user which spec to close out.

## Pre-Execution Checks

**Check for extension hooks (before closeout)**:

- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_closeout` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing slash commands from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):

    ```text
    ## Extension Hooks

    **Optional Pre-Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```

  - **Mandatory hook** (`optional: false`):

    ```text
    ## Extension Hooks

    **Automatic Pre-Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}

    Wait for the result of the hook command before proceeding to the Goal.
    ```

    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.

- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Goal

Close out a completed feature specification by promoting its durable knowledge
into the project's living documentation, creating Architecture Decision Records,
harvesting lessons learned, and archiving the spec. This is the **final step**
in the spec-driven development lifecycle and MUST run only after
`/speckit-implement` (and, where used, `/speckit-converge`) have completed the
feature's work.

## Core Principle: Edit, Don't Append

**Always locate existing sections and update them in place.** Appending is a
fallback only when no suitable section exists. Specific examples:

- In `spec.md`, if a `**Status**:` line already exists, **update it** with the
  new value — do NOT insert a second `**Status**:` line.
- In living docs, find the relevant section/table and update or extend it rather
  than appending duplicate content at the end of the file.
- When in doubt, search the target file for the key phrase before writing.

## Operating Constraints

- **DOCUMENTATION-ONLY**: This command only touches documentation files. Do
  **not** modify application code.
- **Do not run tests or builds** — these are documentation-only changes.
- **Only close out the target spec** — do not perform a separate repo-wide cross-spec supersession audit beyond updating the living docs/ADRs that overlap this spec’s topics.
- **Ignore `specs/GAP_ANALYSIS.md`** — that is a separate periodic audit.
- **`--dry-run`**: when this flag is present, produce the close-out summary and
  promotion plan **without modifying any `docs/` files**.

**Constitution Authority**: The project constitution
(`.specify/memory/constitution.md`) governs this close-out (e.g., Principle X
requires every configuration entry to have a Bicep location). If the
constitution is an unfilled template, skip constitution checks gracefully.

## Execution Steps

Execute these steps **in order**. Do NOT skip steps — mark items N/A with a
reason if they don't apply.

### Step 0: Load Context

1. Run `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`
   once from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS. If the
   user provided an explicit spec folder path, use that as FEATURE_DIR.
   For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").
2. Read `.specify/templates/closeout-template.md` — this is your working checklist
3. Read `.specify/memory/constitution.md` — understand governance requirements
4. Read `.specify/memory/lessons-learned/README.md` — understand lesson criteria
5. Read all files in the target spec folder: `spec.md`, `plan.md`, `tasks.md`,
   `research.md`, `data-model.md`, `quickstart.md`, `contracts/`, and any
   completion summaries (`PHASE*_*`, `US*-IMPL-*`, `*_SUMMARY.md`)
6. Read the current state of ALL target living docs in `docs/`:
   - `docs/product-spec.md`
   - `docs/product-spec-traceability.md`
   - `docs/technical-overview.md`
   - `docs/architecture.md`
   - `docs/data-model-reference.md`
   - `docs/glossary.md`
   - `docs/configuration.md`
   - `docs/scenarios.md`
   - `docs/adr/README.md`

### Step 1: Pre-Flight Checks

1. Scan `tasks.md` for unchecked items (`- [ ]`). If any exist:
   - List them in your output
   - Ask the user whether to proceed anyway or abort
2. Note any open items — they will go into the Close-Out Summary

### Step 2: Plan Promotions (Semantic Delta Analysis)

Promotion is **not** an additive operation. For each target living doc, perform
a semantic delta analysis — match by **topic and decision subject**, not by
exact string. Closeouts that produce purely additive growth in docs that
describe a *changed* part of the system are almost always wrong: prior content
should have been rewritten or superseded.

#### 2.1 Per-target-doc procedure

For each target doc (product-spec, technical-overview, architecture,
data-model-reference, ADRs, glossary, configuration, scenarios), do the following:

1. **Inventory existing content** that touches the same capability, component,
   data flow, decision area, entity, term, or env var as the new spec. Match by
   topic, not by string. Read the relevant sections in full before classifying.
2. **Classify each candidate promotion** as exactly one of:
   - **NEW** — no prior coverage of this topic exists; add it.
   - **UPDATE-IN-PLACE** — prior coverage exists and is still partially correct;
     rewrite the existing section to reflect the new state. Do **not** add a
     parallel section that duplicates or contradicts it.
   - **SUPERSEDE** — prior coverage is now wrong, obsolete, or contradicted;
     mark it superseded (Superseded Requirements table entry, "Superseded by
     SPEC-####" annotation, or ADR `Status: Superseded by ADR-####-MMM`) and
     replace it with the new content in the same edit.
   - **SKIP** — already accurately covered; do nothing.

#### 2.2 Doc-specific matching rules

- **ADRs (`docs/adr/`)** — match by **decision subject** (the question being
  decided: "which vector store", "which auth flow", "how do agents
  communicate"), regardless of title wording. Titles drift; subjects don't.
  - If an existing ADR decides the same question and the answer is unchanged →
    SKIP (do not create a near-duplicate).
  - If the answer has changed → create a new ADR and mark the prior one
    `Status: Superseded by ADR-####-MMM` in the same edit.
- **`docs/product-spec.md`** — match user stories, FRs, NFRs, and SCs by
  **capability + actor + outcome**, not by ID or wording. The presence or
  absence of a `*(US-####-NN)*` back-reference is **not** sufficient evidence
  of novelty: a capability promoted under a different phrasing in a prior
  closeout is still prior coverage. If a prior story is now subsumed, narrowed,
  or contradicted, route it to the Superseded Requirements table in the same
  edit that promotes the new story.
- **`docs/architecture.md`** — match by **component identity** (component
  names) and **data-flow endpoints**. Rewrite the existing
  component/flow description in place; do not add a second paragraph describing
  the same component.
- **`docs/technical-overview.md`** — match by **dependency name**. A version
  bump is an UPDATE-IN-PLACE on the existing row plus a single Version Change
  Log entry — never a new dependency row.
- **`docs/data-model-reference.md`** — match by **entity name**. Schema changes
  are UPDATE-IN-PLACE; renamed/removed entities go to the Deprecated Entities
  table.
- **`docs/glossary.md` / `docs/configuration.md`** — exact-string matching on
  term/variable name remains correct here; UPDATE-IN-PLACE if the definition or
  semantics changed, SKIP if identical, and annotate as deprecated if removed.
- **`docs/product-spec-traceability.md`** — match by **spec number + user story
  ID**. When a US/FR/NFR/SC is added, updated, or superseded in
  `docs/product-spec.md`, update the corresponding traceability row in lockstep.
- **`docs/scenarios.md`** — match by **scenario ID** first, then by
  **purpose + expected outcomes** (the demo's intent and what it proves), not
  by name wording. Scenario names drift the same way ADR titles do.
  - Same ID, same purpose/outcomes → SKIP or UPDATE-IN-PLACE if data files,
    expected outcomes, or supporting components changed.
  - New scenario covering a purpose already demonstrated by an existing row →
    prefer UPDATE-IN-PLACE (extend the existing row's outcomes / data files)
    over adding a near-duplicate row in a different category.
  - Scenario retired or replaced by this spec → mark the existing row as
    superseded (annotate the row, e.g., "Superseded by spec ####" in the Name
    column) rather than silently leaving stale rows in the catalog.
  - Genuinely new purpose → append a new row and place it under the correct
    Scenario Category (add a new category subsection only if none fits).

#### 2.3 Promotion plan output

The promotion plan you present to the user **must** include, per target doc, a
classification count:

```
docs/product-spec.md:        NEW: 3   UPDATE-IN-PLACE: 2   SUPERSEDE: 1   SKIP: 4
docs/architecture.md:        NEW: 0   UPDATE-IN-PLACE: 2   SUPERSEDE: 0   SKIP: 0
docs/adr/:                   NEW: 1   UPDATE-IN-PLACE: 0   SUPERSEDE: 1   SKIP: 2
...
```

This makes growth visible up-front. If every doc's plan is `NEW: N, others: 0`
for a spec that *changed* existing behavior, re-do the inventory — you almost
certainly missed prior coverage.

Wait for user confirmation before proceeding to Step 3. (When `--dry-run` is
set, stop here after presenting the plan and the summary — do not edit `docs/`.)

### Step 3: Execute Promotions

**Subtractive pass first — before any additive edit to a doc:**

> Identify what this spec **invalidates** in the target doc. List existing user
> stories, FRs, NFRs, SCs, components, data flows, entities, ADRs, env vars, or
> glossary terms that this spec changes, narrows, retires, or contradicts.
> Apply those changes — mark as Superseded, rewrite in place, move to a
> Deprecated table, or remove — **first**. Only then promote new content.

This is the operational counterpart to Step 2's UPDATE-IN-PLACE and SUPERSEDE
classifications: do not leave stale content next to its replacement.

Work through each close-out template section:

#### 3.1 Product Specification (`docs/product-spec.md`)

- Organize content by **capability area** (not by spec number)
- Add back-references like `*(US-0014-01)*` to every promoted item
- If a capability area section doesn't exist yet, create it
- Mark superseded requirements in the Superseded Requirements table

#### 3.2 Technical Overview (`docs/technical-overview.md`)

- Add new rows to the appropriate tables
- Record the spec number in the "Introduced" column
- Reference ADR where applicable (e.g., link to `ADR-####-MMM` for technology choices)
- If an existing dependency version changed, also add a Version Change Log entry
- Use **one Version Change Log row per dependency version change** to match the
  existing `Dependency | Old Version | New Version | Spec | Rationale` table structure
- Do **not** add net-new dependencies to the Version Change Log; record them only in
  the appropriate dependency tables with the spec number in the "Introduced" column

#### 3.3 Architecture (`docs/architecture.md`)

- Update component descriptions, data flows, and diagrams
- Only modify if this spec introduced architectural changes
- Preserve the existing document structure and ASCII art style

#### 3.4 Data Model Reference (`docs/data-model-reference.md`)

- Add new entity sections with schema details from `data-model.md`
- Update the Contracts Index table with any new API contracts
- Move deprecated entities to the Deprecated Entities table

#### 3.5 Architecture Decision Records (`docs/adr/`)

- ADR numbering: `ADR-####-MMM` where #### = spec number, MMM = sequential
- Scan `research.md` for `RT-*` tagged decisions
- For each significant decision, **first** check whether an existing ADR
  decides the **same question** (subject + scope), regardless of title wording.
  Title drift is expected; subject identity is what matters. Examples of "same
  question": "which vector store", "which auth flow for the operator UI", "how
  do agents communicate", "where does conversation state live".
  - Same question, same answer → SKIP (do not create a near-duplicate ADR).
  - Same question, different answer → create a new ADR from
    `.specify/templates/adr-template.md` AND mark the prior ADR
    `Status: Superseded by ADR-####-MMM` with a forward link in the same edit.
  - New question → create a new ADR.
- **Every new ADR's `Context` section MUST explicitly cite related prior ADRs**
  (extends, refines, supersedes, or "no related prior ADR"). This forces the
  agent to look. A new ADR with no Related-ADRs statement fails the Step 6 ADR
  gate.
- Update the Index table in `docs/adr/README.md`.

#### 3.6 Lessons Learned (`.specify/memory/lessons-learned/`)

- Scan completion summaries for cross-cutting gotchas
- Apply the criteria from `.specify/memory/lessons-learned/README.md`
- Append to EXISTING topic files — do NOT create new per-spec files
- Leave implementation-specific details in PR descriptions
- **Stricter four-gate check** — each candidate lesson MUST pass ALL four:
  1. **Novelty** — not already captured in the constitution, copilot-instructions,
     or an existing lessons-learned file
  2. **Actionability** — phrased as "do X" or "don't Y" (not a vague observation)
  3. **Specificity** — tied to a concrete API, failure mode, or tool behavior
  4. **Reusability** — applies beyond this single spec (useful on an unrelated task)
- Zero lessons is an acceptable and expected outcome when no new cross-cutting
  insight was discovered.
- **Footer-aware insertion** — if the target lessons-learned topic file ends
  with a `**Last Updated**: YYYY-MM-DD` footer, insert the new section
  **before** that footer line and update the date to today in the same edit.
  Otherwise, follow the file's existing structure: append at the end if there
  is no footer, or update an existing header date (for example, `**Date**`) if
  that is the file's established convention.

#### 3.7 Glossary (`docs/glossary.md`)

- Add new domain terms; maintain alphabetical sort order
- Include the spec number in the "Introduced" column
- When updating an existing term's definition, set the "Updated" column to
  this spec number (leave blank on first introduction)

#### 3.8 Configuration Reference (`docs/configuration.md`)

- Cross-reference `plan.md` and application code for env vars / secrets
- Verify each entry has a Bicep location (Constitution Principle X)

#### 3.9 Scenarios (`docs/scenarios.md`)

- Only applicable if this spec introduced demo scenarios
- Mark N/A if not applicable

#### 3.10 Product Spec Traceability (`docs/product-spec-traceability.md`)

- Update the traceability matrix when user stories, FRs, NFRs, or SCs are
  added, updated, or superseded in `docs/product-spec.md`
- Each row maps a spec → user story → FR/NFR → SC chain; keep chains unbroken
- Flag gaps or ambiguity with ⚠️ annotations
- Mark N/A if no product-spec changes were promoted

#### 3.11 README & Quickstart

- Merge per-spec `quickstart.md` content if it adds new setup steps
- Update `README.md` only if top-level usage or getting-started changed

### Step 4: Archive the Spec

1. Update the `**Status**:` line in `spec.md` to `Closed — [YYYY-MM-DD]`.
   If no `**Status**:` line exists yet, add one below any existing
   frontmatter/title. **Never insert a duplicate status line.**
2. Do NOT delete any files from the spec folder — they are immutable history

### Step 4.5: Lint Pass

A **mechanical** pass that runs after promotions and before the summary. Each
sub-step has a matching gate in Step 6; failing here means re-doing the
relevant Step 3 promotion, not just ticking a box.

#### 4.5.1 Living-doc header block

For **every** living doc you touched (and any seed doc you created from a
template), ensure the file begins with the standard header block.

Default form (docs without a Superseded Requirements section, e.g.
`technical-overview.md`, `data-model-reference.md`, `configuration.md`,
`glossary.md`, `scenarios.md`, `product-spec-traceability.md`):

```
**Version:** YYYY.MM.DD  •  **Living document** — updated by `speckit.closeout`.
**Last updated:** YYYY-MM-DD by closeout of SPEC-####.
**Changelog:** see `specs/`.
```

Extended form for `docs/product-spec.md` only (which has a Superseded
Requirements section):

```
**Version:** YYYY.MM.DD  •  **Living document** — updated by `speckit.closeout`.
**Last updated:** YYYY-MM-DD by closeout of SPEC-####.
**Changelog:** see `specs/` and the [Superseded Requirements](#superseded-requirements) table.
```

Rules:

- `Version` is `YYYY.MM.DD` of today's closeout (calendar-versioned, not semver).
- `Last updated` and the closing `SPEC-####` reference are bumped to **this**
  closeout — never left as the prior closeout's values, never left as `TBD`.
- Only include the second `[Superseded Requirements](#superseded-requirements)`
  link when the doc actually contains a `## Superseded Requirements` heading
  (otherwise the anchor is broken).
- If the file already has the header block, this is an **UPDATE-IN-PLACE**
  edit on those three lines. If the file is missing the block entirely,
  **insert** it directly under the H1.

#### 4.5.2 FR-ID continuity audit

For every spec whose requirements are referenced or promoted in this closeout
(typically just the closing spec, but include any spec touched by a
SUPERSEDE action), enumerate its `FR-####-001 .. FR-####-MAX` ID space and
account for every gap. A gap is any missing integer in the contiguous
sequence. Each gap MUST be resolved by exactly one of:

(a) **Restore** — the FR text was lost; recover it from spec history and
    re-promote.
(b) **Supersede** — the FR was replaced or retired; add a row to the
    Superseded Requirements table in `docs/product-spec.md` with a forward
    pointer.
(c) **Intentional gap** — the IDs were skipped during refactoring; add a
    single sentence to the spec's introduction (and to its capability section
    in `docs/product-spec.md` where that spec contributed):
    *"FR-####-XXX through FR-####-YYY were intentionally not assigned during
    refactoring."*

Apply the same audit to SC-IDs and US-IDs when the spec uses contiguous
numbering.

Record each gap and its resolution in `CLOSEOUT_SUMMARY.md` under a new
`### FR-ID Continuity Audit` heading. Zero gaps is reported as
`No gaps detected in SPEC-####.`

#### 4.5.3 Cross-cutting requirements tagging

Spec-level FRs / NFRs / SCs that have no owning user story are **cross-cutting**,
not orphan. The traceability chain (spec → US → FR/SC) must remain unbroken.

In `docs/product-spec.md`, every promoted FR / NFR / SC MUST carry one of:

- `*(US-####-NN)*` — owned by user story `US-####-NN`
- `*(US-####-XC)*` — spec-level / cross-cutting (no owning US)

Procedure:

1. Scan `docs/product-spec.md` for promoted items missing both tags.
2. For each such item, decide whether it belongs to an existing user story
   (tag `*(US-####-NN)*`) or is genuinely cross-cutting (tag `*(US-####-XC)*`).
3. Cross-cutting items live under a `## Cross-cutting Requirements` heading —
   the top-level section in `docs/product-spec.md`, or a per-capability
   sub-heading where a cross-cutting item is clearly capability-scoped.

### Step 5: Generate Close-Out Summary

Create `specs/<####-feature-name>/CLOSEOUT_SUMMARY.md` containing:

- A filled-in copy of the close-out template with all items checked/annotated
- The full Close-Out Summary section (items promoted, ADRs created, etc.)
- Any open items or follow-up work identified
- A **per-doc growth budget** showing the net change made to each living doc.
  Use the format below — one line per touched doc:

  ```
  docs/product-spec.md:        +3 sections, ~2 rewritten, -1 superseded (net +2)
  docs/architecture.md:        +0 sections, ~2 rewritten,  0 superseded (net  0)
  docs/adr/:                   +1 ADR,       0 rewritten, -1 superseded (net  0)
  docs/technical-overview.md:  +0 rows,     ~1 rewritten,  0 superseded (net  0)
  ```

  `+` = NEW additions, `~` = UPDATE-IN-PLACE rewrites, `-` = SUPERSEDE actions.
  Net growth must be **consistent with the nature of the spec**: a spec that
  *changed* existing behavior should usually have non-zero `~` and/or `-`
  counts. Purely additive growth across every doc for a behavior-changing spec
  is a red flag — re-run Step 2's inventory before opening the PR.

### Step 6: Pre-PR Self-Review — Per-Document Coherence Gates

Before creating the PR, re-read **every file you touched** and answer each
gate below as a yes/no question with **specific evidence** (file path +
section/heading) supporting the "yes." A "no" on any gate requires a fix
before opening the PR. Record the gate results in `CLOSEOUT_SUMMARY.md`.

**Mechanical gates** (carry over from prior process):

1. **No pure appends where in-place edits were possible** — if content was
   added at the end of a section but a more appropriate location existed,
   move it now.
2. **Exactly one Version Change Log row per dependency version change** —
   not one per spec, not one per dependency.
3. **All back-references use the canonical format** — e.g., `*(US-0014-01)*`.
4. **Every lessons-learned item passes the four gates** (novelty,
   actionability, specificity, reusability). Remove any that fail.

**Per-document coherence gates** — each living doc must answer "yes":

5. **`docs/product-spec.md` gate** — Does this doc, read top-to-bottom,
   describe the system *as it will be after this spec ships*? Are there any
   user stories, FRs, NFRs, or SCs the new spec invalidated, narrowed, or
   contradicted that are still listed as active (rather than in the Superseded
   Requirements table)? Every active requirement must be consistent with the
   new behavior.
6. **`docs/architecture.md` gate** — Is the document internally consistent with
   the new component / data-flow reality? Do ASCII diagrams, the component map,
   and prose all agree? Are removed components actually removed (not
   merely sitting next to a new component that replaced them)?
7. **`docs/adr/` gate** — Does every new ADR trace to a decision recorded in
   `research.md` or `plan.md`? Does every new ADR's `Context` explicitly cite
   related prior ADRs (or state "no related prior ADR")? Is every prior ADR
   whose decision was changed by this spec marked `Status: Superseded by
   ADR-####-MMM` with a forward link? Are there any two active ADRs that decide
   the same question in conflicting ways?
8. **`docs/technical-overview.md` gate** — One row per dependency (no
   duplicates from prior specs). Version Change Log contains exactly the rows
   for *this spec's* version bumps, no more and no fewer.
9. **`docs/data-model-reference.md` gate** — Every entity reflects current
   schema. Renamed or removed entities are in the Deprecated Entities table,
   not silently duplicated next to their replacements.
10. **`docs/configuration.md` gate** — Every env var referenced in application
    code introduced or modified by this spec exists here AND has a Bicep
    location (Constitution Principle X). Removed env vars are annotated as
    deprecated, not silently dropped.
11. **`docs/glossary.md` gate** — Alphabetical sort intact; no duplicate or
    contradictory definitions for the same term.
12. **Cross-doc gate** — Capability names, component names, entity
    names, and ADR IDs are referenced consistently across all touched docs.
    No doc references a component name or ADR ID that doesn't exist (or that
    a sibling doc spells differently).
13. **Header-block gate** — Every touched living doc starts with
    the standard `**Version:** YYYY.MM.DD  •  **Living document** …
    **Last updated:** … **Changelog:** …` block, with `Version`,
    `Last updated`, and the closing `SPEC-####` reference bumped to **this**
    closeout (no leftover `TBD` or stale prior values).
14. **FR-ID continuity gate** — Every gap in the `FR-####-XXX`
    numbering of any spec touched by this closeout has been resolved by
    exactly one of: (a) restoring the FR text, (b) recording it in the
    Superseded Requirements table with a forward pointer, or (c) annotating
    the spec introduction with *"FR-####-XXX through FR-####-YYY were
    intentionally not assigned during refactoring."* Same rule applies to
    contiguous SC-IDs and US-IDs. Zero gaps is reported explicitly.
15. **Cross-cutting tagging gate** — Every promoted FR / NFR / SC in
    `docs/product-spec.md` carries a `*(US-####-NN)*` or `*(US-####-XC)*`
    back-reference, and every cross-cutting item lives under a
    `## Cross-cutting Requirements` heading.

### Step 7: Finalize

- Summarize the close-out in-session: items promoted, ADRs created/superseded,
  lessons harvested, and any open follow-up items.
- Title suggestion for the PR: `docs: close out spec #### — [feature name]`.
- Keep all changes in a single documentation-only changeset with the
  Close-Out Summary as the description.

## Error Handling

- If a target living doc doesn't exist, create a minimal stub (H1 + the standard living-doc header block) and mark any unknown sections as N/A until a proper seed template exists.
- If a spec folder is missing expected files (e.g., no `research.md`), skip
  the dependent sections and note them as N/A in the summary.
- If you cannot determine the correct capability area for a user story,
  flag it for human review in the summary.

## Quality Standards

- Every promoted item MUST have a spec back-reference.
- ADRs MUST follow the template in `.specify/templates/adr-template.md`.
- Lessons MUST meet the criteria in `.specify/memory/lessons-learned/README.md`.
- Glossary entries MUST be alphabetically sorted.
- Tables MUST maintain consistent column alignment.

## Check for Extension Hooks

After producing the close-out, check if `.specify/extensions.yml` exists in the project root.

- If it exists, read it and look for entries under the `hooks.after_closeout` key
- If the YAML cannot be parsed or is invalid, skip hook checking silently and continue normally
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate hook `condition` expressions:
  - If the hook has no `condition` field, or it is null/empty, treat the hook as executable
  - If the hook defines a non-empty `condition`, skip the hook and leave condition evaluation to the HookExecutor implementation
- When constructing slash commands from hook command names, replace dots (`.`) with hyphens (`-`). For example, `speckit.git.commit` → `/speckit-git-commit`.
- For each executable hook, output the following based on its `optional` flag:
  - **Optional hook** (`optional: true`):

    ```text
    ## Extension Hooks

    **Optional Hook**: {extension}
    Command: `/{command}`
    Description: {description}

    Prompt: {prompt}
    To execute: `/{command}`
    ```

  - **Mandatory hook** (`optional: false`):

    ```text
    ## Extension Hooks

    **Automatic Hook**: {extension}
    Executing: `/{command}`
    EXECUTE_COMMAND: {command}
    ```

    After emitting the block above you MUST actually invoke the hook and wait for it to finish before continuing. Run it the same way you would run the command yourself in this agent/session (the invocation may differ from the literal `{command}` id shown above, e.g. a skills-mode agent runs it as `/skill:speckit-...` or `$speckit-...`). Emitting the block alone does not run the hook.

- If no hooks are registered or `.specify/extensions.yml` does not exist, skip silently

## Context

$ARGUMENTS
