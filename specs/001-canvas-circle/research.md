# Research: Canvas Circle

## Decision: Use Canvas 2D `arc` inline after `fillRect`

**Rationale**: Matches the existing single-file rectangle demo, needs no libraries, and is the standard way to draw a filled+stroked circle. Approved in brainstorming (Approach 1).

**Alternatives considered**:

- Extract draw helpers — clearer for later shapes, but YAGNI for one circle.
- `ellipse` API — equivalent for a circle; less familiar, no benefit.

## Decision: Fixed geometry (center 200,150; radius 100; lineWidth 3)

**Rationale**: Locked in approved design and spec FR-003–FR-005. Leaves an obvious radius for future B-001 (2× size).

**Alternatives considered**: Relative sizing to canvas — unnecessary while canvas size is fixed at 400×300.

## Decision: Manual visual verification only

**Rationale**: Spec SC-002 and project scope exclude automated UI tests for this static page.

**Alternatives considered**: Playwright/Puppeteer screenshot tests — out of scope for T-002.
