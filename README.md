# Tim's Spec Kit Modifications

This repository contains Tim's customizations to the default
[GitHub Spec Kit](https://github.com/github/spec-kit) workflow. It is a focused
set of Spec Kit assets rather than a complete copy of the upstream repository.

## What changed

### End-to-end traceability

Spec-scoped identifiers now connect artifacts from specification through
implementation and closeout:

| Artifact | Identifier |
|---|---|
| User story | `US-####-NN` |
| Acceptance scenario | `AS-####-NN-MM` |
| Edge case | `EC-####-NN` |
| Functional requirement | `FR-####-NNN` |
| Success criterion | `SC-####-NNN` |
| Checklist item | `CHK-####-NNN` |
| Task | `T-####-NNN` |
| Architecture decision | `ADR-####-MMM` |

The `####` segment is the feature's four-digit spec number. The modified
specification, checklist, task, analysis, convergence, and issue-conversion
skills preserve and validate these references across `spec.md`, `plan.md`,
`tasks.md`, reports, and GitHub issues.

### GitHub issue-aligned feature numbering

The feature creation workflow now:

- accepts `--issue N` to align a spec with an existing GitHub issue;
- defaults to four-digit, zero-padded spec numbers while preserving numbers
  greater than 9999;
- supports sequential and timestamp-based numbering;
- links generated task issues back to the parent feature issue; and
- deduplicates task issues using their spec-scoped task IDs.

These defaults are configured in
[`.specify/init-options.json`](.specify/init-options.json) and implemented by
[`create-new-feature.sh`](.specify/scripts/bash/create-new-feature.sh).

### Expanded quality workflow

The stock Spec Kit lifecycle has been extended with these behaviors:

- **`/speckit-clarify`** performs a complete, non-interactive ambiguity audit.
  It always writes or replaces `clarify-report.md`, includes every finding with
  severity and a recommended resolution, and does not modify the specification.
- **`/speckit-analyze`** performs a read-only consistency and coverage audit
  across `spec.md`, `plan.md`, and `tasks.md`, including buildable success
  criteria and constitution violations.
- **`/speckit-converge`** compares the implemented code with the spec package
  and appends traceable remediation tasks without rewriting existing tasks or
  application code.
- **`/speckit-closeout`** promotes durable knowledge into living documentation,
  updates traceability, creates or supersedes ADRs, harvests reusable lessons,
  closes the spec while preserving its files as immutable history, and applies
  coherence and ID-continuity gates.

The standard `specify`, `checklist`, `tasks`, and `taskstoissues` skills were
also updated to generate and carry the identifier scheme above. The legacy
bundled `.specify/workflows/speckit/workflow.yml` was removed in favor of the
skills under [`.github/skills`](.github/skills).

### Living documentation and reusable knowledge

Closeout can maintain the project's canonical product specification,
traceability map, technical overview, architecture, data model, glossary,
configuration reference, scenarios, and ADR index under `docs/`. Promotions
use semantic matching and update-in-place rules to avoid duplicate or
contradictory documentation.

The repository adds:

- [a closeout template](.specify/templates/closeout-template.md);
- [an ADR template](.specify/templates/adr-template.md); and
- [a lessons-learned template](.specify/templates/lessons-learned-template.md).

The spec, plan, task, and checklist templates were updated for the same
traceability conventions.

### Custom agents and on-demand skills

Specialized personas are retained for architecture, data science, Python
development, and security engineering. Reference and procedural content was
moved out of always-loaded agent personas and into on-demand skills:

- `azure-iac-expert`
- `maf-developer`
- `presentation`
- `requirements-spec-manager`
- `rnd`

The misplaced frontend agent was removed. Repository-specific Copilot guidance
and an agent/skill roster document the intended separation: agents define
personas, while skills hold reusable procedures and domain references.

### Two-phase delivery process

The documented working model separates specification from implementation:

1. Build and review a complete spec package on a `specs/NNNN-*` branch, then
   merge it before implementation begins.
2. Implement from the updated default branch on an `impl/NNNN-*` branch, then
   run closeout before the final merge.

See [`.specify/README.md`](.specify/README.md) for the full command sequence and
review gates.
