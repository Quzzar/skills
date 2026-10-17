# Motion skills

The nine skills from [iart-ai/motion-design-skills](https://github.com/iart-ai/motion-design-skills)
are included in `skills/` and the plugin manifest, at source commit `3c129f769d90a1328c209c386492333c9ac62312`.
Each skill retains iart.ai's MIT license. Upstream instructions and references are preserved.
Repository-level verification helpers referenced by a skill are copied into that skill's
`scripts/` directory so a standalone install includes them. Resolve helper paths relative to
the installed skill, passing the animation/output path explicitly. Shell helpers require Bash
and their documented dependencies; After Effects scripts require After Effects.

To refresh, compare the upstream skills and helper files at a chosen commit, update their
copies and licenses, then update this source revision. Keep the explicit plugin skill list,
README catalog, and routing in `quzzar-design` and `setup` aligned. Bump the plugin version
for a release. The extra packs linked upstream are project-specific additions in Visual Audible,
not part of the general Quzzar collection.

Packaging metadata uses `metadata.version` for Agent Skills compatibility. Instruction bodies
are preserved. Reapply the metadata adaptation after upstream updates.
