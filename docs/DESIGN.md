# DESIGN.md

# Pulse — Design Specification

## 1. Purpose

This document defines the visual and interaction design direction for Pulse.

Pulse should feel like a native macOS utility that remains lightweight, glanceable, and immediately useful during development work.

The design should prioritize:

- fast status recognition
- low visual noise
- compact presentation
- clear hierarchy
- smooth transitions
- native macOS behavior
- minimal interruption to the user's workflow

This document intentionally focuses on interaction and visual behavior rather than implementation architecture.

---

## 2. Product character

Pulse should feel:

- precise
- technical
- calm
- responsive
- lightweight
- native
- modern without appearing ornamental

Pulse should not feel like:

- a web dashboard embedded in a desktop app
- a monitoring console with excessive chrome
- a generic settings window
- a terminal replacement
- a dense analytics product
- a decorative widget with little operational value

The application should communicate that it is infrastructure tooling, but without adopting the visual density of traditional infrastructure dashboards.

---

## 3. Primary interaction model

Pulse is centered around a compact floating surface that can expand into a larger control surface.

The experience is composed of three levels:

1. **Compact state**
2. **Expanded state**
3. **Full detail views**

The compact and expanded states are intended for frequent interaction.

Full detail views are intended for deeper inspection, configuration, and historical information.

---

## 4. Compact state

The compact state should provide useful status at a glance.

It should answer one primary question depending on context:

- Is the local runtime active?
- How fast is the current model generating?
- Is a provider approaching its usage limit?
- Is something unavailable or in error?

Examples of possible compact content:

```text
21.8
tok/s
```

```text
72%
CODEX
```

```text
READY
QWEN
```

```text
OFF
LOCAL
```

Only one primary metric should dominate the compact surface.

Secondary information should be omitted unless it materially improves interpretation.

---

## 5. Expanded state

Expanding Pulse should reveal operational detail without becoming a full dashboard.

The expanded state may include:

- active provider or runtime
- model name
- runtime state
- generation speed
- prompt processing speed
- TTFT
- context usage
- memory usage
- usage window
- reset time
- primary actions
- navigation to other views

The expanded state should remain concise.

Detailed graphs, historical logs, advanced settings, and provider diagnostics belong elsewhere.

---

## 6. Views

The initial product should support four main views.

### 6.1 Overview

The default aggregate view.

It should provide high-level state for:

- Local runtime
- Codex
- Claude

The overview should prioritize comparison and awareness rather than configuration.

Example information hierarchy:

```text
Pulse

Local
Qwen3-Coder 30B
21.8 tok/s

Codex
72%
Reset in 2d 14h

Claude
82%
Reset in 51m
```

---

### 6.2 Local

The local runtime view should provide control and performance information.

Primary information:

- runtime state
- active model
- quantization when available
- generation tokens/sec
- prompt tokens/sec
- TTFT
- memory usage
- context usage
- API endpoint

Primary actions:

- start
- stop
- restart
- switch model

Secondary actions may include:

- copy endpoint
- open logs
- open advanced settings

---

### 6.3 Codex

The Codex view should focus on:

- current usage
- usage window
- reset time
- active or available model information
- relevant secondary limits when available

The most important metric should remain visually dominant.

Do not create fake precision or visual elements for unavailable data.

---

### 6.4 Claude

The Claude view should focus on:

- current usage
- current window
- reset time
- provider status
- relevant model limits when available

The structure should remain consistent with Codex where the underlying concepts are comparable.

---

## 7. Navigation

Navigation should be lightweight.

Preferred patterns:

- horizontal swipe between views
- click or tap on compact provider summaries
- discrete page indicators
- keyboard navigation where practical

Avoid:

- large tab bars
- persistent sidebars for the compact experience
- deeply nested navigation
- multi-level menus for common actions

A full settings or analytics window may use more conventional macOS navigation.

---

## 8. Visual hierarchy

Each view should follow a simple hierarchy:

1. primary metric or state
2. provider/model identity
3. supporting metrics
4. actions
5. secondary metadata

Example:

```text
21.8 tok/s
Qwen3-Coder 30B

TTFT       382 ms
Memory     19.1 / 24 GB
Context    18K / 32K

Stop   Models   More
```

The primary metric should be readable immediately without scanning the entire view.

---

## 9. Layout principles

### Compact

- minimal footprint
- one dominant value
- one short label at most
- no multi-column layout
- no graphs
- no long text

### Expanded

- compact vertical rhythm
- small number of sections
- values aligned consistently
- readable at a glance
- avoid excessive cards inside cards

### Full views

- may use denser layouts
- may include history or charts
- may include logs and advanced configuration
- should still preserve the same typography and spacing system

---

## 10. Shape language

Pulse should use soft, continuous geometry.

Preferred characteristics:

- rounded surfaces
- visually connected compact and expanded states
- minimal hard edges
- restrained borders
- subtle depth

Avoid:

- excessive outlines
- nested rectangles
- strong card borders
- exaggerated shadows
- highly decorative geometry

The transition between compact and expanded state should feel like one surface changing shape rather than one view being replaced by another.

---

## 11. Color

Pulse should use a predominantly neutral palette.

Recommended direction:

- dark neutral base
- high-contrast primary text
- subdued secondary text
- one restrained status accent at a time

Status color should communicate meaning, not decoration.

Examples:

- ready
- generating
- stopped
- warning
- error

Do not rely exclusively on color to communicate state.

Text, iconography, or motion should reinforce meaning.

---

## 12. Typography

Typography should remain native and compact.

Use the macOS system typeface unless a future product decision explicitly changes this.

Recommended hierarchy:

- primary metric: large, high emphasis
- provider/model: medium emphasis
- labels: small
- secondary metadata: smaller and subdued

Numeric metrics should align cleanly.

Where appropriate, monospaced digits may improve stability for rapidly changing values.

---

## 13. Metrics presentation

Metrics should be shown with explicit units.

Preferred examples:

```text
21.8 tok/s
382 ms
19.1 / 24 GB
18K / 32K
72%
```

Avoid:

```text
Speed: 21.812739
Memory utilization ratio: 0.796
```

Precision should reflect actual usefulness.

Suggested defaults:

- tokens/sec: one decimal place
- milliseconds: whole numbers unless very small
- percentages: whole numbers
- memory: one decimal place in GB
- large token counts: compact notation when space is constrained

---

## 14. Progress indicators

Progress bars may be used for bounded quantities such as:

- memory usage
- context usage
- provider usage limits

Progress indicators should:

- remain visually quiet
- avoid unnecessary labels inside the bar
- include a numeric value nearby
- preserve readability at small sizes

Do not use a progress bar for unbounded metrics such as tokens/sec.

---

## 15. Runtime states

### Stopped

Should communicate inactivity without appearing broken.

Preferred language:

```text
OFF
```

or:

```text
STOPPED
```

### Starting

Should clearly communicate model load or runtime startup.

Avoid misleading performance metrics during startup.

### Ready

Should communicate that the server is available even when no request is active.

Preferred:

```text
READY
```

rather than:

```text
0 tok/s
```

### Active

During inference, generation speed may become the dominant metric.

Example:

```text
21.8
tok/s
```

### Error

Error states should be visually distinct but not overwhelming.

The compact state may show:

```text
ERROR
LOCAL
```

Expanded state should expose the actionable error summary.

---

## 16. Motion

Motion should communicate state transitions.

Recommended uses:

- compact → expanded
- expanded → compact
- view switching
- runtime starting
- model loading
- metric updates
- error appearance

Motion should be:

- short
- smooth
- interruptible
- physically coherent
- subtle

Prefer spring-like transitions for resizing and repositioning when they feel native.

Avoid:

- constant decorative animation
- large bounces
- excessive blur
- motion that delays interaction

Respect Reduce Motion.

---

## 17. Live metrics behavior

Metrics that change rapidly should not visually jitter.

Techniques may include:

- monospaced digits
- stable alignment
- controlled refresh intervals
- subtle value transitions
- avoiding full-layout recomputation where possible

The interface should not update faster than the user can meaningfully perceive.

For example, tokens/sec may update several times per second internally but render at a lower visual cadence if that improves stability.

---

## 18. Provider consistency

Codex and Claude should share layout conventions when displaying comparable concepts.

For example:

```text
Provider
Usage
Reset
Window
Status
```

Do not force providers into identical layouts when their available data differs materially.

Missing fields should be omitted or marked unavailable rather than replaced with empty decorative components.

---

## 19. Controls

Primary actions should be immediately understandable.

Examples:

- Start
- Stop
- Restart
- Models
- Copy Endpoint
- Open Logs

Use native macOS control behavior.

Destructive or disruptive actions should not be placed where they are easy to trigger accidentally.

Stopping an active local runtime may require stronger visual distinction than passive actions such as copying an endpoint.

---

## 20. Menu bar role

Pulse may expose a conventional macOS menu bar item in addition to the floating experience.

The menu bar item should serve as a stable system-level control point for actions such as:

- show or hide Pulse
- open Pulse
- open settings
- launch at login
- runtime quick actions
- quit

The menu bar item should not duplicate the entire expanded interface.

---

## 21. Full detail surfaces

Some information should intentionally remain outside the compact experience.

Examples:

- historical metrics
- charts
- detailed logs
- configuration
- provider authentication
- model directories
- advanced llama.cpp arguments
- diagnostics

These may live in a standard macOS window.

This preserves the lightweight character of the primary Pulse surface.

---

## 22. Settings

Settings should be grouped by responsibility.

Possible categories:

```text
General
Local Runtime
Models
Providers
Appearance
Advanced
```

Settings should not contain operational information that belongs in the main runtime view.

---

## 23. Empty states

Empty states should explain what is missing and what to do next.

Examples:

### No local runtime configured

```text
Local runtime not configured

Choose a llama-server executable and a GGUF model to begin.
```

### No Codex integration

```text
Codex is not configured
```

### No model found

```text
No GGUF models found

Add a model directory in Settings.
```

Avoid decorative empty states that do not provide recovery guidance.

---

## 24. Error states

Errors should be concise in primary UI.

Example:

```text
Runtime failed to start

Port 8080 is already in use.
```

Actions may include:

- Retry
- Change Port
- Open Logs

Detailed technical information may be available separately.

---

## 25. Accessibility

Pulse should support:

- VoiceOver
- keyboard navigation
- Reduce Motion
- sufficient contrast
- Dynamic Type where practical on macOS
- state communication without relying only on color

Interactive regions should remain usable even when the compact surface is small.

---

## 26. Responsive behavior

The interface should tolerate:

- varying menu bar configurations
- multiple displays
- different display scaling
- screen edge changes
- full-screen apps
- Stage Manager
- changing active display

The floating surface should never become permanently inaccessible because of display changes.

---

## 27. Multi-display behavior

Initial behavior should favor predictability over complexity.

Recommended MVP rule:

- Pulse belongs to one active display at a time
- the user may move it explicitly
- its last display/position may be remembered when reliable

Automatic jumping between displays should be avoided unless clearly justified.

---

## 28. Interaction density

The compact surface should remain intentionally sparse.

A useful test:

> If removing a metric does not reduce the user's ability to understand current state, it probably does not belong in compact mode.

The expanded state should also avoid becoming a miniature analytics dashboard.

---

## 29. Design system direction

A formal component system may be introduced as the implementation grows.

Likely reusable primitives:

- metric value
- metric row
- usage bar
- status indicator
- provider header
- runtime action
- model label
- error summary
- page indicator

These components should share spacing, typography, and state behavior.

---

## 30. MVP design scope

The design MVP should cover:

- compact state
- expanded state
- overview
- local runtime view
- Codex view
- Claude view
- loading state
- stopped state
- ready state
- active inference state
- error state
- basic settings access
- menu bar fallback/control

The MVP does not require:

- historical charts
- advanced analytics
- model marketplace
- onboarding animation
- themes
- customizable layouts
- complex personalization

---

## 31. Open design questions

The following decisions remain intentionally open:

- exact compact dimensions
- exact expanded dimensions
- left-edge vs right-edge default placement
- whether the compact surface is always visible
- exact material/background treatment
- exact status accent colors
- precise animation curves
- whether provider switching uses swipe, click, or both
- whether local runtime activity automatically becomes the compact primary view
- whether usage warnings temporarily override the selected compact metric

These should be validated through prototypes rather than decided only in documentation.

---

## 32. Design principles

### Glanceable

The user should understand the primary state in under a second.

### Native

The app should behave like a macOS utility, not a web product.

### Quiet

Pulse should be present without constantly demanding attention.

### Operational

Every prominent element should communicate state or enable action.

### Stable

Live data should not make the interface visually unstable.

### Honest

Unavailable or uncertain data should remain unavailable rather than being visually implied.

### Progressive

More detail should appear only when the user asks for it.
