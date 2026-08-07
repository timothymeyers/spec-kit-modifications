# Spec Kit Modifications

A general-purpose collection of amended [Spec Kit](https://github.com/github/spec-kit)
skills, prompts, templates, and workflow guidance.

**Baseline**: spec-kit **v0.14.4**, installed with the Copilot skills integration:

```bash
specify init --here --force --integration copilot --integration-options=--skills
```

That command generates `.github/skills/speckit-*/SKILL.md`, `.specify/templates/`,
`.specify/scripts/bash/`, and the `.specify/*.json` state files. Everything below
describes what this repository changes **on top of** that generated baseline.

> **Note on generated output.** `specify init` itself rewrites the upstream
> `templates/commands/*.md` sources when it installs them: it resolves
> `__SPECKIT_COMMAND_*__` tokens to `/speckit-*`, expands `{SCRIPT}` to a concrete
> `.specify/scripts/bash/...` invocation, normalises `{ARGS}` to `$ARGUMENTS`,
> rewrites `/memory/` and `templates/` to `.specify/memory/` and
> `.specify/templates/`, adds the `SKILL.md` YAML frontmatter, injects the
> "replace dots with hyphens" hook note, and rewrites `format_speckit_command`
> calls in the shell scripts. Those are **not** modifications — they are vanilla
> CLI behaviour and are excluded from the delta list below.

---

## Summary of changes

| # | Change | Kind |
|---|--------|------|
| 1 | Spec numbers are GitHub issue numbers, zero-padded to 4 digits | Modified |
| 2 | Spec-scoped traceability IDs on every artifact item | Modified |
| 3 | `/speckit-clarify` is non-interactive and writes `clarify-report.md` | Modified |
| 4 | `/speckit-analyze` writes `spec-analysis.md` | Modified |
| 5 | `/speckit-taskstoissues` links task issues back to the parent spec issue | Modified |
| 6 | `/speckit-closeout` — promote specs into `docs/` living documents | New skill |
| 7 | `/specify-model-council-review` — three-model review council | New skill |
| 8 | ADR, close-out, and lessons-learned templates | New templates |
| 9 | Five non-speckit domain skills | New skills |
| 10 | Four custom agents (`.github/agents/`) | New |
| 11 | Two-phase (spec branch → impl branch) workflow guide | New docs |

---

## 1. Spec numbers are GitHub issue numbers

Vanilla spec-kit numbers features sequentially with a 3-digit prefix
(`specs/003-user-auth`). This repository aligns the spec number to the **GitHub
issue that tracks the feature**, zero-padded to 4 digits (`specs/0042-user-auth`
for issue #42). Numbers ≥ 10000 keep their natural width.

The number is *consumed*, never created — the issue must already exist.

**What changed**

- `.specify/scripts/bash/create-new-feature.sh`
  - new `--issue N` flag (accepts an optional leading `#`, digits only)
  - `--issue` takes precedence over `--number` and is never auto-corrected on
    conflict; a mismatched `--number` emits a warning
  - `--issue` is ignored (with a warning) when `--timestamp` is used
  - zero-padding widened from `printf "%03d"` to `printf "%04d"`, including the
    conflict-resolution retry loop
- `.specify/init-options.json` — adds `"spec_numbering": "issue"` and
  `"spec_number_width": 4`
- `speckit-specify` — documents the issue-alignment rule, reads `spec_numbering`
  and `spec_number_width` from `init-options.json`, and uses `0042-user-auth`
  style examples
- `spec-template.md` — new `**Spec Number**` and `**GitHub Issue**` header fields
- `plan-template.md` / `tasks-template.md` — `[####-feature-name]` placeholders

## 2. Spec-scoped traceability IDs

Every item in every artifact carries a stable, spec-scoped identifier, so a task
can be traced to an acceptance scenario, to a user story, to a requirement, and
back to the originating GitHub issue.

| Artifact item | This repo | Vanilla |
|---------------|-----------|---------|
| User story | `US-####-NN` | unnumbered heading |
| Acceptance scenario | `AS-####-NN-MM` | unnumbered list item |
| Edge case | `EC-####-NN` | unnumbered bullet |
| Functional requirement | `FR-####-NNN` | `FR-001` |
| Success criterion | `SC-####-NNN` | `SC-001` |
| Task | `T-####-NNN` | `T001` |
| Task story label | `[US-####-NN]` | `[US1]` |
| Checklist item | `CHK-####-NNN` | `CHK001` |
| Clarification finding | `CLAR-NNN` | n/a |
| Architecture decision record | `ADR-####-MMM` | n/a |

`####` is the spec number from change 1, so identifiers are globally unique
across specs rather than only unique within one spec.

**What changed**

- `spec-template.md` — an `ID CONVENTIONS` comment block plus IDs applied to
  user stories, acceptance scenarios, edge cases, functional requirements, and
  success criteria
- `tasks-template.md` — `T-####-NNN` task IDs, `[US-####-NN]` story labels,
  user-story IDs on phase headings and in the dependency section
- `checklist-template.md` — `CHK-####-NNN` item IDs
- `plan-template.md` — `**Spec Number**` field in the header line
- `speckit-specify` — an explicit "Identifier conventions" step that lists each
  scheme and states that downstream skills depend on them
- `speckit-tasks` — task ID and story-label format rules plus correct/incorrect
  examples rewritten for the new scheme
- `speckit-checklist` — new/append numbering starts at `CHK-####-001`, and the
  worked examples reference `FR-####-NNN`
- `speckit-analyze` — the requirements inventory keys on `FR-####-NNN` /
  `SC-####-NNN`
- `speckit-converge` — loads `FR-####-NNN`, `SC-####-NNN`, `US-####-NN`,
  `AS-####-NN-MM`, and `EC-####-NN`; appended convergence tasks continue the
  `T-####-NNN` sequence and use IDs such as `AS-0042-01-02` as their source ref

## 3. `/speckit-clarify` is non-interactive and writes a report

Vanilla `clarify` is an interactive loop: it asks at most five questions one at
a time, waits for answers, and edits `spec.md` in place (adding a
`## Clarifications` section).

Here it is a **non-interactive, report-only audit**:

- asks no questions and has no question quota — every ambiguity is reported
- never modifies `spec.md` or any checklist
- writes `FEATURE_DIR/clarify-report.md` with an executive summary, a severity
  summary table, per-finding detail, and a taxonomy coverage summary
- each finding gets a stable `CLAR-NNN` ID (sorted by severity, then location),
  a severity of `CRITICAL`/`HIGH`/`MEDIUM`/`LOW`, a concrete recommendation,
  a rationale, and alternatives with tradeoffs
- the taxonomy scan gains a `Not Applicable` status, and planning-phase
  decisions are still reported with an explicit "defer to `/speckit-plan`"
  recommendation

This makes clarification reviewable in a pull request instead of consumed in a
chat session, and it is what feeds `/specify-model-council-review` (change 7).

## 4. `/speckit-analyze` writes a report

Vanilla `analyze` is strictly read-only and prints its report to the session.

Here it is **report-only** rather than read-only: the analysed artifacts
(`spec.md`, `plan.md`, `tasks.md`, the constitution) are still never modified,
but the analysis is written to `FEATURE_DIR/spec-analysis.md`.

**What changed**

- `REPORT = FEATURE_DIR/spec-analysis.md` added to the derived paths; it is an
  output, so its absence is not an error
- report front-matter block (spec directory, generated date, finding count)
- a zero-findings run still produces a report, with an empty-state message and
  next actions indicating readiness for `/speckit-implement`
- new **Validate the Report** step: unique finding IDs, complete finding fields,
  internally consistent counts, full coverage of the requirements inventory, and
  confirmation that nothing but the report was written
- new **Completion Report** and **Done When** sections

## 5. `/speckit-taskstoissues` traces back to the parent issue

- task ID recognition updated from `\bT\d{3}\b` to `\bT-\d{4,}-\d{3}\b`, so
  spec numbers ≥ 10000 that keep their natural width still match
- canonical issue titles become `T-0042-001: <description>`; `[P]` and
  `[US-####-NN]` markers are stripped when recovering the description
- each created issue references the parent spec issue in its body (for example
  `Relates to #42`, without leading zeros), keeping tasks traceable to the spec

## 6. New skill: `/speckit-closeout`

There is no close-out step in vanilla spec-kit. This skill is the final stage of
the lifecycle, run after `/speckit-implement` (and `/speckit-converge`). It
promotes a completed spec's durable knowledge into project-owned **living
documents** and archives the spec.

- **Documentation-only** — never touches application code, never runs tests or
  builds; supports `--dry-run`
- **Edit, don't append** — locates and updates existing sections in place, with
  appending as a fallback
- Promotes into `docs/product-spec.md`, `docs/technical-overview.md`,
  `docs/architecture.md`, `docs/data-model-reference.md`, `docs/glossary.md`,
  `docs/configuration.md`, `docs/scenarios.md`, and `docs/adr/`
- Creates ADRs and harvests cross-cutting lessons into
  `.specify/memory/lessons-learned/`
- Sets the spec's `**Status**:` to `Closed — [YYYY-MM-DD]` without ever
  duplicating the status line; spec files are immutable history and are not
  deleted
- **Lint pass** — living-doc header block (`Version` / `Last updated` /
  `Changelog`), an FR-ID continuity audit that requires every gap in
  `FR-####-XXX` to be restored, superseded, or annotated as intentionally
  unassigned, and cross-cutting requirement tagging (`*(US-####-XC)*`)
- **Per-doc growth budget** — reports `+` new, `~` rewritten, `-` superseded per
  document; purely additive growth for a behaviour-changing spec is flagged
- **Pre-PR coherence gates** — per-document yes/no gates that must be answered
  with file-and-section evidence before the PR is opened
- Writes `specs/<####-feature-name>/CLOSEOUT_SUMMARY.md`

## 7. New skill: `/specify-model-council-review`

Reviews a Spec Kit report (`clarify-report.md`, `spec-analysis.md`, …) with a
council of three independent specialist reviewers, then acts on the result.

- Launches three read-only reviewers in parallel using one specialist agent type
  (default `csa`), each pinned to a different model. The exact model list is
  pinned in
  [`.github/skills/specify-model-council-review/SKILL.md`](.github/skills/specify-model-council-review/SKILL.md)
  and is the source of truth — update it there, not here, as models change
- Council members get identical inputs, may not edit files, may not invoke
  remediation skills, and may not talk to one another
- **Unanimity is consensus**; a 2–1 split is a disagreement. The coordinator does
  not cast a tie-breaking vote
- Unanimous, low-risk remediations are applied automatically **through the skill
  that owns the artifact** — `/speckit-specify` for `spec.md`, `/speckit-plan`
  for `plan.md` / `research.md` / `data-model.md` / `quickstart.md` /
  `contracts/`, `/speckit-tasks` for `tasks.md`
- Only genuine disagreements or material risks (security, privacy, compliance,
  irreversible data handling, unauthorised cross-cutting commitments) are
  escalated to a human
- Every disposition is recorded back into the source report; chain-of-thought is
  never surfaced

## 8. New templates

| Template | Purpose |
|----------|---------|
| `.specify/templates/adr-template.md` | Architecture Decision Record — `ADR-####-MMM`, status/supersession, options considered, consequences, related ADRs |
| `.specify/templates/closeout-template.md` | The working checklist for `/speckit-closeout`, including the lint pass, growth budget, and coherence gates |
| `.specify/templates/lessons-learned-template.md` | Reusable lessons for `.specify/memory/lessons-learned/`, with cloud-parity notes and tags |

## 9. Additional domain skills

Skills with no vanilla equivalent, loaded on demand when a request matches their
`description`:

| Skill | Purpose |
|-------|---------|
| `azure-iac-expert` | Azure IaC across Commercial / Government / Government Secret clouds — endpoints, service parity, cross-platform deploy scripts |
| `maf-developer` | Microsoft Agent Framework (Python `agent-framework`) development, with a documentation-first policy |
| `presentation` | Turning dense material into decision-driving decks and narratives |
| `requirements-spec-manager` | Auditing specs for consistency, completeness, traceability, and requirements-lifecycle quality |
| `rnd` | Research-and-learning mindset that always captures lessons under `.specify/memory/lessons-learned/` |

## 10. Custom agents

`.github/agents/` holds persona definitions — a distinct mindset for a whole
task, in contrast to skills, which package reference knowledge or a repeatable
procedure. See [`.github/agents/README.md`](.github/agents/README.md) for the
agent-vs-skill heuristic and the record of which former agents were converted to
skills.

| Agent | Purpose |
|-------|---------|
| `csa` | Cloud Solution Architect — architecture, NFRs, diagrams |
| `python-developer` | Python persona with an enforced TDD loop and `ruff` |
| `data-scientist` | Exploratory data analysis working mode |
| `security-engineer` | Microsoft security stack review and threat modeling |

## 11. Two-phase workflow guide

[`.specify/README.md`](.specify/README.md) documents the end-to-end process this
repository is built around: **two branches, two merges**.

- **Phase I — Spec Package** on `specs/NNNN-my-feature`: `/speckit-specify` →
  `/speckit-clarify` → `/specify-model-council-review` → review gate →
  `/speckit-plan` + `/speckit-tasks` → `/speckit-analyze` → iterate until analyze
  reports ready → merge to `main` **before any code exists**
- **Phase II — Implementation** on `impl/NNNN-my-feature`, branched fresh from
  the updated `main`: scoped `/speckit-implement` runs → `/speckit-closeout` →
  review → merge

`.github/copilot-instructions.md` records the repository conventions that follow
from this: consult the constitution, `docs/`, and `specs/` before starting; keep
every artifact independent of any particular product, customer, or deployment.

---

## Inventory

### Skills — `.github/skills/`

| Skill | Origin | Status |
|-------|--------|--------|
| `speckit-specify` | vanilla | modified (changes 1, 2) |
| `speckit-clarify` | vanilla | modified (change 3) |
| `speckit-plan` | vanilla | unmodified |
| `speckit-tasks` | vanilla | modified (change 2) |
| `speckit-checklist` | vanilla | modified (change 2) |
| `speckit-analyze` | vanilla | modified (changes 2, 4) |
| `speckit-implement` | vanilla | unmodified |
| `speckit-converge` | vanilla | modified (change 2) |
| `speckit-taskstoissues` | vanilla | modified (changes 2, 5) |
| `speckit-constitution` | vanilla | unmodified |
| `speckit-closeout` | — | new (change 6) |
| `specify-model-council-review` | — | new (change 7) |
| `azure-iac-expert` | — | new (change 9) |
| `maf-developer` | — | new (change 9) |
| `presentation` | — | new (change 9) |
| `requirements-spec-manager` | — | new (change 9) |
| `rnd` | — | new (change 9) |

### Templates — `.specify/templates/`

| Template | Origin | Status |
|----------|--------|--------|
| `spec-template.md` | vanilla | modified (changes 1, 2) |
| `plan-template.md` | vanilla | modified (changes 1, 2) |
| `tasks-template.md` | vanilla | modified (changes 1, 2) |
| `checklist-template.md` | vanilla | modified (change 2) |
| `constitution-template.md` | vanilla | unmodified |
| `adr-template.md` | — | new (change 8) |
| `closeout-template.md` | — | new (change 8) |
| `lessons-learned-template.md` | — | new (change 8) |

### Scripts — `.specify/scripts/bash/`

| Script | Origin | Status |
|--------|--------|--------|
| `create-new-feature.sh` | vanilla | modified (change 1) |
| `check-prerequisites.sh` | vanilla | unmodified |
| `common.sh` | vanilla | unmodified |
| `setup-plan.sh` | vanilla | unmodified |
| `setup-tasks.sh` | vanilla | unmodified |

---

## Upgrading spec-kit

Re-running `specify init --here --force …` overwrites the generated skills,
templates, and scripts, which discards every modification listed above. After an
upgrade, re-apply the deltas — the last upgrade
(`0.12.11.dev0` → `0.14.4`) needed exactly that, in two follow-up passes:
restoring the spec-numbering customisations, then restoring the non-interactive
clarify and analyze reports.

A practical checklist:

1. Diff each `.github/skills/speckit-*/SKILL.md` against the corresponding
   `templates/commands/*.md` in the upstream tag you upgraded to.
2. Re-apply changes 1–5 to the regenerated skills, templates, and
   `create-new-feature.sh`.
3. Confirm `.specify/init-options.json` still carries `spec_numbering` and
   `spec_number_width`.
4. Confirm the added skills, templates, and agents (changes 6–10) are still
   present and still consistent with the regenerated vanilla skills — `specify
   init` does not know about them, so they are never updated for you.
