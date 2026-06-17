---
name: validating-buildings
description: >-
  Use after writing or editing a React Arch building, or to run the agent
  feedback loop. Covers `react-arch check` and its flags, the JSON report shape,
  the `review()` function, every diagnostic code with how to fix it, and how to
  iterate until the model is clean.
---

# Validating buildings & the feedback loop

A React Arch building isn't done until it validates. There are two ways to get
the machine-readable diagnostics — a **CLI** and a **function** — use whichever
fits.

## 1. CLI — `react-arch check`

```bash
npx react-arch check src/House.tsx --json
```

Flags:

| Flag | Effect |
| --- | --- |
| `--json` | Print the combined report to **stdout** (parse this in an agent loop). |
| `--out <dir>` | Output dir for artifacts (default `react-arch-out`). |
| `--svg` | Also write a 2D plan `*.plan.svg`. |
| `--glb` | Also write a 3D `*.glb`. |
| `--brief <file.json>` | Check the model against a DesignBrief (see below). |

`check` always writes `*.model.json` (the canonical model) and `*.report.json`
(the report). **It exits non-zero when there are errors**, so it works directly
in CI / an agent's shell loop.

## 2. Function — `review()`

When you'd rather stay in code (no shell):

```ts
import { renderToDocument } from "@react-arch/react";
import { review } from "@react-arch/validation";
import { House } from "./House.js";

const doc = renderToDocument(<House />);
const report = review(doc); // structural + quality checks
if (!report.ok) console.log(report.diagnostics);
```

## Report shape (same from CLI `--json` and `review()`)

```jsonc
{
  "ok": true,                       // false if any error-severity diagnostic
  "counts": { "error": 0, "warning": 0, "info": 0 },
  "diagnostics": [
    {
      "severity": "warning",        // "error" | "warning" | "info"
      "code": "door-too-narrow",    // stable, machine-readable
      "entityKind": "opening",
      "entityId": "living-door-south-1",
      "message": "Door … is 0.7 m wide (min 0.8 m)",
      "fix": "Set width to at least 0.8 m.",   // actionable hint
      "data": { "widthM": 0.7, "minM": 0.8 }   // structured detail
    }
  ],
  "schedule": { "totalAreaM2": 120, "roomCount": 8, "floorCount": 2, "rows": [...] },
  "summary": { "name": "...", "floors": 2, "rooms": 8, "walls": 30, "openings": 12, "objects": 9 }
}
```

**Iterate on `code`, not message text.** Group diagnostics by `code`, apply the
`fix`, re-run, repeat until `ok` is `true` and warnings are gone.

## Diagnostic codes & how to fix them

### Structural — errors (must fix; `ok` stays false)
| Code | Meaning → fix |
| --- | --- |
| `schema` | Model failed schema shape — usually a malformed prop. |
| `duplicate-id` | Two entities share an `id` → make ids unique. |
| `bad-floor-ref` / `bad-building-ref` / `bad-wall-ref` | A reference points at a missing entity → fix the id. |
| `zero-length-wall` | A wall's `from`/`to` are equal → give it length. |
| `open-room` | A room polygon doesn't close → fix its walls. |
| `render` | The component threw while rendering → fix the JSX/runtime error. |

### Quality / code-compliance — usually warnings
| Code | Meaning → fix |
| --- | --- |
| `room-too-small` | Room below the min area for its `usage` → enlarge it (e.g. bedroom ≥7 m², kitchen ≥5 m²). |
| `room-no-egress` | A room has no door → add a `<Door>` on one of its walls. |
| `door-too-narrow` | Door < 0.8 m → widen it. |
| `opening-overlap` | Two openings overlap on a wall → space them apart. |
| `low-daylight` | Habitable room glazing below ~8% of floor area → add/enlarge windows. |
| `corridor-too-narrow` | Hallway < 0.9 m → widen it. |
| `overlapping-rooms` | Two room footprints overlap → reposition so they tile. |
| `unknown-material` | `materialId` not declared → add a `<Material>` with that id. |
| `invalid-stair-rise` | Riser outside ~10–22 cm → adjust `steps` or `rise` (aim ~17–19 cm). |
| `missing-stairs` | Multi-storey building with no vertical circulation → add `<Stairs>`. |
| `door-blocks-stair` | A door opens straight onto a flight/stairwell void → move the door or stair so there's a clear landing (~0.6 m). |

### Design-brief conformance (only with `--brief` / `review(doc, { brief })`)
| Code | Meaning → fix |
| --- | --- |
| `brief-missing-room` | A required room type is absent → add it. |
| `brief-room-too-small` | A room is under the brief's min area → enlarge it. |
| `brief-adjacency-missing` | Two rooms that should share a wall don't → reposition them adjacent. |
| `brief-floor-count` | Floor count ≠ the brief → add/remove a floor. |

## DesignBrief — give the agent a target to hit

A brief is a structured spec you validate the model against:

```json
{
  "rooms": [
    { "type": "kitchen", "count": 1, "minAreaM2": 6 },
    { "type": "bedroom", "count": 2, "minAreaM2": 9 }
  ],
  "adjacencies": [["kitchen", "dining"]],
  "floors": 2,
  "site": { "widthM": 12, "depthM": 9 }
}
```

```bash
npx react-arch check src/House.tsx --brief brief.json --json
```

## The loop (what an agent should do)

1. Write/edit the building.
2. `npx react-arch check src/House.tsx --json` (add `--brief` if you have one).
3. Parse `diagnostics`. If `ok` is false or warnings remain, apply each `fix` by
   `code` and go to 2.
4. Stop when `ok` is `true` with no warnings. Optionally export with `--svg`/`--glb`.
