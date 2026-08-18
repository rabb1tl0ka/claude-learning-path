Ad-hoc research, sparked by [agents-and-workflows.md](../agents-and-workflows.md): "how do we create tools that Claude Code can use?"

## Primary mechanism: MCP servers

Claude Code extends its tool capabilities via **Model Context Protocol (MCP) servers** — the standard way to add custom tools. Servers run either **locally** (stdio, a subprocess on your machine) or **remotely** (HTTP/SSE/WebSocket, a hosted service).

Local server example (Playwright browser automation):
```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

Remote server example (hosted service, like Sentry or Linear):
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

## Configuration scopes

Servers register at three scopes:
- **Local** (project-only, default): `~/.claude.json`, tied to a single directory
- **Project** (team-shared): `.mcp.json` in repo root, checked into git
- **User** (all projects): `~/.claude.json`, active everywhere

`.mcp.json` can also be edited directly — both HTTP and stdio entries are supported.

**Naming gotcha**: "local" scope is a visibility distinction, not a file-location one — it's *not* stored in a project-local file. Both local and user scope live in the same global `~/.claude.json`; local-scoped servers are just keyed internally to the current project's path, so they don't show up in other projects. "Project" scope is the one that actually gets its own project-local file (`.mcp.json`). This differs from general Claude Code settings, where "local" *does* mean a project-local file (`.claude/settings.local.json`) — the terminology isn't consistent between the two systems.

**Example**: say you run these two commands from inside `~/code/project-a`:

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest        # default scope: local
claude mcp add --scope user notion -- npx -y @notionhq/mcp-server  # explicit scope: user
```

`~/.claude.json` ends up holding both, but shaped so only `project-a` sees `playwright`:

```json
{
  "mcpServers": {
    "notion": { "command": "npx", "args": ["-y", "@notionhq/mcp-server"] }
  },
  "projects": {
    "/home/you/code/project-a": {
      "mcpServers": {
        "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp@latest"] }
      }
    }
  }
}
```

Open a Claude Code session in `~/code/project-b` and `/mcp` lists only `notion` — `playwright` is invisible there because it's nested under `project-a`'s key, not because it lives in a different file. Compare that to `--scope project`, which would instead write a `.mcp.json` file physically inside `project-a/`, visible to anyone who clones that repo:

```json
// project-a/.mcp.json
{
  "mcpServers": {
    "playwright": { "command": "npx", "args": ["-y", "@playwright/mcp@latest"] }
  }
}
```

## Other tool-like mechanisms (complementary to MCP, not replacements)

- **Custom slash commands** (`/foo`): single-shot prompts, no tool loop
- **Hooks** (pre/post command execution): automate behaviors triggered by shell commands, not new tools
- **Subagents**: spawn child agents for specialized tasks
- **Permission-gated built-in tools**: Bash, Write, Edit, Read, WebSearch, WebFetch are already part of Claude Code's core set — no build needed

MCP is the primary way to add new **callable tools** Claude can invoke autonomously; the others solve different problems.

## End-to-end: adding one new tool

1. Write or find an MCP server (see [modelcontextprotocol.io](https://modelcontextprotocol.io)) implementing `Tool`, `Resource`, or `Prompt` interfaces, or use an existing one from the Anthropic directory.
2. Register it: `claude mcp add [--scope scope] [--transport type] name url-or-command`.
3. Verify: `claude mcp list` (should show `✔ Connected`).
4. Use it: start a session and Claude picks it up automatically, or name it explicitly in a prompt to force its use.
5. Check available tools: `/mcp` inside a session to see all registered servers and their tools.

stdio servers add startup overhead; remote servers are faster if network latency is acceptable — prefer a hosted server when one exists.
