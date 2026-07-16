# Custom Agents

This directory holds **custom agents** — persona definitions that change *how* the
assistant behaves for a whole task (a distinct mindset, a restricted/expanded
toolset, or work that benefits from its own context window). Agents are invoked
deliberately for a task or delegated sub-task and own that job end-to-end.

Contrast with **skills** (`.github/skills/<name>/SKILL.md`), which package
*reference knowledge, a documentation policy, or a repeatable procedure* that a
generic session loads on demand when the request matches the skill's
`description`. Most day-to-day work happens in generic sessions rather than
through explicit agent delegation, so reference-heavy material belongs in a skill
where any session can reach it.

## Heuristic: agent vs. skill

If the definition is ~80% "here are facts / rules / steps to apply," it is a
**skill**. If it is "be this kind of engineer for this job," it is an **agent**.

## Current agents

| Agent | Kept because |
|-------|--------------|
| `csa` (Cloud Solution Architect) | Distinct architecture/NFR/diagramming mindset that changes output for a whole task — not a lookup. |
| `python-developer` | Core coding persona with an enforced TDD loop and `ruff`; Python is this repo's primary language. |
| `data-scientist` | Exploratory "analyze the data" loop is a genuine working mode. |
| `security-engineer` | Review / threat-modeling persona centered on the Microsoft security stack (Sentinel, Defender XDR, Entra, Purview, Security Copilot). |

## Notes on the retained agents

- **`data-scientist` and `python-developer` intentionally remain separate,
  self-contained agents.** They share *topics* (Python best practices, packaging,
  environment setup) but the content is specialized — pandas/EDA vs.
  pytest/type-safety. Because agents are loaded **independently**, each must stand
  alone; do not refactor shared sections into a cross-reference between the two.
  When you touch shared Python conventions in one, mirror the change in the other
  to avoid drift.
- **`security-engineer` overlaps the built-in `security-review` capability.**
  Prefer the built-in reviewer for diff-scoped vulnerability review; reach for this
  agent when you specifically need Microsoft-security-stack architecture and SOC
  guidance. Its generic compliance-framework material (NIST/ISO/CIS/etc.) is
  reference content — consider extracting it into a skill if it grows.

## Converted to skills

The following definitions were reference- or procedure-heavy and moved to
`.github/skills/` so generic sessions can auto-load them:

| Former agent | Now | Why it became a skill |
|--------------|-----|-----------------------|
| `azure-iac-expert` | `skills/azure-iac-expert/` | Cloud endpoints, service-parity tables, Gov/Secret limits, cross-platform shell-safety rules — reference. |
| `maf-developer` | `skills/maf-developer/` | "Check Context7 first" doc-policy plus version-pinned `agent-framework` API notes — procedure + reference. |
| `requirements-spec-manager` | `skills/requirements-spec-manager/` | A spec-audit procedure, same shape as the existing `speckit-*` skills. |
| `rnd` | `skills/rnd/` | Cross-cutting "document lessons learned" behavior, not a domain. |
| `presentation` | `skills/presentation/` | Task-triggered deck/narrative authoring (and the source file had no agent frontmatter). |

## Removed

- `frontend-ui-developer` — hard-coded to the unrelated **OVERWATCH** project
  (React/Vite/MapLibre/milsymbol/SignalR/MSAL). It does not belong in this generic
  spec-kit repository; it should live as a repo-local agent in the OVERWATCH
  repository instead. The prior content remains available in this repo's git
  history for porting.
