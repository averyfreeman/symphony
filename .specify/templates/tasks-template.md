---
description: "Task list template for the Symphony Elixir project"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from specs/[###-feature-name]/

**Prerequisites**: plan.md and spec.md; add research.md, data-model.md, or contracts/ only when they resolve a concrete design need

**Tests**: Include focused ExUnit tasks for behavior changes and regressions. Test tasks may be omitted for documentation-only work.

**Project quality gates**: Run targeted mix test from elixir/ while iterating; run make all and mix specs.check from elixir/ before handoff when authorized.

**Organization**: Group implementation tasks by user story and keep each story independently verifiable.

## Format: [ID] [P?] [Story] Description

- [P] marks work that can run in parallel without file or state conflicts.
- [Story] identifies the user story, such as US1 or US2.
- Include exact repository paths in task descriptions.

## Path Conventions

- Core service modules: elixir/lib/symphony_elixir/
- Phoenix web modules: elixir/lib/symphony_elixir_web/
- ExUnit tests: elixir/test/
- Shared contract: SPEC.md
- Elixir implementation documentation: elixir/README.md and elixir/WORKFLOW.md

## Task Phases

Organize work into:
1. Shared setup and types/configuration
2. Foundational process ownership, interfaces, and error handling
3. User-story implementation with focused ExUnit tests
4. Documentation and cross-cutting quality gates

Include actual test paths and commands. Do not create a new Mix project or generic frontend/backend directory structure for this repository.
