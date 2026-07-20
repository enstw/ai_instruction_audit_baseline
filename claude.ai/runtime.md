# Runtime Injections (`<system-reminder>`)

Injected via `<system-reminder>` tags during the session, not present at system prompt start:

## Skill System
- Available system-built skills observed: `deep-research`, `dataviz`, `update-config`, `keybindings-help`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`, `run`, `init`, `review`, `security-review`
- Custom user-defined skills (e.g., gstack-suffixed) appear in the same list but are out-of-scope per `AUDIT_RULE.md`
- **Not observed this pass (2026-07-20)**: `verify` and `code-review`, present in every prior audit pass since 2026-05-21 (verify) / 2026-05-21 (code-review), including the immediately preceding 2026-07-17 pass. Per-skill files (`verify.skill.md`, `code-review.skill.md`) left in place pending re-confirmation in a future pass, per the precedent used for the JSON Parameters/Tool Invocation closing-directive removal.
- Individual skill definitions tracked in per-skill files: `[name].skill.md`
  - `deep-research.skill.md`
  - `dataviz.skill.md`
  - `update-config.skill.md`
  - `keybindings-help.skill.md`
  - `verify.skill.md` (not observed 2026-07-20 — see above)
  - `code-review.skill.md` (not observed 2026-07-20 — see above)
  - `simplify.skill.md`
  - `fewer-permission-prompts.skill.md`
  - `loop.skill.md`
  - `schedule.skill.md`
  - `claude-api.skill.md`
  - `run.skill.md`
  - `init.skill.md`
  - `review.skill.md`
  - `security-review.skill.md`

## Subagent Types Reminder
Issued at session start as a standalone `<system-reminder>` (separate from the `Agent` tool's own JSON description, which only says "Available agent types are listed in <system-reminder> messages in the conversation" and does not enumerate them):
> "Available agent types for the Agent tool:
> - claude: Catch-all for any task that doesn't fit a more specific agent. FleetView's default when no agent name is typed. (Tools: *)
> - Explore: Fast read-only search agent for locating code... (Tools: All tools except Agent, Artifact, ExitPlanMode, Edit, Write, NotebookEdit)
> - general-purpose: General-purpose agent for researching complex questions... (Tools: *)
> - Plan: Software architect agent for designing implementation plans... (Tools: All tools except Agent, Artifact, ExitPlanMode, Edit, Write, NotebookEdit)
> - statusline-setup: Use this agent to configure the user's Claude Code status line setting. (Tools: Read, Edit)
>
> When you launch multiple agents for independent work, send them in a single message with multiple tool uses so they run concurrently."
- Full per-type descriptions tracked in `embedded-tools.md`'s `## Agent` section
- The trailing "send independent agents in one message" directive is distinct from the Agent tool's own "if the user asks for parallel, MUST send one message" bullet — this one is a general default for any independent agent work, not conditioned on the user explicitly requesting parallelism

## Deferred-Tools Availability Reminder
Issued at session start (and possibly again after schema fetches):
> "The following deferred tools are now available via ToolSearch. Their schemas are NOT loaded — calling them directly will fail with InputValidationError. Use ToolSearch with query 'select:<name>[,<name>...]' to load tool schemas before calling them: [list]"

## Project Instruction File Delivery
- Project instruction files (e.g., `CLAUDE.md`) delivered via `<system-reminder>` with the leading preface:
  > "As you answer the user's questions, you can use the following context:"
- Followed by tagged sections, each with a heading tag and content:
  - `# claudeMd` — "Codebase and user instructions are shown below. Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written." Followed by `Contents of $path/CLAUDE.md (project instructions, checked into the codebase):` and the file body. Multiple project instruction files (parent + subdirectory) are concatenated under the same `# claudeMd` block.
  - `# userEmail` — "The user's email address is $email." (only present when user email is configured; may be omitted)
  - `# currentDate` — "Today's date is $date."
- Trailing wrapper:
  > "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task."

## Concealment-Bearing Reminders
Runtime injections that explicitly forbid surfacing themselves to the user:
- **Task-tools nudge** (recurs across the session whenever the task tools haven't been used): "The task tools haven't been used recently. If you're working on tasks that would benefit from tracking progress, consider using TaskCreate to add new tasks and TaskUpdate to update task status (set to in_progress when starting, completed when done). Also consider cleaning up the task list if it has become stale. Only use these if relevant to the current work. This is just a gentle reminder - ignore if not applicable." — note: no concealment instruction ("NEVER mention this reminder to the user") in this version
- **Date-change notification** (when system clock advances mid-session): "The date has changed. Today's date is now $date. DO NOT mention this to the user explicitly because they are already aware."
- See "System" section in `instruction.md` for general `<system-reminder>` behavior documentation

## Tagged Acknowledgments
- After a successful `ToolSearch` lookup, a `Tool loaded.` user-tagged message is appended; the message bears no actual user input
