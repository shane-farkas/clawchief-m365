# Meeting Notes Ingestion

Status: template v1
Purpose: make AI-generated or shared meeting notes part of the always-on chief-of-staff loop so that decisions, tasks, follow-ups, and auto-resolve opportunities do not die in documents.

## Source

Treat meeting notes as a live operational signal source.

These notes might come from:
- OneNote notebooks shared to the assistant account
- Word documents stored in a known SharePoint / OneDrive folder
- docs created by an AI notetaker
- exported meeting summaries
- Teams meeting transcripts stored in OneDrive
- transcripts stored in any other known folder

Customize the exact source for your environment.

## Canonical state files

- Policy: `clawchief/meeting-notes.md`
- Processed-doc ledger: `workspace/memory/meeting-notes-state.json`
- Task sink: `clawchief/tasks.md`
- Priority / urgency: `clawchief/priority-map.md`
- Auto-resolution policy: `clawchief/auto-resolver.md`

## Core rule

Every recurring executive-assistant sweep and every heartbeat should check for newly shared or recently updated meeting notes that have not yet been processed.

If a note is new and relevant:
- read it
- extract decisions, commitments, open questions, and action items
- add principal tasks to `clawchief/tasks.md`
- add assistant tasks to `clawchief/tasks.md` or auto-resolve them when allowed
- update the relevant live source of truth (calendar, tracker, CRM, inbox, docs, etc.)
- record the note as processed in the meeting-notes ledger

## What counts as successful ingestion

A meeting note is not "handled" just because it was read.

It is only handled when the important outputs have been pushed into the system:
- tasks added or updated
- follow-ups created
- tracker / calendar / inbox state updated when appropriate
- auto-resolved actions completed when safe
- processed-doc ledger updated

## Processing workflow

1. search the configured source for recently shared or recently updated meeting notes
2. compare candidate docs against `workspace/memory/meeting-notes-state.json`
3. for each unprocessed note:
   - read the note text
   - identify:
     - principal action items
     - assistant action items
     - follow-ups waiting on others
     - decisions / commitments
     - items that can be auto-resolved right now
4. classify each extracted item through the priority map
5. run the auto-resolver policy
6. update `clawchief/tasks.md` and any other live source of truth in the same turn
7. record the note as processed with timestamp and a short summary of what was extracted / done

## What to extract from notes

Look specifically for:
- explicit action items
- implied follow-ups
- deadlines
- promises the principal made
- introductions to send
- docs / links / materials to share
- scheduling next steps
- outreach / partnership next steps
- GTM / content tasks
- legal / policy questions that need review
- anything that should become a task, reminder, draft, or auto-resolve action

## Auto-resolve bias for meeting notes

Meeting notes often contain operational follow-ups that should be auto-resolved when safe.

Good auto-resolve examples:
- add tasks for the principal or assistant
- create a follow-up reminder
- update a tracker after a partner / prospect meeting
- draft or send a low-risk scheduling follow-up when authority is clear
- update calendar / task state based on what was agreed
- send the principal a short summary of what was turned into action

Draft-first or escalate examples:
- sensitive legal / policy wording
- investor or board positioning
- emotionally sensitive follow-up language
- unclear commitments where the notes are ambiguous

## Ledger format

Use `workspace/memory/meeting-notes-state.json` to avoid reprocessing the same doc repeatedly.

Each processed record should include, when known:
- `docId`
- `title`
- `processedAt`
- `meetingDate`
- `status`
- `summary`

## Frequency expectation

This is not a once-a-day batch.

The intended behavior is:
- executive-assistant sweep keeps checking
- heartbeat reinforces the same behavior
- low-risk actions get resolved automatically
- the principal only sees what matters or what still needs them
