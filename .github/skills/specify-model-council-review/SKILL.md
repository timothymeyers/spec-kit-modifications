---
name: "specify-model-council-review"
description: "Review a specification report with a three-model specialist council, apply unanimous remediations through speckit-specify, record dispositions, and escalate only disagreements."
compatibility: "Requires task-agent model selection and a spec-kit project with the speckit-specify skill"
metadata:
  author: "timothymeyers"
  source: "custom"
---

# Specify Model Council Review

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding.

## Inputs

- **Review target (required)**: The path to the Markdown report whose findings and
  recommendations the council must review, such as `clarify-report.md` or
  `spec-analysis.md`. A relative path may be resolved from the active feature
  directory.
- **Specification (optional)**: An explicit path to the specification under
  review. When omitted, use the active feature context persisted in
  `.specify/feature.json`.
- **Specialist agent (optional)**: The specialist agent type each council member
  uses. Accept `--specialist <agent-type>` or an unambiguous equivalent in the
  user's request. Default to `csa`.

If the review target is absent, does not exist, or is not a file, stop with a
concise error. If the requested specialist is unavailable, stop and list the
requested specialist; do not silently substitute another specialist.

## Goal

Obtain three independent specialist reviews of every finding and recommendation
in the target report. Automatically remediate outcomes on which all three
members agree, record every disposition in the report, and ask the user only
about outcomes where the council disagrees and a product decision is required.

## Operating Constraints

- Use the same specialist agent type for all three council members.
- Launch all three council members in parallel, with exactly these models:
  1. `gpt-5.6-sol`
  2. `claude-opus-4.8`
  3. `claude-opus-4.7`
- Council members are read-only reviewers. They MUST NOT edit files, invoke
  remediation skills, or communicate with one another.
- Give every member the same report, specification, constitution, and relevant
  supporting artifacts.
- Treat unanimity as consensus. A 2–1 result is disagreement, not consensus.
- The coordinating agent MUST NOT cast a fourth vote or silently resolve a
  disagreement.
- Apply specification changes only through `/speckit-specify` (or the
  platform-equivalent invocation of the `speckit-specify` skill).
- Preserve finding IDs from the source report. Generate stable IDs only for
  unnumbered findings.
- Do not expose the council's chain-of-thought. Retain only decisions, concise
  rationales, and proposed resolutions.

## Execution

### 1. Resolve Context

1. From the repository root, run
   `.specify/scripts/bash/check-prerequisites.sh --json --paths-only` once and
   parse `FEATURE_DIR` and `FEATURE_SPEC`. This is the same active-feature
   resolution used by the other speckit skills: the script honors an explicit
   `SPECIFY_FEATURE_DIRECTORY` override, then reads the `feature_directory`
   persisted in `.specify/feature.json`. Do not infer feature context from the
   current branch.
   - If path resolution fails and the user did not provide an explicit spec
     path, stop and instruct the user to run `/speckit-specify` or repair
     `.specify/feature.json`.
   - If the user supplied an explicit spec path, it remains usable even when no
     active feature context is configured.
2. Resolve the review target from the user input. Accept an absolute path or a
   repository-relative path; when a relative path does not exist from the
   repository root, resolve it relative to `FEATURE_DIR`.
3. Identify the specification under review in this order:
   - an explicit spec path supplied by the user;
   - `FEATURE_SPEC` from the active feature context;
   - the spec path declared in the report;
   - `spec.md` in the report's directory.
4. Stop with a concise error if one existing specification cannot be uniquely
   identified. Do not create a new feature specification.
5. Load:
   - the complete review target;
   - the identified `spec.md`;
   - `.specify/memory/constitution.md`, if present;
   - artifacts explicitly referenced by a finding when needed to judge it.
6. Build a finding inventory from the report. Record each finding's ID,
   location, severity, issue statement, recommendation, and proposed rewrite
   when present.

### 2. Launch the Council

Launch three tasks in one parallel tool call. Use the selected specialist agent
type for each task and assign one required model to each task.

Give each member an identical prompt that includes the resolved file paths and
requires it to:

1. Review every report finding and its recommendation against the specification,
   constitution, and referenced evidence.
2. Return one result per finding using this schema:

   | Field | Allowed content |
   |-------|-----------------|
   | Finding ID | Existing or coordinator-assigned stable ID |
   | Finding verdict | `VALID` or `NOT_VALID` |
   | Recommendation verdict | `ACCEPT`, `MODIFY`, or `REJECT` |
   | Resolution | Exact outcome or concise replacement requirement |
   | Rationale | Brief evidence-based explanation |
   | Decision needed | `YES` only when alternatives require a product choice |

3. Preserve the report's finding order.
4. Make each resolution concrete enough to pass directly to
   `speckit-specify`.
5. Return a concise structured result without hidden reasoning.

Wait for all three tasks to complete. If a task fails, retry that same
specialist/model once. If it still fails, stop and report the failed council
seat; do not infer consensus from fewer than three reviews.

### 3. Determine Consensus

For each finding, compare the three results:

- **Consensus — apply**: All members mark the finding `VALID` and agree on a
  materially equivalent resolution. Minor wording differences do not break
  consensus when scope, behavior, constraints, and acceptance outcome match.
- **Consensus — no change**: All members mark the finding `NOT_VALID`, or all
  agree that its recommendation must be rejected without a replacement.
- **Disagreement — decision required**: Verdicts differ, resolutions have
  materially different product outcomes, or any member marks
  `Decision needed: YES`.

When equivalence is uncertain, classify the result as disagreement. Do not use
severity or majority vote to override this rule.

### 4. Apply Unanimous Remediations

If one or more findings have **Consensus — apply**:

1. Create one consolidated remediation request containing each finding ID, the
   agreed resolution, its target spec location, and relevant constraints.
2. Invoke `speckit-specify` once in explicit refinement mode against the
   existing specification. Instruct it to:
   - update the identified `spec.md` in place;
   - address every listed finding;
   - preserve unrelated content and stable requirement identifiers;
   - avoid creating a feature directory, branch, or second spec;
   - validate the updated specification using its normal quality checks.
3. Verify that every agreed resolution appears in the resulting specification
   and that no unrelated requirement changed.
4. If verification fails, retry the remediation once with the missing finding
   IDs and exact deficiencies. If it still fails, mark those findings
   `REMEDIATION_FAILED`; surface the failure rather than claiming completion.

Do not invoke `speckit-specify` when there are no agreed specification changes.

### 5. Update the Review Report

Update the original report in place after remediation. Preserve its findings,
recommendations, and existing structure. Add or refresh a `Model Council
Disposition` section containing:

| Finding ID | Disposition | Council Resolution | Spec Evidence | Notes |
|------------|-------------|--------------------|---------------|-------|

Use these dispositions:

- `APPLIED` — unanimous resolution was verified in `spec.md`;
- `NO_CHANGE` — unanimous decision that no spec change is warranted;
- `USER_DECISION_REQUIRED` — council disagreement requires user input;
- `REMEDIATION_FAILED` — an agreed change could not be verified.

For `APPLIED`, cite the updated spec heading and requirement IDs. For
`NO_CHANGE`, record a concise shared rationale. For disagreements, summarize
the distinct options without identifying hidden reasoning or presenting a
coordinator preference.

Make the update idempotent: replace the prior council disposition for the same
finding rather than appending duplicate rows. Recalculate any report metadata
that explicitly tracks open, resolved, or disposition counts.

### 6. Respond to the User

- If every finding is `APPLIED` or `NO_CHANGE`, report completion concisely with
  the updated spec and report paths. Do not ask for confirmation.
- If disagreements exist, surface only those findings. For each, show:
  - finding ID and the decision required;
  - the distinct council-supported options and their implications;
  - a neutral prompt for the user's choice.
- Also surface any `REMEDIATION_FAILED` findings with the verification failure.
- Do not include unanimous findings, routine progress, or full council reviews
  in the decision request.

After the user resolves disagreements, apply the selected resolutions through
`speckit-specify`, verify the specification, update the corresponding report
dispositions to `APPLIED` or `NO_CHANGE`, and report completion.

## Done When

- [ ] Three independent reviews completed with the required specialist and models
- [ ] Every source finding has a unanimous or disagreement classification
- [ ] Every unanimous remediation was applied through `speckit-specify`
- [ ] The specification changes were verified
- [ ] The source report contains one current disposition per finding
- [ ] Only disagreements or remediation failures were surfaced to the user
