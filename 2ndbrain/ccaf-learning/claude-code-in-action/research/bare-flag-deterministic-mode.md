Ad-hoc research, sparked by [sharing-and-scaling-claude-code.md](../sharing-and-scaling-claude-code.md): "the course said a `--bear` flag unlocks 'deterministic mode' — what is it even deterministic about, since the model itself isn't guaranteed deterministic on the same prompt?"

## It's `--bare`, not "bear" — a transcription artifact

Gemini's transcript spelled it phonetically ("bear/bare") because it was narrated audio, not read text. The actual flag is `--bare`.

## What "deterministic" actually means here — environment, not model output

`--bare` does **not** make Claude's model output deterministic (nothing does — same prompt, same model, can still yield different completions). What it makes deterministic is the **execution environment**:

- Skips auto-discovery of local **hooks**
- Skips auto-discovery of **MCP servers**
- Skips auto-discovery of **CLAUDE.md**
- Skips auto-discovery of **plugins**

Without `--bare`, a run picks up whatever the local checkout happens to have configured — a hook that's present on one machine and not another, a CLAUDE.md that changed since the last run, a plugin someone installed last week. That's a source of *run-to-run variance in behavior* that has nothing to do with the model — two runs of the identical prompt could behave differently purely because the local environment differs. `--bare` removes that axis of variance: the run gets Claude plus only the tools you explicitly allow, and nothing the local environment happens to load — so a CI run behaves the same regardless of which machine or checkout state it executes from.

It also has a practical side effect: skipping all that discovery makes startup faster.

## Where this fits: headless mode (`-p`/`--print`)

`--bare` is typically paired with headless/non-interactive mode:
- `-p` / `--print` runs Claude Code as a one-shot command — no interactive terminal, reads stdin, writes stdout, so it pipes like any other shell tool.
- Combine `--output-format json` with a JSON schema to constrain structured output; the result lands in a structured-output field you can pull with `jq` and pipe into a script or database.
- For multi-step automation: capture the session ID from the JSON output and resume it later with full context — one script kicks off the work, another resumes it.

So the practical rule of thumb: reach for `--bare` specifically for CI/scheduled jobs where you want the *same* configuration surface every single run, regardless of what's sitting in the checkout or on the runner.

Sources:
- [Claude Code headless mode / non-interactive `-p` mode](https://claudecode101.com/en/tutorial/advanced/headless-mode)
- [Claude Code SDK headless docs](https://docs.anthropic.com/en/docs/claude-code/sdk/sdk-headless)
