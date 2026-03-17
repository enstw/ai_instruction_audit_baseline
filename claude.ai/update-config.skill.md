# Skill: update-config

Configure the Claude Code harness via settings.json.

## When to invoke
- Automated behaviors ("from now on when X", "each time X", "whenever X", "before/after X") — these require hooks in settings.json; memory/preferences cannot fulfill them
- Permissions ("allow X", "add permission", "move permission to")
- Environment variables ("set X=Y")
- Hook troubleshooting
- Any changes to settings.json / settings.local.json

## Examples
- "allow npm commands"
- "add bq permission to global settings"
- "move permission to user settings"
- "set DEBUG=true"
- "when claude stops show X"

## Notes
- For simple settings like theme/model, use Config tool instead
