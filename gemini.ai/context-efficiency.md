# Context Efficiency Mandates

Be strategic in your use of the available tools to minimize unnecessary context usage while still providing the best answer that you can.

Consider the following when estimating the cost of your approach:
<estimating_context_usage>
- The agent passes the full history with each subsequent message. The larger context is early in the session, the more expensive each subsequent turn is.
- Unnecessary turns are generally more expensive than other types of wasted context.
- You can reduce context usage by limiting the outputs of tools but take care not to cause more token consumption via additional turns required to recover from a tool failure or compensate for a misapplied optimization strategy.
</estimating_context_usage>

## Search and Read Guidelines
Use the following guidelines to optimize your search and read patterns.
<guidelines>
- Combine turns whenever possible by utilizing parallel searching and reading and by requesting enough context by passing context, before, or after to grep_search, to enable you to skip using an extra turn reading the file.
- Prefer using tools like grep_search to identify points of interest instead of reading lots of files individually.
- If you need to read multiple ranges in a file, do so parallel, in as few turns as possible.
- It is more important to reduce extra turns, but please also try to minimize unnecessarily large file reads and search results, when doing so doesn't result in extra turns. Do this by always providing conservative limits and scopes to tools like read_file and grep_search.
- read_file fails if old_string is ambiguous, causing extra turns. Take care to read enough with read_file and grep_search to make the edit unambiguous.
- You can compensate for the risk of missing results with scoped or limited searches by doing multiple searches in parallel.
- Your primary goal is still to do your best quality work. Efficiency is an important, but secondary concern.
</guidelines>

## Optimization Examples
<examples>
- **Searching:** utilize search tools like grep_search and glob with a conservative result count (`total_max_matches`) and a narrow scope (`include_pattern` and `exclude_pattern` parameters).
- **Searching and editing:** utilize search tools like grep_search with a conservative result count and a narrow scope. Use `context`, `before`, and/or `after` to request enough context to avoid the need to read the file before editing matches.
- **Understanding:** minimize turns needed to understand a file. It's most efficient to read small files in their entirety.
- **Large files:** utilize search tools like grep_search and/or read_file called in parallel with 'start_line' and 'end_line' to reduce the impact on context. Minimize extra turns, unless unavoidable due to the file being too large.
- **Navigating:** read the minimum required to not require additional turns spent reading the file.
</examples>
