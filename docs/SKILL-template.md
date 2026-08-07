# SKILL.md template

Copy this into `skills/<your-skill-name>/SKILL.md`. Kept in `docs/` on purpose — the
`skills` CLI only discovers directories that contain a `SKILL.md`, so a template living
here never shows up as an installable skill.

After adding a skill, add its path to the `skills` array in
[`.claude-plugin/plugin.json`](../.claude-plugin/plugin.json) — that file is only read by
the Claude Code plugin installer, and it does **not** auto-discover.

---

```markdown
---
name: your-skill-name
description: One sentence on what this does, then the trigger conditions. Use when the user says "X", asks to "Y", or mentions Z.
---

# your-skill-name

One or two lines on what this skill is for.

## When to use

- Concrete trigger.
- Another concrete trigger.

Do **not** use for: the nearest thing this is not, so the model doesn't over-trigger.

## Instructions

1. First step.
2. Second step.

## Rules

- Constraints that hold regardless of the steps above.
```

---

## Frontmatter

| Field | Required | Notes |
| --- | --- | --- |
| `name` | yes | Lowercase, hyphens only. Must match the directory name. |
| `description` | yes | The **only** part loaded into context until the skill triggers. |

Optional: `metadata.internal: true` hides a skill from discovery unless
`INSTALL_INTERNAL_SKILLS=1` is set.

## Writing a description that triggers correctly

The `description` is the entire basis for the model deciding whether to load the skill —
the body is not read until then. So it needs both halves:

1. **What it does** — so the model knows if it's relevant.
2. **When to use it** — literal phrases the user is likely to type.

Compare:

- ✗ `description: Helps with pull requests` — too vague to match against, will misfire.
- ✓ `description: Draft a pull request description from the actual branch diff. Use when the user asks to "write a PR description", "open a PR", or "describe these changes".`

Naming the negative case (`Do not use for...`) in the body is what stops a skill firing on
every adjacent request.

## Body

Plain markdown, written as instructions to an agent rather than documentation for a human.
Imperative, specific, and short — anything the model already knows is noise. Reference
extra files relatively (`./reference.md`) and they travel with the skill on install.
