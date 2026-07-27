# GitHub Copilot Instructions

## Purpose

This repository is a general-purpose collection of Spec Kit skills, prompts,
templates, scripts, and workflow guidance. Keep all reusable artifacts independent
of any particular product, customer, deployment, or application.

## Before Starting Work

Consult these artifacts before beginning any task, when present:

- **Constitution**: `.specify/memory/constitution.md` — governance principles the target project may customize
- **Product Spec**: `docs/product-spec.md` — canonical feature and capability reference
- **Architecture**: `docs/architecture.md` — component map, invariants, and data flow
- **Technical overview**: `docs/technical-overview.md` — languages, frameworks, testing, infra
- **Data model**: `docs/data-model-reference.md` — stores, entities, status lifecycle
- **Configuration**: `docs/configuration.md` — environment variables and runtime settings
- **Feature specs**: `specs/` — per-feature specification records
- **Lessons learned**: `.specify/memory/lessons-learned/` — prior findings

Follow the relevant `.github/skills/<skill>/SKILL.md`. Treat `docs/` as
project-owned living documentation only when the target project provides it. A new
Feature's specs may override, amend, or supersede any of these documents.

## Repository Layout

- `.github/skills/` — reusable skill definitions and procedures.
- `.github/agents/` — reusable agent personas and supporting documentation.
- `.specify/templates/` — templates for specs, plans, tasks, checklists, closeout
  reports, ADRs, and lessons learned.
- `.specify/scripts/` — cross-project Spec Kit workflow scripts.
- `.specify/memory/` — constitution and reusable lessons learned.
- `.specify/workflows/` and `.specify/integrations/` — workflow and integration metadata.

## Generality Requirements

- Do not hard-code product, customer, repository, or resource names, deployment
  locations, credentials, or application-specific paths; use clearly marked
  placeholders or neutral examples instead.
- Reference project-local artifacts only after checking that they exist.
- Keep technology-specific guidance inside the skill dedicated to that technology;
  keep core Spec Kit templates technology-agnostic.
- Preserve traceability identifiers and workflow conventions defined by the
  templates and skills.

## Validation

- Re-scan changed artifacts for product-specific terminology and absolute paths.
- Validate modified shell scripts and edited JSON files with the existing checks.
- Add dependencies or build requirements only when essential to a reusable workflow.
