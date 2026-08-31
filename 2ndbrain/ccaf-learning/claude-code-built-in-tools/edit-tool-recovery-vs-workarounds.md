## Summary

The Edit tool does exact string replacement — no regex, no fuzzy matching. Three checks gate every edit:

1. **Read-before-edit**: the file must have been read in the current conversation (newer models — Opus 4.6+, Haiku 4.5+ — can skip this when reading wouldn't need a permission prompt; older models always require it). Viewing via Bash (`cat`, `nl`, `bat`, `head`, `tail`, `sed -n 'X,Yp'`, `grep`/`rg` on a single file with no pipes) also satisfies this.
2. **Match**: `old_string` must appear character-for-character, including whitespace — one space off and it fails.
3. **Uniqueness**: `old_string` must appear exactly once, unless `replace_all: true`.

**When `old_string` isn't unique, the tool errors — and the fix stays inside the Edit tool, it doesn't mean switching tools:**
- **Expand `old_string`** to include enough surrounding context (a few extra lines) to make it unique — this is the default recovery for a single targeted change.
- **Use `replace_all: true`** only when every occurrence of the pattern should genuinely change the same way (e.g. renaming a variable, flipping a config flag file-wide).

**Read+Write is a separate tool choice, not an Edit-tool recovery path.** It's for when the whole file needs rewriting — reformatting, complex multi-section restructuring, or so many scattered edits that doing them one-by-one is fragile. It is **not** the answer to "Edit can't find a unique match" — that's what expanding context or Grep-first is for.

File-change detection (v2.1.208+): if the file changed on disk since Claude last read it, Edit can still proceed if `old_string` still matches exactly and uniquely; otherwise it re-reads before editing. Before v2.1.208, any edit to an unread-since-change file was refused outright.

## My Insights

Two distinct mistakes, both drifting *away* from the Edit tool instead of using its own recovery path — and they pull in opposite directions:

- **Reaching for `sed`/Bash instead of expanding `old_string` or using Grep first.** This bypasses the Edit tool's uniqueness/match safety guarantees for the sake of a "just find and replace it" shell command that's actually more fragile (line numbers shift, special characters need escaping, no read-before-edit safety net).
- **Reaching for `replace_all` (or Read+Write) when the actual job is a single precise insertion into a file with repetitive patterns.** `replace_all` is a blunt instrument for pattern-wide changes — forcing it to "match a common pattern" for a one-off insertion risks touching every other occurrence of that pattern in the file. The safe move there is either expanding `old_string`'s context until it's unique, or (only when the change is genuinely file-wide/complex) Read the whole file, insert at the right spot, Write it back.

The unifying rule: **a documented recovery step inside the native tool always beats a generic workaround outside it** — whether the workaround is a shell command or a same-tool flag being misapplied to a job it's not built for.

## Ideas

- None yet — this is a mechanics reference, not something to build on.

## Challenges

- Distinguishing "this pattern should change everywhere" (→ `replace_all`) from "this pattern merely looks similar in several places but I only want one of them" (→ expand context, or Grep first to identify the right occurrence) in the moment, under exam time pressure.

## Actions

- [ ] None — reference note, revisit if this pattern recurs in a future mock exam (owner: bruno)
