# MCP Server Integration

## Source
https://claudecertificationguide.com/learn/2-tool-design-mcp/2-4-mcp-server-integration

## Summary

### Core Definition
"MCP (Model Context Protocol) servers extend Claude's capabilities by connecting it to external systems — databases, APIs, development tools, issue trackers."

### Configuration Scoping Hierarchy

#### Project-Level Configuration (`.mcp.json`)
Located at the project repository root. This file is **version-controlled and shared** with all team members who clone or pull the repository.

**Purpose:** Servers the entire team needs — Jira integrations, GitHub tools, internal API connectors.

**Example structure:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "jira": {
      "command": "npx",
      "args": ["-y", "@community/mcp-server-jira"],
      "env": {
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

#### User-Level Configuration (`~/.claude.json`)
Located in the user's home directory. This file is **personal, not version-controlled, and not shared** with teammates.

**Purpose:** Experimental servers, personal integrations, or servers being tested before team proposal.

#### Tool Discovery Mechanism
"All tools from all configured servers (both project-level and user-level) are discovered at connection time and available simultaneously. There's no manual activation step — if a server is configured and reachable, its tools appear in the agent's toolkit."

### Environment Variable Expansion
The `.mcp.json` file supports `${VARIABLE_NAME}` syntax for environment variable expansion. This mechanism accomplishes several security and operational objectives:

- Keeps credentials out of version control
- Allows each developer to authenticate with their own credentials locally
- Enables token rotation without config file changes
- Prevents secrets from leaking through repository history

Developers set tokens locally via shell profile, `.env` file, or secrets manager. The configuration file references variable names only, not values.

### MCP Resources vs. Tools
**Resources** expose content catalogues to agents without requiring exploratory tool calls. The agent receives information upfront rather than discovering what data exists through multiple tool invocations.

**Tools** let agents act on data that resources have made visible.

#### Resource Examples
- Issue summaries (current Jira issues with titles and statuses)
- Documentation hierarchies (table of contents for internal docs)
- Database schemas (table names, column types, relationships)

**Key benefit:** "Fewer wasted calls. Without resources, an agent might call `list_tables`, then `describe_table` for every table, burning tool calls just to get its bearings. With a database schema resource, it knows immediately."

### Build vs. Use Decision Framework

#### Use Community Servers For
- Standard integrations (Jira, GitHub, Slack, Linear, Notion)
- Integrations with maintained, community-tested implementations
- Scenarios saving development time and maintenance burden

#### Build Custom Servers Only When
- Team-specific workflows exist that community servers cannot handle
- Custom business logic is required at the tool layer
- Integration with proprietary internal systems lacking community servers

**Exam principle:** "Evaluate community servers first" is always correct for standard integrations. Custom builds are only justified when scenarios explicitly describe team-specific requirements community servers cannot meet.

### Enhanced Tool Descriptions
When MCP tool descriptions are sparse, agents prefer built-in tools like Grep because "the model simply has better context about built-in tools — their descriptions are rich and detailed."

**Weak description example:**
```
search_codebase: "Searches code"
```

**Enhanced description example:**
```
search_codebase: "Performs semantic code search across the entire 
repository using AST-aware indexing. Returns matching functions, 
classes, and methods with full context including file path, line 
numbers, and surrounding code. More accurate than text-based grep 
for finding code by intent rather than exact string match. Use this 
instead of Grep when searching for code by what it does rather than 
what it contains."
```

Enhanced descriptions should be 3-5 sentences, explaining capabilities, outputs, use cases, and comparisons to alternatives.

### Exam Traps

**Trap 1:** Building custom MCP servers for standard integrations like Jira. Community servers exist and should be evaluated first; custom builds only justified for team-specific workflows.

**Trap 2:** Putting team-wide MCP server configuration in `~/.claude.json`. This is user-level, personal, non-version-controlled. Team servers belong in `.mcp.json` at project root.

**Trap 3:** Committing credentials directly in `.mcp.json` instead of using environment variable expansion. Credentials in version control create security risks; use `${VARIABLE}` syntax.

**Trap 4:** Leaving MCP tool descriptions sparse. Models default to better-understood tools. Sparse MCP descriptions lose to detailed built-in descriptions. Enhance MCP descriptions to explain capabilities fully.

### Practice Scenario
**Question:** A team needs Jira integration. A developer proposes building a custom MCP server. What is the correct first step?

**Correct answer:** "Evaluate existing community MCP servers for Jira and only build custom if they cannot handle team-specific workflows."

**Incorrect options:**
- Adding to `~/.claude.json` (wrong scope — not team-wide)
- Using Jira REST API directly via Bash (bypasses MCP structure)
- Building custom to match exact endpoints (premature — evaluate community first)

### Build Exercise Learning Objectives
1. Creating `.mcp.json` at project root with community server configuration
2. Using `${GITHUB_TOKEN}` environment variable expansion
3. Creating a personal MCP server entry in `~/.claude.json`
4. Exposing a content catalogue as an MCP resource with URI, name, description, and mimeType
5. Enhancing tool descriptions to 3-5 sentences explaining capabilities and alternatives
