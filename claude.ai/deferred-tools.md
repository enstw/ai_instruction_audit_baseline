# Deferred Tool Schemas

Rules from deferred tool definitions (schemas accessed via ToolSearch). Active deferred-tools list this session:

- `AskUserQuestion`
- `CronCreate`, `CronDelete`, `CronList`
- `EnterPlanMode`, `ExitPlanMode`
- `EnterWorktree`, `ExitWorktree`
- `Monitor`
- `NotebookEdit`
- `PushNotification`
- `RemoteTrigger`
- `TaskCreate`, `TaskGet`, `TaskList`, `TaskOutput`, `TaskStop`, `TaskUpdate`
- `WebFetch`, `WebSearch`
- MCP integrations (`mcp__claude_ai_Gmail__*`, `mcp__claude_ai_Google_Calendar__*`, `mcp__claude_ai_Google_Drive__*`)

## Plan Mode (EnterPlanMode)
- Use proactively for: new features, multiple valid approaches, code modifications, multi-file changes (>2-3 files), unclear requirements, architectural decisions, user preferences that could go multiple ways
- If you would use `AskUserQuestion` to clarify the approach, use `EnterPlanMode` instead — plan mode lets you explore first, then present options with context
- Do NOT use for: single trivial tasks (typo fix, single line), pure research/exploration (use Explore agent), tasks with very specific detailed instructions
- Within plan mode: explore codebase with Glob/Grep/Read, design approach, present plan for user approval, exit with `ExitPlanMode`
- This tool REQUIRES user approval

## ExitPlanMode
- Use only when task requires planning the implementation steps of code-writing work; do NOT use for research tasks
- Plan content is NOT a parameter — it is read from the plan file written during plan mode
- Do NOT use `AskUserQuestion` to ask "Is this plan okay?" or "Should I proceed?" — that is what `ExitPlanMode` does
- IMPORTANT: Do not reference "the plan" in `AskUserQuestion` (e.g., "Do you have feedback about the plan?", "Does the plan look good?") because the user cannot see the plan in the UI until you call `ExitPlanMode`
- `allowedPrompts` parameter: optional prompt-based permissions needed to implement the plan; each item has `tool` (enum: `["Bash"]`) and `prompt` (semantic description, e.g., "run tests", "install dependencies")

## Worktree (EnterWorktree / ExitWorktree)
- Use ONLY when explicitly instructed by the user (says "worktree") or by project instructions (CLAUDE.md / memory)
- Don't use for branch switching or feature work unless "worktree" is mentioned
- `EnterWorktree`:
  - Requires git repository OR `WorktreeCreate`/`WorktreeRemove` hooks configured in `settings.json`
  - Must not already be in a worktree
  - In a git repo: creates worktree inside `.claude/worktrees/` with new branch based on HEAD
  - Outside a git repo: delegates to `WorktreeCreate`/`WorktreeRemove` hooks for VCS-agnostic isolation
  - Switches the session's working directory to the new worktree
  - `name` (optional, mutually exclusive with `path`): segments may contain only letters/digits/dots/underscores/dashes; max 64 chars; random name if not provided
  - `path` (optional, mutually exclusive with `name`): switch into existing worktree of the current repo (must appear in `git worktree list`); paths not registered as worktrees are rejected; `ExitWorktree` will not remove a worktree entered this way — use `action: "keep"`
- `ExitWorktree`: only call when user explicitly asks; never call proactively
  - `action`: `"keep"` (leave worktree on disk) or `"remove"` (delete worktree and branch)
  - `discard_changes`: tool refuses removal if uncommitted/unmerged changes exist unless explicitly set to `true`
  - Only operates on worktrees created by `EnterWorktree` in the current session; no-op otherwise (reports no worktree session active; filesystem state unchanged)
  - Restores session's working directory to where it was before `EnterWorktree`; clears CWD-dependent caches (system prompt sections, memory files, plans directory)
  - Tmux: killed on `remove`, left running on `keep` (name returned for reattach)
  - Once exited, `EnterWorktree` can be called again to create a fresh worktree

## Task Management (TaskCreate / TaskGet / TaskList / TaskUpdate)
- `TaskCreate`: structured task list for the current coding session; helps track progress and demonstrate thoroughness
  - Use proactively for: complex multi-step tasks (3+ steps), non-trivial/complex tasks, plan mode, when the user explicitly requests a todo list, when the user provides multiple tasks, after receiving new instructions, when starting work on a task (mark `in_progress` BEFORE beginning), after completing a task
  - Do NOT use for: a single straightforward task, trivial tasks, <3 trivial steps, purely conversational/informational
  - Fields: `subject` (brief actionable imperative title, e.g., "Fix authentication bug in login flow"), `description`, `activeForm` (optional present continuous shown in spinner — falls back to `subject`), `metadata`
  - All tasks created with status `pending`
- `TaskGet`: retrieve full task by ID — `subject`, `description`, `status`, `blocks`, `blockedBy`
  - After fetching, verify `blockedBy` is empty before beginning work
- `TaskList`: list all tasks in summary form — `id`, `subject`, `status`, `owner`, `blockedBy`
  - "Prefer working on tasks in ID order (lowest ID first) when multiple tasks are available — earlier tasks often set up context for later ones"
- `TaskUpdate`:
  - Mark resolved when fully accomplished; if errors/blockers/partial implementation, keep as `in_progress`; create a new task describing what needs to be resolved when blocked
  - Status workflow: `pending` → `in_progress` → `completed`; use `deleted` to permanently remove
  - Never mark `completed` if tests are failing, implementation partial, unresolved errors, missing files/dependencies
  - Read latest state via `TaskGet` before updating
  - Fields updatable: `status`, `subject`, `description`, `activeForm`, `owner`, `metadata`, `addBlocks`, `addBlockedBy`

## Background Tasks (TaskOutput / TaskStop)
- `TaskOutput` is marked DEPRECATED in its description: background tasks return their output file path in the tool result; you receive a `<task-notification>` with the same path on completion
  - For bash tasks: prefer `Read` on the output file path (contains stdout/stderr)
  - For local_agent tasks: use the Agent tool result directly; do NOT Read the `.output` file — it is a symlink to the full sub-agent conversation transcript (JSONL) and will overflow context
  - For remote_agent tasks: prefer `Read` on the output file path
  - `block=true` (default) waits for completion; `block=false` non-blocking; `timeout` max 600000 ms
- `TaskStop`: stops a running background task by `task_id`

## AskUserQuestion
- 1-4 questions per invocation, each with 2-4 options
- Each question: `question` text, `header` (max 12 chars chip/tag label), `options` (label + description), `multiSelect` (boolean)
- Auto "Other" option always appended for custom text input
- "If you recommend a specific option, make that the first option in the list and add '(Recommended)' at the end of the label"
- `preview` (optional, on options): for ASCII mockups, code snippets, diagram variations, config examples; rendered as markdown in monospace box; triggers side-by-side layout (vertical option list left, preview right); single-select only — do not use for simple preference questions where labels and descriptions suffice
- `annotations`: optional per-question user annotations (notes on selections, including selected preview)
- `metadata`: optional tracking/analytics data (e.g., `source: "remember"`)
- Plan mode: use to clarify requirements / choose between approaches BEFORE finalizing the plan; do NOT ask "Is my plan ready?" — use `ExitPlanMode`; do not reference "the plan" in question text since user cannot see it until `ExitPlanMode`

## NotebookEdit
- Completely replaces the contents of a specific cell in a Jupyter notebook (.ipynb file) with new source
- `notebook_path` must be absolute (not relative)
- `cell_number` is 0-indexed (legacy reference); cells are addressed via `cell_id` in the schema
- `edit_mode`: `replace` (default), `insert` (new cell inserted AFTER the cell with `cell_id`, or at the beginning if not specified), `delete`
- `cell_type`: `code` or `markdown`; required for `insert`

## WebFetch
- Fetches content from a URL and processes it with an AI model; read-only
- **WILL FAIL for authenticated or private URLs** — check for specialized MCP tools providing authenticated access first; if an MCP-provided web fetch tool is available, prefer it
- Takes URL + prompt; converts HTML to markdown; processes with a small, fast model
- HTTP auto-upgraded to HTTPS
- 15-minute self-cleaning cache for repeated URL hits
- Redirect handling: when URL redirects to different host, tool informs and provides redirect URL for a new request
- For GitHub URLs, prefer `gh` CLI via Bash

## WebSearch
- Searches the web; returns results as markdown hyperlinks
- **CRITICAL — MUST include "Sources:" section** at end of response with all relevant URLs as markdown hyperlinks: `[Title](URL)`; MANDATORY, never skip
- Domain filtering: `allowed_domains` (include only) and `blocked_domains` (exclude)
- Only available in the US
- Must use the correct year in queries — current month is May 2026

## Cron Tools (CronCreate / CronDelete / CronList)
- `CronCreate`: standard 5-field cron in user's local timezone (`minute hour day-of-month month day-of-week`); no timezone conversion needed (`"0 9 * * *"` means 9am local)
  - **One-shot tasks** (`recurring: false`): for "remind me at X" / "at <time>, do Y" requests; pin minute/hour/day-of-month/month to specific values; fires once then auto-deletes
  - **Recurring jobs** (`recurring: true`, default): `"*/5 * * * *"` (every 5 min), `"0 * * * *"` (hourly), `"0 9 * * 1-5"` (weekdays at 9am local)
  - **Avoid :00 and :30 minute marks** when the user request is approximate — every "9am" gets `0 9` and every "hourly" gets `0 *`, hitting the API in unison globally; pick an off-minute (`"57 8 * * *"`, `"3 9 * * *"`, `"7 * * * *"`); only use :00 or :30 when user names that exact time and clearly means it
  - Session-only by default; `durable: true` persists to `.claude/scheduled_tasks.json` and survives restarts (use only when user asks)
  - Runtime: jobs only fire while REPL is idle (not mid-query); deterministic jitter — recurring tasks fire up to 10% of period late (max 15 min); one-shot tasks landing on :00 or :30 fire up to 90 s early
  - Recurring tasks auto-expire after 7 days — fire one final time, then deleted; tell the user about the 7-day limit when scheduling recurring jobs
  - Returns a job ID for `CronDelete`
- `CronDelete`: cancel a job by ID returned from `CronCreate`; removes from in-memory session store
- `CronList`: list all jobs scheduled in the current session

## Monitor
- Background monitor that streams events from a long-running script; each stdout line is an event; you keep working and notifications arrive in chat; events arrive on their own schedule and are not user replies
- Pick by how many notifications you need:
  - **One** ("tell me when X is ready / build finishes") → use `Bash` with `run_in_background` and a command that exits when condition is true (e.g., `until grep -q "Ready in" dev.log; do sleep 0.5; done`)
  - **One per occurrence, indefinitely** ("every ERROR line") → `Monitor` with unbounded command (`tail -f`, `inotifywait -m`, `while true`)
  - **One per occurrence, until known end** ("each CI step result, stop when run completes") → `Monitor` with command that emits lines and then exits
- "Don't use an unbounded command for a single notification" — never exits, monitor stays armed until timeout; `tail -f log | grep -m 1 ...` does NOT fix this (pipe hangs after match if log goes quiet)
- Script quality:
  - Always use `grep --line-buffered` in pipes — without it, pipe buffering delays events by minutes
  - In poll loops handle transient failures (`curl ... || true`)
  - Poll intervals: 30s+ for remote APIs (rate limits), 0.5-1s for local checks
  - Write a specific `description` — appears in every notification
  - Only stdout is the event stream; stderr goes to the output file (readable via `Read`) but does not trigger notifications — for direct commands merge with `2>&1`
- **Coverage — silence is not success**: filter must match every terminal state, not just the happy path; a monitor that greps only the success marker stays silent through crashloops, hangs, unexpected exits — silence looks identical to "still running"; broaden grep alternation (e.g., `"elapsed_steps=|Traceback|Error|FAILED|assert|Killed|OOM"`); for poll loops emit on every terminal status (`succeeded|failed|cancelled|timeout`)
- Output volume: every stdout line is a conversation message — filter must be selective ("the lines you'd act on, not only good news"); never pipe raw logs; monitors producing too many events are automatically stopped — restart with tighter filter
- Stdout lines within 200 ms are batched into a single notification
- Same shell environment as `Bash`; exit ends the watch (exit code reported); timeout → killed
- `persistent: true` for session-length watches (PR monitoring, log tails) — runs until `TaskStop` or session ends
- `timeout_ms`: default 300000, max 3600000; ignored when persistent

## PushNotification
- Sends a desktop notification in the user's terminal; if Remote Control is connected, also pushes to phone
- Pulls the user's attention from whatever they're doing — cost is real
- "Err toward not sending one" — don't notify for routine progress, when the user is clearly still watching, or when a quick task completes
- Notify when there's a real chance the user has walked away AND something is worth coming back for, OR when the user explicitly asked
- Keep `message` under 200 characters, one line, no markdown — mobile OSes truncate
- Lead with what they'd act on ("build failed: 2 auth tests" beats "task done")
- If the result says the push wasn't sent, that's expected — no action needed
- `status` field is constant `"proactive"`

## RemoteTrigger
- Calls the claude.ai remote-trigger API; OAuth token added automatically in-process and never exposed
- Use this instead of `curl`
- Actions:
  - `list`: GET `/v1/code/triggers`
  - `get`: GET `/v1/code/triggers/{trigger_id}`
  - `create`: POST `/v1/code/triggers` (requires body)
  - `update`: POST `/v1/code/triggers/{trigger_id}` (requires body, partial update)
  - `run`: POST `/v1/code/triggers/{trigger_id}/run` (optional body)
- Response is the raw JSON from the API
