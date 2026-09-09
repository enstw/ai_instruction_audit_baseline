# Skill: workflow-authoring

**[ADDED 2026-08-28]**: newly observed this pass — not previously tracked in any prior baseline.

Reference for writing a Workflow tool script (script API and gotchas, resume, quality patterns, worked examples). Load before authoring a script for a workflow the user already opted into; it does not itself authorize running one.

## Note
- Cross-referenced from the `Workflow` tool's own description (`embedded-tools.md` `## Workflow`): "Before writing a script, load the `workflow-authoring` skill — the workflow authoring reference: script API and gotchas, resume, the **Ultracode** section, quality patterns, worked examples." and "see **Ultracode** in the workflow authoring reference."
- **[2026-09-09]**: the `Workflow` tool's own description was directly re-read this pass (not invoked, but its full text is always visible as the tool's embedded schema) and confirmed to no longer contain the script format rules / script body hooks / concurrency / ultracode mode / quality patterns / resume / subagents material that a prior pass had documented inline under `embedded-tools.md` `## Workflow`. That material is presumed to now live inside this skill's own internal instructions, but the skill has still not been invoked, so its internal content remains unverified and out of tracked scope (per `AUDIT_RULE.md`, only the skill's listing description is tracked, not its body).
