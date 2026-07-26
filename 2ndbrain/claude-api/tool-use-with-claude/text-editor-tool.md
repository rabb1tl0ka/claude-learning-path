## Source
- [The text edit tool](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287760)

## Summary

Unlike every tool built so far in this project, the **text editor tool** is one Claude has access to by default — it gives Claude the ability to open files/directories, read specific ranges, add/replace text, create new files, and undo edits, essentially letting it act like a developer working in a text editor.

**The key nuance**: only the *JSON schema* half of this tool is actually built into Claude. The actual implementation — the code that runs on your server and performs the real file operations Claude requests — still has to be written by you. Every tool always has two halves: (1) the JSON schema telling Claude the tool exists and what arguments it takes, and (2) the function implementation that executes when Claude uses it. For the text editor tool, Anthropic provides half of that (the schema); you still have to provide the other half. The course's demo notebook (`005 text editor tool`) includes a pre-built class implementing all the required functions for this.

## My Insights

Bruno's core confusion (his words): "why doesn't the Claude API come with an SDK that already comes ready with a text editor implementation per operating system?" He gets *why* the implementation has to be server-side (it's touching your actual filesystem) but wanted to know why Anthropic doesn't just ship the OS-specific implementation code as part of an SDK, the way the schema itself is bundled. See [text-editor-tool-no-sdk.md](text-editor-tool-no-sdk.md) for the research — short version: file I/O is inherently environment/permission-specific in a way that resists a safe universal default, and the Agent SDK (used by Claude Code) already does ship a working implementation for people who don't want to write their own.

## Ideas

(none new this lesson)

## Challenges

- Why the text editor tool ships only half-built (schema, no implementation) when the schema-generation logic feels like it could plausibly extend to a reference implementation too.

## Actions
- [x] Research why the Claude API doesn't ship an SDK with pre-built text editor tool implementations per operating system (Windows/macOS/Linux) (owner: claude) — done same session, see [text-editor-tool-no-sdk.md](text-editor-tool-no-sdk.md)
