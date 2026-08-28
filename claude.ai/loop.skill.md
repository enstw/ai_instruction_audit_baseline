# Skill: loop

Run a prompt or slash command on a recurring interval (e.g. `/loop 5m /foo`). Omit the interval to let the model self-pace.

**[MODIFIED 2026-08-28]**: previously "defaults to 10m" when the interval was omitted; now omitting the interval means the model self-paces (dynamic mode) instead of falling back to a fixed 10-minute default. Corroborated by `ScheduleWakeup`'s own description this pass: "Schedule when to resume work in `/loop` dynamic mode — the user invoked `/loop` without an interval, asking you to self-pace iterations of a specific task."

## When to invoke
- User wants to set up a recurring task
- Poll for status
- Run something repeatedly on an interval (e.g. "check the deploy every 5 minutes", "keep running /babysit-prs")

## Do NOT invoke
- One-off tasks
