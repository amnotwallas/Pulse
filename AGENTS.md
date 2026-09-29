# AGENTS.md

This document defines the operating rules for coding agents working in the Pulse repository.

Pulse is a native macOS application for monitoring and managing local and cloud AI workflows. The initial scope includes local inference through `llama.cpp`, runtime metrics, and usage monitoring for Codex and Claude.

Agents should optimize for correctness, maintainability, clear boundaries, and verifiable behavior.

---

## 1. General principles

- Follow the existing repository structure and conventions before introducing new patterns.
- Prefer small, composable changes over broad rewrites.
- Avoid speculative abstractions that are not required by the current task.
- Do not introduce dependencies without a concrete need.
- Keep provider-specific logic isolated from shared application logic.
- Keep UI code separate from process management, networking, parsing, persistence, and metrics collection.
- Preserve native macOS behavior unless the task explicitly requires custom behavior.
- Do not silently expand scope.
- When requirements are ambiguous and the choice would materially affect architecture, security, persistence, external integrations, or user-facing behavior, stop and ask.
- For minor reversible implementation details, choose the simplest reasonable option and document the assumption.

---

## 2. Repository context

Before modifying code:

1. Read the task carefully.
2. Inspect the files directly relevant to the requested change.
3. Read any referenced documentation before implementation.
4. Check nearby tests and existing patterns before creating new ones.
5. Use repository-local instructions over generic assumptions.

Relevant documentation may include:

- `README.md`
- `docs/PRODUCT.md`
- `docs/ARCHITECTURE.md`
- `docs/DESIGN.md`
- other task-specific documentation

Do not load unrelated files or documentation without a reason.

---

## 3. Architecture boundaries

Pulse should keep core concepts independent from specific providers and runtimes.

Expected high-level boundaries:

```text
Application / UI
       |
       v
      Core
       |
       v
Provider / Runtime abstractions
       |
       +-- llama.cpp
       +-- Codex
       +-- Claude
```

### Core

Core code may define:

- shared domain models
- provider protocols
- runtime protocols
- metrics models
- normalized status models
- shared errors
- application-level state

Core code should not depend directly on provider-specific APIs, shell commands, or UI implementation details.

### Infrastructure

Infrastructure code owns external integrations such as:

- `llama.cpp`
- `llama-server`
- Codex usage sources
- Claude usage sources
- process management
- networking
- provider-specific parsing
- system metrics collection

Infrastructure implementations should conform to interfaces defined by the Core layer when appropriate.

### UI

UI code may:

- display normalized state
- trigger application actions
- observe state
- present errors and loading states

UI code must not directly:

- launch shell processes
- parse provider payloads
- read provider-specific files
- construct `llama-server` command lines
- perform provider authentication
- contain provider-specific branching that belongs in adapters or view models

---

## 4. Swift and macOS conventions

Use:

- Swift
- SwiftUI
- Swift Concurrency (`async` / `await`)
- AppKit only where SwiftUI does not provide the required macOS behavior

Prefer:

- value types where appropriate
- explicit domain models
- structured concurrency
- typed errors
- dependency injection through initializers
- protocol-based boundaries where they provide real isolation

Avoid:

- force unwraps
- force casts
- global mutable state
- blocking the main actor with process or networking work
- unnecessary singletons
- shell commands embedded in views
- large multi-purpose view models
- provider-specific logic in shared UI

Use `@MainActor` only for state that actually belongs on the main actor.

---

## 5. llama.cpp integration

Pulse manages `llama.cpp`; it does not implement model inference itself.

The runtime integration may be responsible for:

- locating or validating `llama-server`
- discovering GGUF models
- constructing runtime configuration
- starting the server
- stopping the server
- restarting the server
- monitoring process state
- reading runtime output
- collecting runtime metrics
- exposing the configured API endpoint

Do not assume a fixed installation path unless the project explicitly defines one.

Do not expose `llama-server` publicly by default.

Default local bindings should prefer loopback interfaces unless remote access is an explicit requirement.

Runtime configuration should be represented as data rather than assembled ad hoc across the codebase.

---

## 6. Provider integrations

Provider integrations should expose normalized information to the rest of Pulse.

Examples include:

- usage percentage
- usage window
- reset time
- active model
- recent activity
- provider status

Do not fabricate unavailable provider data.

If a provider does not expose a reliable metric, represent it as unavailable rather than estimating it unless the product specification explicitly defines an estimation strategy.

Provider-specific authentication, parsing, API behavior, and storage belong in the provider adapter or supporting infrastructure.

---

## 7. Metrics

Metrics should have explicit units and semantics.

Examples:

- `tokensPerSecond`
- `promptTokensPerSecond`
- `timeToFirstTokenMilliseconds`
- `inputTokens`
- `outputTokens`
- `contextTokensUsed`
- `contextTokensMaximum`
- `memoryBytes`

Do not treat streamed chunks as tokens unless the underlying runtime guarantees that relationship.

When metrics are derived rather than directly reported, document the derivation in code or tests.

Do not present false precision.

---

## 8. Security and credentials

Never commit:

- API keys
- access tokens
- cookies
- session credentials
- user identifiers that are not required fixtures
- private provider payloads
- machine-specific secrets

Use macOS Keychain or the repository's established secure storage mechanism for secrets when persistent credentials are required.

Do not log secrets.

Redact sensitive values from diagnostics and test fixtures.

Any change involving authentication, credentials, remote access, sandbox entitlements, or persistent sensitive data requires explicit attention in the completion report.

---

## 9. Process management

External processes must be managed predictably.

Process-management code should account for:

- start failures
- invalid executable paths
- invalid model paths
- duplicate starts
- graceful termination
- unexpected termination
- stale process state
- stdout/stderr capture where required
- cleanup when Pulse exits

Do not use process existence alone as proof that the runtime is ready.

When practical, confirm readiness through the runtime's health or API endpoint.

---

## 10. Error handling

Errors should be actionable.

Prefer errors that preserve:

- operation
- underlying cause
- provider or runtime
- relevant path or endpoint when safe
- recovery context

Do not silently swallow failures.

User-facing messages should remain concise while logs may contain more technical detail.

---

## 11. Testing

Changes to non-trivial behavior should include appropriate tests.

Prioritize tests for:

- parsers
- provider normalization
- runtime configuration
- process state transitions
- command construction
- metrics derivation
- error cases
- persistence behavior

Avoid tests that merely mirror implementation details.

Use deterministic fixtures.

Do not require live provider credentials for the normal test suite.

Integration tests that depend on local runtimes or external services should be clearly separated from fast unit tests.

---

## 12. Verification

Before reporting completion, run the most relevant available checks for the change.

Examples:

```bash
swift test
```

or the repository's established Xcode build/test commands.

Verification should match the changed surface area.

Do not claim a test, build, lint, or manual verification passed unless it was actually executed.

If a relevant check cannot be run, state why.

---

## 13. Scope control

Modify only files required for the task plus tests or documentation directly necessary to support the change.

Do not perform opportunistic refactors during unrelated work.

If an adjacent issue is discovered:

- fix it only if it blocks the requested task and the fix is low-risk, or
- report it as a follow-up

Any intentional deviation from the requested scope must be stated in the completion report.

---

## 14. Documentation

Update documentation when a change materially affects:

- public behavior
- architecture
- runtime requirements
- configuration
- provider behavior
- setup
- developer workflow

Use:

- `README.md` for project overview and setup
- `docs/PRODUCT.md` for product scope and behavior
- `docs/ARCHITECTURE.md` for architectural decisions and boundaries
- `docs/DESIGN.md` for visual and interaction design
- `docs/ROADMAP.md` for milestone sequencing and implementation priorities
- `AGENTS.md` for agent operating rules

Do not duplicate the same specification across multiple documents unnecessarily.

---

## 15. Changes agents should not make implicitly

Do not make these changes without explicit task scope or approval:

- change the supported macOS deployment target
- replace the primary UI framework
- replace `llama.cpp` as the initial runtime
- introduce a database
- introduce analytics or telemetry
- add automatic cloud synchronization
- expose the local inference server to the public internet
- add provider credentials to source-controlled configuration
- introduce a new package manager or build system
- perform repository-wide architectural rewrites

---

## 16. Completion checkpoint

Every completion report must state:

- **Context used:** relevant instructions, documentation, or context actually loaded beyond the current task.
- **Evidence status:** confidence or evidence classification for material claims, using the project's evidence model when one is defined.
- **Verification:** checks actually executed and their results.
- **Scope:** files changed and any intentional deviations from the requested scope.

Keep the checkpoint concise by default: one line per field when practical.
In `Context used`, list sources comma-separated. Expand any field only when
additional detail is materially necessary to explain uncertainty, verification
failures, scope deviations, or risk.

For code or configuration changes, also state what changed, checks not performed
with justification, and pending risks or follow-ups. Those additional details
are not required for read-only tasks with no modifications.

---

## 17. Completion report example

```text
Implemented llama-server process lifecycle handling with explicit runtime states.

Context used: AGENTS.md, docs/ARCHITECTURE.md
Evidence status: High confidence; behavior covered by unit tests
Verification: swift test — passed
Scope: Pulse/Infrastructure/LlamaCpp/*, PulseTests/LlamaCpp/*; no deviations

Changed: Added start, stop, readiness, and unexpected-exit handling.
Not performed: Live GGUF inference test; no model fixture is stored in the repository.
Risks / follow-ups: Readiness timeout may need tuning once tested across larger models.
```

The completion checkpoint is required even when the rest of the completion message is brief.
