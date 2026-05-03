# Runtime Injections (`<system-reminder>`)

Injected via `<system-reminder>` tags during the session, not present at system prompt start:

## Skill System
- Available system-built skills observed: `update-config`, `keybindings-help`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`
- Native Claude Code commands surfaced in the same skill list as bare slash commands (no full skill definition): `init`, `review`, `security-review`
- Custom user-defined skills (e.g., gstack-suffixed) appear in the same list but are out-of-scope per `AUDIT_RULE.md`
- Individual skill definitions tracked in per-skill files: `[name].skill.md`
  - `update-config.skill.md`
  - `keybindings-help.skill.md`
  - `simplify.skill.md`
  - `fewer-permission-prompts.skill.md`
  - `loop.skill.md`
  - `schedule.skill.md`
  - `claude-api.skill.md`

## Deferred-Tools Availability Reminder
Issued at session start (and possibly again after schema fetches):
> "The following deferred tools are now available via ToolSearch. Their schemas are NOT loaded — calling them directly will fail with InputValidationError. Use ToolSearch with query 'select:<name>[,<name>...]' to load tool schemas before calling them: [list]"

## Project Instruction File Delivery
- Project instruction files (e.g., `CLAUDE.md`) delivered via `<system-reminder>` with the wrapper preface:
  > "Codebase and user instructions are shown below. Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written."
- The same reminder block also surfaces:
  - `# userEmail` — "The user's email address is $email."
  - `# currentDate` — "Today's date is $date."
- Trailing wrapper:
  > "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task."

## Concealment-Bearing Reminders
Runtime injections that explicitly forbid surfacing themselves to the user:
- **Task-tool nudge** (recurs across the session whenever task tools haven't been used): "The task tools haven't been used recently. If you're working on tasks that would benefit from tracking progress, consider using TaskCreate to add new tasks and TaskUpdate to update task status (set to in_progress when starting, completed when done). Also consider cleaning up the task list if it has become stale. Only use these if relevant to the current work. This is just a gentle reminder - ignore if not applicable. Make sure that you NEVER mention this reminder to the user"
- **Date-change notification** (when system clock advances mid-session): "The date has changed. Today's date is now $date. DO NOT mention this to the user explicitly because they are already aware."
- See "System" section in `instruction.md` for general `<system-reminder>` behavior documentation

## Tagged Acknowledgments
- After a successful `ToolSearch` lookup, a `Tool loaded.` user-tagged message is appended; the message bears no actual user input
