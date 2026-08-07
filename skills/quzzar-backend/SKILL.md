---
name: quzzar-backend
description: Quzzar's backend conventions — API design, data access and migrations, validation, auth, error handling, background jobs, external integrations, observability, secrets, and backend testing. Use before writing or changing any server-side code, API route, database query, migration, or backend test.
---

# quzzar-backend

Backend conventions. Repo-wide rules (naming, commits, dependencies, file organization) live
in `quzzar-workplace` — read that too for anything not specific to the server layer.

## When to use

- Writing or changing an API route, handler, Server Action, or service function.
- Touching the database — queries, schema, or migrations.
- Adding auth, validation, a background job, or an external API integration.
- Writing or fixing a backend test.

## First: read the repo's own context

`CLAUDE.md` at the repo root carries this repo's real runtime, database, and commands. Repo
facts beat anything general below. If it has no `<!-- quzzar-skills:start -->` block, run the
`setup` skill first.

## Tool choices are already decided

**Which** tool to use is fixed by the stack table in `quzzar-workplace` — bun, Temporal, Deno,
Supabase, GitHub Actions, Langfuse, Zod, Playwright. Read it and use what it says.

This skill covers **how** we use them. Adding a second datastore or auth vendor is explicitly
out; anything else outside the table is an `AskUserQuestion`, not a judgment call.

## Handling unfilled sections

Sections marked `<!-- TODO -->` have no convention recorded yet.

**Do not invent one.** Match the nearest existing handler or query, and name the missing
convention in your report. If existing code disagrees and the choice matters, use
`AskUserQuestion`.

---

## Runtime — two runtimes, don't mix them up

- **bun** — runtime *and* package manager for service packages and the workspace root.
  **Never npm.**
- **Deno** — the Supabase edge-function runtime. **Not bun.**

Check which context you're in before writing a runtime-specific import or a lockfile-touching
command. Code written for one does not necessarily run on the other.

<!-- TODO: Which APIs are safe to share between the bun and Deno sides, and how shared code
     is structured so it runs on both.
     - Execution limits on the edge-function side that shape how handlers are written. -->

## API design

Every response follows the `ApiResponse<T, F>` shape from `quzzar-workplace` core guideline 5.
The three-way split is load-bearing: `success`, `fail` (client's fault — validation and
business rules, per-field in `data`), `error` (server's fault — `message` safe to surface,
never leaking internals).

<!-- TODO: Route/procedure naming and URL shape.
     - Which HTTP status codes accompany fail vs. error, and what a client can rely on.
     - Versioning: how, or explicitly not.
     - Pagination style (cursor vs. offset) and its standard param names. -->

## Layering

<!-- TODO: How a request flows — handler → service → data access, or thinner?
     - What a handler is allowed to do directly vs. must delegate.
     - Where business logic must NOT live.
     - Whether Supabase client types are allowed to leak to callers. -->

## Validation

Zod, from the shared schema package. Every cross-boundary shape has a schema — wire shapes,
intake, agent contracts, and form validation via the RHF resolver. **One schema, validated on
both ends.**

Objects are **parsed** at the boundary, not cast (core guideline 4). Types are derived from
schemas rather than declared alongside them.

<!-- TODO: What happens on a parse failure at each boundary — which become `fail` responses,
     which are `error`, and which are non-recoverable.
     - Whether internal service-to-service calls re-parse, or trust the caller. -->

## Data access — Supabase

Postgres is the control plane, with **RLS**. Auth and Storage are Supabase too. Don't add a
second datastore or auth vendor.

<!-- TODO: Where query code lives, and whether queries are reusable functions or inline.
     - Raw SQL: allowed, and when, vs. the Supabase client?
     - Transaction boundaries — who opens one, and what must be inside it.
     - RLS as the enforcement point vs. checks in application code — which is authoritative?
     - Soft delete or hard delete? Timestamp columns on every table? -->

## Migrations

<!-- TODO: The Supabase migration workflow — how they're generated, reviewed, and applied.
     - Backwards-compatibility requirement: must a migration be safe to deploy before the
       code that uses it?
     - How destructive changes are staged (note: core guideline 1 wants dead columns gone,
       so record how that lands safely across a release).
     - Seeding, and whether it runs in any non-local environment. -->

## Auth

Supabase Auth, with RLS in Postgres.

<!-- TODO: Where the session is read, and where authorization is enforced — RLS, handler,
     or both?
     - Is every endpoint protected by default or opt-in?
     - How row ownership / tenancy is expressed in policies.
     - Service-to-service auth for internal calls and the runner. -->

## Orchestration — Temporal

Durable orchestration for long-running pipelines.

One task queue per pipeline, each with its **own worker entry point** — deliberately separate
so a local `pkill` can't cross-kill the other workers.

Keep that separation when adding workers — a shared entry point reintroduces the cross-kill
problem it exists to prevent.

<!-- TODO: Activity idempotency requirements, and what must be safe to retry.
     - Retry policy and timeout defaults; where they're configured.
     - What belongs in a workflow vs. an activity.
     - Versioning long-running workflows when their code changes.
     - How a failed run surfaces (and how that relates to Langfuse). -->

## The runner

GitHub Actions hosts the runner today — free tier, roughly 2000 min/month. An always-on VM is
the known upgrade path.

<!-- TODO: What the minute budget means in practice — what's safe to run in CI vs. what needs
     the VM, and the signal that it's time to migrate. -->

## Errors

The client-facing shape is fixed by core guideline 5. Internals never leak through
`error.message`.

<!-- TODO: Custom error classes — do they exist, and where?
     - Which failures are expected (return a `fail`) vs. exceptional (throw).
     - Retry policy for transient failures outside Temporal.
     - What must never be caught-and-ignored. -->

## External integrations

<!-- TODO: How third-party APIs are wrapped — a client module per service?
     - Timeout and retry defaults.
     - Webhook handling: signature verification, idempotency keys, replay safety.
     - How a non-critical dependency's failure degrades rather than breaks the request. -->

## Config and secrets

<!-- TODO: Where env vars are declared and validated — a single Zod-parsed config module?
     (Core guideline 4 points that way; record it if so.)
     - The rule against reading `process.env` / `Deno.env` outside it.
     - How secrets differ across bun, Deno, GitHub Actions, and Supabase.
     - What happens at boot if a required var is missing. -->

## Observability — Langfuse

Tracing and quality scores for pipeline runs.

<!-- TODO: What must be traced — every run, or specific activities?
     - Which quality scores are recorded, and what a bad score triggers.
     - Logger of record outside Langfuse, and the rule on `console.*`.
     - What must never appear in a trace or log (PII, tokens, full payloads). -->

## Testing — Playwright

End-to-end against a running app, per `quzzar-workplace`. Note that an e2e-first tool covers
API surface reached through the app; workflows and activities that no request path touches need
a deliberate answer below.

<!-- TODO: Real Supabase instance or a seeded test project? If real, how it's isolated per run.
     - How Temporal workflows are tested (test environment, time-skipping) — Playwright won't
       reach these on its own.
     - How external APIs are handled (fixtures, sandbox credentials).
     - Whether a new endpoint needs a spec to merge. -->

---

## Rules

- Tool selection is `quzzar-workplace`'s stack table, not a per-change decision. A second
  datastore or auth vendor is out; anything else outside the table is an `AskUserQuestion`.
- **Never npm.** bun for service packages and the workspace root; Deno for Supabase edge
  functions.
- Every cross-boundary shape is a Zod schema in the shared schema package, parsed at the
  boundary and shared by both ends.
- Every response is `ApiResponse<T, F>`. `fail` is the client's fault, `error` is ours.
- `CLAUDE.md` overrides this file — it holds the facts of the repo in front of you.
- Where this file is silent, the nearest existing handler or query is the convention.
- Never present a rule from a `<!-- TODO -->` section as established.
- Deleting an unused endpoint means deleting its schema, tests, types, and any orphaned
  migration path too (core guideline 1).
