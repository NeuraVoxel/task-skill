# Data Model: Canvas Circle

No persistent data store. In-memory drawing parameters on page load:

## Rectangle (existing)

| Field | Value |
|-------|--------|
| Position | `(80, 60)` |
| Size | `240 × 160` |
| Fill | `#4a90d9` |

## Circle (new)

| Field | Value | Validation |
|-------|--------|------------|
| Center X | `200` | Canvas midpoint (`width / 2`) |
| Center Y | `150` | Canvas midpoint (`height / 2`) |
| Radius | `100` | Must fit within 400×300 with stroke |
| Fill | `#9b59b6` | Required |
| Stroke | `#8e44ad` | Required |
| Stroke width | `3` | Clearly visible |

## Relationships

- Both shapes share one canvas context.
- Draw order: rectangle first, then circle (circle on top when overlapping).
