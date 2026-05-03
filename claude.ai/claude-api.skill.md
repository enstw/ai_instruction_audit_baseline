# Skill: claude-api

Build, debug, and optimize Claude API / Anthropic SDK apps. Apps built with this skill should include prompt caching. Also handles migrating existing Claude API code between Claude model versions (4.5 → 4.6, 4.6 → 4.7, retired-model replacements).

## When to invoke (TRIGGER)
- Code imports `anthropic` or `@anthropic-ai/sdk`
- User asks for the Claude API, Anthropic SDK, or Managed Agents
- User adds/modifies/tunes a Claude feature (caching, thinking, compaction, tool use, batch, files, citations, memory) or model (Opus/Sonnet/Haiku) in a file
- Questions about prompt caching / cache hit rate in an Anthropic SDK project

## Do NOT invoke (SKIP)
- File imports `openai` or another provider's SDK
- Filename like `*-openai.py` / `*-generic.py`
- Provider-neutral code
- General programming or ML tasks
