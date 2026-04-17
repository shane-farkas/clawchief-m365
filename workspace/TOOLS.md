# TOOLS.md - Local Notes

Use this file for environment-specific details that should *not* live in public skills.

Examples:
- real inboxes and calendars to check
- tracker / CRM notes
- browser profile preferences
- business-development target segments
- voice, device, or camera names
- local SSH aliases

## Operating mode

- `read_only: true`

When `read_only` is true, every skill in this pack must behave as follows:

- **Reads are unrestricted** — inbox, calendar, conversation history, SharePoint tracker, Microsoft To Do, OneNote, and local files may all be read freely.
- **Graph / SharePoint writes against M365 are forbidden**: do **not** send mail, create / update / cancel events, mark messages read, move or archive messages, add or update SharePoint list items, or edit OneNote / Word documents.
- **Local-file writes are still allowed**: `clawchief/tasks.md`, `clawchief/tasks-completed.md`, and `workspace/memory/*` continue to be updated normally.
- **Microsoft To Do mirror stays on**: the `daily-task-prep` skill may still rebuild `{{MSTODO_MIRROR_LIST}}` from `clawchief/tasks.md` via `m365 todo task add / remove`. This is the one write path that is explicitly allowed in read-only mode because the blast radius is a single private list the principal controls.
- **Every would-be write becomes a drafted proposal**: surface it to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` as a short preview (recipients, subject or event title, proposed body or field change, the exact message id / event id / list item id it would touch) and wait for the principal to act manually.

Flip to full write mode later by:

1. changing this knob to `read_only: false`
2. upgrading the Entra app registration's delegated Graph permissions (see `SETUP-M365.md` section **Flip to write mode**) and re-granting admin consent
3. running `m365 logout` and then `m365 login --authType deviceCode --appId {{ENTRA_APP_ID}} --tenant {{ENTRA_TENANT_ID}} --connectionName clawchief` again so the new scopes get consented

Both steps are required — flipping the TOOLS.md knob without re-consenting new scopes will just cause Graph writes to fail with 403; re-consenting without flipping the knob will still behave as read-only because the skills check this file first.

## Communication defaults

- Principal name: `{{OWNER_NAME}}`
- Assistant name: `{{ASSISTANT_NAME}}`
- Primary assistant email: `{{ASSISTANT_EMAIL}}`
- Primary work email / calendar: `{{PRIMARY_WORK_EMAIL}}`
- Personal email: `{{PERSONAL_EMAIL}}`
- Time zone: `{{TIMEZONE}}`
- Primary proactive update route: `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}`

Single-operator note: `m365 login` authenticates exactly one user at a time and every Graph call resolves to `me`. If you are the only operator (you signed in as yourself rather than a separate shared mailbox), set `{{ASSISTANT_EMAIL}}` to the same value as `{{PRIMARY_WORK_EMAIL}}`. The authority rules in `skills/executive-assistant/references/access-and-defaults.md` then collapse to "always authorized" because every action is already being taken on your own behalf.

## Calendars to check

Calendars that count as real availability constraints:

- `{{PRIMARY_WORK_EMAIL}}`
- `{{PERSONAL_EMAIL}}`

Add additional calendars here as needed (e.g. shared team calendar, family calendar as conflict source only).

## Calendar display preferences

- **Always display times in the principal's time zone** (`{{TIMEZONE}}`). Never show raw UTC.
- When reading events via Graph, pass the `Prefer` header so the API returns local times:
  ```bash
  m365 request \
    --url "@graph/me/calendarView?startDateTime=$(date -u +%Y-%m-%dT00:00:00Z)&endDateTime=$(date -u -d '+2 days' +%Y-%m-%dT00:00:00Z)&\$orderby=start/dateTime&\$top=50" \
    --accept "application/json" \
    -H "Prefer: outlook.timezone=\"{{TIMEZONE}}\"" \
    --output json
  ```
  If the `Prefer` header is not supported by the CLI, convert UTC timestamps to `{{TIMEZONE}}` before displaying.
- **Format:** `9:30 AM PT` (12-hour clock, PT/PDT shorthand)
- **Skip these event types in daily summaries:**
  - Cancelled events
  - Events the principal declined
  - All-day events unless they look like deadlines or PTO (use subject line to judge)
  - Recurring "focus time" or "lunch" blocks the principal set for themselves
- **Include in daily summaries:**
  - Meetings with other people (show attendees)
  - Calls and interviews
  - Deadlines or reminders set as calendar events
  - Events from all checked calendars, not just the default

## Outreach tracker

- Live tracker site: `{{SHAREPOINT_TRACKER_SITE_URL}}`
- Live tracker list title: `{{SHAREPOINT_TRACKER_LIST_TITLE}}`
- Treat this as the source of truth for outreach state.
- Document the current list's internal field names here if your schema is customized (SharePoint display names and internal names can differ, and `m365 spo listitem set` expects internal names).
- If you host the tracker as an Excel workbook in SharePoint or OneDrive instead of a SharePoint list, document the drive item id and table name here as well.

## Microsoft To Do mirror

- Mirror list name: `{{MSTODO_MIRROR_LIST}}`
- Canonical task source: `clawchief/tasks.md`
- The `daily-task-prep` skill rebuilds this Microsoft To Do list at the end of each run from the principal `## Today` section of `clawchief/tasks.md`.
- The mirror is one-way: tasks.md is the source of truth, the To Do list is a phone / desktop visibility surface. Do not edit the To Do list expecting changes to flow back into tasks.md — check items off in tasks.md (or let the next prep run rebuild the list).
- Assistant-owned tasks are intentionally excluded from the mirror.

## Business-development playbook

Document your default sourcing motion here.

Example knobs to define:
- target geography
- primary target segment
- optional secondary target segment
- default daily batch size
- verification requirements
- default outreach tone
- follow-up cadence overrides

Example:
- Geography: `{{TARGET_GEOGRAPHY}}`
- Primary segment: `{{TARGET_MARKET}}`
- Secondary segment: optional custom segment
- Daily new leads: 10
- Verify working website + real email before adding
