# quzzar-skills

My agent skills, for Claude Code. Installed with the
[`skills`](https://github.com/vercel-labs/skills) CLI.

## Install

```bash
npx skills@latest add Quzzar/skills -g -a claude-code --copy
```

That opens a picker for which skills to install. All three flags matter:

| Flag | Why |
| --- | --- |
| `-g` | Global. Installs to `~/.claude/skills/`, available in every repo. Drop it for project-level `./.claude/skills/`. |
| `-a claude-code` | Claude Code only. Without it the CLI reuses your last agent selection, which is how skills end up in Cursor, Codex, Zed, and every other agent it can find. |
| `--copy` | Real files in `.claude/`. Without it you get symlinks pointing into `~/.agents/skills/`. |

To skip the picker and name the skills up front:

```bash
npx skills@latest add Quzzar/skills -g -a claude-code --copy -s setup,quzzar-workplace -y
```

Update and remove:

```bash
npx skills@latest update -g
```

```bash
npx skills@latest remove -g -a claude-code
```

## Where the files actually go

With `--copy`, the skill files are real files under `.claude/`:

```
~/.claude/skills/quzzar-workplace/SKILL.md    ← the actual file
~/.agents/.skill-lock.json                    ← bookkeeping only
```

Without `--copy`, the CLI installs one canonical copy into `~/.agents/skills/<name>/` and
symlinks each agent's directory at it, so `~/.claude/skills/<name>` becomes a link rather than
a file. That's how one `npx skills update` refreshes every agent at once, but it means the
content isn't really in `.claude` at all.

`~/.agents/.skill-lock.json` is written either way. It's the CLI's record of what's installed,
where it came from, and its content hash. `update` and `remove` need it. It holds no skill
content, and with `--copy` no `~/.agents/skills/` directory is created.

One thing worth knowing: the lock file also stores `lastSelectedAgents`, and the picker
pre-checks whatever you chose last time. A single "select all" is sticky and will keep
installing everywhere. Passing `-a claude-code` every time avoids it; running the picker once
and deselecting rewrites the list.

## The skills

| Skill | What it does |
| --- | --- |
| [`setup`](skills/setup/SKILL.md) | Run once per repo. Surveys the stack and existing practices, writes the skill-routing block into `CLAUDE.md`, installs Matt Pocock's skills, and reports where the code falls short of `quzzar-workplace`. |
| [`quzzar-workplace`](skills/quzzar-workplace/SKILL.md) | The five core guidelines, the canonical tool stack, the top-level `docs/` knowledge tree, and repo-wide rules: naming, file organization, error handling, comments, formatting. Read before any work. |
| [`quzzar-frontend`](skills/quzzar-frontend/SKILL.md) | The layer boundary, component structure, styling and tokens, state, forms, animation, accessibility, UI copy, the component gallery. |
| [`quzzar-backend`](skills/quzzar-backend/SKILL.md) | The two runtimes, shared schemas across runtimes, the single write path, invariants in Postgres, orchestration, secrets, deploys, observability. |

### How they get invoked

`setup` writes a marker-fenced block into the repo's `CLAUDE.md` that routes agents to the right
skill: `quzzar-workplace` for any work, then `quzzar-frontend` or `quzzar-backend` by layer.
`CLAUDE.md` loads into every session automatically, so that block is what makes the conventions
apply without anyone remembering to ask.

`setup` also records the repo's real stack and commands into that block, and all three skills
treat it as authoritative: the skills hold preferences that travel between repos, `CLAUDE.md`
holds the facts of the repo in front of you.

**Run `setup` first in a new repo.** The other three assume its block exists.

## Scope

These skills describe roles and rules, never a particular project's package names, directories,
or queues. Anything specific to one codebase belongs in that repo's own `CLAUDE.md` or its own
project skill, not here.

## Adding a skill

1. `mkdir -p skills/<name>` and write `SKILL.md`. See
   [`docs/SKILL-template.md`](docs/SKILL-template.md) for the frontmatter and how to write a
   `description` that triggers reliably. It's the only part loaded into context until the skill
   fires, so a vague one misfires.
2. Add `./skills/<name>` to the `skills` array in
   [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json). That manifest is an explicit list
   and does not auto-discover. The `skills` CLI *does*, so this only matters for the plugin
   install path below.
3. Test it from the working copy before pushing:

   ```bash
   npx skills@latest add . -g -a claude-code --copy -s <name> -y
   ```

`npx skills init <name>` scaffolds a bare `SKILL.md` if you'd rather start from one.

## Also installable as a Claude Code plugin

Claude-only by construction, versioned, and all four skills come as one unit:

```bash
claude plugin marketplace add Quzzar/skills
```

```bash
claude plugin install quzzar-skills@quzzar
```

Plugin skills are namespaced (`quzzar-skills:quzzar-frontend` rather than `quzzar-frontend`),
and they live in the plugin cache rather than `.claude/skills/`. Bump `version` in
[`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) when you want installs to pick up
changes.

## License

MIT
