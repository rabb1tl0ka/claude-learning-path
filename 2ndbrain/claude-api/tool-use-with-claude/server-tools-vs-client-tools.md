Research note, requested by Bruno during the "Web search tool" session: why does web search come with an automatic implementation while the text editor tool requires you to write one?

## Findings

The Claude API splits tools into two categories by **where the code actually runs**, not by how the schema is provided:

- **Server tools** (`web_search`, `web_fetch`, `code_execution`, `tool_search`) execute entirely on Anthropic's own infrastructure. You just enable the tool in your request (schema + config, e.g. `type: web_search_20250305`, `name: web_search`, `max_uses: 5`) and Anthropic runs the actual search/fetch/execution and hands you the result — there's no `tool_result` round-trip for you to fill in, because you never implement anything.
- **Client tools** — both fully custom ones and Anthropic-defined-schema ones like `bash` and `text_editor` — execute in *your* environment. Anthropic gives you the schema (and for `bash`/`text_editor`, a fixed one you can't modify), but the code that actually runs a shell command or edits a file has to live in your codebase, because it needs access to *your* machine/filesystem/sandbox.

Why web search specifically is server-side: searching the web doesn't require touching anything private to you — Anthropic can run the query and return results without ever needing access to your infrastructure. Editing files or running bash, by contrast, is meaningless without access to *your* filesystem/environment, which Anthropic doesn't have and shouldn't have by default. That's the actual dividing line: **tools that only need external, non-private data can be server-side; tools that need to touch your environment must be client-side.**

A single agent can freely mix both kinds in one conversation (e.g. `web_search` for external info + a custom database-query client tool for internal data).

Sources:
- [Server tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/server-tools)
- [Web search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)
- [Tool combinations](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-combinations)
