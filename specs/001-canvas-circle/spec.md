# Feature Specification: Canvas Circle

**Feature Branch**: `task/T-002`

**Created**: 2026-08-31

**Status**: Draft

**Input**: User description: "canvas 中绘制一个圆形 — keep the existing rectangle; add a centered circle with purple fill and darker purple outline, large enough to dominate (radius ~100 on the existing canvas)"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - See rectangle and circle together (Priority: P1)

A visitor opens the demo page and immediately sees both the existing blue rectangle and a new large purple circle on the same drawing surface. The circle is filled, outlined, and centered so it is the dominant shape.

**Why this priority**: This is the entire feature — without both shapes visible, the task is incomplete.

**Independent Test**: Open the page in a browser and confirm both shapes appear with the agreed appearance.

**Acceptance Scenarios**:

1. **Given** the demo page is opened, **When** the drawing surface loads, **Then** the existing blue filled rectangle is still visible and unchanged in position and size.
2. **Given** the demo page is opened, **When** the drawing surface loads, **Then** a filled purple circle with a darker purple outline appears centered on the drawing surface and is large enough to dominate the composition.
3. **Given** both shapes are drawn, **When** they overlap, **Then** the circle appears on top of the rectangle in the overlap region.

---

### Edge Cases

- Opening the page with JavaScript disabled: no shapes appear (same as today; no new requirement).
- Very small viewports: the page still centers the drawing surface; shapes keep their fixed canvas size (no responsive redraw required).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The demo page MUST continue to show the existing blue filled rectangle with its current size and position.
- **FR-002**: The demo page MUST show one circle on the same drawing surface as the rectangle.
- **FR-003**: The circle MUST be centered on the drawing surface.
- **FR-004**: The circle MUST use a purple fill (`#9b59b6`) and a darker purple outline (`#8e44ad`) with a clearly visible outline thickness.
- **FR-005**: The circle MUST be large enough to dominate the composition (radius 100 on the existing 400×300 surface).
- **FR-006**: The page title MUST indicate multiple shapes (use `Canvas Shapes`).
- **FR-007**: The feature MUST NOT add new interactive controls, animation, or additional pages/files beyond updating the existing demo page.

### Key Entities

- **Rectangle**: Existing blue filled shape on the drawing surface; must remain unchanged.
- **Circle**: New centered shape with purple fill and darker purple outline.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A first-time viewer can identify both a rectangle and a circle on the page within 5 seconds of opening it.
- **SC-002**: 100% of acceptance scenarios for User Story 1 pass on a manual visual check in a modern browser.
- **SC-003**: No additional demo pages or navigation steps are required to see both shapes (single page open).

## Assumptions

- The existing static `index.html` demo page is the sole delivery surface.
- Visual verification in a browser is sufficient; automated UI tests are out of scope.
- Triangle drawing (T-003) and doubling circle size (B-001) remain separate follow-up work.
- Outline thickness of 3 CSS pixels is an acceptable default for “clearly visible.”
