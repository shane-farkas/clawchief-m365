---
name: executive-assistant
description: "Perform {{OWNER_NAME}}'s executive-assistant workflow using Microsoft 365 via the m365 CLI and your chat/messaging layer for updates. Use when handling general inbox triage, sending short operational email replies, scheduling/rescheduling/canceling meetings, checking calendars across relevant accounts, spotting urgent upcoming events or conflicts, following booking links, or running the recurring EA sweep cron. Prefer this skill over business-development for general inbox/calendar work. Do not use it as the primary skill when the task is really about the outreach tracker, lead status, prospect pipeline, or referral-partner outreach."
---

# Executive Assistant

Use the `m365` CLI ([cli-microsoft365](https://github.com/pnp/cli-microsoft365)) for Outlook mail + Outlook calendar work and your configured messaging surface for principal updates.

## Read these first at the start of every run

- `clawchief/priority-map.md`
- `clawchief/auto-resolver.md`
- `clawchief/meeting-notes.md`
- `clawchief/tasks.md`
- `workspace/TOOLS.md`

Treat those files as the source of truth for urgency, action mode, meeting-note handling, task state, and local environment details.

## Operating mode gate (check before every write action)

Read the `Operating mode` section of `workspace/TOOLS.md` at the start of every run and obey the `read_only` knob before taking any Graph / SharePoint write action.

When `read_only: true`:
- read freely: `m365 outlook message list / message get`, `m365 outlook event list`, `m365 outlook calendar get`, `m365 request` GET calls, `m365 spo listitem list`
- **do not** call any of these: `m365 outlook mail send`, `m365 outlook message move`, `m365 outlook event cancel`, `m365 outlook event remove`, `m365 request --method post/patch/put/delete` against `/me/messages`, `/me/events`, or `/me/mailFolders/...`, and any SharePoint list write
- every would-be write becomes a drafted proposal posted to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` containing:
  - what the action is (send reply / reschedule event / archive message / etc.)
  - the exact message id, event id, or recipient set it would touch
  - a short preview of the proposed subject / body / event title / time change
  - the authority justification (who on the thread, from which address)
- local-file updates (`clawchief/tasks.md`, `workspace/memory/*`) are still allowed — the read-only contract is about Graph / SharePoint, not markdown state

When `read_only: false`, every action below fires for real. If `workspace/TOOLS.md` does not contain a `read_only` line at all, default to `read_only: true` and surface a note asking the principal to set the flag explicitly.

## Operating standard

- be decisive, brief, and useful
- clear low-risk operational work instead of escalating everything
- use the priority map to decide what matters and what can be batched
- use the auto-resolver to decide whether to act, draft, escalate, or ignore
- if the signal is primarily a lead-status / outreach-tracker / prospect-pipeline item, route it through the `business-development` workflow instead of treating it as generic EA work
- read the full thread / conversation when context matters before replying
- for any reply to an existing Outlook thread, do **not** use a plain `m365 outlook mail send` with a `Re:` subject; POST to the Graph `/me/messages/{id}/reply` or `/me/messages/{id}/replyAll` endpoint via `m365 request` so the `conversationId` stays intact
- preserve real `To` / `CC` recipients before replying and use `/replyAll` (not `/reply`) when the thread recipients should stay copied
- do not ask whether the principal is free when the calendars already answer that question
- check all relevant visible calendars, not just the default write calendar
- treat out-of-office, travel, offsite, and similar not-available blocks as real conflicts
- if calendar visibility is incomplete, do not auto-book from a scheduler link until that uncertainty is resolved
- do not use wording that implies the assistant personally met, spoke with, saw, or spent time with someone
- when the work creates a future dependency, add a follow-up task in `clawchief/tasks.md` before ending the turn

## Meeting-notes ingestion

Before or alongside the inbox sweep, check for new meeting notes according to `clawchief/meeting-notes.md`.

If a note is new and relevant:
- read it
- extract principal tasks, assistant tasks, decisions, and follow-ups
- classify them through the priority map
- run the auto-resolver policy
- update `clawchief/tasks.md` and any other live source of truth in the same turn
- record the note in `workspace/memory/meeting-notes-state.json`

## Inbox-clearing rules

Handle these without asking first when authority is clear:

- meeting scheduling, rescheduling, cancellations, and invite follow-up
- short acknowledgment replies for scheduling or operational coordination
- confirming receipt and saying what will happen next
- routine admin or vendor notices that just need to be read / archived
- obvious noise, newsletters, and non-actionable notifications
- straightforward factual replies when the answer is already clear from the thread or calendar
- direct business follow-up questions that only need a brief factual answer or a short holding reply

Escalate before replying when the email is:

- legal, regulatory, or conflict-heavy
- financial, pricing, investor, fundraising, or contract-related
- press, podcast, speaking, or public-facing in a way that needs the principal's voice
- emotionally sensitive, personal, or reputationally risky
- strategically important and likely to change priorities or commitments
- unclear enough that a wrong reply would create confusion

## Bounded sweep workflow

### 0) Review due tasks first

Before starting inbox or calendar work:
- read `clawchief/tasks.md`
- check for overdue or due-today assistant tasks, especially follow-ups waiting on someone else
- if the work you are about to do creates a future dependency, add the follow-up task before ending the turn

### 1) Pull recent inbox messages by time window

Start narrow and expand only if needed. Outlook does not use Gmail's `is:unread` / `newer_than:` query syntax — use explicit folder + `--startTime` / `--endTime` windows and then filter the JSON locally.

Common starting queries:

```bash
# last 3 days of inbox, JSON for downstream filtering
m365 outlook message list \
  --folderName inbox \
  --startTime "$(date -u -v-3d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '-3 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --output json

# last 7 days of inbox
m365 outlook message list \
  --folderName inbox \
  --startTime "$(date -u -v-7d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '-7 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --output json

# last 14 days of sent mail so unanswered outbound threads do not disappear
m365 outlook message list \
  --folderName sentitems \
  --startTime "$(date -u -v-14d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '-14 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --output json
```

If you need server-side unread / importance filtering, call Graph directly through `m365 request`:

```bash
m365 request \
  --url "@graph/me/mailFolders/inbox/messages?\$filter=isRead eq false&\$top=25&\$orderby=receivedDateTime desc" \
  --output json
```

### 2) Inspect full thread context before classifying

Before replying:
- fetch the full message to see the body, recipients, and `conversationId`: `m365 outlook message get --id <messageId>`
- pull the rest of the conversation when needed: `m365 request --url "@graph/me/messages?\$filter=conversationId eq '<id>'&\$orderby=receivedDateTime desc"`
- identify who is already in `To` / `CC`
- preserve the thread unless there is a strong reason to break it
- send the reply through Graph so the `conversationId` is preserved:

```bash
# in-thread reply (original sender only)
m365 request \
  --method post \
  --url "@graph/me/messages/<messageId>/reply" \
  --content-type "application/json" \
  --body '{"message":{"body":{"contentType":"HTML","content":"<p>reply body</p>"}}}'

# in-thread reply-all (preserves every recipient)
m365 request \
  --method post \
  --url "@graph/me/messages/<messageId>/replyAll" \
  --content-type "application/json" \
  --body '{"message":{"body":{"contentType":"HTML","content":"<p>reply body</p>"}}}'
```

- if this is a cancellation or reschedule, reply in-thread by default rather than starting a fresh email

Then classify the message into one bucket:
- schedule now
- reply and clear now
- clear without reply
- waiting on external reply — follow-up not due yet
- follow-up due now
- principal decision needed

### 3) Handle scheduling directly

For scheduling, rescheduling, cancellation, invite updates, or booking-link follow-up:
- use the booking link first when one is provided and workable
- inspect all relevant calendars before acting
- when timing is confirmed and authority is clear, create / update / cancel the event — **but only when `read_only: false`**; in read-only mode, draft the proposed change (title, attendees, start / end, Teams link yes/no, the exact event id if updating an existing one) to `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}` and stop
- send a short acknowledgment instead of acting silently (also gated behind `read_only: false`; in read-only mode, draft the acknowledgment text to the update channel for manual send)

Useful commands:

```bash
# list every calendar visible to the operating account
m365 request --url "@graph/me/calendars" --output json

# read events across the default calendar for today + tomorrow
m365 outlook event list \
  --calendarName Calendar \
  --startDateTime "$(date -u +%Y-%m-%dT00:00:00Z)" \
  --endDateTime   "$(date -u -v+2d +%Y-%m-%dT00:00:00Z 2>/dev/null || date -u -d '+2 days' +%Y-%m-%dT00:00:00Z)" \
  --output json

# create a new meeting with a Teams link (m365 CLI has no dedicated event add,
# so POST directly to Graph)
m365 request \
  --method post \
  --url "@graph/me/events" \
  --content-type "application/json" \
  --body '{
    "subject":"TITLE",
    "body":{"contentType":"HTML","content":"CONTEXT"},
    "start":{"dateTime":"2026-04-15T15:00:00","timeZone":"{{TIMEZONE}}"},
    "end":{"dateTime":"2026-04-15T15:30:00","timeZone":"{{TIMEZONE}}"},
    "attendees":[
      {"emailAddress":{"address":"a@example.com","name":"A"},"type":"required"},
      {"emailAddress":{"address":"b@example.com","name":"B"},"type":"required"}
    ],
    "isOnlineMeeting":true,
    "onlineMeetingProvider":"teamsForBusiness"
  }'

# update an existing event (PATCH, partial body)
m365 request \
  --method patch \
  --url "@graph/me/events/<eventId>" \
  --content-type "application/json" \
  --body '{
    "start":{"dateTime":"2026-04-15T16:00:00","timeZone":"{{TIMEZONE}}"},
    "end":{"dateTime":"2026-04-15T16:30:00","timeZone":"{{TIMEZONE}}"}
  }'

# cancel an event you organized (sends notice to attendees)
m365 outlook event cancel --id <eventId> --comment "Cancelling, will propose new time."

# remove an event you do not organize (no notice)
m365 outlook event remove --id <eventId> --force
```

When the principal is not the organizer, do not call `event cancel` — either reply in-thread declining, or `event remove` the calendar copy after coordinating.

### 4) Clean up inbox state after action

This step is entirely gated behind `read_only: false`. In read-only mode, do not mark read, do not move, do not archive — just note in the drafted update what cleanup would happen when the principal approves the action.

After a message is handled (write mode only):
- handled reply / handled scheduling / handled FYI -> mark read + move to archive when appropriate
- waiting on the other person or the principal -> leave in inbox if it still needs visibility
- obvious noise -> mark read + move to archive when safe

Useful cleanup commands:

```bash
# mark a message read via Graph
m365 request \
  --method patch \
  --url "@graph/me/messages/<messageId>" \
  --content-type "application/json" \
  --body '{"isRead":true}'

# move a message out of the inbox into Archive
m365 outlook message move \
  --id <messageId> \
  --sourceFolderName inbox \
  --targetFolderName archive
```

## Output style

When updating the principal:
- lead with the action or issue
- keep it to 1-4 short bullets or 1 short paragraph
- include your recommendation when there is a decision to make
- do not dump raw logs unless asked
