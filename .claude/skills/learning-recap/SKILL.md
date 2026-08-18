---
name: learning-recap
description: Turns a Gemini transcript of a self-scheduled "watch the course video + narrate/comment on it" Google Meet session into a single chapter note (Summary / My Insights / Ideas / Challenges / Actions), following the learning-repo template. Generic across any learning repo built on this template — course/meeting-title matching is read from a per-repo config file, not hardcoded. Triggers on "/learning-recap", "process this chapter's recording", "make my chapter notes from the meeting".
argument-hint: [--url <gemini-doc-url>] [--date <yyyy-MM-dd>] [--preview]
---

# /learning-recap — Gemini self-study transcript → chapter note

Operates relative to the **current working directory**: a learning repo with a root `CLAUDE.md` (course table) and a `2ndbrain/CLAUDE.md` (chapter note structure — `## Summary` / `## My Insights` / `## Ideas` / `## Challenges` / `## Actions`). If either is missing, stop and tell the user this doesn't look like a learning-repo-template directory.

## How the source material works

Bruno schedules a Google Meet with himself, turns on Gemini transcription, and screen-shares the browser tab playing the course video. Gemini's transcript then contains two distinct speaker streams:

- **`Bruno Coelho's Presentation: ...`** — the course video's own audio/captions, captured because he's presenting that tab. This is the raw material for `## Summary`.
- **`Bruno Coelho: ...`** (no "Presentation" suffix) — Bruno actually talking, usually pausing the video to comment. This is the raw material for `## My Insights`, `## Ideas`, `## Challenges`, and `## Actions`.

Never blend the two streams when drafting — narration only informs the Summary, spoken commentary only informs the other four sections.

### Two kinds of commentary: Bruno's own vs. requests directed at Claude

Within the `Bruno Coelho: ...` (commentary) stream, Bruno sometimes talks to himself/reflects, and sometimes directly addresses Claude with a request — e.g. "Claude, go research how to set up Jupyter notebooks and save it as a note for this chapter," or "can you get all the URLs from the course page." This will keep happening — treat it as a normal, recurring part of the commentary stream, not a one-off surprise.

Tell the two apart by grammatical person, not just keyword-matching "Claude":
- **Bruno's own commitment** — first person ("I need to...", "let me...", "I'll..."). → an Action with `(owner: bruno)`.
- **A request directed at Claude** — imperative addressed to Claude, or a question asking Claude to do something ("Claude, can you...", "please research...", "could you get..."). → an Action with `(owner: claude)`.

Both still get logged as `## Actions` items — a Claude-directed request never gets acted on *silently*, and it also never gets *dropped silently*. See **Handling `owner: claude` actions** in step 4 for what to do with these during the same recap run.

## Config file (per-repo, generic skill)

Read `.claude/learning-recap.config.yaml`:

```yaml
MEETING_TITLE_FILTER:
  - "Claude Code Learning Path"
  - "Learning Path"
CALENDAR_ID: "primary"   # optional, defaults to primary
LOOKBACK_DAYS: 14         # optional, defaults to 14 — how far back to search Calendar each run
```

`MEETING_TITLE_FILTER` is a list of one or more strings. An event matches if its title contains **any** of them (case-insensitive substring match, OR logic) — no need for every string to appear.

If the config file doesn't exist, ask once and create it:

> "What string(s) should I match in your Google Calendar event titles to find your self-study sessions for this repo? (comma-separated if more than one, e.g. 'Claude Code Learning Path, Learning Path')"

Split the answer on commas, trim whitespace, write each as a list item. `CALENDAR_ID` defaults to `primary` — only ask about it if the user brings up multiple calendars. `LOOKBACK_DAYS` defaults to `14` — only ask about it if the user brings up wanting a longer/shorter search window; it exists purely to bound the Calendar query, not to dedupe (that's the state file's job).

If an existing config has `MEETING_TITLE_FILTER` as a bare string (old single-value format) rather than a list, treat it as a one-item list — don't error or ask the user to migrate it.

### State file — per-event tracking, not a date cursor

A date cursor can't handle multiple same-day meetings where some are already processed and others aren't. Instead, `.claude/learning-recap-state.md` tracks **which specific calendar events have been handled**:

```markdown
## Processed events
- id: 1vq77pagc28rqs1te3s9n7e2f4
  date: 2026-07-26
  title: "Claude Code Learning Path - Building API #1"
  status: done
- id: abc123xyz
  date: 2026-07-26
  title: "Claude Code Learning Path - MCP #1"
  status: skipped

## TOC hint shown
- claude-api
```

The `## TOC hint shown` list tracks which course slugs have already been told about the optional `.course-toc.json` source-link feature (see **Course TOC lookup** below) — so the one-time tip only ever appears once per course, not every run. Create this list empty if missing.

- `status: done` — chapter note(s) already saved for this event, never reprocess it.
- `status: skipped` — Gemini notes weren't ready last time this event was checked; retry it.
- Create the file (empty `## Processed events` list) if missing. If an old `last_ran: YYYY-MM-DD` file exists from a previous version of this skill, treat it as an empty processed-events list — don't error, don't try to migrate it, just start tracking events fresh from here.

---

## Modes

### Default — `/learning-recap`

1. Read the processed-events list from `.claude/learning-recap-state.md`.
2. Search Google Calendar (`CALENDAR_ID`) for events whose title contains any entry in `MEETING_TITLE_FILTER`, within the last `LOOKBACK_DAYS` days up to now. This is just a search bound — dedup happens in the next step.
3. For each matching event, check it against the processed-events list:
   - `status: done` → already handled, skip silently (don't mention it in the report unless the user asks).
   - `status: skipped` or **not in the list at all** (new event) → this run will attempt to process it.
4. If nothing needs attempting (all matches are `done`, or no matches at all), report that and stop.
5. Process each event that needs attempting (see **Processing** below). An event whose Gemini doc isn't found yet (notes not ready) is **skipped, not an error** — note its event date and move on to the next event.
6. After each event, upsert its record in the processed-events list: `status: done` on success, `status: skipped` if notes weren't ready. Events stay in the list indefinitely (bounded naturally by `LOOKBACK_DAYS` — an event that ages out of the search window stops being re-checked anyway, so the list doesn't need separate pruning).

### `--url <gemini-doc-url>`

Skip Calendar entirely — process that single Gemini doc directly. There's no calendar event ID to key off of here, so this mode never touches the processed-events state — it's a manual override, not part of the tracked workflow. Fine to reprocess the same doc deliberately via this flag.

### `--date <yyyy-MM-dd>`

Search Calendar for matching-title events on that exact date (ignoring `LOOKBACK_DAYS`). If more than one matches, list them and ask which to process. This mode **does** consult and update the processed-events list — same dedup rules as the default mode — so re-running it on the same date doesn't reprocess an event already marked `done`.

### `--preview` (combinable with any mode above)

**Default behavior (no flag) skips the draft-review checkpoint in step 4** ("Show the full draft to the user before saving anything and ask...") — draft, then go straight to saving. Bruno trusts the classification and wants the end result without an extra round-trip. Pass `--preview` to opt back into that checkpoint for a given run (e.g. a chapter with unusually messy or ambiguous commentary he wants to sanity-check before it's written).

This flag only affects that one checkpoint:

- Step 3's course/chapter confirmation always happens regardless of `--preview` — that's disambiguation the skill can't infer, not a preview.
- The `owner: claude` do-it-now-vs-log-open `AskUserQuestion` (step 4) always happens regardless of `--preview` — that's a substantive decision about executing a task, not a content preview.
- Step 6's report always runs. When `--preview` was **not** used (the default), add one short line per saved chapter note summarizing what it covers (a compressed version of its `## Summary`) so there's still a lightweight sanity check even though the user never saw the draft — not the full draft, just enough to catch something badly wrong.

If the user asks to always see the preview going forward rather than just one run, that's a standing preference — suggest they add `--preview` each time they invoke the command, or note it for yourself for this session; this skill has no persistent per-user settings file of its own.

---

## Processing one transcript

### 1 — Locate the Gemini doc

If not given directly via `--url`: derive it from the calendar event's title and start time using Gemini's filename convention:

`{meeting title} - {YYYY/MM/DD HH:MM TZ} - Notes by Gemini`

Search Google Drive for a file matching that pattern. If not found, the notes likely aren't ready yet (Gemini can take a while to generate them, or Bruno hasn't had the session transcribed) — don't guess at a different file, don't treat this as an error. Record the event (title + date) as **skipped**, tell the user inline ("notes for '<title>' on <date> aren't ready yet, skipping"), and continue to the next matching event.

### 2 — Read and split the transcript

Read the doc via Google Drive MCP. Also note Gemini's own auto-generated "Summary" section at the top of the doc — useful as a scaffold, but the full `Bruno Coelho's Presentation:` stream is the primary source since Gemini's summary can miss nuance.

Split the transcript body into the two streams described above (narration vs. commentary), preserving order.

### 3 — Identify course and chapter

Meeting titles here don't reliably encode course/chapter, so never infer this silently:

1. Read the root `CLAUDE.md` course table for the list of valid course dir slugs.
2. Give the user a 1-2 sentence synopsis of what the narration covered, then ask: "Which course is this for? [list course slugs from the table]"
3. Propose a chapter slug (kebab-case) from the content and ask them to confirm or rename it.

Wait for confirmation before proceeding — the answer determines the save path.

#### Course TOC lookup (optional, per-course)

If `2ndbrain/<course-slug>/.course-toc.json` exists — an array of entries mirroring the course platform's own chapter list, each `{title, url}` and optionally a `section` field naming the curriculum section/module the lesson belongs to (Bruno generates this file by hand per course, it's not something this skill scrapes itself) — use it to auto-attach source links and, when `section` is present, to file the chapter note into the matching section subfolder (see **Section-aware save path** in step 5).

**Generating the file**: on the course's own site, open DevTools console on a page showing the full curriculum/sidebar nav, then run a query that grabs every lesson link's visible text and `href`, and — if the sidebar renders section headers as distinct elements — the section each lesson falls under. This example targets Skilljar's course platform specifically, whose sidebar groups lessons under `h3.section-title` headers and represents each lesson as a `div.lesson-row` (with the lesson name in its `title` attribute) — both rendered via JavaScript after login, so a plain page fetch can't see them, this has to run in the live, authenticated browser tab:

```js
(function(){var c=document.getElementById("curriculum-list-2");if(!c){console.error("container not found");return;}var nodes=c.querySelectorAll("h3.section-title, div.lesson-row");var result=[];var currentSection=null;nodes.forEach(function(n){if(n.tagName==='H3'){currentSection=n.textContent.trim();return;}var a=n.querySelector('a');if(!a){a=n.closest('a');}var url=null;if(a){url=a.href;}result.push({section:currentSection,title:n.getAttribute('title'),url:url});});console.table(result);console.log(JSON.stringify(result,null,2));})();
```

The container/section/lesson selectors will differ per course platform — inspect the page's sidebar nav element to find the right ones, swap them in, then paste the printed JSON into `.course-toc.json` for that course. If a course's sidebar doesn't expose distinct section elements at all, fall back to the flatter form without `section` (just `{title, url}` per entry, as generated by the single-selector script from an earlier version of this skill) — the section-aware save path in step 5 degrades gracefully to a flat save when no `section` field is present. Chrome's console can mangle a multi-line paste (e.g. turning `a?a.href:null` into `a?.href:null`) — if you hit a syntax error, retype it or ask Claude for a version that avoids the problematic construct rather than fighting the paste.

Once generated, use the file to auto-attach source links:

1. Match narration content against TOC entry titles (case-insensitive, substring/fuzzy match is fine — a session's narration may cover more than one TOC entry, e.g. a lesson plus its exercise plus a quiz).
2. Collect every TOC entry that plausibly corresponds to material in this transcript.
3. If genuinely ambiguous (multiple TOC entries could match and it's not clear which), just include the ones you're confident about rather than asking — this is a nice-to-have convenience link, not something worth interrupting the flow over.
4. If every matched TOC entry shares the same `section` value, that's the session's section for filing purposes (step 5). If matched entries span more than one section (rare — a session usually stays within one section), ask the user which section to file the note under rather than guessing.

If the file doesn't exist for this course, check the state file's `## TOC hint shown` list (see below). If this course slug isn't in that list yet, mention the feature **once**, briefly, as part of the final report (not a blocking prompt): something like "Tip: `2ndbrain/<course-slug>/.course-toc.json` doesn't exist yet — if you generate one (see the console snippet from our earlier chat, or the equivalent for this course's platform), I'll auto-attach chapter source links instead of you pasting URLs manually." Then add the course slug to that list so this never repeats for the same course. If the course slug is already in the list, skip mentioning it entirely — no error, no repeated prompts.

### 4 — Draft the chapter note

Build a draft following `2ndbrain/CLAUDE.md`'s structure:

- **`## Source`** (only if the TOC lookup above found matches) — one markdown link per matched TOC entry, in curriculum order: `- [Temperature](https://anthropic.skilljar.com/...)`. Place this section first, before `## Summary`.
- **`## Summary`** — synthesized from the narration stream (cross-checked against Gemini's own summary), organized by topic rather than strictly chronologically. Weight toward whatever topics Bruno's commentary clusters around (if he paused and commented right after something, that's a signal it matters) — call those out as highlights.
- **`## My Insights`** — drawn from the commentary stream: his own takes, connections, reactions. Lightly cleaned of filler ("um", false starts) but never reworded into a different meaning.
- **`## Ideas`** — commentary that reads as "I should try/build X."
- **`## Challenges`** — commentary that reads as "this is confusing / I need to dig into X."
- **`## Actions`** — commentary phrased as a concrete commitment ("I need to…", "let me…", "I'll…") **or** a request directed at Claude (see above), formatted `- [ ] description (owner: bruno|claude) (due: yyyy-MM-dd)`. Only include a due date if Bruno actually stated one — never invent one. Omitting `(owner: ...)` entirely is only for legacy notes predating this tag — always include it going forward.

Classifying commentary into Insights/Ideas/Challenges/Actions is inherently fuzzy. **Only if `--preview` was passed** (see that mode's section above), show the full draft to the user before saving anything and ask: "I bucketed your commentary like this — anything misclassified, missing, or worth cutting?" and apply any corrections before writing the file. Otherwise (the default), skip straight to saving.

#### Handling `owner: claude` actions

For every action tagged `(owner: claude)` found in this transcript, don't assume Bruno wants it done immediately just because it's this skill's job to process the transcript — he may only have been thinking out loud, or the ask may need more context than this one recap run has. Before finalizing the draft, batch all of this run's `owner: claude` actions into a single `AskUserQuestion` (one question per action, or one multi-part question if there's just one) offering: **do it now** vs. **log it as open and stop there**. Do this once per action, not once per run of unrelated actions from different sessions.

- **Do it now**: complete the task (e.g. research + write a standalone note, following the existing convention for ad-hoc notes in step 5 — saved into that section's `research/` subdirectory), then mark the action `[x]` in the chapter note with a short pointer to what was produced (e.g. "done same session, see `research/jupyter-notebooks-setup.md`"), instead of leaving it open.
- **Log it as open and stop there**: leave it as `- [ ] ... (owner: claude)` in the Actions section, unstarted. It'll then surface next time `/learning-actions` is run, same as any other open action — just filed under Claude instead of Bruno.

Never silently execute one of these without asking, and never silently drop it from the Actions list either — both failure modes defeat the point of surfacing it.

### 5 — Save

#### Section-aware save path

- If the TOC lookup found a `section` for this session's chapter (step 3): path is `2ndbrain/<course-slug>/<section-slug>/<chapter-slug>.md`, where `<section-slug>` is the kebab-case form of the TOC `section` string. Create the section directory if it doesn't exist yet.
- Otherwise (no TOC, no `section` field, or this course doesn't use section folders): path is `2ndbrain/<course-slug>/<chapter-slug>.md`, same as before.
- Create the course directory if it doesn't exist yet either way.
- Any `owner: claude` research notes produced during this same run (see above) are saved into a `research/` subdirectory alongside the chapter note — `2ndbrain/<course-slug>/<section-slug>/research/<note-slug>.md` if a section applies, otherwise `2ndbrain/<course-slug>/research/<note-slug>.md` at the course root. Create that `research/` subdirectory if it doesn't exist yet. Since these notes live one level deeper than the chapter note, any link from a research note back to its parent chapter note needs a `../` prefix (e.g. `[extended-thinking](../extended-thinking.md)`); conversely, a chapter note linking into `research/` needs that prefix (e.g. `[tool-loading-strategy.md](research/tool-loading-strategy.md)`). Links between two research notes in the same `research/` dir need no prefix at all.

Bruno sometimes drops standalone ad-hoc research notes directly into a course's `2ndbrain/<course-slug>/` dir (course root, not inside any section folder) outside this workflow entirely, since they aren't tied to one specific lesson. These still belong in that location's `research/` subdirectory (course-root `research/`, since there's no section to nest under), not loose at the course root. These stay separate files — never fold their content into `## My Insights`. If one looks topically related to the chapter being processed, just add a link to it under `## My Insights` (e.g. "See also: `research/other-note.md`") rather than merging or deleting it.
- Update `2ndbrain/CLAUDE.md`'s Progress section: mark the chapter `[x]` with a link to the new file, and update "Current course" if it changed.
- If this looks like the last chapter of the course, ask the user before marking the course's status as complete in the root `CLAUDE.md` table — don't infer that silently.

### 6 — Report

Tell the user:
- Where each chapter note was saved (one per successfully processed event). If `--preview` was **not** used (the default), add one short line per note summarizing what it covers (see that mode's section above).
- Which course/chapter progress fields were updated.
- Any events **skipped** this run for missing notes — title + date for each — and that they're recorded as `skipped` so they'll be retried automatically next run.
- Remind them that `/learning-actions` (from the `learning-actions` skill) rolls up this chapter's new `## Actions` items with everything else open across the repo.
