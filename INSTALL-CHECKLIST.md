# Install Checklist

The install is good only if every item below passes.

## M365 CLI

- [ ] `m365 status` shows the correct operating account, Entra app id, and tenant
- [ ] `m365 outlook message list --folderName inbox --startTime "$(date -u -d '-7 days' +%Y-%m-%dT%H:%M:%SZ)" --output json` returns recent inbox messages
- [ ] `m365 request --url "@graph/me/events?\$filter=start/dateTime ge '$(date -u +%Y-%m-%dT00:00:00)' and start/dateTime le '$(date -u -d '+2 days' +%Y-%m-%dT00:00:00)'&\$orderby=start/dateTime&\$top=50" --output json` returns events (the v11 CLI dropped `outlook event list`, so calendar reads now go through Graph)
- [ ] `m365 request --url "@graph/me/calendars"` returns the calendar set you expect
- [ ] `m365 spo listitem list --webUrl {{SHAREPOINT_TRACKER_SITE_URL}} --listTitle {{SHAREPOINT_TRACKER_LIST_TITLE}}` reads the outreach tracker
- [ ] `m365 todo task list --listName {{MSTODO_MIRROR_LIST}}` succeeds (list exists and is readable)
- [ ] OneNote / SharePoint read works if meeting-notes ingestion is enabled

## Operating mode

- [ ] `workspace/TOOLS.md` has an explicit `read_only:` line (`true` for first-run, `false` only after the Entra permission swap in `SETUP-M365.md` has been done)
- [ ] the Entra app's delegated Graph permissions match the section in `SETUP-M365.md` that corresponds to the current `read_only` setting (read-only default or the `Flip to write mode` upgrade set)
- [ ] `{{PRIMARY_UPDATE_CHANNEL}}` / `{{PRIMARY_UPDATE_TARGET}}` is a channel the principal actually watches — in read-only mode every would-be action gets drafted here
- [ ] if flipping modes, `m365 logout` + `m365 login --authType deviceCode --appId {{ENTRA_APP_ID}} --tenant {{ENTRA_TENANT_ID}} --connectionName clawchief` has been re-run so the new scopes are consented on the active token

## Skills

- [ ] `executive-assistant` is installed in `~/.openclaw/skills/`
- [ ] `business-development` is installed in `~/.openclaw/skills/`
- [ ] `daily-task-manager` is installed in `~/.openclaw/skills/`
- [ ] `daily-task-prep` is installed in `~/.openclaw/skills/`

## Workspace

- [ ] `clawchief/priority-map.md` is installed
- [ ] `clawchief/auto-resolver.md` is installed
- [ ] `clawchief/meeting-notes.md` is installed
- [ ] `clawchief/tasks.md` is installed
- [ ] `clawchief/tasks-completed.md` is installed
- [ ] `workspace/HEARTBEAT.md` is installed
- [ ] `workspace/TOOLS.md` is installed
- [ ] `workspace/memory/meeting-notes-state.json` is installed
- [ ] private workspace files have been authored if your setup depends on them
- [ ] all placeholders are replaced

## Behavior

- [ ] heartbeat reads the source-of-truth files instead of duplicating workflow logic
- [ ] proactive updates route to the intended channel + target
- [ ] when `read_only: true`, no Graph / SharePoint writes are being issued against Outlook mail, events, or the tracker — every would-be write becomes a drafted proposal on `{{PRIMARY_UPDATE_CHANNEL}}`
- [ ] the Microsoft To Do mirror is the only Graph write in read-only mode (`Tasks.ReadWrite` is the only write scope granted in the default permission set)
- [ ] inbox sweeps use `m365 outlook message list` with explicit time-window filters (not conversation-only views)
- [ ] scheduling checks all relevant calendars before booking (and in read-only mode, drafts the proposed event to the update channel instead of creating it)
- [ ] the task system uses `clawchief/tasks.md` as the live source of truth
- [ ] prior-day completed tasks archive into `clawchief/tasks-completed.md`
- [ ] meeting notes are treated as a live signal source if enabled
- [ ] business-development work treats the outreach tracker as the live source of truth (and in read-only mode, drafts proposed row changes to the update channel instead of writing them)
- [ ] daily task prep promotes due-today items into `## Today`
- [ ] daily task prep mirrors the final principal `## Today` list into the `{{MSTODO_MIRROR_LIST}}` Microsoft To Do list as a one-way human-visibility surface (runs in both read-only and write modes)

## Cron

- [ ] executive assistant sweep exists
- [ ] daily task prep exists
- [ ] daily business-development sourcing exists
- [ ] optional nightly backup is configured only if desired
- [ ] optional self-update is enabled only if explicitly desired

If any box is unchecked, the install is not done.
