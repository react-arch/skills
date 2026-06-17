---
name: react-arch
description: >-
  Use when creating or editing buildings, floor plans, or architecture as code
  with React Arch. Covers the whole loop: scaffold a project, write the building
  in JSX, validate it with the CLI, read the diagnostics, fix, and export. Start
  here, then load the writing-buildings and validating-buildings skills.
---

# React Arch

React Arch is a declarative framework for describing buildings as code. You
write a building as a React component tree (`<Building><Floor><Room>…`), and
React Arch turns it into a typed model it can **validate, visualize, and
export**. The code is the single source of truth — the Studio is a read-only
visualizer, not an editor (think Remotion, not CAD).

The product thesis: **AI writes architecture-as-code; React Arch validates,
visualizes, compares, and exports it.** These skills are the contract for doing
that well.

## The workflow you should follow

1. **Scaffold** a project (once):
   ```bash
   npx create-react-arch-app my-house
   cd my-house && npm install
   ```
   This gives you `src/House.tsx` (the building) and `src/main.tsx` (mounts the
   Studio). `npm run dev` opens the visualizer with live reload.

2. **Write** the building in `src/House.tsx`. Load the **writing-buildings**
   skill for the component contract, coordinate system, and patterns.

3. **Check** it — this is the agent feedback loop:
   ```bash
   npx react-arch check src/House.tsx --json
   ```
   It renders the building, runs every validation, and prints a machine-readable
   report. Load the **validating-buildings** skill to read the report and fix
   each diagnostic code.

4. **Iterate** — fix diagnostics, re-run `check`, until the report's `ok` is
   `true` (no errors). Warnings are quality/code-compliance issues worth fixing.

5. **Export** when done: `check` writes `*.model.json` (and `*.plan.svg` with
   `--svg`). The Studio's top bar exports JSON / SVG / GLB.

## Package map

| Package | Use |
| --- | --- |
| `@react-arch/react` | The components you author with, plus `renderToDocument()`. |
| `@react-arch/validation` | `review()` — run the full validation pass in code. |
| `@react-arch/studio` | The `<Studio>` visualizer component. |
| `react-arch` (CLI) | `react-arch studio` (visualize) and `react-arch check` (validate/export). |
| `create-react-arch-app` | `npx create-react-arch-app <name>` scaffolder. |

## Rules of thumb

- Coordinates are **metres**; plan X increases right, plan Y increases **down**.
- A building that doesn't validate is not done. Always end with a clean `check`.
- Prefer reusable module components (a `Bathroom()` that returns a `<Room>`) over
  copy-pasting rooms — see writing-buildings.
- Don't try to make the Studio edit the model; change the code instead.

Next: load **writing-buildings** to author, then **validating-buildings** to
close the loop.
