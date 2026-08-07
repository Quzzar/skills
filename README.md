# quzzar-skills

My agent skills. Installable with the [`skills`](https://github.com/vercel-labs/skills) CLI
or as a Claude Code plugin.

## Skills

| Skill | What it does |
| --- | --- |
| [`setup`](skills/setup/SKILL.md) | Run once per repo. Surveys the stack and existing practices, writes the skill-routing block into `CLAUDE.md`, installs Matt Pocock's skills, reports where the code falls short of `quzzar-workplace`. |
| [`quzzar-workplace`](skills/quzzar-workplace/SKILL.md) | Issue tracking, branches, commits, PRs, review, CI — plus code rules that apply to every layer. |
| [`quzzar-frontend`](skills/quzzar-frontend/SKILL.md) | Components, styling, state, data fetching, forms, accessibility, performance, UI tests. |
| [`quzzar-backend`](skills/quzzar-backend/SKILL.md) | API design, data access, migrations, validation, auth, errors, jobs, observability, backend tests. |

The three `quzzar-*` skills are **scaffolds** — their sections are marked `<!-- TODO -->`
until you fill them in. That's deliberate: each one instructs the agent that a `TODO`
section means *no convention exists*, so it should match surrounding code and say so rather
than invent a rule. Fill them in as you notice yourself correcting the same thing twice.

### How they get invoked

`setup` writes a marker-fenced block into the repo's `CLAUDE.md` that routes agents to the
right skill — `quzzar-workplace` for any work, then `quzzar-frontend` or `quzzar-backend` by
layer. Since `CLAUDE.md` loads into every session automatically, that block is what makes the
conventions apply without anyone remembering to ask for them.

`setup` also records the repo's real stack and commands into that block, and all three skills
treat it as authoritative — the skills hold preferences that travel between repos, `CLAUDE.md`
holds the facts of the repo in front of you. Run `setup` first.

## Install — Claude Code only

The `skills` CLI installs into every agent it can find unless you tell it otherwise. Pin it
with `-a claude-code`:

```bash
npx skills@latest add Quzzar/skills -g -a claude-code
```

That opens a picker for *which* skills to install. To skip the picker and choose up front:

```bash
npx skills@latest add Quzzar/skills -g -a claude-code -s setup,quzzar-workplace -y
```

Flags worth knowing:

| Flag | Effect |
| --- | --- |
| `-g` | Global — installs to `~/.claude/skills/`. Omit for project-level `./.claude/skills/`. |
| `-a claude-code` | Only Claude Code. Without this, the CLI reuses your last agent selection. |
| `-s <names>` | Pick skills without the interactive picker. Comma-separated, `'*'` for all. |
| `-y` | Skip confirmation prompts. |
| `--copy` | Copy files instead of symlinking (see below). |

Update and remove:

```bash
npx skills@latest update -g
```

```bash
npx skills@latest remove -g -a claude-code
```

## Install — as a Claude Code plugin

Claude-only by construction, and versioned. All skills in the repo come as one unit:

```bash
claude plugin marketplace add Quzzar/skills
```

```bash
claude plugin install quzzar-skills@quzzar
```

Plugin skills are namespaced — `quzzar-skills:quzzar-frontend` rather than
`quzzar-frontend`. Bump `version` in [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json)
when you want installs to pick up changes.

## How the CLI lays skills out on disk

Worth understanding, because it looks like the CLI is scattering files around:

```
~/.agents/skills/quzzar-frontend/SKILL.md      ← the one real copy
~/.claude/skills/quzzar-frontend   → symlink ──┘
~/.cursor/skills/quzzar-frontend   → symlink ──┘
~/.codex/skills/quzzar-frontend    → symlink ──┘
```

`~/.agents/` is the canonical store — a cross-agent convention, not a Claude thing. Each
agent's own skills directory gets a symlink pointing back into it. One copy on disk, one
`npx skills update` to refresh all of them, and `~/.agents/.skill-lock.json` records where
each skill came from plus a content hash.

This is how **global** (`-g`) installs work. A project-scope install copies the files into
`./.claude/skills/` instead, so they commit with the repo — `~/.agents/` isn't involved.

Two consequences:

- **`lastSelectedAgents` in the lock file is sticky.** The CLI pre-checks whatever you
  picked last time, so a one-off "select all" keeps installing everywhere. Either pass
  `-a claude-code` every time, or run the picker once and deselect — the lock file updates.
- **`--copy` opts out of symlinking** and gives each agent an independent copy. Only worth
  it if you want to hand-edit one agent's version, or on a filesystem without symlinks.

## Adding a skill

1. `mkdir -p skills/<name>` and write `SKILL.md` — see
   [`docs/SKILL-template.md`](docs/SKILL-template.md) for the frontmatter and how to write a
   `description` that triggers reliably.
2. Add `./skills/<name>` to the `skills` array in
   [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json). The plugin manifest is an
   explicit list — it does not auto-discover. The `skills` CLI *does*, so this step only
   matters for the plugin install path.
3. Test it before pushing, straight from the working copy:

   ```bash
   npx skills@latest add . -g -a claude-code -s <name> -y
   ```

`npx skills init <name>` also scaffolds a bare `SKILL.md` if you'd rather start there.

## License

MIT
