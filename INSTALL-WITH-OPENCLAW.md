# Install With OpenClaw

Follow this in order.

## 0. Get the m365 CLI working

Complete `SETUP-M365.md` first.

**Important:** If OpenClaw runs as a dedicated service user (e.g. `openclaw`), the `m365` CLI must be accessible to that user — not just to your admin/SSH account. Common approaches:

- Install `m365` system-wide with `sudo npm install -g @pnp/cli-microsoft365` so it lands in `/usr/bin/`
- Or install under the service user: `sudo -u openclaw -H npm install -g @pnp/cli-microsoft365`
- Run `m365 login` as the service user too: `sudo -u openclaw -H m365 login --authType deviceCode ...` — tokens are per-user and do not transfer between OS accounts
- If the service runs via systemd, ensure `/usr/bin` (or wherever `m365` lives) is in the service's PATH. You may need to add `PATH=...` to the `EnvironmentFile` referenced by the unit.

The default install is **read-only** against Outlook, Calendar, and SharePoint. The only Graph write scope the Entra app ships with is `Tasks.ReadWrite`, so the Microsoft To Do mirror of `clawchief/tasks.md` can run. Every other would-be write (sending mail, creating events, moving messages, updating the SharePoint tracker) is drafted to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` for manual approval.

Do not continue until these all work:

- `m365 status` shows the correct operating account and Entra app registration
- `m365 outlook message list --folderName inbox --startTime "$(date -u -d '-7 days' +%Y-%m-%dT%H:%M:%SZ)" --output json` returns recent inbox messages
- `m365 request --url "@graph/me/events?\$filter=start/dateTime ge '$(date -u +%Y-%m-%dT00:00:00)' and start/dateTime le '$(date -u -d '+2 days' +%Y-%m-%dT00:00:00)'&\$orderby=start/dateTime&\$top=50" --output json` returns events (the v11 CLI dropped `outlook event list`, so calendar reads now go through Graph)
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

## 2. Find the real workspace path

OpenClaw's workspace location depends on your installation. Common locations:

- `~/.openclaw/workspace/` (user installs)
- `/var/lib/openclaw/.openclaw/workspace/` (systemd service with a dedicated `openclaw` user)

Check which user the service runs as and where its home directory is:

```bash
getent passwd openclaw          # shows home directory
systemctl show openclaw --property=ExecStart  # shows the binary path
ls /var/lib/openclaw/.openclaw/workspace/     # typical service install
```

Skills typically live at `<workspace>/skills/`, **not** at `~/.openclaw/skills/` as a sibling to `workspace/`. Check your existing workspace layout before copying.

## 3. Install the skills

Copy these directories into `<workspace>/skills/` alongside any existing OpenClaw skills (e.g. `memento-memory`, `x-search`):

- `skills/executive-assistant`
- `skills/business-development`
- `skills/daily-task-manager`
- `skills/daily-task-prep`

If the service runs as a dedicated user, fix ownership after copying:
```bash
sudo chown -R openclaw:openclaw <workspace>/skills/
```

## 4. Install the workspace files

Copy these into `<workspace>/`:

- `clawchief/`
- `workspace/HEARTBEAT.md`
- `workspace/TOOLS.md`
- `workspace/memory/meeting-notes-state.json`

Note:
- `workspace/tasks/current.md` is included only as a deprecation note for older installs
- the live task source of truth is now `clawchief/tasks.md`
- if the workspace already has `HEARTBEAT.md`, `TOOLS.md`, etc. from OpenClaw's default scaffold, the pack's versions replace them

## 5. Add your private workspace files

This public pack does *not* ship personal context files.

Create your own versions of these if your setup depends on them:

- `AGENTS.md`
- `SOUL.md`
- `USER.md`
- `IDENTITY.md`
- `MEMORY.md`
- `memory/`

## 6. Replace placeholders

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

## 7. Configure the sandbox

If OpenClaw uses Docker sandboxing (`sandbox.mode: "all"` or `"non-main"` in `openclaw.json`), the workspace needs write access for the pack to function:

- `clawchief/tasks.md` and `clawchief/tasks-completed.md` are written during task management
- `workspace/memory/` is updated by meeting-note ingestion
- `HEARTBEAT.md` is updated during heartbeat checks

Set `"workspaceAccess": "rw"` in the sandbox config, or use `"mode": "non-main"` so the main session (where the EA sweep runs) executes on the host with full access to `m365` and the workspace.

If sandbox mode is `"all"`, tools installed on the host (like `m365`) are not available inside the Docker container. Use `"non-main"` or bind-mount the `m365` binary and its token storage into the container.

## 8. Create the cron jobs

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

## 9. Validate the install

Run `INSTALL-CHECKLIST.md`.
