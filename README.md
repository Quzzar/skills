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
| [`setup`](skills/setup/SKILL.md) | Run once per repo. Surveys the stack, writes skill routing and launch configuration, installs Impeccable with design checks and Matt Pocock's skills, and reports gaps. |
| [`quzzar-workplace`](skills/quzzar-workplace/SKILL.md) | The five core guidelines, language selection (TypeScript with Bun, Python with uv, Rust with Cargo), the canonical tool stack, the top-level `docs/` knowledge tree, and repo-wide code rules. Read before any work. |
| [`quzzar-frontend`](skills/quzzar-frontend/SKILL.md) | UI architecture and conventions, with Mobbin research and Impeccable design and quality checks for visual work. |
| [`quzzar-design`](skills/quzzar-design/SKILL.md) | Distinctive UI through product and website references, Impeccable design and refinement, AI-pattern detection, and the render-screenshot-critique loop. |
| [`quzzar-backend`](skills/quzzar-backend/SKILL.md) | Backend language toolchains, Rust implementation and checks, shared contracts across languages and runtimes, the single write path, invariants in Postgres, orchestration, secrets, deploys, observability. |
| [`illustrated-svg`](skills/illustrated-svg/SKILL.md) | Cohesive generated illustration sets, transparent asset extraction, vector reconstruction, and source/render verification. Includes a tracing helper and distinguishes editable vectors from articulated animation rigs. |
| [`animation-principles`](skills/animation-principles/SKILL.md) | Timing, easing, weight, anticipation, follow-through, and natural movement across tools. |
| [`color-motion`](skills/color-motion/SKILL.md) | Palettes, perceptual color interpolation, grading, and render color management. |
| [`shot-composition`](skills/shot-composition/SKILL.md) | Focal hierarchy, grids, safe areas, depth, and camera framing. |
| [`motion-art-direction`](skills/motion-art-direction/SKILL.md) | Motion language, tone, pacing, hierarchy, restraint, and shot consistency. |
| [`after-effects`](skills/after-effects/SKILL.md) | After Effects expressions, rigging, automation, and export. |
| [`beat-sync-editing`](skills/beat-sync-editing/SKILL.md) | Editing rhythm, beat-aligned cuts, transitions, and retiming. |
| [`remotion-video`](skills/remotion-video/SKILL.md) | Frame-based React video compositions and MP4/GIF rendering. |
| [`logo-animation`](skills/logo-animation/SKILL.md) | Logo reveals, stingers, splash screens, and loaders. |
| [`motion-background`](skills/motion-background/SKILL.md) | Subtle loops, gradients, shader backgrounds, and particles. |

The nine motion skills are vendored from [iart.ai](https://github.com/iart-ai/motion-design-skills).
See [motion skill maintenance](docs/motion-skills.md) for source revision, licensing, and updates.

### How they get invoked

`setup` writes a marker-fenced block into the repo's `CLAUDE.md` that routes agents to the right
skill: `quzzar-workplace` for any work, then `quzzar-frontend` or `quzzar-backend` by layer, and
`quzzar-design` on top of those for visual work. `CLAUDE.md` loads into every session
automatically, so that block is what makes the conventions apply without anyone remembering to
ask.

`setup` also records the repo's real stack and commands into that block, and every other skill
treats it as authoritative: the skills hold preferences that travel between repos, `CLAUDE.md`
holds the facts of the repo in front of you.

It writes one more file: `.claude/launch.json`, naming how to start each runnable app and on
what port. That's what lets a later session actually render the UI it just wrote instead of
guessing at it, which is the loop `quzzar-design` is built around.

**Run `setup` first in a new repo.** The others assume its block exists.

### Impeccable and visual research

Setup installs the full official [Impeccable](https://impeccable.style/) build for the coding
tools used in the repo, including its playbooks, scripts, and supported design hooks. It
checks the installation and hook status, initializes product context for frontend work, and
documents an existing visual system. The report identifies any reload or human hook-trust
action still needed. Re-running setup preserves existing configuration and installation scope.

Visual work uses Mobbin to study interaction patterns and broader website references to
explore composition and identity. Impeccable then guides design, critique, focused refinement,
polish, and audit. Source and rendered detector scans catch known AI patterns and quality
issues; screenshots and interaction checks verify the actual result. The
[visual workflow](skills/quzzar-design/references/impeccable.md) defines these checks.

Requests for multiple versions use Impeccable's variation workflows to explore the requested
number of distinct directions, then critique and polish each. Existing-element variants can
be compared live with `generate`; new page identities use the new-design workflow.

Impeccable is agent tooling and adds no runtime dependency to the app. Its commands use
`/impeccable` in Claude Code and `$impeccable` in Codex. Setup uses Bun's `bunx` runner for
the official package; the [upstream installation
guide](https://impeccable.style/tutorials/getting-started/) uses `npx`.

## Scope

These skills describe roles and rules, never a particular project's package names, directories,
or queues. Anything specific to one codebase belongs in that repo's own `CLAUDE.md` or its own
project skill, not here.

## Adding a skill

1. `mkdir -p skills/<name>` and write `SKILL.md`. See
   [`docs/skill-template.md`](docs/skill-template.md) for the frontmatter and how to write a
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

Claude-only by construction, versioned, and all fifteen skills come as one unit:

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
