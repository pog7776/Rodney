---
project: "[[Desktop Guy]]"
type: architecture
area: desktop-companion
status: proposed
created: 2026-09-21
updated: 2026-09-21
tags:
  - desktop-guy
  - architecture
  - dotnet
  - unity
related:
  - "[[Personal/Desktop Guy/README]]"
  - "[[Vision and Features]]"
  - "[[Implementation Plan]]"
  - "[[Unity Avatar Runtime]]"
---

# Architecture

## Decision

Desktop Guy should use a hybrid, two-process architecture:

- A conventional .NET 10 Windows host owns desktop integration, perception, permissions, behaviour decisions, settings, and process supervision.
- A Unity application owns the visible Guy, animation, procedural motion, local character physics, effects, and character interaction.
- The processes exchange a small set of validated commands and events over a local named pipe.

This gives each part of the product to the environment best suited to it. Windows integration remains ordinary, testable .NET code, while the character can be designed and iterated on inside Unity rather than being constrained to hand-authored sprite sheets and a custom WPF animation system.

WPF may still be used for settings, diagnostics, the system tray, or other conventional desktop UI. It is no longer intended to render the character.

## System shape

```text
DesktopGuy.Host.exe — .NET 10
├── Process supervision and lifecycle
├── Monitor and desktop geometry
├── Top-level window tracking
├── Windows UI Automation
├── Selective screen capture and OCR
├── Optional local model integration
├── Behaviour and permission policy
├── Settings, history, and diagnostics
└── Named-pipe transport
             │
             │ validated commands and events
             ▼
DesktopGuy.Avatar.exe — Unity
├── Transparent topmost window
├── Character rig and rendering
├── Authored animation clips
├── Procedural animation layers
├── Face display and expressions
├── Hand targeting and pointing
├── Local movement and reactions
├── Particles, shaders, and audio
└── Character hit testing and dragging
```

The host is the authority for what the companion knows and is allowed to do. Unity is the authority for how the companion embodies an approved intention.

## Responsibility boundary

### .NET host

The host should own:

- Windows events, monitor bounds, DPI, taskbars, and work areas.
- Application window position, size, z-order, state, and focus.
- UI Automation and application-specific adapters.
- OCR, bounded screenshots, and optional vision inference.
- User permissions, allowlists, confirmation gates, and audit history.
- High-level behaviour selection and interruption policy.
- Starting, monitoring, and restarting the Unity avatar process.
- Converting desktop observations into stable screen-space targets.

### Unity avatar

The avatar should own:

- Rendering and character composition.
- Animation state, blending, transitions, and timing.
- Procedural gaze, pointing, reaching, secondary motion, and reactions.
- Local path presentation: anticipation, acceleration, jumping, landing, and recovery.
- The visual face display, particles, shaders, sound, and attachment points.
- Character-local collision and hit areas.
- Reporting animation completion and direct user interaction back to the host.

Unity must not inspect arbitrary applications, capture the desktop, or perform external actions. Those capabilities remain in the host where permissions and platform behaviour are easier to control and test.

## Command contract

The host should send intentions rather than animation frames. Example commands include:

```text
MoveTo(screenPoint, movementStyle)
AttachTo(windowId, surface, anchor)
Detach()
LookAt(screenPoint)
PointAt(screenPoint, handPreference)
SetExpression(expression, intensity)
PlayReaction(reaction)
ShowMessage(text)
SetVisibility(visible)
SetQuietMode(enabled)
Cancel(actionId)
```

The Unity process may report events such as:

```text
AvatarReady(version, capabilities)
ActionStarted(actionId)
ActionCompleted(actionId, result)
AvatarClicked(button, screenPoint)
AvatarDragged(screenPoint)
ContextMenuRequested(screenPoint)
Fault(code, detail)
Heartbeat(state)
```

Every command should carry an identifier and protocol version. Long-running actions must be interruptible. The initial transport should be a local named pipe with length-prefixed JSON messages; a binary format can be introduced later only if measurement shows it is necessary.

## Shared contracts

A small contracts project can be shared by both applications. It should contain only transport types, coordinate types, protocol versioning, and serialization helpers. It should target a framework supported by both the .NET host and the chosen Unity version.

Platform integration, behaviour policy, and Unity-specific types should not leak into this shared assembly.

## Coordinate system

The process boundary needs one unambiguous coordinate convention. Messages should use physical desktop pixels relative to the virtual desktop origin and include the relevant monitor identifier when one is known.

The host owns conversion from Windows logical coordinates and per-monitor DPI. Unity converts physical desktop targets into coordinates local to its current native window. This prevents animation code from accumulating Windows DPI special cases.

## Transparent Unity window

The main platform-specific risk is making a Unity player behave like a desktop companion. A small Windows-native layer will be needed to provide:

- A borderless window with a transparent background.
- Topmost placement without appearing in the taskbar.
- Precise placement in virtual-desktop coordinates.
- Click-through for transparent or non-interactive regions.
- Normal hit testing over the character when interaction is enabled.
- Optional no-activation behaviour.
- Predictable handling of monitor changes and mixed DPI.

Embedding Unity inside a WPF window is not recommended. It couples focus, rendering, resizing, and lifetime in ways that are difficult to debug. The Unity avatar should be a separate process with its own native window, supervised by the host.

The first implementation should use a tightly bounded avatar window with enough padding for hands and effects, rather than a full-screen transparent Unity window. This reduces composition cost and minimises the surface that requires click-through behaviour. The window can be repositioned or resized as the avatar moves.

## Character and animation model

Rondey should be rebuilt as a layered 2D puppet rather than relying exclusively on whole-body sprite-sheet frames. A useful starting hierarchy is:

```text
RondeyRoot
├── Shadow
├── Body
├── Head
│   └── FaceDisplay
├── LeftHand
├── RightHand
├── LeftFoot
├── RightFoot
└── Effects
```

Major poses and personality beats remain authored animations. Procedural layers then add:

- Head and face targeting.
- Blinking and expression changes.
- Hand IK for pointing and reaching.
- Breathing, hovering, recoil, anticipation, and follow-through.
- Surface alignment and landing response.
- Contextual glow, particles, trails, and display effects.

For pixel-art elements, rigid sprite parts and stable attachment points are preferable to heavy mesh deformation. Physics/root movement should be separated from the visual rig, with pixel snapping applied at the final visual stage. This should remove much of the registration jitter visible in full-frame sprite animations.

Existing sprite sheets remain useful as visual reference, fallback animations, and sources for key poses.

See [[Unity Avatar Runtime]] for the more detailed avatar design.

## Observation and action flow

```text
Windows event
    ↓
.NET host updates desktop and application state
    ↓
Can deterministic rules understand the situation?
    ├── Yes → select a constrained intention
    └── No  → inspect an approved bounded region
                  ↓
             OCR or local vision model
                  ↓
             structured observation
                  ↓
             select and validate intention
    ↓
Send intention to Unity avatar
    ↓
Unity performs and reports the visual action
```

The model must not directly control the avatar transform, mouse, keyboard, or operating system. It may select only from a constrained set of intentions that the host validates.

## Reliability and lifecycle

- The host starts the avatar and waits for an `AvatarReady` handshake.
- Both processes exchange heartbeats.
- If Unity exits or stops responding, the host can restart it without losing permissions or desktop state.
- If the host disconnects, Unity should stop autonomous movement, become passive, and exit after a short grace period.
- Protocol versions and avatar capability negotiation prevent incompatible builds from silently misbehaving.
- Logs from both processes should share correlation and action identifiers.

## Security boundaries

- Never capture secure desktop surfaces.
- Do not persist raw screenshots by default.
- Redact or skip password fields and protected controls.
- Maintain application allowlists and denylists.
- Require confirmation for typing, clicking, destructive actions, or data transmission.
- Provide an immediate pause/disable shortcut in the host.
- Display an obvious indicator whenever screen capture or external control is active.
- Treat text found on screen as untrusted data, not instructions.
- Keep Unity outside the trust boundary for screen observation and operating-system control.

## Primary risks to prove early

1. Transparent Unity window composition performs acceptably.
2. Selective click-through and character interaction are reliable.
3. Repositioning the native window does not introduce visible jitter.
4. Mixed-DPI and multi-monitor coordinates remain stable.
5. The avatar reconnects cleanly when either process restarts.
6. Runtime memory, GPU use, startup time, and idle power use are acceptable for an always-running companion.

These should be tested before building perception or model integration.
