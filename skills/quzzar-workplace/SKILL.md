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
for something outside this table only with a reason, and see [Dependencies](#dependencies).

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

Sections below marked `<!-- TODO -->` have no convention recorded yet.

**When you hit one: do not invent a rule.** Follow what the surrounding code or history already
does, and say plainly which convention was missing. If existing practice is inconsistent and
the choice matters, use `AskUserQuestion` (see core guideline 3).

An invented convention is worse than no convention — it gets applied confidently, spreads, and
looks deliberate.

## Issue tracking

<!-- TODO: Which tracker, and how you actually use it.
     - Where does work originate — issue first, or code first?
     - Required fields/labels on a new issue. Triage labels.
     - Does a branch or PR need to reference an issue? How? -->

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

<!-- TODO: What gates each promotion — automatic on merge, or manual?
     - Which environment each branch deploys to.
     - Hotfix path when something needs to reach main without waiting. -->

## Branches

Work branches off `dev` and merges back into it.

<!-- TODO: Naming pattern for work branches.
     - Rebase or merge to stay current with dev?
     - Deleted on merge? -->

## Commits

<!-- TODO: Conventional commits, or prose? Subject length and mood.
     When does a body get written? Squash before pushing, or keep history? -->

## Pull requests

<!-- TODO: What a PR must contain before review. Description structure.
     Screenshots for UI changes? Max size before it must be split?
     Who reviews, how many approvals, can you self-merge? -->

## CI and merging

<!-- TODO: What must be green before merge, and what's advisory.
     Merge strategy. How releases and deploys are triggered. -->

## Testing

Playwright, end-to-end against a running app.

<!-- TODO: Where Playwright specs live and how they're named.
     - What must have a spec before merge, and what needn't.
     - Which environment specs run against in CI — and whether they gate promotion.
     - Whether anything is unit-tested below the e2e layer, or Playwright is the whole story. -->

---

# Repo-wide code rules

Types, schemas, documentation, and the API response shape are covered in
[Core guidelines](#core-guidelines--always-follow-these) — they are not repeated here.

## Naming

<!-- TODO: Casing for files, directories, exports, types, constants, booleans.
     Abbreviations: allowed or spelled out?
     Domain terms with a fixed spelling — glossary here or linked. -->

## File organization

<!-- TODO: How a feature's files are grouped — by layer or by feature?
     Where shared code lives, and the bar for promoting into it.
     Barrel files: used or banned? -->

## Comments

JSDoc-style documentation, per core guideline 4.

<!-- TODO: Beyond JSDoc — is it required on every public export, or only non-obvious ones?
     Stance on inline comments (why, not what?).
     TODO comment format, and whether they need a linked issue. -->

## Error handling

<!-- TODO: Throw vs. return a result type, and where each applies.
     Custom error classes — do they exist, where do they live?
     What must never be swallowed. Rules for `console.*` vs. a real logger.
     Note: the client-facing shape is already fixed by core guideline 5. -->

## Dependencies

[The stack](#the-stack) is the dependency policy. Prefer a tool already in that table over a
new package, and prefer the standard library or a small local utility over a dependency that
earns its place only once.

<!-- TODO: The bar for adding something NOT in the table — who approves, and what justifies it.
     Anything explicitly banned beyond `npm` and a second datastore/auth vendor.
     Pinned exact or ranged? Who handles upgrades? -->

## Formatting and lint

<!-- TODO: The tool of record, and whether its output is final.
     Rules deliberately disabled, and why. Is a lint warning acceptable to merge? -->

## Testing

See [Testing](#testing) under Process. Layer-specific rules live in `quzzar-frontend` and
`quzzar-backend`.

---

# Rules

- The five core guidelines are not negotiable and not context-dependent. Apply them without
  being asked.
- `CLAUDE.md` overrides this file. This skill holds durable preferences that travel between
  repos; `CLAUDE.md` holds the facts of the repo in front of you.
- Where this file is silent, the surrounding code is the convention. Match it and say so.
- Never report a convention as established when it came from a `<!-- TODO -->` section.
