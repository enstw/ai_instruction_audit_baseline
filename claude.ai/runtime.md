# Runtime Injections (`<system-reminder>`)

Injected via `<system-reminder>` tags during the session, not present at system prompt start:

## Skill System
- Available system-built skills observed: `dataviz`, `update-config`, `keybindings-help`, `code-review`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`, `run`, `init`, `security-review`
- Custom user-defined skills (e.g., gstack-suffixed) appear in the same list but are out-of-scope per `AUDIT_RULE.md`
- **`code-review` description expanded again (2026-08-10)**: the effort-level parenthetical gained a third tier, `ultra`: "Review the current diff, or a PR number/branch/path target, for correctness bugs and reuse/simplification/efficiency cleanups at the given effort level (low/medium: fewer, high-confidence findings; high→max: broader coverage, may include uncertain findings; ultra: deep multi-agent review in the cloud); with no level given, it reuses the level you typed last. Pass --comment to post findings as inline PR comments, or --fix to apply the findings to the working tree after the review." This lines up with the new "ultrareview" guidance added to `instruction.md`'s Session-Specific Guidance this pass (`/code-review ultra` / deprecated `/ultrareview` alias, launches a billed multi-agent cloud review).
  - Prior (2026-08-07) reappearance context: `code-review` had been absent since being confirmed removed 2026-07-23, came back 2026-08-07 with a two-tier description (PR/branch/path target support, effort-level memory) — see git history for that version.
- **Confirmed removed (2026-08-10)**: `review` — absent in both the 2026-08-07 and 2026-08-10 passes, after being present and stable through 2026-08-04 (`review.skill.md`: "Review a GitHub pull request; for your working diff use /code-review"), per the two-consecutive-absent-passes precedent used for prior skill removals. Per-skill file (`review.skill.md`) removed from the baseline this pass.
- **Confirmed removed (2026-07-23)**: `verify` — absent in both the 2026-07-20 and 2026-07-23 passes, after being stable and unchanged across every audit pass from 2026-05-21 through 2026-07-17, per the precedent used for the JSON Parameters/Tool Invocation closing-directive removal (two consecutive absent passes following long stability). Per-skill file (`verify.skill.md`) removed from the baseline that pass. (`code-review` was also confirmed removed alongside `verify` on 2026-07-23, but see above — it has since reappeared as of 2026-08-07.)
- **Confirmed removed (2026-07-26)**: `deep-research` — absent in both the 2026-07-23 and 2026-07-26 passes, after being present through 2026-07-20 and stable before that, per the same two-consecutive-absent-passes precedent. Per-skill file (`deep-research.skill.md`) removed from the baseline that pass.
- `simplify`'s cross-reference to `/code-review` ("use /code-review for that") is live/accurate again now that `code-review` has returned.
- Individual skill definitions tracked in per-skill files: `[name].skill.md`
  - `dataviz.skill.md`
  - `update-config.skill.md`
  - `keybindings-help.skill.md`
  - `code-review.skill.md`
  - `simplify.skill.md`
  - `fewer-permission-prompts.skill.md`
  - `loop.skill.md`
  - `schedule.skill.md`
  - `claude-api.skill.md`
  - `run.skill.md`
  - `init.skill.md`
  - `security-review.skill.md`

## Subagent Types Reminder
Issued at session start as a standalone `<system-reminder>` (separate from the `Agent` tool's own JSON description, which only says "Available agent types are listed in <system-reminder> messages in the conversation" and does not enumerate them):
> "Available agent types for the Agent tool:
> - claude: Catch-all for any task that doesn't fit a more specific agent. FleetView's default when no agent name is typed. (Tools: *)
> - Explore: Fast read-only search agent for locating code. Use it to find files by pattern (eg. "src/components/**/*.tsx"), grep for symbols or keywords (eg. "API endpoints"), or answer "where is X defined / which files reference Y." Do NOT use it for code review, design-doc auditing, cross-file consistency checks, or open-ended analysis — it reads excerpts rather than whole files and will miss content past its read window. When calling, specify search breadth: "quick" for a single targeted lookup, "medium" for moderate exploration, or "very thorough" to search across multiple locations and naming conventions. (Tools: All tools except Agent, Artifact, ExitPlanMode, Edit, Write, NotebookEdit)
> - general-purpose: General-purpose agent for researching complex questions, searching for code, and executing multi-step tasks. When you are searching for a keyword or file and are not confident that you will find the right match in the first few tries use this agent to perform the search for you. (Tools: *)
> - Plan: Software architect agent for designing implementation plans. Use this when you need to plan the implementation strategy for a task. Returns step-by-step plans, identifies critical files, and considers architectural trade-offs. (Tools: All tools except Agent, Artifact, ExitPlanMode, Edit, Write, NotebookEdit)
> - statusline-setup: Use this agent to configure the user's Claude Code status line setting. (Tools: Read, Edit)
>
> When you launch multiple agents for independent work, send them in a single message with multiple tool uses so they run concurrently."
- This block quote is the verbatim, complete text of the reminder (confirmed this pass — previously stored with a "..." mid-quote elision for Explore/general-purpose/Plan; no separate fuller version exists in `embedded-tools.md`, so the prior cross-reference to that file's `## Agent` section for "full per-type descriptions" was stale and has been removed)
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
