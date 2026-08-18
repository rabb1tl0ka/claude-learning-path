# CCAF exam docs — summary

Source PDFs (downloaded 2026-08-18 from the official registration page,
`anthropic-partners.skilljar.com/claude-certified-architect-foundations-certification/486716`):
- `ccaf-exam-guide.pdf` — Claude Certified Architect – Foundations Exam Guide, v1.0, effective July 2026
- `certification-terms-and-conditions.pdf` — Anthropic's Certification Terms and Conditions
- `anthropic-certification-exam-policy.pdf` — Anthropic Certification Exam Policy, last updated June 25, 2026

## 0. Why test-center, not online (OnVUE technology requirements)

Bruno's daily driver is Linux, and Pearson's OnVUE online-proctoring requirements
(screenshotted from the scheduler, 2026-08-18) don't support it:

- **Minimum requirements**: Windows 10 or macOS 14+ only — no Linux in the supported OS list.
  Working webcam/mic/speaker (no headphones/headsets), single display only, ≥6 Mbps
  down / ≥2 Mbps up, and the ability to close everything except OnVUE.
- **Prohibited technology explicitly includes virtual machines** — so running Windows in a VM
  on the Linux box isn't a workaround either. Also prohibited: beta OSes, phones/tablets/
  headphones/earbuds/styluses/watches, smart/recording/AI devices (smart speakers, smart
  glasses), secondary/touchscreen displays, VPNs, and corporate/public/shared networks.
- Before exam day: run the System Test on the exact device/network you'll use, restart to free
  resources, and make sure no one else on the network is streaming/downloading heavily.

This is why Bruno is sitting the exam at a physical Pearson VUE test center instead — see the
test-center admission policy below, which has its own (different) rules.

## 1. Exam Guide — the authoritative scope document

This is the single most important document for exam prep — it's far more specific and current
than anything found in Slack or third-party mock exams so far.

### Exam details at a glance
| | |
|---|---|
| Credential | Claude Certified Architect – Foundations |
| Exam code | CCAR-F |
| Items | 60, multiple-choice and multiple-response (each item states how many responses to select) |
| Structure | 4 scenarios drawn at random from a bank of 6 |
| Time limit | 120 minutes |
| Delivery | Proctored — online proctored and/or Pearson VUE test center |
| Passing score | Scaled score of 720 on a 100–1,000 scale (criterion-referenced — measured against a fixed standard, not against other candidates) |
| Fee | $125 USD |
| Validity | 12 months from the date the credential is awarded |
| Result reporting | Pass/fail with scaled score, plus percent-correct per content domain (domain percentages are informational only, not what determines pass/fail) |

### Intended audience
A solution architect with hands-on experience (typically 6+ months) building with the Claude
APIs, **Agent SDK**, Claude Code, and MCP — not just someone who completed the learning-path
courses. Explicitly expects experience with: multi-agent orchestration/subagent delegation/
lifecycle hooks, CLAUDE.md + Agent Skills + MCP server configuration, MCP tool/resource
interface design, and prompt engineering for structured output.

### Content domains (blueprint weights)
| # | Domain | Weight |
|---|---|---|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |

**This is a major finding**: nearly half the exam (Domains 1+2, 45%) is about the **Claude Agent
SDK and MCP tool design** — coordinator/subagent orchestration, hooks, `Task` tool spawning,
`AgentDefinition`, tool description quality, MCP error-response design — none of which the 4
official learning-path courses (`ccaf-learning/`) cover in this depth. The courses' MCP section is
introductory (defining/accessing resources and prompts); this exam tests production
orchestration patterns (hub-and-spoke, hook-based enforcement, error propagation) that read
much closer to real Agent SDK application-building than anything in the coursework.

### The 6 scenarios (4 are randomly picked per exam)
1. **Customer Support Resolution Agent** — Agent SDK support agent with custom MCP tools
   (`get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`); 80%+ first-contact
   resolution target. *Domains: 1, 2, 5.*
2. **Code Generation with Claude Code** — Claude Code for codegen/refactoring/debugging;
   slash commands, CLAUDE.md, plan mode vs direct execution. *Domains: 3, 5.*
3. **Multi-Agent Research System** — coordinator delegating to search/analysis/synthesis
   subagents, producing cited reports. *Domains: 1, 2, 5.*
4. **Developer Productivity with Claude** — Agent SDK dev-productivity agent using built-in
   tools (Read/Write/Bash/Grep/Glob) + MCP servers. *Domains: 2, 3, 1.*
5. **Claude Code for Continuous Integration** — Claude Code in CI/CD for automated review,
   test generation, PR feedback; minimizing false positives. *Domains: 3, 4.*
6. **Structured Data Extraction** — extracting from unstructured docs into validated JSON,
   handling edge cases, downstream integration. *Domains: 4, 5.*

### Detailed objectives (task statements) — condensed by domain
Full task-statement text (knowledge/skills bullets) is in the PDF; headline topics per domain:

- **Domain 1 (Agentic Architecture & Orchestration):** agentic loop control flow via `stop_reason`
  (`tool_use` vs `end_turn`); hub-and-spoke coordinator/subagent orchestration (subagents don't
  share memory, coordinator routes everything); `Task` tool + `AgentDefinition` for spawning
  subagents; explicit context-passing (subagents don't auto-inherit parent context);
  hooks (`PostToolUse`) for tool-result normalization and for enforcing business rules
  deterministically vs prompt-based (probabilistic) compliance; task decomposition strategy
  (prompt chaining vs dynamic decomposition); session management (`--resume`, `fork_session`).
- **Domain 2 (Tool Design & MCP Integration):** writing tool descriptions that disambiguate
  similar tools (this is *the* primary mechanism for reliable tool selection); structured MCP
  error responses (`isError`, `errorCategory` transient/validation/permission, `isRetryable`);
  scoping tools per-agent role (fewer tools = more reliable selection); `tool_choice`
  (`auto`/`any`/forced); MCP server scoping (project `.mcp.json` vs user `~/.claude.json`) with
  env-var credential expansion; MCP resources for exposing content catalogs; built-in tool
  selection (Grep vs Glob vs Read/Write vs Edit).
- **Domain 3 (Claude Code Configuration & Workflows):** CLAUDE.md hierarchy (user/project/
  directory) and `@import`; `.claude/rules/` with YAML frontmatter glob-based path scoping
  (superior to directory-level CLAUDE.md for conventions spread across a codebase); custom
  slash commands (`.claude/commands/`, project- vs user-scoped) and Skills (`SKILL.md`
  frontmatter: `context: fork`, `allowed-tools`, `argument-hint`); plan mode vs direct execution
  (complexity-based selection); iterative refinement (few-shot I/O examples, test-driven
  iteration, the "interview pattern"); CI/CD integration (`-p`/`--print` flag, `--output-format json`,
  `--json-schema`, avoiding duplicate findings across re-runs).
- **Domain 4 (Prompt Engineering & Structured Output):** explicit review criteria over vague
  instructions to cut false positives; few-shot prompting for consistency/ambiguous cases;
  `tool_use` + JSON schema for guaranteed-valid structured output (still doesn't prevent
  *semantic* errors); validation-retry loops with specific error feedback (retries don't help when
  data is simply absent from the source); Message Batches API (50% cost savings, ≤24h window,
  no multi-turn tool calling, `custom_id` correlation — only for non-blocking workloads);
  multi-instance/multi-pass review (independent review beats self-review; split large reviews
  into per-file + cross-file passes).
- **Domain 5 (Context Management & Reliability):** avoiding lossy progressive summarization of
  hard facts; the "lost in the middle" effect; trimming verbose tool output to relevant fields;
  escalation criteria (explicit triggers, not sentiment/self-reported confidence); structured error
  propagation (failure type + partial results + alternatives, not generic "search unavailable"
  statuses or silent success); scratchpad files and state-export manifests for crash recovery in
  long-running exploration; human review routing via stratified sampling and calibrated
  field-level confidence (aggregate accuracy metrics can mask per-segment failure); preserving
  claim-source provenance and annotating (not silently resolving) conflicting source data.

### Sample questions
The guide includes 12 fully worked sample questions (with correct answer + explanation) spanning
all 6 scenarios — genuinely useful as calibration for question style: every wrong option represents
a specific, named failure of reasoning (relying on probabilistic prompt compliance where
deterministic enforcement is needed, over-engineering, treating symptoms instead of root cause,
etc.), matching what `/exam-analysis` has already been finding in mock-exam misses.

### In-scope vs out-of-scope (Appendix)
**Explicitly in scope:** agentic loop implementation, multi-agent orchestration, subagent context
management, tool interface design, MCP tool/resource/server config, error handling &
propagation, escalation decision-making, CLAUDE.md config, custom commands/skills, plan mode,
iterative refinement, structured output via `tool_use`, few-shot prompting, batch processing,
context window optimization, human review workflows, information provenance.

**Explicitly out of scope:** fine-tuning/training custom models, Claude API auth/billing/account
management, language/framework implementation details, deploying/hosting MCP servers
(infra/networking/container orchestration), Claude's internal architecture/training/model
weights, Constitutional AI/RLHF/safety training methodology, embeddings/vector DB
implementation, computer use, vision/image analysis, streaming API implementation, rate
limiting/quotas/pricing, OAuth/API key rotation, cloud-provider-specific config (AWS/GCP/Azure),
model benchmarking, prompt-caching implementation details (beyond knowing it exists), token
counting/tokenization.

### Registration & scheduling
Anthropic Partner Academy → download Exam Guide + review Terms/Policy → register & checkout
(fee reflects partner-tier discount) → create Pearson VUE account → schedule online-proctored or
test-center session. Cancel/reschedule up to 24h before, or forfeit the fee — **but see the
Pearson VUE test-center policy below, which overrides this with a 48h window.**

### Pearson VUE test-center admission policy (from the scheduler, 2026-08-18)

Bruno is taking the **test-center** path, not online-proctored — the online option doesn't
support Linux. Pearson VUE's own policy for the test-center differs from the Exam Guide in one
place and adds detail the guide doesn't cover:

- **Reschedule/cancel window is 48 hours**, not the 24 hours stated in the Exam Guide (Section
  11.6) — this is a direct conflict between the two docs; go with 48h since it's the delivery-method-
  specific source.
- Arrive **15 minutes early** — more than 15 minutes late risks refused admission and fee
  forfeiture.
- ID: one **original** (no photocopies), unexpired, government-issued photo ID with name,
  photo, and signature, issued by the country you're testing in. First/last name must exactly
  match your registration. If you lack a qualifying local ID, an international travel passport from
  your country of citizenship works instead.
- Possible **palm vein scan** for identity verification (retained for future test days and fraud
  prevention) and/or a physical scan — consent to this is bundled into agreeing to the testing
  policies.
- **No personal items** in the testing room: bags, books (unless sponsor-authorized), notes,
  phones, pagers, watches, wallets.
- **Zero-tolerance on unauthorized electronic devices** — must be off and in a locker before the
  self-pat-down/scan; found afterward, you're asked to leave and forfeit the fee.

### Exam-day policies
- Government-issued photo ID matching registration name exactly.
- Retake policy: 14 days after 1st fail, 30 after 2nd, 90 after 3rd; max 4 attempts per rolling 12
  months (per exam — doesn't affect other exams).
- No-show / late arrival beyond the permitted window forfeits the fee, must re-register.
- Rules of conduct: must stay in webcam view (online), clear workspace (no notes/phones/second
  monitor/other materials), no communicating with anyone, no capturing/reproducing exam
  content. Must accept a confidentiality/NDA click-through before the exam starts, or the session
  ends with no refund.
- Credential valid 12 months; on-time renewal is a **free, non-proctored** assessment on the
  Partner Academy — no fee. If it lapses, full paid retake required. Anthropic can require a full
  retake instead of the renewal assessment if content changed materially.
- Appeals: 14 calendar days from the decision/result notice, submitted through Pearson VUE
  (first point of contact) which may loop in Anthropic. The standard-setting outcome and the
  content of individual items are **not** appealable.

## 2. Certification Terms and Conditions — key points

- Governs participation in the "Claude Certification Program" (formerly Anthropic Academy).
  Incorporates the Anthropic Usage Policy and the Exam Policy by reference.
- **Certification is not a warranty of ability** and can't be represented as such. It expires per its
  Certification Term (12 months here) and must be actively renewed; can't claim to be certified
  once expired.
- Anthropic/its Providers own all IP in the Program/Platform/Exams/Courses — no rights granted
  beyond access/use.
- Personal data (name, email, course/exam info) may be shared with third-party service providers
  (proctors, instructors) to deliver the Program. If accessed via a Partner (e.g. through Loka),
  Anthropic may disclose pass/fail and certification status **to the Partner**, and the Partner's use
  of that info for HR/performance decisions is governed by the Partner's own privacy policy, not
  Anthropic's — worth knowing since this means Loka could see pass/fail status via that channel.
- Usage limitations: no unauthorized access, no bulk-downloading/archiving Program content
  beyond normal participation, no reverse engineering, no using it to build a competing
  product/service, no misrepresenting affiliation with Anthropic.
- **No refunds** once access is granted, except as required by law or stated in the materials.
  Anthropic can change fees or discontinue materials without notice.
- Termination: Anthropic can suspend/revoke access and certification at will for suspected
  breach, misconduct/fraud, reputational risk, or legal compliance — without notice, no refund
  obligation.
- Liability capped at the greater of (fees paid to Anthropic in the prior 12 months) or $1,000;
  no liability for indirect/consequential damages. Governing law: California; venue: San
  Francisco courts.

## 3. Anthropic Certification Exam Policy — key points

- **Confidentiality**: exam content (tasks/questions/answers/anything exam-related) is
  Anthropic's Confidential Information — can't distribute, copy, display, publish, record,
  download, transmit, or post it. (Note: this doesn't cover the Exam *Guide* itself, which is meant
  to be downloaded/studied — that's official prep material, distinct from live exam content.)
- **Misconduct — explicitly prohibited, non-exhaustive list**: misrepresenting identity or
  location, altering exam/score/record, submitting non-original work, **using AI products or
  services to assist during the exam**, receiving/giving improper assistance or having someone
  else take it, seeking unauthorized access to exam content, possessing unauthorized items
  (electronics, extra monitors, notes) during the exam or breaks, reverse-engineering exam
  content, taking unscheduled breaks without pre-approval.
  - Consequences (Anthropic's sole discretion): suspend/permanently ban from exams, forced
    retake, invalidate/modify results, expel from the Program, refuse/suspend/revoke
    certification — **no obligation to refund fees** in any case.
- **Exam-related actions**: Anthropic/proctors may verify identity, monitor via audiovisual
  recording, and take immediate action on rule violations.
- **Appeal policy**: 14 calendar days from notice of a decision (suspension, forced retake, result
  invalidation, expulsion, cert refusal/suspension/revocation). Third-party platform (Pearson VUE)
  is first point of contact; may loop in Anthropic.
- **Certification expiration**: Anthropic can retire/update/replace courses, exams, and
  certifications at its discretion as the product evolves. A cert earned via a beta/practice exam or
  a beta-product exam **may expire once the final exam or production product ships** — worth
  flagging since this exam guide is v1.0 "effective July 2026" and both cert-terms and exam-policy
  reference dates in mid/late 2026, so there's some chance of a version bump before or shortly
  after Bruno's sitting.
- **Renewals**: must renew before the Certification Term expires; once expired, no renewal path —
  must fully re-earn (retake all required courses/exams).
- **Accommodations**: must be requested/approved (via the third-party platform, i.e. Pearson VUE)
  *before* scheduling the exam — can't request after booking.
- **Access**: Anthropic can exclude specific regions/countries from the Program at its sole
  discretion.

## Implications for exam prep (flagged, not yet acted on)

The domain weights and task statements above are far more specific than anything in
`exam-prep/CLAUDE.md` or the CyberSkill/claudecertificationguide mock exams reference so far,
and they confirm (with much more precision) that this exam leans heavily on **Claude Agent SDK
orchestration mechanics** (`Task` tool, `AgentDefinition`, hooks, `fork_session`) and **Claude Code
configuration internals** (`.claude/rules/` YAML frontmatter, Skill frontmatter fields, CI flags) that
the 4 official courses under `ccaf-learning/` don't cover in this depth. Worth a follow-up pass to
check the existing `ccaf-learning/` notes and `exam-prep/gap-topics`-turned-`ccaf-learning/`
topic dirs against this blueprint, and likely create new topic notes for domain-1/domain-2
material (Agent SDK orchestration specifics) that's currently a blind spot.
