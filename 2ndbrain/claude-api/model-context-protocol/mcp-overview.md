## Source
- [Introducing MCP](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287780)
- [MCP clients](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287775)
- [Project setup](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287785)
- [Defining tools with MCP](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287797)
- [The server inspector](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287781)
- [Implementing a client](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287793)
- [Defining resources](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287782)
- [Accessing resources](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287783)
- [Defining prompts](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287784)
- [Prompts in the client](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287786)
- [MCP review](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287790)
- [Quiz on Model Context Protocol](https://anthropic.skilljar.com/claude-with-the-anthropic-api/289126)

## Summary

### MCP basics
MCP (Model Context Protocol) is a communication layer designed to give Claude context and tools without requiring the developer to write a lot of tedious integration code. A typical MCP diagram shows two major elements: the **client** and the **server**, where the server exposes internal components named **tools**, **resources**, and **prompts**.

The motivating example: a chat app that lets a user ask Claude about their GitHub data (e.g. "what open pull requests do I have?"). Building this without MCP means authoring and maintaining every tool schema/function for GitHub's large surface area yourself. MCP shifts that burden away from the developer's own server to a separate **MCP server**, which can be thought of as an interface to an outside service (e.g. a GitHub MCP server wraps GitHub functionality as a set of tools).

Three common questions/misconceptions addressed:
- **Who authors MCP servers?** Anyone — official implementations are common too (e.g. AWS releasing their own).
- **How is this different from calling the API directly?** Using an MCP server saves you from writing the tool schema and function yourself; someone else already did.
- **"MCP and tool use are the same thing"** — they're complementary, not identical. Tool use is the mechanism; MCP is about *who* implements and executes the tool.

### The MCP client
The client is the communication bridge between your application and an MCP server. MCP is **transport agnostic** — client and server can talk over stdio (very common for same-machine setups), HTTP, WebSockets, etc. Once connected, they exchange defined message types, including `list_tools` request/result and `call_tool` request/result.

**Full request walkthrough** (user → app server → MCP client → MCP server → GitHub → back): the app server asks the MCP client for the tool list, which the client fetches from the MCP server; the app server sends the user's query + tool list to Claude; Claude responds wanting to use a tool; the app server asks the MCP client to run it; the MCP client sends `call_tool` to the MCP server; the MCP server calls GitHub, wraps the result, and the chain unwinds back through the client to the app server, which sends the tool result back to Claude for a final answer.

![[mcp-diagram.png]]

### Project setup
To reinforce these concepts, the session builds a CLI-based chatbot that manages fake, in-memory documents, implementing **both** an MCP client and server in one project (normally you'd only build one side — this project does both purely for learning). Setup: download/extract the provided CLI project zip, add the API key, and (if using UV) confirm the environment is active.

### Defining tools with MCP
Tools are implemented in `mcp_server.py` using the official **MCP Python SDK**. Defining a tool with the `@mcp.tool` decorator (name + description + typed args) is enough — the SDK auto-generates the JSON schema that would otherwise have to be hand-written. The session implements `read_doc_contents`, which looks up a document by ID in an in-memory `docs` dict and returns its contents.

### The server inspector
The Python SDK ships with an in-browser debugger: run `mcp dev mcp_server.py`, which starts a server (e.g. port 6277) with a direct inspector URL. In the inspector, clicking **Connect** starts the MCP server; a **Tools** tab lists defined tools and lets you manually invoke one with arguments (e.g. a doc ID) to verify it works before wiring it into a real application.

### Implementing a client
The MCP client lives in `mcpclient.py` as a class wrapping the SDK's `ClientSession` (the actual connection to the server), which needs cleanup handled in `connect`/`cleanup`/`__aenter__`/`__aexit__`. The class exposes functions (`list_tools`, `call_tool`, `list_prompts`, `get_prompt`) that the rest of the app calls. `list_tools` awaits `session.list_tools()` and returns `.tools`; `call_tool` awaits `session.call_tool(tool_name, tool_input)`. A small testing harness at the bottom of the file starts the server and prints results (`uv run mcp_client.py` / `python mcpclient.py`) to confirm the tool definitions come back correctly.

### Defining & accessing resources
**Resources** let the MCP server expose data to the client — used here to build an `@document-name` mention feature: typing `@` shows an autocomplete of document names, and submitting a message with a mention fetches that document's contents and inserts it into the prompt sent to Claude (so Claude doesn't need to call a tool to see it).

Two resources are needed: one returning the list of document names, one returning a single document's contents. Resources come in two flavors:
- **Direct/static** — fixed URI, e.g. `docs://documents`.
- **Templated** — URI with a wildcard parameter, e.g. `docs://document/{doc_id}`, where the SDK parses the parameter and passes it as a keyword argument matching the name in the URI template.

Client-side, a `read_resource` function is added: it awaits `session.read_resource(AnyUrl(uri))` (imports `json` and `AnyUrl` from pydantic) and parses the result based on MIME type.

### Defining prompts & prompts in the client
**Prompts** implement predefined, high-quality workflows — here, a `/format` slash command that reformats a document into Markdown. The key point: a user could already ask Claude to "reformat this as markdown" with no code changes needed and get a decent result — but a prompt authored, tested, and evaluated ahead of time by the MCP server author can deliver a *better*, more consistent result than ad-hoc user phrasing.

Server-side: the `@mcp.prompt` decorator defines a prompt with a name/description; the function (e.g. `format_document(doc_id, ...)`) returns a sequence of user/assistant messages. Client-side: `list_prompts` (`await session.list_prompts()` → `.prompts`) and `get_prompt` (`await session.get_prompt(prompt_name, args)` → `.messages`) let the app retrieve and interpolate variables (like a doc ID) into these predefined workflows.

### MCP review — the three primitives by control
- **Tools** are **model-controlled** — Claude alone decides when to run one.
- **Resources** are **app-controlled** — your application code decides when to fetch/use one (e.g. populating a UI list, or augmenting a prompt).
- **Prompts** are **user-controlled** — the user decides when to trigger one (e.g. clicking a UI button, or a slash command).

Examples from claude.ai itself: suggested-prompt buttons under the chat input = prompts; "Add from Google Drive" file picker = a resource; asking Claude to run JS to compute something = a tool, model-controlled.

## My Insights
- On when to build your own MCP server vs. just calling a service's API directly: if you only need the integration for yourself, calling the API directly saves time. Building an MCP server only makes sense if you want to let *other* people/AI systems reuse the same integration — effectively turning it into a shared "tool server" other models can call, instead of locking the tool into one app.
- On the request-flow diagram: whether "tool use" actually calls out to the MCP server depends on whether the requested tool is implemented in your own app server or lives behind the MCP server — the lesson's walkthrough assumes all tools live behind the MCP server, but that's not universally true.

## Ideas
- The "prompts deliver above-average, pre-tested results a user can't get from ad-hoc prompting" pattern could apply to the AI-designed-cards project — expose specific, pre-tuned design prompts as MCP prompts to deliver more consistent AI-powered card design functionality.

## Challenges
(none flagged this session)

## Actions
- [x] Save the last diagram (the full client/server/GitHub/Claude request-flow picture) from the MCP clients lesson page (owner: bruno)
