# Custom Claude Code agent for the reminder tools (bash + cron)

Standalone reference note (not a course chapter recap), requested by Bruno during the [introducing-tool-use](introducing-tool-use.md) session — he asked whether the three reminder tools (get current time, add duration to datetime, set a reminder) could be reimplemented as a custom Claude Code agent instead of Python functions called through the API.

## Short answer

Yes for the first two, only partially for the third.

- **Get current time**: trivial — an agent's Bash tool can just run `date`. No need for a dedicated Python tool function at all.
- **Add duration to datetime**: also fine via Bash (`date -d "+379 days" "+%Y-%m-%d"` on GNU date, or a one-line Python `datetime + timedelta` call through Bash). This sidesteps Claude's known weakness at date arithmetic entirely — the agent doesn't do the math itself, the shell does.
- **Set a reminder**: this is the one that doesn't map cleanly onto "a Claude Code agent" alone. An agent (as defined in `.claude/agents/*.md`) is *invoked* — it runs once, does its work, and returns. It isn't a persistent process, so it can't itself "wait" until the reminder's due time. Something has to run unattended later and re-invoke Claude (or fire a notification) when the time comes.

## Why cron fits, but isn't the only option

`cron` (or the simpler one-shot `at` command) is a completely reasonable way to solve the "run something later, unattended" problem — write a crontab entry or `at` job that fires a script at the target time (send a system notification, write to a file Claude reads next session, ping a webhook, etc.). That's a robust, standard Linux mechanism and doesn't require Claude to be running at all in between.

Worth noting: this exact environment already ships two tools that solve the same problem without hand-rolling cron — `CronCreate`/`CronList` (schedule a recurring or one-off cloud agent run) and `ScheduleWakeup` (schedule this session to resume later with a given prompt). If the goal is specifically "a Claude Code agent reminds me of something later," those are the more antifragile default over shelling out to `crontab` by hand, since they're already integrated with how the agent gets re-invoked and doesn't require managing a separate system-level scheduler.

## Suggested shape

- Tool functions (`get_current_time`, `add_duration_to_datetime`) → plain Bash/Python, no agent needed, exactly as the course exercise already does.
- Reminder delivery → `CronCreate` (or `ScheduleWakeup` for a single one-off) rather than raw `crontab`/`at`, unless working outside a Claude Code session entirely (e.g. a bare script with no Claude runtime involved), in which case `at`/`cron` calling a small script is the right fallback.
