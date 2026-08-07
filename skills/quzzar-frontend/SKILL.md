---
name: quzzar-frontend
description: Quzzar's frontend conventions — component structure, styling, state management, data fetching, forms, routing, accessibility, performance, and UI testing. Use before writing or changing any UI code, React components, styles, client-side state, or frontend tests.
---

# quzzar-frontend

Frontend conventions. Repo-wide rules (naming, commits, dependencies, file organization)
live in `quzzar-workplace` — read that too for anything not specific to the UI layer.

## When to use

- Writing or changing a component, page, layout, or route.
- Touching styles, design tokens, or theme.
- Adding client state, a data fetch, or a form.
- Writing or fixing a frontend test.

## First: read the repo's own context

`CLAUDE.md` at the repo root carries this repo's real framework, styling approach, and
commands. Repo facts beat anything general below. If it has no
`<!-- quzzar-skills:start -->` block, run the `setup` skill first.

## Tool choices are already decided

**Which** tool to use is fixed by the stack table in `quzzar-workplace` — React + Vite +
TypeScript, React Router, vanilla-extract, Jotai, React Query, nuqs, TanStack Table, React Hook
Form, Radix, Motion, Playwright, and the rest, each with its stated role and layer. Read it and
use what it says.

This skill covers **how** we use them. Don't re-litigate the tool; if you think something
outside the table is needed, that's an `AskUserQuestion`, not a judgment call.

## Handling unfilled sections

Sections marked `<!-- TODO -->` have no convention recorded yet.

**Do not invent one.** Match the nearest existing component, and name the missing convention
in your report. If existing components disagree and the choice matters, use `AskUserQuestion`.

---

## Routing

React Router (`react-router-dom`), host-aware — marketing and portal are distinct hosts.

<!-- TODO: Route file layout, and how the marketing/portal host split is expressed in code.
     - Where route definitions live, and whether they're colocated with features.
     - Loading and error boundary conventions per route.
     - Route-level code splitting: default, or on evidence? -->

## Component structure

<!-- TODO: One component per file, or colocated helpers?
     - Default export or named? Function declaration or arrow const?
     - Props: inline type or a named `Props` type? Destructured in the signature?
     - When a component gets split — prop count, line count, or responsibility?
     - The boundary between the shared UI package and app-local components: what earns
       promotion into the shared package, and what must stay in the app. -->

## Styling — vanilla-extract

Zero-runtime CSS-in-TypeScript. Styles compile to static CSS at build time via the Vite
integration, so no style engine ships to the browser. Class names, variables, and theme
contracts are all type-checked.

This is the same bargain as core guideline 4, applied to CSS: the compiler catches a bad token
reference instead of it silently rendering nothing. So don't route around it — a hardcoded
value or an inline `style` prop is a type check you've opted out of.

Reach for the API that fits:

- `style()` — a single style, with selectors, media queries, pseudo-classes.
- `styleVariants()` — a named set of variants.
- `createTheme()` / theme contracts — typed tokens.
- `createVar()` — CSS variables where a value must change at runtime.
- `globalStyle()` — sparingly, for genuinely app-wide rules.
- `createContainer()` — container queries.

<!-- TODO: Where `.css.ts` files live relative to their component.
     - The theme contract's location, and whether raw hex/px is ever acceptable.
     - Dark mode: how the theme class is applied (the theme is a Jotai atom — how do they meet?).
     - Responsive strategy — breakpoints in the contract, or container queries by default?
     - Sprinkles and Recipes: in use, or deliberately not? -->

## Design tokens

<!-- TODO: The token source of truth, and whether it's shared with the UI package.
     - What must come from a token rather than a literal. -->

## Component library — the shared UI package

Radix (`radix-ui`, unified pkg) provides the accessible primitives under the shared UI
package's interactive components. `three` must **never** appear inside it.

<!-- TODO: Are shared UI components treated as editable or as vendor?
     - When to reach for a primitive vs. build bespoke.
     - How variants are expressed (cva, props, or something else).
     - What makes a component eligible for the shared package in the first place. -->

## State

Three stores, three jobs — pick by where the state belongs, not by convenience:

- **Jotai** — global/client state, atomically (e.g. the theme atom). Also the storage layer,
  via `atomWithStorage`.
- **nuqs** — URL / query-string state. Filters, tabs, and pagination belong in the URL.
- **React Query** (`@tanstack/react-query`) — server state and data fetching, in the app's
  data layer.

Server state does not go in Jotai. URL-worthy state does not go in either.

<!-- TODO: Local `useState` first — at what point does state graduate to a Jotai atom?
     - When React context is justified vs. prop drilling vs. an atom.
     - Stance on `useEffect` — where it's legitimate and where it signals a mistake.
     - Derived state: computed inline or memoized? `useMemo`/`useCallback` threshold. -->

## Data fetching

React Query, in the app's data layer. Responses are **parsed through their Zod schema on
fetch** (core guideline 4) — never cast, never trusted. Schemas come from the shared schema
package.

Expect the `ApiResponse` shape from `quzzar-workplace` core guideline 5, and handle `fail`
(client's fault, per-field) separately from `error` (server's fault).

<!-- TODO: Query key conventions, and where they're defined.
     - Where a query/mutation hook lives relative to the component using it.
     - Loading and error UI: Suspense boundaries, skeletons, or inline states?
     - staleTime / gcTime defaults, and invalidation strategy.
     - Optimistic updates: standard or exceptional? -->

## Forms

React Hook Form, with `@hookform/resolvers` bridging to the Zod schema. **One schema, validated
on both ends** — the form and the server share it, from the shared schema package.

<!-- TODO: Where errors render, and how they're announced to screen readers.
     - Submit state, double-submit prevention, and success feedback (Sonner?).
     - Validation timing — on blur, on change, on submit? -->

## Tables

TanStack Table, headless — sorting, pagination, selection, and column logic.

<!-- TODO: Where column definitions live, and whether they're shared or per-view.
     - How table state interacts with nuqs — which parts belong in the URL?
     - Server-side vs. client-side pagination: the default, and when to switch. -->

## Animation

- **Motion** (`motion/react`) is the default — enter/exit, layout transitions, gestures,
  cursor-reactive grid and parallax.
- **GSAP** is narrow: SVG plugins (DrawSVG, MorphSVG, MotionPath) and heavy scroll-scrubbed
  timelines (ScrollTrigger). Don't reach for it when Motion covers the case.
- **Number Flow** for animated number transitions — metrics, counters.

<!-- TODO: Duration and easing defaults. Reduced-motion handling.
     - What must never animate. -->

## 3D

`three` (+ `@types/three`), lazy-chunked behind the panel that hosts it. Narrow scope: it is
**not** a general graphics tool, and it must **never** appear inside the shared UI package.

<!-- TODO: Resource disposal and teardown expectations on unmount.
     - Performance budget for the scene, and what to do when it's exceeded. -->

## Other UI tools

- **Vaul** — mobile bottom-sheet drawers (responsive dialogs).
- **Sonner** — toasts and transient notifications.
- **cmdk** — the command palette (⌘K). Wrap it in a Radix Dialog.
- **Headless Tree** (`@headless-tree/core` + `/react`) — folder/document trees: drag-move,
  inline rename, multi-select, keyboard nav. It's the `react-complex-tree` successor, so don't
  reintroduce the predecessor.
- **Lucide** (`lucide-react`) — the icon set.
- **dayjs** — dates and time.
- **lodash** (`lodash-es`) — deep clone/merge, `groupBy`, `keyBy`, `uniqBy`, and friends.

## Accessibility

<!-- TODO: The bar you hold — WCAG level, or a practical checklist?
     - Non-negotiables: semantic elements over divs, labels on every input,
       visible focus, keyboard reachability, contrast minimums.
     - When ARIA is appropriate vs. a sign Radix should have been used instead.
     - Whether a11y is checked automatically, and with what. -->

## Performance

Heavy things are lazy-chunked — a `three` scene behind its own panel is the reference case.

<!-- TODO: What you actually care about — LCP/INP/CLS budgets, or bundle size?
     - Images: the component/loader used, and required attributes.
     - Fonts: loading strategy.
     - Beyond the known-heavy cases, is code splitting default or on evidence?
     - Whether "don't optimize without a measurement" is the rule. -->

## Testing — Playwright

End-to-end against a running app, per `quzzar-workplace`.

<!-- TODO: Query priority — roles and labels over `data-testid`? Stance on test ids.
     - What's stubbed vs. real (network, time, auth session).
     - How canvas-based, lazy-chunked views like a `three` scene are handled.
     - Whether a UI change needs a spec to merge. -->

---

## Rules

- Tool selection is `quzzar-workplace`'s stack table, not a per-change decision. Something
  outside it is an `AskUserQuestion`.
- Fetched data is parsed through its Zod schema. No casting into shape.
- `CLAUDE.md` overrides this file — it holds the facts of the repo in front of you.
- Where this file is silent, the nearest existing component is the convention.
- Never present a rule from a `<!-- TODO -->` section as established.
- Deleting an unused component means deleting its styles, stories, tests, and exports too
  (core guideline 1).
