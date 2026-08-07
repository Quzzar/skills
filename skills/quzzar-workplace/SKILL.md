---
name: quzzar-workplace
description: Quzzar's core engineering guidelines and canonical tool stack — dead-code deletion, consolidation, when to ask for alignment, strict TypeScript and Zod, the standard API response shape, plus repo process (issues, branches, commits, PRs, CI). Read before starting any work in a quzzar repo.
---

# quzzar-workplace

The rules that hold everywhere, and the tools we build with. Read this before starting any
work; then read the skill for the layer you're touching.

- **Frontend things** — UI, components, styling, client state, forms: read `quzzar-frontend`.
- **Backend things** — APIs, server code, database, orchestration, edge functions: read
  `quzzar-backend`.

A change spanning both layers needs both.

## First: read the repo's own context

`CLAUDE.md` at the repo root carries this repo's real stack and commands, written by the
`setup` skill. Repo facts beat anything general in this skill — if they disagree, the repo
is right.

If `CLAUDE.md` has no `<!-- quzzar-skills:start -->` block, this repo hasn't been set up.
Run the `setup` skill first.

---

# Core guidelines — ALWAYS follow these

These five are non-negotiable. Everything below them is detail.

## 1. NEVER keep old, unused code

Git and GitHub exist for this. If something is no longer used or needed, **delete it** — don't
comment it out, don't rename it to `*.old`, don't leave it behind a dead flag, don't keep a
`legacy/` copy "just in case."

- Old code is recoverable from commit history. That is what history is for.
- Deleting isn't breaking. The change that replaces an old system *ships without it*, and it
  gets exercised at `dev` and `staging` before it reaches `main`.
- Dead code found while working on something else is a finding: say so, and delete it as part
  of the release that supersedes it.

When you delete, delete completely: the code, its tests, its types, its exports, its config
entries, and any now-orphaned dependency.

## 2. CONSOLIDATE repetition

If what you're building repeats something that already exists elsewhere, stop and investigate
before you duplicate it. This shows up most when expanding an existing system.

The procedure:

1. **Spin up a subagent** to investigate the overlap — where else this pattern lives, how much
   is genuinely shared, and what the seam would be.
2. **Then ask, with `AskUserQuestion`.** Present the finding and real options: consolidate into
   one thing, keep them separate and why, or extract only the shared part.

Do not silently consolidate, and do not silently duplicate. Both are decisions that belong to
the user.

## 3. Ask with the question tool, not in prose

When you need alignment or have a real question, you **MUST** use the `AskUserQuestion` tool —
the UI tool that renders selectable options. Give a few distinct options to pick from.

This is the preferred way to resolve ambiguity. Reach for it early rather than guessing and
building the wrong thing.

## 4. Strict types, Zod schemas, clean documented code

- **Full strict TypeScript**, always. Every strict flag on.
- **Zod for every schema.** Objects are **parsed** through their Zod schema on fetch — validated
  at the boundary, not trusted and cast later.
- **Always explicit types. No implicit types.**
- **`as` casts should be rare.** A cast is a signal the types are wrong somewhere upstream —
  fix that instead. If a cast is genuinely unavoidable, say why.
- **JSDoc-style documentation** on the code we write. Clean, readable, documented.

## 5. Standard API response shape

Every API response follows this:

```ts
type ApiResponse<T, F = Record<string, string>> =
  | { status: "success"; data: T }
  | { status: "fail"; data: F }                    // client's fault: validation, business rules
  | { status: "error"; message: string; code?: string; data?: unknown }; // server's fault
```

The distinction matters and isn't optional:

- `fail` — **the client's fault.** Validation errors, business-rule rejections. `data` carries
  the per-field reasons.
- `error` — **the server's fault.** Something broke on our side. `message` is safe to surface;
  never leak internals through it.

---

# The stack

These are our main solutions to problems — the core tools we should be building with. Reach
for something outside this table only with a reason.

## Frontend — the base of every web app

| Tool | Role |
| --- | --- |
| **React + Vite + TypeScript** | UI · dev/build · types. The base of every web app, both layers. |
| **React Router** (`react-router-dom`) | Client-side routing, host-aware: marketing vs. portal. |
| **vanilla-extract** | Styling, both layers. Zero-runtime CSS-in-TypeScript — styles compile to static CSS at build time, so nothing ships a style engine to the browser. Type-safe class names, variables, and theme contracts, via the official Vite integration. |

## Frontend · app

| Tool | Role |
| --- | --- |
| **Jotai** | Global/client state (atomic — e.g. the theme atom). Also the storage layer via `atomWithStorage`. |
| **React Query** (`@tanstack/react-query`) | Server state + data fetching. Lives in the app's data layer. |
| **nuqs** | URL / query-string state — filters, tabs, pagination reflected in the URL. |
| **TanStack Table** | Headless tables — sorting, pagination, selection, column logic. |
| **React Hook Form** (`@hookform/resolvers`) | Forms. The resolver bridges RHF to the Zod schema. |
| **three** (`three` + `@types/three`) | 3D rendering, lazy-chunked behind the panel that hosts it. Narrow scope — not a general graphics tool, and **never inside the shared UI package**. |

## Frontend · ui

| Tool | Role |
| --- | --- |
| **Radix** (`radix-ui`, unified pkg) | Accessible primitives under the shared UI package's interactive components. |
| **Motion** (`motion/react`) | Default animation — enter/exit, layout transitions, gestures, cursor-reactive grid + parallax. |
| **GSAP** | Narrow: SVG plugins (DrawSVG, MorphSVG, MotionPath) and heavy scroll-scrubbed timelines (ScrollTrigger). |
| **Vaul** | Mobile bottom-sheet drawers (responsive dialogs). |
| **Sonner** | Toasts / transient notifications. |
| **cmdk** | Command palette (⌘K) — wrap in a Radix Dialog. |
| **Number Flow** | Animated number transitions (metrics, counters). |
| **Headless Tree** (`@headless-tree/core` + `/react`) | Folder/document trees — drag-move, inline rename, multi-select, keyboard nav. The `react-complex-tree` successor. |
| **Lucide** (`lucide-react`) | Icon set. |

## Everywhere

| Tool | Role |
| --- | --- |
| **Zod** | Every cross-boundary schema — wire shapes, intake, agent contracts, and form validation via the RHF resolver. Lives in a shared schema package. **One schema, validated on both ends.** |
| **Playwright** | Testing. The test tool of record — end-to-end against a running app. |
| **dayjs** | Dates and time. |
| **lodash** (`lodash-es`) | General utilities — deep clone/merge, `groupBy`, `keyBy`, `uniqBy`… |

## Backend

| Tool | Role |
| --- | --- |
| **bun** | Runtime + package manager for service packages and the workspace root. **Never npm.** |
| **Temporal** | Durable orchestration for long-running pipelines. One task queue per pipeline, each with its own worker entry point so a local `pkill` can't cross-kill the others. |
| **Deno** | The edge-function runtime (Supabase functions). **Not bun.** |
| **Supabase** | Postgres (control plane + RLS), Auth, Storage. The whole managed backend — **don't add a second datastore or auth vendor.** |
| **GitHub Actions** | Hosts the runner today (free tier, ~2000 min/mo). An always-on VM is the upgrade path. |
| **Langfuse** | Tracing + quality scores for pipeline runs. |

---

# Process

## Environments and promotion

Three branches, promoted in order:

```
dev  →  staging  →  main
```

That's the whole release process. There's no separate versioning ceremony — a change ships by
being promoted along that path.

This is what makes core guideline 1 safe: deleting an unused system isn't risky, because the
deletion travels the same path as any other change and gets exercised at each stop before it
reaches `main`.

---

# Repo-wide code rules

Types, schemas, documentation, and the API response shape are covered in
[Core guidelines](#core-guidelines--always-follow-these) — they are not repeated here.

## Naming

| Thing | Casing | Example |
| --- | --- | --- |
| Component file | `PascalCase.tsx` | `SettingsPanel.tsx`, `UserAvatar.tsx` |
| Non-component module | lowercase — one word, or `kebab-case` | `format.ts`, `theme.ts`, `asset-layer.ts`, `user-manager.ts` |
| Directory | lowercase — one word, or `kebab-case` | `atoms`, `routes`, `site-view`, `vector-db` |
| Type / interface | `PascalCase` | `LoadState`, `FieldSpec` |
| Value / function | `camelCase` | `resolveSite`, `queryClient` |
| True constant | `SCREAMING_SNAKE_CASE` | `HEADER_H`, `PROJECT_SECTIONS`, `SUPABASE_URL` |
| Jotai atom | `camelCase` + `Atom` suffix | `themeAtom`, `selectionAtom` |
| Boolean | `is` / `has` / `can` / `should` prefix | `isLoading`, `hasOutput`, `canDrag`, `shouldCreateUser` |

Prefer `type` over `interface` — that's the dominant choice in practice, and `type` covers
unions, which the result types in [Error handling](#error-handling) need.

Spell words out. `resolveSite`, not `resSite`. The exceptions are terms already standard in the
domain or the platform (`url`, `id`, `db`, `otp`, `ui`).

**camelCase filenames are wrong, including where you find them.** Existing code has some —
`resolveSite.ts`, `siteModels.ts` sitting next to `asset-layer.ts` in the same directory. That's
drift, not a second convention. Write `resolve-site.ts`.

This is the one place where "match the surrounding code" does **not** apply. A wrong neighbour
is not a precedent — matching it is how the drift spread in the first place. Directories are
`kebab-case` for the same reason; `snake_case` directories in older code are drift too.

## File organization

**Top level of `src/` is by layer.** One lowercase directory per concern:

```
src/
  atoms/       client state
  auth/        session and route guards
  components/  shared UI
  data/        server state — queries and mutations
  lib/         configured clients and adapters
  routes/      pages
  utils/       pure helpers
```

**Feature grouping happens inside a layer, not above it.** A feature that grows past a couple of
files gets a subdirectory in the layer it belongs to — `components/site-view/`, not a top-level
`site-view/` holding its own atoms, queries, and components.

Shared UI packages organize by component instead: one lowercase directory per component
(`button/`, `accordion/`, `input-cards/`), with named exports barrelled through `src/index.ts`.

A file earns promotion into a shared package when a second surface needs it — not when you
anticipate one might.

## Comments

JSDoc and comment blocks. Documented code is core guideline 4 — this is the form it takes.

## Error handling

**Return a result type.** A function that can fail says so in its return type, so the caller
has to deal with it — the same discriminated-union shape as `ApiResponse` in core guideline 5,
scaled down to whatever the function needs.

`throw` is a last resort, not the default. It's invisible in the signature and turns every
caller into a maybe-handler. Reach for it only where there's genuinely nothing to return to.

**Log.** Logging is wanted, not tolerated — a failure that's handled quietly still gets
recorded. Swallowing an error without a log is the thing to avoid.

## Formatting and lint

**ESLint** for linting, **Prettier** for formatting. Their output is the answer — don't
hand-format around them, and don't argue with a rule in review when it can be configured
instead.

Run `eslint-config-prettier` so ESLint stops enforcing the stylistic rules Prettier owns.
Without it the two fight over the same lines and every save flips the file back and forth.

---

# Rules

- The five core guidelines are not negotiable and not context-dependent. Apply them without
  being asked.
- `CLAUDE.md` overrides this file. This skill holds durable preferences that travel between
  repos; `CLAUDE.md` holds the facts of the repo in front of you.
- Where this file is silent, the surrounding code is the convention. Match it and say so.
- Where this file is **not** silent, it beats the surrounding code. Existing code that breaks a
  rule recorded here is drift to be corrected, not a local convention to be matched.
- Never report a convention as established when it came from a `<!-- TODO -->` section.
