# M365 Setup

This is a hard prerequisite.

The pack uses [CLI for Microsoft 365](https://github.com/pnp/cli-microsoft365) (`m365`) for Outlook mail, Outlook calendar, and SharePoint list access.

## 1. Install the CLI

```bash
npm install -g @pnp/cli-microsoft365
m365 --version
```

`m365` requires Node.js 18+.

## 2. Create a Microsoft Entra app registration

You need an Entra (Azure AD) app registration before `m365 login` will work for this pack. The CLI ships with its own default `appId`, but a dedicated app registration keeps scopes, consent, and audit cleaner.

Create it in the [Entra admin center](https://entra.microsoft.com):

1. Go to **Identity -> Applications -> App registrations -> New registration**
2. Name: `clawchief-m365` (or whatever you want)
3. Supported account types: **Single tenant** (unless you specifically need multi-tenant)
4. Redirect URI: **Public client / native (mobile & desktop)** -> `http://localhost`
5. Register the app
6. Open **Authentication** and make sure **Allow public client flows** is set to **Yes**
7. Open **API permissions** and add the following **delegated** Microsoft Graph permissions. The default install is **read-only** against Outlook, Calendar, and SharePoint — the only write scope that ships enabled is `Tasks.ReadWrite`, because the Microsoft To Do mirror of the daily `## Today` list is explicitly allowed in read-only mode (the blast radius is a single private list the principal controls):
   - `Mail.Read`
   - `Calendars.Read`
   - `MailboxSettings.Read`
   - `Sites.Read.All` (read the SharePoint outreach tracker)
   - `Files.Read.All` (read OneDrive / shared documents)
   - `Tasks.ReadWrite` (kept on so the Microsoft To Do mirror of `clawchief/tasks.md` keeps working)
   - `Notes.Read.All` (optional, for OneNote meeting-notes ingestion)
   - `User.Read`
8. Click **Grant admin consent** for the tenant
9. Copy the **Application (client) ID** and the **Directory (tenant) ID** from the **Overview** page

### Flip to write mode

When you are ready to let the assistant actually send mail, create / update / cancel events, mark messages read or archive them, and update the SharePoint outreach tracker, do **all** of the following:

1. in the Entra admin center, open the same app registration and swap the permission set to:
   - `Mail.ReadWrite` (replaces `Mail.Read`)
   - `Mail.Send` (new)
   - `Calendars.ReadWrite` (replaces `Calendars.Read`)
   - `MailboxSettings.Read` (unchanged)
   - `Sites.ReadWrite.All` (replaces `Sites.Read.All`)
   - `Files.ReadWrite.All` (replaces `Files.Read.All`)
   - `Tasks.ReadWrite` (unchanged)
   - `Notes.Read.All` (unchanged, upgrade to `Notes.ReadWrite.All` if you also want the assistant editing OneNote)
   - `User.Read` (unchanged)
2. click **Grant admin consent** again
3. flip `read_only: true` to `read_only: false` in `workspace/TOOLS.md`
4. re-run `m365 logout` and then `m365 login --authType deviceCode --appId {{ENTRA_APP_ID}} --tenant {{ENTRA_TENANT_ID}} --connectionName clawchief` so the newly consented scopes are attached to the active token

Both the Entra permission swap *and* the `read_only` flag must change. Skipping the Entra step means Graph writes fail with 403; skipping the flag means the skills keep drafting proposals instead of acting even though they technically have permission.

## 3. Log in with device code flow

```bash
m365 login \
  --authType deviceCode \
  --appId {{ENTRA_APP_ID}} \
  --tenant {{ENTRA_TENANT_ID}} \
  --connectionName clawchief
```

Complete the device code prompt in a browser using the operating mailbox account (`{{ASSISTANT_EMAIL}}`).

Persist the active connection:

```bash
m365 connection use clawchief
m365 status
```

`m365 status` should show the operating account, the app registration id, and the tenant.

## 4. Verify Outlook message list

```bash
m365 outlook message list \
  --folderName inbox \
  --startTime "$(date -u -v-7d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '-7 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --output json
```

Important: use Outlook **message-level** listing over defined time windows for inbox sweeps. Do not rely on conversation-only views that hide individual unread messages.

## 5. Verify Outlook calendar read

`m365` v11 dropped the dedicated `outlook event list` / `outlook calendar get` commands — calendar reads now go through `m365 request` against Microsoft Graph.

```bash
m365 request --url "@graph/me/events?\$filter=start/dateTime ge '$(date -u +%Y-%m-%dT00:00:00)' and start/dateTime le '$(date -u -d '+2 days' +%Y-%m-%dT00:00:00)'&\$orderby=start/dateTime&\$top=50" --output json
```

If the principal has more than one calendar (work, personal, shared), list them explicitly through Graph:

```bash
m365 request --url "@graph/me/calendars" --output json
```

## 6. Verify Outlook calendar write via Graph

The `m365` CLI does not ship a dedicated `event add` / `event set` command. Creating and updating events goes through `m365 request` against the Microsoft Graph `/me/events` endpoint. Smoke-test it with a throwaway event and then delete it:

```bash
m365 request \
  --method post \
  --url "@graph/me/events" \
  --content-type "application/json" \
  --body '{
    "subject":"clawchief smoke test",
    "start":{"dateTime":"2099-01-01T09:00:00","timeZone":"UTC"},
    "end":{"dateTime":"2099-01-01T09:15:00","timeZone":"UTC"}
  }'

m365 request --method delete --url "@graph/me/events/<event-id-from-previous-response>"
```

## 7. Verify SharePoint outreach tracker

The outreach tracker in this pack lives in a SharePoint list, not a Google Sheet. Verify that the operating account can read it:

```bash
m365 spo listitem list \
  --webUrl "{{SHAREPOINT_TRACKER_SITE_URL}}" \
  --listTitle "{{SHAREPOINT_TRACKER_LIST_TITLE}}" \
  --output json
```

The list must be on a SharePoint site the operating mailbox can access. An Excel table inside SharePoint / OneDrive works too, but the business-development skill is written against SharePoint list items by default.

## 8. Verify the Microsoft To Do mirror list

`daily-task-prep` mirrors the final `## Today` section of `clawchief/tasks.md` into a dedicated Microsoft To Do list so the principal can see today's list on their phone / desktop To Do app without touching the markdown file. Pick a list name, put it in `workspace/TOOLS.md` as `{{MSTODO_MIRROR_LIST}}`, and make sure the list exists:

```bash
# list all Microsoft To Do lists for the current user
m365 todo list list --output json

# create the mirror list if it does not exist yet
m365 todo list add --name "{{MSTODO_MIRROR_LIST}}"

# smoke-test: list tasks in it (should return [] on a fresh list)
m365 todo task list --listName "{{MSTODO_MIRROR_LIST}}" --output json
```

The mirror is one-way: `clawchief/tasks.md` is canonical, the To Do list is a read-only mirror for human visibility. Tasks completed inside the To Do app do not flow back — check them off in `clawchief/tasks.md` (or let the next `daily-task-prep` run rebuild the list from scratch).

## 9. Optional: verify OneNote / meeting notes source

If meeting notes are ingested from OneNote:

```bash
m365 request --url "@graph/me/onenote/pages?\$top=5" --output json
```

If meeting notes live as Word documents in a shared SharePoint folder, use `m365 spo file list` or `m365 onedrive list` instead.

## Stop conditions

Stop and fix setup if:

- the Entra app registration was never created or admin consent was not granted
- the wrong M365 account is authenticated
- `m365 outlook message list` works but `m365 outlook event list` does not (missing `Calendars.ReadWrite`)
- `m365 spo listitem list` fails against the tracker site (missing `Sites.ReadWrite.All` or the list is not shared to the operating account)
- a reply workflow uses a fresh `m365 outlook mail send` instead of a Graph `/me/messages/{id}/reply` call when an existing thread should have been preserved
