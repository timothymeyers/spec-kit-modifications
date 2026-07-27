---
name: "specify-model-council-review"
description: "Review a specification report with a three-model specialist council, synthesize its recommendations, route unanimous remediations to the appropriate speckit skill, and escalate only disagreements or material risks."
compatibility: "Requires task-agent model selection and a spec-kit project with the speckit-specify, speckit-plan, and speckit-tasks skills"
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
in the target report. Synthesize the completed council's findings into explicit
recommendations and narrowly scoped human-review items. Automatically remediate
low-risk outcomes on which all three members agree, record every disposition in
the report, and ask the user only about substantive disagreements or material
risks that require a human decision.

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
- Minimize human escalation. Do not escalate minor wording differences,
  implementation details already constrained by the specification, or a
  finding's severity label by itself. Treat an outcome as materially high risk
  only when applying it could affect security, privacy, compliance, irreversible
  data handling, or a cross-cutting architectural or product commitment that
  the reviewed artifacts do not already authorize.
- Apply changes only through the skill that owns the affected artifact:
  - use `/speckit-specify` for `spec.md`;
  - use `/speckit-plan` for `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, and `contracts/`;
  - use `/speckit-tasks` for `tasks.md`.
  Platform-equivalent skill invocations are acceptable.
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
   - `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, `contracts/`, and `tasks.md` when they exist or
     are referenced by a finding;
   - other artifacts explicitly referenced by a finding when needed to judge it.
6. Build a finding inventory from the report. Record each finding's ID,
   location, affected artifact, severity, issue statement, recommendation, and
   proposed rewrite when present. Classify each affected artifact as
   specification (`spec.md`), plan (`plan.md`, `research.md`, `data-model.md`,
   `quickstart.md`, or `contracts/`), or task list (`tasks.md`). If the affected artifact is ambiguous,
   treat the finding as requiring a user decision rather than guessing.

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
   | Affected artifact | Exact artifact path from the finding inventory |
   | Finding verdict | `VALID` or `NOT_VALID` |
   | Recommendation verdict | `ACCEPT`, `MODIFY`, or `REJECT` |
   | Resolution | Exact outcome or concise replacement requirement |
   | Rationale | Brief evidence-based explanation |
   | Decision needed | `YES` only when alternatives require a product choice |

3. Preserve the report's finding order.
4. Make each resolution concrete enough to pass directly to
   the skill that owns the affected artifact.
5. Return a concise structured result without hidden reasoning.

Wait for all three tasks to complete. If a task fails, retry that same
specialist/model once. If it still fails, stop and report the failed council
seat; do not infer consensus from fewer than three reviews.

### 3. Synthesize the Council Review

After all three council reviews are complete, compare their results for every
finding and synthesize one coordinator-owned disposition:

- **Consensus — apply**: All members mark the finding `VALID` and agree on a
  materially equivalent resolution. Minor wording differences do not break
  consensus when scope, behavior, constraints, and acceptance outcome match.
- **Consensus — no change**: All members mark the finding `NOT_VALID`, or all
  agree that its recommendation must be rejected without a replacement.
- **Disagreement — decision required**: Verdicts differ, resolutions have
  materially different product outcomes, or any member marks
  `Decision needed: YES`.
- **High risk — human review required**: The council agrees on an outcome, but
  applying it meets the material high-risk definition in the operating
  constraints.

When equivalence is uncertain, classify the result as disagreement. Do not use
severity or majority vote to override this rule.

Immediately add or refresh these sections in the original report:

#### Council Review Recommendations

Record every low-risk unanimous disposition, including both changes to apply and
decisions that no change is warranted:

| Finding ID | Recommendation | Council Resolution | Shared Rationale |
|------------|----------------|--------------------|------------------|

Use `APPLY` or `NO_CHANGE` for `Recommendation`. Consolidate materially
equivalent wording into a concise resolution without introducing a fourth
opinion.

#### Needs Human Review

Record only substantive disagreements and materially high-risk outcomes:

| Finding ID | Escalation Reason | Council-Supported Options | Decision Needed |
|------------|-------------------|---------------------------|-----------------|

State whether the reason is `DISAGREEMENT` or `HIGH_RISK`, summarize the
distinct supported options and implications neutrally, and identify the
specific human decision. Do not duplicate a finding across both sections.

Make both sections idempotent: replace prior synthesis rows for the same finding
and remove stale rows when its classification changes. Preserve the rest of the
report. The synthesis MUST be written before any remediation begins.

### 4. Apply Unanimous Remediations

If one or more findings have low-risk **Consensus — apply**:

1. Group findings by owning skill and create one consolidated remediation
   request per skill. Each request must contain every applicable finding ID, the
   agreed resolution, target artifact and location, and relevant constraints.
2. Invoke each required skill once in this dependency order:
   1. `speckit-specify` for findings targeting `spec.md`;
   2. `speckit-plan` for findings targeting `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, or `contracts/`;
   3. `speckit-tasks` for findings targeting `tasks.md`.
3. Invoke each skill in explicit refinement mode against the existing
   artifacts. Instruct it to:
   - update only the artifacts owned by that skill in place;
   - address every finding assigned to that skill;
   - preserve unrelated content and stable identifiers;
   - avoid creating a feature directory, branch, or duplicate artifact;
   - validate the updated artifacts using its normal quality checks.
4. Verify that every agreed resolution appears in its target artifact and that
   no unrelated content changed.
5. If verification fails, retry the responsible skill once with the missing
   finding IDs and exact deficiencies. If it still fails, mark those findings
   `REMEDIATION_FAILED`; surface the failure rather than claiming completion.

Do not invoke a remediation skill when no agreed changes target artifacts owned
by that skill. Do not remediate findings listed under `Needs Human Review`
until the user provides a decision.

### 5. Update the Review Report

Update the original report in place after remediation. Preserve its findings,
recommendations, and existing structure. Add or refresh a `Model Council
Disposition` section containing:

| Finding ID | Disposition | Council Resolution | Artifact Evidence | Notes |
|------------|-------------|--------------------|-------------------|-------|

Use these dispositions:

- `APPLIED` — unanimous resolution was verified in its target artifact;
- `NO_CHANGE` — unanimous decision that no artifact change is warranted;
- `HUMAN_REVIEW_REQUIRED` — council disagreement or material risk requires
  human input;
- `REMEDIATION_FAILED` — an agreed change could not be verified.

For `APPLIED`, cite the updated artifact path, heading, and stable identifiers
when present. For `NO_CHANGE`, record a concise shared rationale. For
`HUMAN_REVIEW_REQUIRED`, summarize the escalation reason and distinct options
without identifying hidden reasoning or presenting a coordinator preference.

Make the update idempotent: replace the prior council disposition for the same
finding rather than appending duplicate rows. Recalculate any report metadata
that explicitly tracks open, resolved, or disposition counts.

### 6. Respond to the User

- If every finding is `APPLIED` or `NO_CHANGE`, report completion concisely with
  the updated artifact and report paths. Do not ask for confirmation.
- If human-review items exist, surface only those findings. For each, show:
  - finding ID and the decision required;
  - the distinct council-supported options and their implications;
  - whether escalation is due to disagreement or material risk;
  - a neutral prompt for the user's choice.
- Also surface any `REMEDIATION_FAILED` findings with the verification failure.
- Do not include unanimous findings, routine progress, or full council reviews
  in the decision request.

After the user resolves human-review items, apply each selected resolution
through the skill that owns its affected artifact, verify the artifact, remove
the finding from `Needs Human Review`, add its resolved outcome to `Council
Review Recommendations`, update the corresponding report disposition to
`APPLIED` or `NO_CHANGE`, and report completion.

## Done When

- [ ] Three independent reviews completed with the required specialist and models
- [ ] Every source finding has a consensus, disagreement, or high-risk classification
- [ ] `Council Review Recommendations` contains every low-risk unanimous disposition
- [ ] `Needs Human Review` contains only substantive disagreements and material risks
- [ ] Every unanimous remediation was applied through its artifact-owning skill
- [ ] All changed artifacts were verified
- [ ] The source report contains one current disposition per finding
- [ ] Only human-review items or remediation failures were surfaced to the user
