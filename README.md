# React Arch — Agent Skills

Installable [agent skills](https://skills.sh) for [React Arch](https://github.com/react-arch/react-arch),
the framework where **AI writes architecture-as-code and React Arch validates,
visualizes, compares, and exports it**.

These skills teach an agent the React Arch contract: how to author a building,
run the validation feedback loop, and iterate until it's clean.

## Install

Install all skills (into Claude Code, Cursor, Codex, and 65+ other agents):

```bash
npx skills add react-arch/skills --all
```

Or pick specific skills / a specific agent:

```bash
npx skills add react-arch/skills --list
npx skills add react-arch/skills --skill writing-buildings -a claude-code
```

## Skills

| Skill | What it teaches |
| --- | --- |
| [`react-arch`](skills/react-arch/SKILL.md) | The overview + the full loop: scaffold → write → check → fix → export. Start here. |
| [`writing-buildings`](skills/writing-buildings/SKILL.md) | The authoring contract: components, coordinate system, rooms/walls/doors/windows/stairs/roofs, reusable modules, an annotated example. |
| [`validating-buildings`](skills/validating-buildings/SKILL.md) | The feedback loop: `react-arch check`, the JSON report, the `review()` function, every diagnostic code with its fix. |

## How they're meant to be used

1. `npx create-react-arch-app my-house`
2. Write the building in `src/House.tsx` (writing-buildings).
3. `npx react-arch check src/House.tsx --json` and fix each diagnostic
   (validating-buildings) until the report's `ok` is `true`.
4. Export JSON / SVG / GLB.

## License

MIT
