# AI Instruction Baseline

Tracks operational baselines for AI coding agents (Claude, Codex, Gemini) and their drift over time.

## What's here

| Directory | Description |
|---|---|
| `claude.ai/` | Operational baseline for Claude (Claude Code CLI) |
| `codex.ai/` | Operational baseline for Codex (OpenAI Codex CLI) |
| `gemini.ai/` | Operational baseline for Gemini (Google Gemini CLI) |

Each directory holds the model's full instruction state: `instruction.md` (core baseline), and optionally `embedded-tools.md`, `deferred-tools.md`, `runtime.md`, and per-skill files.

## How it gets updated

A scheduled GitHub Actions workflow in a private sibling repo wakes daily, restores the target AI's CLI auth from secrets, runs an `audit track` procedure inside this repo, and pushes any baseline drift back here.

Each commit message follows a fixed shape:

```
Audit: Audit Track <date>

- [ADDED <native reference>] ...
- [MODIFIED <native reference>] ...
- [REMOVED <native reference>] ...

Tokens: <prompt> prompt, <completion> completion
```

When no drift is detected, an empty commit lands instead:

```
Audit: No Changes Detected [<MODEL>] <date>
```

## History

```
git log --oneline
git show <hash>
```
