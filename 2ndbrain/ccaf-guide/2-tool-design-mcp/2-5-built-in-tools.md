# Built-in Tools

## Source
https://claudecertificationguide.com/learn/2-tool-design-mcp/2-5-built-in-tools

## Summary

### Overview
This lesson covers six built-in tools in Claude Code: Read, Write, Edit, Bash, Grep, and Glob. The material emphasizes that selecting the wrong tool wastes time and context tokens, and the exam deliberately tests this distinction.

### Core Tool Distinctions

#### Grep vs Glob: The Critical Distinction

**Grep's function:** "Grep searches file CONTENTS for patterns." Use Grep when finding text inside files, including function callers, error messages, import statements, and variable assignments. Any search for what files *contain* should use Grep.

Examples provided:
- `Grep: "processLegacyOrder"` — finds all files calling this function
- `Grep: "timeout"` — locates error messages containing "timeout"
- `Grep: "import.*from 'utils/auth'"` — finds files importing a specific module

**Glob's function:** "Glob matches file PATHS by naming patterns." Use Glob when finding files by name, extension, or directory structure. This includes test files, configuration files, and all files of a specific type in a directory.

Examples provided:
- `Glob: "**/*.test.tsx"` — finds all test files
- `Glob: "**/config.*"` — finds all configuration files
- `Glob: "content/domains/**/*.mdx"` — finds MDX files in domains directory

**Concise distinction:** "Grep finds what is INSIDE files. Glob finds files by their NAMES."

The material warns that exams present scenarios where developers use wrong tools — using Glob to find function callers fails because Glob matches paths, not contents. Using Grep to find test files by pattern technically works (by searching for "test" in filenames via content) but represents incorrect tool selection that the exam expects candidates to identify.

#### Read, Write, and Edit

**Edit tool:** Performs targeted modifications using unique text matching. The developer specifies exact text to find and its replacement. It operates with speed and precision by touching only specified text.

Example:
```
Edit:
  old_string: "function processOrder(id: string)"
  new_string: "function processOrder(id: string, validate: boolean = true)"
```

**Edit limitations and recovery:** Edit requires unique text matching. When specified text appears multiple times in a file, Edit cannot determine which occurrence to modify and fails as a safety mechanism. Recovery options (per Edit tool documentation):

1. Widen `old_string` with more surrounding context until it pins down one location
2. Set `replace_all: true` if every occurrence should be updated

Both options remain on Edit and cost minimal context tokens.

**Modification tool ordering:**
1. Try Edit with the shortest anchor that's plausibly unique
2. On non-unique match, widen `old_string` or use `replace_all: true`
3. Fall back to Read + Write only when neither option can disambiguate the target

The material explicitly states: "Don't default to Read + Write for every modification. The exam penalises that because it burns context tokens. It also penalises jumping straight from a non-unique Edit failure to Read + Write."

### Codebase Exploration Strategy

#### Incremental Discovery (Correct Approach)
The material contrasts wrong and right exploration methods:

**Wrong:** "Read all files upfront." Loading every file into context before determining what's needed is identified as the costliest context-budget mistake. A 200-file codebase read completely consumes the entire context window, mostly on irrelevant files.

**Right:** Incremental discovery following this sequence:
1. **Grep to find entry points** — search for the function name, class name, or error message anchoring investigation. This identifies relevant files.
2. **Read to follow imports and trace flows** — once relevant files are known, read them to understand code structure and follow import statements to discover related files.
3. **Grep again to trace usage** — when wrapper functions or re-exports are found, grep for those names across the codebase to find all consumers.
4. **Read only what's needed** — each file read should be justified by discoveries from the previous step.

This approach achieves "minimal context for maximum understanding" by progressively mapping the codebase, spending tokens only on files relevant to the task.

#### Tracing Function Usage Across Wrapper Modules
A common pattern: functions defined in one module are re-exported through wrappers and consumed through the wrapper's name. A simple Grep for the original name misses indirect consumers.

**Correct approach:**
1. Grep for the function definition to locate where it's defined
2. Read the defining file to identify exported names
3. Grep for each exported name across the codebase to find consumers
4. For barrel file re-exports (e.g., `index.ts`), grep for the barrel file's module name to find consumers importing through it

"The multi-step trace catches indirect consumers a single Grep would miss."

#### The Deprecation Scenario
This pattern appears "constantly in exam prep": finding every file calling a deprecated function AND the test files exercising it.

**Correct sequence:**
1. **Grep for the function name** — finds every file referencing the function in contents, including tests that import it directly
2. **Glob for sibling test files** — finds the test file paired with each caller by naming convention (e.g., `OrderProcessor.ts` → `OrderProcessor.test.tsx`), even when the test exercises the function indirectly through the source module
3. **Grep again for wrapper names** — when a caller exposes the function through a wrapper (e.g., `applyLegacyOrder` calls `processLegacyOrder` internally), grep for the wrapper name to find tests covering the function transitively through it

Example given: If Grep reveals `OrderProcessor.ts` and `RefundHandler.ts` call the deprecated function, Glob for `**/OrderProcessor.test.*` and `**/RefundHandler.test.*` to pull in sibling test files even if those tests never mention `processLegacyOrder` by name. If source files wrap the function under a new name, Grep for the wrapper to catch remaining tests.

"This is Grep, then Glob, then Grep again — content search for direct references, path matching for adjacent tests, content search for indirect coverage. Not Glob first."

### Key Concept Summary
"Grep searches file contents. Glob matches file paths. Edit is the default for modifications. On a non-unique match, widen the anchor or use `replace_all: true`. Read + Write is the last-resort fallback. Build codebase understanding incrementally. Never read all files upfront."

### Exam Traps (Explicitly Called Out)

**Trap 1 — Using Glob to find function callers.** Glob searches paths, not contents. Cannot search inside files for function calls. Must use Grep for searching file contents for function names, import statements, or error messages.

**Trap 2 — Using Grep to find files by extension or naming pattern.** While Grep could technically find filenames mentioned in content, Glob is the purpose-built tool for matching file paths. Use Glob for `**/*.test.tsx`, `**/config.*`, and similar path-based searches.

**Trap 3 — Reading all source files upfront.** Loading every file into context is a context-budget killer. Correct approach is incremental: Grep to find entry points, then Read to trace flows from specific entry points only.

**Trap 4 — Defaulting to Read + Write for every modification.** Edit is faster and uses less context because it touches only specific text. Read + Write loads the entire file. Try Edit first; widen the anchor or use `replace_all` when Edit reports a non-unique match. Read + Write is the last-resort fallback, not the standard response.

**Trap 5 — Jumping to Read + Write on non-unique Edit match.** The documented Edit recovery per Edit tool docs is to widen `old_string` with surrounding context until it pins down one occurrence, or set `replace_all: true` for global changes. Both stay on Edit and cost almost nothing. Escalate to Read + Write only when neither option can disambiguate the target.

### Practice Scenario
The scenario presents finding all files calling `processLegacyOrder()` and all test files for those callers. The correct answer (Option C) is:

"Grep for processLegacyOrder to find callers (this also surfaces tests that import the function directly), then Glob for the sibling test file of each caller (e.g. **/OrderProcessor.test.*) to catch tests that exercise the function through the source module without naming it."

### Build Exercise Overview
The exercise involves five steps:
1. **Grep** to search for all callers of the target function across the codebase (content search)
2. **Glob** to find test files matching caller filenames (path matching by naming convention)
3. **Read** to examine each caller file incrementally — only after Grep identifies relevant files
4. **Edit** to replace deprecated function calls with the new API in each caller file
5. **Edit recovery** on non-unique matches: widen `old_string` or use `replace_all: true` before falling back to Read + Write

The material reiterates throughout that reading all source files upfront is "a context-budget killer that the exam explicitly penalises."

### Additional Tools Mentioned
The lesson mentions six built-in tools exist (Read, Write, Edit, Bash, Grep, Glob) but provides detailed guidance only for Grep, Glob, Read, Write, and Edit. Bash is named but not detailed in the provided content.
