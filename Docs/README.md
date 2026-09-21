---
project: "[[Desktop Guy]]"
type: project-overview
area: desktop-companion
status: planning
created: 2026-09-21
updated: 2026-09-21
tags:
  - desktop-guy
  - desktop-companion
  - dotnet
  - unity
related:
  - "[[Vision and Features]]"
  - "[[Architecture]]"
  - "[[Implementation Plan]]"
  - "[[Unity Avatar Runtime]]"
---

# Desktop Guy

Desktop Guy is a locally running animated desktop companion that can understand enough about the current screen to move around it, react to applications, and—when explicitly permitted—help the user interact with them.

The core idea is feasible. It combines several established techniques rather than depending on one unusually capable model:

- A transparent, always-on-top Unity window renders and animates the character.
- Windows supplies monitor bounds, application window positions, and an accessibility/UI Automation tree.
- OCR and selective screenshots cover interfaces that do not expose useful structured information.
- A small local language or vision-language model interprets ambiguous situations and selects high-level intentions.
- A .NET host validates those intentions and Unity turns them into deterministic, expressive behaviour.

The character does not need to continuously send the entire desktop through a vision model. Most of its awareness can be event-driven and obtained from structured operating-system information. Vision is a fallback for genuinely visual content.

## Product direction

The first version should be a charming, screen-aware companion rather than an autonomous computer-control agent. It should be useful and expressive while remaining predictable.

Initial behaviours could include:

- Running between monitors.
- Sitting on the taskbar or active window.
- Following a window while it moves.
- Climbing around window edges and title bars.
- Reacting when applications open, close, minimise, or gain focus.
- Reacting to useful application state, such as a build completing or an error dialog appearing.
- Pointing toward a relevant control or notification.
- Receiving limited user-provided context and remembering preferences locally.

“Climbing into” another application would normally be a visual illusion. Desktop Guy remains in its own transparent overlay, but tracks and clips itself against the target window. This is much safer and more compatible than injecting code into other applications.

## Design principles

1. **Local first** — screen observations, context, and memory remain on the machine by default.
2. **Structured before visual** — prefer window metadata and UI Automation over screenshots.
3. **Event-driven awareness** — observe meaningful changes rather than sampling the screen continuously.
4. **Deterministic embodiment** — models choose intentions; ordinary code owns movement, collision, and animation.
5. **Permission before action** — observing, suggesting, and controlling are separate capability levels.
6. **Visible state** — the user should always be able to tell when the companion is observing or acting.

## Technical direction

Desktop Guy uses a hybrid architecture. A .NET 10 Windows host owns desktop awareness, behaviour policy, permissions, settings, and operating-system integration. A separate Unity process owns the visible Guy, including authored and procedural animation, facial expressions, hands, local physics, and visual effects.

The processes communicate through a small versioned named-pipe protocol. The host sends intentions such as moving, looking, pointing, or reacting; Unity decides how to perform them. This keeps sensitive desktop access out of the avatar process while allowing the character to be built and tuned visually in Unity.

The initial technical milestone is a feasibility prototype proving transparent-window composition, selective click-through, smooth native-window movement, multi-monitor DPI behaviour, and host-to-avatar communication.

## Notes

- [[Vision and Features]] describes the intended experience and capability levels.
- [[Architecture]] describes the proposed Windows implementation.
- [[Implementation Plan]] breaks development into testable milestones.
- [[Unity Avatar Runtime]] describes the character rig, procedural animation, face display, native window, and host protocol.
