# SpecKit Process — End to End (How Tim tends to work)

This document describes the full end-to-end SpecKit workflow used in this repo: a
two-phase process that first produces a complete **Spec Package**, then executes the
**Implementation**.

> **Two branches, two merges.** The `specs/NNNN-*` branch merges to `main` *before any
> code exists*. The `impl/NNNN-*` branch is then created fresh from that updated `main`.
> Do **not** write implementation code on the specs branch.

---

## Phase I: Create the "Spec Package"

Produces `spec.md`, `plan.md`, `tasks.md`, and all speckit non-implementation artifacts.

**Branch:** `specs/NNNN-my-feature` (from `main`)

| # | Actor / Agent | Action |
|---|---|---|
| 1 | **You** | Create branch `specs/NNNN-my-feature` from `main` |
| 2 | Agent | "`/speckit-specify` GH issue #NNNN" |
| 3 | Agent | "`/speckit-clarify` spec NNNN and produce a full report with all questions and recommendations in `clarify-report.md`" |
| 4 | **csa · python-dev · specialized agents** | "Review `clarify-report.md` and recommendations. Address everything you agree with; flag anything I need to review" |
| 5 | **You** | Review PR — comment, request changes, iterate until happy with `spec.md` |
| 6 | Agent | "`/speckit-plan` and `/speckit-tasks` spec NNNN" |
| 7 | Agent | "`/speckit-analyze` spec package for NNNN. Produce a `spec-analysis.md`" |
| 8 | **You** | Review PR (all artifacts + `spec-analysis.md`). Prompt: "@copilot Address all feedback" |
| 9 | **You + Agent** | Iterate until `/speckit-analyze` confirms you're **ready for `/speckit-implement`** |
| 10 | **You** | Merge the full Spec Package back to `main` |

> **The `analyze` loop (step 10) is the real gate.** Everything before it is
> throwaway-cheap; everything after is expensive. Do not leave Phase I until
> `/speckit-analyze` says you're ready and you are happy with the full spec package.

---

## Phase II: "Implement" — all the coding

**Branch:** `impl/NNNN-my-feature` (from `main`)

| # | Actor / Agent | Action |
|---|---|---|
| 1 | **You** | Create branch `impl/NNNN-my-feature` from `main` |
| 2 | Agent | "`/speckit-implement` feature NNNN" — scoped as needed: `Phase 1` · `Phases 1–2` · `Tasks T-01 to T-06` · `User Story 2, 5, 6` · `Everything` |
| 3 | **You** + Agent | Dev loop — iterate with multiple `/speckit-implement` assignments; local testing in VS Code; commit to branch |
| 4 | Agent | "`/speckit-closeout` spec NNNN" (once built, tested, and you're happy) |
| 5 | **You** | Review PR + closeout output |
| 6 | **You** | Merge to `main` — **Done!** |
