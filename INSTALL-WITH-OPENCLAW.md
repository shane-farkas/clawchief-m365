# Install With OpenClaw

Follow this in order.

## 0. Get the m365 CLI working

Complete `SETUP-M365.md` first.

The default install is **read-only** against Outlook, Calendar, and SharePoint. The only Graph write scope the Entra app ships with is `Tasks.ReadWrite`, so the Microsoft To Do mirror of `clawchief/tasks.md` can run. Every other would-be write (sending mail, creating events, moving messages, updating the SharePoint tracker) is drafted to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` for manual approval.

Do not continue until these all work:

- `m365 status` shows the correct operating account and Entra app registration
- `m365 outlook message list --folderName inbox --startTime ...` returns recent inbox messages
- `m365 outlook event list --calendarName Calendar --startDateTime ... --endDateTime ...` returns events
- `m365 request --url "@graph/me/calendars"` returns the calendar set you expect
- `m365 spo listitem list --webUrl {{SHAREPOINT_TRACKER_SITE_URL}} --listTitle {{SHAREPOINT_TRACKER_LIST_TITLE}}` reads the outreach tracker
- `m365 todo list list` shows the Microsoft To Do list named `{{MSTODO_MIRROR_LIST}}` (create it with `m365 todo list add --name "{{MSTODO_MIRROR_LIST}}"` if it does not exist)
- OneNote / SharePoint Word read works if you plan to use meeting-notes ingestion
- `workspace/TOOLS.md` has `read_only: true` in its `Operating mode` section
- `{{PRIMARY_UPDATE_CHANNEL}}` and `{{PRIMARY_UPDATE_TARGET}}` route to a surface you actually watch (Telegram chat, Slack DM, etc.) — in read-only mode this is the only way the assistant can reach you

When you are ready to flip to full write mode, follow the `Flip to write mode` section at the bottom of `SETUP-M365.md` — both the Entra permission swap *and* the `read_only` flag in `workspace/TOOLS.md` have to change.

## 1. Gather install values

Collect these before editing files:

- `{{OWNER_NAME}}`
- `{{ASSISTANT_NAME}}`
- `{{ASSISTANT_EMAIL}}`
- `{{PRIMARY_WORK_EMAIL}}`
- `{{PERSONAL_EMAIL}}`
- `{{BUSINESS_NAME}}`
- `{{BUSINESS_URL}}`
- `{{TIMEZONE}}`
- `{{PRIMARY_UPDATE_CHANNEL}}`
- `{{PRIMARY_UPDATE_TARGET}}`
- `{{ENTRA_APP_ID}}`
- `{{ENTRA_TENANT_ID}}`
- `{{SHAREPOINT_TRACKER_SITE_URL}}`
- `{{SHAREPOINT_TRACKER_LIST_TITLE}}`
- `{{MSTODO_MIRROR_LIST}}`
- `{{TARGET_MARKET}}`
- `{{TARGET_GEOGRAPHY}}`

Optional additional values if you use more calendars:

- `{{SECONDARY_CALENDAR_EMAIL_1}}`
- `{{SECONDARY_CALENDAR_EMAIL_2}}`
- `{{SECONDARY_CALENDAR_EMAIL_3}}`

## 2. Install the skills

Copy these directories into `~/.openclaw/skills/`:

- `skills/executive-assistant`
- `skills/business-development`
- `skills/daily-task-manager`
- `skills/daily-task-prep`

## 3. Install the workspace files

Copy these into `~/.openclaw/workspace/`:

- `clawchief/`
- `workspace/HEARTBEAT.md`
- `workspace/TOOLS.md`
- `workspace/memory/meeting-notes-state.json`

Note:
- `workspace/tasks/current.md` is included only as a deprecation note for older installs
- the live task source of truth is now `clawchief/tasks.md`

## 4. Add your private workspace files

This public pack does *not* ship personal context files.

Create your own versions of these if your setup depends on them:

- `AGENTS.md`
- `SOUL.md`
- `USER.md`
- `IDENTITY.md`
- `MEMORY.md`
- `memory/`

## 5. Replace placeholders

Replace every placeholder token before testing.

Minimum search list:

- `{{OWNER_NAME}}`
- `{{ASSISTANT_NAME}}`
- `{{ASSISTANT_EMAIL}}`
- `{{PRIMARY_WORK_EMAIL}}`
- `{{PERSONAL_EMAIL}}`
- `{{BUSINESS_NAME}}`
- `{{BUSINESS_URL}}`
- `{{TIMEZONE}}`
- `{{PRIMARY_UPDATE_CHANNEL}}`
- `{{PRIMARY_UPDATE_TARGET}}`
- `{{ENTRA_APP_ID}}`
- `{{ENTRA_TENANT_ID}}`
- `{{SHAREPOINT_TRACKER_SITE_URL}}`
- `{{SHAREPOINT_TRACKER_LIST_TITLE}}`
- `{{MSTODO_MIRROR_LIST}}`
- `{{TARGET_MARKET}}`
- `{{TARGET_GEOGRAPHY}}`

Then customize these files for your real workflow:

- `workspace/TOOLS.md`
- `clawchief/priority-map.md`
- `clawchief/tasks.md`
- `skills/business-development/resources/partners.md`
- `cron/jobs.template.json`

## 6. Create the cron jobs

Use `cron/jobs.template.json` as the starting pattern.

Recommended starting jobs:

1. executive assistant sweep
2. daily task prep
3. daily business-development sourcing

Optional jobs:

4. nightly backup
5. self-update

Notes:
- keep prompts short and let the skill carry the workflow details
- if your sweep window cannot be expressed cleanly in one cron, split it into a main recurring job plus boundary jobs
- keep backup disabled until the remote repo and allowlist are correct
- keep self-update disabled unless you explicitly want that behavior

## 7. Validate the install

Run `INSTALL-CHECKLIST.md`.
