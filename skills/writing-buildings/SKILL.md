---
name: writing-buildings
description: >-
  Use when writing or editing a building in React Arch (architecture-as-code):
  the component contract, the metric coordinate system, rooms/walls/doors/
  windows/stairs/roofs, reusable modules, and an annotated example. Pairs with
  the validating-buildings skill for the check-and-fix loop.
---

# Writing buildings in React Arch

You author a building as a React component tree and export it as a function the
Studio (or `check`) can render.

```tsx
import { Building, Floor, Room, Door, Window, Fixture } from "@react-arch/react";

export function House() {
  return (
    <Building name="My House" units="metric">
      <Floor id="ground" name="Ground Floor" elevation={0} height={2.8}>
        <Room id="living" name="Living Room" x={0} y={0} width={5} depth={4}>
          <Door wall="south" offset={1} width={0.9} height={2.1} />
          <Window wall="west" offset={2} width={2.2} height={1.4} />
          <Fixture type="sofa" x={2.5} y={3} />
        </Room>
      </Floor>
    </Building>
  );
}
```

## Coordinate system (read this first)

- Everything is in **metres**.
- **Plan X increases to the right; plan Y increases DOWN** (screen coordinates,
  not math). A room at `x={0} y={0}` is top-left.
- A `<Room>` is an axis-aligned box: `x,y` is its top-left corner, `width` runs
  along X, `depth` runs along Y.
- Floors stack vertically by `elevation` (metres). Floor N+1 usually sits at
  `elevation = previous.elevation + previous.height`.
- In 3D this maps: plan X → world X, plan Y → world Z, height → world Y (up).

## Component contract

### `<Building>` — root
`name`, `units` (`"metric"` | `"imperial"`), optional `id`. Children: `<Floor>`,
and optionally one or more `<Roof>`.

### `<Floor>` — a storey
`id`, `name`, `elevation` (m), `height` (m). Children: `<Room>`, `<Wall>`,
`<Stairs>`, `<Material>`.

### `<Room>` — an axis-aligned space (auto-generates 4 perimeter walls)
`id`, `name`, `x`, `y`, `width`, `depth`. Optional: `height`, `wallThickness`
(default 0.2), `floorMaterial`, `ceilingMaterial`, `usage`.
Children: `<Door>`, `<Window>`, `<Opening>`, `<Fixture>`.

`usage` drives the quality checks — use one of: `living`, `kitchen`, `bedroom`
/`sleeping`, `dining`, `office`, `bathroom`/`wet`, `corridor`/`hallway`,
`garage`. (Min-area and daylight rules key off this.)

Adjacent rooms that share an edge get their coincident walls **merged**
automatically, so a `<Door>` on the shared wall becomes a real passage between
both rooms.

### `<Door>`, `<Window>`, `<Opening>` — attach to a room side
Placed **inside** a `<Room>`. `wall` (`"north"`|`"south"`|`"east"`|`"west"`),
`offset` (m along that wall from its start corner), `width`, `height`,
`sillHeight` (windows). `<Opening>` is a hole with no leaf.

### `<Wall>` — explicit wall (when rooms aren't enough)
`from={[x,y]}`, `to={[x,y]}`, `thickness`, `height`, `materialId`. A direct child
of `<Floor>`.

### `<Stairs>` — a straight flight (direct child of `<Floor>`, NOT a Room)
`at={[x,y]}` (bottom-start corner), `width`, `run` (plan length), `rise`
(defaults to the floor height so it reaches the storey above), `direction`
(`"north"`|`"south"`|`"east"`|`"west"` or degrees), `steps`, `materialId`.
React Arch cuts a stairwell void in the slab above automatically.

### `<Roof>` — a covering (direct child of `<Building>`, NOT a Floor)
`type` (`"flat"`|`"gable"`|`"hip"`), `pitch` (deg, gable/hip), `overhang` (m),
`thickness` (flat), `floorId` (cap a specific floor; defaults to the top floor),
`materialId`. Hidden by default in the Studio (toggle it on).

### `<Fixture>` / `<Furniture>` — objects (inside a `<Room>`)
`type` (e.g. `"sofa"`, `"bed"`, `"sink"`, `"toilet"`, `"desk"`), `x`, `y`,
optional `z`, `rotation`, `scaleX/Y/Z`.

### `<Material>` — a PBR material (child of `<Floor>` or `<Building>`)
`id`, `name`, `category`, `baseColor`, `roughness`, `metalness`, `opacity`,
`textureUrl`. Reference it via `materialId` / `floorMaterial` props.

## Reusable modules (do this instead of copy-paste)

A module is just a function returning a `<Room>` (or fragment). Parameterize
position so you can place it anywhere:

```tsx
function Bathroom({ id, x, y }: { id: string; x: number; y: number }) {
  return (
    <Room id={id} name="Bathroom" x={x} y={y} width={2.4} depth={2.2} usage="wet">
      <Door wall="south" offset={1.2} width={0.8} height={2.1} />
      <Fixture type="toilet" x={x + 0.5} y={y + 1.7} />
      <Fixture type="sink" x={x + 1.5} y={y + 1.7} />
      <Fixture type="shower" x={x + 0.6} y={y + 0.6} />
    </Room>
  );
}
```

Generate repetitive layouts with normal JS (`.map()`), e.g. an apartment block
of N floors × M units.

## Gotchas that cause validation failures

- **Multi-storey buildings need `<Stairs>`** or you get `missing-stairs`.
- **Keep a landing in front of doors near a stair** — a door right at a flight or
  stairwell void triggers `door-blocks-stair`. Give ~0.6 m+ clearance.
- **`<Stairs>` goes in `<Floor>`, `<Roof>` goes in `<Building>`** — nesting them
  in a `<Room>` means they're ignored.
- **Don't overlap rooms** — footprints must tile, not overlap (`overlapping-rooms`).
- **Doors need to be wide enough** (≥0.8 m) and rooms big enough for their
  `usage` — see validating-buildings for every rule.

When the building looks right, hand off to the **validating-buildings** skill and
run `react-arch check` until it's clean.
