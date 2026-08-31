# Canvas Circle Design (T-002)

## Goal

Extend the existing `index.html` canvas demo so it draws a circle in addition to the existing rectangle.

## Scope

**In scope**

- Keep the current blue filled rectangle unchanged.
- Draw one circle on the same canvas, centered, with fill and stroke.
- Update the document title to `Canvas Shapes`.

**Out of scope**

- New files or build tooling
- UI controls, animation, or interactivity
- Changing canvas size, page layout, or CSS (except as needed for the title)
- Triangle (T-003) or circle size doubling (B-001)

## Visual requirements

| Property | Value |
|----------|--------|
| Center | Canvas midpoint `(200, 150)` on `400×300` |
| Radius | `100` |
| Fill | `#9b59b6` |
| Stroke | `#8e44ad` |
| Stroke width | `3` |

Where the circle overlaps the rectangle, the circle is drawn on top (draw circle after rectangle).

## Approach

Inline Canvas 2D `arc` in the existing `<script>`, matching the current single-file style (no helper extraction yet).

## Implementation steps

1. After the existing `fillRect` call:
   - `ctx.beginPath()`
   - `ctx.arc(200, 150, 100, 0, Math.PI * 2)`
   - `ctx.fillStyle = "#9b59b6"` then `ctx.fill()`
   - `ctx.strokeStyle = "#8e44ad"`, `ctx.lineWidth = 3`, then `ctx.stroke()`
2. Update `<title>` to `Canvas Shapes`.

## Success criteria

- Opening `index.html` shows the original blue rectangle and the purple filled circle with dark-purple outline, centered at radius 100.
- No other files or behavior changes.

## Verification

Visual check in a browser. No automated tests for this static page.
