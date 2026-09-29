# ROADMAP.md

# Pulse — Roadmap

## 1. Purpose

This roadmap defines a practical implementation sequence for Pulse.

It is intentionally milestone-based rather than date-based. Each milestone should leave the repository in a usable and verifiable state before the next one begins.

The roadmap is not a substitute for task-level implementation plans. Agents should still break individual milestones into small, testable changes.

---

## 2. Guiding principles

The implementation order should prioritize:

1. reliable local runtime management
2. observable runtime state
3. trustworthy metrics
4. provider integrations
5. polished interaction
6. optional expansion

The project should avoid building visual polish around unstable runtime behavior.

Each milestone should preserve:

- buildability
- testability
- clear architectural boundaries
- explicit verification
- minimal scope creep

---

## 3. Milestone 0 — Repository foundation

### Goal

Create a clean native macOS project with the architectural boundaries required for later work.

### Scope

- create Xcode project
- configure macOS deployment target
- establish initial folder structure
- add Core, Application, Infrastructure, and UI boundaries
- add test target
- add basic logging categories
- add shared error model
- add CI for build and tests if desired

### Required outcomes

- application launches
- test target runs
- repository structure matches documented architecture
- no provider or runtime behavior yet
- no unnecessary third-party dependencies

### Acceptance criteria

- project builds successfully
- default tests pass
- no force unwraps or placeholder production logic
- architecture boundaries are visible in code structure

---

## 4. Milestone 1 — Local runtime configuration

### Goal

Represent and validate local runtime configuration without launching `llama.cpp`.

### Scope

Implement domain models for:

- runtime configuration
- model path
- executable path
- host
- port
- context size
- local model metadata
- runtime state

Implement validation for:

- missing executable
- invalid executable path
- missing model
- invalid model path
- invalid port
- invalid context configuration

### Required outcomes

Pulse can represent a complete `llama-server` configuration safely.

### Acceptance criteria

- valid configurations can be created
- invalid configurations produce typed errors
- configuration logic is unit tested
- no shell command strings are constructed in UI code

---

## 5. Milestone 2 — llama.cpp process lifecycle

### Goal

Start and stop `llama-server` reliably.

### Scope

Implement:

- `LlamaProcessController`
- safe process argument construction
- start
- stop
- restart
- stdout/stderr capture
- duplicate-start protection
- unexpected-exit handling
- process state transitions

### Runtime states

At minimum:

- stopped
- starting
- ready
- stopping
- error

### Required outcomes

Pulse can manage one local `llama-server` process.

### Acceptance criteria

- server can be started from a valid configuration
- server can be stopped cleanly
- repeated start commands do not create duplicate processes
- unexpected exits are surfaced
- process-related logic is covered by tests where practical

---

## 6. Milestone 3 — Runtime readiness and health

### Goal

Distinguish process launch from actual model readiness.

### Scope

Implement:

- health client
- readiness polling or event-driven readiness
- startup timeout
- readiness failure state
- endpoint validation
- restart after failed startup

### Required outcomes

Pulse only reports `ready` when the server can actually serve requests.

### Acceptance criteria

- process launch does not immediately imply readiness
- startup timeout is handled explicitly
- invalid endpoints produce actionable errors
- runtime state returns to a stable state after failure

---

## 7. Milestone 4 — GGUF model discovery

### Goal

Allow Pulse to discover and select local GGUF models.

### Scope

Implement:

- configured model directories
- `.gguf` file discovery
- stable model identity
- file size
- file path
- optional metadata parsing when reliable
- selected model persistence

### Required outcomes

The user can select a model without manually typing its full path every time.

### Acceptance criteria

- configured directories can be scanned
- invalid directories fail gracefully
- scanning does not block the main actor
- model discovery is deterministic
- selected model persists across launches

---

## 8. Milestone 5 — Local runtime metrics

### Goal

Expose trustworthy runtime and inference metrics.

### Initial metrics

- runtime state
- active model
- generation tokens/sec
- prompt tokens/sec where available
- time to first token where available
- input tokens
- output tokens
- context usage where available
- process memory usage

### Scope

Implement:

- metrics collector
- normalized metrics model
- derived metric calculation
- update cadence
- unavailable metric behavior

### Required outcomes

Metrics have explicit semantics and units.

### Acceptance criteria

- missing values are represented as unavailable
- streamed chunks are not treated as tokens without evidence
- derived metrics are tested
- UI-facing data does not depend on raw llama.cpp output formats

---

## 9. Milestone 6 — Local runtime UI

### Goal

Provide a usable interface for the complete local runtime workflow.

### Scope

Support:

- runtime status
- active model
- model selection
- start
- stop
- restart
- endpoint display
- core metrics
- configuration errors
- runtime errors

### Required outcomes

The user can manage the local runtime without opening Terminal.

### Acceptance criteria

- full runtime lifecycle is operable from Pulse
- loading and error states are explicit
- controls reflect actual runtime state
- UI does not invoke infrastructure directly

---

## 10. Milestone 7 — Compact and expanded Pulse surface

### Goal

Implement the primary Pulse interaction model.

### Scope

Implement:

- compact state
- expanded state
- show/hide behavior
- basic state transitions
- placement behavior
- stable macOS window behavior
- menu bar control point

### Required outcomes

Pulse can remain available during normal development work without behaving like a conventional always-open app window.

### Acceptance criteria

- compact state remains usable
- expansion and collapse are stable
- interaction works across normal desktop usage
- menu bar access remains available
- Pulse does not become permanently inaccessible after display changes

---

## 11. Milestone 8 — Codex provider

### Goal

Add the first cloud usage provider.

### Scope

Implement:

- Codex adapter
- provider status
- usage parsing
- reset information
- unavailable-data handling
- authentication or session access strategy if required
- normalized provider model

### Required outcomes

Pulse can display reliable Codex usage information.

### Acceptance criteria

- raw Codex data does not leak into shared UI
- missing fields remain unavailable
- provider failures do not break local runtime features
- authentication material is handled securely
- parsing has deterministic tests

---

## 12. Milestone 9 — Claude provider

### Goal

Add Claude usage monitoring.

### Scope

Implement:

- Claude adapter
- usage parsing
- reset information
- provider status
- unavailable-data handling
- secure credential/session behavior where required

### Required outcomes

Pulse can display reliable Claude usage information independently of Codex.

### Acceptance criteria

- provider is isolated behind shared abstractions
- failures are contained
- provider data is normalized
- parsing logic is covered by tests
- no sensitive data is logged

---

## 13. Milestone 10 — Overview

### Goal

Provide a unified status view across all active integrations.

### Scope

Aggregate:

- local runtime
- Codex
- Claude

The overview should surface:

- current state
- primary metric
- availability
- relevant reset information
- errors that require attention

### Required outcomes

A user can understand the overall state of their AI tooling without opening each provider view.

### Acceptance criteria

- one provider failure does not invalidate the whole overview
- unavailable providers remain clearly distinguishable from zero usage
- local runtime activity remains visible
- state aggregation has tests

---

## 14. Milestone 11 — Settings and persistence

### Goal

Make Pulse practical across repeated launches.

### Scope

Persist:

- llama-server path
- model directories
- selected model
- host
- port
- context size
- enabled providers
- interface preferences where appropriate

Securely persist:

- provider credentials or tokens if required

### Required outcomes

The user does not need to reconfigure Pulse after every launch.

### Acceptance criteria

- non-sensitive preferences survive restart
- secrets are stored securely
- invalid persisted state can recover gracefully
- migrations are unnecessary or minimal for MVP

---

## 15. Milestone 12 — Reliability pass

### Goal

Harden the application against expected operational failures.

### Test scenarios

- llama-server missing
- model missing
- invalid GGUF file
- port conflict
- process crash
- startup timeout
- provider unavailable
- malformed provider data
- expired provider session
- network failure
- display configuration change
- app relaunch while runtime is already active

### Required outcomes

Pulse fails predictably and recovers where possible.

### Acceptance criteria

- major failure modes have explicit behavior
- errors are actionable
- no silent failures remain in core runtime flows
- recovery paths are tested

---

## 16. Milestone 13 — UX and performance refinement

### Goal

Polish the experience after behavior is stable.

### Scope

- animation tuning
- metric update cadence
- layout stability
- accessibility
- keyboard support
- Reduce Motion
- multi-display behavior
- launch performance
- memory footprint
- runtime polling efficiency

### Acceptance criteria

- live metrics do not cause visible layout jitter
- UI remains responsive during model loading
- accessibility basics are covered
- Pulse does not materially affect local inference throughput

---

## 17. Milestone 14 — Release preparation

### Goal

Prepare Pulse for repeatable installation and use.

### Decisions required

- direct distribution vs Mac App Store
- signing
- notarization
- sandboxing
- update strategy
- llama.cpp installation expectations

### Scope

Potential tasks:

- app signing
- notarization
- packaging
- release notes
- versioning
- installation instructions
- runtime dependency checks
- release verification

### Acceptance criteria

- release build succeeds
- clean-machine setup is documented
- runtime dependency behavior is explicit
- no development-only secrets or paths remain

---

## 18. Post-MVP tracks

Post-MVP work should be promoted into explicit milestones only when needed.

Possible tracks:

### Additional runtimes

- MLX
- Ollama
- remote llama.cpp

### Analytics

- request history
- model benchmarks
- historical throughput
- historical memory usage
- latency percentiles

### Usage intelligence

- warning thresholds
- provider notifications
- reset reminders
- local-vs-cloud comparison

### Multi-machine

- remote runtime discovery
- runtime status across Macs
- secure remote control
- Tailscale-aware endpoints

### Developer tooling

- CLI companion
- Apple Shortcuts support
- local API for Pulse state
- automation hooks

---

## 19. Recommended implementation order

The recommended order is:

```text
Repository foundation
        ↓
Runtime configuration
        ↓
Process lifecycle
        ↓
Health / readiness
        ↓
Model discovery
        ↓
Metrics
        ↓
Local runtime UI
        ↓
Compact / expanded surface
        ↓
Codex
        ↓
Claude
        ↓
Overview
        ↓
Persistence
        ↓
Reliability
        ↓
Polish
        ↓
Release
```

This order intentionally delays cloud integrations and visual polish until the local runtime foundation is stable.

---

## 20. Definition of done for a milestone

A milestone is complete only when:

- required behavior is implemented
- relevant tests exist
- relevant checks pass
- documentation is updated when behavior or architecture changed
- no known blocker remains inside the milestone scope
- completion checkpoint requirements from `AGENTS.md` are satisfied

A milestone should not be marked complete based only on code existing.

---

## 21. Deferred decisions

The roadmap intentionally does not decide:

- final product distribution model
- App Sandbox strategy
- exact Codex integration mechanism
- exact Claude integration mechanism
- automatic llama.cpp installation
- remote access behavior
- historical metrics storage
- third-party dependency choices

These should be resolved when the corresponding milestone requires them.
