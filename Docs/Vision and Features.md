---
project: "[[Desktop Guy]]"
type: product-vision
area: desktop-companion
status: planning
created: 2026-09-21
updated: 2026-09-21
tags:
  - desktop-guy
  - product-vision
  - local-ai
related:
  - "[[Personal/Desktop Guy/README]]"
  - "[[Architecture]]"
  - "[[Implementation Plan]]"
---

# Vision and Features

## Experience

Desktop Guy should feel like a small creature that inhabits the desktop rather than a chat window with legs. It notices meaningful changes, moves with purpose, and expresses state through animation. It should remain quiet enough to live with every day.

The companion might understand situations such as:

- “The user moved to the other monitor.”
- “Rider has reported a build error.”
- “Unity entered Play Mode.”
- “A confirmation dialog appeared.”
- “The active application is busy.”
- “There is a control I can point out, but I should not click it without permission.”

## Potential feature set

### Embodiment

- Transparent Unity-rendered desktop overlay.
- Layered 2D character rig with authored base clips and procedural animation.
- Walking, running, climbing, jumping, falling, sitting, sleeping, pointing, and reacting.
- Independent face display, gaze, hand targeting, expressions, particles, and shader effects.
- Window edges, title bars, taskbars, and monitor boundaries represented as navigable surfaces.
- Per-monitor positioning with correct DPI scaling.
- Optional sound, speech bubbles, or text-to-speech.

### Awareness

- Active application and active window.
- Window position, size, z-order, minimised state, and monitor.
- UI Automation hierarchy: controls, names, roles, values, states, and screen bounds.
- Focus changes, notifications, dialog appearance, and application lifecycle events.
- OCR for text that is visible but not exposed through accessibility APIs.
- Selective vision over a window or small screen region when structured information is insufficient.
- User-supplied context such as current task, preferred applications, and do-not-disturb periods.

### Assistance

- Point at a relevant button or region.
- Explain what appeared on screen.
- Offer a suggested action.
- Invoke an accessible control after approval.
- Type or perform pointer actions only in an explicitly enabled control mode.
- Support allowlists and per-application permissions.

### Personality and memory

- Configurable temperament and activity level.
- Local preferences and short summaries rather than raw screenshot history.
- Application-specific reactions.
- Daily routines, idle behaviour, and contextual animation.
- A quiet mode that suppresses unsolicited reactions.

## Capability levels

Desktop interaction should be divided into clearly visible modes:

1. **Companion** — observes approved context and reacts; never controls other applications.
2. **Assistant** — highlights or points at controls and suggests actions.
3. **Agent** — performs a specific approved action.
4. **Trusted automation** — performs a small allowlisted set of repeatable actions.

The project should begin and remain useful at level 1. Higher levels should be added only after observation, permissions, audit history, and cancellation are dependable.

## Out of scope for the first version

- Unrestricted autonomous mouse and keyboard control.
- Continuous full-resolution desktop recording.
- Injecting code into arbitrary application processes.
- Attempting to interact with secure desktop prompts.
- Cloud-dependent perception as the default.
- General-purpose long-horizon automation.
