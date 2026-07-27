# GitHub Copilot Instructions

## Before Starting Work

Consult these living artifacts before beginning any task, when present:
- **Product Spec**: `docs/product-spec.md` — canonical feature and capability reference
- **Architecture**: `docs/architecture.md` — component map, invariants, and data flow
- **Technical overview**: `docs/technical-overview.md` — languages, frameworks, testing, infra
- **Data model**: `docs/data-model-reference.md` — stores, entities, status lifecycle
- **Configuration**: `docs/configuration.md` — environment variables and runtime settings
- **Lessons learned**: `.specify/memory/lessons-learned/` — prior findings
 
If implementing a new Feature, note that that Feature's specs may override, amend, or supersede any of these documents.

## Purpose

This repository is a general-purpose collection of Spec Kit skills, prompts,
templates, scripts, and workflow guidance. Keep all reusable artifacts independent
of any particular product, customer, deployment, or application.

## Before Starting Work

- Read `.specify/memory/constitution.md` when the target project has customized it.
- Consult the feature artifacts under `specs/` when they are present.
- Follow the instructions in the relevant `.github/skills/<skill>/SKILL.md`.
- Treat files under `docs/` as project-owned living documentation only when the
  target project provides them.

## Repository Layout

- `.github/skills/` — reusable skill definitions and procedures.
- `.github/agents/` — reusable agent personas and supporting documentation.
- `.specify/templates/` — templates for specifications, plans, tasks, checklists,
  closeout reports, ADRs, and lessons learned.
- `.specify/scripts/` — cross-project Spec Kit workflow scripts.
- `.specify/memory/` — constitution and reusable lessons learned.
- `.specify/workflows/` and `.specify/integrations/` — workflow and integration
  metadata.

## Generality Requirements

- Do not hard-code product names, customer names, repository names, resource names,
  deployment locations, credentials, or application-specific paths.
- Use clearly marked placeholders or neutral examples when concrete values are
  necessary.
- Do not reference project-local artifacts unless the reusable instruction first
  checks that they exist.
- Keep technology-specific guidance inside the skill dedicated to that technology;
  keep core Spec Kit templates technology-agnostic.
- Preserve traceability identifiers and workflow conventions defined by the
  templates and skills.

## Validation

- Re-scan changed artifacts for product-specific terminology and absolute paths.
- Validate modified shell scripts with the existing repository checks.
- Validate JSON files after editing them.
- Do not add dependencies or project-specific build requirements to this
  collection unless they are essential to a reusable workflow.
