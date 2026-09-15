# Runtime Injections (`<system-reminder>`)

Injected via `<system-reminder>` tags during the session, not present at system prompt start:

## Skill System
- Available system-built skills observed: `dataviz`, `update-config`, `keybindings-help`, `code-review`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`, `workflow-authoring`, `run`, `init`, `security-review`
- Custom user-defined skills (e.g., gstack-suffixed) appear in the same list but are out-of-scope per `AUDIT_RULE.md`
- **[ADDED 2026-08-28]**: `workflow-authoring` — new system-built skill, not previously tracked: "Reference for writing a Workflow tool script (script API and gotchas, resume, quality patterns, worked examples). Load before authoring a script for a workflow the user already opted into; it does not itself authorize running one." Ties directly into the `Workflow` tool's own description, which now cross-references "the workflow authoring reference" by name (see `embedded-tools.md` `## Workflow`). Per-skill file added: `workflow-authoring.skill.md`.
- **[MODIFIED 2026-08-28]** `loop`: interval-omission behavior changed — previously "defaults to 10m" when no interval given; now "Omit the interval to let the model self-pace." (dynamic mode, tied to the `ScheduleWakeup` tool's dynamic-loop pacing). See `loop.skill.md`.
- **`code-review` description expanded again (2026-08-10)**: the effort-level parenthetical gained a third tier, `ultra`: "Review the current diff, or a PR number/branch/path target, for correctness bugs and reuse/simplification/efficiency cleanups at the given effort level (low/medium: fewer, high-confidence findings; high→max: broader coverage, may include uncertain findings; ultra: deep multi-agent review in the cloud); with no level given, it reuses the level you typed last. Pass --comment to post findings as inline PR comments, or --fix to apply the findings to the working tree after the review." This lines up with the new "ultrareview" guidance added to `instruction.md`'s Session-Specific Guidance this pass (`/code-review ultra` / deprecated `/ultrareview` alias, launches a billed multi-agent cloud review).
  - **[ADDED 2026-09-15]**: one more sentence appended to the description, not previously tracked: "For ultra on a GitHub.com PR target, --post asks to post the finished review's findings to the PR as a single comment from the user's GitHub account (not a review; the launch dialog still confirms in interactive sessions, while non-interactive mode posts on the flag alone) and --no-post hides that option." New `--post`/`--no-post` flags for `ultra` reviews of GitHub.com PR targets.
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
  - `workflow-authoring.skill.md`
  - `run.skill.md`
  - `init.skill.md`
  - `security-review.skill.md`

## Subagent Types Reminder
Issued at session start as a standalone `<system-reminder>` (separate from the `Agent` tool's own JSON description, which only says "Available agent types are listed in <system-reminder> messages in the conversation" and does not enumerate them):
> "Available agent types for the Agent tool:
> - claude: Catch-all for any task that doesn't fit a more specific agent. FleetView's default when no agent name is typed. (Tools: *)
> - Explore: Fast read-only search agent for locating code. Use it to find files by pattern (eg. "src/components/**/*.tsx"), grep for symbols or keywords (eg. "API endpoints"), or answer "where is X defined / which files reference Y." Do NOT use it for code review, design-doc auditing, cross-file consistency checks, or open-ended analysis — it reads excerpts rather than whole files and will miss content past its read window. When calling, specify search breadth: "quick" for a single targeted lookup, "medium" for moderate exploration, or "very thorough" to search across multiple locations and naming conventions. (Tools: All tools except Agent, Artifact, ArtifactComments, ArtifactData, ArtifactCheck, ExitPlanMode, Edit, Write, NotebookEdit)
> - general-purpose: General-purpose agent for researching complex questions, searching for code, and executing multi-step tasks. When you are searching for a keyword or file and are not confident that you will find the right match in the first few tries use this agent to perform the search for you. (Tools: *)
> - Plan: Software architect agent for designing implementation plans. Use this when you need to plan the implementation strategy for a task. Returns step-by-step plans, identifies critical files, and considers architectural trade-offs. (Tools: All tools except Agent, Artifact, ArtifactComments, ArtifactData, ArtifactCheck, ExitPlanMode, Edit, Write, NotebookEdit)
> - statusline-setup: Use this agent to configure the user's Claude Code status line setting. (Tools: Read, Edit)
>
> When you launch multiple agents for independent work, send them in a single message with multiple tool uses so they run concurrently."
- This block quote is the verbatim, complete text of the reminder (confirmed this pass — previously stored with a "..." mid-quote elision for Explore/general-purpose/Plan; no separate fuller version exists in `embedded-tools.md`, so the prior cross-reference to that file's `## Agent` section for "full per-type descriptions" was stale and has been removed)
- **[MODIFIED 2026-09-03]** Explore's and Plan's tool-exclusion lists gained three entries: `ArtifactComments`, `ArtifactData`, `ArtifactCheck`, inserted between `Artifact` and `ExitPlanMode`. Prior baseline read "...except Agent, Artifact, ExitPlanMode, Edit, Write, NotebookEdit" for both; live text now reads "...except Agent, Artifact, ArtifactComments, ArtifactData, ArtifactCheck, ExitPlanMode, Edit, Write, NotebookEdit" for both.
- The trailing "send independent agents in one message" directive is distinct from the Agent tool's own "if the user asks for parallel, MUST send one message" bullet — this one is a general default for any independent agent work, not conditioned on the user explicitly requesting parallelism

## Deferred-Tools Availability Reminder
Issued at session start (and possibly again after schema fetches):
> "The following deferred tools are now available via ToolSearch. Their schemas are NOT loaded — calling them directly will fail with InputValidationError. Use ToolSearch with query 'select:<name>[,<name>...]' to load tool schemas before calling them: [list]"

## Token Budget Reminder
A standalone `<system-reminder>`: bundled right after the deferred-tools/agent-types/skills reminders at session start, then recurs standalone after subsequent turns:
> "<total_tokens>15000000 tokens left</total_tokens>"
Unlike the Project Instruction File Delivery block below, it carries no leading preface ("As you answer the user's questions...") and no trailing relevance-disclaimer wrapper — it stands alone as a bare tagged value.

**Confirmed dynamic (2026-08-25)**: across one session, the value decremented on every subsequent turn (15000000 → 14965898 → 14957165 → 14956160 → 14943539 → 14910011 → 14903731), confirming it tracks remaining output-token budget rather than being a fixed constant. Also recurs far more frequently than the "observed twice" first noted at ADDED 2026-08-19 — this pass it reappeared after nearly every turn boundary, not just once more later in the session.

## Project Instruction File Delivery
- Project instruction files (e.g., `CLAUDE.md`) delivered via `<system-reminder>` with the leading preface:
  > "As you answer the user's questions, you can use the following context:"
- Followed by tagged sections, each with a heading tag and content:
  - `# claudeMd` — "Codebase and user instructions are shown below. Be sure to adhere to these instructions. IMPORTANT: These instructions OVERRIDE any default behavior and you MUST follow them exactly as written." Followed by `Contents of $path/CLAUDE.md (project instructions, checked into the codebase):` and the file body. Multiple project instruction files (parent + subdirectory) are concatenated under the same `# claudeMd` block.
  - `# userEmail` — "The user's email address is $email." (only present when user email is configured; may be omitted)
  - `# currentDate` — "Today's date is $date."
- Trailing wrapper:
  > "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task."

**[MODIFIED 2026-09-12, confirmed stable 2026-09-15]**: the `# claudeMd` block does NOT carry the leading preface or trailing wrapper described above — it arrives as its own standalone `<system-reminder>` containing only the "Codebase and user instructions..." sentence plus the `Contents of $path/CLAUDE.md (project instructions, checked into the codebase):` bodies (still concatenated for parent + subdirectory files), with nothing before it and nothing after it. The preface ("As you answer the user's questions...") + trailing wrapper ("IMPORTANT: this context may or may not be relevant...") instead wrap only the `# gitStatus` content, in an immediately-following but separate `<system-reminder>` tag. `# currentDate` ("Today's date is $date.") appears as a third, fully standalone reminder near the end of the session — after the deferred-tools/agent-types/skills/token-budget reminders — with no preface and no trailing wrapper at all. No `# userEmail` block was observed in either the 2026-09-12 or 2026-09-15 pass (consistent with it being conditional on email configuration). Net effect: the three tagged sections this rule previously documented as co-delivered under one shared preface/trailer are split across three independently-wrapped (or unwrapped) reminders. Observed identically across two consecutive passes (2026-09-12, 2026-09-15) — per the two-consecutive-pass precedent used elsewhere in this baseline, now treated as the stable current format rather than single-pass-pending.

## Concealment-Bearing Reminders
Runtime injections that explicitly forbid surfacing themselves to the user:
- **Task-tools nudge** (recurs across the session whenever the task tools haven't been used): "The task tools haven't been used recently. If you're working on tasks that would benefit from tracking progress, consider using TaskCreate to add new tasks and TaskUpdate to update task status (set to in_progress when starting, completed when done). Also consider cleaning up the task list if it has become stale. Only use these if relevant to the current work. This is just a gentle reminder - ignore if not applicable." — note: no concealment instruction ("NEVER mention this reminder to the user") in this version
- **Date-change notification** (when system clock advances mid-session): "The date has changed. Today's date is now $date. DO NOT mention this to the user explicitly because they are already aware."
- See "System" section in `instruction.md` for general `<system-reminder>` behavior documentation

## Attribution Reminder
**[ADDED 2026-09-09]**: standalone `<system-reminder>`, observed at session start, not previously tracked:
> "Attribution for git commits and pull requests you create from here on (this replaces any earlier attribution guidance):
> - End git commit messages with:
> Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>
> - End pull request descriptions with:
> 🤖 Generated with [Claude Code](https://claude.com/claude-code)"

**[MODIFIED 2026-09-15]**: the parenthetical has been substantially expanded. Live text:
> "Attribution for git commits and pull requests you create from here on (this replaces Claude Code's own earlier attribution guidance, such as a previous copy of this reminder; the user's own instructions about these lines, such as a CLAUDE.md or memory rule, take precedence over this reminder, but do not add attribution lines this reminder leaves out):"
Prior baseline parenthetical read only "(this replaces any earlier attribution guidance)". New clauses: (1) scopes "replaces" specifically to "Claude Code's own earlier attribution guidance, such as a previous copy of this reminder" rather than any/all earlier guidance; (2) adds that the user's own instructions (CLAUDE.md, memory rule) take precedence over this reminder; (3) adds "but do not add attribution lines this reminder leaves out" — i.e., a user rule may suppress a line this reminder specifies, but must not be read as license to add lines beyond what this reminder specifies. The trailer body itself (Co-Authored-By / Generated with Claude Code lines) is unchanged this pass.
- The `Co-Authored-By` name matches the session's current model name ($model_name); generalize accordingly
- The reminder's own text explicitly says it "replaces any earlier attribution guidance" — this corroborates the companion change in `embedded-tools.md`'s `Bash` tool `## Committing changes with git` / `## Creating pull requests` sections, where the previously hardcoded `Co-Authored-By` trailer and `🤖 Generated with [Claude Code]` PR footer have been replaced with a generic instruction to use "the attribution lines given in the conversation's system-reminder, when one is present." Attribution content has moved from static tool-description text to this dynamic, per-session runtime reminder.

## Tagged Acknowledgments
- After a successful `ToolSearch` lookup, a `Tool loaded.` user-tagged message is appended; the message bears no actual user input
