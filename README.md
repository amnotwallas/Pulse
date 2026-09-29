# Pulse

Pulse is a native macOS application for monitoring and managing local and cloud AI workflows from a single place.

It is intended to provide visibility into:

- local LLM runtimes
- model performance
- inference metrics
- AI provider usage
- usage limits and reset windows

The first local runtime target is `llama.cpp`, with initial cloud integrations focused on Codex and Claude.

## Goals

Pulse aims to provide a unified macOS-native control layer for AI development workflows.

The project focuses on three main areas.

### Local AI Runtime

Manage and monitor local models running on Apple Silicon.

Initial capabilities:

- start and stop `llama-server`
- select local GGUF models
- configure runtime parameters
- monitor runtime state
- expose the local API endpoint
- inspect inference performance

### Runtime Metrics

Collect useful runtime information such as:

- generation tokens per second
- prompt evaluation tokens per second
- time to first token
- input tokens
- output tokens
- context usage
- memory usage
- active model
- quantization
- server status

### Cloud Provider Usage

Provide usage information for supported AI services.

Initial providers:

- Codex
- Claude

Depending on the data exposed by each provider, Pulse may display:

- usage percentage
- active usage window
- reset time
- model usage
- recent activity
- relevant limits

## Scope

The initial version of Pulse focuses on:

- macOS
- Apple Silicon
- local inference through `llama.cpp`
- GGUF models
- Metal acceleration
- Codex usage monitoring
- Claude usage monitoring

Pulse does not implement LLM inference itself.

Instead, it manages and observes external runtimes such as `llama.cpp`.

## Architecture

Pulse is designed around provider abstractions so that runtime and cloud integrations remain independent from the rest of the application.

```text
Pulse
│
├── Core
│   ├── Providers
│   ├── Models
│   ├── Metrics
│   └── Runtime
│
├── Infrastructure
│   ├── llama.cpp
│   ├── Codex
│   └── Claude
│
└── Application
```

Provider-specific logic should remain isolated behind common interfaces.

This allows additional runtimes and services to be supported later without coupling the application to a single backend.

## Local Runtime

The first supported local runtime is `llama.cpp`.

Pulse will manage a `llama-server` process and observe its state.

A typical runtime may look like:

```bash
llama-server \
  -m ~/Models/model.gguf \
  --host 127.0.0.1 \
  --port 8080 \
  -c 32768
```

Pulse is expected to handle responsibilities such as:

```text
discover model
      ↓
build runtime configuration
      ↓
start llama-server
      ↓
monitor process
      ↓
collect metrics
      ↓
stop / restart runtime
```

The API exposed by `llama-server` may then be consumed by external tools such as:

- OpenCode
- Continue
- Aider
- custom agents
- development tools using OpenAI-compatible APIs

## Technology

Initial technology stack:

- Swift
- SwiftUI
- AppKit where required
- Swift Concurrency
- macOS
- Apple Silicon
- Metal
- llama.cpp

Additional dependencies should be kept minimal.

## Project Structure

The exact structure may evolve, but the repository is expected to separate core logic from provider-specific integrations.

```text
Pulse/
├── Pulse/
│   ├── App/
│   ├── Core/
│   │   ├── Models/
│   │   ├── Providers/
│   │   ├── Metrics/
│   │   └── Runtime/
│   │
│   ├── Infrastructure/
│   │   ├── LlamaCpp/
│   │   ├── Codex/
│   │   └── Claude/
│   │
│   └── UI/
│
├── PulseTests/
├── docs/
│   ├── DESIGN.md
│   ├── ARCHITECTURE.md
│   └── PRODUCT.md
│
├── AGENTS.md
└── README.md
```

## MVP

The initial milestone should provide a usable local runtime manager and basic provider monitoring.

### Local runtime

- detect or configure `llama.cpp`
- discover GGUF models
- start `llama-server`
- stop `llama-server`
- restart runtime
- select model
- configure context size
- expose runtime status
- expose API endpoint

### Metrics

- generation tokens/sec
- prompt evaluation tokens/sec
- time to first token
- input tokens
- output tokens
- context usage
- memory usage
- active model
- quantization
- server state

### Providers

- Codex usage monitoring
- Claude usage monitoring

## Non-Goals for the First Version

The first version should avoid unnecessary scope expansion.

Not required initially:

- multiple local runtimes loaded simultaneously
- distributed inference
- cloud synchronization
- advanced historical analytics
- automatic model downloads
- model training or fine-tuning
- remote account management

## Documentation

Detailed project decisions should live outside this README:

- `docs/PRODUCT.md` — product scope and behavior
- `docs/ARCHITECTURE.md` — technical architecture and boundaries
- `docs/DESIGN.md` — UI/UX direction and interaction patterns
- `AGENTS.md` — instructions and constraints for coding agents working in the repository

## Status

Pulse is currently in the planning and prototyping stage.

The initial focus is validating the local runtime integration and establishing stable provider abstractions before expanding the feature set.
