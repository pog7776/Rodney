# Rondey — Unity 2D sprite export

Rondey uses transparent `192 × 208` pixel cells arranged in eight columns.

## Recommended Unity import settings

- Texture Type: **Sprite (2D and UI)**
- Sprite Mode: **Multiple**
- Alpha Source: **Input Texture Alpha**
- Alpha Is Transparency: **On**
- Filter Mode: **Bilinear** for the smooth 3D-toy artwork, or **Point** for a deliberately crisp look
- Compression: **None** while evaluating quality; choose a platform format later
- Sprite Editor → Slice: **Grid by Cell Size**, `192 × 208`, zero offset and padding
- Pivot: **Bottom Center** is a useful starting point for grounded animation

## Atlases

- `Rondey-standard-8x9.png`: rows 0–8 only, `1536 × 1872`.
- `Rondey-v2-8x11.png`: complete Codex/Unity atlas, `1536 × 2288`.
- WebP copies are included for reference and Codex use; Unity workflows generally prefer the PNG files.

## Row map

| Row | Animation | Used columns | Notes |
|---:|---|---:|---|
| 0 | idle | 0–5, plus neutral in 6 | subtle breathing/blink loop |
| 1 | running-right | 0–7 | rightward travel gait |
| 2 | running-left | 0–7 | leftward travel gait |
| 3 | waving | 0–3 | friendly hand wave |
| 4 | jumping | 0–4 | anticipation, lift, peak, descent, settle |
| 5 | failed | 0–7 | disappointment/failure reaction |
| 6 | waiting | 0–5 | approval/input-needed poses |
| 7 | running | 0–5 | active work/processing, not locomotion |
| 8 | review | 0–5 | focused review poses |
| 9 | look 000°–157.5° | 0–7 | up through down-right in 22.5° steps |
| 10 | look 180°–337.5° | 0–7 | down through up-left in 22.5° steps |

Unused atlas cells are fully transparent. The `source-row-strips` folder contains the original generated chroma-backed animation strips for visual reference; use the transparent full atlases for production.

The `previews` folder contains ready-made GIF loops for quickly reviewing animation timing.
