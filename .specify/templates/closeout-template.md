# Spec Close-Out: [SPEC NAME]

**Spec**: `specs/[####-feature-name]/`
**Date**: [DATE]
**Closer**: [human or agent]

<!--
  ============================================================================
  IMPORTANT: This template is the working checklist for the speckit.closeout
  agent. It is also suitable for manual close-out by human contributors.

  The agent reads this template, creates a copy as
  specs/<####-feature-name>/CLOSEOUT_SUMMARY.md, and works through each section. Items
  marked N/A are annotated with a reason and left unchecked.

  DO NOT modify this template for a specific spec — copy it first.
  ============================================================================
-->

## Pre-Flight Checks

- [ ] All tasks in tasks.md are checked complete (or explicitly deferred with reason)
- [ ] No open PRs remain against the spec's feature branch
- [ ] CI is green on the final merge to main

## 1. Product Specification → `docs/product-spec.md`

Organize by **capability area** (Visualization, Alerting, COA, NL Query, etc.),
not by spec number. Each promoted item includes a parenthetical back-reference
for traceability.

**Canonical back-reference format** (use exactly these patterns — no variants):

- User stories and acceptance scenarios: `*(US-NNNN-MM)*` (e.g., `*(US-0001-01)*`)
- Functional/non-functional requirements and success criteria: `*(US-NNNN-MM)*`
  for items owned by a user story, or `*(US-NNNN-XC)*` for cross-cutting items
  with no owning story

- [ ] User stories promoted to the appropriate capability-area section
- [ ] Functional requirements promoted (FR-####-NNN → capability area)
- [ ] Non-functional requirements / success criteria promoted
- [ ] Superseded requirements from prior specs annotated as such

## 2. Technical Overview → `docs/technical-overview.md`

- [ ] New languages, frameworks, or libraries recorded
- [ ] New Azure services recorded with one-line rationale (reference ADR where applicable)
- [ ] Version changes to existing dependencies noted

**Version Change Log guidance**: Add **exactly one** Version Change Log row per
dependency version change (not one per spec). Net-new dependencies are recorded
only in the dependency tables with the spec number in the "Introduced" column —
not in the Version Change Log.

## 3. Architecture → `docs/architecture.md`

- [ ] New or modified components updated in the component map
- [ ] New or modified data flows updated
- [ ] ASCII diagrams still accurate after changes

## 4. Data Model & Contracts → `docs/data-model-reference.md`

- [ ] New entities added (with schema, indexes, validation)
- [ ] Modified entities updated (note which spec last touched)
- [ ] Contracts index updated (OpenAPI/AsyncAPI locations + introducing spec)
- [ ] Removed or deprecated entities annotated

## 5. Architecture Decision Records → `docs/adr/`

ADRs use spec-prefixed numbering: `ADR-####-MMM` where #### is the spec number
and MMM is a sequential counter within that spec (e.g., ADR-0001-001, ADR-0001-002).

- [ ] Each RT-* decision in research.md evaluated for ADR creation
- [ ] New ADR created for each significant decision not already recorded
- [ ] Existing ADRs checked — supersede if this spec overturned a prior choice

## 6. Lessons Learned → `.specify/memory/lessons-learned/`

- [ ] Completion summaries (PHASE*_, US*-IMPL-*) scanned for cross-cutting gotchas
- [ ] Cross-cutting lessons appended to the appropriate existing topic file
- [ ] Implementation-specific details left in PR descriptions (not promoted here)

## 7. Glossary → `docs/glossary.md`

- [ ] New domain terms introduced by this spec added
- [ ] Existing definitions updated if meaning changed

## 8. Configuration Reference → `docs/configuration.md`

- [ ] New env vars documented (name, purpose, default, Bicep location)
- [ ] Removed env vars annotated as deprecated

## 9. Scenarios → `docs/scenarios.md` (if applicable)

- [ ] New demo scenarios cataloged (ID, name, purpose, expected outcomes)

## 10. README & Quickstart

- [ ] `README.md` or `docs/README.md` updated if setup or usage changed
- [ ] Per-spec `quickstart.md` content merged into docs (not duplicated)

## 11. Archive Spec

- [ ] `spec.md` header annotated with `**Status**: Closed — [DATE]`
- [ ] No files deleted from `specs/####/` (immutable history)

## 12. Lint Pass

Run these mechanical checks across every living doc touched by this closeout.
Each item is a hard gate — a "no" requires a fix before the PR is opened.

### 12.1 Living-Doc Header Block

- [ ] Every touched living doc has a stable header block. Default form (docs
      without a Superseded Requirements section):
      ```
      **Version:** YYYY.MM.DD  •  **Living document** — updated by `speckit.closeout`.
      **Last updated:** YYYY-MM-DD by closeout of SPEC-####.
      **Changelog:** see `specs/_archive/`.
      ```
      Extended form for `docs/product-spec.md` (has a Superseded Requirements
      section), append the second link to the Changelog line:
      `… and the [Superseded Requirements](#superseded-requirements) table.`
- [ ] `Version`, `Last updated`, and the closing-spec ID have been bumped to
      reflect this closeout (not left as the prior closeout's values, and not
      left as `TBD`).
- [ ] The `[Superseded Requirements](#superseded-requirements)` link is only
      present in docs that actually have that heading (no broken anchors).

### 12.2 FR-ID Continuity Audit

For every spec whose requirements were promoted (or are referenced) in this
closeout, walk the FR-ID space (`FR-####-001 .. FR-####-MAX`) and account for
every gap:

- [ ] No unexplained gaps in the `FR-####-XXX` numbering of any touched spec.
      For each gap, exactly one of the following is true and recorded:
      (a) the missing FR text has been restored; OR
      (b) the missing FR-ID appears in the Superseded Requirements table with
          a pointer to its replacement / retirement; OR
      (c) the spec's introduction (or `docs/product-spec.md` capability section
          for that spec) contains a single sentence:
          *"FR-####-XXX through FR-####-YYY were intentionally not assigned
          during refactoring."*
- [ ] Same audit performed for SC-IDs and US-IDs where the spec uses a
      contiguous numbering scheme.

### 12.3 Cross-cutting Requirements Tagging

Every promoted FR / NFR / SC must carry a back-reference. Items that legitimately
have no owning user story are **cross-cutting**, not orphan.

- [ ] No promoted item in `docs/product-spec.md` is missing both a
      `*(US-####-NN)*` tag **and** a `*(US-####-XC)*` tag.
- [ ] Items that are spec-level (no owning US) live under a
      `## Cross-cutting Requirements` heading (the top-level section in
      `docs/product-spec.md`, or a per-capability sub-heading where
      appropriate) and are tagged `*(US-####-XC)*`.
- [ ] Any US-orphan items surfaced during the audit have either been re-tagged
      to their owning user story or moved under a Cross-cutting Requirements
      sub-heading.

---

## Close-Out Summary

- **Items promoted**: [count]
- **ADRs created**: [list, e.g., ADR-0001-001, ADR-0001-002]
- **ADRs superseded**: [list with forward links, e.g., ADR-0007-001 → ADR-0014-002]
- **Lessons harvested**: [list of topic files appended to]
- **Skipped / N/A**: [list with reasons]
- **Open items for follow-up**: [list]

### Per-Doc Growth Budget

Report net change for every living doc touched. `+` = NEW, `~` = UPDATE-IN-PLACE,
`-` = SUPERSEDE / Deprecated. Purely additive growth across every doc for a
behavior-changing spec is a red flag — re-run the Step 2 inventory if you see
that pattern.

```
docs/product-spec.md:        +N sections, ~N rewritten, -N superseded (net ±N)
docs/architecture.md:        +N sections, ~N rewritten, -N superseded (net ±N)
docs/adr/:                   +N ADRs,     ~N rewritten, -N superseded (net ±N)
docs/technical-overview.md:  +N rows,     ~N rewritten, -N superseded (net ±N)
docs/data-model-reference.md:+N entities, ~N rewritten, -N deprecated (net ±N)
docs/configuration.md:       +N vars,     ~N rewritten, -N deprecated (net ±N)
docs/glossary.md:            +N terms,    ~N rewritten, -N superseded (net ±N)
```

### Per-Document Coherence Gates

Each gate must answer **yes** with specific evidence (file path + section)
before the PR is opened. A "no" requires a fix first.

- [ ] **Mechanical**: no pure appends where in-place edits were possible
- [ ] **Mechanical**: exactly one Version Change Log row per dependency version change
- [ ] **Mechanical**: all back-references use the canonical `*(US-####-NN)*` format
- [ ] **Mechanical**: every lessons-learned item passes all four gates (novelty, actionability, specificity, reusability)
- [ ] **product-spec gate**: doc reads as the system *as it will be after this spec ships*; no invalidated user stories / FRs / NFRs / SCs left listed as active
- [ ] **architecture gate**: `architecture.md` is internally consistent; ASCII diagrams, component map, and prose all agree; removed components are actually removed
- [ ] **ADR gate**: every new ADR traces to research/plan and cites related prior ADRs; superseded ADRs marked with forward link; no two active ADRs decide the same question in conflict
- [ ] **technical-overview gate**: one row per dependency (no duplicates); Version Change Log has exactly this spec's bumps
- [ ] **data-model-reference gate**: every entity reflects current schema; renamed/removed entities are in the Deprecated Entities table
- [ ] **configuration gate**: every env var referenced in app code is documented here AND has a Bicep location (Constitution Principle X); removed vars annotated deprecated
- [ ] **glossary gate**: alphabetical sort intact; no duplicate or contradictory definitions
- [ ] **cross-doc gate**: capability names, component names, entity names, and ADR IDs are referenced consistently across all touched docs
- [ ] **header-block gate**: every touched living doc has the standard `**Version:** … **Last updated:** … **Changelog:** …` block, bumped for this closeout
- [ ] **FR-ID continuity gate**: every gap in `FR-####-XXX` numbering of any touched spec is explained — restored, recorded in Superseded Requirements, or annotated as intentionally unassigned
- [ ] **cross-cutting tagging gate**: no promoted FR / NFR / SC in `docs/product-spec.md` lacks both a `*(US-####-XC)*` back-reference; spec-level items live under a `## Cross-cutting Requirements` sub-heading
