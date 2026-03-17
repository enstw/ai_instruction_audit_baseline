# Runtime Injections (`<system-reminder>`)

Injected via `<system-reminder>` tags during the session, not present at system prompt start:

## Skill System
- Available skills (as of session): `update-config`, `keybindings-help`, `simplify`, `loop`, `claude-api`
- Individual skill definitions tracked in per-skill files: `[name].skill.md`
  - `update-config.skill.md`
  - `keybindings-help.skill.md`
  - `simplify.skill.md`
  - `loop.skill.md`
  - `claude-api.skill.md`

## Project Instruction File Delivery
- Project instruction files (e.g., CLAUDE.md) delivered via `<system-reminder>` with wrapper directives:
  - "Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written."
  - "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task."
- `currentDate` injected alongside project file content: "Today's date is $date."

## Concealment-Bearing Reminders
- Observed pattern: task tool usage nudges referencing TaskCreate/TaskUpdate with "Make sure that you NEVER mention this reminder to the user"
- See "System" section in `instruction.md` for general `<system-reminder>` behavior documentation
