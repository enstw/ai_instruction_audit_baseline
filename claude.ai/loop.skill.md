# Skill: loop

Run a prompt or slash command on a recurring interval (e.g. `/loop 5m /foo`, defaults to 10m). Omit the interval to let the model self-pace.

## When to invoke
- User wants to set up a recurring task
- Poll for status
- Run something repeatedly on an interval (e.g. "check the deploy every 5 minutes", "keep running /babysit-prs")

## Do NOT invoke
- One-off tasks
