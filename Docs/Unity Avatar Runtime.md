---
project: "[[Desktop Guy]]"
type: technical-design
area: desktop-companion
status: proposed
created: 2026-09-21
updated: 2026-09-21
tags:
  - desktop-guy
  - unity
  - animation
  - rondey
related:
  - "[[Personal/Desktop Guy/README]]"
  - "[[Architecture]]"
  - "[[Implementation Plan]]"
  - "[[Vision and Features]]"
---

# Unity Avatar Runtime

## Purpose

Unity is the visual and physical embodiment layer for Desktop Guy. It exists so that building, tuning, and extending the character feels like character work rather than desktop UI programming.

The Unity runtime receives constrained intentions from the .NET host and turns them into expressive movement. It does not decide what applications mean, capture the screen, grant permissions, or directly operate other software.

## Why Unity

Whole-character sprite sheets work well for fixed loops but become expensive when animations must respond to arbitrary desktop targets. Pointing at any control, looking toward a changing window, combining locomotion with expressions, or changing Rondey’s face display otherwise requires many duplicated frames and directions.

Unity provides a visual workflow for:

- Layered animation and state blending.
- Procedural transforms and inverse kinematics.
- Animation events and timelines.
- Particle systems, shaders, lighting, and screen-space effects.
- Rapid tuning while the avatar is running.
- Debugging attachment points, targets, paths, and collision shapes visually.

The goal is not to make Desktop Guy a conventional Unity game. Unity is a specialised renderer and animation runtime inside a larger desktop application.

## Proposed Rondey rig

Rondey should begin as a 2D hierarchy of rigid sprite pieces with carefully placed pivots:

```text
RondeyRoot                 physical screen position
├── Grounding              surface alignment and locomotion offset
│   ├── Shadow
│   └── VisualRoot         authored and procedural visual offsets
│       ├── Body
│       ├── Head
│       │   └── FaceDisplay
│       ├── LeftHand
│       ├── RightHand
│       ├── LeftFoot
│       ├── RightFoot
│       └── EffectAnchors
└── InteractionShape       clicking and dragging
```

Keeping `RondeyRoot`, `Grounding`, and `VisualRoot` separate is important:

- `RondeyRoot` follows the authoritative movement path.
- `Grounding` aligns the character with a taskbar or window surface.
- `VisualRoot` may squash, lean, recoil, bob, or anticipate without corrupting navigation.

## Animation layers

### Authored base motion

Use authored clips for strong silhouettes and personality:

- Idle variations
- Walk and run cycles
- Sit and sleep
- Jump, fall, and land
- Climb and pull-up
- Startled, pleased, confused, and disappointed reactions

These clips define what Rondey feels like. They should not contain assumptions about a particular screen target.

### Procedural targeting

Apply procedural motion after the base pose:

- Turn or bias the head toward a screen point.
- Select the most suitable hand for a target.
- Solve the hand toward a reachable local target.
- Rotate or swap the hand sprite to preserve a clean pointing pose.
- Lean the body slightly toward distant targets.
- Clamp every adjustment to character-specific limits.

If a target is outside the comfortable procedural range, the avatar should reposition or choose an authored whole-body pointing pose instead of distorting the rig.

### Secondary motion

Small additive behaviours can prevent the avatar from feeling mechanical:

- Breathing or hovering
- Blink timing
- Delayed hand and head follow-through
- Landing compression and recovery
- Movement anticipation
- Tiny reactions to a window moving underneath the avatar

These effects should be deterministic enough to replay and debug. Random variation should be seeded or reported where it affects visible behaviour.

## Face display

Rondey’s face should be independent of the body animation. It can be implemented as a child display renderer driven by expression parameters rather than baking the face into every body frame.

An initial vocabulary might include:

- Neutral
- Curious
- Focused
- Happy
- Concerned
- Surprised
- Sleepy
- Error
- Thinking
- Speaking

The display can later present symbolic information such as a progress indicator, question mark, warning, success mark, or application-specific icon. Expression changes should support intensity, blink, transition duration, and optional timed return to neutral.

## Pointing and hands

Pointing is a core interaction, not merely a canned animation. The host supplies a physical screen coordinate. Unity converts it into the avatar window’s local space and chooses one of three strategies:

1. Use procedural hand targeting when the point is within reach.
2. Turn or step toward the point, then use procedural targeting.
3. Use a directional authored pose when the point is far away or the silhouette would become unclear.

Hands should have explicit attachment pivots, reach limits, preferred elbow or arc direction, and interchangeable open/point/grab sprites. The host should not need to know which hand or pose is selected.

## Avoiding visual jitter

The avatar should maintain continuous internal positions while presenting pixel-aligned visuals:

- Use one authoritative movement clock.
- Update physical movement consistently and visual adjustment late in the frame.
- Avoid independently rounding parent and child transforms.
- Snap only the final visual root or render position to the pixel grid.
- Keep camera, native-window movement, and avatar movement from correcting the same displacement twice.
- Preserve stable pivots and sprite bounds across imported assets.
- Separate path motion from bobbing, squash, recoil, and other authored offsets.

When the native player window moves, its screen origin and the character’s local origin must be updated atomically from the avatar’s perspective. Instrument both values during the feasibility prototype so single-pixel oscillation is visible in diagnostics.

## Native window integration

The avatar needs a Windows-specific native window bridge. Its responsibilities include:

- Discovering the Unity player window handle.
- Applying borderless, topmost, tool-window, and activation styles.
- Enabling transparent composition.
- Moving and resizing the window in physical pixels.
- Returning transparent hit-test results outside interactive avatar regions.
- Forwarding monitor, DPI, and native-window changes to Unity.

The bridge should be kept narrow and treated as infrastructure. Character code should consume a Unity-facing interface such as `IAvatarWindow`, not call arbitrary Win32 APIs throughout the project.

## Window sizing strategy

Start with a compact native window that follows the avatar. Its bounds should include configurable effect padding so a hand, speech bubble, glow, or particle does not get clipped.

A virtual-desktop-sized transparent player would simplify coordinates but could increase GPU composition cost and create a very large click-through surface. A per-monitor window or full-desktop mode should be considered only if measurement shows that a compact moving window cannot deliver stable visuals.

## Host communication

Use a local named pipe with asynchronous, length-prefixed messages. The protocol should support:

- Handshake and compatible protocol versions.
- Unique action identifiers.
- Commands and acknowledgements.
- Cancellation and interruption.
- Heartbeats and health state.
- Avatar capability reporting.
- Development diagnostics.

The avatar should queue only work that remains meaningful. A new movement target may supersede an older one, while a confirmed reaction may need to complete unless explicitly cancelled.

## Disconnected behaviour

If the host connection disappears, the avatar should:

1. Stop accepting or beginning autonomous actions.
2. Finish or safely cancel the current visual transition.
3. Enter a passive disconnected pose.
4. Attempt a bounded reconnection if the host is expected to return.
5. Exit after a short grace period if supervision is lost.

It must not continue interpreting stale commands after the host has lost control.

## Feasibility prototype

The prototype should demonstrate one complete interaction rather than a broad animation catalogue:

1. The .NET host starts the Unity avatar.
2. Rondey appears in a transparent, topmost window.
3. Transparent space passes clicks to the application underneath.
4. Rondey remains clickable and draggable.
5. The host supplies a screen target.
6. Rondey walks toward it, looks at it, points, and changes expression.
7. Both processes report completion using the same action identifier.

The test must include monitor boundaries, mixed DPI, target-window movement, host restart, avatar restart, and at least several minutes of idle observation for resource measurement.

## Success criteria

Unity is a suitable production avatar runtime if:

- Native transparency and hit testing are reliable.
- Motion stays smooth while the native window moves.
- Rondey can point and emote without combinatorial sprite-sheet authoring.
- New visual behaviours are substantially easier to create and tune in the Unity Editor.
- Idle resource usage is reasonable for an always-running utility.
- The avatar remains subordinate to the host’s lifecycle and permission model.
