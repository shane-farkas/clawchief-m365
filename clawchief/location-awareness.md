# Location Awareness

Status: v1
Created: 2026-04-06
Purpose: make clawchief location-aware so {{OWNER_NAME}}'s task list, reminders, and appointment triage reflect where he physically is and whether a task is actually doable from that place.

## Ownership boundary

This file owns:
- place-based actionability
- travel-aware task interpretation
- location constraints that affect scheduling or task visibility

This file does not own:
- urgency ranking
- task-system schema
- inbox reply procedure
- meeting-note ingestion

## Core principle

Do not present a task as actionable if {{OWNER_NAME}} cannot physically do it from his current location.

Location is part of task context, not a nice-to-have.
A good daily plan should distinguish between:
- can do anywhere
- can do while traveling
- requires being at the principal's home base
- requires being at a specific appointment location
- can be delegated by {{ASSISTANT_NAME}}

## Current known anchor

- Some recurring tasks may be inherently home-based.
- Tasks involving in-person household or pet care should default to the principal's home base unless a different caregiver or location is explicitly noted.

## Operating rules

1. When {{OWNER_NAME}} is traveling, suppress or reframe home-only tasks as blocked, delegated, or rescheduled instead of showing them as normal active tasks.
2. When a task clearly depends on a place, add the location constraint into the task text.
3. When an appointment or meeting is time-bound and location-bound, preserve both in the task text when known.
4. If {{OWNER_NAME}}'s travel status creates a conflict with an existing task, update the clawchief/tasks.md task in the same turn.
5. If a task is still important but cannot be done from {{OWNER_NAME}}'s current location, choose one of these states:
   - blocked while traveling
   - delegate to someone else
   - reschedule to the next plausible date/location window
6. Prefer specific language over implicit assumptions. Example: "blocked while the principal is traveling; home-base task" is better than silently leaving the task open.
7. Use calendar/travel context plus explicit statements from the principal as the source of truth for current location assumptions.
8. For scheduling decisions, the designated family/shared conflict calendar is a required travel source of truth, not optional supporting context.
9. If location is uncertain and it changes whether the task is actionable or whether the principal can take a meeting, ask or mark the assumption clearly and do not book.

## Integration points

- clawchief/tasks.md task wording, section, due date, and labels should reflect location constraints when relevant.
- `clawchief/priority-map.md` already treats travel and schedule integrity as priority context.
- `daily-task-prep` should use this rule when curating the `## Today` section.
- Heartbeat check-ins should avoid nudging {{OWNER_NAME}} about tasks he cannot physically do from where he is.
- The executive-assistant workflow should use this rule before offering times or treating travel windows as available.

## Examples

- "Handle pet-care task" while the principal is traveling -> move out of `Today`, mark blocked while traveling, and make the home-base constraint visible.
- "Phone call with family" while traveling -> still actionable because location does not block it.
- "Intro call" while traveling -> still actionable unless travel logistics or time zone conflict interfere.
- "Check publication queue" while traveling -> still actionable if it can be done online.

## Follow-through

When a new recurring place-based pattern appears, compile it into clawchief instead of relying on chat memory.
