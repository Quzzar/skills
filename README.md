# quzzar-skills

Quzzar's reusable agent skills for engineering, design, illustration, and motion. The
instructions are agent agnostic. Codex is the default for examples; use the equivalent
skill discovery, question, browser, and delegation capabilities in other hosts.

## Use with Codex

Install the skills you need from this repository with Codex's built-in `skill-installer`.
For example, ask:

> Use $skill-installer to install quzzar-workplace from Quzzar/skills, path skills/quzzar-workplace.

For local development, Codex discovers skill folders under a project's `.agents/skills/`
and the user's `~/.agents/skills/`, including symlinked folders. See the
[official skill documentation](https://learn.chatgpt.com/docs/build-skills) for discovery
and installation details. Keep the source checkout and installed copies distinct: edits
to copied installations do not automatically update this repo, or vice versa.

The collection requires no bootstrap skill, Impeccable installation, or Matt Pocock skill
package. Specialized workflows need their task-specific capabilities: image generation,
browser review, and independent agents for UI/UX; quzzar-flow for system diagrams; or a
video renderer for motion work.

## The skills

| Skill | What it does |
| --- | --- |
| [`quzzar-workplace`](skills/quzzar-workplace/SKILL.md) | The five core guidelines, Rust-first language selection, shared-source Tauri/React applications, the canonical tool stack, the top-level `docs/` knowledge tree, and repo-wide code rules. Read before any work. |
| [`quzzar-frontend`](skills/quzzar-frontend/SKILL.md) | UI architecture and conventions, for shared browser/Tauri UI, recommended libraries, [Prompt Motion](https://prompt-motion.com) animation references, terse copy, and rendered verification. |
| [`quzzar-ui-ux-design`](skills/quzzar-ui-ux-design/SKILL.md) | Every new UI: three generated directions, complete image/interaction coverage, faithful component-based implementation, and user-reviewed consolidation. |
| [`quzzar-backend`](skills/quzzar-backend/SKILL.md) | Rust-first implementation, RLS-protected reads and writes, shared contracts, Postgres invariants, optional orchestration, secrets, and Langfuse for AI calls. |
| [`quzzar-system-design`](skills/quzzar-system-design/SKILL.md) | Data-oriented system design with quzzar-flow: new-system discovery or honest existing-system audit, visual iteration, exhaustive planning, implementation, and independent verification. |
| [`rust-skills`](skills/rust-skills/SKILL.md) | Bundled Rust rule collection used in Spellwake, with task-specific guidance and a portable compiler/check workflow. |
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

## How the engineering skills fit together

Read `quzzar-workplace` before work, then `quzzar-frontend` or `quzzar-backend` for the layer
being changed. Read both when a change spans layers. For Rust writing, review, or refactoring,
read the bundled `rust-skills` index and the task-relevant rules. Use `quzzar-system-design`
for architecture design or audits, including schema-backed table and data-flow contracts.

Rust/Cargo is the default for application logic and backends. Use one shared React application
for Tauri desktop/mobile and the website, with small typed host adapters. Use Bun for
TypeScript, uv for Python, Supabase for managed data/auth/storage, and Langfuse for every
application AI call. Temporal is optional when durable orchestration is needed.

Repository instructions and actual configuration supply project-specific facts. Use
`AGENTS.md` for shared guidance and honor other instruction files used by the active host.
No generated marker block or agent-specific launch configuration is required. Get build,
test, and dev-server commands from the project itself.

Use `quzzar-ui-ux-design` for every new UI. Approve a direction and the complete screen/state
image walkthrough before product implementation. Build and verify the faithful component-based
version, then present visible consolidation as a separate user choice. The final review shows
approved images, faithful renders, consolidated renders, and both component galleries.

## Scope

These skills describe reusable roles and rules. Project-specific package names, directories,
queues, and commands belong in that repository's instructions and documentation.

## Changing or adding a skill

Each installable skill lives in `skills/<name>/SKILL.md`. Supporting references, scripts,
and assets stay inside that skill's folder so they travel with an individual installation.
See [the skill template](docs/skill-template.md) for frontmatter and trigger guidance.

When adding or removing a skill, update this catalog and the explicit `skills` list in
[the compatibility plugin manifest](.claude-plugin/plugin.json). Verify relative references
and any changed scripts, then exercise the skill in the target agent before release.

## Optional Claude Code plugin

Claude Code remains a supported installation target. Its compatibility packaging lives in
`.claude-plugin/`; it does not dictate the shared skill instructions. All sixteen skills
can be installed together:

```bash
claude plugin marketplace add Quzzar/skills
claude plugin install quzzar-skills@quzzar
```

Plugin skills are namespaced, such as `quzzar-skills:quzzar-frontend`. Bump `version` in
[the plugin manifest](.claude-plugin/plugin.json) when releasing plugin changes.

## License

MIT
