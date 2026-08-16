Research note, requested by Bruno during the "Text editor tool" session: why doesn't the Claude API ship an SDK with a ready-made text editor tool implementation per OS (Windows/macOS/Linux)?

## Findings

- The text editor tool (`text_editor_20250728`, formerly `text_editor_20250124`) is "Anthropic-defined" only in the sense that its **JSON schema** is built into the model — you don't write or supply an `input_schema` for it, and Claude already knows the shape of its commands (`view`, `str_replace`, `create`, `insert`, `undo_edit`).
- It is still a **client tool**: the actual filesystem operations (open a file, read a range, apply a string replacement, create a file, undo the last edit) execute in *your* application, on *your* infrastructure, against *your* filesystem — same execution model as a fully custom tool. Anthropic has no visibility into or access to your machine, so there's nothing for Anthropic to ship that would "just work" universally.
- Contrast this with true **server tools** (`web_search`, `web_fetch`, `code_execution`, `tool_search`) — those run on Anthropic's own infrastructure, so Anthropic can and does provide the full implementation, not just the schema.
- File I/O is also inherently OS/environment-specific (permissions, path semantics, sandboxing expectations, which directories should even be writable) in a way a generic SDK couldn't safely default — a shipped "one size fits all" file editor would either be too permissive (security risk) or too restrictive to be useful, so leaving the implementation to the developer is a deliberate scoping choice, not an oversight.
- In practice, most people don't hand-roll this either — the Claude Agent SDK (used by Claude Code) already ships a working implementation and handles the tool loop for you; the course's from-scratch version is for understanding the underlying mechanics, not the recommended path for production use.

Sources:
- [Text editor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool)
- [Server tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/server-tools)
- [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)
