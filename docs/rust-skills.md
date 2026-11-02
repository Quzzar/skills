# Bundled Rust guidance

`skills/rust-skills/` vendors Leonardo Maldonado's MIT-licensed
[rust-skills](https://github.com/leonardomso/rust-skills/tree/fd2a861ab0406a4ac536a55274d14ea6fd1ca9c9)
version 1.5.1, revision `fd2a861ab0406a4ac536a55274d14ea6fd1ca9c9`. The source was the
pinned copy already installed in Spellwake on November 1, 2026. The bundle includes
`SKILL.md`, all 265 rule files, and the original MIT license. It excludes upstream
repository administration files and the upstream rule-example check harness; no rule
content or referenced rule file was omitted.

Local adaptations: the entry point uses host-neutral invocation, defers Rust edition and
toolchain selection to each project, and adds the portable compiler workflow used in
Spellwake. Project invariants, typed errors, safe-code policy, lockfiles, and behavioral
tests take precedence over generic optimization or dependency recommendations. No
Spellwake domain data, internal docs, application code, or Claude hook was copied.

For an update, review an explicit upstream revision. Compare rules and their index, retain
the license, reapply the local entry-point adaptations, and update this provenance record.
Check every index-to-rule link and the plugin inventory. The bundled instructions are
agent tooling, not a runtime dependency or an automatic Rust toolchain installation.
