# PRODUCT.md

# Pulse — Product Specification

## 1. Product summary

Pulse is a native macOS application that provides a unified control and monitoring surface for local and cloud AI workflows.

The initial product scope focuses on three areas:

- local inference through `llama.cpp`
- runtime and performance metrics
- usage monitoring for Codex and Claude

Pulse is intended for developers who actively use multiple AI tools and want fast visibility into their current runtime state, model performance, provider limits, and reset windows without constantly switching between terminals, dashboards, and web applications.

---

## 2. Product goals

Pulse should:

- provide a single place to observe local and cloud AI activity
- make local LLM runtimes easy to start, stop, inspect, and switch
- expose useful inference metrics without requiring terminal inspection
- surface provider usage limits and reset windows clearly
- remain lightweight enough to stay available throughout a development session
- feel native to macOS
- preserve clear separation between providers and runtimes
- avoid requiring a cloud account for local-only functionality

---

## 3. Non-goals

The initial version of Pulse is not intended to:

- implement LLM inference directly
- replace full AI coding agents
- replace provider applications or dashboards completely
- provide a chat interface
- act as a model training or fine-tuning tool
- synchronize data across devices
- provide team or organization management
- provide billing or subscription management
- expose local inference publicly over the internet by default
- support every local AI runtime in the first release

These may be considered later only if they align with the core product direction.

---

## 4. Target user

The primary user is a developer who:

- uses macOS on Apple Silicon
- works with local and cloud AI tools
- runs local models for coding or experimentation
- uses Codex, Claude, or similar services regularly
- cares about model latency, throughput, context usage, and resource consumption
- wants a compact control surface rather than another full dashboard

The initial release may be optimized for a single-user developer workflow.

---

## 5. Core product areas

### 5.1 Overview

The overview provides normalized high-level state across configured integrations.

It should answer questions such as:

- Is the local runtime active?
- Which model is currently loaded?
- Is inference happening now?
- How fast is the current model generating?
- How much Codex usage remains?
- When does a usage window reset?
- How much Claude usage remains?
- Are any providers unavailable or reporting errors?

The overview should favor current actionable state over historical detail.

---

### 5.2 Local runtime

The first supported local runtime is `llama.cpp`.

Pulse should manage `llama-server` as an external process.

The local runtime feature should support:

- locating or configuring a `llama-server` executable
- selecting a GGUF model
- starting the runtime
- stopping the runtime
- restarting the runtime
- reporting runtime state
- detecting unexpected termination
- exposing the configured API endpoint
- representing startup, ready, active, error, and stopped states
- configuring supported runtime parameters

Initial runtime configuration may include:

- model path
- context size
- host
- port
- GPU offload behavior where applicable
- additional supported `llama-server` arguments

Pulse should not assume that a successfully launched process is immediately ready to serve requests.

---

### 5.3 Local model discovery

Pulse should provide a way to identify GGUF models available to the user.

The initial version may support configured model directories rather than automatically searching the entire filesystem.

For each discovered model, Pulse may expose:

- file name
- file path
- file size
- model metadata when available
- quantization when reliably detectable

Pulse must not infer metadata that cannot be determined reliably.

---

### 5.4 Runtime metrics

Pulse should expose runtime metrics with clear semantics and units.

Initial metrics should include, where available:

- generation tokens per second
- prompt evaluation tokens per second
- time to first token
- input token count
- output token count
- context tokens used
- maximum configured context
- process memory usage
- runtime status
- active model

Metrics may come directly from `llama.cpp` or be derived from reliable timing and token data.

Pulse must not assume that one streamed response chunk equals one token.

Derived metrics should be calculated consistently and documented.

---

### 5.5 Codex usage

Pulse should provide a normalized view of Codex usage information when a reliable source is available.

Possible data includes:

- current usage percentage
- active usage window
- remaining capacity
- reset time
- model-specific availability
- relevant secondary limits

The exact supported fields depend on what Codex exposes reliably.

Unavailable information should be represented as unavailable rather than guessed.

The Codex integration must be isolated from shared product logic so that changes in upstream provider behavior do not require redesigning unrelated parts of Pulse.

---

### 5.6 Claude usage

Pulse should provide a normalized view of Claude usage information when a reliable source is available.

Possible data includes:

- current usage
- session or rolling window usage
- reset time
- model-specific limits
- provider status

As with Codex, Pulse should only show values supported by reliable evidence.

No usage percentage should be synthesized unless the derivation is explicit and supported by the product specification.

---

## 6. Provider state model

Provider and runtime integrations should expose normalized states where practical.

Example states:

- `unconfigured`
- `stopped`
- `starting`
- `ready`
- `active`
- `degraded`
- `unavailable`
- `error`

Not every provider must support every state.

The product should preserve provider-specific detail internally while exposing consistent state to the rest of the application.

---

## 7. Runtime lifecycle

The initial local runtime lifecycle should support the following conceptual flow:

```text
Unconfigured
     |
     v
Configured
     |
     v
Stopped
     |
     | start
     v
Starting
     |
     +---- failure ----> Error
     |
     v
Ready
     |
     | request activity
     v
Active
     |
     v
Ready
     |
     | stop
     v
Stopped
```

Unexpected process termination should result in a state that clearly communicates failure rather than silently returning to `stopped`.

---

## 8. Error behavior

Pulse should distinguish between:

- configuration errors
- runtime launch failures
- model path errors
- port conflicts
- readiness failures
- provider authentication failures
- provider parsing failures
- provider network failures
- metrics unavailable
- unsupported data

Errors shown to the user should answer, where possible:

1. what failed
2. what component failed
3. whether Pulse can recover automatically
4. what action the user can take next

Technical details may be available separately for debugging.

---

## 9. Data and persistence

Pulse may persist local configuration required for operation.

Examples:

- selected model
- model directories
- runtime executable path
- port
- context size
- enabled providers
- user preferences

Sensitive credentials must not be stored in plain text.

When credentials must be persisted, Pulse should use macOS Keychain or another explicitly approved secure mechanism.

The MVP does not require persistent historical analytics.

---

## 10. Networking

Local runtime networking should default to safe local behavior.

Default behavior:

- bind `llama-server` to a loopback interface
- avoid public exposure
- make remote access an explicit configuration choice

Support for LAN or Tailscale access may be added, but must not weaken the default local security posture.

Pulse should make the effective endpoint visible to the user.

---

## 11. Performance expectations

Pulse should remain lightweight relative to the model runtime it manages.

The application should:

- avoid blocking the main thread with runtime or network work
- avoid aggressive polling when event-driven updates are available
- avoid retaining unnecessary provider payloads
- avoid materially affecting local inference throughput
- remain responsive while models are loading or generating

No strict numeric performance targets are defined until real hardware measurements are available.

---

## 12. Reliability expectations

Pulse should correctly handle:

- runtime already running
- runtime already stopped
- invalid executable path
- invalid model path
- model load failure
- port already in use
- process crash
- provider unavailable
- malformed provider response
- missing provider credentials
- unavailable metrics
- temporary network failure

Recovery behavior should be predictable and observable.

---

## 13. Privacy

Pulse should follow a local-first privacy model.

The application should not collect analytics, telemetry, prompt content, source code, provider payloads, or model inputs unless that behavior is explicitly added and documented later.

Local runtime traffic should remain local by default.

Logs should avoid storing:

- prompts
- generated responses
- credentials
- access tokens
- session cookies
- sensitive provider payloads

unless required by an explicitly enabled debugging mode with appropriate disclosure.

---

## 14. MVP definition

The MVP is complete when a user can:

1. launch Pulse on macOS
2. configure a local `llama-server` runtime
3. select a GGUF model
4. start and stop the local runtime
5. see whether the runtime is ready
6. see the local API endpoint
7. observe core inference metrics
8. view available Codex usage information
9. view available Claude usage information
10. understand integration failures without using terminal logs for normal operation

Core local metrics for the MVP:

- runtime state
- active model
- generation tokens/sec
- prompt evaluation tokens/sec when available
- TTFT when available
- input/output tokens when available
- context usage when available
- memory usage

---

## 15. Post-MVP possibilities

Potential future work includes:

- MLX backend
- Ollama backend
- multiple simultaneous local runtimes
- model download management
- historical performance analytics
- per-model benchmarking
- usage notifications
- configurable warning thresholds
- cost tracking
- estimated local-vs-cloud savings
- remote runtime management
- multiple machines
- Apple Shortcuts integration
- CLI companion
- launch-at-login runtime policies
- provider plugin architecture

These are not part of the initial implementation unless promoted into scope by a future product decision.

---

## 16. Product principles

### Accurate over complete

If a metric cannot be obtained reliably, show it as unavailable.

### Local-first

Local inference should work independently of cloud integrations.

### Observable

The application should make runtime state and performance easy to inspect.

### Provider-independent

No cloud provider should define the architecture of the overall product.

### Safe defaults

Runtime and credential handling should favor secure local behavior.

### Focused

Pulse should solve monitoring and runtime management well before expanding into unrelated AI tooling.

---

## 17. Open product questions

The following decisions are intentionally not fixed by this document:

- exact visual design and interaction model
- distribution strategy
- Mac App Store vs direct distribution
- sandboxing strategy
- automatic installation or bundling of `llama.cpp`
- exact Codex data source
- exact Claude data source
- whether remote runtime access is supported in the MVP
- whether historical metrics belong in the first public release

These should be resolved in the appropriate design, architecture, or implementation documentation rather than assumed during development.
