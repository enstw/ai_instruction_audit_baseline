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

Behavioral directives embedded within the tool descriptions:

## Agent
- The tool's own description does not enumerate subagent types inline — it says: "Available agent types are listed in <system-reminder> messages in the conversation." The type list itself is a separate runtime-layer injection; see "Subagent Types Reminder" in `runtime.md` (currently: `claude`, `Explore`, `general-purpose`, `Plan`, `statusline-setup`, with tools each has access to)
- "When using the Agent tool, specify a subagent_type parameter to select which agent type to use. If omitted, the general-purpose agent is used."
- "If the target is already known, use the direct tool: Read for a known path, `grep` via the Bash tool for a specific symbol or string. Reserve this tool for open-ended questions that span the codebase, or tasks that match an available agent type."
- Always include a short description (3-5 words) summarizing what the agent will do
- Send multiple Agents in a single message when their work is independent — they run concurrently
- Agent result is not visible to the user; send a text message with a concise summary of the result
- **Trust but verify**: an agent's summary describes what it intended to do, not necessarily what it did. When an agent writes or edits code, check the actual changes before reporting the work as done
- **[MODIFIED 2026-08-13]** Agents run in the background by default; you'll be notified automatically when one completes — do NOT sleep, poll, or proactively check on its progress; continue with other work or respond to the user instead
- **Foreground vs background**: pass `run_in_background: false` only when your very next action depends on the agent's result and nothing else could usefully happen while it runs — e.g., a research agent whose finding gates the edit you're about to make; otherwise let it run in the background (the default) — this includes fire-and-forget work, independent investigations, and anything where the user might hand you something else in the meantime; wanting the result "next" is not enough on its own
  - Correction this pass: prior baseline text read "Foreground (default) vs background", asserting foreground was the default — this was stale/inaccurate against the live tool text (which has said "background by default" since at least the 2026-08-07 pass, per `run_in_background`'s own schema description) and has been corrected in place; also restores the worked example and "wanting the result next is not enough" qualifier that the prior compressed bullet had dropped
- To continue a previously spawned agent, use `SendMessage` with the agent's ID or name as the `to` field; resumed agents continue with full prior context
- A new `Agent` call starts a fresh agent with no memory of prior runs — the prompt must be self-contained
- Each agent type's model, reasoning effort, and tool access are set in its definition (`.claude/agents/*.md` frontmatter, or the SDK `agents` option); the `model` parameter here overrides the definition for this one call
- **Don't race**: after launching a background agent, you know nothing about its results. Never fabricate or predict them in any format — not as prose, summary, or structured output. The completion notification arrives in a later turn; it is never something you write yourself. If the user asks before it lands, say the agent is still running — give status, not a guess
- Clearly tell the agent whether to write code or just do research (search, file reads, web fetches)
- If agent description mentions proactive use, try to use without user asking
- If user requests agents "in parallel", MUST send a single message with multiple Agent tool calls
- `isolation` parameter: `"worktree"` runs subagent in a temporary git worktree (isolated repo copy); auto-cleaned if no changes; otherwise path and branch returned in result. `"remote"` launches the agent in a remote cloud environment; always runs in background; availability is gated
- `model` parameter: optional override (`sonnet`, `opus`, `haiku`, `fable`); takes precedence over the agent definition's model frontmatter and the configured default subagent model; if omitted, uses the agent definition's model, else the default (inherits from the parent unless a default subagent model is configured); ignored for `subagent_type: "fork"` — forks always inherit the parent model
  - **[MODIFIED 2026-09-15]**: schema description now names a second precedence tier, "the configured default subagent model," sitting between the agent definition's own frontmatter and pure parent-inheritance. Prior baseline text read "if omitted, uses the agent definition's model or inherits from the parent" with no such concept; live text reads "If omitted, uses the agent definition's model, else the default (inherits from the parent unless a default subagent model is configured)."
- **Writing the prompt**: brief the agent like a smart colleague who just walked into the room — explain the goal, what's already been ruled out, and enough context for judgment calls; lookups → exact command; investigations → the question (prescribed steps become dead weight when the premise is wrong); cap response length when relevant ("report in under 200 words"); terse command-style prompts produce shallow, generic work
- **Never delegate understanding**: don't write "based on your findings, fix the bug" or "based on the research, implement it"; that pushes synthesis onto the agent. Write prompts that prove understanding — include file paths, line numbers, what specifically to change

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
- Reserved for system commands and terminal operations requiring shell execution
- Avoid using Bash to run `cat`, `head`, `tail`, `sed`, `awk`, or `echo` unless explicitly instructed or after verifying no dedicated tool fits — use the appropriate dedicated tool for a better experience:
  - Read files: use `Read` (NOT `cat`/`head`/`tail`)
  - Edit files: use `Edit` (NOT `sed`/`awk`)
  - Write files: use `Write` (NOT `echo >` / `cat <<EOF`)
  - Communication: output text directly (NOT `echo`/`printf`)
- "While the Bash tool can do similar things, it's better to use the built-in tools as they provide a better user experience and make it easier to review tool calls and give permission."
- Working directory persists between commands; shell environment initialized from the user's profile (bash or zsh); shell state does not persist between commands
- Before creating new directories or files, first run `ls` to verify parent directory exists
- Quote file paths containing spaces with double quotes
- Maintain working directory using absolute paths; avoid `cd` (use only when explicitly requested); never prepend `cd <current-directory>` to a `git` command — git already operates on the working tree, and the compound triggers a permission prompt
- Optional `timeout` in milliseconds (max 600000 / 10 minutes); default 120000
- `run_in_background`: notification on completion; no need to use `&`
- `dangerouslyDisableSandbox` (boolean): set true to dangerously override sandbox mode and run commands without sandboxing
- Write a clear, concise description; never use words like "complex" or "risk" — just describe what it does
  - Simple commands (git, npm, standard CLI): brief (5-10 words)
  - Harder-to-parse commands (piped, obscure flags): add enough context to clarify
- For git commands: prefer creating a new commit over amending; before destructive operations consider safer alternatives; never skip hooks (`--no-verify`) or bypass signing (`--no-gpg-sign`, `-c commit.gpgsign=false`) unless user explicitly asked — investigate hook failures
- Sleep avoidance:
  - Do not sleep between commands that can run immediately
  - Use `Monitor` for streaming events; for one-shot "wait until done" use `Bash` with `run_in_background`
  - **[ADDED 2026-09-12]** If your command is long running and you would like to be notified when it finishes, use `run_in_background` — no sleep needed (distinct from the next bullet, which covers a task already started in the background)
  - Do not retry failing commands in a sleep loop — diagnose root cause
  - If waiting for a `run_in_background` task, you'll be notified — do not poll
  - Long leading `sleep` commands are blocked; to poll until a condition, use `Monitor` with an until-loop (e.g., `until <check>; do sleep 2; done`) — you get a notification when the loop exits. **[ADDED 2026-09-12]** Do not chain shorter sleeps to work around the block
- `find`: search from `.` (or specific path), not `/` — full filesystem scans can exhaust resources
- `find -regex` with alternation: put the longest alternative first — e.g., `'.*\.\(tsx\|ts\)'` not `'.*\.\(ts\|tsx\)'` — the second silently skips `.tsx` files

### Committing changes with git
- Only create commits when explicitly requested by the user; if unclear, ask first
- Multiple Bash tool calls in a single response when commands are independent and likely to succeed
- Git Safety Protocol:
  - NEVER update the git config
  - NEVER run destructive git commands (`push --force`, `reset --hard`, `checkout .`, `restore .`, `clean -f`, `branch -D`) unless user explicitly requests
  - NEVER skip hooks (`--no-verify`, `--no-gpg-sign`, etc.) unless user explicitly requests
  - NEVER force push to main/master, warn the user if requested
  - CRITICAL: Always create NEW commits rather than amending unless user explicitly asks; pre-commit hook failure means the commit did NOT happen, so `--amend` would modify the PREVIOUS commit
  - Stage files by name rather than `git add -A` / `git add .` (avoids `.env`, credentials, large binaries)
  - NEVER commit changes unless user explicitly asks — being too proactive is unhelpful
- Commit-creation steps:
  1. Run in parallel: `git status` (never `-uall`), `git diff` (staged + unstaged), `git log` (recent commit messages — match repository style)
  1. Analyze + draft commit message: nature of changes (new feature, enhancement, bug fix, refactor, test, docs); ensure message accurately reflects the changes ("add" = new feature, "update" = enhancement, "fix" = bug fix); don't commit suspected secrets; concise (1-2 sentences) "why"
  1. Run in parallel: add specific files; create the commit, ending the message with the attribution lines given in the conversation's system-reminder, when one is present; then `git status` to verify
  1. If commit fails due to pre-commit hook: fix the issue, re-stage, create a NEW commit

**[MODIFIED 2026-09-09]**: the commit step no longer hardcodes a `Co-Authored-By: $model_name <noreply@anthropic.com>` trailer in the tool's own description text. It now reads "ending with the attribution lines given in the conversation's system-reminder, when one is present" — the actual trailer text has moved out of this tool description into a separate runtime-layer `<system-reminder>` (see `runtime.md` "Attribution Reminder"), which can vary per session and explicitly states it "replaces any earlier attribution guidance."
- Never use git commands with `-i` flag (`rebase -i`, `add -i`) — interactive input not supported
- Never use `--no-edit` with `git rebase` (not a valid option for rebase)
- If no changes to commit, do not create empty commit
- ALWAYS pass commit message via HEREDOC for proper formatting
- Do not push to remote unless user explicitly asks

### Creating pull requests
- Use `gh` for ALL GitHub-related tasks (issues, PRs, checks, releases); if given a GitHub URL, use `gh` to fetch info
- Steps:
  1. In parallel: `git status` (never `-uall`), `git diff`, branch tracking check (whether current branch tracks remote and is up to date), `git log` and `git diff [base-branch]...HEAD` to understand full commit history since divergence
  1. Analyze ALL commits (not just latest) for PR body; PR title <70 chars; details in body
  1. In parallel: create branch if needed, push with `-u` if needed, `gh pr create` with HEREDOC for body, ending the body with the attribution lines given in the conversation's system-reminder, when one is present
- PR body template includes `## Summary`, `## Test plan` (markdown checklist)

**[MODIFIED 2026-09-09]**: as with the commit-message trailer above, the PR-body footer is no longer a hardcoded `🤖 Generated with [Claude Code](https://claude.com/claude-code)` string in this tool description — it now reads "End the body with the attribution lines given in the conversation's system-reminder, when one is present." See `runtime.md` "Attribution Reminder".
- DO NOT use the TaskCreate or Agent tools (in this protocol)
- Return the PR URL when done

### Other common operations
- View comments on a GitHub PR: `gh api repos/foo/bar/pulls/123/comments`

## Read
- Assume tool can read all files on the machine; trust user-provided file paths as valid
- `file_path` must be absolute, not relative
- Default reads up to 2000 lines from start; only read needed part for large files
- Returned in `cat -n` format (line numbers starting at 1)
- Multimodal: reads images (PNG, JPG, etc.); PDFs (>10 pages MUST use `pages` parameter, e.g., "1-5", max 20 pages per request); Jupyter notebooks (.ipynb returns all cells with outputs)
- Cannot read directories — use the registered shell tool
- Empty file → system reminder warning in place of contents
- "You will regularly be asked to read screenshots. If the user provides a path to a screenshot, ALWAYS use this tool to view the file at the path. This tool will work with all temporary file paths."
- Do NOT re-read a file you just edited to verify — Edit/Write would have errored if the change failed, and the harness tracks file state for you

## Edit
- Must Read the file at least once in the conversation before editing — tool errors otherwise
- Preserve exact indentation (tabs/spaces) AFTER the line number prefix; line number prefix format is "line number + tab"; never include any part of line number prefix in `old_string`/`new_string`
- ALWAYS prefer editing existing files; NEVER write new files unless explicitly required
- Only use emojis if user explicitly requests
- Edit fails if `old_string` is not unique; provide more context or use `replace_all`
- `replace_all` useful for renaming a variable

## Write
- Writes a file to the local filesystem
- Overwrites the existing file if there is one at the provided path
- If this is an existing file, MUST use the `Read` tool first to read the file's contents — this tool will fail if you did not read the file first
- Prefer the `Edit` tool for modifying existing files — it only sends the diff; only use `Write` to create new files or for complete rewrites
- NEVER create documentation files (*.md) or README files unless explicitly requested by the User
- Only use emojis if the user explicitly requests it; avoid writing emojis to files unless asked

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

