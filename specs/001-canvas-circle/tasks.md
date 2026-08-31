# Tasks: Canvas Circle

**Input**: Design documents from `/specs/001-canvas-circle/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: None requested — visual check via quickstart.md only.

## Phase 1: Setup

**Purpose**: Confirm Speckit feature artifacts are in place

- [x] T001 Verify `specs/001-canvas-circle/` contains spec, plan, research, data-model, quickstart, and contracts

---

## Phase 2: Foundational

**Purpose**: No shared infrastructure beyond existing `index.html`

- [x] T002 Confirm existing rectangle draw in `index.html` remains the baseline (no foundation changes)

---

## Phase 3: User Story 1 - See rectangle and circle together (Priority: P1) 🎯 MVP

**Goal**: Show both shapes on the demo page

**Independent Test**: Open `index.html` and see blue rectangle + centered purple circle

### Implementation for User Story 1

- [x] T003 [US1] Update `<title>` to `Canvas Shapes` in `index.html`
- [x] T004 [US1] After `fillRect`, draw circle via `arc` (center 200,150; radius 100; fill `#9b59b6`; stroke `#8e44ad`; lineWidth 3) in `index.html`

**Checkpoint**: Visual demo matches contracts/visual-demo.md

---

## Phase 4: Polish

- [x] T005 Validate against `specs/001-canvas-circle/quickstart.md`

## Dependencies

- T001 → T002 → T003 → T004 → T005

## Implementation Strategy

MVP = Phase 3 only on existing `index.html`. No parallel work (single file).
