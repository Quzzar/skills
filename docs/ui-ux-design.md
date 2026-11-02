# UI/UX design workflow

`quzzar-ui-ux-design` is the replacement for the removed `quzzar-design` skill. It applies to
every new UI. Its three approval boundaries are visual-direction selection, the complete
screen/state image set and UX flow, and any appearance-changing consolidation after faithful
implementation. Existing component libraries and new ones follow the same core process.

The workflow was written from the user's requirements in this chat. Local reference research
inspected Spellwake's installed Impeccable 0.1.18 new-work, visualize, region-map, extract, and
component-review guidance, plus Spellwake's button/select implementations and component workshop.
The useful ideas carried into the independent workflow are visual comparison, region-based
measurement, separation of artwork from semantic UI, image-guided asset production, and
reviewable variant/state galleries. No Impeccable launcher, hook, script, or runtime dependency
was copied or required. The user's workflow governs direction count and approval; Impeccable's
random direction selection, delegated picks, and unrelated process requirements were not adopted.

Independent agents inventory views, verify each image's interactions, inventory components,
verify each implemented page, and propose consolidation. The user remains the authority on
product behavior and visible design changes. Review tooling keeps the approved reference,
actual faithful render, and actual consolidated render distinct; a generated mockup is never
implementation evidence. Full references remain available when comparing cropped regions.

Each image receives an interaction check, then a verifier repeatedly inspects the entire
package to discover omitted views or controls. The author resolves each finding by generating
missing states or removing unnecessary unapproved controls, then resubmits the updated package.
The current package must pass coverage review before user flow approval; a stale verdict or
self-reported fix does not close the loop.

Primary-viewport design is intentionally completed first. The final presentation names the
finished target and every outstanding requested device layout. Each additional target restarts
the complete skill from Step 1, including three directions, UX discovery, complete images,
verifier iteration, user approval, faithful implementation, and consolidation review. Settled
requirements and identity carry forward as context, not automatic approval of new device flows.
The same review workspace and shared production application cover all target passes. Shared
component edits require regression checks on previously approved targets. This does not remove
accessibility or the requirement to verify the platforms the task actually promises.
