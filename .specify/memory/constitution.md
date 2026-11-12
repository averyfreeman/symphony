# Symphony Elixir Constitution

**Status**: Draft, derived from the current repository for review

## Core Principles

### I. Keep the service contract aligned
The root SPEC.md defines Symphony's service contract. The Elixir implementation may extend that contract, but must not conflict with it. Update the shared specification when a behavior or configuration change alters the contract.

### II. Keep orchestration state authoritative
The Orchestrator owns mutable issue, retry, and reconciliation state. Preserve its state transitions, host assignment, and cleanup behavior. Dashboards and APIs present state; they do not become alternate authorities.

### III. Protect workspaces and remote execution
Codex turn working directories must remain inside the configured workspace root and outside the source repository. Validate and quote values crossing process or SSH boundaries. Do not pass tracker credentials to agent subprocesses.

### IV. Preserve the existing module boundaries
Core Elixir modules live under elixir/lib/symphony_elixir. Phoenix web modules live under elixir/lib/symphony_elixir_web. Keep tracker adapters behind the tracker boundary and access workflow settings through SymphonyElixir.Config. Preserve current boundaries unless a concrete need justifies a change.

### V. Make behavior testable and typed
Add focused ExUnit coverage for behavior changes and regressions under elixir/test. Public functions in elixir/lib require adjacent @spec declarations. Favor process-level tests that prove observable behavior for stateful OTP components.

### VI. Keep observability separate from correctness
The HTTP dashboard and observability API remain optional views of orchestrator state. Dashboard availability must not be required for issue polling, dispatch, retries, or cleanup. Keep logs structured and avoid recording secrets or unnecessary raw payloads.

### VII. Prefer the smallest coherent design
Give each mutable state one clear owner. Prefer existing project patterns over new abstractions or dependencies. Justify added complexity with a concrete requirement.

## Project Structure and Constraints

- The project is an Elixir/OTP service with a Phoenix LiveView dashboard under elixir/.
- Runtime settings are loaded from WORKFLOW.md front matter through Workflow, WorkflowStore, and Config.
- The current app-server client uses JSON-RPC over stdio; SSH worker execution is an existing optional extension.
- Tests and validation live in the Elixir project. Its documented gates are make all and mix specs.check, run from elixir/.
- If behavior or configuration changes, keep SPEC.md, elixir/README.md, and elixir/WORKFLOW.md synchronized where relevant.
- No additional global dependency rule is documented; preserve the existing adapter and client boundaries and justify new dependencies.

## Governance

This constitution is an evidence-based starting point for Spec Kit work. It complements, and does not override, SPEC.md, elixir/AGENTS.md, or direct user instructions. Amend it when project conventions change and record the reason.

**Version**: 0.1.0 | **Status**: Draft | **Created**: 2026-11-11
