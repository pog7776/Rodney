---
project: "[[Desktop Guy]]"
type: implementation-plan
area: desktop-companion
status: proposed
created: 2026-09-21
updated: 2026-09-21
tags:
  - desktop-guy
  - roadmap
  - dotnet
  - unity
related:
  - "[[Personal/Desktop Guy/README]]"
  - "[[Vision and Features]]"
  - "[[Architecture]]"
  - "[[Unity Avatar Runtime]]"
---

# Implementation Plan

## Phase 0 — Hybrid feasibility prototype

Goal: prove that Unity can behave like a polished Windows desktop companion before building the wider system.

- Create a minimal Unity 2D avatar project.
- Render a placeholder or initial Rondey puppet against a transparent player background.
- Make the native Unity window borderless, topmost, and absent from the taskbar.
- Position the window using physical virtual-desktop coordinates.
- Implement click-through outside the character and interaction over its hit area.
- Create a tiny .NET 10 console host that starts Unity and exchanges messages over a named pipe.
- Send `MoveTo`, `LookAt`, `PointAt`, and `SetExpression` commands.
- Test on multiple monitors and mixed DPI settings.
- Measure startup time, idle CPU/GPU use, memory, and movement smoothness.

Acceptance criteria:

- The desktop remains usable through transparent regions.
- Rondey can be clicked and dragged reliably.
- The Unity window does not steal focus during passive behaviour.
- Movement remains visually stable across monitor boundaries.
- The avatar reconnects or exits safely if the host is restarted.
- Resource use is acceptable for an application intended to remain open.

This phase is a decision gate. Do not begin model or perception work until the overlay is proven.

## Phase 1 — Solution foundation

Goal: establish the production host, avatar, and shared protocol.

- Retain the .NET 10 Windows solution as the host foundation.
- Add projects for host services, shared contracts, and tests.
- Add the Unity avatar project beside the .NET solution.
- Define versioned commands, events, heartbeats, cancellation, and capability negotiation.
- Add structured logging with shared action identifiers.
- Add a single-instance guard and host-owned avatar process supervision.
- Add a tray menu with Show, Hide, Pause, Diagnostics, and Exit actions.

Acceptance criteria:

- Both applications build reproducibly.
- The host starts and stops the avatar cleanly.
- Protocol mismatches fail visibly rather than producing undefined behaviour.
- A crashed or unresponsive avatar can be restarted safely.

## Phase 2 — Rondey avatar foundation

Goal: replace the full-frame sprite dependency with an expressive layered character.

- Split Rondey into stable body, head, face, hands, feet, shadow, and effect layers.
- Establish pivots, attachment points, sorting, and pixel-perfect import settings.
- Build authored idle, walk, run, sit, sleep, jump, fall, land, and reaction clips.
- Add procedural gaze, blinking, breathing, anticipation, and follow-through.
- Add left- and right-hand targets for pointing and reaching.
- Implement a dynamic face display with a small expression vocabulary.
- Separate physical/root movement from the visual rig and apply final pixel snapping.
- Keep existing sprite sheets as references and fallback clips.

Acceptance criteria:

- Core poses do not jump because of inconsistent frame registration.
- Rondey can look and point at an arbitrary screen-space target.
- Facial expressions can change independently of locomotion.
- New expressions and gestures can be added without redrawing every locomotion frame.
- Reduced-motion and animation-speed settings are respected.

## Phase 3 — Desktop geometry and surfaces

Goal: allow the avatar to inhabit the desktop deterministically without AI.

- Discover monitors, work areas, scaling factors, and taskbars in the host.
- Track top-level windows and their visible rectangles.
- Observe foreground, move, resize, minimise, restore, and close events.
- Convert window and monitor boundaries into navigation surfaces.
- Send stable physical-pixel anchors to Unity.
- Implement walking, jumping, falling, climbing, attachment, and detachment presentation.
- Follow a selected window while it moves.

Acceptance criteria:

- The avatar can travel between monitors with different DPI scaling.
- It can sit on and follow an application window.
- It handles minimised or closed target windows gracefully.
- It never becomes permanently stranded off-screen.
- Native-window repositioning does not produce visible jitter.

## Phase 4 — Behaviour coordination

Goal: coordinate host-owned intent with Unity-owned performance.

- Implement an interruptible host behaviour state machine.
- Add action priorities, cooldowns, budgets, and cancellation.
- Map high-level intentions to Unity performance sequences.
- Let Unity acknowledge start, completion, interruption, and failure.
- Add passive and disconnected states.
- Add a development inspector showing the current intention, animation state, targets, and message traffic.

Acceptance criteria:

- Behaviour does not thrash between states.
- Higher-priority reactions can interrupt lower-priority idle activity cleanly.
- The host never needs to prescribe animation frames.
- Every visible action can be traced to a host intention or direct user interaction.

## Phase 5 — Structured awareness

Goal: understand common application interfaces without vision models.

- Read approved windows using Windows UI Automation.
- Build a compact representation of relevant controls and bounds.
- Detect dialogs, buttons, status text, progress indicators, and focused controls.
- Add application-specific adapters only where generic accessibility data is insufficient.
- Implement reactions through deterministic rules and the behaviour state machine.

Acceptance criteria:

- The companion can identify useful controls in common WPF, Win32, WinForms, Electron, and browser interfaces where accessibility data is available.
- It reacts to meaningful changes without polling the complete desktop.
- Sensitive and password fields are excluded.
- Observations are not exposed directly to the Unity process.

## Phase 6 — Selective visual perception

Goal: interpret visual information that UI Automation cannot expose.

- Capture only an approved foreground window or bounded region.
- Add OCR as the first fallback.
- Define a provider interface for a local vision-language model.
- Run inference in a separate process outside both the host and avatar.
- Cache stable observations and impose strict frequency and resource limits.
- Return structured observations rather than free-form instructions.

Acceptance criteria:

- The companion can describe a bounded visual region on demand.
- Screen capture is visibly indicated and can be disabled immediately.
- Raw screenshots are not persisted by default.
- The avatar remains responsive while inference runs.

## Phase 7 — Context and personality

Goal: turn observations into coherent but predictable behaviour.

- Store user preferences and compact contextual summaries locally.
- Add temperament, activity, interruption, and quiet-mode settings.
- Combine deterministic rules with model-selected high-level intentions.
- Prevent repeated or distracting reactions through cooldowns and budgets.
- Add a transparent “why did it do that?” history.
- Expand the Unity expression and reaction catalogue as behaviours require it.

Acceptance criteria:

- Behaviour remains understandable and does not thrash between states.
- The companion respects quiet mode and application exclusions.
- Context can be reviewed and deleted by the user.
- Personality changes presentation without bypassing host policy.

## Phase 8 — Assisted interaction

Goal: add useful help without granting uncontrolled access.

- Implement pointing and highlighting first.
- Add suggestions that the user explicitly accepts.
- Add UI Automation invocation for approved controls.
- Add pointer and keyboard synthesis only when necessary.
- Introduce capability levels, per-application permissions, confirmations, cancellation, and audit history.

Acceptance criteria:

- Observation mode cannot perform application actions.
- Every external action is attributable, interruptible, and constrained.
- Sensitive or destructive actions require explicit confirmation.
- Unity can present an action but cannot authorise or execute it directly.

## Recommended first development slice

The first useful vertical slice is:

1. Transparent, topmost Unity avatar window.
2. Selective click-through and character dragging.
3. Named-pipe connection to the .NET host.
4. A layered Rondey with procedural gaze and one pointing hand.
5. Walk to and sit on a host-provided window edge.
6. A dynamic face reaction to a simulated build result.

This demonstrates both the distinctive experience and the highest-risk platform boundary without depending on perception or a language model.

## Open decisions

- Final project and product name.
- Exact Rondey layer breakdown and initial expression catalogue.
- Unity render pipeline and pixel-art import conventions.
- Tight moving avatar window versus a larger per-monitor window if effects require more space.
- Whether interaction temporarily activates the avatar window or is handled without activation.
- Initial application integrations: Rider, Unity, browser, or general Windows UI.
- Local inference hardware target and acceptable memory budget.
- Acceptable Unity runtime memory, startup, GPU, and idle-power budgets.
- Whether macOS support is a future requirement; the first native overlay layer will be Windows-specific.
