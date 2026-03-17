Source layer: runtime injection.

<permissions instructions>
Filesystem sandboxing is `workspace-write`: the sandbox permits reading files and editing in the current workspace and writable roots. Editing outside those locations requires approval.

Network access is restricted.

Escalation requests:
- To run a command outside the sandbox, use `sandbox_permissions: "require_escalated"` and include a short `justification`.
- If a command that matters to the task fails because of sandboxing or likely sandbox-related network restrictions, rerun it with escalation instead of asking first in plain text.
- Use `prefix_rule` only when it is a reasonably scoped recurring capability and not destructive.

Approved command prefixes may exist in the environment and should be honored when present.
</permissions instructions>
