# Review Notes

Optional background.

Key lessons from the live evolution of this setup:

- keep a separate source-of-truth layer instead of burying everything in heartbeat text
- use `m365 outlook message list` with explicit `--startTime` / `--endTime` windows, not conversation-only views that can hide individual unread messages
- preserve thread context on replies by posting to the Graph `/me/messages/{id}/reply` endpoint instead of starting a fresh `m365 outlook mail send`
- check *all relevant calendars* before offering or booking time
- keep one canonical live task file and a separate completed-task archive
- separate *task management* from *daily task prep*
- treat the outreach tracker as a live source of truth and update it in the same turn
- treat meeting notes as an operational signal source, not passive documents
- use a priority map to decide what matters
- use an auto-resolver policy to decide whether to act, draft, escalate, or ignore
- keep proactive update transport configurable instead of hardcoding one surface
- keep cron prompts DRY and let skills hold workflow logic
- keep optional backup and self-update as explicit choices, not assumptions
