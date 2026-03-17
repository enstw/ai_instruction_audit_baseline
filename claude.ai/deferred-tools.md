# Deferred Tool Schemas

Rules from deferred tool definitions (schemas accessed via ToolSearch):

## Plan Mode (EnterPlanMode)
- Use proactively for: new features, multiple valid approaches, multi-file changes, unclear requirements, architectural decisions
- Do not use for: single trivial tasks, pure research/exploration, tasks with very specific detailed instructions
- Within plan mode: explore codebase, design approach, present for user approval, exit with `ExitPlanMode`
- Use `AskUserQuestion` to clarify approach before finalizing; do NOT ask "is this plan okay?" — that is what `ExitPlanMode` does
- `ExitPlanMode` accepts optional `allowedPrompts` parameter: prompt-based permissions needed to implement the plan; each item has `tool` (enum: `["Bash"]`) and `prompt` (semantic description of the action, e.g., "run tests", "install dependencies")

## Worktree (EnterWorktree / ExitWorktree)
- Only use when the user explicitly says the word "worktree"
- Do not use for branch switching or feature work unless "worktree" is mentioned
- `EnterWorktree`:
  - Requires: git repository OR WorktreeCreate/WorktreeRemove hooks configured in settings.json
  - Must not already be in a worktree
  - In a git repo: creates worktree inside `.claude/worktrees/` with new branch based on HEAD
  - Outside a git repo: delegates to WorktreeCreate/WorktreeRemove hooks for VCS-agnostic isolation
  - Optional `name` parameter for naming the worktree
- `ExitWorktree`: only call when user explicitly asks to exit/leave the worktree; never call proactively
  - `action`: `"keep"` (leave worktree on disk) or `"remove"` (delete worktree and branch)
  - `discard_changes`: tool refuses removal if uncommitted changes exist unless explicitly set to `true`
  - Only operates on worktrees created by `EnterWorktree` in the current session; no-op otherwise
  - Tmux handling: if a tmux session was attached to the worktree, killed on `remove`, left running on `keep` (name returned for reattach)
  - Once exited, `EnterWorktree` can be called again to create a fresh worktree

## Task Management (TaskCreate, TaskGet, TaskList, TaskUpdate) and Background Tasks (TaskOutput, TaskStop)
- Individual CRUD task tools replacing the former single-array `TodoWrite`:
  - `TaskCreate`: create tasks with `subject` (imperative title), `description` (detailed context/acceptance criteria), optional `activeForm` (present continuous for spinner), optional `metadata` (arbitrary key-value data)
  - `TaskGet`: retrieve full task details by ID (subject, description, status, blocks, blockedBy)
  - `TaskList`: list all tasks in summary form (id, subject, status, owner, blockedBy); prefer working on tasks in ID order
  - `TaskUpdate`: update `status`, `subject`, `description`, `activeForm`, `owner`, `metadata`; manage dependencies via `addBlocks`/`addBlockedBy`; set status to `deleted` to permanently remove a task
- Status workflow: `pending` → `in_progress` → `completed` (also `deleted`)
- Use for tasks with 3+ distinct steps, non-trivial/complex tasks, plan mode tracking, or when user provides a list
- Do not use for single trivial tasks, informational responses, or tasks completable in <3 trivial steps
- Only mark completed when FULLY accomplished; keep as in_progress if errors, blockers, or partial implementation
- `TaskOutput`: read output from running or completed tasks (background shells, agents, or remote sessions); `block` parameter (true = wait for completion, false = non-blocking check); `timeout` parameter
- `TaskStop`: stop a running background task by ID

## AskUserQuestion
- Prompt user for clarification; supports 1-4 questions per invocation, each with 2-4 options
- Each question requires: `question` text, `header` (short chip/tag label, max 12 chars), `options` (label + description), `multiSelect` (boolean)
- `preview` feature: optional field on options for presenting concrete artifacts (ASCII mockups, code snippets, diagram variations, config examples); rendered as markdown in monospace box; triggers side-by-side layout; single-select only
- `annotations`: optional per-question user annotations (notes on selections)
- `metadata`: optional tracking/analytics data (e.g., `source: "remember"`)
- Auto "Other" option always appended for custom text input

## NotebookEdit
- Edit Jupyter notebook cells; absolute path required
- `cell_id`: ID of cell to edit; for insert mode, new cell inserted after this ID
- `edit_mode`: `replace` (default), `insert` (add new cell), `delete` (remove cell)
- `cell_type`: `code` or `markdown`; required for insert mode

## WebFetch
- Fetch content from URL and process with AI model; read-only
- **WILL FAIL for authenticated or private URLs** — check for specialized MCP tools providing authenticated access first
- If MCP-provided web fetch tool is available, prefer using that instead
- HTTP auto-upgraded to HTTPS; 15-minute self-cleaning cache
- Redirect handling: when URL redirects to different host, tool informs and provides redirect URL for a new request
- For GitHub URLs, prefer `gh` CLI via Bash

## WebSearch
- Search the web; returns results with links as markdown hyperlinks
- **MUST include "Sources:" section** at end of response with all relevant URLs
- Domain filtering: `allowed_domains` (include only) and `blocked_domains` (exclude)
- Only available in the US
- Must use correct year in queries (current: March 2026)
