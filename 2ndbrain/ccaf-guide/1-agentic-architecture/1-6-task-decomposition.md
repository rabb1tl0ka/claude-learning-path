# Task Decomposition Strategies

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-6-task-decomposition

## Summary

### Core Definition
"Task decomposition is how you break complex work into pieces an agentic system can actually handle." The exam tests two decomposition patterns and one failure mode (attention dilution).

### Pattern 1: Fixed Sequential Pipelines (Prompt Chaining)
**Mechanism:** Predetermined steps execute in order. Each step's output becomes the next step's input. The sequence never changes based on intermediate findings.

**Code Review Pipeline Example:**
1. Local analysis pass per file (style, bugs, complexity)
2. Cross-file integration pass (data flow, API consistency, import chains)
3. Unified report compilation

**Best For:** Predictable, structured tasks — code reviews, document processing, data extraction, compliance checks.

**Advantages:**
- Consistency: identical input always follows identical path
- Debuggability: know which step produced which output
- Monitorability: log each step's output

**Limitations:** Cannot adapt to unexpected findings. If Step 2 discovers something requiring Step 3 adjustment, the pipeline cannot respond.

### Pattern 2: Dynamic Adaptive Decomposition
**Mechanism:** Agent begins with high-level goal, investigates initially, generates plan from findings, then adapts the plan as execution reveals new information.

**Legacy Codebase Testing Example:**
1. Map structure (directories, modules, dependencies)
2. Identify high-impact areas (most-used modules, buggy modules, untested critical paths)
3. Create prioritized test plan
4. Write tests → discover Module A depends on untested Module B
5. Reprioritize: test Module B first
6. Continue adapting as dependencies emerge

**Best For:** Open-ended investigation tasks with unknown scope — legacy exploration, security audits, research, debugging unfamiliar codebases.

**Advantages:** Adapts to problem complexity, discovers unexpected issues, produces thorough results.

**Limitations:** Unpredictable execution time, harder to estimate completion/resources, harder to debug failures.

### Pattern Selection Decision Matrix

| Task Characteristics | Pattern | Reasoning |
|---|---|---|
| Steps known in advance, structured input | Fixed pipeline | Consistency outweighs adaptability |
| Open-ended, unknown scope | Dynamic decomposition | Adaptability essential when undefined |
| Multi-file code review | Fixed pipeline | Per-file + cross-file predictable |
| Legacy codebase exploration | Dynamic decomposition | Dependencies emerge during investigation |
| Document extraction | Fixed pipeline | Fields/format predetermined |
| Debugging unfamiliar system | Dynamic decomposition | Root cause unknown, investigation adapts |

### Attention Dilution: The Specific Failure Mode
**Definition:** Failure mode occurring when an agent processes too many items in a single pass, producing inconsistent analysis depth across items.

**Telltale Symptoms:**
- Detailed feedback for first few files, increasingly shallow for later files
- Pattern flagged problematic in one file, approved (identical code) in another
- Obvious bugs missed in some files while minor style issues caught in others

**Root Cause:** Model allocates attention across all context items. More items = less attention per item. Early items disproportionately prioritized.

**Critical Exam Trap:** "Suggesting a more powerful model or larger context window as the fix for attention dilution" — this is incorrect. Attention dilution is architectural, not a model capability problem. Processing too many items in a single pass produces inconsistent depth regardless of model power or context size.

### Multi-Pass Architecture Solution
**Two-Layer Structure:**
1. **Per-item local analysis passes:** Each file/document/module analyzed individually in an isolated pass. Full attention budget focused on single item.
2. **Cross-item integration pass:** Separate pass after all local passes complete. Focuses on cross-cutting concerns: data flow issues, inconsistent pattern usage, cross-file dependencies.

**Result:** Per-item passes catch local issues consistently (dedicated attention per item). Integration pass catches cross-item issues (focused on relationships rather than everything at once).

### Practical Example: 14-File Code Review Attention Dilution
**Single-Pass Results:**
- Files 1-5: detailed feedback, specific line references, bug identification, improvement suggestions
- Files 6-9: moderate feedback, some issues, less thorough analysis
- Files 10-14: superficial feedback, **misses obvious null pointer bugs and SQL injection vulnerabilities**
- forEach loop flagged inefficient in File 3, **identical code in File 11 receives no comment**

**Multi-Pass Fix:**
- 14 per-file analysis passes (each focused on one file) + cross-file integration pass
- Result: null pointer bugs in Files 10-14 caught (dedicated per-file pass), forEach inconsistency identified (integration pass checks cross-file pattern consistency)

### Additional Exam Traps
1. **"Proposing single-pass with better prompts as equivalent to multi-pass"** — Better prompts improve average quality but don't solve fundamental attention allocation problem. Multi-pass ensures dedicated attention.
2. **"Batching files into groups without cross-file integration pass"** — Batching reduces dilution within batch but misses cross-batch issues. Data flow issues and pattern inconsistencies across batches go undetected.
3. **"Applying fixed pipelines to open-ended investigation"** — Cannot respond to unexpected findings. Dynamic decomposition required when full scope is unknown.

### Practice Scenario Answer
**Scenario:** 14-file review shows detailed feedback for files 1-5, misses bugs in 10-14, flags forEach inefficient in one file but approves identical code elsewhere.

**Correct Answer:** "Split the review into per-file local analysis passes plus a separate cross-file integration pass to avoid attention dilution" (Option D).

**Why others fail:**
- Reducing to 5-file batches without integration pass still misses cross-batch issues
- Upgrading context window doesn't fix architectural attention allocation problem
- Stronger prompt doesn't solve fundamental single-pass limitation
