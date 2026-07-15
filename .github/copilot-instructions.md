# GitHub Copilot Instructions

## Before Starting Work

Consult these artifacts before beginning any task:

- **Constitution**: `.specify/memory/constitution.md` — governance principles (authoritative)
- **Product Spec**: `docs/product-spec.md` — canonical feature and capability reference (living)
- **Architecture**: `docs/architecture.md` — component map, invariants, and data flow
- **Technical overview**: `docs/technical-overview.md` — languages, frameworks, testing, infra
- **Data model**: `docs/data-model-reference.md` — stores, entities, status lifecycle
- **Configuration**: `docs/configuration.md` — environment variables and runtime settings
- **Specs** (historical): `specs/` — per-feature specification records
- **Lessons learned**: `.specify/memory/lessons-learned/` — prior findings (when present)

## Repository Layout

- `src/atlas_agent/` — the application package:
  - `api.py` (FastAPI app + endpoints), `cli.py` / `__main__.py` (CLI review),
    `rubric.py` (deterministic scoring baseline), `foundry.py` (optional Foundry/Azure OpenAI),
    `persistence.py` (Cosmos + in-memory stores), `notifications.py` (Graph sendMail),
    `static/` (workbench UI: `index.html`, `app.js`, `styles.css`).
- `tests/` — pytest suites: `test_api.py`, `test_rubric.py`, `test_persistence.py`, `test_foundry.py`.
- `infra/main.bicep` + `infra/main.bicepparam` — single-file Azure dev infrastructure (Commercial).
- `scripts/deploy-dev.ps1` — ACR cloud-build deploy; `scripts/generate_docs.py` — archived-doc `.docx` regen.
- `pyproject.toml` — package + optional deps (`.[api]`, `.[dev]`, `.[debug]`); `requirements.txt` mirrors runtime deps.
- `docs/` — living Markdown docs; `docs/archive/` — legacy docs; `docs/archive/generated/` — generated `.docx`.

## Atlas vs. Walker

**Walker** is a separate app/agent kept only as a calibrated **hero sample** in the catalog — it is
NOT a grounding source for other stories. Do not generalize Walker/GWAC details to unrelated records
(product rule: `docs/product-spec.md`, ASM-0001-05).

## Python & Tooling

- Python 3.11+. Single installable package (`atlas-agent`); deps in `pyproject.toml` optional groups.
  Develop with `pip install -e .[dev]` (or `.[api]` to run the API). Keep `requirements.txt` in sync.
- New env vars follow the `ATLAS_<INTEGRATION>_<SETTING>` convention and MUST be documented in
  `docs/configuration.md` and provisioned in `infra/main.bicep` in the same PR.
- **Linting**: run `ruff check` at the conclusion of all Python code development and address every
  finding before committing.
- **TDD workflow**: for all Python code, (1) write automated tests that fail, (2) write the minimum
  code to make them pass, then (3) refactor for optimization, clarity, and maintainability.

## Testing

No `.github/workflows` CI exists yet — this local suite is the gate (runnable with `.[dev]` only):

```powershell
python -m pytest
python -m compileall src tests scripts
node --check src\atlas_agent\static\app.js
```

Cover new API behaviour, persistence transitions, and Foundry paths with tests. (Coverage target and
test discipline: constitution Principle V.)

## Foundry Agents

The initial version of this project was built with the **microsoft-foundry** skill. That skill is
NOT currently part of this repository; if it becomes available, read it before working on or
answering questions about Foundry agents. Until then, always prefer **Microsoft-Docs MCP** for Azure
service / SDK references and **Context7 MCP** for library/API documentation — do not rely solely on
training knowledge, which may reference deprecated APIs.

## Azure Environment

Coding agents may be logged into Azure and use the live dev environment for deploys and verification.

- **Resource group**: do not assume a default. The dev infrastructure targets `rg-atlas-dev`
  (region `switzerlandnorth`); always confirm against `infra/main.bicep` / the PR before acting, and
  use `az group list` when working interactively.
- **Before any Azure task**, verify the token with `az account show`. If it has expired, do NOT
  attempt Azure operations — wrap up cleanly, document remaining work in the PR, and end the session
  so a follow-on session can continue with a fresh token.

## Agent File Access

The coding agent MAY read and edit files under `.github/` (including speckit and custom agents) when
explicitly instructed by the user; these are authored and maintained via the coding agent.

## Common Pitfalls

The product behaviour invariants (curator-gated lanes, single-lane/stale-copy cleanup,
Foundry/Assistant optionality, approved-only Assistant grounding, and export
sanitization) and the cross-cutting NFRs (scoring integrity, data-model integrity,
security/configuration) are owned by `docs/product-spec.md` — follow those FR/NFR
requirements; do not restate or contradict them here. The remaining items below are
repo-local dev hygiene:

- ❌ Polluting the ACR build context in `scripts/deploy-dev.ps1` with Office lock files or local artifacts.
- ❌ Regenerating `.docx` from living `docs/` (they are Markdown-only) — `generate_docs.py` only
  regenerates `docs/archive/generated/*.docx` from archived sources.
