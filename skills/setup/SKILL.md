---
name: setup
description: Set up a repo to use the quzzar skills — survey the existing stack and practices, write the skill-routing block into CLAUDE.md, install Matt Pocock's skills, then report where the codebase falls short of quzzar-workplace standards. Use when the user runs /setup or asks to set up the quzzar skills in a repo.
---

# setup

Point this repo's agents at the `quzzar-*` skills, install the supporting toolkit, then report
where the codebase doesn't yet meet those standards.

Run once per repo. Re-running is safe — every write below is idempotent.

## When to use

- The user runs `/setup`, or asks to set up / configure the quzzar skills in a repo.

Do **not** use this for general project setup — installing dependencies, scaffolding an app,
configuring CI. This skill wires up the quzzar skills and nothing else.

---

## Step 1 — Survey what's already here

Read the repo before writing anything to it. You need this both for the CLAUDE.md block and
for the gap report in Step 4.

**Read the `quzzar-workplace` skill first, and let its section list drive the survey.** Survey
the dimensions it actually holds a standard on — if it has a rule about barrel files, look at
whether the repo uses them; if it says nothing about them, don't spend the reading budget.
That way Step 4's comparison lines up one-to-one instead of guessing which findings matter.

**Tooling** — what the repo already uses:

| Fact | Where to look |
| --- | --- |
| Package manager | Lockfile: `pnpm-lock.yaml`, `package-lock.json`, `yarn.lock`, `bun.lockb` |
| Layout | `pnpm-workspace.yaml`, `turbo.json`, `workspaces` in `package.json` |
| Frontend | `dependencies` — `next`, `react`, `vue`, `svelte`, `astro` |
| Backend / runtime | `wrangler.jsonc`, `vercel.json`/`vercel.ts`, `hono`, `express`, `fastify` |
| Database / ORM | `drizzle.config.*`, `prisma/`, `pg`, `d1_databases` in wrangler config |
| Styling | `tailwind.config.*`, `components.json` (shadcn), `*.module.css` |
| Tests | `vitest`, `jest`, `playwright`, and the `scripts` block |
| Typecheck / lint / format | `scripts`; `tsconfig.json` `strict`; `biome.json`, `.eslintrc*`, `.prettierrc*` |
| CI | `.github/workflows/` — what runs, and what blocks merge |
| Issue tracker | `.github/ISSUE_TEMPLATE/`, Linear-style branch names in `git log` |

Record the **real commands** from the `scripts` block, verbatim. `pnpm test` and
`pnpm run test:unit` are not interchangeable.

The tooling table above is surveyed regardless — `CLAUDE.md` needs it whether or not
`quzzar-workplace` has an opinion.

**Practices** — what the code actually does today, not what it should do. This is the
baseline the gap report measures against. Cover the `quzzar-workplace` dimensions that have
a standard recorded, and use these as the reading strategy:

- Read `git log --oneline -30` and `git branch -a` for the commit and branch conventions
  in current use. Note if two competing styles are visible.
- Skim a representative handful of files across both layers. Note how they handle naming,
  file organization, error handling, validation, and types.
- Note existing agent config: `CLAUDE.md`, `AGENTS.md`, `.cursor/rules`, `CONTEXT.md`.
- Note what's inconsistent. Inconsistency is the cleanup surface — a convention followed
  in nine files and broken in one is a finding.

Report the survey to the user before moving on. Keep it to a short list of findings, not a
file-by-file dump.

---

## Step 2 — Write the routing block into CLAUDE.md

The block tells every future agent session which skill to read for which work. This is the
core of what `setup` exists to do.

### Where it goes

- **No `CLAUDE.md`** — create it with the block as the entire contents.
- **`CLAUDE.md` exists** — insert the block at the **top**, so it's read first. If the file
  opens with an H1 (`# Project Name`), insert immediately *after* that heading rather than
  above it; the title should stay first. Otherwise insert at line 1.
- **The markers are already present** — replace the content between them in place. Never
  add a second copy.
- Preserve everything else in the file byte for byte. This is an insertion, not a rewrite.

If `CLAUDE.md` already exists, show the user the exact insertion and get confirmation before
writing. It's a file they maintain by hand.

### The block

Write this verbatim, filling the Stack list from your Step 1 survey:

```markdown
<!-- quzzar-skills:start -->
## Agent skills

**Before starting any work in this repo, read the `quzzar-workplace` skill.** It holds the
conventions for issues, branches, commits, PRs, and review, plus the code rules that apply
at every layer — naming, file organization, comments, types, error handling, dependencies.

Then, before writing code, read the skill for the layer you're touching:

- **`quzzar-frontend`** — UI, components, styling, client state, data fetching, forms,
  accessibility, frontend tests.
- **`quzzar-backend`** — API routes, server code, database queries, migrations, validation,
  auth, background jobs, backend tests.

Read the layer skill *before* writing, not as a check afterwards. A change spanning both
layers needs both.

### Stack

<!-- filled in by /setup from the repo itself — correct it by hand if it drifts -->

- Package manager:
- Frontend:
- Backend / runtime:
- Database:
- Test:
- Typecheck:
- Lint:
<!-- quzzar-skills:end -->
```

Keep the Stack list to what an agent needs to act. `CLAUDE.md` is loaded into every session,
so length here costs context on every request — omit a line rather than writing `n/a` for it.

Verify the commands you recorded actually run before you write them down. A wrong test
command in `CLAUDE.md` misleads every future session.

---

## Step 3 — Install Matt Pocock's skills

They're part of the standard toolkit for a quzzar repo — TDD, code review, diagnosing bugs,
domain modeling, and the rest. Install them so they're available alongside the quzzar skills.

Check what's present first — `claude plugin list`, and look for `mattpocock-skills` in the
available skills. Then:

```bash
claude plugin marketplace add mattpocock/skills
```

```bash
claude plugin install mattpocock-skills@mattpocock
```

If it's already installed, update instead — he adds skills across minor versions, so an older
install is missing some:

```bash
claude plugin update mattpocock-skills
```

**A plugin install or update needs a session restart to take effect.** His skills won't be
available for the rest of this session, so don't attempt to use one — tell the user to restart
and note that the toolkit lands then. Step 4 doesn't depend on them, so setup still finishes
in full.

Worth mentioning in your final report: `/setup-matt-pocock-skills` configures the issue
tracker and domain-doc layout that several of his skills expect. It's user-invoked, so
recommend it as a next step rather than trying to run it.

---

## Step 4 — Report where the codebase falls short

Now compare the Step 1 survey against the standard, and report the gaps. You do this
yourself — nothing here is handed off.

1. Re-read the `quzzar-workplace` sections you surveyed against. That's the standard.
2. For each one, state what the standard says and what the code actually does. A match needs
   one word; a gap needs the specifics.
3. If `quzzar-workplace` says nothing about a dimension, it isn't a gap. Don't invent a
   standard to measure against, and don't generate cleanup work from silence.

Report it like this:

> **Meets standard:** <list, one line each>
>
> **Gaps:**
> - <dimension> — standard says <X>; code does <Y>. Affects <where, roughly how many files>.
>
> **Not covered by the standard** (so not assessed): <dimensions>

Order the gaps by how much they'd cost to leave alone, not by how easy they'd be to fix.

### Do not start fixing

Setup reports; it does not clean. A convention sweep touches many files at once and needs its
own review — folding it into setup produces an unreviewable diff and buries the one change
that mattered. Leave the fixing to a separate, scoped piece of work.

If the user wants to act on the gaps immediately, the honest options are: pick one gap and do
it as its own change, or run `/improve-codebase-architecture` (from Matt's skills, after a
restart) for a broader architectural scan presented as a report to choose from.

---

## Rules

- Survey before writing. Every question you ask about something discoverable in the repo is
  a question you shouldn't have asked.
- Never overwrite `CLAUDE.md`. Insert between markers, preserve the rest, confirm first.
- Never invent a convention because a `quzzar-workplace` section was empty. An empty section
  means *ask*, not *assume*.
- Report gaps; don't fix them. A convention sweep is its own scoped change.
- Matt's skills won't be available until the session restarts. Say so rather than claiming to
  have used them — and never claim to have run a `disable-model-invocation` skill like
  `/setup-matt-pocock-skills` or `/improve-codebase-architecture`, which only the user can invoke.
