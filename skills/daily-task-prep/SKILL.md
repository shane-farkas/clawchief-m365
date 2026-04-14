---
name: daily-task-prep
description: "Prepare {{OWNER_NAME}}'s task list for the day using `clawchief/tasks.md` plus their calendars. Use when a cron or direct request asks to prepare today's tasks before the day starts; when recurring weekday tasks, due-today backlog items, and principal-owned meetings / calls should be added to `## Today`; or when the task list needs a safe early-morning refresh without overwriting manual priorities."
---

# Daily Task Prep

Use `clawchief/tasks.md` as the canonical live task file and `clawchief/tasks-completed.md` as the completed-task archive.

## Operating mode

This skill is **not gated** by the `read_only` knob in `workspace/TOOLS.md`:

- `clawchief/tasks.md` and `clawchief/tasks-completed.md` are local files, not Graph state, so markdown edits continue regardless of operating mode.
- `m365 outlook event list` / `m365 request --url "@graph/me/calendars"` are reads, not writes.
- The Microsoft To Do mirror is the one Graph write this skill makes, and it stays **on** in both modes on purpose: the blast radius is a single private list the principal controls, the `Tasks.ReadWrite` Graph scope is granted in both the read-only and write-mode permission sets in `SETUP-M365.md`, and losing the mirror defeats the point of having today's list on the principal's phone.

If the principal explicitly says "no M365 writes of any kind, including To Do" then stop running the mirror step and leave `clawchief/tasks.md` as the only surface.

## Core rules

- read `clawchief/priority-map.md` before regrouping or inserting active tasks
- preserve existing manually added open tasks in `## Today` unless they are obviously stale past meetings
- on weekdays, treat `## Every weekday` as the recurring seed list
- on weekends, do not auto-add `## Every weekday` items unless explicitly asked
- promote items due today from `## Backlog with due date` into `## Today`, and remove the backlog copy in the same edit
- scan `## Recurring reminders` and add any reminder that is due today into `## Today` without deleting the recurring source item
- add principal-owned meetings and calls for today to `## Today`
- exclude personal or family calendar blocks that are only conflict sources, not principal-owned tasks
- keep assistant tasks clearly separate from principal tasks
- archive tasks completed yesterday out of `clawchief/tasks.md` into `clawchief/tasks-completed.md`
- keep tasks completed today in `clawchief/tasks.md` until the next morning's prep run unless the user explicitly wants earlier cleanup
- after the markdown edit settles, mirror the final principal `## Today` list into the Microsoft To Do list named `{{MSTODO_MIRROR_LIST}}` so the principal sees it on their phone / desktop To Do app (mirror is one-way, markdown stays canonical)
- update the file's `Last updated` timestamp
- stay silent unless something needs human attention

## Preparation workflow

1. read `clawchief/tasks.md`
2. read `clawchief/priority-map.md`
3. read `clawchief/tasks-completed.md` if it exists
4. determine whether today is a weekday
5. archive tasks completed yesterday from `clawchief/tasks.md` into `clawchief/tasks-completed.md`
6. build the candidate `## Today` list from:
   - current open `## Today` tasks worth carrying forward
   - weekday recurring items from `## Every weekday` if today is Monday-Friday
   - items in `## Backlog with due date` that are due today
   - items in `## Recurring reminders` whose recurrence makes them due today
   - today's principal-owned meetings / calls from calendar
7. remove duplicates by normalized task text while keeping the most specific wording already present in the file
8. preserve or assign each active task to the best matching owner section plus program / person grouping header
9. reorder the open tasks in priority-first order within each owner section
10. write back only the minimal necessary edits
11. mirror the final principal `## Today` list into Microsoft To Do (see `Microsoft To Do mirror` below)

## Calendar workflow

Use the `m365` CLI ([cli-microsoft365](https://github.com/pnp/cli-microsoft365)) to inspect the principal's visible calendars before adding meeting tasks.

Useful pattern (today's events on the default calendar):

```bash
m365 outlook event list \
  --calendarName Calendar \
  --startDateTime "$(date -u +%Y-%m-%dT00:00:00Z)" \
  --endDateTime   "$(date -u -v+1d +%Y-%m-%dT00:00:00Z 2>/dev/null || date -u -d '+1 day' +%Y-%m-%dT00:00:00Z)" \
  --output json
```

If the principal has more than one calendar, enumerate them first through Graph and repeat the `event list` call per calendar:

```bash
m365 request --url "@graph/me/calendars" --output json
```

Only add calendar items that the principal is actually expected to attend.

## Microsoft To Do mirror

After the markdown task file has been rewritten, rebuild the Microsoft To Do list named `{{MSTODO_MIRROR_LIST}}` from the **principal** `## Today` section. The list is a human-visibility surface, not a second source of truth — `clawchief/tasks.md` remains canonical.

Run the mirror only when the principal `## Today` list changed during this run (or when the mirror list is currently empty). If tasks.md was not modified, skip the mirror entirely.

Mirror logic:

1. read the current tasks on the mirror list:

   ```bash
   m365 todo task list --listName "{{MSTODO_MIRROR_LIST}}" --output json
   ```

2. if the list does not exist yet, create it once:

   ```bash
   m365 todo list add --name "{{MSTODO_MIRROR_LIST}}"
   ```

3. delete every existing task on the mirror list:

   ```bash
   m365 todo task remove --listName "{{MSTODO_MIRROR_LIST}}" --id <taskId> --force
   ```

4. for each principal-owned item in the new `## Today` section of `clawchief/tasks.md`, create a task:

   ```bash
   m365 todo task add \
     --listName "{{MSTODO_MIRROR_LIST}}" \
     --title "<task text from tasks.md>"
   ```

Optional enrichments when the task text supports them:
- if the task has a timed due (for example `2026-04-15 14:30 {{TIMEZONE}}`), pass `--dueDateTime <ISO 8601 UTC>`
- if the task is tagged P0 / P1 in the priority map, pass `--importance high`
- do not try to sync `bodyContent`, categories, or reminders — keep the mirror simple

What the mirror must **not** do:
- do not include assistant-owned tasks (they are not for the phone)
- do not include `## Every weekday`, `## Backlog with due date`, or `## Recurring reminders` sources — only the materialized `## Today`
- do not try to read Microsoft To Do state back into `clawchief/tasks.md`; MS To Do is downstream only
- if `m365 todo` calls fail, leave `clawchief/tasks.md` alone, log a short note, and continue — the markdown ledger must never be blocked on the mirror

## Task text rules

- use concise one-line tasks
- keep existing wording when it is already good
- use `YYYY-MM-DD` for all-day due dates and `YYYY-MM-DD HH:MM TZ` for timed due dates
- if a backlog due-date item is promoted into `## Today`, remove the backlog copy immediately
- for recurring reminders, keep the recurring source entry in place and only add the due instance into `## Today`

## Safety

- do not wipe `## Today` just to rebuild it
- do not archive recurring source items from `## Recurring reminders`
- do not archive tasks completed today during the same day's prep run
- if calendar access fails, still do file-based prep and only notify the user if the failure matters
- if the Microsoft To Do mirror call fails, keep the markdown edits and log the failure quietly — never roll back `clawchief/tasks.md` because of a mirror failure
- if nothing needs to change, do nothing
