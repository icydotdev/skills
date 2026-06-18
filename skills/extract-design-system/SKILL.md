---
name: extract-design-system
description: Infer a React project's implicit design system and scaffold a structured ui/ folder — unified components, design tokens, Storybook stories, and accessibility + unit tests — while a live Lumen dashboard visualises progress. Use when the user asks to extract, infer, scaffold, organise, or set up a design system / component library / UI tokens from an existing React or Next.js codebase.
---

# Extract Design System

You infer the design system already hiding in a React/Next.js codebase and
scaffold a clean `ui/` folder, while the **Lumen** dashboard shows it happening
live. You do the thinking; Lumen is the engine + visualiser.

## 0. Start the dashboard (do this first)

Run the Lumen engine **in the background** so it keeps serving while you work:

```bash
npx -y @icydotdev/lumen@latest --serve
```

- Run it backgrounded (don't block on it). It prints a banner like
  `✨ Lumen ready · http://localhost:3719`. **Read the port from that banner** —
  it may differ from 3719 if the port was busy. Use that base URL (call it `$API`)
  for every POST below.
- It opens the dashboard in the browser on a loading screen. It fills in as you
  POST progress. If it errors with "React was not found", tell the user this
  isn't a React project and stop.

## 1. Announce the scan

```bash
curl -s -X POST $API/api/scan-start
```

## 2. Find the components

Search the codebase for React components: `**/*.tsx` and `**/*.jsx`.
Ignore `node_modules`, `.next`, `dist`, `build`, `.git`, `coverage`, and the
`ui/` folder. A component = a file exporting a PascalCase function/const that
returns JSX. In a monorepo, focus on the app/package where the components live.

## 3. For EACH component — analyse, post, write

a) **Analyse**: infer variant names, props, and the colour/spacing values it
   uses. If several near-identical components are really one component with
   variants, **unify them** into one.

b) **Post it** so a row animates into the dashboard table (full ComponentInfo):

```bash
curl -s -X POST $API/api/component -H 'content-type: application/json' -d '{
  "name": "Button",
  "filePath": "src/components/Button.tsx",
  "variants": ["primary","ghost"],
  "props": [{"name":"variant","type":"\"primary\" | \"ghost\"","optional":true}],
  "colorValues": ["#3b82f6"],
  "spacingValues": ["p-4"],
  "hasTests": false, "hasStory": false,
  "a11yScore": null, "testCoverage": null
}'
```

c) **Mark generating → write files → mark generated**:

```bash
curl -s -X POST $API/api/generating -H 'content-type: application/json' -d '{"name":"Button"}'
# ...write the files (step 5)...
curl -s -X POST $API/api/generated  -H 'content-type: application/json' -d '{"name":"Button"}'
```

## 4. Name tokens & flag inconsistencies

Cluster the colour / spacing / typography values you collected. Give colours
**semantic** names from how they're used (`primary`, `background`, `muted`,
`danger`…), never generic ones. POST each token:

```bash
curl -s -X POST $API/api/token -H 'content-type: application/json' \
  -d '{"kind":"colors","name":"primary","value":"#3b82f6"}'
# kind = colors | spacing | typography | borderRadius | shadows
```

When you find near-duplicates (e.g. six slightly different greys), POST an
inconsistency so it shows amber in the dashboard:

```bash
curl -s -X POST $API/api/inconsistency -H 'content-type: application/json' \
  -d '{"type":"color","message":"6 near-duplicate greys unified as muted","values":["#999","#9a9a9a"],"suggestedToken":"muted"}'
```

## 5. Write files to ui/ (idiomatic — NOT placeholder stubs)

Detect the styling approach (Tailwind / CSS Modules / styled-components) from the
project and match it. Per component, create `ui/components/<Name>/`:

- `<Name>.tsx` — unified component, variants as props, real styling
- `<Name>.stories.tsx` — one Storybook story per variant + meaningful edge cases
- `<Name>.test.tsx` — **behaviour** tests (interactions, not just "it renders")
  plus an a11y test via `jest-axe`:

  ```tsx
  import { render, screen } from "@testing-library/react";
  import { axe } from "jest-axe";
  // test real interactions + expect(await axe(container)).toHaveNoViolations();
  ```

- `index.ts` — re-export

Plus, once, at `ui/`:

- `tokens.ts` — raw token values as a typed `const`
- `theme.ts` — Tailwind config extension object / ThemeProvider theme /
  CSS custom properties, matching the styling approach
- `lumen.config.ts` — `{ stylingApproach, testRunner, hasStorybook }`
- `lib/cn.ts` — a small className combiner (Tailwind projects only)

`ui/` is **net-new and additive**. Do NOT modify the user's existing files
unless they explicitly ask you to replace usages.

## 6. Finish

- Add to the user's `package.json` scripts:
  `"lumen": "npx @icydotdev/lumen --dashboard"`
- POST the summary (flips the dashboard to "done"):

```bash
curl -s -X POST $API/api/complete -H 'content-type: application/json' \
  -d '{"componentCount":12,"tokenCount":18,"inconsistencyCount":3}'
```

- Tell the user it's done, and that they can re-open the dashboard anytime with
  `npm run lumen`.

The dashboard server keeps running so the user can keep viewing it; mention they
can stop it with `lsof -ti:3719 | xargs kill` (use the real port if different).

## Tips

- POST components one at a time, as you finish analysing each, so the user
  watches the table populate in realtime — that's the point.
- Quality over speed: tokens named from real usage, tests that test behaviour,
  components that are genuinely unified. That's the value Lumen can't compute on
  its own.
