Source layer: runtime injection.

<skills_instructions>
## Skills

A skill is a set of local instructions stored in a `SKILL.md` file.
If the user names a skill, including by plain text or `$SkillName`, or the task clearly matches a listed skill description, use that skill for the turn.
Do not carry a skill across turns unless it is mentioned again.

### Available skills

- `openai-docs`: use when the user asks how to build with OpenAI products or APIs and needs up-to-date official documentation with citations, model guidance, or prompt-upgrade guidance.
- `skill-creator`: use when the user wants to create or update a skill.
- `skill-installer`: use when the user wants to list or install skills.

### How to use skills

- Announce which skill is being used and why in one short line.
- Open the relevant `SKILL.md` and read only enough to follow the workflow.
- Resolve relative paths from the skill directory first.
- Load only the specific referenced files needed for the request.
- Prefer reusing scripts, assets, and templates that the skill already provides.
- If multiple skills apply, use the minimal set that covers the task and state the order.
- If a skill cannot be applied cleanly, state the issue briefly and continue with the best fallback.
- Keep context small and avoid loading unrelated references.
- If the skill includes scripts or templates, prefer using them instead of recreating the content manually.
</skills_instructions>
