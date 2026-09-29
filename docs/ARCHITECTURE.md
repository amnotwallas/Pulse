# ARCHITECTURE.md

# Pulse — Architecture

## 1. Purpose

This document defines the initial software architecture for Pulse.

Pulse is a native macOS application that monitors and manages local and cloud AI workflows. The first release focuses on:

- local inference through `llama.cpp`
- runtime metrics
- Codex usage monitoring
- Claude usage monitoring

The architecture should preserve strong boundaries between:

- UI
- application state
- core domain models
- runtime management
- provider integrations
- system metrics
- persistence

The design should remain simple enough for an MVP while avoiding provider-specific coupling that would make future integrations difficult.

---

## 2. Architectural goals

Pulse should be:

- modular
- testable
- provider-independent
- local-first
- observable
- concurrency-safe
- resilient to provider failures
- lightweight relative to the inference runtime it manages

The architecture should support replacing or adding providers without requiring broad UI rewrites.

---

## 3. High-level architecture

```text
+--------------------------------------------------+
|                    Pulse App                     |
+--------------------------------------------------+
                     |
                     v
+--------------------------------------------------+
|               Application Layer                  |
|                                                  |
|  - App state                                     |
|  - User actions                                  |
|  - Use cases                                     |
|  - Coordination                                  |
+--------------------------------------------------+
                     |
                     v
+--------------------------------------------------+
|                    Core                          |
|                                                  |
|  - Domain models                                 |
|  - Protocols                                     |
|  - Normalized provider state                     |
|  - Runtime configuration                         |
|  - Metrics models                                |
+--------------------------------------------------+
                     |
         +-----------+-----------+
         |                       |
         v                       v
+----------------------+  +------------------------+
| Infrastructure       |  | System Integration     |
|                      |  |                        |
| - llama.cpp          |  | - process metrics      |
| - Codex adapter      |  | - memory               |
| - Claude adapter     |  | - filesystem           |
| - HTTP               |  | - Keychain             |
| - parsing            |  | - launch services      |
+----------------------+  +------------------------+
                     |
                     v
+--------------------------------------------------+
|                  External Systems                |
|                                                  |
|  llama-server   Codex   Claude   macOS           |
+--------------------------------------------------+
```

---

## 4. Layers

## 4.1 UI layer

The UI layer is implemented primarily in SwiftUI.

AppKit may be used only where SwiftUI does not provide sufficient macOS control.

Responsibilities:

- render normalized state
- display runtime/provider status
- present errors
- trigger user actions
- observe application state

The UI must not directly:

- spawn `llama-server`
- construct process arguments
- parse provider-specific payloads
- perform provider authentication
- read provider-specific configuration files
- access Keychain directly
- poll external systems directly

Views should depend on application-facing models and actions, not infrastructure types.

---

## 4.2 Application layer

The application layer coordinates user intent and system behavior.

Responsibilities:

- start runtime
- stop runtime
- restart runtime
- switch models
- refresh provider usage
- coordinate state transitions
- aggregate overview state
- translate infrastructure failures into application errors

Examples of application services or use cases:

```text
StartRuntime
StopRuntime
RestartRuntime
SelectModel
RefreshProviderUsage
RefreshOverview
```

This layer should remain thin.

Business rules that are reusable and independent of orchestration belong in Core.

---

## 4.3 Core layer

The Core layer contains provider-independent domain concepts.

It must not depend on:

- SwiftUI
- AppKit
- shell commands
- provider-specific HTTP schemas
- filesystem layout assumptions
- credentials storage implementation

Core may define:

- `ProviderID`
- `ProviderStatus`
- `ProviderMetrics`
- `RuntimeState`
- `RuntimeConfiguration`
- `LocalModel`
- `RuntimeMetrics`
- `ProviderUsage`
- shared errors
- capability protocols

Example protocol:

```swift
protocol AIProvider: Sendable {
    var id: ProviderID { get }

    func status() async throws -> ProviderStatus
    func metrics() async throws -> ProviderMetrics
}
```

Local runtimes may expose additional capabilities:

```swift
protocol LocalRuntimeProvider: AIProvider {
    func start(configuration: RuntimeConfiguration) async throws
    func stop() async throws
    func restart(configuration: RuntimeConfiguration) async throws
    func availableModels() async throws -> [LocalModel]
}
```

The exact API may evolve, but the architectural rule is stable:

> provider-specific details should not leak into shared UI or application logic.

---

## 5. Provider adapters

Each external provider should have its own adapter.

Initial adapters:

```text
LlamaCppProvider
CodexProvider
ClaudeProvider
```

Each adapter is responsible for converting provider-specific data into normalized Core models.

Example:

```text
Codex response
      |
      v
CodexProvider
      |
      v
ProviderUsage
```

Shared application code should not understand the raw Codex schema.

---

## 6. llama.cpp architecture

The `llama.cpp` integration is the most stateful infrastructure component.

It should be split into focused responsibilities.

Suggested components:

```text
LlamaCppProvider
|
+-- LlamaProcessController
|
+-- LlamaConfigurationBuilder
|
+-- LlamaHealthClient
|
+-- LlamaMetricsCollector
|
+-- GGUFModelRepository
```

### 6.1 LlamaProcessController

Responsible for:

- launching `llama-server`
- tracking process lifecycle
- graceful termination
- detecting unexpected exit
- capturing stdout/stderr where needed
- preventing duplicate starts

It should not parse model metadata or provider usage.

---

### 6.2 LlamaConfigurationBuilder

Responsible for translating `RuntimeConfiguration` into safe `llama-server` arguments.

Example input:

```swift
RuntimeConfiguration(
    modelPath: ...,
    contextSize: 32768,
    host: "127.0.0.1",
    port: 8080
)
```

Example output:

```text
-m /path/model.gguf
--host 127.0.0.1
--port 8080
-c 32768
```

Command construction should be testable without launching a process.

---

### 6.3 LlamaHealthClient

Responsible for determining whether the server is actually ready.

Process state alone is insufficient.

The health client should:

- query the configured local endpoint
- distinguish startup from ready
- enforce a timeout
- report connection and readiness failures

---

### 6.4 LlamaMetricsCollector

Responsible for collecting or deriving runtime metrics.

Possible sources:

- llama.cpp response metadata
- streaming events
- runtime logs
- local timing
- process metrics

Derived metrics must have explicit semantics.

The metrics collector should not assume streamed chunks equal tokens.

---

### 6.5 GGUFModelRepository

Responsible for discovering configured GGUF model files.

Initial behavior may be limited to user-selected directories.

Responsibilities may include:

- enumerate `.gguf` files
- resolve paths
- file size
- stable model identifiers
- metadata where reliably available

Filesystem scanning should not block the main actor.

---

## 7. Runtime state machine

The local runtime should have an explicit state model.

Suggested state:

```swift
enum RuntimeState: Equatable, Sendable {
    case unconfigured
    case stopped
    case starting
    case ready
    case active
    case stopping
    case error(RuntimeFailure)
}
```

Possible transitions:

```text
unconfigured
     |
     v
stopped
     |
   start
     v
starting
  |     |
  |     +---- failure ---> error
  |
  v
ready
  |
 request
  v
active
  |
  v
ready
  |
  stop
  v
stopping
  |
  v
stopped
```

Illegal transitions should be rejected rather than silently accepted.

Examples:

- starting while already starting
- starting while already ready
- stopping while already stopped

The exact policy for idempotent commands should be defined during implementation.

---

## 8. Concurrency model

Pulse should use Swift structured concurrency.

Preferred tools:

- `async` / `await`
- `Task`
- actors where state needs serialized access
- `AsyncSequence` where continuous updates are useful

Avoid:

- unmanaged background threads
- ad hoc callback pyramids
- shared mutable state without isolation

The runtime controller is a strong candidate for actor isolation.

Example:

```swift
actor RuntimeController {
    private var state: RuntimeState
    private var process: Process?
}
```

UI-facing state should be bridged to the main actor.

---

## 9. State propagation

Infrastructure should not directly mutate UI state.

Preferred flow:

```text
Infrastructure
     |
     v
Application service
     |
     v
App state / observable model
     |
     v
UI
```

An observable application state may expose:

- runtime state
- selected model
- local metrics
- Codex usage
- Claude usage
- current errors

The exact observation mechanism should follow current SwiftUI conventions.

---

## 10. Error model

Errors should preserve enough information for diagnostics while remaining safe for user presentation.

Suggested structure:

```swift
struct RuntimeFailure: Error, Sendable {
    let operation: RuntimeOperation
    let kind: RuntimeFailureKind
    let message: String
}
```

Potential failure kinds:

```text
executableNotFound
invalidModelPath
portUnavailable
launchFailed
readinessTimeout
unexpectedExit
metricsUnavailable
providerUnavailable
authenticationFailed
invalidResponse
```

Do not expose secrets or raw credentials in errors.

---

## 11. Persistence

The MVP should persist only required configuration.

Possible persisted values:

- runtime executable path
- model directories
- selected model
- context size
- host
- port
- enabled providers
- user preferences

Simple non-sensitive preferences may use `UserDefaults`.

Sensitive values must use Keychain or another explicitly approved secure store.

Do not introduce a database for the MVP unless a real requirement appears.

---

## 12. Credentials

Provider credentials must be isolated behind a credential-store abstraction.

Conceptually:

```swift
protocol CredentialStore {
    func readCredential(for provider: ProviderID) throws -> Data?
    func writeCredential(_ data: Data, for provider: ProviderID) throws
    func deleteCredential(for provider: ProviderID) throws
}
```

The macOS implementation should use Keychain.

Provider adapters should not know the underlying Keychain API.

---

## 13. Networking

Networking should be isolated behind infrastructure clients.

Possible clients:

```text
LlamaHealthClient
CodexClient
ClaudeClient
```

Networking responsibilities:

- request creation
- response decoding
- timeout handling
- transport errors
- provider-specific headers

Application code should not construct raw HTTP requests.

Default local runtime binding:

```text
127.0.0.1
```

Remote access must be explicit.

---

## 14. Metrics architecture

Metrics should flow through normalized models.

Example:

```swift
struct RuntimeMetrics: Sendable {
    let generationTokensPerSecond: Double?
    let promptTokensPerSecond: Double?
    let timeToFirstTokenMilliseconds: Double?
    let inputTokens: Int?
    let outputTokens: Int?
    let contextTokensUsed: Int?
    let contextTokensMaximum: Int?
    let memoryBytes: UInt64?
}
```

Unknown values should remain `nil`.

Do not replace missing values with zero unless zero is semantically correct.

---

## 15. Overview aggregation

The overview should not query infrastructure directly.

A dedicated application component may aggregate provider state.

Example:

```text
OverviewService
|
+-- LocalRuntimeProvider
+-- CodexProvider
+-- ClaudeProvider
```

It returns normalized overview data.

This allows one provider to fail without making the entire overview unavailable.

---

## 16. Dependency direction

Dependencies should point inward.

```text
UI
 |
 v
Application
 |
 v
Core
 ^
 |
Infrastructure
```

Core must not import infrastructure.

Infrastructure may implement Core protocols.

This is the most important architecture rule in the project.

---

## 17. Suggested repository structure

```text
Pulse/
|
+-- App/
|   +-- PulseApp.swift
|   +-- AppState.swift
|
+-- UI/
|   +-- Overview/
|   +-- Local/
|   +-- Codex/
|   +-- Claude/
|   +-- Shared/
|
+-- Core/
|   +-- Models/
|   +-- Providers/
|   +-- Runtime/
|   +-- Metrics/
|   +-- Errors/
|
+-- Application/
|   +-- Runtime/
|   +-- Providers/
|   +-- Overview/
|
+-- Infrastructure/
|   +-- LlamaCpp/
|   |   +-- LlamaProcessController.swift
|   |   +-- LlamaConfigurationBuilder.swift
|   |   +-- LlamaHealthClient.swift
|   |   +-- LlamaMetricsCollector.swift
|   |   +-- GGUFModelRepository.swift
|   |
|   +-- Codex/
|   +-- Claude/
|   +-- Keychain/
|   +-- System/
|
+-- Resources/
```

Tests should mirror architectural boundaries.

---

## 18. Testing strategy

### Core tests

Test:

- state transitions
- domain validation
- normalized models
- derived metric logic

### Application tests

Test:

- use case orchestration
- error propagation
- provider failure isolation
- overview aggregation

Use fake providers.

### Infrastructure tests

Test:

- command construction
- path validation
- response parsing
- provider normalization
- process event handling

Avoid requiring real credentials.

### Integration tests

Optional integration tests may validate:

- live `llama-server`
- real GGUF model loading
- readiness behavior
- local API compatibility

These should be separate from the default fast test suite.

---

## 19. Logging

Logging should use structured categories.

Suggested categories:

```text
app
runtime
llama
provider.codex
provider.claude
network
metrics
security
```

Logs should include useful operational context without leaking:

- prompts
- generated responses
- tokens
- cookies
- credentials
- sensitive provider payloads

Debug logging may be expanded intentionally, but sensitive content should still remain protected.

---

## 20. Security boundaries

The architecture should assume external provider data and local model files may be invalid.

Validate:

- executable paths
- model paths
- runtime arguments
- decoded provider responses
- endpoint configuration

Do not pass untrusted text through shell interpolation.

Prefer `Process.arguments` over shell command strings.

---

## 21. Extensibility

Future runtimes should be added through new adapters rather than modifying shared product logic.

Potential future implementations:

```text
MLXProvider
OllamaProvider
RemoteLlamaCppProvider
```

Potential future cloud providers:

```text
OpenAIProvider
GeminiProvider
GitHubCopilotProvider
```

The architecture should permit these additions without requiring UI-specific provider branches.

---

## 22. Decisions intentionally deferred

The architecture does not yet define:

- exact Swift package structure
- whether the project uses one or multiple Xcode targets
- exact dependency injection framework
- whether dependencies are bundled or externally installed
- automatic `llama.cpp` updates
- App Sandbox strategy
- XPC services
- privileged helpers
- remote runtime protocol
- historical metrics storage

These decisions should be made only when required by implementation constraints.

---

## 23. Initial architectural constraints

For the MVP:

- native Swift application
- SwiftUI-first
- AppKit only where needed
- structured concurrency
- no database
- no telemetry
- no custom inference engine
- one active local runtime at a time
- `llama.cpp` is the initial runtime
- provider integrations remain isolated
- local runtime binds to loopback by default
- secrets use secure storage
- missing metrics remain unavailable rather than fabricated

These constraints may change through explicit architectural decisions.
