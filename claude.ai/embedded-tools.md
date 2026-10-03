# Embedded Tool Definitions

Tool definition blocks are the first injection. Always-loaded tools at session start:

- `Agent`
- `Bash`
- `Edit`
- `ListAgents`
- `Read`
- `ReportFindings`
- `ScheduleWakeup`
- `Skill`
- `ToolSearch`
- `Workflow`
- `Write`

(Cron tools are NOT embedded — they are deferred and loaded via `ToolSearch`.
`Glob` and `Grep` have been removed — no longer available as embedded or deferred tools.)

**`Write` restored (2026-07-26 pass)**: present again as a full standalone embedded tool definition,
after being absent (with only cross-references in surrounding text) across the 2026-07-17 and
2026-07-23 passes. See `## Write` below for the restored definition.

**`ListAgents` added (2026-08-07 pass)**: new embedded tool, not previously tracked. See `## ListAgents`
below.

**`ToolSearch` restored to the visible embedded-tool block (2026-08-10 pass)**: back in the printed
`<functions>` block at session start, as the 9th of 11 always-loaded tools (between `Skill` and
`Workflow`), reversing the single-pass 2026-08-07 observation that it had gone undocumented. Description
text unchanged from the last-known version in `## ToolSearch` below. It was never actually uncallable —
the 2026-08-07 pass already confirmed it functional despite being undocumented — so this is a visibility
restoration, not a functional change.

**[ADDED 2026-10-03]** a standalone paragraph following the tool/skill listings in the prompt preamble, not previously tracked: "If you intend to call multiple tools and there are no dependencies between the calls, make all of the independent calls in the same function_calls block, otherwise you MUST wait for previous calls to finish first to determine the dependent values."

Behavioral directives embedded within the tool descriptions:

## Agent
**[MODIFIED 2026-09-30]** description substantially condensed; prior bullets ("If the target is already known, use the direct tool...", short-description requirement, "Trust but verify", "Don't race", "Writing the prompt", "Never delegate understanding", "If user requests agents 'in parallel', MUST send a single message", "Clearly tell the agent whether to write code or just do research", "If agent description mentions proactive use") are no longer in the live text. Live text:
- "Launch a new agent to handle complex, multi-step tasks. Each agent type has specific capabilities and tools available to it."
- "Available agent types are listed in <system-reminder> messages in the conversation." (type list is a runtime-layer injection; see "Subagent Types Reminder" in `runtime.md`)
- "When using the Agent tool, specify a subagent_type parameter to select which agent type to use. If omitted, the general-purpose agent is used."
- `## When to use`: "Reach for this when the task matches an available agent type, when you have independent work to run in parallel, or when answering would mean reading across several files — delegate it and you keep the conclusion, not the file dumps. For a single-fact lookup where you already know the file, symbol, or value, search directly. Once you've delegated a search, don't also run it yourself — wait for the result."
- "The agent's final report is not shown to the user — relay what matters."
- "Use SendMessage with the agent's ID or name to continue a previously spawned agent with its context intact; a new Agent call starts fresh."
- "Each agent type's model, reasoning effort, and tools come from its definition (`.claude/agents/*.md` frontmatter or SDK `agents`)."
- "`isolation: "worktree"` gives the agent its own git worktree (auto-cleaned if unchanged)."
- "Subagents run in the background by default; you'll be notified when one completes. Pass `run_in_background: false` only when your very next action depends on the result and nothing else could usefully happen while it runs — otherwise background it so the user can interject. Never fabricate or predict a pending agent's results — the notification is never something you write yourself; if the user asks before it arrives, say it's still running."
- Parameters: `description` ("A short (3-5 word) description of the task"), `isolation` (enum `worktree`/`remote`; "Isolation mode. "worktree" creates a temporary git worktree so the agent works on an isolated copy of the repo. "remote" launches the agent in a remote cloud environment (always runs in background; availability is gated)."), `model` (enum `sonnet`/`opus`/`haiku`/`fable`; precedence text unchanged from 2026-09-15), `prompt`, `run_in_background`, `subagent_type`

## ListAgents
- "Lists agents you can SendMessage to — in-process subagents you spawned, the teammates on your team, other local Claude sessions on this machine, your Claude sessions running in the cloud (when this session has cloud access; a cloud session receives your message but cannot message any session back yet — do not ask it to reply, read its answer in its own transcript), and (when Remote Control is connected here) your account's other sessions — Remote Control sessions on other machines and cloud sessions, each row labeled by kind."
- **[ADDED 2026-08-22]**: "the teammates on your team," inserted between "in-process subagents you spawned," and "other local Claude sessions on this machine" — explicitly calls out agent-team teammates as a distinct listed category, separate from in-process subagents and local sessions.
- **[ADDED 2026-08-16]**: new parenthetical inserted into the cloud-access clause: "; a cloud session receives your message but cannot message any session back yet — do not ask it to reply, read its answer in its own transcript" — clarifies that cloud sessions are one-way SendMessage targets this pass.
- **[MODIFIED 2026-08-13]**: the closing clause changed from "your Remote Control sessions on other machines" to "your account's other sessions — Remote Control sessions on other machines and cloud sessions, each row labeled by kind" — now explicitly folds cloud sessions into the Remote-Control-gated bucket and adds the "each row labeled by kind" detail describing the listing's presentation.
- **Modified (2026-08-10 pass)**: the prior "remote bridge sessions, which are reply-only — you can message one only in reply, after it messages you first, and no connector reaches it by name either" language is gone; Remote Control sessions on other machines are now described the same way as any other listed peer, without the reply-only / no-by-name-addressing restriction.
- "Names are the address: send with `SendMessage({to: \"<name>\", message: \"...\"})`, copying the name exactly as a row prints it. Append a row's ` [ref]` only when the bare name is not enough — two rows share it, or an error asks you to disambiguate."
- Parameters: `channel` (string, max 256 chars) — "Not available in this build; leave unset"; `q` (string, max 256 chars) — "Not available in this build; leave unset". No required parameters.
- Directly ties into the expanded `SendMessage` cross-session addressing (see `deferred-tools.md` `## SendMessage`) — `ListAgents` is the discovery step, `SendMessage` is the send step.

## Bash
**[MODIFIED 2026-09-30]** description substantially condensed; the prior `Committing changes with git` / `Creating pull requests` / `Other common operations` subsections, Git Safety Protocol, sleep-avoidance bullets, `find` bullets, and "Quote file paths" / "run `ls` first" bullets are no longer in the live text. Live text:
- "Executes a bash command and returns its output."
- "Working directory persists between calls, but prefer absolute paths — `cd` in a compound command can trigger a permission prompt. Shell state (env vars, functions) does not persist; the shell is initialized from the user's profile."
- "IMPORTANT: Avoid using this tool to run `cat`, `head`, `tail`, `sed`, `awk`, or `echo` commands, unless explicitly instructed or after you have verified that a dedicated tool cannot accomplish your task. Instead, use the appropriate dedicated tool as this will provide a much better experience for the user."
- "Command output is displayed to you, not reliably to the user."
- "`timeout` is in milliseconds: default 120000, max 600000 for a foreground command."
- "`run_in_background` runs the command detached: it keeps running across turns and re-invokes you when it exits. With it, `timeout` is how long the command may run in the background (default 1800000, max 7200000); at that limit it is stopped and you are re-invoked. No `&` needed. Foreground `sleep` is blocked; use Monitor with an until-loop to wait on a condition."
- `# Git`:
  - "Interactive flags (`-i`, e.g. `git rebase -i`, `git add -i`) are not supported in this environment."
  - "Use the `gh` CLI for GitHub operations (PRs, issues, API)."
  - "Commit or push only when the user asks. If on the default branch, branch first."
  - "End git commit messages and PR bodies with the attribution lines given in the conversation's system-reminder, when one is present."
- Parameters: `command`; `dangerouslyDisableSandbox` ("Set this to true to dangerously override sandbox mode and run commands without sandboxing."); `description` ("Clear, concise description of what this command does in active voice. Never use words like "complex" or "risk" in the description - just describe what it does. Say what the command does in plain words: do not echo the command's text, its flags, or file paths - the user reads this description, often without seeing the command." with simple-command (5-10 words) and harder-to-parse examples); `run_in_background`; `timeout`

## Read
**[MODIFIED 2026-09-30]** condensed; "Assume tool can read all files on the machine", "You will regularly be asked to read screenshots..." and "Cannot read directories — use the registered shell tool" no longer present. Live text:
- "Reads a file from the local filesystem."
- "`file_path` must be an absolute path."
- "Reads up to 2000 lines by default."
- "When you already know which part of the file you need, only read that part. This can be important for larger files."
- "Results are returned using cat -n format, with line numbers starting at 1"
- "Reads images (PNG, JPG, …) and presents them visually. Reads PDFs via the `pages` parameter (e.g. "1-5", max 20 pages/request; required for PDFs over 10 pages). Reads Jupyter notebooks (.ipynb) as cells with outputs."
- "Reading a directory, a missing file, or an empty file returns an error or system reminder rather than content."
- "Do NOT re-read a file you just edited to verify — Edit/Write would have errored if the change failed, and the harness tracks file state for you."

## Edit
**[MODIFIED 2026-09-30]** condensed; emoji, "ALWAYS prefer editing existing files", and "replace_all useful for renaming" bullets no longer present. Live text:
- "Performs exact string replacement in a file."
- "You must Read the file in this conversation before editing, or the call will fail."
- "`old_string` must match the file exactly, including indentation, and be unique — the edit fails otherwise. Strip the Read line prefix (line number + tab) before matching."
- "`replace_all: true` replaces every occurrence instead."

## Write
**[MODIFIED 2026-09-30]** condensed; the Edit-preference, no-documentation-files, and emoji bullets no longer present. Live text:
- "Writes a file to the local filesystem, overwriting if one exists."
- "When to use: creating a new file, or fully replacing one you've already Read. Overwriting an existing file you haven't Read will fail. For partial changes, use Edit instead."

## Skill
A skill is a packaged set of instructions the user or project has set up for a particular kind of task (deploy steps, a review checklist, a repo-specific workflow). Available skills appear in a system-reminder listing with one-line descriptions. When the task at hand is one a listed skill covers, call this tool first — the skill's instructions load into the turn for you to follow in place of your default approach; some skills instead run in a subagent and return the finished result. **[ADDED 2026-08-10]** A skill that runs in the background returns only the agent's name — its result arrives later as a task notification, so don't wait on it or invoke it again in the meantime. Users may also ask for one by name (`/<name>`, or "slash command"); that's a request to invoke it.
- `skill`: exact name from the listing, no leading slash. Plugin skills use `plugin:skill`. Directory-scoped skills are listed with a path prefix (`apps/web:deploy`); when both scoped and unscoped variants of a name exist, pick the one whose directory contains the files you're working on (most specific wins; unscoped otherwise).
- `args`: optional arguments to pass through
- Only names from the listing (or that the user typed explicitly) are valid. Built-in CLI commands (`/help`, `/clear`, …) aren't skills
- If a `<command-name>` block is already present this turn, the skill is loaded — follow it directly rather than calling again

## ToolSearch
- Fetches full schema definitions for deferred tools so they can be called
- Deferred tools appear by name in `<system-reminder>` messages; until fetched, only the name is known — there is no parameter schema, so the tool cannot be invoked
- Result format: each matched tool appears as one `<function>{...}</function>` line inside a `<functions>` block — the same encoding as the tool list at the top of the prompt; once a tool's schema appears, it is callable exactly like any tool defined at the top of the prompt
- Query forms:
  - `select:Read,Edit,Grep` — fetch these exact tools by name
  - `notebook jupyter` — keyword search, up to `max_results` best matches
  - `+slack send` — require "slack" in the name, rank by remaining terms

## ReportFindings
- Report code-review findings as a typed list so the host UI can render them
- Use only when the active code-review instructions say to report findings with this tool; otherwise follow whatever output format those instructions specify
- Call once with the verified findings ranked most-severe first (empty array if nothing survived verification); do not also print the findings as text
- When re-reporting after applying fixes (only if the apply instructions ask for it), set `outcome` on each finding to what actually happened
- `findings` (max 32 items): each item has `file` (required, repo-relative path), `summary` (required, one-sentence statement of the defect), `failure_scenario` (required, concrete inputs/state → wrong output/crash), `line` (optional, 1-indexed), `category` (optional, short kebab-case slug of the finding type, e.g. "correctness", "simplification", "efficiency", "test-coverage"; max 40 chars), `short_summary` (optional, compressed label for compact UI: the claim alone, no rationale or consequence clause; max 60 chars), `verdict` (optional enum: `CONFIRMED`/`PLAUSIBLE` — set when a verify pass ran, absent on inline-only reviews), `outcome` (optional enum: `fixed`/`skipped`/`no_change_needed` — set ONLY when re-reporting after applying fixes)
- `level` (optional enum: `low`/`medium`/`high`/`xhigh`/`max`): effort level the review ran at

## ScheduleWakeup
- Schedule when to resume work in `/loop` dynamic mode — when the user invoked `/loop` without an interval, asking you to self-pace iterations of a specific task
- **Do NOT schedule a short-interval wakeup to poll for background work you started** — when harness-tracked work finishes, you are re-invoked automatically, so polling is wasted; instead schedule a long fallback (1200s+) so the loop survives if the work hangs or never notifies; the exception is external work the harness cannot track (a CI run, a deploy, a remote queue) — there, pick a delay matched to how fast that state actually changes
- Pass the same `/loop` prompt back via `prompt` each turn so the next firing repeats the task
- For an autonomous `/loop` (no user prompt), pass the literal sentinel `<<autonomous-loop-dynamic>>` as `prompt` instead — runtime resolves it back to the autonomous-loop instructions at fire time; (there is a similar `<<autonomous-loop>>` sentinel for CronCreate-based autonomous loops; do not confuse the two — ScheduleWakeup always uses the `-dynamic` variant)
- `stop` (boolean): set to true to end the dynamic loop immediately instead of scheduling another wakeup — when true, all other fields are ignored and no further wakeups fire (call with `stop: true`, omitting every other field, to end the loop)
- Picking `delaySeconds`: this session's requests use a 1-hour Anthropic prompt-cache TTL, so effectively every allowed delay (the runtime clamps to `[60, 3600]`) wakes up with conversation context still cached; there is no cache cliff inside that range to pace around, and scheduling extra wakeups just to keep the cache warm is pure waste — never do that; (if the session enters usage overage, later requests drop to the 5-minute TTL — don't try to track or preempt that, the guidance stays the same)
  - Match the delay to what you're actually waiting for, not to cache windows
  - **Actively polling external state the harness can't notify you about** (a CI run, a deploy, a remote queue): pick the delay from how fast that state actually changes — a CI run that takes ~8 minutes deserves one ~480s check, not eight 60s ones
  - **The long fallback heartbeat** (something else — a Monitor, a task notification — is the primary wake signal): 1200s+, so quiet wakeups stay rare
  - **Idle ticks with no specific signal to watch**: default to **1200s–1800s** (20–30 min); the loop still checks back regularly, and the user can always interrupt if they need you sooner
- `reason` field: one short sentence explaining the chosen delay; goes to telemetry and is shown to the user; be specific
- **[ADDED 2026-09-24]** `noop` (boolean): "true = nothing changed (you checked and there is nothing to report). false = something happened worth keeping (edited a file, posted a message, advanced state, surfaced a finding). Consecutive noop:true ticks are collapsed in the user's terminal view and tracked as a streak." Required unless `stop` is true. Not previously tracked in this baseline.

## Workflow

- Execute a workflow script that orchestrates multiple subagents deterministically; runs in the background — returns immediately with a task ID; `<task-notification>` arrives on completion; use `/workflows` to watch live progress
- **ONLY call when the user has explicitly opted into multi-agent orchestration**; workflows can spawn dozens of agents and consume large amounts of tokens — the user must request that scale, not have it inferred
- Explicit opt-in triggers:
  - The user included the keyword `"ultracode"` in their prompt (you'll see a system-reminder confirming it)
  - Ultracode is on for the session (a system-reminder confirms it) — see **Ultracode** in the workflow authoring reference
  - The user directly asked you to run a workflow or use multi-agent orchestration in their own words (`"use a workflow"`, `"run a workflow"`, `"fan out agents"`, `"orchestrate this with subagents"`); the ask must be in the user's words — a task that would merely benefit from a workflow does not count
  - The user invoked a skill or slash command whose instructions tell you to call Workflow
  - The user asked you to run a specific named or saved workflow
- For any other task — even one that would clearly benefit from parallelism — do NOT call this tool; use the Agent tool (if available) for individual subagents, or briefly describe what a multi-agent workflow could do and how much it would roughly cost, and ask the user whether to run it. Mention they can ask for one with `"use a workflow"` in a future message to skip the ask.
- Every script must begin with `export const meta = {...}`: a PURE LITERAL (no variables, calls or interpolation) giving the workflow's `name`, a one-line `description` (shown in the permission dialog) and optionally `phases` — one `{ title, detail? }` per `phase()` call, titles matched exactly
- Pass the script inline via `script` — do not Write it to a file first, and do not also set the tool's `name` input (that selects a saved workflow); it is plain JavaScript, not TypeScript
- Includes a worked "canonical multi-stage pattern" code example (pipeline-by-default review-and-verify sample) directly in the tool description
- Before writing a script, load the `workflow-authoring` skill — the workflow authoring reference: script API and gotchas, resume, the **Ultracode** section, quality patterns, worked examples
- The session carries a default workflow size guideline (observed this pass: "medium — keep workflows under 10 agents"); a guideline, not a hard limit — follow it unless the user's prompt calls for a different scale; the user can raise or remove it with "Dynamic workflow size" in `/config`
  - **[MODIFIED 2026-09-15]**: the medium-guideline agent cap dropped from 15 to 10 (prior baseline: "medium — keep workflows under 15 agents").

**[REMOVED 2026-09-09]**: The live tool description no longer contains any of the following subsections, which this baseline previously tracked as inline tool-description content: **Script format rules** (meta-literal field details, TypeScript/async/Date-Math.random restrictions, no filesystem access), **Script body hooks** (`agent()`/`pipeline()`/`parallel()`/`log()`/`phase()`/`args`/`budget`/`workflow()` reference), **Concurrency** (per-workflow agent caps, 1000-agent lifetime cap, 4096-item batch cap), **Ultracode mode** (standing opt-in behavior, quality-pattern pointer), **DEFAULT: pipeline() over parallel()** (when a barrier is/isn't justified), **Resume** (`runId`/`resumeFromRunId`/prefix-cache mechanics), **Quality patterns** (adversarial verify, perspective-diverse verify, judge panel, loop-until-dry, multi-modal sweep, completeness critic, no-silent-caps), and **Subagents** (return-value convention, `schema` validation, MCP tool access). Also gone: the standalone "Common single-phase workflows you can chain across turns" list (Understand/Design/Review/Research/Migrate) and the "hybrid — scout inline first" guidance paragraph. The tool description now only points to the `workflow-authoring` skill ("load the `workflow-authoring` skill — the workflow authoring reference: script API and gotchas, resume, the Ultracode section, quality patterns, worked examples") instead of inlining this material. This corroborates `workflow-authoring.skill.md`'s 2026-08-28 note that its own internal content (which is out of audit scope — only its listing description is tracked) could not be cross-checked against this inline copy; the inline copy is now confirmed gone from the tool description itself, so there is nothing left here to duplicate. Per the Native Structure/Wording Fidelity rules, this baseline no longer restates that removed content — see `workflow-authoring.skill.md` for the (out-of-scope) skill that now houses it.
- Parameters: `script` (inline, max 524288 chars), `scriptPath` (file path; takes precedence), `name` (predefined workflow), `resumeFromRunId`, `args` (pass arrays/objects as actual JSON values — NOT JSON-encoded strings), `title`/`description` (ignored — set in meta block)
- Every invocation automatically persists its script to a file under the session directory and returns the path in the tool result; to iterate, edit that file with Write/Edit and re-invoke with `{scriptPath: "<path>"}` instead of resending the full script

