---
name: business-development
description: "Manage {{BUSINESS_NAME}} business-development and outreach-tracking work using Microsoft 365 via the m365 CLI. Use when handling prospecting replies, referral-partner outreach, updating the outreach tracker, logging lead status changes, booking or confirming outreach meetings tied to a lead / prospect, or maintaining the operational record of sales / outreach conversations. Prefer this skill over executive-assistant whenever the task touches the outreach tracker, lead status, prospect pipeline, or referral-partner outreach, even if scheduling is involved."
---

# Business Development

Use this skill for outreach and prospect tracking work. Keep it separate from generic executive-assistant inbox clearing.

## Read these first at the start of every run

- `clawchief/priority-map.md`
- `clawchief/auto-resolver.md`
- `workspace/TOOLS.md`
- `skills/business-development/resources/partners.md`

## Operating mode gate (check before every write action)

Read the `Operating mode` section of `workspace/TOOLS.md` at the start of every run and obey the `read_only` knob before taking any Graph / SharePoint write action.

When `read_only: true`:
- read the tracker freely (`m365 spo listitem list`, `m365 spo field list`), read the inbox / sent folder freely, read inbound replies freely
- **do not** call any of these: `m365 outlook mail send`, `m365 outlook message move`, `m365 spo listitem add`, `m365 spo listitem set`, `m365 spo listitem remove`, or any Graph write against `/me/messages` or the tracker list
- every would-be write becomes a drafted proposal posted to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` containing:
  - the lead name / partner / thread
  - what the tracker change would be (new row, status update, follow-up timestamp) with the exact list item id when updating
  - what the email action would be (initial outreach / follow-up / reply) with a preview of recipients + subject + body
- you may still note in-memory that a row *would* be updated for downstream cadence reasoning inside the same run, but must not persist that assumption back to the tracker

When `read_only: false`, every action below fires for real.

## Core rules

- the outreach tracker is the live source of truth; do not treat local prospect files as current state
- do not silently broaden default prospecting beyond the configured target market / geography without explicit direction
- verify a working website and a real public email before adding a new lead unless the user explicitly waives that requirement
- ignore placeholder or junk addresses from site code
- sweep sent mail so unanswered outreach does not disappear
- if the work touches lead status, pipeline state, or the outreach tracker, this skill owns it even when scheduling is part of the job

## Source of truth

The outreach tracker lives in a SharePoint list (not a Google Sheet, not a local CSV).

- Site URL: `{{SHAREPOINT_TRACKER_SITE_URL}}`
- List title: `{{SHAREPOINT_TRACKER_LIST_TITLE}}`

Treat this as the live source of truth for outreach status.
Do not rely on local `.md` or `.csv` prospect files as the current record.

## Current focus

Customize the business-development playbook in `workspace/TOOLS.md`.

At minimum define:
- target geography
- target market or target segments
- default daily batch size
- verification requirements
- any follow-up cadence overrides

If no more specific override exists, default to:
- prospecting inside `{{TARGET_MARKET}}` in `{{TARGET_GEOGRAPHY}}`
- adding only verified leads
- using the default follow-up cadence in this skill

## When to update the tracker

Update the tracker every time outreach state changes.

That includes when you:
- send the initial outreach email
- get any meaningful reply
- ask for a meeting
- book, confirm, reschedule, or cancel a meeting
- record a decline / not-a-fit outcome
- learn a follow-up or next-step detail worth preserving

Do this before you mark the thread handled.

## Inbound reply operating procedure

When partner / referral emails start coming back, process the inbox and the tracker as one workflow.

1. pull inbound messages by time window using Outlook message list (Outlook does not have Gmail's query syntax — use folder + start/end time windows)
2. review each inbound thread and identify the current state:
   - positive interest
   - meeting requested / meeting booked
   - decline / not a fit
   - question that needs a response
   - auto-reply / invalid contact / left organization
3. check whether the person already exists in the tracker
4. update the row with what changed
5. only after the tracker is current should the email be considered handled
6. if a meeting is booked through a scheduler, update the row immediately after the booking succeeds

Useful patterns:

```bash
# last 30 days of inbox, JSON for downstream filtering by sender / subject
m365 outlook message list \
  --folderName inbox \
  --startTime "$(date -u -v-30d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '-30 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --output json

# server-side filter by sender domain via Graph
m365 request \
  --url "@graph/me/mailFolders/inbox/messages?\$filter=from/emailAddress/address eq 'partner@example.com'&\$top=25&\$orderby=receivedDateTime desc" \
  --output json
```

When replying in-thread, POST to `/me/messages/{id}/reply` (or `/replyAll`) via `m365 request` so the conversation stays intact. See the executive-assistant skill for the exact reply pattern.

## Tracker read / write guidance

Read the current list schema before writing. Do not assume an old example column set is still correct.

Useful patterns:

```bash
# read every list item (paginated automatically)
m365 spo listitem list \
  --webUrl "{{SHAREPOINT_TRACKER_SITE_URL}}" \
  --listTitle "{{SHAREPOINT_TRACKER_LIST_TITLE}}" \
  --output json

# inspect the list's actual fields before writing so you use real internal names
m365 spo field list \
  --webUrl "{{SHAREPOINT_TRACKER_SITE_URL}}" \
  --listTitle "{{SHAREPOINT_TRACKER_LIST_TITLE}}" \
  --output json

# add a new lead row (use each field's internal name, not the display name)
m365 spo listitem add \
  --webUrl "{{SHAREPOINT_TRACKER_SITE_URL}}" \
  --listTitle "{{SHAREPOINT_TRACKER_LIST_TITLE}}" \
  --Title "Example Partner" \
  --Company "Example Co" \
  --ContactEmail "ops@example.com" \
  --Status "Initial outreach sent" \
  --LastTouch "$(date -u +%Y-%m-%d)"

# update an existing row by item id
m365 spo listitem set \
  --webUrl "{{SHAREPOINT_TRACKER_SITE_URL}}" \
  --listTitle "{{SHAREPOINT_TRACKER_LIST_TITLE}}" \
  --id 42 \
  --Status "Meeting booked" \
  --LastTouch "$(date -u +%Y-%m-%d)"
```

Prefer updating by the list's actual current internal field names rather than assuming a frozen fixed layout. If the tracker uses an Excel workbook in SharePoint / OneDrive instead of a SharePoint list, substitute `m365 request` calls against the Graph `/me/drive/items/{id}/workbook/tables/{tableName}/rows` endpoints and document that choice in `workspace/TOOLS.md`.

## Follow-up cadence

For unanswered outreach where the lead should stay active, use this default cadence unless the user overrides it:

- first follow-up: about 2 days after the last unanswered outbound
- second follow-up: about 5 days after the previous follow-up
- third follow-up: about 7 days after the previous follow-up

Rules:
- reset the clock after each new outbound follow-up
- record each follow-up in the tracker notes
- after the third unanswered follow-up, stop the automatic sequence and surface the lead if it still matters
- do not auto-follow up when the thread is sensitive, clearly closed, or the user told you to stop

## Default outbound workflow

1. verify the lead is not already in the tracker
2. verify the lead matches the configured target market / geography
3. verify a working website unless explicitly waived
4. inspect the website for a real public email address before leaving email blank
5. send the initial outreach email using `resources/partners.md` via `m365 outlook mail send`
6. add or update the tracker row immediately after each action
7. sweep sent mail for unanswered outreach and follow up on cadence

## Operating standard

- keep outreach state current as part of doing the work, not as an afterthought
- preserve thread context and recipient context before replying
- process inbound partner replies and tracker updates as one combined workflow
- when an outreach conversation turns into scheduling, coordinate the meeting and then update the tracker immediately after the scheduling action succeeds
- use the priority map to decide what should interrupt the principal and what can be batched
- use the auto-resolver to decide when to act directly versus draft or escalate
