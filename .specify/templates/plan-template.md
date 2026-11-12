# Implementation Plan: [FEATURE]

**Branch**: [Branch name, or “Not created during specification”] | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from /specs/[###-feature-name]/spec.md

## Summary

[Summarize user value, existing behavior, and the proposed approach.]

## Technical Context

**Language/Version**: Elixir ~> 1.19; OTP 28 as documented in elixir/AGENTS.md

**Primary Dependencies**: Existing Elixir, Phoenix, and Phoenix LiveView dependencies; justify any addition

**Storage**: WORKFLOW.md front matter for workflow settings; identify the selected persistent store for new state

**Testing**: ExUnit under elixir/test; targeted mix test from elixir/, followed by make all and mix specs.check when authorized

**Target Platform**: Linux host running Symphony; remote hosts are reached through the configured network and OpenSSH identity

**Project Type**: Elixir/OTP service with Phoenix LiveView observability UI

**Performance Goals**: Preserve the configured dashboard refresh interval and keep remote connection work off the LiveView request path

**Constraints**: Preserve the root SPEC.md contract, single-owner orchestrator state, workspace safety, and existing worker-host behavior

**Scale/Scope**: [State the number of services, hosts, and user journeys in this feature]

## Constitution Check

- [ ] Existing service contract remains aligned with SPEC.md
- [ ] Mutable state has one clear owner
- [ ] Workspace and process-boundary safety is preserved
- [ ] Public functions have @spec declarations
- [ ] Focused ExUnit coverage and relevant documentation are planned

## Project Structure

### Feature Documentation

specs/[###-feature-name]/
- spec.md
- plan.md
- tasks.md
- optional research.md, data-model.md, contracts/, and quickstart.md when they add needed detail

### Existing Source Boundaries

- Core service: elixir/lib/symphony_elixir/
- Phoenix UI and HTTP interface: elixir/lib/symphony_elixir_web/
- Tests: elixir/test/
- Shared behavior contract: SPEC.md
- Elixir usage and workflow documentation: elixir/README.md and elixir/WORKFLOW.md

**Structure Decision**: Extend the existing Elixir/Phoenix application and its current module boundaries. Do not create a second service or a separate frontend unless the feature specification establishes a concrete need.

## Complexity Tracking

Record each constitution exception, why it is needed, and the simpler alternative considered. Use “None” when no exception is required.
