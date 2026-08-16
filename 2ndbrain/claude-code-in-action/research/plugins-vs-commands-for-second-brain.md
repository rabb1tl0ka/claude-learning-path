Ad-hoc research, sparked by [sharing-and-scaling-claude-code.md](../sharing-and-scaling-claude-code.md): "would a plugin be a better way to share the Claude Code setup I've built in my AI-project-second-brain repo (a `kernel/` directory holding `release.sh`, and a `/template-upgrade-repos` command that pushes updates out to client repos) than what I'm doing today?"

**Assumption**: this note takes Bruno's own description of his current setup at face value (a `kernel/` directory, a `release.sh` script, and a custom slash command named `/template-upgrade-repos`) — it wasn't independently inspected, since the ask was about the plugin-vs-command tradeoff in general, not an audit of that specific repo.

## What a plugin actually buys you over standalone commands/scripts

A plugin bundles skills, sub-agents, commands, hooks, and MCP server configs into **one versioned, installable unit** with a manifest (`.claude-plugin/plugin.json`: name, version, description, author). Distribution and version tracking is the main thing it adds that a folder of markdown files and a shell script doesn't:

- **Centralized discovery**: add a private marketplace once (`/plugin marketplace add org/claude-plugins`), and every client repo installs from it with `/plugin install org/plugin-name` — no copy-pasting `.claude/` folders between repos.
- **Versioning**: a plugin is versioned like any dependency; a client repo can pin to a version rather than picking up whatever's currently on disk.
- **Namespacing**: skills/agents/commands ship namespaced under the plugin name, so they can't clash with anything else in a client repo's own `.claude/` setup.
- **Update propagation**: bumping the plugin version and re-running the install is the update mechanism, instead of `release.sh` pushing file copies out to every client repo by hand.

## What it costs

- A plugin runs code with the installing user's privileges, and its hooks fire on every matching tool call whether or not the user reads them — so anyone installing it (including Bruno's future self on a new machine, or anyone else on client engagements) needs to trust or inspect it first.
- It's closer to shipping an npm package: manifest, versioning, a marketplace to maintain. If the thing being shared doesn't need any of that overhead, a standalone skill/command is lighter weight and skips the manifest entirely.
- Plugins don't replace a repo's own configuration — they run *alongside* it (hooks stack, components are namespaced) — so this isn't a wholesale replacement for `.claude/` in each client repo, just the distribution layer for the shared pieces.

## Where this points for the second-brain setup specifically

`release.sh` pushing updates via `/template-upgrade-repos` is functionally a hand-rolled version of what a plugin's install/update mechanism does for free (versioning + push-based distribution). The tradeoff is really: is the current script-based push model causing enough pain (version drift across client repos, no visibility into who's on what version, manual copy/paste) to justify standing up a private marketplace and packaging the kernel as a plugin? If client repos rarely need to know *which version* of the shared skills they're on, and the script already reliably pushes updates everywhere, the plugin's main advantage (centralized versioned discovery) may not be worth the added packaging/manifest overhead. If version drift or selectively-opted-in updates across client repos has actually been a problem, that's the concrete signal to package it as a plugin instead.

Sources:
- [Claude Code plugins overview](https://claude.com/blog/claude-code-plugins)
- [Claude Code plugins: from personal setup to org standard](https://claudefa.st/blog/tools/mcp-extensions/plugins-distribution)
- [Claude Code skills vs. plugins](https://www.mindstudio.ai/blog/claude-code-skills-vs-plugins-difference)
